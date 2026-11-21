# IoT REST API - Riviana Serly Septianty

**NPM:** 2410010248
**Mata Kuliah:** Pemrograman Internet of Things (TIF1504)
**Tugas:** Praktek 9. REST API Node-RED

## Deskripsi Proyek

Proyek ini adalah sistem monitoring dan kontrol IoT sederhana. Data suhu dan kelembaban dari sensor (disimulasikan dengan Python) dikirim lewat MQTT ke Node-RED. Node-RED berperan sebagai backend REST API, lalu data ditampilkan secara real-time di dashboard web (HTML, CSS, JavaScript). Dashboard juga dapat mengirim perintah kontrol lampu kembali ke perangkat lewat MQTT.

## Teknologi yang Digunakan

- Node-RED (backend REST API dan WebSocket)
- MQTT (Mosquitto, broker lokal)
- Python dan paho-mqtt (simulasi ESP32)
- HTML, CSS, dan JavaScript (dashboard)
- Chart.js (grafik real-time)
- SQLite (penyimpanan history)

## Struktur Repository

```
README.md             penjelasan proyek
dashboard.html        dashboard web
simulasi_esp32.py     simulator ESP32 (publisher MQTT)
flows.json            hasil export flow Node-RED
sensor.db             database SQLite berisi history sensor
screenshot/           bukti pengujian
```

## Alur Data

```
simulasi_esp32.py --MQTT--> Node-RED --REST API / WebSocket--> dashboard.html
                                ^                                     |
                                +----------- MQTT (iot/kontrol) <-----+
```

- Topic sensor: `iot/sensor`
- Topic kontrol: `iot/kontrol`

## Cara Menjalankan

1. Jalankan broker Mosquitto (`net start mosquitto` atau `mosquitto -v`).
2. Jalankan Node-RED dengan perintah `node-red`, lalu import `flows.json` lewat menu Import.
3. Set folder static di `settings.js`, contohnya `httpStatic: 'C:/nodered_statik'`, lalu letakkan `dashboard.html` di folder itu.
4. Untuk SQLite, tambahkan di `functionGlobalContext` pada `settings.js`:
   ```javascript
   functionGlobalContext: {
       sqlite: require('node:sqlite'),
   },
   ```
5. Install dependency Python dan jalankan simulator:
   ```
   pip install paho-mqtt
   python simulasi_esp32.py
   ```
6. Buka dashboard di `http://localhost:1880/dashboard.html`.

## Endpoint REST API

| Endpoint | Method | Fungsi |
|---|---|---|
| `/api/suhu` | GET | Data sensor terakhir |
| `/api/history` | GET | 50 data sensor terakhir (dari SQLite) |
| `/api/kontrol` | POST | Kirim perintah ke perangkat (butuh API key) |
| `/api/threshold` | PUT | Set batas suhu (butuh API key) |

---

# Bagian 1: Praktikum Utama

## Verifikasi End-to-End

| No | Komponen | Cara Verifikasi |
|---|---|---|
| 1 | Simulator ke MQTT | Terminal simulator menampilkan `Publish: {...}` |
| 2 | MQTT ke Node-RED | Debug panel Node-RED menampilkan data sensor |
| 3 | Node-RED ke REST | `/api/suhu` mengembalikan JSON |
| 4 | REST ke Web | Dashboard menampilkan angka yang berubah |
| 5 | Web ke REST ke MQTT | Klik tombol lampu, perintah diterima MQTT subscriber |

### 1. Simulator ke MQTT
<img width="1600" height="900" alt="01-simulator" src="https://github.com/user-attachments/assets/fed54ef6-e8c0-4352-949e-d19fc5772f37" />

### 2. MQTT ke Node-RED
<img width="1920" height="1080" alt="02-debug-nodered" src="https://github.com/user-attachments/assets/d796011b-fcfd-4230-8f12-4a0a05c70173" />

### 3. Node-RED ke REST
<img width="1600" height="900" alt="03-api-suhu" src="https://github.com/user-attachments/assets/2bcb6073-3106-4ad2-a88f-066208b0a0de" />

### 4. REST ke Web
<img width="1600" height="900" alt="WhatsApp Image 2026-11-19 at 21 53 57" src="https://github.com/user-attachments/assets/6492bd10-bdf5-4036-9d29-badfada82826" />

### 5. Web ke REST ke MQTT
<img width="1600" height="900" alt="05-mqtt-subscriber" src="https://github.com/user-attachments/assets/7e02f56a-7ac5-48d4-893d-d54e43ded839" />

---

# Bagian 2: Tugas Pengembangan (Nilai Tambah)

## 1. Endpoint `/api/threshold` (PUT) dan Notifikasi Suhu

Endpoint PUT untuk mengatur batas suhu maksimum. Setiap data sensor yang melewati batas akan memicu notifikasi di panel Debug Node-RED.

Contoh pengujian:
```
curl.exe -X PUT http://127.0.0.1:1880/api/threshold -H "Content-Type: application/json" -H "X-API-Key: rahasia123" -d "{\"max_suhu\":27}"
```

<img width="1600" height="900" alt="WhatsApp Image 2026-11-20 at 10 49 50" src="https://github.com/user-attachments/assets/9f82b36f-acc8-4a14-87e0-a20210f0372f" />

<img width="1600" height="900" alt="WhatsApp Image 2026-11-20 at 11 03 02" src="https://github.com/user-attachments/assets/fb51832b-d534-4876-b27a-f25d051e9987" />

<img width="1600" height="900" alt="WhatsApp Image 2026-11-20 at 11 03 06" src="https://github.com/user-attachments/assets/bcd5cafa-8ce2-424c-b16e-f2807552dfdd" />

## 2. Grafik Real-time dengan Chart.js

Dashboard menampilkan grafik garis suhu dan kelembaban untuk 20 data terakhir, diperbarui otomatis setiap ada data baru.

<img width="1600" height="900" alt="WhatsApp Image 2026-11-20 at 12 28 04" src="https://github.com/user-attachments/assets/d654b491-c662-494d-a217-a44ae779aa95" />

## 3. Autentikasi API Key (`X-API-Key`)

Endpoint yang mengubah data (`POST /api/kontrol` dan `PUT /api/threshold`) hanya menerima permintaan dengan header `X-API-Key` yang benar. Tanpa key atau dengan key salah, server membalas `401 Unauthorized`.

Catatan: key `rahasia123` hanya contoh untuk latihan.

```
curl.exe -i -X POST http://localhost:1880/api/kontrol -H "Content-Type: application/json" -d "{\"device\":\"lamp\",\"action\":\"on\"}"
```
<img width="1600" height="900" alt="WhatsApp Image 2026-11-20 at 12 25 20" src="https://github.com/user-attachments/assets/ba672549-c8d7-4184-a83b-d9fda403cb37" />

## 4. WebSocket (Update Tanpa Polling)

Node-RED mengirim data sensor ke dashboard lewat WebSocket di path `/ws/sensor`, sehingga dashboard tidak lagi melakukan polling ke `/api/suhu` dan `/api/history` secara berulang.

<img width="1600" height="900" alt="14-websocket-messages" src="https://github.com/user-attachments/assets/89157d34-507e-4fe9-bb03-7115f70cae03" />

<img width="1920" height="1080" alt="15-tanpa-polling" src="https://github.com/user-attachments/assets/08a17f5b-77c7-4963-945e-5f7f45eb9e91" />

## 5. History Disimpan ke Database SQLite

Setiap data sensor disimpan ke file SQLite (`sensor.db`), dan `/api/history` membaca data dari database tersebut. Dengan begitu history tidak hilang saat Node-RED di-restart.

<img width="1600" height="900" alt="16-flow-sqlite" src="https://github.com/user-attachments/assets/48ba866e-903e-4de7-8df3-21cad409ef76" />

<img width="1600" height="900" alt="19-api-history-sqlite" src="https://github.com/user-attachments/assets/f4b1a45b-eb51-4f21-b96b-4b60c0579456" />

---
