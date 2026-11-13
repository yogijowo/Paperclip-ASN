# PANDUAN PENGEMBANGAN DAN IMPLEMENTASI PAPERCLIP-ASN
> **Sistem Operasi Tim AI Agent Otonom Khusus Tata Kelola Administrasi & Kinerja Aparatur Sipil Negara (ASN)**
> *Berbasis Open Source [Paperclip](https://github.com/paperclipai/paperclip) dengan Antarmuka Ringan Tauri v2 (Mac & Windows) dan Kendali Jarak Jauh Telegram.*

---

## DAFTAR ISI
1. [Ringkasan Eksekutif & Visi](#1-ringkasan-eksekutif--visi)
2. [Arsitektur Sistem (Tauri v2 + Paperclip Core)](#2-arsitektur-sistem-tauri-v2--paperclip-core)
3. [Spesifikasi Personas & System Prompt Agent ASN](#3-spesifikasi-personas--system-prompt-agent-asn)
4. [Akses Folder & Manajemen Privasi Lokal](#4-akses-folder--manajemen-privasi-lokal)
5. [Integrasi Remote Control Telegram Bot](#5-integrasi-remote-control-telegram-bot)
6. [Manajemen Model AI & API Key per Agent](#6-manajemen-model-ai--api-key-per-agent)
7. [Panduan Langkah Demi Langkah (Step-by-Step) Pengerjaan](#7-panduan-langkah-demi-langkah-step-by-step-pengerjaan)
8. [Cara Build Installer Ringan (.dmg & .msi / .exe)](#8-cara-build-installer-ringan-dmg--msi--exe)
9. [Kepatuhan Lisensi & Hak Cipta (MIT)](#9-kepatuhan-lisensi--hak-cipta-mit)

---

## 1. RINGKASAN EKSEKUTIF & VISI

Banyak ASN di instansi pemerintah menghadapi beban kerja administratif yang tinggi, mulai dari penyusunan naskah dinas, reviu regulasi yang panjang, penyusunan risalah rapat, hingga pembuatan laporan kinerja tahunan (LAKIP/SAKIP).

**Paperclip-ASN** dibangun sebagai solusi asisten otonom multi-agen lokal:
* **Ringan & Hemat Sumber Daya:** Menggunakan **Tauri v2** (RAM ~40-80 MB), sangat cocok untuk laptop dinas standar (Core i3/i5, RAM 4-8 GB) dibandingkan Electron yang berat.
* **100% Mengutamakan Keamanan & Kerahasiaan Data Kedinasan:** Database dan eksekusi berjalan lokal di komputer pengguna; dokumen kedinasan tidak diunggah ke pihak ketiga di luar API resmi pilihan instansi.
* **Standar Tata Naskah Dinas Resmi:** Mengacu pada **PermenPAN-RB No. 1 Tahun 2023** dan **Perpres No. 95 Tahun 2018 tentang SPBE**.
* **Kendali Mobile Fleksibel:** ASN dapat memonitor atau memberi tugas kepada agen saat sedang rapat atau dinas luar via **Telegram Bot**.

---

## 2. ARSITEKTUR SISTEM (TAURI V2 + PAPERCLIP CORE)

```
┌─────────────────────────────────────────────────────────────┐
│                       PENGGUNA ASN                          │
│        (Aplikasi Desktop Tauri ATAU Chat Telegram)          │
└──────────────┬──────────────────────────────┬───────────────┘
               │                              │
               ▼ (Webview Ringan)             ▼ (Bot Polling)
┌─────────────────────────────────┐   ┌───────────────────────┐
│     Tauri v2 Desktop App        │   │ Telegram Bot Gateway  │
│  - Native File Picker (Folder)  │   │ - Perintah /tugas     │
│  - UI Dashboard (React + i18n)  │   │ - Notifikasi Approval │
│  - System Tray & Notifications  │   │ - Unduh Draft Dokumen │
└────────────────┬────────────────┘   └───────────┬───────────┘
                 │                                │
                 ▼                                ▼
┌─────────────────────────────────────────────────────────────┐
│                 Paperclip Server Core (Sidecar)             │
│  - Task Queue & Heartbeat Engine                            │
│  - Multi-Agent Orchestrator (Roles, Skills & Workflows)     │
│  - Multi-Provider Adapter (Gemini, Claude, GPT, Ollama)     │
│  - Embedded PostgreSQL (Database lokal di AppData/Library)  │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│             Folder Dokumen Kerja Lokal ASN                  │
│       (Folder Dokumen Dinas, Format Word/PDF/Excel)         │
└─────────────────────────────────────────────────────────────┘
```

### Mengapa Tauri v2?
1. **Ukuran File Kecil:** Installer hanya ~15-25 MB (dibandingkan Electron yang 100-200 MB).
2. **Konsumsi RAM Sangat Rendah:** Memanfaatkan native webview bawaan OS (WebKit di macOS, WebView2 di Windows 10/11).
3. **Keamanan Ekstra:** Fitur sandboxing bawaan Rust dan permission model ketat untuk akses filesystem.

---

## 3. SPESIFIKASI PERSONAS & SYSTEM PROMPT AGENT ASN

Berikut adalah 4 agen utama yang wajib ditanamkan sebagai *Default Preset* di Paperclip-ASN:

### A. Agen 1: Tata Naskah Dinas (TND)
* **Tugas:** Menyusun draft Nota Dinas, Surat Edaran, Telaahan Staf, Surat Undangan, dan Berita Acara.
* **Aturan:** Menggunakan kaidah Bahasa Indonesia Baku (EYD V) dan struktur baku PermenPAN-RB No. 1 Tahun 2023.
* **System Prompt:**
```text
Anda adalah Agen Tata Naskah Dinas (TND) yang bertugas menyusun naskah dinas resmi pemerintah Republik Indonesia.
Pedoman baku Anda adalah Peraturan Menteri PAN-RB No. 1 Tahun 2023 tentang Tata Naskah Dinas Instansi Pemerintah.

ATURAN STRUKTUR:
1. KEPALA SURAT: Judul Naskah (contoh: NOTA DINAS), Nomor, Tujuan (Yth.), Dari, Hal, Tanggal.
2. BATANG TUBUH:
   - Alinea Pembuka: Menguraikan dasar pertimbangan, latar belakang, atau rujukan surat masuk.
   - Alinea Isi: Uraian substansi yang jelas, padat, menggunakan penomoran hierarkis (1, a, 1), a)).
   - Alinea Penutup: Kalimat simpulan/harapan/tindak lanjut yang lazim (contoh: "Demikian untuk menjadi maklum dan mohon petunjuk lebih lanjut").
3. KAKI SURAT: Nama Jabatan, Tanda Tangan/Digital, Nama Lengkap (tanpa/dengan NIP sesuai jenis naskah), dan Tembusan.

Gaya bahasa wajib formal, santun, lugas, tanpa basa-basi berlebihan, serta bebas dari kesalahan ejaan.
```

### B. Agen 2: Analis Regulasi & Hukum
* **Tugas:** Menelaah peraturan perundang-undangan (UU, PP, Perpres, Permen, Perda), memetakan konsideran, dan memeriksa kesesuaian kebijakan (*compliance*).
* **System Prompt:**
```text
Anda adalah Analis Regulasi dan Kepatuhan Hukum Pemerintahan.
Tugas Anda adalah menelaah dokumen peraturan yang ada di folder kerja pengguna.

STANDAR ANALISIS:
1. Identifikasi hierarki peraturan perundang-undangan (UU No. 12/2011 jo UU No. 13/2022).
2. Bedah Konsideran: Menimbang (landasan filosofis & sosiologis) dan Mengingat (landasan yuridis).
3. Uraikan Implikasi Praktis: Dampak peraturan terhadap unit kerja pengguna.
4. Buat Matriks Perbandingan: Sebelum vs Sesudah perubahan pasal secara tabelatis.
```

### C. Agen 3: Notulis & Risalah Rapat Koordinasi
* **Tugas:** Mengubah catatan mentah atau transkrip rapat menjadi Risalah Rapat Kedinasan yang siap didistribusikan.
* **System Prompt:**
```text
Anda adalah Notulis Risalah Rapat Kedinasan.
Format output wajib mencakup:
1. Waktu, Hari, Tanggal, dan Tempat Pelaksanaan.
2. Pimpinan Rapat dan Notulis.
3. Daftar Peserta (Satuan Kerja / Unit yang hadir).
4. Agenda Rapat.
5. Poin-poin Pembahasan & Dinamika Rapat.
6. KEPUTUSAN & RENCANA TINDAK LANJUT (Matrix Action Items: Kegiatan, Target Waktu, PIC/Penanggung Jawab).
```

### D. Agen 4: Evaluator Akuntabilitas & Laporan Kinerja (SAKIP/LAKIP)
* **Tugas:** Membantu mengolah draft Capaian Kinerja, menganalisis Deviasi Realisasi vs Target IKU (Indikator Kinerja Utama), dan merumuskan langkah korektif.
* **System Prompt:**
```text
Anda adalah Analis Akuntabilitas Kinerja Instansi Pemerintah (AKIP).
Anda membantu ASN menyusun narasi evaluasi kinerja triwulanan dan tahunan.
Fokus pada:
1. Realisasi IKU terhadap target Renstra/PK (Perjanjian Kinerja).
2. Analisis efisiensi penggunaan pagu anggaran DIPA/DPA.
3. Identifikasi kendala internal & eksternal penyebab target tidak tercapai.
4. Rekomendasi perbaikan operasional untuk periode berikutnya.
```

---

## 4. AKSES FOLDER & MANAJEMEN PRIVASI LOKAL

Agar pengguna dapat menentukan folder dokumen mana yang boleh dibaca atau ditulisi oleh agen:

1. **Native Dialog Picker (Tauri API):**
   Tambahkan plugin `@tauri-apps/plugin-dialog` pada Tauri:
   ```typescript
   import { open } from '@tauri-apps/plugin-dialog';

   async function selectWorkDirectory() {
     const selected = await open({
       directory: true,
       multiple: false,
       title: 'Pilih Folder Dokumen Kerja Dinas'
     });
     if (selected) {
       console.log('Folder terpilih:', selected);
       // Simpan path ke instance settings Paperclip
     }
   }
   ```
2. **Keamanan Read-Only vs Read-Write:**
   * Di dashboard, beri toggle:
     * `Hanya Baca (Read-Only)`: Agen hanya menganalisis dokumen tanpa mengubah file asli.
     * `Izin Tulis (Read-Write)`: Agen boleh membuat draft file baru (misal: `Draft_Nota_Dinas_v1.docx`) di subfolder `/output/`.

---

## 5. INTEGRASI REMOTE CONTROL TELEGRAM BOT

Untuk ASN yang mobile, Telegram Bot bertindak sebagai gerbang kendali jarak jauh yang aman.

### Alur Integrasi:
1. **Daftarkan Bot di @BotFather:** Dapatkan `TELEGRAM_BOT_TOKEN`.
2. **Whitelist User ID:** Di pengaturan Paperclip-ASN, masukkan `TELEGRAM_ALLOWED_USER_ID` (agar orang lain tidak bisa memerintah bot Anda).
3. **Mekanisme Polling (Tanpa Perlu IP Publik / Domain):**
   Gunakan library Node.js ringan (seperti `grammy`):
   ```typescript
   import { Bot } from "grammy";

   const bot = new Bot(process.env.TELEGRAM_BOT_TOKEN!);

   bot.command("start", (ctx) => {
     if (ctx.from?.id.toString() !== process.env.TELEGRAM_ALLOWED_USER_ID) {
       return ctx.reply("Akses ditolak: User ID Anda tidak terdaftar.");
     }
     ctx.reply("Selamat datang di Paperclip-ASN Controller. Ketik /menu untuk melihat daftar agen dinas.");
   });

   bot.command("tugas", async (ctx) => {
     // Format: /tugas [nama_agen] [instruksi]
     // Masukkan instruksi ke antrean Paperclip Task Queue
     ctx.reply("Tugas telah diterima dan sedang diproses oleh Agen ASN...");
   });

   bot.start();
   ```
4. **Approval Flow di HP:**
   Ketika agen selesai membuat draf dokumen, bot mengirimkan ringkasan + file draf ke Telegram dengan tombol *Inline Keyboard*:
   * `[✅ Setujui]` -> Agen memindahkan file ke folder final.
   * `[✏️ Minta Revisi]` -> ASN membalas dengan catatan revisi langsung di chat.

---

## 6. MANAJEMEN MODEL AI & API KEY PER AGENT

Pengguna dapat memilih model yang paling sesuai per kebutuhan:

| Agen ASN | Model Rekomendasi | Alasan |
| :--- | :--- | :--- |
| **Tata Naskah Dinas** | GPT-4o / Claude 3.5 Sonnet / Gemini 2.5 Pro | Kemampuan bahasa Indonesia formal dan kerapian format naskah |
| **Analis Regulasi** | Gemini 2.5 Flash / Pro (Context Window 1M+) | Mampu membaca dokumen UU/Perda ratusan halaman dalam 1 prompt |
| **Dokumen Rahasia Negara** | Local LLM via Ollama (Llama 3 / Mistral / DeepSeek) | Data 100% tidak keluar dari komputer lokal (offline ready) |

Setiap agen memiliki kolom konfigurasi terpisah:
* `Provider` (Google Gemini, Anthropic, OpenAI, DeepSeek, Ollama/Local)
* `API Key` (Disimpan dengan enkripsi AES-256 lokal)
* `Model Name` (Contoh: `gemini-2.5-flash`, `gpt-4o`, `ollama/qwen2.5:7b`)
* `Temperature` (Disetel rendah, misal `0.2` untuk akurasi hukum/administrasi)

---

## 7. PANDUAN LANGKAH DEMI LANGKAH PENGERJAAN

### Langkah 1: Persiapan Lingkungan Pengembangan
Pastikan perangkat Anda memiliki:
* **Node.js:** v24.11 atau lebih baru (`node -v`)
* **pnpm:** v9+ (`npm install -g pnpm`)
* **Rust & Cargo:** (`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`)

### Langkah 2: Inisialisasi Tauri di Proyek Paperclip-ASN
Di dalam direktori `Paperclip-ASN`:
```bash
# Tambahkan dependensi Tauri CLI
pnpm add -D @tauri-apps/cli @tauri-apps/api @tauri-apps/plugin-dialog

# Jalankan inisialisasi Tauri
pnpm tauri init
```
Konfigurasi `tauri.conf.json`:
* `beforeDevCommand`: Script untuk menyalakan frontend Paperclip.
* `devUrl`: `http://localhost:3100` (atau port dev Paperclip UI).
* `beforeBuildCommand`: `pnpm build`.
* `frontendDist`: Path hasil build frontend UI.

### Langkah 3: Menerapkan Lokalisasi Bahasa Indonesia (i18n)
1. Buka folder `ui/src/`.
2. Pasang library `i18next` dan `react-i18next`.
3. Buat file kamus istilah ASN `ui/src/locales/id.json` untuk mengganti istilah:
   * *Company* -> *Instansi / Satuan Kerja*
   * *Project* -> *Program / Kegiatan Dinas*
   * *Issue / Task* -> *Disposisi / Agenda Kerja*
   * *Agent* -> *Pejabat Fungsional / Asisten Virtual*
   * *Budget* -> *Pagu Anggaran / Kuota Token*

### Langkah 4: Tanamkan Agent Preset ASN
Buat file seeder default di `server/src/seeds/asn-agents.ts` yang otomatis mendaftarkan 4 Agen ASN di atas saat aplikasi pertama kali dijalankan (*first boot*).

### Langkah 5: Pasang Modul Telegram Controller
Buat service di `server/src/services/connectors/telegram-bot.ts` yang mengikat bot Telegram ke Paperclip task service.

---

## 8. CARA BUILD INSTALLER RINGAN (.DMG & .MSI / .EXE)

Untuk menghasilkan file installer yang siap dibagikan ke rekan-rekan ASN:

### A. Untuk macOS (M-Series & Intel)
```bash
pnpm tauri build
```
Hasil installer akan berada di:
`src-tauri/target/release/bundle/dmg/Paperclip-ASN_x.x.x_aarch64.dmg`

### B. Untuk Windows 10/11
Jalankan perintah yang sama di mesin Windows atau via GitHub Actions CI:
```bash
pnpm tauri build
```
Hasil installer akan berada di:
`src-tauri/target/release/bundle/nsis/Paperclip-ASN_x.x.x_x64-setup.exe`  
(atau format `.msi`).

---

## 9. KEPATUHAN LISENSI & HAK CIPTA (MIT)

Proyek ini dibangun di atas pondasi [Paperclip](https://github.com/paperclipai/paperclip). Agar tetap sah dan menjunjung etika *open-source*:

1. **File Lisensi:** Tetap pertahankan file `LICENSE` asli berlisensi MIT.
2. **Pernyataan Atribusi:** Di menu *Tentang Aplikasi*, cantumkan:
   ```text
   Paperclip-ASN — Asisten Otonom Tim AI untuk Aparatur Sipil Negara.
   Dikembangkan berdasarkan proyek open-source Paperclip (https://github.com/paperclipai/paperclip).
   Lisensi: MIT License. Hak Cipta (c) 2026 Paperclip Authors & Kontributor ASN Indonesia.
   ```
3. **Distribusi Open Source:** Unggah kode hasil fork ini ke GitHub publik agar seluruh ASN di kementerian, lembaga, dan pemerintah daerah di seluruh Indonesia dapat berkontribusi dan memanfaatkan teknologi ini secara gratis.

---

*Dokumen ini dirancang sebagai acuan kerja lengkap dalam merealisasikan Paperclip-ASN.*

