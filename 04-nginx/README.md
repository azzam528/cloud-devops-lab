# 04 — Nginx Reverse Proxy

## Overview

This lab configures **Nginx as a reverse proxy** in front of the PersonalDiary Flask application.

Previously, the Flask application was accessed directly through port `5000`:

```text
http://192.168.3.129:5000
```

After configuring Nginx, the application can be accessed through the standard HTTP port:

```text
http://192.168.3.129
```

Nginx receives the incoming HTTP request on port `80` and forwards it internally to the Flask application running on port `5000`.

---

## Architecture

### Before Nginx

```text
Windows Host
     │
     │ HTTP :5000
     ▼
Debian Server
     │
     ▼
Flask :5000
     │
     ▼
MongoDB
```

### After Nginx

```text
Windows Host
     │
     │ HTTP :80
     ▼
┌─────────────────────┐
│ Debian Server       │
│                     │
│  Nginx :80          │
│      │              │
│      │ reverse      │
│      │ proxy        │
│      ▼              │
│  Flask :5000        │
│      │              │
│      ▼              │
│  MongoDB            │
└─────────────────────┘
```

The browser communicates with Nginx, while Nginx communicates with Flask internally.

---

## Why Use a Reverse Proxy?

A reverse proxy sits between the client and the application server.

Instead of exposing the Flask application directly:

```text
Client → Flask :5000
```

the request goes through Nginx:

```text
Client → Nginx :80 → Flask :5000
```

This provides a dedicated web server layer in front of the application.

Nginx can later be used for:

* HTTPS/TLS termination
* Domain-based routing
* Serving static files
* Request routing
* Load balancing
* Access logging
* Reverse proxying multiple applications

This makes Nginx an important component of the deployment architecture.

---

## Nginx Configuration Structure

On Debian, Nginx provides two important directories:

```text
/etc/nginx/
├── sites-available/
└── sites-enabled/
```

### `sites-available`

This directory contains configurations that are available to Nginx but are not necessarily enabled.

The PersonalDiary configuration is stored at:

```text
/etc/nginx/sites-available/personaldiary
```

### `sites-enabled`

This directory contains configurations that are currently enabled.

The enabled configuration is:

```text
/etc/nginx/sites-enabled/personaldiary
```

The file in `sites-enabled` is a symbolic link pointing to the configuration in `sites-available`.

---

## Symbolic Link

A symbolic link, or symlink, is a filesystem reference that points to another file or directory.

The PersonalDiary configuration is enabled using:

```bash
sudo ln -s /etc/nginx/sites-available/personaldiary \
/etc/nginx/sites-enabled/personaldiary
```

The resulting structure is:

```text
sites-enabled/
└── personaldiary
        │
        └──→ ../sites-available/personaldiary
```

This allows the configuration to be maintained in `sites-available` while enabling or disabling it through `sites-enabled`.

To verify the symlink:

```bash
ls -l /etc/nginx/sites-enabled/
```

Expected output contains something similar to:

```text
personaldiary -> /etc/nginx/sites-available/personaldiary
```

---

## Nginx Configuration

The configuration used for this lab:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:5000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

A copy of the configuration used in this lab is available in:

[`personaldiary.conf`](./personaldiary.conf)

---

## Configuration Explanation

### `server`

```nginx
server {
```

Defines a server block containing the configuration for a website or application.

---

### `listen`

```nginx
listen 80;
```

Tells Nginx to listen for HTTP requests on port `80`.

Therefore:

```text
http://192.168.3.129
```

is received by Nginx.

---

### `server_name`

```nginx
server_name _;
```

The underscore is used here as a simple catch-all configuration for the local lab environment.

The server does not currently use a public domain name.

For example, a future deployment using a domain could use:

```nginx
server_name diary.example.com;
```

---

### `location`

```nginx
location / {
```

Defines how Nginx handles requests matching the `/` path.

Because the location is `/`, it handles the application's general HTTP requests.

For example:

```text
/
 /diary
```

are forwarded to the Flask application.

---

### `proxy_pass`

```nginx
proxy_pass http://127.0.0.1:5000;
```

This is the core of the reverse proxy configuration.

It tells Nginx to forward incoming requests to the Flask application running locally on port `5000`.

The request flow becomes:

```text
Browser
   │
   │ HTTP :80
   ▼
Nginx
   │
   │ proxy_pass
   ▼
127.0.0.1:5000
   │
   ▼
Flask
```

`127.0.0.1` refers to the Debian server itself.

---

## Proxy Headers

The configuration also forwards important request information to Flask.

### Host

```nginx
proxy_set_header Host $host;
```

Preserves the original `Host` header.

### Client IP

```nginx
proxy_set_header X-Real-IP $remote_addr;
```

Passes the client's IP address to the backend.

### Forwarded IP Chain

```nginx
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

Preserves information about the client and proxy chain.

### Original Protocol

```nginx
proxy_set_header X-Forwarded-Proto $scheme;
```

Passes the original request protocol, such as `http` or `https`.

These headers become especially useful when an application is deployed behind one or more reverse proxies.

---

## Disable the Default Nginx Site

A default Nginx configuration is normally enabled after installation:

```text
/etc/nginx/sites-enabled/default
```

This configuration serves the default Nginx welcome page.

Before configuring the reverse proxy, accessing:

```text
http://192.168.3.129
```

displayed the default Nginx page.

The default site was removed from `sites-enabled`:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

This does not uninstall Nginx. It only disables the default site configuration.

The resulting configuration is:

```text
/etc/nginx/sites-enabled/
└── personaldiary
```

---

## Configuration Procedure

### 1. Create the site configuration

```bash
sudo nano /etc/nginx/sites-available/personaldiary
```

Add the reverse proxy configuration.

---

### 2. Disable the default site

```bash
sudo rm /etc/nginx/sites-enabled/default
```

---

### 3. Enable PersonalDiary

```bash
sudo ln -s /etc/nginx/sites-available/personaldiary \
/etc/nginx/sites-enabled/personaldiary
```

---

### 4. Test the configuration

Before reloading Nginx, test the configuration:

```bash
sudo nginx -t
```

Expected result:

```text
syntax is ok
test is successful
```

Testing the configuration before reloading prevents an invalid configuration from being applied.

---

### 5. Reload Nginx

After the configuration test succeeds:

```bash
sudo systemctl reload nginx
```

A reload applies the new configuration without unnecessarily stopping the Nginx service.

---

## Verification

The Flask application was previously accessible directly through:

```text
http://192.168.3.129:5000
```

After configuring Nginx, it can also be accessed through:

```text
http://192.168.3.129
```

The request flow is:

```text
Windows Browser
       │
       │ HTTP :80
       ▼
    Nginx
       │
       │ HTTP proxy
       ▼
 Flask :5000
       │
       ▼
 MongoDB
```

The PersonalDiary application was successfully served through Nginx.

---

## Useful Commands

Check Nginx status:

```bash
sudo systemctl status nginx
```

Test configuration:

```bash
sudo nginx -t
```

Reload configuration:

```bash
sudo systemctl reload nginx
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

View enabled sites:

```bash
ls -l /etc/nginx/sites-enabled/
```

View available sites:

```bash
ls -l /etc/nginx/sites-available/
```

View Nginx error logs:

```bash
sudo tail -f /var/log/nginx/error.log
```

View access logs:

```bash
sudo tail -f /var/log/nginx/access.log
```

---

## Troubleshooting

### Nginx shows the default welcome page

Check enabled sites:

```bash
ls -l /etc/nginx/sites-enabled/
```

If `default` is still enabled, disable it:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Then test and reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

### `502 Bad Gateway`

A `502 Bad Gateway` usually means Nginx cannot reach the upstream application.

Check the Flask service:

```bash
sudo systemctl status personaldiary
```

Check whether port `5000` is listening:

```bash
sudo ss -lntp | grep 5000
```

Test Flask directly from the Debian server:

```bash
curl http://127.0.0.1:5000
```

If Flask is not running, restart it:

```bash
sudo systemctl restart personaldiary
```

---

### Nginx configuration error

Always run:

```bash
sudo nginx -t
```

before reloading Nginx.

If the test fails, inspect the error message before applying the configuration.

---

## What I Learned

This lab introduced several important server and DevOps concepts:

* Nginx as a web server
* Reverse proxy architecture
* Server blocks
* HTTP port `80`
* Upstream application port `5000`
* `sites-available`
* `sites-enabled`
* Symbolic links
* Proxy headers
* Nginx configuration testing
* Graceful configuration reloads
* Nginx access and error logs
* Basic reverse proxy troubleshooting

The main concept is:

```text
Client
  │
  ▼
Nginx
  │
  ▼
Application Server
  │
  ▼
Database
```

This separates the public-facing web server from the application process.

---

## Current Deployment State

At this point, the local production-like environment consists of:

```text
Windows Host
     │
     │ HTTP
     ▼
Debian Server
     │
     ├── Nginx :80
     │      │
     │      ▼
     │   Flask :5000
     │      │
     │      ▼
     │   MongoDB
     │
     └── systemd
            │
            └── PersonalDiary Service
```

The application is now accessible through the standard HTTP port without exposing the Flask port to the user.

---

## Next Step

The next milestone is to replace Flask's development server with a production WSGI server such as **Gunicorn**.

The target architecture will become:

```text
Browser
   │
   ▼
Nginx :80
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

This will make the deployment architecture closer to a real production web application.

