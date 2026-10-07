# Checklist Implementasi — Audit 08 (FINAL)

> Sumber: `08-AUDIT-KRITIS-PEDAS-DAN-RENCANA-PERBAIKAN.md` §11 (Revisi 2, 6 Okt 2026).  
> Status diperbarui saat setiap item selesai. Jangan hapus item — centang saja.  
> Last updated: 7 Oktober 2026

---

## Sprint 0 — Keputusan Strategis

- [ ] **S0-1** Pilih Opsi A/B/C (`§7`) dan tulis sebagai **satu paragraf keputusan + alasan + apa yang berhenti dikerjakan** di `docs/RENCANA.md` (bukan wacana, tapi keputusan yang bisa dicetak di onboarding).
- [ ] **S0-2** Rewrite status tabel lama di `docs/RENCANA.md` (baseline 13 Sept) — addendum ngaku tidak menggambarkan branch terbaru; jangan nambah di belakang, ganti tabelnya.
- [ ] **S0-3** Verifikasi lisensi `wokwi-arduino` npm package jika Opsi C dipilih — cek apakah feasible di level engine sebelum dijadikan keputusan.
- **Acceptance:** `docs/RENCANA.md` berisi keputusan yang bisa langsung dibaca onboarding kontributor baru.

---

## Sprint 1 — Guardrail & Quick Win

### 1a. README Frontend
- [ ] **S1-1** Rewrite `frontend/README.md`: paragraf apa ini, cara jalan (`npm run dev`, butuh backend Go `:8080` atau bisa tanpa?), link ke `docs/INVENTARIS.md` + disclaimer ideal, cara reproduksi test (`npm test`, `test:kinematics-reference`), batas mobile.
- [ ] **S1-2** Pastikan kontributor baru bisa setup <10 menit tanpa tanya — verifikasi dengan langkah-langkah di README.
- **Acceptance:** README tidak ada satu baris template `create-next-app` lagi; ada seksi Prerequisites, Quick Start, Testing, dan Known Limitations.

### 1b. Fidelity Katalog Komponen
- [ ] **S1-3** Tambah field `fidelity: "ideal" | "quasi-static" | "datasheet"` dan `experimental: true` di `ComponentRegistry` (`frontend/src/lib/components/ComponentRegistry.ts`).
- [ ] **S1-4** Tampilkan label `Edukasi-Ideal` / `Tervalidasi-Datasheet` / `Kuasi-Statik` di card komponen di UI (minimal tooltip atau badge).
- [ ] **S1-5** Sembunyikan komponen non-P0 (di luar 10 tipe: `arduino_uno, breadboard, led_red, resistor_220, push_button, potentiometer, dht11, hcsr04, servo_sg90, jumper_*`) di balik flag `NEXT_PUBLIC_SHOW_EXPERIMENTAL=true` — **flag, bukan delete**, reversible.
- [ ] **S1-6** Update `docs/INVENTARIS.md`: tambah kolom eksplisit `Tingkat Fidelity` — kontennya sudah ada di kolom "Batas yang masih ada", sinkronkan saja.
- **Acceptance:** 10 tipe katalog default semua punya label fidelity; komponen non-P0 tersembunyi tanpa flag; tidak ada nama IC komersial tanpa disclaimer di card.

### 1c. CI GitHub Actions (dari nol — tidak ada `.github/` sekarang)
- [ ] **S1-7** Buat `.github/workflows/ci.yml` dengan pipeline: `npm ci` → `npx tsc --noEmit` → `npx eslint --max-warnings 0` → `npm test` → `next build`.
- [ ] **S1-8** CI harus jalan di Ubuntu latest, node version sesuai `package.json` engines atau LTS terkini.
- [ ] **S1-9** Pastikan build Next (Turbopack/Webpack) berhasil di CI — sesuaikan flag build jika perlu berdasarkan catatan `RENCANA.md` (Turbopack gagal di sandbox child process port, gunakan Webpack jika ada masalah).
- **Acceptance:** CI hijau di branch; badge CI bisa ditambahkan ke README.

### 1d. Bundle Analyzer Baseline
- [ ] **S1-10** Install `@next/bundle-analyzer` (jika belum ada di `package.json`).
- [ ] **S1-11** Tambah script `npm run analyze` di `package.json` yang menjalankan build dengan analyzer aktif.
- [ ] **S1-12** Jalankan analyzer, catat angka JS/WASM first-load (total bytes) di `docs/BUNDLE-BASELINE.md`.
- [ ] **S1-13** Identifikasi top-5 kontributor terbesar bundle untuk menjadi target Sprint 3.
- **Acceptance:** `docs/BUNDLE-BASELINE.md` ter-commit dengan angka konkret; target "-30%" Sprint 3 punya baseline yang sah.

---

## Sprint 2 — Test & UX

### 2a. Vitest untuk lib murni
- [ ] **S2-1** Install `vitest` sebagai dev dependency.
- [ ] **S2-2** Buat `frontend/vitest.config.ts` dengan setup minimal.
- [ ] **S2-3** Migrasi / tambah test untuk modul lib murni: `CircuitSolver`, `ArduinoInterpreter`/`SketchRuntime`, kinematika (`FK/IK/Jacobian`), `vhal/*`.
- [ ] **S2-4** `run.cjs` (1779 baris) dipertahankan sebagai legacy logic gate — jalankan keduanya di CI (`npm test` tetap jalan `run.cjs`).
- **Acceptance:** `npm run test:unit` hijau; modul lib punya coverage test yang bisa dijalankan terpisah dari `run.cjs`.

### 2b. Playwright E2E (5 alur wajib)
- [ ] **S2-5** Install `@playwright/test` dan init config minimal.
- [ ] **S2-6** **E2E-1:** Load example Blink → Run → serial monitor menampilkan ON/OFF (atau output yang relevan).
- [ ] **S2-7** **E2E-2:** Wiring kabel ke pin tidak valid ditolak (error message muncul).
- [ ] **S2-8** **E2E-3:** Save project ke JSON → reload halaman → load JSON → konten sama (roundtrip).
- [ ] **S2-9** **E2E-4:** Buka Flash Panel tanpa serial device terhubung → tidak crash, tampilkan pesan informatif.
- [ ] **S2-10** **E2E-5:** Tambahan pilihan bebas (misal: ganti bahasa/tema, atau alur Inspector komponen servo).
- [ ] **S2-11** E2E masuk CI (jalankan setelah `next build` atau dev server dalam CI).
- **Acceptance:** 5 E2E hijau di CI; "Browser E2E belum terverifikasi" di `docs/README.md` tidak berlaku lagi.

### 2c. Fuzz ringan contract test
- [ ] **S2-12** Tambah fuzz ringan di Vitest untuk `parseProject` dengan input: pin MISSING, wire duplikat, komponen ID tidak dikenal.
- [ ] **S2-13** Tambah fuzz ringan untuk `validWire` / `buildElectricalConnectivity` — material sudah ada di `run.cjs:592-606`, tinggal otomasi.
- [ ] **S2-14** Fuzz test masuk CI.
- **Acceptance:** edge case pin MISSING dan wire duplikat di-test otomatis, tidak hanya di `run.cjs` manual.

### 2d. Inspector Lazy
- [ ] **S2-15** Ganti mount tanpa kondisi `<OnnxInferencePanel />` (`page.tsx:1199`) dan `<CloudAIGatewayPanel />` (`page.tsx:909`) dengan `React.lazy` + mounting kondisional.
- [ ] **S2-16** Semua panel inspector (10+ panel kondisional `page.tsx:972-1022`) di-refactor ke `React.lazy` per `typeId` — unmount saat tidak aktif.
- [ ] **S2-17** Verifikasi dengan React DevTools: buka inspector Servo → ONNX/gateway tidak di-mount.
- **Acceptance:** tidak ada panel berat (ONNX, LiDAR, RGB-D) yang ter-mount saat tidak sedang dipakai.

### 2e. Shortcut Keyboard
- [ ] **S2-18** Pindah semua definisi shortcut dari `page.tsx:210-271` ke `frontend/src/hooks/useWorkspaceShortcuts.ts` (buat file baru).
- [ ] **S2-19** Tambahkan daftar shortcut ke Help modal (`page.tsx:546-596`) — tampil secara discoverably.
- [ ] **S2-20** Single-letter shortcut (`R`, `H`, `O`) hanya aktif saat canvas fokus (bukan saat editor Monaco atau input field fokus) — perkuat guard yang sudah ada di `:213-220`.
- **Acceptance:** tidak ada shortcut nyasar saat ngetik di Monaco; Help modal menampilkan semua shortcut.

### 2f. Backend Error UX
- [ ] **S2-21** Bedakan kondisi error backend: `offline` (tidak bisa konek sama sekali), `timeout 5s` (AbortSignal), `rejected 4xx/5xx` (respons error) — tiap kondisi punya pesan berbeda di UI.
- [ ] **S2-22** Tambah tombol **Retry** pada error koneksi backend.
- [ ] **S2-23** Dokumentasikan 2 mode operasi di `frontend/README.md` dan di UI (banner atau footer): `frontend-only (localStorage)` vs `dengan backend Go :8080`.
- **Acceptance:** user yang jalankan tanpa backend Go tahu itu normal dan bisa lanjut pakai localStorage.

---

## Sprint 3 — Split God-File & Bundle Optimization

### 3a. Split `page.tsx` (1210 baris → modular)
- [ ] **S3-1** Buat `frontend/src/app/WorkspacePage.tsx` — orkestrator utama (import semua panel, render layout).
- [ ] **S3-2** Ekstrak `LibraryPanel.tsx` — sidebar katalog komponen.
- [ ] **S3-3** Ekstrak `WiringPanel.tsx` — panel wiring/koneksi.
- [ ] **S3-4** Ekstrak `InspectorPanel.tsx` — panel inspector properti komponen.
- [ ] **S3-5** Ekstrak `SceneSection.tsx` — area canvas 3D (R3F/Three.js).
- [ ] **S3-6** Pastikan `useWorkspaceShortcuts.ts` (dari S2-18) sudah dipakai di `WorkspacePage.tsx`.
- [ ] **S3-7** **Disiplin commit:** satu ekstrak per commit, test hijau di setiap commit — jangan big-bang.
- **Acceptance:** `page.tsx` < 200 baris (hanya re-export/setup); tidak ada file baru yang masuk >500 baris.

### 3b. Split `simulation.worker.ts` (1627 baris → modular)
- [ ] **S3-8** Buat `frontend/src/lib/simulation/steps/circuitStep.ts`.
- [ ] **S3-9** Buat `frontend/src/lib/simulation/steps/motorStep.ts`.
- [ ] **S3-10** Buat `frontend/src/lib/simulation/steps/batteryPowerStep.ts`.
- [ ] **S3-11** Buat `frontend/src/lib/simulation/steps/i2cSensorStep.ts`.
- [ ] **S3-12** Buat `frontend/src/lib/simulation/steps/roboticsStep.ts`.
- [ ] **S3-13** `simulation.worker.ts` jadi dispatcher: impor modul step, dispatch berdasarkan tipe pesan.
- [ ] **S3-14** Setiap step file harus punya test sendiri (integrasi dengan `vitest` dari Sprint 2).
- **Acceptance:** worker tinggal dispatcher; setiap step bisa di-test mandiri; 95 checks + E2E tetap hijau.

### 3c. Split `useSimulatorStore.ts` (471 baris → slice)
- [ ] **S3-15** Buat `useWorkspaceStore.ts` — state workspace (komponen, posisi, seleksi).
- [ ] **S3-16** Buat `useWiringStore.ts` — state kabel/koneksi.
- [ ] **S3-17** Buat `useEditorStore.ts` — state editor kode.
- [ ] **S3-18** Buat `useSimRuntimeStore.ts` — state runtime simulasi (clock, status, pause/play).
- [ ] **S3-19** Buat `useSensorFrameStore.ts` — state frame sensor (`rgbdFrames`, `lidarFrames`, `imuFrames`).
- [ ] **S3-20** Buat `frontend/src/store/index.ts` untuk re-export semua store.
- **Acceptance:** tidak ada store tunggal >200 baris; setiap slice bisa diimpor independen.

### 3d. Split `CodeEditorPanel.tsx` (1240 baris → modular)
- [ ] **S3-21** Ekstrak `CodeEditorShell.tsx` — layout utama panel editor.
- [ ] **S3-22** Ekstrak `LibraryCatalog.ts` — katalog library Arduino yang bisa diimpor.
- [ ] **S3-23** Ekstrak `HardwareFlashPanel.tsx` — sub-panel flash ke hardware fisik (esptool-js).
- [ ] **S3-24** Ekstrak `MonacoWrapper.tsx` — wrapper tipis `@monaco-editor/react` dengan konfigurasi default.
- **Acceptance:** `CodeEditorPanel.tsx` atau `CodeEditorShell.tsx` <500 baris; flash dan Monaco bisa di-test/di-render mandiri.

### 3e. Bundle Lazy Loading
- [ ] **S3-25** Lazy-load `pyodide` — hanya di-import saat Python execution diminta (dynamic import + loading state/spinner).
- [ ] **S3-26** Lazy-load `onnxruntime-web` — hanya saat `OnnxInferencePanel` aktif.
- [ ] **S3-27** Lazy-load `@mediapipe/tasks-vision` — hanya saat fitur MediaPipe diaktifkan.
- [ ] **S3-28** Lazy-load `@dimforge/rapier3d-compat` — hanya saat scene punya physics body.
- [ ] **S3-29** Lazy-load `@monaco-editor/react` — sudah via `MonacoWrapper`, pastikan dynamic import.
- [ ] **S3-30** Pisah route: `/` (rangkaian dasar, bundle ringan) vs `/lab` (robotik+AI, bundle berat) di Next.js App Router.
- **Acceptance:** Lighthouse mobile tidak merah; first-load JS/WASM lebih kecil ≥20% dari baseline di `docs/BUNDLE-BASELINE.md`; halaman blink bisa dibuka tanpa download WASM fisika/AI.

### 3f. Validasi Akhir Sprint 3
- [ ] **S3-31** `npx tsc --noEmit` clean (exit 0).
- [ ] **S3-32** `npx eslint --max-warnings 0` clean.
- [ ] **S3-33** `npm test` (run.cjs) → 95+ checks hijau.
- [ ] **S3-34** `npm run test:unit` (vitest) hijau.
- [ ] **S3-35** 5 E2E Playwright hijau.
- [ ] **S3-36** `next build` berhasil.
- [ ] **S3-37** Catat delta bundle vs baseline di `docs/BUNDLE-BASELINE.md`.
- [ ] **S3-38** `git status` clean: tidak ada build artifact, folder `https:`, atau file generated yang ter-commit.
- **Acceptance:** semua validation gate dari §11 terpenuhi.

---

## Item P1 Tambahan (Dokumentasi Teknis)

- [ ] **D1** Tiap model V-HAL punya `ASUMSI.md` 5 baris: rumus, parameter default, sumber datasheet (jika ada), apa yang diabaikan. Minimal untuk: `a4988`, `l298n`, `dc_motor`, `battery`, `charger_thermal`, `dc_dc`, `ssd1306`.
- [ ] **D2** Tambah seksi di UI atau docs: komponen mana yang `Edukasi-Ideal` vs `Tervalidasi-Datasheet` — link dari card komponen ke definisi di `docs/INVENTARIS.md`.
- [ ] **D3** Dokumentasikan apa yang TIDAK dimodelkan secara eksplisit di panel simulator (bukan hanya di docs): integer division C++, overflow, cycle-accurate ISR, SPI, UART timing, switching PWM, winding R-L cycle.

---

## Summary Metrics (isi saat selesai)

| Metrik | Sebelum | Sesudah Target | Sesudah Aktual |
|--------|---------|----------------|----------------|
| `page.tsx` baris | 1210 | <200 | — |
| `simulation.worker.ts` baris | 1627 | <300 (dispatcher) | — |
| `useSimulatorStore.ts` baris | 471 | <200/slice | — |
| `CodeEditorPanel.tsx` baris | 1240 | <500 | — |
| `run.cjs` checks | 95 | 95+ | — |
| E2E tests | 0 | 5 | — |
| CI pipeline | tidak ada | hijau | — |
| First-load JS/WASM | ? (ukur S1) | -30% | — |
| Komponen tanpa label fidelity | banyak | 0 | — |

---

*Checklist ini adalah satu-satunya sumber kebenaran status implementasi audit. Update segera saat item selesai.*
