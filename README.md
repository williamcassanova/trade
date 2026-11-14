# MEMECOIN AI HQ — PREFLIGHT & CLARIFICATION DOCUMENT

**Versi:** 0.1 (Draft untuk Persetujuan)
**Status:** Menunggu konfirmasi sebelum Fase 1
**Referensi:** Master Specification & Development Blueprint v1.0
**Tanggal:** 2026-11-14
**Disusun oleh:** OpenCode (Senior Architect Mode)

---

## 1. TUJUAN DOKUMEN

Dokumen ini menjadi jembatan antara Master Specification v1.0 dan implementasi Fase 1. Isinya:

1. Ringkasan pemahaman inti spesifikasi.
2. Poin klarifikasi yang wajib disepakati sebelum coding.
3. Rekomendasi tech stack final.
4. Rencana struktur project.
5. Risk register awal.
6. Definition of Done (DoD) per fase.
7. Komitmen kerja OpenCode.

Dokumen ini **bukan** pengganti Master Specification. Jika ada konflik, Master Specification yang menang.

---

## 2. RINGKASAN PEMAHAMAN INTI

**Produk:** Sistem monitoring pasar crypto + simulasi trading virtual berbasis multi-agent AI, berjalan mandiri di Windows RDP dan Android Termux sebagai dua deployment terpisah.

**Batasan keras (non-negotiable):**
- Tidak ada eksekusi transaksi nyata.
- Tidak ada wallet signing atau penyimpanan private key.
- Tidak ada koneksi wallet write-access.
- Hanya read-only blockchain data + simulasi virtual.
- Modal virtual $100, risk 1% per simulasi ($1).

**Pipeline utama:**
Market Data → Normalisasi → Cache/DB → Scanner → Change Detector → Scoring → Security/Anomaly → Queue → 7 AI Agents → Discussion Room → Skeptic Review → Risk Calculator → Virtual Decision → Diary → Dashboard/Telegram.

**Prinsip arsitektur kunci:**
- Rule Engine = sumber kebenaran numerik. AI tidak boleh menimpanya.
- Data stale ≠ data terbaru.
- Setiap analisis wajib punya bukti + timestamp + limitations.
- Setiap fase harus lulus QA sebelum lanjut.
- Secret hanya di `.env` backend, tidak pernah di log atau frontend.

---

## 3. POIN KLARIFIKASI YANG WAJIB DISEPAKATI

### A. Prioritas Deployment

| Item | Usulan | Alasan |
|---|---|---|
| Deployment utama Fase 1–11 | **Windows RDP** | Dev tooling lengkap, debugging mudah |
| Deployment validasi | **Android Termux** (Fase 12) | Sesuai jadwal di spec |
| Node.js minimum | **Node.js 20 LTS** | Stabil di Termux ARM64 dan Windows |
| Node.js maksimum diuji | **22 LTS** | Untuk kompatibilitas forward |

**Pertanyaan:** Setuju dengan pembagian ini?

---

### B. Provider AI — Strategi Awal

| Item | Usulan |
|---|---|
| Pola integrasi | Provider Adapter generik + 1 provider konkret default |
| Provider default | OpenRouter (free tier) atau Google AI Studio free tier |
| Kilo | Adapter opsional, diaktifkan via `.env` |
| Testing Fase 6 | Mock provider wajib agar test tidak memakai kuota nyata |
| Failover final | Local Rule Engine (deterministik, tanpa API) |

**Pertanyaan:**
1. Setuju mulai dengan adapter generik + 1 provider konkret?
2. Provider default mana yang Anda miliki API key-nya?
3. Jika belum ada key, Fase 6 diuji penuh dengan mock — setuju?

---

### C. Market Data — Rate Limit Realistis

| Parameter | Usulan default |
|---|---|
| Trending polling | 60–90 detik |
| Watchlist polling | 120–180 detik |
| Backoff 429 | Eksponensial (1s → 2s → 4s → … max 60s) |
| Hard cap token | 100 = target kapasitas, boleh turun adaptif |
| Minimum operasional | 40 token (batas bawah agar tetap berguna) |
| Multi-source consensus | Harga/volume dari ≥2 sumber, selisih >5% → flag WARNING |

**Pertanyaan:**
- Apakah "100 token" bersifat hard cap absolute atau target kapasitas adaptif?
- Usulan: target adaptif — sistem otomatis turun ke 80/60/40 jika rate limit terdeteksi, dan menaikkan kembali saat sehat. Setuju?

---

### D. Frontend Stack

| Komponen | Usulan | Alasan |
|---|---|---|
| Build tool | Vite | Ringan, cepat, cocok Termux |
| Framework | React 18 + TypeScript (strict) | Sesuai spec |
| Styling | TailwindCSS | Konsisten dengan dark mode terminal |
| Komponen | shadcn/ui | Copy-in, tidak bloated |
| Grafik | lightweight-charts (TradingView) | Sesuai spec |
| State | Zustand + TanStack Query | Ringan |
| Routing | React Router v6 | Standar |

**Pertanyaan:** Setuju? Ada preferensi lain (misal Mantine, MUI, Ant Design)?

---

### E. Autentikasi

| Item | Usulan |
|---|---|
| Model | Single-user auth (bukan multi-tenant) |
| Password storage | argon2id |
| Session | JWT di httpOnly cookie (SameSite=Strict) |
| Session TTL | 7 hari (dapat diubah di `.env`) |
| Rate limit login | 5 percobaan / 15 menit / IP |
| Logout | Single + logout-all-sessions |
| 2FA | Tidak di v1 (bisa ditambah nanti) |

**Pertanyaan:** Setuju single-user tanpa 2FA di v1?

---

### F. Telegram Onboarding

**Opsi 1 — Manual (aman, eksplisit):**
- Anda mengisi `TELEGRAM_ALLOWED_USER_IDS` di `.env` sebelum bot jalan.

**Opsi 2 — Onboarding pertama (nyaman, perlu pengamanan):**
- `/start` pertama dari user mana pun → bot mencatat ID itu sebagai owner dan menolak ID lain.
- Risiko: jika token bocor sebelum onboarding, orang lain bisa jadi "owner".

**Opsi 3 — Hybrid (rekomendasi):**
- Jika `TELEGRAM_ALLOWED_USER_IDS` kosong → onboarding aktif, tapi bot hanya menerima onboarding dalam 10 menit setelah startup pertama dan mengirim log WARNING ke console.
- Jika terisi → onboarding nonaktif, hanya ID di `.env` yang dilayani.

**Pertanyaan:** Pilih Opsi 1, 2, atau 3?

---

### G. Cakupan Fase 1 (Fondasi Project)

**In scope Fase 1:**
1. Struktur monorepo: `backend/`, `frontend/`, `shared/`, `reports/`, `data/`, `scripts/`.
2. `.env.example` + validator env (zod) dengan fail-fast.
3. Inisialisasi SQLite + migration system versioned (Drizzle).
4. Structured logger (pino) dengan redaksi secret.
5. Endpoint `/health` dan `/api/version`.
6. Test harness (vitest) — 1 smoke test backend + 1 smoke test frontend build.
7. Script `dev` dan `build` untuk backend + frontend.
8. `README.md` setup minimal.
9. Laporan `reports/fase-01-fondasi.md`.

**Out of scope Fase 1:**
- Scanner, provider market, agent, discussion room, portfolio, dashboard halaman, Telegram, backtesting, replay.

**Pertanyaan:** Setuju cakupan ini?

---

## 4. REKOMENDASI TECH STACK FINAL

| Layer | Pilihan | Alasan |
|---|---|---|
| Runtime | Node.js 20 LTS | Stabil di Termux ARM64 & Windows |
| Bahasa | TypeScript (strict) | Sesuai spec |
| Backend framework | Fastify | Ringan, cepat, schema validation bawaan, cocok Termux |
| ORM/DB | better-sqlite3 + Drizzle ORM | Sync API cepat, migrasi versioned, ringan di Termux |
| Frontend | Vite + React + TypeScript + Tailwind + shadcn/ui | Ringan, build statis |
| Grafik | lightweight-charts | Sesuai spec |
| Realtime | ws (WebSocket native) | Simple, tanpa overhead socket.io |
| Auth | argon2 + JWT (httpOnly cookie) | Aman, ringan |
| Logger | pino | Structured, cepat, redaksi bawaan |
| Validasi | zod | Shared schema backend/frontend |
| Test | vitest + supertest | Unit + integration |
| Process mgmt | pm2 (RDP) / termux-services atau nohup (Termux) | Sesuai platform |
| Telegram | grammY | Modern, TS-friendly |

**Pertanyaan:** Tech stack final disetujui? Ada penggantian?

---

## 5. RENCANA STRUKTUR PROJECT

```
memecoin-ai-hq/
├── .env.example
├── .gitignore
├── package.json                # root (workspaces)
├── pnpm-workspace.yaml         # atau npm workspaces
├── README.md
├── docs/
│   ├── master-spec.md          # salinan spec v1.0
│   └── preflight.md            # dokumen ini
├── reports/
│   ├── fase-00-preflight.md
│   ├── fase-01-fondasi.md
│   └── ... (satu per fase)
├── data/                       # SQLite, backup, log rotasi
│   └── .gitkeep
├── scripts/
│   ├── dev.sh
│   ├── build.sh
│   ├── migrate.ts
│   └── backup-db.sh
├── shared/
│   ├── package.json
│   └── src/
│       ├── schemas/
│       └── types/
├── backend/
│   ├── package.json
│   ├── tsconfig.json
│   ├── vitest.config.ts
│   └── src/
│       ├── index.ts
│       ├── app.ts
│       ├── config/
│       │   ├── env.ts
│       │   └── constants.ts
│       ├── db/
│       │   ├── client.ts
│       │   ├── migrations/
│       │   └── schema/
│       ├── logging/
│       │   └── logger.ts
│       ├── routes/
│       │   ├── health.ts
│       │   └── version.ts
│       ├── plugins/
│       └── utils/
└── frontend/
    ├── package.json
    ├── tsconfig.json
    ├── vite.config.ts
    ├── tailwind.config.ts
    ├── index.html
    └── src/
        ├── main.tsx
        ├── App.tsx
        ├── pages/
        ├── components/
        └── lib/
```

**Pertanyaan:** Struktur monorepo disetujui?

---

## 6. RISK REGISTER AWAL

| ID | Risiko | Dampak | Mitigasi |
|---|---|---|---|
| R1 | Rate limit DexScreener/GeckoTerminal memaksa turun di bawah 40 token | Tinggi | Adaptive polling + multi-source + cache agresif + fitur "kapasitas efektif" di dashboard |
| R2 | Provider AI gratis habis kuota / mati | Sedang | Failover ke Local Rule Engine + cache analisis + notifikasi WARNING |
| R3 | Termux tidak kompatibel dengan better-sqlite3 native build | Tinggi | Uji build di Termux sejak Fase 1 (bukan Fase 12), siapkan fallback node:sqlite (Node 22) |
| R4 | Skeptic veto berlebihan → sistem tidak pernah SIMULATE | Sedang | Konfigurasi threshold veto di settings + audit log + metrik "veto rate" |
| R5 | Wash trading detector false positive | Sedang | Butuh ≥2 indikator sebelum status CRITICAL + batas data limitation |
| R6 | Kebocoran secret via log | Tinggi | Pino redact paths wajib + audit test di Fase 12 |
| R7 | SQLite corruption saat graceful shutdown gagal | Tinggi | WAL mode + backup otomatis harian + migration versioned |
| R8 | Backtest look-ahead bias | Tinggi | Test khusus di Fase 11: validasi bahwa hanya data ≤ timestamp keputusan yang dipakai |
| R9 | Onboarding Telegram dibajak | Sedang | Opsi F (hybrid) + window 10 menit + log WARNING |
| R10 | Skor tinggi tapi token kritis tetap lolos analisis | Sedang | Hard-stop CRITICAL di Security Layer, terpisah dari skor |

---

## 7. DEFINITION OF DONE (DoD) PER FASE

Setiap fase dianggap selesai hanya jika memenuhi semua:

1. Semua fitur fase berjalan dan didemokan (via test atau manual run).
2. Unit test + integration test untuk fitur baru lulus (0 gagal tak terjelaskan).
3. Regression test fase sebelumnya lulus.
4. Tidak ada secret di log (diverifikasi manual + test redaksi).
5. Dokumentasi (`README` atau `docs/`) diperbarui.
6. `reports/fase-XX-nama.md` dibuat dengan format di Section 8.
7. Backup DB dan config dilakukan sebelum perubahan destruktif.
8. Tidak ada `TODO`/`FIXME` kritis tersisa di kode fase tersebut.

**Gate:** Fase N+1 tidak boleh dimulai jika Fase N belum memenuhi DoD.

---

## 8. FORMAT LAPORAN FASE

Setiap `reports/fase-XX-nama.md` berisi:

```
# Fase XX — <Nama>

## Status
<DONE / BLOCKED / PARTIAL>

## Fitur Selesai
- ...

## File Dibuat / Diubah
- path — deskripsi singkat

## Pengujian Dijalankan
| Test | Perintah | Hasil |
|---|---|---|

## Jumlah Test
- Lulus: N
- Gagal: M
- Skipped: K

## Masalah Ditemukan
- ...

## Perbaikan Dilakukan
- ...

## Risiko Tersisa
- ...

## Status Akhir
- ...

## Rekomendasi Fase Berikutnya
- ...
```

---

## 9. KOMITMEN KERJA OPENCODE

OpenCode **akan:**
1. Memeriksa seluruh project sebelum mengubah file.
2. Bekerja satu fase per iterasi, dengan gate QA.
3. Menjalankan test setelah setiap perubahan besar.
4. Menyimpan laporan detail di `reports/`.
5. Memperbarui dokumentasi setiap fase.
6. Memakai `.env` untuk semua konfigurasi.
7. Menjaga log tetap bersih dari secret.
8. Menanyakan klarifikasi jika spesifikasi ambigu.

OpenCode **tidak akan:**
1. Membuat ulang project jika sudah ada.
2. Menghapus fitur yang berjalan.
3. Mengklaim fitur selesai tanpa test nyata.
4. Mengarang hasil pengujian.
5. Menambahkan wallet signing, private key, atau transaksi nyata.
6. Melanjutkan fase jika fase sebelumnya gagal.
7. Menampilkan secret di log, report, atau output terminal.

---

## 10. YANG PERLU ANDA KONFIRMASI

Silakan jawab poin berikut agar eksekusi bisa dimulai:

| # | Pertanyaan | Usulan Default |
|---|---|---|
| 1 | Target deployment utama Fase 1–11 = Windows RDP? | Ya |
| 2 | Node.js versi minimum = 20 LTS? | Ya |
| 3 | Provider AI default | Sebutkan provider yang Anda punya key-nya, atau "mock dulu" |
| 4 | "100 token" = target adaptif, bukan hard cap? | Ya, adaptif |
| 5 | Frontend stack (Vite + React + TS + Tailwind + shadcn/ui) | Setuju? |
| 6 | Auth = single-user + argon2 + JWT cookie, tanpa 2FA di v1 | Setuju? |
| 7 | Telegram onboarding | Opsi 1 / 2 / 3 (rekomendasi: 3) |
| 8 | Cakupan Fase 1 sesuai Section 3.G | Setuju? |
| 9 | Tech stack final (Section 4) | Setuju? |
| 10 | Struktur monorepo (Section 5) | Setuju? |
| 11 | Repo status: greenfield atau sudah ada folder? | Sebutkan |
| 12 | Tambahan lain yang ingin Anda tetapkan sebelum coding? | — |

---

## 11. LANGKAH SETELAH PERSETUJUAN

1. Saya buat `reports/fase-00-preflight.md` (salinan dokumen ini, status APPROVED).
2. Saya mulai Fase 1 — Fondasi Project:
   - Scaffold monorepo.
   - Env validator.
   - SQLite + migrasi.
   - Logger dengan redaksi.
   - Endpoint `/health` + `/api/version`.
   - Smoke test.
3. Saya kirim `reports/fase-01-fondasi.md` dengan hasil test nyata.
4. Menunggu approval Anda → lanjut Fase 2.

---

**Akhir dokumen. Menunggu konfirmasi untuk memulai Fase 1.**
