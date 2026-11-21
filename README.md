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
![Simulator](screenshot/01-simulator.png)

### 2. MQTT ke Node-RED
![Debug Node-RED](screenshot/02-debug-nodered.png)

### 3. Node-RED ke REST
![API suhu](screenshot/03-api-suhu.png)

### 4. REST ke Web
![Dashboard](screenshot/04-dashboard.png)

### 5. Web ke REST ke MQTT
![Kontrol lampu](screenshot/05-kontrol-lampu.png)

---

# Bagian 2: Tugas Pengembangan (Nilai Tambah)

## 1. Endpoint `/api/threshold` (PUT) dan Notifikasi Suhu

Endpoint PUT untuk mengatur batas suhu maksimum. Setiap data sensor yang melewati batas akan memicu notifikasi di panel Debug Node-RED.

Contoh pengujian:
```
curl.exe -X PUT http://127.0.0.1:1880/api/threshold -H "Content-Type: application/json" -H "X-API-Key: rahasia123" -d "{\"max_suhu\":27}"
```

![Flow threshold](screenshot/06-flow-threshold.png)
![Hasil PUT threshold](screenshot/07-put-threshold.png)
![Notifikasi suhu](screenshot/08-notifikasi-suhu.png)

## 2. Grafik Real-time dengan Chart.js

Dashboard menampilkan grafik garis suhu dan kelembaban untuk 20 data terakhir, diperbarui otomatis setiap ada data baru.

![Grafik real-time](screenshot/09-grafik.png)

## 3. Autentikasi API Key (`X-API-Key`)

Endpoint yang mengubah data (`POST /api/kontrol` dan `PUT /api/threshold`) hanya menerima permintaan dengan header `X-API-Key` yang benar. Tanpa key atau dengan key salah, server membalas `401 Unauthorized`.

Catatan: key `rahasia123` hanya contoh untuk latihan.

```
curl.exe -i -X POST http://localhost:1880/api/kontrol -H "Content-Type: application/json" -d "{\"device\":\"lamp\",\"action\":\"on\"}"
```
![Tanpa API key (401)](screenshot/10-tanpa-key-401.png)
![Dengan API key (200)](screenshot/11-dengan-key-200.png)

## 4. WebSocket (Update Tanpa Polling)

Node-RED mengirim data sensor ke dashboard lewat WebSocket di path `/ws/sensor`, sehingga dashboard tidak lagi melakukan polling ke `/api/suhu` dan `/api/history` secara berulang.

![WebSocket Messages](screenshot/12-websocket-messages.png)
![Tidak ada polling](screenshot/13-tanpa-polling.png)

## 5. History Disimpan ke Database SQLite

Setiap data sensor disimpan ke file SQLite (`sensor.db`), dan `/api/history` membaca data dari database tersebut. Dengan begitu history tidak hilang saat Node-RED di-restart.

![Flow SQLite](screenshot/14-flow-sqlite.png)
![History setelah restart](screenshot/15-history-setelah-restart.png)

---

## Catatan

- `sensor.db` berisi data sensor acak hasil simulasi.
- Dashboard harus dibuka lewat `http://localhost:1880/dashboard.html` (bukan `file:///`) agar tidak terkena pembatasan CORS.
