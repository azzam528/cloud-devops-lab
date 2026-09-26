# 03 — systemd Service

## Objective

Menjalankan aplikasi Flask sebagai **Linux system service** menggunakan `systemd`.

Sebelumnya aplikasi dijalankan secara manual:

```bash
python3 app.py
```

Dengan systemd, aplikasi dapat dikelola seperti service Linux:

```bash
sudo systemctl start personaldiary
sudo systemctl stop personaldiary
sudo systemctl restart personaldiary
sudo systemctl status personaldiary
```

---

## Environment

```text
OS       : Debian GNU/Linux 13
User     : azzam
App      : Flask
Project  : PersonalDiary
Python   : Virtual Environment
Port     : 5000
```

Project directory:

```text
/home/azzam/personaldiary
```

---

## Creating the Service

Service dibuat di:

```text
/etc/systemd/system/personaldiary.service
```

Isi:

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

## Reload systemd

Setelah membuat atau mengubah unit file:

```bash
sudo systemctl daemon-reload
```

---

## Enable Service

Agar service otomatis dijalankan ketika Debian boot:

```bash
sudo systemctl enable personaldiary
```

---

## Start Service

```bash
sudo systemctl start personaldiary
```

---

## Check Service

```bash
sudo systemctl status personaldiary
```

Jika berhasil:

```text
Active: active (running)
```

---

## Useful Commands

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

### Enable at boot

```bash
sudo systemctl enable personaldiary
```

### Disable at boot

```bash
sudo systemctl disable personaldiary
```

---

# Troubleshooting

## Error 203/EXEC

Pada percobaan pertama service gagal dengan:

```text
status=203/EXEC
```

Error ini menunjukkan systemd gagal menjalankan executable yang diberikan pada `ExecStart`.

Penyebabnya adalah path Python virtual environment pada service tidak sesuai dengan environment yang digunakan.

Service awal menggunakan:

```ini
ExecStart=/home/azzam/personaldiary/.venv/bin/python app.py
```

Sedangkan virtual environment yang digunakan berada pada:

```text
/home/azzam/personaldiary/venv/
```

Path kemudian diperbaiki menjadi:

```ini
ExecStart=/home/azzam/personaldiary/venv/bin/python app.py
```

Setelah itu:

```bash
sudo systemctl daemon-reload
sudo systemctl restart personaldiary
```

Service berhasil berjalan.

---

# What I Learned

Dari praktik ini saya memahami bahwa:

1. Linux dapat menjalankan aplikasi sebagai service menggunakan systemd.
2. `systemctl` digunakan untuk mengontrol service.
3. `WorkingDirectory` menentukan working directory aplikasi.
4. `ExecStart` harus menunjuk ke executable yang benar.
5. `EnvironmentFile` dapat digunakan untuk memberikan environment variables kepada aplikasi.
6. `Restart=always` dapat digunakan agar service mencoba berjalan kembali ketika aplikasi berhenti.
7. `systemctl status` sangat berguna untuk troubleshooting service.

---

# Verification

Aplikasi dapat diakses melalui:

```text
http://192.168.3.129:5000
```

Flask berhasil berjalan sebagai systemd service dan tetap dapat terhubung ke MongoDB.

---

## Next Step

Tahap berikutnya adalah menggunakan **Nginx sebagai reverse proxy** sehingga user tidak perlu mengakses Flask secara langsung melalui port `5000`.

Target:

```text
Browser
   │
   │ HTTP :80
   ▼
 Nginx
   │
   │ proxy
   ▼
Flask :5000
```

