# Cloud & DevOps Lab

A hands-on laboratory for learning **Cloud Computing, DevOps, Linux System Administration, Networking, Deployment, Automation, and Infrastructure Engineering** through real projects.

This repository documents my journey from deploying a simple application on a Linux server to building automated, containerized, monitored, and cloud-ready infrastructure.

The goal is not simply to collect tutorials or commands, but to **build, troubleshoot, understand, and document real systems**.

---

## 🧠 Learning Approach

This laboratory follows a project-based approach:

```text
Build → Break → Debug → Understand → Document
```

Instead of studying technologies independently, I introduce each technology when it becomes necessary for the project.

For example:

```text
Deploy Application
       ↓
Need Linux Server
       ↓
Need Process Management
       ↓
Learn systemd
       ↓
Need Reverse Proxy
       ↓
Learn Nginx
       ↓
Need Reproducible Environment
       ↓
Learn Docker
       ↓
Need Automated Deployment
       ↓
Learn CI/CD
```

This approach allows every concept to be learned in a practical context.

---

# 🖥️ Current Environment

## Local Infrastructure

| Component       | Technology             |
| --------------- | ---------------------- |
| Host OS         | Windows                |
| Virtualization  | VMware Workstation     |
| Server OS       | Debian GNU/Linux       |
| Server Type     | CLI-based Linux Server |
| Application     | Flask                  |
| Database        | MongoDB                |
| Web Server      | Nginx                  |
| Version Control | Git & GitHub           |

The initial laboratory environment runs locally inside a virtual machine to avoid cloud costs while still providing a realistic Linux server environment.

Cloud infrastructure will be introduced later as the projects become more advanced.

---

# 📚 Learning Progress

## 01 — Linux Server

Built a Debian Linux server as the foundation for the laboratory.

### Topics

* Debian Server
* Linux networking
* IP addressing
* SSH
* User management
* `sudo`
* Package management with `apt`
* Linux filesystem
* System services
* Basic server administration

📁 [`01-linux-server`](./01-linux-server/)

---

## 02 — Flask Application Deployment

Deployed an existing Flask application to the Debian server.

The application used for this laboratory is **PersonalDiary**.

Repository:

`https://github.com/azzam528/personaldiary`

### Deployment flow

```text
GitHub
   │
   │ git clone
   ▼
Debian Server
   │
   ├── Python
   ├── Virtual Environment
   └── Flask
          │
          ▼
      MongoDB
```

### Topics

* Git
* Repository cloning
* Python virtual environments
* Python dependency management
* `requirements.txt`
* Environment variables
* `.env`
* Flask deployment
* MongoDB connectivity
* Application testing over a network

📁 [`02-flask-deployment`](./02-flask-deployment/)

---

## 03 — systemd

Converted the Flask application from a manually executed process into a **Linux system service** using systemd.

Instead of:

```bash
python3 app.py
```

the application can now be managed using:

```bash
sudo systemctl start personaldiary
sudo systemctl stop personaldiary
sudo systemctl restart personaldiary
sudo systemctl status personaldiary
```

The service is also configured to start automatically when the server boots.

### Topics

* systemd
* Unit files
* `systemctl`
* `ExecStart`
* `WorkingDirectory`
* `EnvironmentFile`
* Service restart policies
* Automatic startup
* Service troubleshooting

📁 [`03-systemd`](./03-systemd/)

---

# 🏗️ Target Architecture

The infrastructure will gradually evolve from a simple local deployment into a more production-oriented architecture.

### Current

```text
                    Windows
                       │
                  HTTP / SSH
                       │
                       ▼
              ┌─────────────────┐
              │ Debian Server   │
              │                 │
              │     Flask       │
              │      :5000      │
              │        │        │
              └────────┼────────┘
                       │
                       ▼
                  MongoDB
```

### Planned

```text
                         GitHub
                            │
                         Git Push
                            │
                            ▼
                    GitHub Actions
                            │
                 ┌──────────┴──────────┐
                 │                     │
                Test                 Build
                 │                     │
                 └──────────┬──────────┘
                            │
                         Deploy
                            │
                            ▼
                    Debian Server
                            │
                       Docker Compose
                            │
              ┌─────────────┼─────────────┐
              │             │             │
            Nginx         Flask        Database
              │
              ▼
           Internet
```

---

# 🗺️ Roadmap

The roadmap is intentionally project-driven rather than organized as a fixed daily study schedule.

### Foundation

* [x] Debian Linux server
* [x] SSH
* [x] Linux user management
* [x] `sudo`
* [x] Basic networking
* [x] Flask deployment
* [x] Python virtual environment
* [x] Environment variables
* [x] MongoDB connection
* [x] systemd service

### Web & Deployment

* [ ] Nginx
* [ ] Reverse proxy
* [ ] Production application server
* [ ] HTTPS / TLS
* [ ] Domain configuration

### Containers

* [ ] Docker
* [ ] Dockerfile
* [ ] Docker networking
* [ ] Docker volumes
* [ ] Docker Compose
* [ ] Multi-container applications

### CI/CD

* [ ] GitHub Actions
* [ ] Automated testing
* [ ] Image building
* [ ] Automated deployment
* [ ] Deployment rollback

### Observability

* [ ] Application logging
* [ ] Linux system monitoring
* [ ] Metrics
* [ ] Grafana
* [ ] Prometheus
* [ ] Alerting

### Infrastructure as Code

* [ ] Terraform
* [ ] Infrastructure provisioning
* [ ] Configuration management
* [ ] Reproducible infrastructure

### Cloud

* [ ] Cloud networking
* [ ] Compute instances
* [ ] Cloud storage
* [ ] Managed databases
* [ ] Cloud deployment
* [ ] Cloud security

---

# 🔬 Projects

The laboratory uses real applications as the foundation for experimentation.

## PersonalDiary

A Flask-based personal diary application using MongoDB.

Repository:

`https://github.com/azzam528/personaldiary`

The application is used as the first deployment target for practicing:

* Linux deployment
* Process management
* Reverse proxy configuration
* Containerization
* CI/CD
* Monitoring
* Infrastructure automation

Rather than creating artificial demo applications for every technology, existing applications are progressively improved and operationalized.

---

# 🛠️ Technologies

Technologies planned or currently used in this laboratory include:

```text
Linux
Debian
SSH
Git
GitHub
Python
Flask
MongoDB
systemd
Nginx
Docker
Docker Compose
GitHub Actions
Prometheus
Grafana
Terraform
Cloud Infrastructure
```

The technology stack will evolve as the projects become more advanced.

---

# 📖 Documentation Structure

Each major learning milestone has its own documentation:

```text
cloud-devops-lab/
│
├── README.md
│
├── 01-linux-server/
│   └── README.md
│
├── 02-flask-deployment/
│   └── README.md
│
├── 03-systemd/
│   └── README.md
│
├── 04-nginx/
│   └── README.md
│
├── 05-docker/
│   └── README.md
│
├── 06-docker-compose/
│   └── README.md
│
├── 07-ci-cd/
│   └── README.md
│
├── 08-monitoring/
│   └── README.md
│
└── 09-terraform/
    └── README.md
```

Directories are added as each technology is actually implemented.

---

# 🎯 Goals

The long-term goal of this laboratory is to develop practical skills in:

* Linux system administration
* Networking
* Application deployment
* Server management
* Web infrastructure
* Containerization
* CI/CD
* Monitoring and observability
* Infrastructure as Code
* Cloud infrastructure
* Automation

More importantly, I want to be able to **design, deploy, operate, troubleshoot, and automate systems**, rather than only understand individual technologies in isolation.

---

# 📌 Current Status

The current application has successfully been:

```text
✓ Deployed to Debian Linux
✓ Connected to MongoDB
✓ Accessible over the network
✓ Running inside a Python virtual environment
✓ Managed by systemd
✓ Configured for automatic startup
```

### Next milestone

**Nginx Reverse Proxy**

Target:

```text
Browser
   │
   │ HTTP :80
   ▼
 Nginx
   │
   │ reverse proxy
   ▼
Flask :5000
```

---

## Philosophy

> **Don't just learn the technology. Build something with it. Break it. Fix it. Understand why it works. Then document it.**
