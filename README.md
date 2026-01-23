# Job Tracker Application

A comprehensive job tracking application built with a React frontend, FastAPI Python backend, and PostgreSQL database. 
This full-stack application helps you manage job opportunities throughout the hiring process with features for tracking 
applications, managing contacts, scheduling interviews, and organizing notes. It also includes a number of AI assistance 
for presenting your best self, which includes: resume rewriting tailored to job, cover letters, and researched 
company reporting, resume feedback and tips and other tools. Highly customizable to suit your personal preference 
and easily executed to be able to use all within a Docker stack.


## Features
- Job posting tracking and organization through 4 universal stages
- Job qualifications and keywords extracted and individually selectable for approval / removal
- Resume custom modified from baseline by AI and tailored to the Job Posting with reversible edits
- Automatic AI generated cover letter creation specific to Job posting
- AI researched company information report generation with interview tips
- AI tools that will rewrite sentences and paragraphs with full explanation of alterations
- AI generated elevator pitch using job posting description and resume content
- All data is exportable per section
- Supports docx, odt, pdf and html file formats
- Wysiwyg editor for making manual changes and adjustments to resume or cover letters
- Configurable use with multiple conversion methods, including third party services
- Automated configurable rotation of job posting status after given number of days
- Contacts available both globally and per Job Posting
- Generic notes used both globally and per Job Posting
- Calendar appointment both global and per Job
- Calendar reminders with notification alerts
- Running job score total tracked using average across interview scores and posting score given
- Unlimited baseline resumes can be created and used
- Selection of LLM models to use for different actions is configurable to your preference
- Intuitive, clean and easy to read display of information across all pages
- Dynamic searches that update display per character entered, makes identifying entries quick and easy
- Free stuffed animal! ... I mean, you can use it with your favorite stuffed animal


## 🏗️ Architecture Overview

The Job Tracker is built as a **3-tier containerized application**:

### **Frontend Service** (React + Nginx)
- **Technology**: React 18 with TypeScript-style components
- **Features**: Responsive UI, real-time search, drag-and-drop job management, OAuth2 sessions
- **URL**: https://jobtracknow.com or http://localhost
- **Purpose**: User interface for interacting with job data

### **Backend Service** (FastAPI + Python)
- **Technology**: Python 3.10 using FastAPI with SQLAlchemy ORM
- **Features**: RESTful API, automatic documentation, data validation
- **URL**: http://api.jobtracknow.com or http://localhost:8000
- **Purpose**: Business logic and database operations

### **Database Service** (PostgreSQL)
- **Technology**: PostgreSQL 15 with persistent storage
- **Features**: Relational data storage, ACID compliance
- **URL**: http://psql.jobtracknow.com:5432 or http://localhost:5432
- **Purpose**: Data persistence and complex querying

## 🚀 Quick Start with Docker

### Prerequisites

Software
- **Docker** (v20.10+) or **Docker Desktop** (4.43.2)
- **Docker Compose** (v2.0+)  (Included in Docker Desktop)
- **Git** (for cloning the repository or you can just download the zip file)

Service Subscriptions
- **OpenAI Key** (for customizing resume/cover letters) A fair amount of functionality will require having this key, but 
    you can sign-up for an account and only put $5 towards it and it will last for well over a month.  Create an account at 
    [OpenAI website](https://auth.openai.com/create-account/), during which you can also create an API key. If you lost, 
    skipped or forgot your API key, you can create new ones at [OpenAI API Keys](https://platform.openai.com/api-keys)
- **ConvertAPI** (optional) is a method for converting between formats.  It is by far the most accurate and best conversion method 
    from a **docx** source that I've found.  While they claim to convert between a large number of formats, honestly I would 
    only recommend it for docx files.  Other formats either don't work or are such poor quality conversion that you won't want 
    to use it. Using docx and this method will produce the best conversions by far.  You can sign-up for a 30-day free trial 
    at [ConvertApi](https://www.convertapi.com/a/signup) and you can find the API key at [ConvertApi API key](https://www.convertapi.com/a/authentication) 
    where you can use pre-existing keys made or create a new one - any will work.
- **TinyMCE** (optional) is a great wysiwyg text editor.  At one point this was free to use, but I guess those days are gone. 
    You do however get a 30-day free trial to use, which I recommend doing.  Sign-up at [TinyMCE](https://www.tiny.cloud/), after 
    which you will need to generate a key to use at [Tiny JWT](https://www.tiny.cloud/my-account/jwt/).  If you want to make 
    small edits to your resume or cover letters, then I highly recommend using this tool.


### Step-by-Step Setup

1. **Clone the Repository**
   ```bash
   git clone git@github.com:dwalbeck/job_track_now.git
   cd job_track_now
   ```

2. **Configure Environment Variables**
   ```bash
   cp ./job_track_now-api/env.example ./job_track_now-api/.env
   cp ./job_track_now-portal/env.example ./job_track_now-portal/.env
   # for Windows and Mac users
   cp ./job_track_now-api/env.example-win ./job_track_now-api/.env
   cp ./job_track_now-portal/env.example-win ./job_track_now-portal/.env
   ```
3. **Customize Configuration**

    Use your preferred text editor to modify the settings to your preference.  Everything is setup to simply work for
    your OS based on which env.example file you used.  Changing URL's, DB name, DB user, DB password can potentially 
    break things, so make sure it's worth it and that you understand how to fix things should things go bad.  Remember, 
    you can also change any setting you want later at any time as well.

4. **Build and Start All Services**
   ```bash
   # To both build the services are start them
   docker compose up --build -d

   # Or you can do my preference, which is build then start
   docker compose build
   docker compose up -d
   
   # Or for Windows OS and MacOS
   docker compose -f docker-compose-win.yml up --build -d
   
   # or separately
   docker compose -f docker-compose-win.yml build
   docker compose -f docker-compose-win.yml up -d
   
   ```
5. **Edit DNS Routing**
    ```bash
   # Skip this step for Windows and MacOS
   echo "172.20.0.5      jobtracknow.com
   172.20.0.10     api.jobtracknow.com
   172.20.0.15     psql.jobtracknow.com" >> /etc/hosts
   
   # this will effectively return these values upon DNS request for the sub-domains defined
    ```

6. **Verify Services are Running**
   ```bash
   docker compose ps
   
   # You should see the following:
   api.jobtracknow.com      job_track_now-backend    "python -m uvicorn a…"   backend    18 minutes ago   Up 18 minutes (healthy)   0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp
   portal.jobtracknow.com   job_track_now-frontend   "/usr/local/bin/dock…"   frontend   18 minutes ago   Up 18 minutes (healthy)   443/tcp, 0.0.0.0:80->80/tcp, [::]:80->80/tcp, 3000/tcp
   psql.jobtracknow.com     postgres:12              "docker-entrypoint.s…"   db         18 minutes ago   Up 18 minutes (healthy)   5432/tcp
   ``` 
    The key thing to note in all that output, is the column that states the current status, run time and health check, 
    which looks like: **Up 18 minutes (healthy)**


7. **Access the Application**
   - **Frontend**: https://jobtracknow.com
   - **Backend API**: https://api.jobtracknow.com
   - **API Documentation**: https://api.jobtracknow.com/docs
   - **Database**: postgresql://apiuser:change_me@psql.jobtracknow.com:5432/jobtracker
     - host: psql.jobtracknow.com
     - user: apiuser
     - password: change_me
     - port: 5432
     - database: jobtracker
     
    or for Windows and MacOS use:
   - **Frontend**: http://localhost (preferred) or http://localhost:3000
   - **Backend API**: http://localhost:8000
   - **API Documentation**: http://localhost:8000/docs
   - **Database**: postgresql://apiuser:change_me@localhost:5432/jobtracker
     - host: localhost
     - user: apiuser
     - password: change_me
     - port: 5432
     - database: jobtracker

NOTE: For **Windows** and **Mac** users, network interfaces work a bit differently then they do on Linux, and as such are not 
able to do aliasing or virtual addressing, which is used by Docker for some networking capabilities.  The short version 
is that this means you can't use a fake domain name for accessing things.  No biggie, just use the correct mapped port and 
the domain name **localhost**.

## 🔄 Usage

This section will quickly cover some key points for operation and functional capabilities for each of the pages.  It would 
be good to read through the **First Steps** listing and the remaining pages can be read should you have a question on a 
page, but you could skip reading the rest and still be able to operate things.

### First Steps

* Upon first starting up the application, your DB will be empty and you won't be able to login.  So creating a new user 
    account is pretty simple. On the left side menu at the bottom you'll see **Settings**, where you'll move your mouse 
    cursor to be, and then select **User** from the sub-menu.
* Once you create a user, you will no longer be able to access this page without being logged in, so make sure you at the 
    bare minimum fill in the **username** and **password** fields.
* You should now be able to login using the credentials that you defined in your profile.
* Many pages and features are dependent upon having a valid OpenAI AI key, so in the **Settings** sub-menu click on
  **General** and be sure to add your OpenAI API Key.  Optionally you can add other keys for upgraded capabilities as well.
* Next you'll need to upload your current resume (referred to as a baseline resume), which can be formatted as odt, docx, 
  pdf or html. The formatting of my resume worked best as docx, but this will be dependent on how yours is formatted.
  A baseline resume is the starting point, which is then modified to cater to a job posting. 
      You can create a baseline resume by selecting the **Resume** menu option in the left side navigation menu.
* While not required, I recommend editing your converted resume (which will be HTML formatted) to insure 
  that it is styled the way you want.
* You are now ready to start adding Job Postings, which is pretty straight forward, and your off to the races.

### Job Posting

The Job Posting page is the main page that you'll use.  Here you will add new jobs, which are automatically placed in 
the **Applied** column.  You can drag and drop each card to any column you want.  Each of the four columns keeps a 
count at the top for reference.  When using the search box, you'll notice that with each letter typed non-matching cards 
are removed.  Each card displays the time difference between when it was created and now, as well as the difference 
from the last contact.  Job scoring is based on two things, the **interest level** and **outcome score** from an 
interview, which is averaged for the score value.

Clicking on any card will take you to the Job Details page, where you have a summary of job information entered when you 
created the job. Once you customize a resume for the job, you'll additionally see some stats. **Keywords** are skills 
or technologies, found in your resume and the job posting description. Keywords can also become **Focused Keywords**, 
which are basically the same things, but given a bit more attention when rewriting your resume. The **Baseline Score** 
is the percentage of matching keywords from your baseline resume and the job posting.  Your **Rewrite Score** is the 
percentage after your resume has be rewritten.  You can also add and view calendar appointments, contacts, reminders 
and notes that are specifically related to the job posting.  You can also edit or delete the job posting.

### Resume

You can optimize your resume from the Job Details page by clicking on the **Optimize Resume** button.  You'll notice 
that to the left of that button is the baseline resume that will be used (your selected default), but should you wish 
to use a different one, simply click on it and select the one you want to use. The application will then pull the 
qualifications from the job posting and grab all the keywords from it.  This is then displayed on the next page and 
each keyword is highlighted in orange. Click on it once to make it a "focused keyword" and once more to remove it from 
being selected.  You can also select non-highlighted word(s) to add them as keywords.  Once your happy with the 
keywords, click **Finalize** to have it start rewriting your resume. Depending on the LLM model chosen, this can 
take a while to complete, so be patient.  Upon completion, you be shown your baseline resume on the left and rewritten 
resume on the right.  Changes are highlighted in green on the right, and items removed are shown highlighted in 
orange on the left side. Click on any highlighted text to remove it and revert it back to it's original state. Finally, 
click **Accept Changes** and your resume is complete and ready to be downloaded, which you can do from the Job Details page.

The Resume section, is for managing all your resumes, which are separated into two groups - **Baseline** and **Customized 
for Jobs**.  Regardless of the file format for a resume you upload, it is converted into a markdown version. This makes 
it easier for the LLM models to interact with, and you can directly edit the markdown for baseline resumes.  Resumes can 
be **cloned** (copied) or deleted and the **Tailor** option let's you re-pick the keywords you want applied and then rewrites 
the resume again (overwriting the previous version).

### Cover Letter

On the Job Details page, you can select the **Create Cover Letter** button to generate a cover letter specific for that 
job posting. You can select the length that you want it, as well as the overall tone of the letter.  You also have the 
option to type in any additional instruction that you want included, for example: "Make three bullet points with examples 
on why I'm the best fit for this job". You can generate or re-generate cover letters at will.

### And the Rest

The Contacts, Calendar and Notes pages are all pretty self-explanatory and intuitive to use.  On the Calendar pages, 
there will be a green dot on interviews that you have updated with an outcome score.  This is so you can easily view 
if you missed scoring an appointment. The Documents page is where your company reports are kept, but are also accessible 
from the Job Details page.  You can however request a company report for companies that do not have a job posting.

For the sections **Job Posting**, **Contacts**, **Calendar**, **Notes** and **Resume** you have the ability to export 
all the records from the DB, for whatever reason.  You can do this by clicking on the 3 dot collapsed menu icon on 
the top right of the page and then selecting the export option.  This will dump the DB contents to a CSV file that will 
automatically start downloading upon selection.


## 📋 Service Details

### 🖥️ Frontend Service (`job_track_now-frontend`)

**What it does:**
- Serves the React application through the reverse proxy Nginx (Linux only)
- Application is ran by using uvicorn with 4 workers on port 3000 (Windows and Mac)
- Provides the user interface for all job tracking features
- Handles client-side routing and state management
- Manages authentication and validation

**How it works:**
- For Linux users, the frontend uses the encrypted HTTPS layer and a fake domain using self-signed certificates. The website 
    is accessed directly through Nginx, which simply points to the compiled files that are served up. Communication 
    is done through HTTPS requests from the frontend application to the backend API, which can also connect to the DB and 
    provides all the data and logic
- For Windows and Mac users, the frontend runs using the React web server, which run on port 3000.  This port is mapped to
    port 80 on your host machine so that it can be accessed without specifying the port.  It communicates with the backend 
    API service directly from the running Python service using port 8000. Port 8000 is also mapped to the host machine using 
    the same port 8000.  This is necessary for the web application, which is used on the host machine and therefore doesn't 
    have access to Docker networking, to be able to route to the backend.

**Key Features:**
- Job management with drag-and-drop interface
- Contact management and relationship tracking
- Calendar scheduling for interviews and reminders
- Notes and document organization
- Real-time search and filtering
- Resume customization and management
- Cover letter generation
- Company research report generation and interview tips

### ⚙️ Backend Service (`job_track_now-backend`)

**What it does:**
- Provides RESTful API endpoints for all data operations
- Handles business logic and data validation
- Manages database connections and transactions
- Enforces authentication and validation for session management

**How it works:**
- For Linux users, the API service is ran using uvicorn on port 7080.  It also has Nginx setup as a reverse proxy that 
    handles the fake domain name routing, as well as the SSL certificate, enabling all communications to be done over 
    HTTPS.  
- For Windows and Mac users, the API service is accessed directly from the uvicorn process running on port 7080.
- Common use on both setups is that it's able to connect to the DB and manages all data interactions.  It also can connect 
  to the OpenAI API to make requests to their large language models to have directed work completed.  All requests return 
    the necessary data back to the frontend, where it is then organized and displayed for consumption.

### 🗄️ Database Service (`job_tracker_db`)

**What it does:**
- Stores all application data in PostgreSQL tables
- Provides ACID compliance for data integrity
- Handles complex queries and relationships
- Referenced across multi-stage OAuth2 workflow

**How it works:**
1. **Initialization**: Automatically runs schema.sql on first startup
2. **Data Storage**: Persistent storage using Docker volumes
3. **Connections**: Accepts connections from backend service and is mapped to localhost:5432


## 🔄 Operational Flow

See flow charts for a deeper dive 
- General communication flow: docs/flow/communication_flow.pdf
- Resume rewrite flow: docs/flow/resume_rewrite_flow.pdf
- New baseline resume flow: docs/flow/add_new_baseline_resume.pdf


## 📊 Monitoring and Health Checks

### Service Health

Each service includes health checks:

```bash
# Check all service status
docker-compose ps

# View service logs
docker-compose logs frontend
docker-compose logs backend
docker-compose logs db

# Monitor in real-time
docker-compose logs -f
```

### Health Endpoints

Linux
- **Backend Health**: https://api.jobtracknow.com/health
- **Frontend Health**: https://jobtracknow.com (returns React app)

Windows/Mac
- **Backend Health**: http://localhost:8000/health
- **Frontend Health**: http://localhost:3000 (returns React app)

## 🔧 Configuration Options

### Backend Environment Variables

Create a `.env` file to customize:

```bash
# Database Configuration
DATABASE_URL=postgresql://apiuser:change_me@psql.jobtracknow.com:5432/jobtracker
POSTGRES_HOST=psql.jobtracknow.com              # database host to connect to
POSTGRES_PASSWORD=change_me                     # database password part of credentials
POSTGRES_USER=apiuser                           # database user/role to connect as
POSTGRES_DB=jobtracker                          # name of the database to connect to
POSTGRES_PORT=5432                              # the port used to establish the connection
PGDATA=/var/lib/postgresql/data                 # path for the main postgresql data storage
POSTGRES_HOST_AUTH_METHOD=password              # password method to use for authentication

# Application Configuration
APP_NAME=Job Track Now                          # name of the application
APP_VERSION=1.0.0                               # application version
DEBUG=True                                      # not really used

# File Storage Configuration
BASE_JOB_FILE_PATH=/app/job_docs                # defines the full path where job docs are stored
RESUME_DIR=/app/job_docs/resumes                # defines the full path where resumes are written to
COVER_LETTER_DIR=/app/job_docs/cover_letters    # defines the full path where cover letters are saved to
EXPORT_DIR=/app/job_docs/export                 # defines the full path for DB export dump file
LOGO_DIR=/app/job_docs/logo                     # defines the full path where company logos are save (not used anymore)
REPORT_DIR=/app/job_docs/report                 # defines the full path where reports are saved to for download purposes

# Logging Configuration
LOG_LEVEL=DEBUG                                 # defines the level at which logs are recorded
LOG_FILE=api.log                                # filename for the log file

BACKEND_URL=http://localhost:8000               # URL used to call the backend API
JWT_SECRET_KEY=<random_key>                     # Random static string used with encryptions

# CORS Configuration
ALLOWED_ORIGINS=["http://localhost:3000", "http://portal.jobtracknow.com:3000"]   # applied for CORS enforcement

# AI Configuration
OPENAI_PROJECT=<open_ai_project_name>           # OpenAI project name
```

### Frontend Environment Variables

```bash
# API Configuration
REACT_APP_API_BASE_URL=http://api.jobtracknow.com                     # backend API URL for access

# Logging Configuration
REACT_APP_LOG_LEVEL=DEBUG                                             # level to use for logging
REACT_APP_LOG_FILE=portal.log                                         # name of the log file
REACT_APP_ENABLE_CONSOLE_LOGGING=true                                 # whether to enable console logging

REACT_APP_FRONTEND_BASE_URL=https://jobtracknow.com                   # URL for the frontend application
REACT_APP_OAUTH_REDIRECT_CALLBACK=https://jobtracknow.com/callback    # URL used during the OAuth2 flow

NODE_ENV=production
```


## 🗂️ Data Persistence

### Volumes
- **postgresql**: For Linux users, a directory is mounted and bound to the directory **/var/lib/postgresql-docker/16/data** 
    where the actual DB files are written to.  This acts identical to how PostgreSQL would run if installed locally.  
    Windows and Mac users, use a managed Docker volume that is allocated and managed by Docker.  Both of these actions 
    enable data saved to the DB to persist across starts and stopping of the DB and stack.
- **job_docs**: Mounts to a local directory for saving job-specific documents and files.  The files saved there remain 
    accessible after the Docker stack is stopped.
- **timezone**: Linux users additionally share configurations for timezone settings, which is bound into each container, 
    so that all the containers (servers) are in sync and use the same timezone.  Otherwise UTC is the default, and your 
    browser operates from localtime - meaning if you don't live exactly on Greenwich mean time, then there is an offset difference.

### Backup and Restore

```bash
# Backup
docker compose exec db pg_dump -U apiuser --inserts jobtracker > backup.sql

# Restore
docker compose exec -T db psql -U apiuser job_tracker < backup.sql
```

## 🛠️ Logging

### Docker

So you are able to view the Docker logs using your terminal as follows:

```bash
# View logs from all services
docker compose logs

# Continual log output
docker compose logs -f

# Show only frontend service logs
docker compose logs frontend

# Show only backend service logs
docker compose logs backend

# Show only database logs
docker compose logs db
```


### Back-end API

The Python application is using a logger package to ouput logs to a log file that is written locally within the container.  You 
can access this log file by opening a terminal session into the container, where you can then view a file however you like. After 
the shell connection is established, you should be placed in the document root for the service, which is **/app**. The log 
file is named **api.log**

```bash
docker exec -it api.jobtracknow.com bash

# You shouldn't need to do this, but in case wondered off and got lost
cd /app

# View last 30 lines
tail -n 30 api.log

# Continually view log file
tail -f api.log

# Open the log file in a text editor
vim api.log 
```

### Front-end Application

The React application also is setup to log things, but uses the browsers local storage to write the logs to.  You can access 
the logs using your browser's inspect tool.  For Firefox and Chrome, right-click anywhere on the page, then select "Inspect(Q)". 
This will open the developer tools and on the top tab navigation select the "Console" tab. At the bottom of the displayed output, 
you'll see either a ">" or ">>", which when you click there, it then allows you to execute any javascript.

```console
// View all logs
logger.getLogs()

// Download logs as a text file
logger.exportLogs()

// Clear all logs
logger.clearLogs()

// Log a test message
logger.debug('Test message')

    Or Use Direct localStorage Access (Works Immediately)

// View logs
JSON.parse(localStorage.getItem('portal.log'))

// Clear logs
localStorage.removeItem('portal.log')
```


## 🚨 Troubleshooting

### Common Issues
* **Page not found** or just won't load. So I didn't want to mess around with SSL certificates, which self-signed certs still 
    have to be manually added or accepted in order to work.  This application doesn't store anything secret, so there isn't any 
    need to do big security implementations.  This being the case, access to the services uses standard non-encrypted HTTP. You 
    may find that your browser has automatically added the "S", which won't work, as the application isn't configured to use 
    SSL.  So verify that your using **http://** and NOT **https://**

    If you still are having issues, verify that all 3 services are actively running.  More than likely you'll find that one failed to start. Still 
    can't pull up the page?  Try looking at the logs files, they can provide some insight.

* **Port Conflicts**  Insure that you don't have a locally installed version of PostgreSQL or another container 
    running with PostgreSQL installed on it.  Also make sure that you don't have a local install of Apache2 or Nginx 
    running, as by default they both bind to port 80 and likely other additional ports.
   ```bash
   # Check what's using ports (Linux and MacOS)
   lsof -i :80
   lsof -i :8000
   lsof -i :3000
   lsof -i :5432
  
   # On Windows you can execute the following from terminal or powershell
   netstat -aon | findstr :80
   netstat -aon | findstr :8000
   netstat -aon | findstr :3000
   netstat -aon | findstr :5432
   # The last column displayed should be the process ID number or PID. Execute the following to find the program:
   tasklist | findstr <PID>
   ```
* **Database apiuser can't authenticate** If you wisely decided to change the default password to use 
    your own given password, it was the smart thing to do, however the Docker PostgreSQL image didn't 
    seem to always set the apiuser password correctly.  This being the case, I added setting the password 
    directly to the DB schema SQL that gets executed on creation.  So it's quite likely that the apiuser 
    got set with the password of "change_me", but since you changed the password, it's trying to connect 
    with a different value. I apologize for not flushing that out more, but it's an easy fix. Execute 
    the following from a terminal, which will shell into the DB container, where you can change the password.
    ```bash
    # first we need to get some information.  Make sure your Docker stack is currently running, and from a terminal type this:
    docker ps
  
    # Note the first column displayed CONTAINER ID - You'll need this value for the container with the 
    # IMAGE set as "postgres:15.15-trixie".  Now with this value we can shell into the container with the following:
    docker exec -it <container_id> bash
    
    # Now your in the database container with a terminal, so we'll need to connect to the PostgreSQL service 
    psql -U apiuser jobtracker
    
    # That connects to PostgreSQL using their console.  No password is needed because the image sets local connections to "trust".
    # So now you'll simply change the password for that role to whatever you have configured in the backend/.env file
    jobtracker=# ALTER ROLE apiuser WITH PASSWORD '<your_password>';
    ```
* **Database - User authentication fails** If you find that the credentials are failing to authenticate, then you can reset 
    them pretty easily (if the DB container is able to run).  Start up the Docker stack and then we'll shell into the database 
    container and execute the SQL to set the credentials.
    ```bash
    # First let's shell into the PostgreSQL database container
    docker exec -it psql.jobtracknow.com bash
  
    # Next we'll login to the database using the PostgreSQL console
    # NOTE: no password is needed, as the DB is configured to trust local connections.
    psql -U apiuser jobtracker
  
    # Now we'll simply overwrite any existing password with a new value
    ALTER ROLE apiuser WITH PASSWORD '<password>';
  
    # If the apiuser account didn't get created, then you can try to login as root
    psql -U root jobtracker
    CREATE ROLE apiuser LOGIN INHERIT PASSWORD '<password>';
    ```

* **Database Connection Issues**
   ```bash
   # First check if the Database is running
   docker compose ps
   # Look at the STATUS column, and make sure it says "Up"
  
   # Check database logs for some type of error that is happening
   docker compose logs db
   # correct the error. with so many possible issues, it's impossible to list them all here, but google is your friend

   # Restart database
   docker compose restart db
   ```
* **Frontend Build Issues** You can force a complete rebuilding of every layer, which can overwrite some problems encountered
   ```bash
   # Rebuild frontend
   docker compose build --no-cache frontend
  
   # For Windows and MacOS users, do the following
   docker compose -f docker-compose-win.yml build --no-cache frontend
   ```

* **Backend Build Issues** You can force a complete rebuilding of every layer, which can overwrite some problems encountered
   ```bash
   # Rebuild backend
   docker compose build --no-cache backend
  
   # For Windows and MacOS users, do the following
   docker compose -f docker-compose-win.yml build --no-cache backend

* **Reset Everything**
    ```bash
    # Stop all services
    docker compose down
    
    # Remove volumes (WARNING: This deletes all data)
    docker compose down -v
    
    # Rebuild from scratch
    docker compose up --build
    ```

## 🎯 Tips

* Conversion between file formats is never 100% and whatever format you upload your first baseline resume in matters. I've 
found that if it's formatted as **docx**, that file conversion works best.  Regardless of the format it needs to be 
converted to markdown, which works better with the LLM models. 
* Your original resume file is also used as a template for formatting your modified resume (if downloading the docx format)
* After your original resume has been formatted to markdown, I recommend manually editing the markdown to insure it's 
formatted as you want.
* Play around with using different language models, as you'll notice a big difference in the time it takes to execute 
between the different versions.  You can change the LLM used for different operations in the **Personal** section

## 📚 API Documentation

Once running, visit:
- **Interactive API Docs**: http://api.jobtracknow.com or http://localhost:8000/docs

## 🧪 Testing

```bash
# Test backend endpoints
curl http://api.jobtracknow.com/health
curl http://localhost:8000/health

# Test frontend
curl http://portal.jobtracknow.com
curl http://localhost

# Test database connection
docker-compose exec db pg_isready -U apiuser
```

## 🔒 Security Considerations

- Database password should be changed from default
- Backend runs as non-root user
- Frontend / Backend uses security headers via Nginx
- CORS is configured for specific origins only
- OAuth2 authorization code flow used with JWT tokens


## 📄 License

This project is for educational/personal use.