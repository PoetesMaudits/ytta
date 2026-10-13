# TELKRAF STUDIO — UI/UX DESIGN CONSTITUTION & REPOSITORY ARCHITECTURE

> **Dokumen Panduan Standar Tunggal (SSOT)**  
> Diterapkan secara wajib untuk seluruh pengembangan antarmuka web, portal klien, dan dokumen interaktif di lingkungan repositori GitHub Telkraf.id (termasuk `YTTA` dan sub-modul kemitraan).

---

## 1. Identitas Visual & Tipografi (Strict)

### A. Tipografi Wajib
* **Primary Display & Body Font:** `Plus Jakarta Sans` (Weight: 400, 500, 600, 700, 800, 900).
  ```html
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800;900&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
  ```
* **Mono Font (Angka, Kalkulator, Tag Metrik, Kode):** `JetBrains Mono` (Weight: 400, 500, 700).
* **Font Family Token:**
  ```css
  --font-display: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
  ```

### B. Palet Warna & Signature Studio Gradient
* **Signature Studio Gradient Telkraf:**
  `linear-gradient(90deg, #F5E03A 0%, #F0527A 50%, #8B2FC2 100%)`  
  *Penggunaan:* Aksen border atas kartu/modal, highlight judul teks (via `-webkit-background-clip: text`), dan tombol aksi utama (CTA).
* **Dark Canvas Background:** `#08080a` atau `#0D0F1D`.
* **Card Surface (Glassmorphic):** `rgba(18, 18, 24, 0.88)` dengan `backdrop-filter: blur(20px)`.
* **Border Token:**
  * Subtle: `rgba(255, 255, 255, 0.08)` (Garis pembatas kartu, separator, input).
  * Strong: `rgba(255, 255, 255, 0.14)` (State aktif, focus, divider vertikal).
* **Warna Aksen Fungsional:**
  * Kuning / Primary Accent: `#F5E03A`
  * Merah Muda / Secondary Accent: `#F0527A`
  * Ungu / Tertiary Accent: `#8B2FC2`
  * Hijau / Success: `#00e676`
  * Sian / Info: `#00d2ff`

### C. Logo & Favicon Resmi
* **Favicon:** File SVG resmi dari master asset [`favicon.svg`](./favicon.svg) wajib ditautkan pada setiap halaman HTML:
  ```html
  <link rel="icon" type="image/svg+xml" href="favicon.svg">
  ```
* **Badan Hukum & Entitas:** PT Telekomunikasi & Industri Kreatif Indonesia / Telkraf.id.
* **Rekening Resmi (Bila Tercantum):** Bank Syariah Indonesia (BSI) No. 1043097241 a.n. Khairul Hafizi.

---

## 2. Golden Rules Antarmuka Pengguna (UI Rules)

1. **Aturan Tombol (Buttons):**
   * Format wajib: `[Icon] + [Maksimal 2 Kata]`.
   * Bersifat ringkas (*concise*), berwibawa, dan **tanpa angka**.
   * *Contoh Benar:* `<span>🔓</span> <span>Buka Vault</span>`, `<span>📑</span> <span>Baca Laporan</span>`, `<span>←</span> <span>Kembali</span>`.
   * *Contoh Dilarang:* "Buka Tab Baru Dokumen 1", "Klik di sini untuk melihat 3 dokumen".

2. **Haram Pill / Kapsul Liar:**
   * Dilarang menggunakan elemen pill/kapsul dekoratif mengambang yang tidak memiliki fungsi interaktif.
   * Tag status teknis diperbolehkan selama berukuran micro, minimalis, dan menggunakan font mono tipis.

3. **Integritas Bahasa & Anti-Placeholder:**
   * DILARANG keras menyisipkan teks bercanda, lelucon internal, atau placeholder informal (misal: *"Welcome to my ytta repo; you were chosen..."*).
   * Seluruh teks harus mencerminkan reputasi studio kreatif berstandar korporat dan kredibel.

4. **Radius & Glassmorphism:**
   * Card utama: `border-radius: 16px–20px`.
   * Tombol & Input: `border-radius: 8px–10px`.
   * Efek Hover: `transform: translateY(-2px) scale(1.01); border-color: rgba(255, 255, 255, 0.22);`.

---

## 3. Arsitektur Multi-Dokumen (Skalabel & Native)

Setiap portal multi-dokumen (seperti `utary/`) harus mematuhi pola arsitektur berikut:

1. **Halaman Beranda Direktori Hub (`#doc-hub`):**
   * Bertindak sebagai galeri kartu dokumen utama (*Single Entry Point*).
   * Bersih dari kotak komitmen atau pesan operasional penahan.

2. **Render Native (Tanpa Iframe):**
   * Setiap dokumen baru (contoh: Blueprint Canva, Riset Tren, Laporan Eksekutif) dirender langsung sebagai `<div id="doc-[name]" class="workspace-doc-view">`.
   * Dilarang membungkus dokumen dengan `<iframe>` yang menyebabkan *double authentication*, *double scrollbar*, atau tombol *'Buka Tab'* yang membingungkan klien.

3. **Navigasi Kontekstual & Tombol 'Kembali':**
   * Tombol `Kembali` di top header hanya muncul saat dokumen dibuka (`docId !== 'doc-hub'`) dan otomatis tersembunyi saat berada di beranda Direktori Hub.
   * Berpindah antar tab dalam dokumen menggunakan fungsi `switchTab(tabId, btnEl)` yang telah diisolasi (*scoped*) per `.workspace-doc-view`:
   ```javascript
   function switchTab(tabId, btnEl) {
     const scope = btnEl ? (btnEl.closest('.workspace-doc-view') || document) : document;
     scope.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
     scope.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
     const activeP = document.getElementById(tabId);
     if (activeP) activeP.classList.add('active');
     if (btnEl) btnEl.classList.add('active');
   }
   ```

4. **Skalabilitas Masa Depan:**
   * Untuk menambah dokumen baru:
     1. Tambahkan kartu representasi pada grid Direktori Hub.
     2. Tambahkan satu blok `<div id="doc-[id]" class="workspace-doc-view">` di dalam `#workspace-viewport`.
     3. Sistem navigasi dan tab switcher akan langsung bekerja tanpa perlu refactoring arsitektur.

---

## 4. Keamanan & Otentikasi Sesi (Client-Side Gate)
* Otentikasi menggunakan verifikasi hash **SHA-256** pada sisi klien tanpa mengekspos PIN/kunci akses plaintext dalam kode sumber:
  ```javascript
  const VALID_HASHES = new Set([
    "84e41149998523aa38383223198cadc2d45b15a0d066b7edbece63afaf588229", // MantapKali
    "8db2e41ba3e1df839256608708bf01a0bfd727de7d4d463f1eae486c8e573c1f"  // mantapkali
  ]);
  ```
* Sesi disimpan di `sessionStorage.getItem("utary_auth") === "granted"`.

---
*Telkraf.id — Built for Precision, Aesthetics, and Scalability.*
