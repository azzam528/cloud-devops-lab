# 03 — Systemd Service

## Objective

Run the PersonalDiary Flask application as a Linux system service using `systemd`.

Instead of manually starting the Flask application with:

```bash
python app.py
```

the application is managed by `systemd`, allowing it to:

* Start automatically when the server boots
* Restart automatically if the application stops
* Run as a dedicated non-root user
* Load environment variables from `.env`
* Be managed using standard Linux service commands

---

## Architecture

```text
Debian Server
│
├── systemd
│     │
│     ▼
│  PersonalDiary Service
│     │
│     ▼
│  Flask Application :5000
│     │
│     ▼
│  MongoDB
│
└── SSH :22
```

---

## Service Configuration

The service file is located at:

```text
/etc/systemd/system/personaldiary.service
```

Repository copy:

[`personaldiary.service`](./personaldiary.service)

Configuration:

```ini
[Unit]
Description=PersonalDiary Flask Application
After=network.target

[Service]
User=azzam
WorkingDirectory=/home/azzam/personaldiary
EnvironmentFile=/home/azzam/personaldiary/.env
ExecStart=/home/azzam/personaldiary/venv/bin/python app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## Configuration Explanation

### `[Unit]`

```ini
Description=PersonalDiary Flask Application
After=network.target
```

Defines the service description and tells `systemd` to start the application after the network is available.

### `User`

```ini
User=azzam
```

The application runs as the `azzam` user instead of `root`.

### `WorkingDirectory`

```ini
WorkingDirectory=/home/azzam/personaldiary
```

Sets the application directory before starting Flask.

### `EnvironmentFile`

```ini
EnvironmentFile=/home/azzam/personaldiary/.env
```

Loads environment variables such as the MongoDB connection string.

The `.env` file is intentionally excluded from Git.

### `ExecStart`

```ini
ExecStart=/home/azzam/personaldiary/venv/bin/python app.py
```

Starts the Flask application using the Python interpreter inside the project's virtual environment.

### `Restart`

```ini
Restart=always
```

Automatically restarts the application if the process exits.

---

## Creating the Service

Copy the service file to:

```bash
sudo cp personaldiary.service /etc/systemd/system/
```

Then reload the `systemd` configuration:

```bash
sudo systemctl daemon-reload
```

---

## Enable the Service

Enable automatic startup during boot:

```bash
sudo systemctl enable personaldiary
```

Start the service:

```bash
sudo systemctl start personaldiary
```

Check its status:

```bash
sudo systemctl status personaldiary
```

Expected state:

```text
Active: active (running)
```

---

## Service Management

### Start

```bash
sudo systemctl start personaldiary
```

### Stop

```bash
sudo systemctl stop personaldiary
```

### Restart

```bash
sudo systemctl restart personaldiary
```

### Status

```bash
sudo systemctl status personaldiary
```

### Enable at Boot

```bash
sudo systemctl enable personaldiary
```

### Disable at Boot

```bash
sudo systemctl disable personaldiary
```

---

## Viewing Logs

View recent service logs:

```bash
sudo journalctl -u personaldiary
```

Follow logs in real time:

```bash
sudo journalctl -u personaldiary -f
```

Show logs from the current boot:

```bash
sudo journalctl -u personaldiary -b
```

---

## Verification

The Flask application was verified from the Windows host using:

```text
http://192.168.3.129:5000
```

The application successfully:

* Served the Flask web interface
* Connected to MongoDB
* Read diary data
* Stored new diary data

The service also starts automatically through `systemd`.

---

## Troubleshooting

### Error: `203/EXEC`

One issue encountered during deployment was:

```text
status=203/EXEC
```

The cause was an incorrect Python virtual-environment path.

The actual virtual environment was:

```text
/home/azzam/personaldiary/venv/
```

but the service initially referenced:

```text
/home/azzam/personaldiary/.venv/
```

The correct configuration is:

```ini
ExecStart=/home/azzam/personaldiary/venv/bin/python app.py
```

After correcting the path:

```bash
sudo systemctl daemon-reload
sudo systemctl restart personaldiary
```

the service started successfully.

---

## Security Notes

Sensitive configuration is stored in:

```text
/home/azzam/personaldiary/.env
```

The `.env` file is excluded from Git.

The repository also excludes:

```text
.env
*.pem
*.key
__pycache__/
*.pyc
```

Never commit database credentials, API keys, private keys, or other secrets.

---

## Result

The PersonalDiary Flask application is now managed as a Linux service:

```text
systemd
   │
   ▼
personaldiary.service
   │
   ▼
Flask :5000
   │
   ▼
MongoDB
```

This removes the need to manually run the Flask application after every server reboot.

## Next Step

The next milestone is to configure **Nginx as a reverse proxy** so users can access the application through:

```text
http://192.168.3.129
```

instead of:

```text
http://192.168.3.129:5000
```

