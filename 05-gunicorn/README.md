# 05 — Gunicorn Application Server

## Overview

This lab replaces Flask's built-in development server with Gunicorn, a production-oriented WSGI application server.

The application is a Flask-based PersonalDiary application connected to MongoDB.

This milestone builds on the previous deployment stages:

* Linux server administration
* Flask application deployment
* systemd service management
* Nginx reverse proxy
* Gunicorn application server

---

## Architecture

### Before

```text
Client
  │
  ▼
Nginx :80
  │
  ▼
Flask Development Server :5000
  │
  ▼
MongoDB
```

The Flask development server is useful during development, but it is not intended to be the application server for a production deployment.

### After

```text
Client
  │
  ▼
Nginx :80
  │
  ▼
Gunicorn :5000
  │
  ▼
Flask Application
  │
  ▼
MongoDB
```

systemd manages the Gunicorn process:

```text
systemd
   │
   ▼
Gunicorn
   │
   ▼
Flask
```

---

## Why Gunicorn?

Flask includes a development server that is convenient for local development:

```bash
python app.py
```

However, a production-style deployment should use a dedicated WSGI application server.

Gunicorn provides:

* WSGI application serving
* multiple worker processes
* process management
* better separation between the application and the web server
* integration with systemd and reverse proxies such as Nginx

In this lab, Nginx handles HTTP traffic while Gunicorn handles the Flask application.

---

## Gunicorn Installation

Gunicorn was installed inside the application's Python virtual environment:

```bash
cd ~/personaldiary
source venv/bin/activate
pip install gunicorn
```

Verify:

```bash
gunicorn --version
```

Example:

```text
gunicorn (version 26.2.0)
```

---

## Manual Test

Before modifying the systemd service, Gunicorn was tested manually:

```bash
gunicorn --bind 127.0.0.1:5000 app:app
```

The command uses:

```text
app:app
```

The first `app` refers to:

```text
app.py
```

The second `app` refers to the Flask application object:

```python
app = Flask(__name__)
```

Therefore:

```text
app:app
│   │
│   └── Flask application object
└────── Python module
```

---

## systemd Integration

After the manual test succeeded, the existing systemd service was updated to use Gunicorn.

Service file:

```text
/etc/systemd/system/personaldiary.service
```

The important configuration is:

```ini
[Service]
User=azzam
WorkingDirectory=/home/azzam/personaldiary
EnvironmentFile=/home/azzam/personaldiary/.env
ExecStart=/home/azzam/personaldiary/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app
Restart=always
```

### Configuration Explanation

#### User

```ini
User=azzam
```

Gunicorn runs as the dedicated application user instead of root.

#### WorkingDirectory

```ini
WorkingDirectory=/home/azzam/personaldiary
```

Sets the application's working directory.

#### EnvironmentFile

```ini
EnvironmentFile=/home/azzam/personaldiary/.env
```

Loads application environment variables such as the MongoDB connection string.

The `.env` file is not committed to Git.

#### ExecStart

```ini
ExecStart=/home/azzam/personaldiary/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app
```

Starts Gunicorn using the Python virtual environment.

The application listens only on:

```text
127.0.0.1:5000
```

Nginx accesses Gunicorn through this local address.

#### Workers

```text
--workers 2
```

Runs two Gunicorn worker processes.

#### Restart

```ini
Restart=always
```

systemd automatically restarts Gunicorn if the process stops unexpectedly.

---

## Applying the Configuration

After modifying the service:

```bash
sudo systemctl daemon-reload
sudo systemctl restart personaldiary
```

Check the service:

```bash
sudo systemctl status personaldiary
```

Expected state:

```text
Active: active (running)
```

---

## Verification

### Check Gunicorn directly

```bash
curl http://127.0.0.1:5000
```

### Check through Nginx

```bash
curl http://192.168.3.129
```

The application should also be accessible from the Windows host:

```text
http://192.168.3.129
```

The request flow is:

```text
Windows Browser
       │
       ▼
Debian VM
       │
       ▼
Nginx :80
       │
       ▼
127.0.0.1:5000
       │
       ▼
Gunicorn
       │
       ▼
Flask
       │
       ▼
MongoDB
```

---

## Useful Commands

Check service:

```bash
sudo systemctl status personaldiary
```

Restart:

```bash
sudo systemctl restart personaldiary
```

Stop:

```bash
sudo systemctl stop personaldiary
```

Start:

```bash
sudo systemctl start personaldiary
```

View logs:

```bash
sudo journalctl -u personaldiary
```

Follow logs:

```bash
sudo journalctl -u personaldiary -f
```

Check listening ports:

```bash
sudo ss -ltnp
```

Check Gunicorn processes:

```bash
ps aux | grep gunicorn
```

---

## Troubleshooting

### Address already in use

If Gunicorn reports:

```text
Address already in use
```

another process is already using port 5000.

Check:

```bash
sudo ss -ltnp | grep :5000
```

If the old systemd service is still running:

```bash
sudo systemctl stop personaldiary
```

Then test Gunicorn manually again.

---

### Service fails to start

Check:

```bash
sudo systemctl status personaldiary
```

Then inspect logs:

```bash
sudo journalctl -u personaldiary -n 50 --no-pager
```

---

### Nginx returns 502 Bad Gateway

Check whether Gunicorn is running:

```bash
sudo systemctl status personaldiary
```

Then:

```bash
curl http://127.0.0.1:5000
```

If this fails, investigate Gunicorn/systemd first.

If it works, investigate Nginx.

---

## What I Learned

This milestone demonstrated how an application server fits into a web deployment architecture.

Key concepts:

* WSGI
* Gunicorn
* worker processes
* systemd process management
* application server vs web server
* localhost binding
* reverse proxy architecture
* service restart policies
* application environment variables

The important architectural distinction is:

```text
Nginx ≠ Gunicorn ≠ Flask
```

Each component has a different responsibility:

```text
Nginx
  → receives HTTP requests and acts as reverse proxy

Gunicorn
  → runs the Flask application through WSGI

Flask
  → contains the application logic

MongoDB
  → stores application data

systemd
  → manages the Gunicorn process
```

---

## Current Deployment State

The PersonalDiary application is now running with:

* Debian GNU/Linux server
* systemd
* Gunicorn
* Flask
* Nginx
* MongoDB
* Python virtual environment

Current architecture:

```text
                   Debian VM
┌────────────────────────────────────────────┐
│                                            │
│  Nginx :80                                 │
│      │                                     │
│      ▼                                     │
│  Gunicorn :5000                            │
│      │                                     │
│      ▼                                     │
│  Flask Application                         │
│      │                                     │
│      ▼                                     │
│  MongoDB                                   │
│                                            │
│  systemd → manages Gunicorn                │
│                                            │
└────────────────────────────────────────────┘
```

---

## Next Step

The next milestone is containerization with Docker.

Planned architecture:

```text
Docker
   │
   ├── Flask Application
   ├── Gunicorn
   └── Nginx
```

After that, Docker Compose will be introduced to manage multiple services.

The learning progression is:

```text
Linux
  ↓
systemd
  ↓
Nginx
  ↓
Gunicorn
  ↓
Docker
  ↓
Docker Compose
  ↓
CI/CD
  ↓
Cloud/VPS
  ↓
Monitoring & Security
  ↓
Infrastructure as Code
  ↓
Kubernetes
```

