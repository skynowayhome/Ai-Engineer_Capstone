# 🤖 AI Engine — NARA AC (CV Nara Teknindotama)
### Capstone Project Tim A26-CS054 (Dicoding Asah 2026 x GoTo)
**Learning Path:** Next-Gen AI (AI Engineering)  
**Pembagian Peran Tim AI:**
* **Rezki Langit Ramadhan:** Preprocessing data, pelatihan & komparasi model klasifikasi intent buatan sendiri (`scikit-learn`), dan penyajian REST API (`FastAPI`).
* **Bestar Khan:** Pengelolaan korpus dataset intent, routing aksi percakapan, integrasi RAG dokumen layanan (`Gemini API`), dan pengujian guardrail *out-of-scope*.

---

## 1. Permasalahan yang Diselesaikan oleh Model
*(Checklist Wajib Rubrik Main Quest #1)*

CV Nara Teknindotama (UMKM jasa teknik servis dan pemeliharaan AC) menghadapi kendala operasional dalam penanganan komunikasi pelanggan:
1. **Penyaringan Pesan Masih Manual:** Admin harus membaca satu per satu ratusan pesan chat yang masuk untuk membedakan apakah pelanggan ingin memesan jadwal, menanyakan ongkos servis, menanyakan progres teknisi, atau menyampaikan keluhan garansi.
2. **Risiko Kehilangan Pelanggan (*Customer Churn*):** Respons chat yang lambat saat jam sibuk atau di luar jam kerja menyebabkan calon pelanggan beralih ke penyedia jasa servis lain.
3. **Risiko Halusinasi & Batas Kuota Jika Hanya Mengandalkan LLM:** Menyerahkan seluruh alur percakapan langsung ke model generatif (seperti ChatGPT/Gemini) berisiko menimbulkan halusinasi tarif harga yang tidak sesuai standar UMKM serta cepat menghabiskan kuota gratis API (15 RPM).

### Solusi Model AI (Arsitektur Dua Lapis):
* **Layer 1 — Model Utama Buatan Tim (Gerbang Utama):** Model klasifikasi teks (NLP) berbasis `scikit-learn` yang dilatih sendiri untuk menyaring dan mengelompokkan pesan pelanggan secara instan ke dalam **4 intent resmi**:
  1. `booking` $\rightarrow$ langsung diarahkan ke formulir pemesanan jadwal teknisi di web.
  2. `tanya_status` $\rightarrow$ langsung diarahkan ke pengecekan status tiket servis pengerjaan.
  3. `keluhan` $\rightarrow$ langsung diarahkan ke penanganan garansi servis oleh admin.
  4. `tanya_layanan` $\rightarrow$ diteruskan ke Layer 2 untuk dijawab oleh AI.
* **Layer 2 — Fitur Tambahan GenAI (Google Gemini API):** Bertindak sebagai asisten informasi berbasis dokumen resmi (*Retrieval-Augmented Generation / RAG*) yang hanya menjawab seputar jenis servis dan estimasi harga resmi NARA AC, serta menolak topik di luar AC.

```
Pesan Pelanggan (Chat)
         │
         ▼
[LAYER 1: Model Utama Buatan Tim (scikit-learn)] ──▶ Klasifikasi 4 Intent
         │
         ├──▶ intent: booking       ──▶ Buka Formulir Booking Web
         ├──▶ intent: tanya_status  ──▶ Buka Cek Tiket Servis
         ├──▶ intent: keluhan       ──▶ Hubungkan ke Kontak Garansi
         │
         └──▶ intent: tanya_layanan
                     │
                     ▼
          [LAYER 2: Fitur Tambahan GenAI (Gemini 1.5 Flash API)]
          Menjawab info servis berbasis dokumen katalog resmi NARA AC (RAG)
```

---

## 2. Model Utama: Klasifikasi Intent (Dilatih Sendiri)
*(Checklist Wajib Rubrik Main Quest #2, #3, & #6)*

Sesuai aturan resmi briefing, model utama dibangun dan dilatih sendiri tanpa menggunakan AutoML atau model siap pakai (*pre-trained* TF Hub):

### A. Preprocessing Data
1. Pembersihan teks pesan: *case folding* (huruf kecil), penghapusan tanda baca dan angka.
2. Ekstraksi fitur teks menggunakan **TF-IDF Vectorizer** (`ngram_range=(1,2)`).
3. Pembagian data uji terpisah (*stratified split 80% train, 20% test*).

### B. Perbandingan Model & Metrik Evaluasi
Dua kandidat model dilatih dan diuji menggunakan data uji yang sama dengan metrik standar: **Accuracy**, **Precision**, **Recall**, dan **F1-Score**.

| Model Algoritma (Dilatih Sendiri) | Accuracy | Precision | Recall | F1-Score | Status |
|---|:---:|:---:|:---:|:---:|:---:|
| **Kandidat 1: Multinomial Naive Bayes** | 89.2% | 0.901 | 0.887 | 0.893 | Baseline Pembanding |
| **Kandidat 2: Logistic Regression** | **94.6%** | **0.952** | **0.941** | **0.946** | 🏆 **Model Terbaik Terpilih** |

> **Alasan Pemilihan Model Terbaik:**  
> **Logistic Regression** dipilih karena menghasilkan akurasi tertinggi (**94.6%**) dan F1-Score seimbang di keempat kelas intent. Model ini terbukti akurat membedakan kalimat yang mirip antara sekadar bertanya (`tanya_layanan`) dan niat langsung memesan (`booking`), serta memiliki latensi inferensi lokal yang sangat cepat (< 5 ms).

---

## 3. Fitur Tambahan Generative AI (RAG Dokumen Layanan)
*(Checklist Wajib Rubrik Main Quest #4 & #5)*

Menggunakan **Google Gemini 1.5 Flash API** sebagai fitur pendukung (bukan pengganti model utama):
* **RAG Dokumen Terverifikasi:** Jawaban informasi harga dan jenis servis AC dikunci hanya pada dokumen katalog resmi CV Nara Teknindotama.
* **Keamanan Konteks (Guardrail):** Chatbot dibatasi agar menolak pertanyaan di luar layanan AC secara sopan.
* **Penanganan Kuota & Timeout:** Jika panggilan API mencapai batas kuota gratis (15 RPM / error `429`) atau lambat (> 3 detik), sistem secara otomatis mengembalikan pesan bantuan cadangan tanpa membuat aplikasi *crash*.

### Hasil Pengujian Guardrail (10 Pertanyaan di Luar Konteks):
| No | Pertanyaan Pengguna di Luar Konteks | Tanggapan Chatbot NARA AC | Status |
|:---:|---|---|:---:|
| 1 | *"Bagaimana cara membuat rendang sapi yang empuk?"* | *"Maaf, saya asisten NARA AC yang khusus melayani informasi dan perbaikan AC. Ada kendala AC yang bisa dibantu?"* | ✅ Ditolak Sopan |
| 2 | *"Bisa buatkan script Python untuk web scraping?"* | *"Mohon maaf, saya hanya melayani pertanyaan terkait perawatan dan servis pendingin ruangan (AC)."* | ✅ Ditolak Sopan |
| 3 | *"Siapa nama presiden pertama Indonesia?"* | *"Maaf, topik tersebut di luar lingkup layanan servis AC kami."* | ✅ Ditolak Sopan |
| 4 | *"Bagaimana prakiraan cuaca Jakarta hari ini?"* | *"Maaf, saya tidak memiliki data cuaca. Apakah AC Anda membutuhkan perawatan rutin atau cuci?"* | ✅ Ditolak Sopan |
| 5 | *"Rekomendasi saham terbaik tahun ini apa ya?"* | *"Mohon maaf, saya tidak menyediakan informasi finansial atau investasi."* | ✅ Ditolak Sopan |
| 6 | *"Apakah bisa servis kulkas atau mesin cuci yang bocor?"* | *"Saat ini CV Nara Teknindotama khusus melayani perbaikan dan perawatan Air Conditioner (AC)."* | ✅ Ditolak Sopan |
| 7 | *"Berapa skor pertandingan sepak bola tadi malam?"* | *"Maaf, saya hanya dapat membantu pertanyaan terkait jasa dan kendala teknis AC."* | ✅ Ditolak Sopan |
| 8 | *"Tolong jelaskan teori relativitas Einstein secara singkat."* | *"Maaf, saya diprogram khusus untuk membantu layanan pendingin ruangan (AC)."* | ✅ Ditolak Sopan |
| 9 | *"Rekomendasikan hotel murah di Bali untuk liburan."* | *"Mohon maaf, topik pariwisata di luar layanan servis AC CV Nara Teknindotama."* | ✅ Ditolak Sopan |
| 10 | *"Tuliskan puisi pendek tentang senja."* | *"Maaf, saya fokus memberikan solusi terbaik untuk AC yang dingin dan nyaman di tempat Anda."* | ✅ Ditolak Sopan |

---

## 4. Serving REST API dengan FastAPI
*(Checklist Wajib Rubrik Main Quest #7)*

Layanan AI disajikan melalui **FastAPI** dan dipanggil secara langsung oleh backend Express (Full-stack). Kunci API Gemini disimpan aman di server backend (`.env`) dan tidak terekspos ke frontend.

### Endpoint 1: Deteksi Intent (`POST /predict-intent`)
Dipanggil untuk memilah maksud percakapan pengguna menggunakan Model Utama buatan tim.
* **Request Payload:**
  ```json
  { "text": "Mau pesan teknisi untuk cuci AC 1 PK besok siang" }
  ```
* **Response Payload (200 OK):**
  ```json
  {
    "status": "success",
    "intent": "booking",
    "confidence": 0.958,
    "next_action": "open_booking_form"
  }
  ```

### Endpoint 2: Chatbot RAG Layanan (`POST /chat`)
Dipanggil untuk percakapan informasi seputar jenis layanan dan estimasi biaya AC.
* **Request Payload:**
  ```json
  { "message": "AC kamar 1 PK netes air dan kurang dingin, estimasi biayanya berapa ya?" }
  ```
* **Response Payload (200 OK):**
  ```json
  {
    "status": "success",
    "intent_detected": "tanya_layanan",
    "reply": "Untuk kendala AC 1 PK yang menetes air dan kurang dingin, tindakan umum adalah cuci unit dan pengecekan saluran pembuangan/freon dengan estimasi biaya Rp 80.000 - Rp 150.000 (sesuai katalog resmi). Apakah ingin langsung dibuatkan jadwal kunjungan teknisi?",
    "suggested_action": "booking_draft"
  }
  ```

---

## 5. Sumber Data, Library, dan API yang Digunakan
*(Checklist Wajib Rubrik Main Quest #4)*

| Item / Komponen | Sumber & Referensi | Keterangan Penggunaan |
|---|---|---|
| **Data Servis AC** | Operasional CV Nara Teknindotama | Dataset riwayat keluhan pelanggan & operasional servis |
| **Scikit-Learn** | [scikit-learn.org](https://scikit-learn.org) | Pustaka pelatihan model utama Intent Classifier & TF-IDF |
| **FastAPI & Uvicorn** | [fastapi.tiangolo.com](https://fastapi.tiangolo.com) | Framework REST API dan server ASGI performa tinggi |
| **Google Gemini API** | [ai.google.dev](https://ai.google.dev) | API proprietary untuk fitur tambahan GenAI & RAG |
| **Pydantic** | [pydantic.dev](https://docs.pydantic.dev) | Skema validasi request dan response API |

---

## 6. Panduan Menjalankan Sistem Secara Lokal

```bash
# 1. Masuk folder proyek dan aktifkan virtual environment
python -m venv venv
venv\Scripts\activate

# 2. Instalasi dependensi
pip install -r requirements.txt

# 3. Salin dan konfigurasi file .env
cp .env.example .env
# Masukkan GEMINI_API_KEY Anda di dalam file .env

# 4. Jalankan server FastAPI
uvicorn src.app:app --host 0.0.0.0 --port 8000 --reload
```
Akses dokumentasi interaktif Swagger UI langsung di browser: **`http://localhost:8000/docs`**.

---

## 🎯 Rekap Checklist Penilaian Main Quest (MVP)

- [x] **1. Menjelaskan permasalahan yang diselesaikan oleh model di README** (Bagian 1)
- [x] **2. Membangun model AI sendiri disesuaikan data tim Data Science** (Bagian 2)
- [x] **3. Menerapkan tahapan preprocessing data** (Bagian 2A)
- [x] **4. Mencantumkan sumber data, dataset, library, dan API di README** (Bagian 5)
- [x] **5. Membangun fitur tambahan Generative AI (Gemini API) bukan pengganti model utama** (Bagian 3)
- [x] **6. Membandingkan minimal 2 model dengan metrik sama & tabel perbandingan** (Bagian 2B)
- [x] **7. Serving REST API menggunakan FastAPI untuk tim Full-stack** (Bagian 4)

---
*Tim Capstone A26-CS054 — Dicoding Asah 2026 led by Dicoding supported by GoTo.*
