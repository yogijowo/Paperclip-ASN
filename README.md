# Paperclip-ASN 🇮🇩

<p align="center">
  <strong>Sistem Operasi & Orkestrasi Tim AI Agent Otonom untuk Aparatur Sipil Negara (ASN)</strong>
  <br />
  <em>Dirancang untuk mempercepat administrasi naskah dinas, telaahan regulasi, risalah rapat, dan akuntabilitas kinerja pemerintah.</em>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="Lisensi MIT" /></a>
  <img src="https://img.shields.io/badge/Platform-macOS%20%7C%20Windows-green.svg" alt="Platform" />
  <img src="https://img.shields.io/badge/Framework-Tauri%20v2%20%2B%20React-orange.svg" alt="Framework" />
  <a href="https://github.com/paperclipai/paperclip"><img src="https://img.shields.io/badge/Based%20on-Paperclip-purple.svg" alt="Berbasis Paperclip" /></a>
</p>

---

## 📌 Tentang Proyek & Hubungan dengan Paperclip

**Paperclip-ASN** merupakan inisiatif *open-source* yang dikembangkan sebagai adaptasi dan pengembangan lanjutan dari **[Paperclip](https://github.com/paperclipai/paperclip)**. 

Jika **Paperclip** asli dirancang sebagai sistem orkestrasi tim AI agent untuk mengelola perusahaan virtual (*virtual business control plane*), maka **Paperclip-ASN** mengadaptasi arsitektur tersebut agar selaras dengan **tata laksana birokrasi, administrasi pemerintahan, dan regulasi di Indonesia**.

Proyek ini sepenuhnya menghormati lisensi asli **MIT License** dari Paperclip dan dirilis kembali secara terbuka agar rekan-rekan ASN di Kementerian, Lembaga, maupun Pemerintah Daerah dapat memanfaatkannya secara mandiri dan aman.

---

## 🌟 Fitur Utama

### 1. ⚡ Ringan dengan Tauri v2 (Mac & Windows)
* Dibungkus menggunakan **Tauri v2** (Rust + Native OS Webview), sehingga konsumsi memori hanya sekitar **40–80 MB RAM** (jauh lebih hemat dibandingkan Electron yang memakan 300 MB–1 GB).
* Sangat lancar dijalankan di laptop kedinasan standar (Intel Core i3/i5, RAM 4–8 GB).

### 2. 👥 4 Preset Agent Bawaan Khusus ASN
Paperclip-ASN menyertakan agen cerdas dengan *system prompt* yang sudah disesuaikan dengan regulasi Indonesia:
1. **Agen Tata Naskah Dinas (TND):** Menyusun Nota Dinas, Surat Edaran, Telaahan Staf, Undangan, dan Berita Acara sesuai **PermenPAN-RB No. 1 Tahun 2023**.
2. **Agen Analis Regulasi & Hukum:** Membedah peraturan (UU, PP, Perpres, Permen, Perda), menganalisis konsideran, dan memeriksa keselarasan pasal.
3. **Agen Risalah & Notulensi Rapat:** Mengolah transkrip atau catatan kasar rapat menjadi Risalah Resmi lengkap dengan matriks *Action Items* dan PIC.
4. **Agen Akuntabilitas Kinerja (SAKIP / LAKIP):** Menganalisis deviasi capaian target IKU terhadap pagu anggaran DIPA/DPA untuk laporan triwulanan dan tahunan.

### 3. 📁 Akses Folder Kerja Lokal Terbatas (*Privacy-First*)
* Pengguna menentukan folder dokumen kerja lokal di komputernya sendiri (misal: `D:\Dokumen_Kerja_2026`).
* Agen hanya boleh membaca dan menyusun draft pada folder yang dipilih, menjaga kerahasiaan data kedinasan Anda.

### 4. 📱 Kendali Jarak Jauh via Telegram Bot (*Remote Control*)
* Bagi ASN yang sedang tugas dinas luar atau rapat, Anda bisa memberikan perintah kerja dan menerima laporan langsung dari aplikasi Telegram di smartphone.
* Mendukung alur persetujuan (*Human-in-the-loop Approval*): draf dokumen dapat disetujui atau direvisi langsung dari chat Telegram.

### 5. 🔑 Multi-Provider AI (Cloud & Offline Lokal)
* Konfigurasikan model dan API Key secara fleksibel per masing-masing agen:
  * **Cloud:** Google Gemini (Flash/Pro), OpenAI (GPT-4o), Anthropic Claude, DeepSeek, Groq.
  * **Offline/Lokal:** Mendukung **Ollama** untuk instansi yang mewajibkan data tidak boleh keluar ke internet (100% On-Premise).

---

## 🏗️ Arsitektur Singkat

```
┌────────────────────────────────────────────────────────┐
│                     PENGGUNA ASN                       │
│       (Aplikasi Desktop Tauri ATAU Chat Telegram)      │
└──────────────┬─────────────────────────┬───────────────┘
               │                         │
               ▼                         ▼
┌────────────────────────────┐    ┌──────────────────────┐
│  Desktop App (Tauri v2)    │    │ Telegram Controller  │
│  - Native Folder Selector  │    │ - Perintah /tugas    │
│  - Dashboard Webview React │    │ - Notifikasi Hasil   │
└──────────────┬─────────────┘    └──────────┬───────────┘
               │                             │
               ▼                             ▼
┌────────────────────────────────────────────────────────┐
│             Paperclip Core Engine (Sidecar)            │
│  - Task Queue & Multi-Agent Orchestrator               │
│  - Adapters: Gemini, Claude, GPT, Ollama (Local)       │
│  - Embedded Database Lokal (SQLite / Postgres)         │
└──────────────────────────────┬─────────────────────────┘
                               │
                               ▼
┌────────────────────────────────────────────────────────┐
│               Folder Dokumen Lokal ASN                 │
│         (File PDF, Word, Excel di Laptop ASN)          │
└────────────────────────────────────────────────────────┘
```

---

## 🚀 Memulai Pengembangan

### Prasyarat:
* **Node.js:** v24.11 atau lebih baru (`node -v`)
* **pnpm:** v9+ (`npm install -g pnpm`)
* **Rust & Cargo:** (untuk kompilasi Tauri)

### 1. Instalasi Dependensi
```bash
pnpm install
```

### 2. Menjalankan Mode Pengembangan (Web Dev)
```bash
pnpm dev
```
Buka peramban di `http://localhost:3100` untuk melihat antarmuka Paperclip-ASN.

### 3. Menjalankan dengan Tauri Desktop
```bash
pnpm tauri dev
```

### 4. Membuat Installer Desktop
* **macOS (.dmg):**
  ```bash
  pnpm tauri build
  ```
* **Windows (.exe / .msi):**
  ```bash
  pnpm tauri build
  ```

---

## 📖 Panduan Lengkap

Silakan baca dokumen [PAPERCLIP_ASN_GUIDE.md](PAPERCLIP_ASN_GUIDE.md) untuk detail teknis mengenai:
* Prompt lengkap masing-masing Persona Agent ASN.
* Langkah integrasi bot Telegram.
* Konfigurasi dialog native folder picker.
* Panduan deployment dan distribusi open-source.

---

## 📜 Lisensi & Atribusi

Proyek ini dilisensikan di bawah **[MIT License](LICENSE)**.

```text
Paperclip-ASN — Asisten Otonom Tim AI untuk Aparatur Sipil Negara (ASN).
Dikembangkan berdasarkan proyek open-source Paperclip (https://github.com/paperclipai/paperclip).
Hak Cipta (c) 2026 Paperclip Authors & yogijowo (Kontributor Paperclip-ASN).
```
