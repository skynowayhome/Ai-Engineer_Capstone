# ADR-001: Arsitektur Dua Lapis AI Engine (Model Diskriminatif Lokal sebagai Gerbang Utama + Gemini API sebagai Fitur Pendukung RAG)

## Status
Accepted

## Tanggal
2026-11-09

## Penanggung Jawab
- Rezki Langit Ramadhan (Model Klasifikasi Intent & Layanan FastAPI)
- Bestar Khan (Korpus Intent, Routing Aksi, RAG Dokumen Layanan, & Uji Out-of-Scope)

## Konteks & Aturan Khusus Learning Path Gen AI
Sesuai Project Plan resmi tim A26-CS054 dan arahan Q&A Briefing #1:
1. **Model AI utama harus dibangun dan dilatih sendiri** (model diskriminatif NLP klasifikasi intent menggunakan `scikit-learn`, bukan model siap pakai TF Hub atau AutoML).
2. **Generative AI (Google Gemini API) adalah fitur tambahan**, tidak boleh menjadi satu-satunya AI dan tidak boleh menggantikan model klasifikasi utama.
3. Alur yang disepakati: **Model klasifikasi intent buatan tim di layer awal (gerbang utama)** $\rightarrow$ request yang sesuai baru diteruskan ke GenAI/RAG atau formulir aksi sistem web.

## Keputusan Arsitektur
Membangun sistem AI dua lapis (*Layered Architecture*):
1. **Layer 1 (Model Utama Buatan Tim — Tanggung Jawab Rezki):**  
   Model klasifikasi teks berbasis `scikit-learn` dengan preprocessing TF-IDF yang memilah pesan ke dalam **4 intent resmi**:
   - `tanya_layanan`: Pertanyaan seputar jenis layanan dan estimasi biaya servis AC.
   - `booking`: Permintaan pemesanan jadwal teknisi ke lokasi.
   - `tanya_status`: Pengecekan progres tiket atau status pengerjaan servis.
   - `keluhan`: Keluhan kendala unit pascaservis atau klaim garansi.
2. **Layer 2 (Fitur Tambahan GenAI / RAG — Tanggung Jawab Bestar):**  
   Google Gemini 1.5 Flash API yang menjawab pertanyaan hanya berbasis dokumen katalog layanan terverifikasi (RAG), menolak pertanyaan di luar konteks servis AC (*guardrail*), serta menangani fallback kuota/jaringan secara aman.
3. **Layer Serving (FastAPI — Tanggung Jawab Rezki):**  
   Menyajikan endpoint `/predict-intent` untuk klasifikasi cepat dan `/chat` untuk alur percakapan interaktif yang diakses oleh backend Express.

## Konsekuensi & Kepatuhan
- **100% Selaras:** Selaras penuh dengan Project Plan tim A26-CS054 dan rubrik Main Quest Gen AI.
- **Efisiensi:** Menghemat kuota gratis Gemini (15 RPM) karena intent `booking`, `tanya_status`, dan `keluhan` langsung diarahkan oleh backend tanpa perlu memanggil Gemini.
- **Keandalan:** Jika API Gemini offline, layanan inti booking dan tiket tetap berjalan normal.
