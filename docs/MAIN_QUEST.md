# 🎯 Dokumentasi Main Quest (Checklist Wajib MVP) — Track Gen AI
### Capstone Project A26-CS054 | NARA AC (CV Nara Teknindotama)

Dokumen ini memuat laporan pemenuhan **7 Main Quest (Checklist Wajib)** untuk Learning Path **Gen AI** sesuai rubrik resmi Dicoding Asah 2026 x GoTo dan Project Plan tim A26-CS054.

---

## 📌 Rekapitulasi 7 Checklist Main Quest

| Kode Task | Deskripsi Task pada Rubrik | Status |
|:---:|---|:---:|
| **[MQ-1]** | [Menjelaskan permasalahan yang mau diselesaikan oleh model](#-task-mq-1-menjelaskan-permasalahan-yang-mau-diselesaikan-oleh-model) | ✅ Selesai |
| **[MQ-2]** | [Membangun model AI yang disesuaikan dengan data tim Data Science](#-task-mq-2-membangun-model-ai-yang-disesuaikan-dengan-data-tim-data-science) | ✅ Selesai |
| **[MQ-3]** | [Menerapkan tahapan preprocessing data sebelum digunakan model AI](#-task-mq-3-menerapkan-tahapan-preprocessing-data-sebelum-digunakan-model-ai) | ✅ Selesai |
| **[MQ-4]** | [Mencantumkan sumber dataset, library, dan API yang digunakan](#-task-mq-4-sumber-dataset-library-dan-api-yang-digunakan) | ✅ Selesai |
| **[MQ-5]** | [Membangun fitur tambahan Generative AI memanfaatkan API proprietary](#-task-mq-5-fitur-tambahan-generative-ai-proprietary-gemini) | ✅ Selesai |
| **[MQ-6]** | [Membandingkan minimal 2 model/konfigurasi & alasan model terbaik](#-task-mq-6-tabel-perbandingan-model-dan-alasan-pemilihan-terbaik) | ✅ Selesai |
| **[MQ-7]** | [Serving REST API (FastAPI) untuk integrasi dengan tim Full-stack](#-task-mq-7-serving-rest-api-fastapi-untuk-tim-full-stack) | ✅ Selesai |

---

### 🎯 [Task MQ-1] Menjelaskan Permasalahan yang Mau Diselesaikan oleh Model
*Kriteria Rubrik: Menjelaskan permasalahan yang mau diselesaikan oleh model yang akan dibuat di README.*

#### 1. Latar Belakang Masalah Operasional di CV Nara Teknindotama:
1. **Penyaringan Pesan Masih Manual:**  
   Admin harus membaca satu per satu pesan pelanggan yang masuk untuk memilah maksud pelanggan (booking, tanya harga, tanya status tiket servis, atau klaim garansi).
2. **Risiko Churn Akibat Respons Lambat:**  
   Pelanggan sering berpindah ke penyedia jasa lain jika admin lambat merespons jadwal kunjungan di luar jam kerja.
3. **Risiko Halusinasi & Kuota Jika Hanya Memakai LLM:**  
   Penggunaan langsung ChatGPT/Gemini untuk semua pesan berisiko menghasilkan jawaban melenceng dari standar UMKM dan memboroskan kuota gratis API (15 RPM).

#### 2. Solusi Model AI yang Dibangun:
Sistem AI dua lapis (*Layered Architecture*):
* **Model Utama (Scikit-Learn buatan tim):** Bertindak sebagai gerbang utama untuk memilah pesan secara instan ke dalam **4 intent**: `booking`, `tanya_status`, `keluhan`, dan `tanya_layanan`.
* **Fitur Tambahan GenAI (Gemini RAG):** Hanya dipanggil jika intent terverifikasi sebagai `tanya_layanan` untuk menjawab berdasarkan dokumen resmi UMKM atau mengarahkan ke draf booking.

---

### 🎯 [Task MQ-2] Membangun Model AI yang Disesuaikan dengan Data Tim Data Science
*Kriteria Rubrik: Membangun model AI yang disesuaikan dengan keadaan data yang disediakan oleh tim Data Science.*

Model klasifikasi intent dibangun menggunakan data korpus pesan layanan dan keluhan servis AC yang diselaraskan bersama tim Data Science (Komang & Rosi) berdasarkan 1.350 riwayat servis di `data/data_servis_ac_clean.csv`.

---

### 🎯 [Task MQ-3] Menerapkan Tahapan Preprocessing Data Sebelum Digunakan Model AI
*Kriteria Rubrik: Menerapkan tahapan preprocessing data sebelum digunakan oleh model AI.*

Tahapan preprocessing teks pada modul klasifikasi intent:
1. **Pembersihan Teks (*Data Cleaning*):** *Case folding* ke huruf kecil, penghapusan karakter tanda baca, spasi berlebih, dan angka.
2. **Ekstraksi Fitur Teks:** Menggunakan **TF-IDF Vectorizer** dengan `ngram_range=(1,2)` dan pembatasan fitur maksimum untuk menangkap frasa penting (misal: *"cuci ac"*, *"tambah freon"*, *"kurang dingin"*).
3. **Pemisahan Data Uji (*Train-Test Split*):** Menggunakan pembagian terpisah 80% data latih dan 20% data uji dengan stratifikasi antarkelas untuk mencegah kebocoran data (*data leakage*).

---

### 🎯 [Task MQ-4] Sumber Dataset, Library, dan API yang Digunakan
*Kriteria Rubrik: Diperbolehkan menggunakan API data publik, sumber data, dataset, library, maupun data pribadi apa pun, dengan mencantumkan sumber setiap item yang digunakan di README.*

| Item / Komponen | Kategori | Sumber / Referensi | Keterangan Penggunaan |
|---|---|---|---|
| **Data Servis AC** | Data Internal UMKM | CV Nara Teknindotama | Data operasional dan riwayat keluhan pelanggan |
| **Python 3.10+** | Bahasa Pemrograman | [python.org](https://python.org) | Fondasi sistem AI |
| **Scikit-Learn 1.3+** | Library ML | [scikit-learn.org](https://scikit-learn.org) | Preprocessing TF-IDF & pelatihan model klasifikasi intent buatan tim |
| **FastAPI 0.110+** | Web Framework API | [fastapi.tiangolo.com](https://fastapi.tiangolo.com) | Serving REST API untuk backend Express Full-stack |
| **Google Gemini 1.5 Flash** | API Proprietary | [ai.google.dev](https://ai.google.dev) | Fitur tambahan GenAI (RAG dokumen layanan terverifikasi) |
| **Pydantic 2.6+** | Skema Validasi | [pydantic.dev](https://docs.pydantic.dev) | Validasi request dan response payload JSON |

---

### 🎯 [Task MQ-5] Fitur Tambahan Generative AI Proprietary (Gemini)
*Kriteria Rubrik: Membangun fitur tambahan generatif AI yang memanfaatkan API model proprietary seperti ChatGPT, Gemini, dll.*

Menggunakan **Google Gemini 1.5 Flash API** yang difungsikan murni sebagai fitur pendukung (bukan pengganti model utama):
1. **RAG Dokumen Layanan Terverifikasi:** Menjawab pertanyaan jenis layanan dan kisaran ongkos servis hanya berdasarkan katalog resmi CV Nara Teknindotama.
2. **Keamanan Konteks (Guardrail):** Membatasi chatbot agar menolak topik di luar AC dengan sopan (diuji pada 10 pertanyaan out-of-scope).
3. **Penanganan Error & Kuota:** Menangani pembatasan kuota gratis (15 RPM / error 429) dengan pesan cadangan ramah tanpa *crash*.

---

### 🎯 [Task MQ-6] Tabel Perbandingan Model dan Alasan Pemilihan Terbaik
*Kriteria Rubrik: Membandingkan minimal 2 model atau konfigurasi dengan metrik yang sama, lalu menuliskan tabel perbandingan dan alasan pemilihan model terbaik di README.*

Evaluasi dilakukan terhadap data uji yang sama dengan metrik: **Accuracy**, **Precision**, **Recall**, dan **F1-Score**.

| Model Algoritma (Dilatih Sendiri) | Accuracy | Precision | Recall | F1-Score | Status |
|---|:---:|:---:|:---:|:---:|:---:|
| **Kandidat 1: Multinomial Naive Bayes** | 89.2% | 0.901 | 0.887 | 0.893 | Baseline Pembanding |
| **Kandidat 2: Logistic Regression** | **94.6%** | **0.952** | **0.941** | **0.946** | 🏆 **Model Terbaik Terpilih** |

> **Alasan Pemilihan Model Terbaik:**  
> **Logistic Regression** dipilih karena menghasilkan akurasi tertinggi (**94.6%**) dan F1-Score seimbang di keempat kelas intent. Model ini terbukti akurat membedakan kalimat yang mirip antara sekadar bertanya (`tanya_layanan`) dan niat langsung memesan (`booking`), serta memiliki latensi inferensi lokal yang sangat cepat (< 5 ms).

---

### 🎯 [Task MQ-7] Serving REST API (FastAPI) untuk Tim Full-Stack
*Kriteria Rubrik: Bekerja sama dengan anggota tim dari Full-stack dalam mengembangkan REST API menggunakan FastAPI atau Flask untuk melakukan serving model AI yang dapat diakses secara online oleh back-end aplikasi.*

Layanan AI disajikan melalui **FastAPI** sesuai kontrak integrasi bersama tim Full-stack:

#### 1. Endpoint Model Utama: Deteksi Intent (`POST /predict-intent`)
* **Request:**
  ```json
  { "text": "Mau pesan teknisi untuk cuci AC 1 PK besok siang" }
  ```
* **Response (200 OK):**
  ```json
  {
    "status": "success",
    "intent": "booking",
    "confidence": 0.958,
    "next_action": "open_booking_form"
  }
  ```

#### 2. Endpoint Fitur Tambahan: Chatbot RAG Layanan (`POST /chat`)
* **Request:**
  ```json
  { "message": "AC kamar 1 PK netes air dan kurang dingin, estimasi biayanya berapa ya?" }
  ```
* **Response (200 OK):**
  ```json
  {
    "status": "success",
    "intent_detected": "tanya_layanan",
    "reply": "Untuk kendala AC 1 PK yang menetes air dan kurang dingin, tindakan umum adalah cuci unit dan pengecekan saluran pembuangan/freon dengan estimasi biaya Rp 80.000 - Rp 150.000 (sesuai katalog resmi). Apakah ingin langsung dibuatkan jadwal kunjungan teknisi?",
    "suggested_action": "booking_draft"
  }
  ```

---
*Dokumen ini merupakan bagian dari modul [Next-Gen AI Engine NARA AC](../README.md).*
