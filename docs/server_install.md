#  Personal AWS Hosting Server 

### User Accounts

```bash
useradd -m -c "Davey Walbeck" -s /bin/bash davey
useradd -m -c "Deployment User" -s /bin/bash deploy

mkdir /home/davey/.ssh
mkdir /home/deploy/.ssh
```

#### Add public keys to "authorized_keys" files

### Install Packages

```bash
apt-get update

curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

apt-get install -y build-essential \
    libpq-dev \
    curl \
    git \
    pandoc \
    texlive-latex-base \
    texlive-latex-recommended \
    texlive-fonts-recommended \
    lmodern \
    vim \
    libpango-1.0-0 \
    libpangocairo-1.0-0 \
    libgdk-pixbuf-2.0-0 \
    libffi-dev \
    shared-mime-info \
    libcairo2 \
    libpangoft2-1.0-0 \
    lsof \
    nginx \
    nodejs \
    postgresql-16 \
    postgresql-16-pgvector \
    uvicorn \
    python3-full \
    python3-pip \
    python3-multipart \
    python3-asyncpg \
    python3-fastapi \
    python3-openai \
    python3-async-timeout  \
    python3-dotenv \
    python3-sniffio \
    python3-itsdangerous \
    python3-httpcore \
    python3-tqdm \
    python3-anyio \
    python3-simplejson  \
    python3-aiofiles \
    python3-pydantic \
    python3-venv
```

### Setup Directories and Maintenance

```bash
mkdir -p /var/log/archive
mkdir -p /var/log/nginx/archive

# set timezone to be matched with local
timedatectl set-timezone MST7MDT

vi /etc/vim/vimrc
    set tabstop=4

cd /etc/logrotate.d
```

#### add the line `olddir /var/log/archive` to the following files:
- apport
- bootlog
- nginx
- rsyslog
- wtmp
    
```bash
cd /var/www
mkdir jobtracknow job_api personal_api personal_site
mkdir job_api/cover_letters job_api/export job_api/logo job_api/report job_api/resumes personal_site/backup personal_site/dist
chown -R deploy:www-data jobtracknow job_api personal_api personal_site
```

#### add .env files
    /var/www/job_api/.env
    /var/www/personal_api/.env
    
```bash
visudo

    davey   ALL=(ALL) NOPASSWD:ALL
    deploy  ALL=NOPASSWD: /usr/bin/systemctl restart personal_api.service, /usr/sbin/service nginx restart, /usr/bin/systemctl restart jobtracknow-api.service, /usr/bin/systemctl status jobtracknow-api.service
```

### Create Systemd Services
```bash
cd /etc/systemd/system
vi personal_api.service
```

```systemd
[Unit]
Description=Personal Website API Service
After=network.target postgresql.service
Wants=postgresql.service

[Service]
Type=notify
User=deploy
Group=www-data
WorkingDirectory=/var/www/personal_api
Environment="PATH=/var/www/personal_api/venv/bin"
EnvironmentFile=/var/www/personal_api/.env
ExecStart=/var/www/personal_api/venv/bin/gunicorn main:app \
    --workers 4 \
    --worker-class uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000 \
    --timeout 120 \
    --graceful-timeout 30 \
    --access-logfile /var/log/nginx/ps-api_access.log \
    --error-logfile /var/log/nginx/ps-api_error.log \
    --log-level info
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=30
PrivateTmp=true
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

vi jobtracknow-api.service

```systemd
[Unit]
Description=Job Track Now API Service
After=network.target postgresql.service
Wants=postgresql.service

[Service]
Type=simple
User=deploy
Group=www-data
WorkingDirectory=/var/www/job_api
Environment="PATH=/var/www/job_api/venv/bin"
Environment="PYTHONPATH=/var/www/job_api"

# Kill any existing process on port 7080 before starting
ExecStartPre=/usr/bin/sh -c 'timeout 5 lsof -t -i:7080 | xargs -r kill -9 2>/dev/null || true'
ExecStartPre=/bin/sleep 2

# Start uvicorn
ExecStart=/var/www/job_api/venv/bin/python3 -m uvicorn app.main:app \
    --host 0.0.0.0 \
    --port 7080 \
    --workers 4 \
    --timeout-graceful-shutdown 200 \
    --timeout-keep-alive 200

# Restart policy
StartLimitIntervalSec=300
StartLimitBurst=5
Restart=always
RestartSec=10
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=30
TimeoutStartSec=30

# Logging
StandardOutput=append:/var/log/jobtracknow-api.log
StandardError=append:/var/log/jobtracknow-api-error.log

# Security hardening (optional)
NoNewPrivileges=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

And now activate both services

```bash
systemctl daemon-reload
systemctl enable jobtracknow-api
systemctl enable personal_api.service
```

### Python Setup

First we need to install all the packages that the code is dependent upon and setup the virtual environment

```bash
su - deploy
cd /var/www/job_api
python3 -m venv venv/

cd /var/www/personal_api
python3 -m venv venv/

vi /var/www/job_api/requirements.txt
```

Insure the following content is up to date with the actual requirements.txt file

```text
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
asyncpg==0.29.0
pydantic==2.7.4
pydantic-settings==2.12.0
alembic==1.13.0
psycopg2-binary==2.9.9
python-multipart==0.0.6
python-jose[cryptography]==3.3.0
odt2md==0.1.0
docx2md==1.0.4
mammoth==1.6.0
markitdown[pdf]==0.1.3
openai==1.54.5
convertapi==2.0.0
httpx==0.27.0
markdown==3.7
python-docx==1.2.0
beautifulsoup4==4.12.3
docx-parser-converter==0.5.1.2
html_to_markdown==2.10.1
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-mock==3.12.0
```

vi /var/www/personal_api/requirements.txt

```text
asyncpg>=0.30.0
fastapi>=0.115.12
gunicorn>=23.0.0
loguru>=0.7.3
openai>=1.75.0
pgvector>=0.4.0
python-dotenv>=1.1.0
python-multipart>=0.0.6
uvicorn>=0.34.2
```

Now install the packages

```bash
su - deploy
cd /var/www/job_api
source venv/bin/activate
pip install -r requirements.txt
exit

su - deploy
cd /var/www/personal_api
source venv/bin/activate
pip install -r requirements.txt
exit
```

### Nginx Configuration

`vi /etc/nginx/sites-available/api.jobtracknow.net`

```nginx configuration
server {
    listen 80;
    server_name api.jobtracknow.net;

    location / {
        proxy_pass http://127.0.0.1:7080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    access_log /var/log/nginx/job-api_access.log;
    error_log /var/log/nginx/job-api_error.log;
}
```

`vi /etc/nginx/sites-available/daveywalbeck.com`
```nginx configuration
server {
    listen 80 default_server;
    server_name daveywalbeck.com www.daveywalbeck.com;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1000;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    root /var/www/personal_site/dist/;
    index index.html;

    location = /health {
#       access_log off;
        add_header Content-Type text/plain;
        return 200 'healthy';
    }

    location / {
        try_files $uri /index.html;
    }

    # Cache static assets (JS/CSS with hash-based filenames)
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Handle preflight requests
    location ~* \.(eot|ttf|woff|woff2)$ {
        add_header Access-Control-Allow-Origin *;
    }

    access_log /var/log/nginx/personal-access.log;
    error_log /var/log/nginx/personal-error.log;
}
```

`vi /etc/nginx/sites-available/jobtracknow.net`
```nginx configuration
server {
    listen 80;
    server_name jobtracknow.net;
    server_name www.jobtracknow.net;
    server_name portal.jobtracknow.net;

    root /var/www/jobtracknow/build;
    index index.html index.htm;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1000;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    # Handle client routing, return all requests to index.html
    location / {
        try_files $uri $uri/ /index.html =404;

        # Prevent caching of HTML files during development
        add_header Cache-Control "no-cache, no-store, must-revalidate";
        add_header Pragma "no-cache";
        add_header Expires "0";
    }

    # Cache static assets (JS/CSS with hash-based filenames)
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1d;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Handle preflight requests
    location ~* \.(eot|ttf|woff|woff2)$ {
        add_header Access-Control-Allow-Origin *;
    }

    access_log /var/log/nginx/jobtrack_access.log;
    error_log /var/log/nginx/jobtrack_error.log;
}
```

`vi /etc/nginx/sites-available/ps-api.daveywalbeck.com`
```nginx configuration
server {
    listen 80;
    server_name ps-api.daveywalbeck.com;

    # Admin endpoints (add-entry, add-file) - restricted by IP
    location ~ ^/(add-entry|add-file)/ {
        # IP restrictions
        allow 127.0.0.1;
        allow 54.237.75.43;
        allow 172.31.0.186;
        allow 136.60.255.152;
        deny all;

        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    access_log /var/log/nginx/ps-api_access.log;
    error_log /var/log/nginx/ps-api_error.log;
}
```

Now activate the virtual website configurations

```bash
cd /etc/nginx/sites-enabled
rm default
ln -sn /etc/nginx/sites-available/api.jobtracknow.net
ln -sn /etc/nginx/sites-available/daveywalbeck.com
ln -sn /etc/nginx/sites-available/jobtracknow.net
ln -sn /etc/nginx/sites-available/ps-api.daveywalbeck.com

chown deploy:www-data /var/log/nginx/ps-api*

nginx -t
service nginx restart
```

### Database Setup
```bash
su - postgres
psql
```

```postgresql
CREATE ROLE apiuser WITH LOGIN PASSWORD '@p!u53Rt0k3n';
CREATE ROLE davey WITH LOGIN SUPERUSER CREATEDB CREATEROLE PASSWORD 'kier*33';
CREATE GROUP application;
CREATE GROUP dba;
GRANT application TO apiuser;
GRANT dba TO davey;

CREATE DATABASE jobtracknow OWNER=apiuser;
CREATE DATABASE personal_ai OWNER=apiuser;

\q
```
#### Import database schemas and data

```bash
gunzip -c personal_ai.sql.gz | psql personal_ai
gunzip -c jobtracker-20260118.sql.gz | psql jobtracknow
```

#### Grant permissions
psql 
```postgresql
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO apiuser;
GRANT USAGE, SELECT, UPDATE ON ALL SEQUENCES IN SCHEMA public TO apiuser;
```

### Github Configuration
- Create **Environment** "production"
- Add the following secrets to environment
    
job_track_now-api
- SSH_PRIVATE_KEY

job_track_now-portal
- SSH_PRIVATE_KEY

personal_api
- SSH_PRIVATE_KEY
- DATABASE_URL
- OPENAI_API_KEY
- OPENAI_ORG_ID
- OPENAI_PROJECT_ID
- SMTP_PASSWORD
- SMTP_USER

personal_site
- SSH_PRIVATE_KEY
- ENV_FILE
- VITE_CONTACT_EMAIL
- VITE_FILES_RESUME
- VITE_GITHUB_URL
- VITE_LINKEDIN_URL
- VIE_API_URL





















