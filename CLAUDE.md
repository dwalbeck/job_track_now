# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Job Track Now is a full-stack application for managing job applications with AI-powered resume customization and cover letter 
generation. Built as a 3-tier containerized application with React frontend, FastAPI backend, and PostgreSQL database.

## Docker
This project uses **Docker Compose** to run the three containers for this project. The **frontend** application is a React 
portal website.  The **backend** is a Python API service, performing all the logic and interaction with the DB.  The third 
container runs the PostgreSQL data.

### Important Points

- Do not start, build or use containers by using the **docker** command. Use **docker compose** instead. Using **docker** will break the docker compose stack
- Both frontend and backend containers have the code copied into them, so to see any code changes requires that they be rebuilt

**Linux:**
```bash
# Start all services
docker compose up --build -d

# View logs
docker compose logs -f

# Stop services
docker compose down

# Rebuild specific service
docker compose build --no-cache backend
docker compose build --no-cache frontend

# Shell into containers
docker exec -it api.jobtracknow.com bash
docker exec -it portal.jobtracknow.com bash
docker exec -it psql.jobtracknow.com bash
```

**Windows/MacOS:**
```bash
# Start all services
docker compose -f docker-compose-win.yml up --build

# Rebuild specific service
docker compose -f docker-compose-win.yml build --no-cache backend
docker compose -f docker-compose-win.yml build --no-cache frontend
```

### Service Communication

**Linux:** Uses custom network (172.20.0.0/16) with static IPs and domain names configured in /etc/hosts:
- Frontend: portal.jobtracknow.com (172.20.0.5:80)
- Backend: api.jobtracknow.com (172.20.0.10:8000)
- Database: psql.jobtracknow.com (172.20.0.15:5432)

**Windows/MacOS:** Uses localhost with port mapping:
- Frontend: localhost:80 or localhost:3000
- Backend: localhost:8000
- Database: localhost:5432

**Windows/MacOS (Development):**
```bash
# Start all services
docker compose -f docker-compose-win.yml up --build

# Rebuild specific service
docker compose -f docker-compose-win.yml build --no-cache backend
docker compose -f docker-compose-win.yml build --no-cache frontend
```

### Standards


## Backend API Service
The backend service is an API written in Python 3.10+, utilizing the package FastAPI

### Backend Structure
```
backend/app/
├── api/           # API route handlers (jobs, contacts, calendar, notes, resume, letter, etc.)
├── core/          # Core configuration (config.py, database.py)
├── models/        # SQLAlchemy models (models.py)
├── schemas/       # Pydantic schemas for validation
├── middleware/    # Custom middleware (logging_middleware.py)
├── utils/         # Utilities (ai_agent.py, conversion.py, file_helpers.py, logger.py, etc.)
|   └── prompts    # AI prompt templates
└── main.py        # FastAPI application entry point
```
**Key Backend Files:**
- `main.py`: FastAPI app with CORS, exception handlers, router registration
- `core/config.py`: Settings loaded from environment variables
- `core/database.py`: Database session management
- `models/models.py`: SQLAlchemy ORM models for all tables
- `utils/ai_agent.py`: OpenAI integration for resume rewriting and cover letter generation
- `utils/conversion.py`: File format conversion (docx, odt, pdf, html ↔ markdown)

**API Endpoints (all prefixed with `/v1`):**
- `/jobs` - Job CRUD, extraction, status updates
- `/contacts` - Contact management
- `/calendar` - Calendar/appointment management (month, week, day views)
- `/notes` - Notes management
- `/resume` - Resume upload, rewrite, download
- `/letter` - Cover letter generation
- `/convert` - File format conversion
- `/personal` - Personal settings
- `/files` - File operations
- `/reminder` - Reminder management
- `/export` - Export data to CSV

### Configuration Settings (.env)
```bash
# Database (change for Windows/MacOS)
DATABASE_URL=postgresql://apiuser:change_me@psql.jobtracknow.com:5432/jobtracker
POSTGRES_HOST=psql.jobtracknow.com  # Use 'db' for Windows/MacOS

# OpenAI
OPENAI_API_KEY=<your_key>
AI_MODEL=gpt-4o-mini

# File Storage
BASE_JOB_FILE_PATH=/app/job_docs
RESUME_DIR=/app/job_docs/resumes
COVER_LETTER_DIR=/app/job_docs/cover_letters
EXPORT_DIR=/app/job_docs/export

# Logging
LOG_LEVEL=DEBUG
LOG_FILE=api.log
```

### Logging & Info

- Structured logging via custom logger (`backend/app/utils/logger.py`)
- Logs to `/app/api.log` inside container
- Includes request/response logging middleware
- All database operations logged with operation type

```bash
# View backend logs
docker exec -it api.jobtracknow.com bash
tail -f /app/api.log

# API documentation available at:
# http://localhost:8000/docs (Windows/Mac)
# http://api.jobtracknow.com/docs (Linux)
```

### AI Integration

The application uses OpenAI API for:
1. **Job Description Extraction**: Extracts structured data from job posting URLs
2. **Resume Rewriting**: Rewrites resume to emphasize keywords from job posting
3. **Cover Letter Generation**: Creates customized cover letters based on job and resume

**Key AI Files:**
- `backend/app/utils/ai_agent.py`: OpenAI client wrapper
- Configuration via `OPENAI_API_KEY` in backend/.env
- Model selection configurable per-operation in Personal settings

### Async Polling Architecture (Resume Rewrite)

The resume rewrite process uses an async polling pattern to handle long-running AI operations without blocking the HTTP connection. This prevents browser connection pool exhaustion and provides better UX.

**Flow:**
1. User clicks "Finalize" on JobAnalysis page
2. Frontend calls `POST /v1/resume/rewrite` with `job_id`
3. Backend:
   - Creates process record in database
   - Starts background thread with thread-local database session
   - Returns HTTP 202 Accepted with `process_id` immediately
4. Frontend navigates to OptimizedResume page with polling state
5. OptimizedResume polls `GET /v1/process/poll/{process_id}` every 5 seconds
6. When process completes, fetches result from `GET /v1/resume/rewrite/{job_id}`
7. Displays resume comparison view

**Key Implementation Details:**

**Backend (`backend/app/api/resume.py:1268-1323`):**
```python
# CRITICAL: Background thread must use thread-local database session
# DO NOT pass request-scoped session to thread - causes HTTP blocking
def run_rewrite():
    thread_db = SessionLocal()  # Create new session for thread
    try:
        thread_ai_agent = AiAgent(thread_db)
        thread_ai_agent.resume_rewrite_process(job_id, process_id)
    finally:
        thread_db.close()

thread = threading.Thread(target=run_rewrite, daemon=True)
thread.start()
```

**Frontend (`frontend/src/pages/OptimizedResume/OptimizedResume.js:150-265`):**
- Polls every 5 seconds with 10 minute timeout (120 attempts)
- Handles process states: `running`, `complete`, `confirmed`, `failed`
- Shows loading message during polling
- Fetches final result when `complete` or `confirmed`

**Endpoints:**
- `POST /v1/resume/rewrite` - Initiates process, returns 202 with `process_id`
- `GET /v1/process/poll/{process_id}` - Returns process state
- `GET /v1/resume/rewrite/{job_id}` - Returns completed resume data

**Database:**
- `process` table tracks background operations
- Columns: `process_id`, `endpoint_called`, `running_method`, `running_class`, `completed`, `confirmed`, `failed`

**Why Thread-Local Session:**
Passing the request-scoped database session to the background thread keeps the HTTP connection open, blocking the browser's connection pool (typically 6 concurrent connections per origin). This prevents polling requests from being sent. Using `SessionLocal()` creates an independent session for the thread, allowing the HTTP connection to close immediately after returning 202.

### File Format Conversion
Supports conversion between: PDF, DOCX, ODT, HTML, Markdown
- Upload any format → converts to Markdown for AI processing
- Download customized resumes in multiple formats
- Original file used as template for formatting (DOCX recommended)

**Conversion utilities:**
- `backend/app/utils/conversion.py`: Main conversion logic
- Uses: pypandoc, mammoth, odt2md, docx2md, markitdown, html2docx, python-docx


## Frontend Portal
The Frontend service is a portal written in React 18, and served via Nginx reverse proxy.

### Frontend Structure

- `src/components` - modals, forms and other page elements
- `src/pages` - all the application pages
- `src/services/api.js` - a wrapper for making backend API calls
- `src/utils` - various tools and helper functions
```
frontend/src/
├── components/    # Reusable components (JobCard, Navigation, Calendar views, Forms, etc.)
├── context/       # React Context (JobContext, ReminderContext)
├── pages/         # Page components (Home, JobTracker, JobDetails, Resume, Calendar, etc.)
├── services/      # API service layer (api.js)
├── utils/         # Utilities (logger.js, dateUtils.js, phoneUtils.js, calendarUtils.js)
├── App.js         # Main app with routing
├── index.js       # Entry point
└── config.js      # Frontend configuration
```

**Key Frontend Files:**
- `services/api.js`: Centralized API client with request/response logging
- `context/JobContext.js`: Global job state management
- `pages/JobTracker/JobTracker.js`: Main job board with drag-and-drop (react-beautiful-dnd)
- `pages/JobDetails/JobDetails.js`: Job detail view with resume optimization, appointments, notes
- `pages/Resume/Resume.js`: Resume management (baseline and customized)
- `pages/OptimizedResume/OptimizedResume.js`: Resume keyword selection and rewrite interface
- `utils/logger.js`: Client-side logging to localStorage

### Configuration Settings (.env)
```bash
# API URL (change for Windows/MacOS)
REACT_APP_API_BASE_URL=http://api.jobtracknow.com  # Use http://localhost:8000 for Windows/MacOS
REACT_APP_TINYMCE_API_KEY=7qrgi9by3dbq7a1w58d4mdvcns386w3fmav69z9bnc6yibtd

# Logging
REACT_APP_LOG_LEVEL=DEBUG
REACT_APP_ENABLE_CONSOLE_LOGGING=true
```

### Logging & Info
- Browser localStorage logging (`frontend/src/utils/logger.js`)
- API requests/responses automatically logged
- Access via browser console: `logger.getLogs()`
```console
logger.getLogs()
logger.exportLogs()
logger.clearLogs()
```

### Common Commands
```bash
# Build command (in Docker): 
npm run build
# Dev server (if running locally): 
npm start
# Restart service in Docker container
uvicorn app.main:app --reload
# restarting Nginx
nginx -s stop
nginx -g 'daemon off;'
```


## Database

Data is stored into a PostgreSQL DB.  The full schema is available in the **docs/schema.sql** file

**Core Tables:**
- `job` - Job postings with status, interest level, salary, location
- `job_detail` - Job description, qualifications, keywords
- `resume` - Resume metadata (baseline/customized, format, title)
- `resume_detail` - Resume content (markdown, HTML, keywords, scores)
- `cover_letter` - Cover letters with length/tone settings
- `contact` - Contacts linked to jobs
- `calendar` - Appointments/interviews
- `note` - Job-related notes
- `reminder` - Reminders for follow-ups
- `personal` - User personal information and settings

**Enums:**
- `job_status`: applied, interviewing, rejected, no response
- `title`: recruiter, hiring manager, hr, engineer, vp, other
- `appt`: phone call, interview, on site, technical
- `comm`: phone, email, sms, message
- `file_fmt`: pdf, odt, md, html, docx
- `content_length`: short, medium, long
- `content_tone`: professional, casual, enthusiastic, informational

## Common Workflows

### Adding a New API Endpoint

1. Create route handler in `backend/app/api/<module>.py`
2. Define Pydantic schemas in `backend/app/schemas/<module>.py`
3. Add database models in `backend/app/models/models.py` (if needed)
4. Register router in `backend/app/main.py`
5. Add API method in `frontend/src/services/api.js`
6. Use in React components

### Adding AI Features

1. Add method to `AiAgent` class in `backend/app/utils/ai_agent.py`
2. Create endpoint in appropriate API router
3. Wire up in frontend

## Platform-Specific Notes

**Linux:** Full networking with custom domains via /etc/hosts. Nginx on port 80, backend on 8000.

**Windows/MacOS:** Limited networking, uses localhost + ports. Different docker-compose file (docker-compose-win.yml). Frontend environment must point to localhost:8000.

## Important Data Flows

1. **Job Application**: Create job → Extract details via AI → Store in job + job_detail tables
2. **Resume Optimization**: Upload resume → Convert to markdown → Select keywords → AI rewrite → Download
3. **Cover Letter**: Select job + resume → Configure tone/length → AI generation → Download
4. **Status Updates**: Drag job card → Update job_status + last_activity → Auto-move based on time threshold
