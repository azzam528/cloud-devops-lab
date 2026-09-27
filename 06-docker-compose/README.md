# 06 - Docker Compose

Di tahap sebelumnya saya sudah mencoba menjalankan PersonalDiary menggunakan Gunicorn dan Nginx secara manual.

Pada tahap ini saya mulai belajar Docker Compose.

Sebelumnya saya sudah berhasil membuat aplikasi Flask berjalan di dalam Docker container. Namun saat itu pengelolaan container, konfigurasi aplikasi, dan Nginx masih dilakukan secara terpisah.

Saya ingin memahami bagaimana aplikasi yang terdiri dari beberapa bagian bisa dikelola dengan lebih rapi menggunakan Docker Compose.

## Kondisi Sebelum Menggunakan Docker Compose

Sebelum menggunakan Compose, saya sudah mempunyai aplikasi PersonalDiary yang berjalan menggunakan Flask.

Aplikasi tersebut kemudian dijalankan menggunakan Gunicorn sebagai application server dan Nginx sebagai reverse proxy.

Secara sederhana alurnya:

Browser → Nginx → Gunicorn → Flask → MongoDB

Masalahnya, ketika semuanya dijalankan secara manual, saya harus mengatur container dan konfigurasi satu per satu.

Dari sini saya mulai belajar bahwa ketika aplikasi sudah mempunyai lebih dari satu service, akan lebih nyaman jika konfigurasi seluruh service tersebut didefinisikan dalam satu file.

Itulah alasan saya mulai menggunakan Docker Compose.

## Mulai Menggunakan Docker Compose

Dengan Docker Compose saya membuat dua service:

- `app`
- `nginx`

Service `app` digunakan untuk menjalankan aplikasi Flask menggunakan Gunicorn.

Service `nginx` digunakan sebagai reverse proxy yang menerima request dari browser dan meneruskannya ke aplikasi.

Arsitekturnya menjadi:

Browser
↓
Host :80
↓
Nginx Container
↓
Docker Compose Network
↓
App Container :5000
↓
Gunicorn
↓
Flask
↓
MongoDB

Hal yang menarik adalah saya tidak perlu menggunakan IP container secara manual.

Nginx cukup menggunakan:

`http://app:5000`

karena `app` merupakan nama service yang didefinisikan di Docker Compose.

Docker Compose secara otomatis membuat network dan menyediakan DNS internal sehingga container Nginx dapat menemukan container aplikasi menggunakan nama service tersebut.

## Memahami ports dan expose

Salah satu hal yang saya pelajari di tahap ini adalah perbedaan `ports` dan `expose`.

Pada service Nginx saya menggunakan:

`80:80`

Artinya port 80 pada host diteruskan ke port 80 pada container Nginx.

Browser dari luar Docker dapat mengakses:

`http://192.168.3.129`

Sedangkan pada service aplikasi saya menggunakan `expose` untuk port 5000.

Port tersebut tidak perlu dibuka ke host karena aplikasi hanya perlu diakses oleh Nginx melalui network Docker.

Jadi alurnya:

Browser → Host:80 → Nginx → app:5000

Saya mulai memahami bahwa tidak semua port container harus dibuka ke luar.

## Docker Compose Network

Docker Compose otomatis membuat network untuk service yang ada di dalam Compose.

Pada project ini terdapat network:

`personaldiary_default`

Karena kedua container berada pada network yang sama, Nginx dapat berkomunikasi dengan aplikasi menggunakan nama service.

Saya melakukan pengujian dari dalam container Nginx menggunakan:

`docker compose exec nginx wget -qO- http://app:5000`

Perintah tersebut berhasil mengembalikan HTML dari aplikasi Flask.

Dari pengujian tersebut saya bisa memastikan bahwa:

- container Nginx dapat berjalan
- container aplikasi dapat berjalan
- kedua container berada pada network yang sama
- DNS internal Docker bekerja
- Nginx dapat mengakses aplikasi menggunakan nama service

Hal ini membuat saya lebih memahami bahwa Docker Compose bukan hanya digunakan untuk menjalankan beberapa container, tetapi juga membantu mengatur komunikasi antar-container.

## Menggunakan Volume

Aplikasi PersonalDiary mempunyai fitur upload file.

File yang diupload disimpan di dalam folder:

`/app/static`

Saya kemudian menggunakan Docker volume:

`static_data`

untuk menyimpan folder tersebut.

Tujuannya adalah supaya file yang diupload tidak bergantung pada lifecycle container.

Saya melakukan pengujian dengan cara:

1. Menjalankan aplikasi.
2. Mengupload file melalui PersonalDiary.
3. Memastikan file berhasil tersimpan.
4. Menjalankan `docker compose down`.
5. Menjalankan kembali `docker compose up -d`.
6. Mengecek file yang sebelumnya diupload.

File masih tersedia setelah container dibuat ulang.

Dari sini saya memahami perbedaan antara container dan volume.

Container bisa dibuat dan dihapus kembali, sedangkan data yang penting sebaiknya disimpan pada storage yang lifecycle-nya terpisah dari container.

Saya juga belajar untuk berhati-hati dengan:

`docker compose down -v`

karena opsi `-v` dapat menghapus volume yang digunakan oleh Compose.

## Masalah HTTP 413

Dalam proses pengujian saya menemukan masalah ketika mencoba melakukan upload file.

Request POST ke aplikasi mendapatkan:

`413 Request Entity Too Large`

Awalnya saya mengira masalah tersebut berasal dari Flask atau aplikasi.

Kemudian saya melihat log Nginx.

Dari sana saya menemukan bahwa request ditolak oleh Nginx sebelum diteruskan ke aplikasi.

Ini menjadi salah satu bagian yang cukup penting bagi saya karena saya mulai memahami bahwa ketika sebuah request gagal, tidak selalu berarti aplikasi yang bermasalah.

Request melewati beberapa layer:

Browser
↓
Nginx
↓
Gunicorn
↓
Flask
↓
Database

Kalau request ditolak di Nginx, Flask bahkan belum menerima request tersebut.

Solusinya adalah menambahkan konfigurasi:

`client_max_body_size 20M;`

Setelah Nginx dikonfigurasi ulang, proses upload kembali berhasil.

Dari masalah ini saya belajar pentingnya memahami alur request dan melakukan troubleshooting berdasarkan layer, bukan langsung mengubah kode aplikasi.

## Memahami Reverse Proxy

Pada tahap sebelumnya saya sudah menggunakan Nginx, tetapi pada tahap Docker Compose saya mulai memahami konsep reverse proxy dengan lebih jelas.

Browser tidak langsung mengakses Gunicorn.

Browser hanya berkomunikasi dengan Nginx.

Nginx kemudian meneruskan request ke:

`app:5000`

Keuntungan dari pendekatan ini adalah aplikasi tidak perlu membuka port aplikasinya langsung ke jaringan luar.

Nginx menjadi pintu masuk aplikasi.

Selain itu Nginx juga dapat menangani konfigurasi seperti:

- reverse proxy
- request header
- batas ukuran upload
- dan nantinya HTTPS

## Kenapa Menggunakan Docker Compose?

Setelah menggunakan Compose, saya mulai melihat perbedaannya dengan menjalankan container secara manual.

Sebelumnya saya perlu menjalankan dan mengatur container secara terpisah.

Dengan Compose, konfigurasi service dapat ditulis dalam satu file.

Saya cukup menjalankan:

`docker compose up -d`

untuk menjalankan seluruh stack.

Kemudian:

`docker compose ps`

untuk melihat status seluruh service.

Dan:

`docker compose down`

untuk menghentikan stack.

Ini membuat deployment menjadi lebih reproducible karena konfigurasi service tidak hanya tersimpan di command history, tetapi ditulis secara deklaratif dalam `docker-compose.yaml`.

## Hasil Akhir

Pada akhir tahap ini PersonalDiary berhasil dijalankan menggunakan Docker Compose dengan dua container utama:

- Nginx
- Flask + Gunicorn

Keduanya dapat berkomunikasi melalui Docker network.

Aplikasi dapat diakses melalui port 80 pada host.

File upload tetap tersimpan menggunakan Docker volume.

Nginx berhasil digunakan sebagai reverse proxy.

Saya juga berhasil menemukan dan memperbaiki masalah upload `413 Request Entity Too Large`.

Secara keseluruhan, pada tahap ini saya mulai memahami bahwa containerization bukan hanya tentang menjalankan aplikasi menggunakan Docker.

Saya mulai belajar bagaimana beberapa service saling terhubung, bagaimana network Docker bekerja, bagaimana data dibuat persistent menggunakan volume, dan bagaimana melakukan troubleshooting berdasarkan layer.

## Yang Saya Pelajari

Beberapa hal yang saya dapat dari tahap ini:

- Docker Compose untuk mengelola beberapa container.
- Service discovery menggunakan nama service.
- Docker Compose network.
- Perbedaan `ports` dan `expose`.
- Docker volume dan persistence.
- Nginx sebagai reverse proxy.
- Komunikasi antar-container.
- Troubleshooting berdasarkan layer.
- HTTP 413 dan batas ukuran request pada Nginx.
- Dasar deployment aplikasi multi-container.
- Pentingnya membuat konfigurasi deployment yang reproducible.

Tahap berikutnya saya ingin melanjutkan ke healthcheck, restart policy, logging, dan resource management sebelum masuk ke bagian CI/CD.
