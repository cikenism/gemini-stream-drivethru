# Drive Thru AI - Voice-based Ordering System

Sistem pemesanan *Drive Thru* berbasis suara ini menggunakan **Gemini Streaming API** untuk menangkap audio dan gambar dari pelanggan, lalu memprosesnya melalui LLM Google Gemini dalam bentuk WebSocket streaming.


## Arsitektur Singkat

```
[User Webcam + Microphone]
      ↓
[Frontend (HTML + JS)]
      ↓ WebSocket (ws://localhost:9082)
[Backend (Python + FastAPI + Gemini)]
      ↓
[Gemini AI] ⇄ [Function Call Tool: save_order()]
```

## Struktur File

```
.
├── main.py               # Backend server (WebSocket) dengan koneksi ke Gemini API
├── index.html            # Frontend UI Drive Thru
├── pcm-processor.js      # Pemrosesan audio untuk pemutaran kembali suara Gemini
├── .env.example          # Contoh environment variable
```

## Cara Menjalankan

### 1. Siapkan environment

Instal dependensi:

```bash
pip install google-genai==0.3.0 websockets
```

Isi file `.env` (atau langsung set di `main.py`):

```
GOOGLE_API_KEY=your_google_api_key
```

### 2. Jalankan WebSocket server

```bash
python main.py
```

WebSocket akan tersedia di:

```
ws://localhost:9082
```

### 3. Jalankan frontend

Buka `index.html` di browser via:

```bash
python -m http.server
```

Akses di: [http://localhost:8000](http://localhost:8000)

---

## Fitur AI yang Digunakan

### Gemini Streaming dengan `genai.Client`

* Menggunakan `live.connect()` untuk membuka sesi streaming dua arah.
* Mengirim:

  * Audio `audio/pcm`
  * Gambar `image/jpeg` hasil capture dari webcam
* Menerima:

  * Teks hasil transkripsi
  * Audio sintetis dari Gemini (`Kore` voice)

### Tool Function: `save_order()`

```python
def save_order(menu, qty, price):
    return {
        "menu": menu,
        "qty": qty,
        "price": price,
    }
```

Gemini akan otomatis memanggil fungsi ini saat mendeteksi pola pemesanan.

---

## Frontend Overview

### UI

* Dibuat dengan Material Design Lite (MDL)
* Elemen penting:

  * Webcam (video)
  * Tombol `Mulai Pesanan` dan `Akhiri Pesanan`
  * Daftar Pesanan
  * Tombol `Konfirmasi Order`

### JavaScript logic

* Capture audio + webcam secara real-time
* Mengirim audio + gambar base64 via WebSocket ke backend
* Menampilkan hasil teks dan update UI dari respons Gemini
* Memainkan kembali suara Gemini via `AudioWorkletProcessor`


## Audio Playback

File `pcm-processor.js` digunakan untuk:

* Menerima audio dari Gemini dalam bentuk base64 PCM
* Mengubah ke `Float32Array`
* Memutar ulang dengan `AudioContext` secara real-time


## Catatan Keamanan

* Saat ini WebSocket menggunakan koneksi `ws://localhost`. Untuk production, gunakan `wss://` dengan reverse proxy (misal NGINX).
* Belum ada autentikasi pengguna. Disarankan menggunakan JWT/OAuth jika sistem akan digunakan publik.

## Potensi Pengembangan

* Tambahkan login pengguna
* Simpan hasil pesanan ke database
* Dashboard kasir real-time
* Deployment ke Cloud Run / Vercel (frontend) + WebSocket di backend
