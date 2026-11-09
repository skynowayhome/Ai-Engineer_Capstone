# NARA AC — AI Engine & Problem Statement
**Capstone Project Tim A26-CS054 (Dicoding Asah 2026 x GoTo)**  
**UMKM Mitra:** CV. Nara Teknindotama (Jasa Teknik & Servis AC)

---

## 📌 Permasalahan Bisnis UMKM

CV. Nara Teknindotama mengalami kendala operasional dalam penanganan komunikasi dan permintaan layanan pelanggan melalui pesan percakapan:

1. **Penyaringan Pesan Manual:**  
   Admin menghabiskan waktu membaca pesan masuk satu per satu untuk memilah niat pelanggan (*booking*, tanya harga, status servis, atau komplain garansi).

2. **Keterlambatan Respons (*Customer Churn*):**  
   Respons yang lambat pada jam sibuk menyebabkan calon pelanggan berpindah ke jasa servis AC lain.

3. **Risiko Biaya & Halusinasi jika Hanya Mengandalkan LLM:**  
   Menggunakan LLM murni berisiko menimbulkan jawaban harga yang berhalusinasi dan cepat melewati batas kuota gratis API.

---

## 💡 Solusi Model AI

Untuk menyelesaikan masalah tersebut, dibangun sistem **AI Dua Lapis**:

- **Model AI Utama (Klasifikasi Intent):**  
  Model klasifikasi teks buatan tim yang secara otomatis mengelompokkan pesan pelanggan ke dalam 4 kategori (*booking*, *tanya_status*, *keluhan*, dan *tanya_layanan*).

- **Fitur Tambahan GenAI (RAG Dokumen Layanan):**  
  Fitur generatif menggunakan Google Gemini API yang terhubung dengan dokumen katalog resmi untuk menjawab pertanyaan seputar estimasi harga dan jenis layanan secara akurat.

---

## 🛠️ Pustaka & Sumber Data

- **Dataset:** Data riwayat percakapan dan katalog layanan resmi CV. Nara Teknindotama.
- **Library ML:** `scikit-learn` (TF-IDF Vectorizer & Intent Classifier).
- **Framework API:** `FastAPI` & `Uvicorn`.
- **API Proprietary:** Google Gemini 1.5 Flash API (khusus fitur RAG).
