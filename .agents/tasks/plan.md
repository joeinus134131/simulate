# Implementation Plan — Audit 08 ArduSim (FINAL)

> Dibuat: 7 Oktober 2026  
> Sumber: bedah langsung codebase + `docs/08-AUDIT-KRITIS-PEDAS-DAN-RENCANA-PERBAIKAN.md` §11  
> Prinsip: **guardrail sebelum refactoring** — CI + test dulu, split god-file belakangan  
> Semua path relatif terhadap `/Users/user/Projects/simulate/`

---

## Temuan Kritis dari Inspeksi Langsung

| File | Baris Aktual | Target Audit | Catatan |
|------|-------------|--------------|---------|
| `frontend/src/app/page.tsx` | **1210** | <200 (setelah split) | Dikonfirmasi `wc -l` |
| `frontend/src/lib/simulation/simulation.worker.ts` | **1627** | <300 (dispatcher) | Dikonfirmasi `wc -l` |
| `frontend/src/store/useSimulatorStore.ts` | **471** | <200/slice | Dikonfirmasi `wc -l` |
| `frontend/src/components/panels/CodeEditorPanel.tsx` | **1240** | <500 | Dikonfirmasi `wc -l` |
| `frontend/src/lib/components/ComponentRegistry.ts` | **690** | N/A | 34+ tipe terdaftar |
| `frontend/tests/run.cjs` | **1779** | Dipertahankan | 95 checks confirmed |
| CI pipeline | **TIDAK ADA** | hijau | Greenfield — tidak ada `.github/` |
| `@next/bundle-analyzer` | **TIDAK ADA** | installed | Tidak ada di `package.json` devDeps |
| `fidelity` field di ComponentRegistration | **TIDAK ADA** | perlu ditambah | Interface tidak punya field ini |
| `hooks/` directory | **TIDAK ADA** | perlu dibuat | Shortcut masih inline di page.tsx:210-271 |
| `OnnxInferencePanel` mount | **page.tsx:1199 tanpa kondisi** | conditional | Audit §5 dikonfirmasi |
| Backend error UX | **hanya notice** | 3 kondisi berbeda | fetch :315-335 dengan AbortSignal.timeout(5000) |
| `frontend/README.md` | **47 baris template** | rewrite total | Baris 1-36 murni create-next-app template |

---

## Urutan Eksekusi WAJIB (Dependency Chain)

```
Sprint 0 (keputusan) → Sprint 1 (guardrail: CI + labels + baseline) 
→ Sprint 2 (test harness: vitest + E2E) 
→ Sprint 3 (split god-files + bundle) 
→ D1/D2/D3 (docs)
```

**JANGAN** memulai Sprint 3 sebelum Sprint 2 selesai. Split god-file tanpa E2E = medan ranjau.

---

## Sprint 0 — Keputusan Strategis

- [ ] **S0-1** Tulis keputusan strategis di `docs/RENCANA.md`.
  - **File:** `docs/RENCANA.md`
  - **Apa:** Tambah seksi `## Keputusan Strategis (Sprint 0)` dengan pilihan **Opsi A** (fokus alat ajar breadboard/blink polished; bekukan robotik/AI sebagai eksperimental; menang di kurikulum Indonesia). Tulis 1 paragraf: keputusan + alasan + apa yang berhenti dikerjakan.
  - **Verify:** `cat docs/RENCANA.md | grep "Keputusan Strategis"` — exist. Review manual: tidak ada "wacana", ada keputusan konkret.

- [ ] **S0-2** Ganti tabel status lama di `docs/RENCANA.md`.
  - **File:** `docs/RENCANA.md`
  - **Apa:** Hapus tabel Tahap 1-6 baseline 13 September (baris setelah `## Tahap dan kriteria penerimaan`). Tambah tabel ringkas status aktual: Sprint 0/1/2/3 + D1/D2/D3 = pending/in-progress/done. Addendum di bawah sudah mengakui tabel lama tidak akurat — eksekusi perintah itu.
  - **Verify:** `cat docs/RENCANA.md` — tidak ada kolom "Tahap 2 Kontrak & backend" lama.

- [ ] **S0-3** Verifikasi lisensi `wokwi-arduino` npm.
  - **File:** `docs/RENCANA.md` (tambah catatan kaki)
  - **Apa:** Cek `https://www.npmjs.com/package/wokwi-arduino` untuk lisensi. Tulis 2-3 baris ringkasan di RENCANA.md: lisensi apa, apakah izinkan embedding, kesimpulan feasibility. Jangan install package.
  - **Verify:** Review manual section Opsi C di RENCANA.md berisi catatan lisensi.

---

## Sprint 1 — Guardrail & Quick Win

### 1a. README Frontend

- [ ] **S1-1** Rewrite `frontend/README.md` dari nol.
  - **File:** `frontend/README.md`
  - **Apa:** Hapus semua 47 baris. Tulis ulang dengan seksi wajib:
    - `## What is This` — 1 paragraf deskripsi proyek
    - `## Prerequisites` — Node 20+, Go (opsional), Python (opsional untuk kinematics reference)
    - `## Quick Start` — `npm run dev` tanpa backend; penjelasan backend Go :8080 opsional
    - `## Running Tests` — `npm test` (95 checks); `npx tsc --noEmit`; `npm run test:kinematics-reference` (opsional)
    - `## Known Limitations` — mobile warning by design; bukan AVR cycle-accurate; bundle berat
    - `## Architecture Overview` — link ke `docs/INVENTARIS.md` dan `docs/08-AUDIT-KRITIS-PEDAS-DAN-RENCANA-PERBAIKAN.md`
  - **Risiko:** rendah — file yang berdiri sendiri
  - **Verify:** `grep "create-next-app" frontend/README.md` — zero results. `grep "Prerequisites" frontend/README.md` — ada.

- [ ] **S1-2** Verifikasi Quick Start < 10 menit.
  - **File:** tidak ada (review)
  - **Apa:** Ikuti langkah README yang baru ditulis. Jika ada langkah yang membutuhkan klarifikasi, update README-nya.
  - **Verify:** Review manual — ada instruksi yang ambiguous? Perbaiki jika ya.

### 1b. Fidelity Katalog

- [ ] **S1-3** Tambah `fidelity` dan `experimental` ke `ComponentRegistration` interface.
  - **File:** `frontend/src/lib/components/ComponentRegistry.ts`
  - **Baris relevan:** Interface `ComponentRegistration` di sekitar baris 25-35. Setiap `ComponentRegistry.register({...})` call.
  - **Apa:** Tambah ke interface: `fidelity: 'ideal' | 'quasi-static' | 'datasheet'; experimental: boolean;`
  - Tandai setiap registrasi:
    - `datasheet` (non-experimental P0): `arduino_uno, breadboard, led_red, resistor_220, push_button, potentiometer, jumper_*` (5 warna)
    - `quasi-static` (non-experimental P0): `dht11, hcsr04, servo_sg90`
    - `quasi-static` (experimental): `dc_motor, stepper_nema17, incremental_encoder, a4988_stepper_driver, l298n_dual_hbridge, dc_dc_converter, battery_pack, battery_charger, capacitor_universal, esp32_wroom, plc_omron_cp1e`
    - `ideal` (experimental): `dc_supply, rgbd_camera, planar_lidar, imu_6axis, oled_ssd1306, lcd1602_i2c, edu_arm_3dof, aero_arm_6dof, rover_bot_4wd`
  - **Risiko:** rendah untuk logic; risiko TS error jika ada consumer yang buat `ComponentRegistration` object di file lain — cek `grep -r "ComponentRegistration" src/`
  - **Verify:** `npx tsc --noEmit` — exit 0. `npm test` — 95 checks.

- [ ] **S1-4** Tampilkan badge fidelity di component-card.
  - **File:** `frontend/src/app/page.tsx` (baris ~635-680, library panel rendering)
  - **Apa:** Di dalam `.map((c) => (<button key={c.typeId} className="component-card">...)`, tambah setelah `<strong>{c.name}</strong>`:
    ```tsx
    <span className={`fidelity-badge fidelity-${c.fidelity}`} title={
      c.fidelity === 'ideal' ? 'Model edukasi ideal — bukan ekuivalen hardware' :
      c.fidelity === 'quasi-static' ? 'Kuasi-statik — mendekati behavior hardware' :
      'Mengacu datasheet — parameter dari spesifikasi resmi'
    }>{c.fidelity === 'ideal' ? 'Ideal' : c.fidelity === 'quasi-static' ? 'Kuasi-Statik' : 'Datasheet'}</span>
    ```
  - Tambah CSS di `globals.css` untuk `.fidelity-badge`, `.fidelity-ideal` (amber), `.fidelity-quasi-static` (blue), `.fidelity-datasheet` (emerald).
  - **Risiko:** rendering di library list — pastikan badge tidak breaking layout dengan text-wrap atau overflow
  - **Verify:** `npm run build -- --webpack` berhasil. Manual review: badge terlihat di library panel.

- [ ] **S1-5** Sembunyikan non-P0 di balik flag `NEXT_PUBLIC_SHOW_EXPERIMENTAL`.
  - **File:** `frontend/src/app/page.tsx` (baris ~635), `frontend/.env.example`
  - **Apa:** Di filter library list (setelah filter `!c.typeId.startsWith("jumper_")`), tambah: `&& (!c.experimental || process.env.NEXT_PUBLIC_SHOW_EXPERIMENTAL === 'true')`. Tambah ke `.env.example`: `NEXT_PUBLIC_SHOW_EXPERIMENTAL=false # Set true untuk tampilkan komponen eksperimental`.
  - **Risiko:** rendah — ini pure filter; komponen tetap terdaftar di Registry, hanya tidak ditampilkan. Tidak ada perubahan simulation logic.
  - **Verify:** Build berhasil. Default build: hanya 10 tipe P0 muncul.

- [ ] **S1-6** Update `docs/INVENTARIS.md` kolom Tingkat Fidelity.
  - **File:** `docs/INVENTARIS.md`
  - **Apa:** Di tabel `## Audit Perupdate` (baris setelah "Kondisi baseline"), tambah kolom `| Tingkat Fidelity |` setelah kolom `Tipe`. Isi sesuai mapping di S1-3. Tidak perlu rewrite konten lain.
  - **Verify:** `cat docs/INVENTARIS.md | grep "Tingkat Fidelity"` — exist.

### 1c. CI GitHub Actions

- [ ] **S1-7** Buat `.github/workflows/ci.yml` di root repo.
  - **File:** `/Users/user/Projects/simulate/.github/workflows/ci.yml`
  - **Apa:** Pipeline lengkap:
    ```yaml
    name: CI
    on: [push, pull_request]
    jobs:
      ci:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
            with:
              submodules: recursive
          - uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: 'npm'
              cache-dependency-path: frontend/package-lock.json
          - working-directory: frontend
            run: npm ci
          - working-directory: frontend
            run: npx tsc --noEmit
          - working-directory: frontend
            run: npx eslint --max-warnings 0
          - working-directory: frontend
            run: npm test
          - working-directory: frontend
            run: npm run build -- --webpack
            env:
              NEXT_PUBLIC_SHOW_EXPERIMENTAL: 'false'
    ```
  - **Baris kritis:** `submodules: recursive` (frontend adalah git submodule). Build dengan `--webpack` (Turbopack gagal di sandbox per RENCANA.md).
  - **Risiko:** copy-pyodide-assets.mjs dan copy-onnx-assets.mjs di predev/prebuild mungkin gagal di CI karena file WASM tidak ada. Solusi: tambah guard di copy scripts (`if (!fs.existsSync(src)) { console.warn('skip'); return; }`) atau tambah `SKIP_ASSET_COPY=true` env di CI.
  - **Verify:** `python3 -c "import yaml, open('.github/workflows/ci.yml')"` — YAML valid. Manual review pipeline steps lengkap.

- [ ] **S1-8** Verifikasi CI bisa jalan di Ubuntu/Node 20.
  - **File:** `.github/workflows/ci.yml`
  - **Apa:** Cek `frontend/package.json` untuk `engines` field — tidak ada → gunakan Node 20 LTS. Pastikan tidak ada OS-specific path separator issue.
  - **Verify:** Baca setiap step di ci.yml — tidak ada `process.env.HOME` atau `/Users/` hardcoded.

- [ ] **S1-9** Pastikan build script pakai `--webpack`.
  - **File:** `frontend/package.json`
  - **Apa:** Ubah script `"build": "next build"` menjadi `"build": "next build --webpack"`. Ini membuat `npm run build` secara default pakai Webpack, aman untuk CI dan sandbox.
  - **Alasan:** RENCANA.md: "Build default Turbopack gagal di sandbox saat child process mencoba membuka port (Operation not permitted)". Webpack berhasil.
  - **Risiko:** Developer lokal yang rely on Turbopack speed — minimal impact, mereka bisa run `next dev` yang sudah pakai --webpack.
  - **Verify:** `npm run build` dari frontend/ berhasil exit 0.

### 1d. Bundle Analyzer Baseline

- [ ] **S1-10** Install `@next/bundle-analyzer`.
  - **File:** `frontend/package.json`
  - **Apa:** `npm install --save-dev @next/bundle-analyzer@16.2.6` (pinned sesuai Next.js version). Tambah ke devDependencies.
  - **Verify:** `cat frontend/package.json | grep bundle-analyzer` — ada.

- [ ] **S1-11** Tambah script `analyze` dan config di `next.config.ts`.
  - **File:** `frontend/package.json`, `frontend/next.config.ts`
  - **Apa di package.json:** Tambah `"analyze": "ANALYZE=true npm run build"` di scripts.
  - **Apa di next.config.ts:**
    ```ts
    import bundleAnalyzer from '@next/bundle-analyzer';
    const withBundleAnalyzer = bundleAnalyzer({ enabled: process.env.ANALYZE === 'true', openAnalyzer: false });
    const nextConfig: NextConfig = { distDir: process.env.ARDUSIM_BUILD_DIR || '.next' };
    export default withBundleAnalyzer(nextConfig);
    ```
  - **Verify:** `npx tsc --noEmit` tetap clean setelah perubahan next.config.ts.

- [ ] **S1-12** Jalankan analyzer, catat baseline.
  - **File:** `docs/BUNDLE-BASELINE.md` (buat baru)
  - **Apa:** `npm run analyze` dari frontend/. Catat output dari `.next/analyze/client.html` atau terminal. Jika analyzer HTML tidak bisa dibuka di sandbox, ambil angka dari `npm run build` terminal output (First Load JS per route).
  - **Format file:**
    ```md
    # Bundle Baseline — Sprint 1
    Date: [tanggal]
    Build: next build --webpack
    ## First Load JS (per route)
    | Route | Size | Notes |
    |-------|------|-------|
    | /     | X kB | main route |
    ## Top 5 Kontributor Terbesar
    1. pyodide — ~X MB
    2. onnxruntime-web — ~X MB
    ...
    ## Target Sprint 3
    - First Load JS: turun ≥20% dari baseline ini
    ```
  - **Verify:** File exist di `docs/BUNDLE-BASELINE.md` dengan angka konkret.

- [ ] **S1-13** Identifikasi top-5 kontributor bundle.
  - **File:** `docs/BUNDLE-BASELINE.md` (update S1-12)
  - **Apa:** Berdasarkan package.json dependencies + analisis: pyodide, onnxruntime-web, @mediapipe/tasks-vision, @dimforge/rapier3d-compat, three/@react-three — semua adalah kandidat utama. Catat estimasi ukuran masing-masing.
  - **Verify:** BUNDLE-BASELINE.md punya seksi "Top 5 Kontributor".

---

## Sprint 2 — Test & UX

### 2a. Vitest Unit Tests

- [ ] **S2-1** Install `vitest`.
  - **File:** `frontend/package.json`
  - **Apa:** `npm install --save-dev vitest@latest` (pinned). Tambah script `"test:unit": "vitest run"`.
  - **Verify:** `cat frontend/package.json | grep vitest` — ada.

- [ ] **S2-2** Buat `frontend/vitest.config.ts`.
  - **File:** `frontend/vitest.config.ts`
  - **Apa:**
    ```ts
    import { defineConfig } from 'vitest/config';
    import path from 'path';
    export default defineConfig({
      test: {
        environment: 'node',
        include: ['src/**/*.test.ts', 'tests/unit/**/*.test.ts'],
        exclude: ['tests/run.cjs', 'tests/e2e/**'],
      },
      resolve: {
        alias: { '@': path.resolve(__dirname, './src') },
      },
    });
    ```
  - **Verify:** `npx tsc --noEmit` clean. `npm run test:unit` jalan (meski zero tests awalnya).

- [ ] **S2-3** Tambah unit tests untuk modul lib murni.
  - **File:** `frontend/tests/unit/CircuitSolver.test.ts`, `tests/unit/SimulationClock.test.ts`, `tests/unit/SketchRuntime.test.ts`, `tests/unit/kinematics.test.ts`, `tests/unit/vhal-pwm.test.ts`
  - **Apa per file:**
    - `CircuitSolver.test.ts`: import dari `@/lib/simulation/CircuitSolver`; test LED+220Ω return Vf≈1.8V, open circuit = 0, short tidak crash
    - `SimulationClock.test.ts`: import `SimulationClock`; test millis/delay sync, pause freezes, fixed step 5ms
    - `SketchRuntime.test.ts`: import `SketchParser, SketchRuntime`; test parse loop minimal, blink logic
    - `kinematics.test.ts`: import dari `@/lib/robotics/kinematics`; test FK identity, round-trip exp/log error < 1e-10
    - `vhal-pwm.test.ts`: import dari `@/lib/simulation/vhal/pwm`; test clampPwmResolution dan pwmSignal range 0-1
  - **Baris referensi run.cjs:** sekitar 200-600 untuk pola assert yang sudah ada
  - **Risiko:** import path alias — pastikan vitest.config.ts resolve `@` ke `src/`
  - **Verify:** `npm run test:unit` — semua tests hijau, minimal 15 assertions total.

- [ ] **S2-4** Pertahankan run.cjs paralel di CI.
  - **File:** `.github/workflows/ci.yml`
  - **Apa:** CI sudah punya step `npm test` (run.cjs). Tambah step setelah itu: `npm run test:unit`. Urutan: tsc → eslint → npm test → npm run test:unit → build.
  - **Verify:** Dua test commands ada di ci.yml.

### 2b. Playwright E2E

- [ ] **S2-5** Install Playwright dan buat config.
  - **File:** `frontend/package.json`, `frontend/playwright.config.ts`
  - **Apa:** `npm install --save-dev @playwright/test@latest` (pinned).
  - **playwright.config.ts:**
    ```ts
    import { defineConfig } from '@playwright/test';
    export default defineConfig({
      testDir: './tests/e2e',
      use: { headless: true, baseURL: 'http://localhost:3000' },
      webServer: {
        command: 'npm run dev',
        url: 'http://localhost:3000',
        reuseExistingServer: !process.env.CI,
        timeout: 120_000,
      },
    });
    ```
  - Tambah script `"test:e2e": "playwright test"`.
  - Buat direktori `frontend/tests/e2e/`.
  - **Verify:** `npx tsc --noEmit` clean. Directory exist.

- [ ] **S2-6** E2E-1: Blink example → Run → serial ON/OFF.
  - **File:** `frontend/tests/e2e/blink.spec.ts`
  - **Selector referensi dari page.tsx:** Run button = `button[data-tour='run-button']`, serial output container = cari className dari SerialMonitor component
  - **Apa:** `test('blink example shows ON OFF in serial', async ({ page }) => { await page.goto('/'); await page.click('button:has-text("Blink")'); await page.click('[data-tour="run-button"]'); await expect(page.locator('.serial-output, [data-testid="serial"]')).toContainText(/ON|OFF/, { timeout: 10000 }); });`
  - **Verify:** Test dapat run tanpa crash (pass/fail tergantung selector — adjust selector jika perlu setelah inspeksi DOM).

- [ ] **S2-7** E2E-2: Wiring invalid ditolak.
  - **File:** `frontend/tests/e2e/wiring.spec.ts`
  - **Referensi:** page.tsx:763 — `addWire` return value; setNotice(t.wiring.duplicateInvalid)
  - **Apa:** Test: switch ke tab Wiring, pilih pin yang sama untuk from dan to, klik Connect, verifikasi notice muncul.
  - **Verify:** Test file exist.

- [ ] **S2-8** E2E-3: Save/Load JSON roundtrip.
  - **File:** `frontend/tests/e2e/save-load.spec.ts`
  - **Referensi:** page.tsx — onSaveLocal (localStorage set), onLoadLocal (localStorage get)
  - **Apa:** Load Blink example → Save Local → reload → Load Local → verifikasi component count > 0.
  - **Verify:** Test file exist.

- [ ] **S2-9** E2E-4: Flash panel tanpa serial device tidak crash.
  - **File:** `frontend/tests/e2e/flash-panel.spec.ts`
  - **Referensi:** CodeEditorPanel.tsx — flash UI dengan useHardwareStore
  - **Apa:** Klik tab Flash atau tombol Flash, verifikasi halaman masih responsive, pesan muncul.
  - **Verify:** Test file exist.

- [ ] **S2-10** E2E-5: Inspector servo terbuka.
  - **File:** `frontend/tests/e2e/inspector.spec.ts`
  - **Referensi:** page.tsx:972-1022 — inspector panel conditional rendering
  - **Apa:** Load example 'servo', klik servo, verifikasi inspector muncul dengan info Servo.
  - **Verify:** Test file exist.

- [ ] **S2-11** E2E masuk CI.
  - **File:** `.github/workflows/ci.yml`
  - **Apa:** Tambah step setelah build: `npx playwright install --with-deps chromium` dan `npm run test:e2e`. Tambah env `CI: true` agar webServer config di playwright.config.ts pakai build server.
  - **Verify:** ci.yml punya playwright steps.

### 2c. Fuzz Contract Tests

- [ ] **S2-12** Fuzz `parseProject` di Vitest.
  - **File:** `frontend/tests/unit/fuzz.test.ts`
  - **Referensi:** `src/lib/project/project.ts` — `parseProject` function; `run.cjs:592-606` untuk pola
  - **Apa:** Test dengan input: `{version: 1, components: [{id: 'x', typeId: 'led_red', pins: [{id: 'MISSING'}], ...}]}` → tidak throw. Wire dengan `sourcePinId: 'NONEXISTENT'` → tidak throw. Komponen `typeId: 'UNKNOWN_TYPE'` → tidak throw atau error gracefully.
  - **Verify:** `npm run test:unit` hijau untuk fuzz.test.ts.

- [ ] **S2-13** Fuzz `validWire` dan `buildElectricalConnectivity`.
  - **File:** `frontend/tests/unit/fuzz.test.ts` (tambah ke file yang sama)
  - **Referensi:** `src/lib/project/project.ts` — `validWire`; `src/lib/simulation/electricalConnectivity.ts`
  - **Apa:** Test `validWire` dengan componentId tidak dikenal → return false, tidak throw. `buildElectricalConnectivity([], [])` → return valid empty structure. Wire duplikat → ignored.
  - **Verify:** `npm run test:unit` hijau.

- [ ] **S2-14** Fuzz masuk CI.
  - **File:** Sudah tercakup oleh `npm run test:unit` yang masuk CI di S2-4.
  - **Verify:** `npm run test:unit` sudah di CI.

### 2d. Inspector Lazy

- [ ] **S2-15** `OnnxInferencePanel` — conditional mount.
  - **File:** `frontend/src/app/page.tsx`
  - **Baris:** `:1199` — `<OnnxInferencePanel />`
  - **Apa:** Tambah state `const [onnxPanelOpen, setOnnxPanelOpen] = useState(false)`. Ubah dari unconditional ke `{onnxPanelOpen && <OnnxInferencePanel />}`. Tambah tombol trigger di toolbar atau menu (misalnya di DesktopMenuBar atau tombol di right panel). Import `OnnxInferencePanel` via `React.lazy`.
  - **Verify:** `npm test` 95 checks. `npx tsc --noEmit` clean.

- [ ] **S2-16** `CloudAIGatewayPanel` — conditional mount.
  - **File:** `frontend/src/app/page.tsx`
  - **Baris:** `:909` — `{!solderingWorkbenchOpen && <CloudAIGatewayPanel />}`
  - **Apa:** Tambah guard tambahan: `{!solderingWorkbenchOpen && aiGatewayOpen && <CloudAIGatewayPanel />}` dengan state `aiGatewayOpen`. Tambah toggle di toolbar/scene area.
  - **Verify:** `npm test` 95 checks. Build berhasil.

- [ ] **S2-17** Verifikasi inspector panels sudah conditional.
  - **File:** `frontend/src/app/page.tsx` (baris 972-1022)
  - **Apa:** Panels inspector (RobotControlPanel, EncoderControlPanel, RGBDCameraPanel, dll) sudah conditional by `selected.typeId` — ini SUDAH BENAR. Tidak perlu refactor. Cukup konfirmasi dan catat di findings FEAT.
  - **Verify:** Review visual — buka inspector untuk komponen dasar, pastikan tidak ada ONNX panel di DOM (React DevTools).

### 2e. Keyboard Shortcuts

- [ ] **S2-18** Buat `frontend/src/hooks/useWorkspaceShortcuts.ts`.
  - **File:** `frontend/src/hooks/useWorkspaceShortcuts.ts` (buat direktori `hooks/` dulu)
  - **Baris asal:** `page.tsx:210-271`
  - **Apa:** Extract seluruh `useEffect(handler, [selectedComponentId, selectedWireId, ...])` ke custom hook. Signature:
    ```ts
    export function useWorkspaceShortcuts(opts: {
      selectedComponentId: string | null;
      selectedWireId: string | null;
      components: CircuitComponent[];
      removeComponent: (id: string) => void;
      removeWire: (id: string) => void;
      updateComponentPosition: (...) => void;
      updateComponentRotation: (...) => void;
      setCameraMode: (mode: 'orbit' | 'pan') => void;
    }): void
    ```
  - Import dan call di `page.tsx` (atau `WorkspacePage.tsx` setelah S3-1) menggantikan useEffect inline.
  - **Risiko:** dependency array di useEffect — pastikan sama persis dengan yang lama untuk menghindari stale closure
  - **Verify:** `npm test` 95. `npx tsc --noEmit` clean. Shortcut masih berfungsi (manual test atau E2E).

- [ ] **S2-19** Tambah daftar shortcut di Help modal.
  - **File:** `frontend/src/app/page.tsx` (baris 546-596 Help modal)
  - **Apa:** Tambah seksi di dalam Help modal content:
    ```tsx
    <div className="p-3 bg-slate-800/80 rounded border border-slate-700">
      <strong className="text-violet-400 block mb-1">Keyboard Shortcuts</strong>
      <p><kbd>R</kbd> — Rotate komponen 90°</p>
      <p><kbd>H</kbd> — Switch ke Pan mode</p>
      <p><kbd>O</kbd> — Switch ke Orbit mode</p>
      <p><kbd>Delete</kbd>/<kbd>Backspace</kbd> — Hapus komponen/kabel</p>
      <p><kbd>Arrow</kbd> — Pindah presisi (+ <kbd>Shift</kbd> = 2×)</p>
      <p><kbd>Esc</kbd> — Batalkan wiring</p>
    </div>
    ```
  - **Verify:** `npm test` 95. Build berhasil. Help modal menampilkan shortcut.

- [ ] **S2-20** Single-letter shortcut hanya aktif saat canvas fokus.
  - **File:** `frontend/src/hooks/useWorkspaceShortcuts.ts`
  - **Referensi:** Guard sudah ada di `page.tsx:213-220` — `if (isInputActive) return;`. Guard ini sudah mencakup `INPUT/TEXTAREA/contenteditable`.
  - **Apa:** Guard sudah benar untuk menghindari shortcut saat Monaco fokus. Verifikasi guard di useWorkspaceShortcuts.ts menggunakan logika yang sama persis. Tambah cek tambahan: `if (activeElem?.closest('[data-monaco-editor]')) return;` untuk keamanan ekstra.
  - **Verify:** `npm test` 95. Manual: ketik di Monaco → R tidak rotate.

### 2f. Backend Error UX

- [ ] **S2-21** Bedakan kondisi error backend.
  - **File:** `frontend/src/app/page.tsx`
  - **Baris:** `315-335` (fetchServerProjects), `381-394` (loadServerProject)
  - **Apa:** Di catch block `guard()`, tambah conditional: `if (e instanceof TypeError && e.message.includes('fetch')) { setNotice('Backend offline — jalankan tanpa backend, data tersimpan di localStorage'); } else if (e instanceof DOMException && e.name === 'TimeoutError') { setNotice('Koneksi timeout (5s) — coba lagi nanti'); } else if (e instanceof Error && e.message.includes(t.notices.backendRejected)) { setNotice('Server backend menolak permintaan: ' + e.message); } else { setNotice(e.message); }`
  - **Verify:** `npx tsc --noEmit` clean. Build berhasil.

- [ ] **S2-22** Tambah tombol Retry.
  - **File:** `frontend/src/app/page.tsx`
  - **Apa:** Ubah state `notice` dari `string` ke `{ message: string; retryFn?: () => void }`. Update semua `setNotice(str)` ke `setNotice({ message: str })`. Untuk fetch errors, `setNotice({ message: '...', retryFn: () => void fetchServerProjects() })`. Di notice render (sekitar baris 535-545), jika `notice.retryFn` ada, render `<button onClick={notice.retryFn}>Coba Lagi</button>`.
  - **Risiko:** medium — `notice` state type change menyentuh banyak call site. Lakukan search-replace careful.
  - **Verify:** `npx tsc --noEmit` clean. `npm test` 95.

- [ ] **S2-23** Dokumentasikan 2 mode operasi.
  - **File:** `frontend/README.md`, `frontend/src/app/page.tsx` (footer atau banner)
  - **Apa di README:** Sudah dicakup di S1-1 (Quick Start section — dua mode). Update jika perlu lebih eksplisit.
  - **Apa di UI:** Di footer workspace (sekitar baris 1178), tambah teks `Mode: Lokal (tanpa backend)` atau detect apakah backend ada dan tampilkan status. Minimal: di `t.footer.text` atau tambah `<span>` kecil.
  - **Verify:** Build berhasil. Footer/banner menampilkan info mode.

---

## Sprint 3 — Split God-File & Bundle Optimization

> **GATE:** Mulai Sprint 3 HANYA setelah `npm test` 95 checks + `npm run test:unit` hijau + 5 E2E ada.

### 3a. Split `page.tsx`

- [ ] **S3-1** Buat `frontend/src/app/WorkspacePage.tsx`.
  - **File:** `frontend/src/app/WorkspacePage.tsx`, `frontend/src/app/page.tsx`
  - **Apa:** Pindahkan `export default function WorkspacePage()` dari `page.tsx` ke `WorkspacePage.tsx`. `page.tsx` menjadi:
    ```tsx
    'use client';
    export { default } from './WorkspacePage';
    ```
  - **Commit:** satu commit. **Verify setelah commit:** `npm test` 95. `npx tsc --noEmit` clean. Build berhasil.

- [ ] **S3-2** Ekstrak `LibraryPanel.tsx`.
  - **File:** `frontend/src/components/panels/LibraryPanel.tsx`, `frontend/src/app/WorkspacePage.tsx`
  - **Apa:** Dari WorkspacePage.tsx, ambil `<aside className="library-panel">` beserta state `query, tab` dan handler add component/example. Buat `LibraryPanel.tsx` dengan props: `{ components, addComponent, selectComponent, onExampleLoad, onNotice, simulationState }`. State lokal: `query, tab`.
  - **Commit:** satu commit. **Verify:** `npm test` 95. Build berhasil.

- [ ] **S3-3** Ekstrak `WiringPanel.tsx`.
  - **File:** `frontend/src/components/panels/WiringPanel.tsx`, `frontend/src/app/WorkspacePage.tsx`
  - **Apa:** Ekstrak `<div className="wiring-panel">` (konten tab wiring) ke `WiringPanel.tsx`. Props: `{ components, addWire, onNotice }`. State lokal: `from, to, color`.
  - **Commit:** satu commit. **Verify:** `npm test` 95. Build berhasil.

- [ ] **S3-4** Ekstrak `InspectorPanel.tsx`.
  - **File:** `frontend/src/components/panels/InspectorPanel.tsx`, `frontend/src/app/WorkspacePage.tsx`
  - **Apa:** Ekstrak `<aside className="right-inspector-panel">` dengan semua inspector cards (baris ~910-1072). Props panjang — semua yang dibutuhkan oleh inspector panel termasuk `selected, rightPanelOpen, onOpenDatasheet, simulationState, updateComponentState, ...`.
  - **Commit:** satu commit. **Verify:** `npm test` 95. Build berhasil.

- [ ] **S3-5** Ekstrak `SceneSection.tsx`.
  - **File:** `frontend/src/components/canvas/SceneSection.tsx`, `frontend/src/app/WorkspacePage.tsx`
  - **Apa:** Ekstrak area canvas 3D (SimulatorCanvas + overlay buttons + diagnostics + CloudAIGatewayPanel, sekitar baris 830-912). Props: `{ solderingWorkbenchOpen, isWiringActive, selectedWireId, cameraMode, setCameraMode, placementNotice, diagnostics, aiGatewayOpen }`.
  - **Commit:** satu commit. **Verify:** `npm test` 95. Build berhasil. `wc -l frontend/src/app/WorkspacePage.tsx` < 200.

- [ ] **S3-6** Integrate `useWorkspaceShortcuts` di `WorkspacePage.tsx`.
  - **File:** `frontend/src/app/WorkspacePage.tsx`
  - **Apa:** Verifikasi useWorkspaceShortcuts dari S2-18 sudah di-call di WorkspacePage (bukan di page.tsx yang sudah menjadi re-export). Jika belum, pindahkan call-site.
  - **Commit:** satu commit jika ada perubahan.
  - **Verify:** Shortcut berfungsi. `npm test` 95.

- [ ] **S3-7** Validasi page.tsx < 200 baris.
  - **Apa:** `wc -l frontend/src/app/page.tsx` → should be < 10 (hanya re-export). `wc -l frontend/src/app/WorkspacePage.tsx` → should be < 200.
  - **Verify:** Output `wc -l` sesuai target. Tidak ada file baru > 500 baris.

### 3b. Split `simulation.worker.ts`

- [ ] **S3-8** Buat `circuitStep.ts`.
  - **File:** `frontend/src/lib/simulation/steps/circuitStep.ts`
  - **Baris asal worker:** function `solve()` (sekitar baris 62-68), `integrateDcMotors()` (sekitar baris 69-90), dan electrical connectivity handling.
  - **Apa:** Export `function runCircuitStep(params: CircuitStepParams): CircuitStepResult`. Pindahkan logic keluar dari worker. Worker import dan call fungsi ini.
  - **Commit.** **Verify:** `npm test` 95. `npx tsc --noEmit` clean.

- [ ] **S3-9** Buat `motorStep.ts`.
  - **File:** `frontend/src/lib/simulation/steps/motorStep.ts`
  - **Baris asal:** import dan call `advanceA4988Pulse`, `advanceDcMotor`, encoder step.
  - **Commit.** **Verify:** `npm test` 95.

- [ ] **S3-10** Buat `batteryPowerStep.ts`.
  - **File:** `frontend/src/lib/simulation/steps/batteryPowerStep.ts`
  - **Baris asal:** import dan call `advanceBatterySoc`, `advanceBatteryProtection`, `advanceBrownout`, `advanceLipo2sSoc`, `advanceBatteryChargerThermal`, `advanceChargerSafetyTimer`, `advanceL298NThermal`, `advanceA4988Thermal`.
  - **Commit.** **Verify:** `npm test` 95.

- [ ] **S3-11** Buat `i2cSensorStep.ts`.
  - **File:** `frontend/src/lib/simulation/steps/i2cSensorStep.ts`
  - **Baris asal:** `i2cBus`, `connectedI2CDevices`, SSD1306, PCA9685, MPU6050 register handling, `sensorReadApi`.
  - **Commit.** **Verify:** `npm test` 95.

- [ ] **S3-12** Buat `roboticsStep.ts`.
  - **File:** `frontend/src/lib/simulation/steps/roboticsStep.ts`
  - **Baris asal:** `integrateDifferentialDrive`, arm robot (`aeroArm6Dof`, `eduArm3Dof`), encoder/interrupt dispatch, rover bindings.
  - **Commit.** **Verify:** `npm test` 95.

- [ ] **S3-13** `simulation.worker.ts` jadi dispatcher.
  - **File:** `frontend/src/lib/simulation/simulation.worker.ts`
  - **Apa:** Setelah semua step diekstrak, worker hanya berisi: imports, message handler, `onmessage` dispatch ke step functions. Target < 300 baris.
  - **Verify:** `wc -l frontend/src/lib/simulation/simulation.worker.ts` < 300. `npm test` 95. Build berhasil.

- [ ] **S3-14** Test per step di Vitest.
  - **File:** `frontend/tests/unit/steps/circuitStep.test.ts`, dll (5 file)
  - **Apa:** Smoke test: setiap step function dapat diimport dan dipanggil dengan minimal state tanpa throw. Tidak perlu full integration.
  - **Verify:** `npm run test:unit` hijau.

### 3c. Split `useSimulatorStore.ts`

- [ ] **S3-15** Buat `workspaceSlice.ts`.
  - **File:** `frontend/src/store/slices/workspaceSlice.ts`
  - **Apa:** Slice untuk: `components, wires, selectedComponentId, selectedWireId, cameraView, cameraMode, placementNotice`. Pattern: `export type WorkspaceSlice = { ... }; export const createWorkspaceSlice: StateCreator<WorkspaceSlice> = (set, get) => ({ ... })`.
  - **Commit.** **Verify:** `npx tsc --noEmit` clean.

- [ ] **S3-16** Buat `wiringSlice.ts`.
  - **File:** `frontend/src/store/slices/wiringSlice.ts`
  - **Commit.** **Verify:** `npx tsc --noEmit` clean.

- [ ] **S3-17** Buat `editorSlice.ts`.
  - **File:** `frontend/src/store/slices/editorSlice.ts`
  - **Commit.** **Verify:** `npx tsc --noEmit` clean.

- [ ] **S3-18** Buat `simRuntimeSlice.ts`.
  - **File:** `frontend/src/store/slices/simRuntimeSlice.ts`
  - **Apa:** State: `simulationState, elapsedMs, voltages, serialOutput, baudRate, sketchBaudRate, diagnostics`. Actions: `startSimulation, stopSimulation, pauseSimulation, addSerialMessage, clearSerial, setBaudRate`.
  - **Commit.** **Verify:** `npx tsc --noEmit` clean.

- [ ] **S3-19** Buat `sensorFrameSlice.ts`.
  - **File:** `frontend/src/store/slices/sensorFrameSlice.ts`
  - **Apa:** State: `rgbdFrames, lidarFrames, imuFrames`. Actions: `setRGBDFrame, setLidarFrame, setImuFrame`.
  - **Commit.** **Verify:** `npx tsc --noEmit` clean.

- [ ] **S3-20** Buat `store/index.ts` yang compose semua slice.
  - **File:** `frontend/src/store/index.ts`, update `frontend/src/store/useSimulatorStore.ts`
  - **Apa:** `useSimulatorStore.ts` diubah menjadi composer yang menggabungkan semua slice menjadi satu store dengan `create<AllSlices>()((...a) => ({ ...createWorkspaceSlice(...a), ...createWiringSlice(...a), ... }))`. Interface `SimulatorState` tetap sama — consumer tidak perlu diubah import-nya.
  - **Commit.** **Verify:** `npm test` 95. `npx tsc --noEmit` clean. Build berhasil. Tidak ada file slice > 200 baris.

### 3d. Split `CodeEditorPanel.tsx`

- [ ] **S3-21** Ekstrak `CodeEditorShell.tsx`.
  - **File:** `frontend/src/components/panels/CodeEditorShell.tsx`
  - **Apa:** Orkestrator panel editor — import LibraryCatalog, HardwareFlashPanel, MonacoWrapper. CodeEditorPanel.tsx re-export CodeEditorShell.
  - **Commit.** **Verify:** `npm test` 95. Build berhasil.

- [ ] **S3-22** Ekstrak `LibraryCatalog.ts`.
  - **File:** `frontend/src/lib/editor/LibraryCatalog.ts`
  - **Baris asal:** `CodeEditorPanel.tsx:44-200` (function `buildLibrariesCatalog` dan interface `LibraryItem`)
  - **Commit.** **Verify:** `npx tsc --noEmit` clean.

- [ ] **S3-23** Ekstrak `HardwareFlashPanel.tsx`.
  - **File:** `frontend/src/components/panels/HardwareFlashPanel.tsx`
  - **Apa:** Semua UI dan logic untuk flash ke hardware fisik (esptool-js, `useHardwareStore`, progress display, WebSerial detection).
  - **Commit.** **Verify:** `npm test` 95. Build berhasil.

- [ ] **S3-24** Ekstrak `MonacoWrapper.tsx`.
  - **File:** `frontend/src/components/editor/MonacoWrapper.tsx`
  - **Baris asal:** `CodeEditorPanel.tsx:32-40` (dynamic import Monaco, loading fallback)
  - **Apa:** Wrapper dengan props: `{ value, onChange, language, theme, height, options? }`. Pastikan dynamic import tetap ada sehingga Monaco tidak masuk initial bundle.
  - **Commit.** **Verify:** `wc -l frontend/src/components/panels/CodeEditorPanel.tsx` (atau CodeEditorShell.tsx) < 500. `npm test` 95. Build berhasil.

### 3e. Bundle Lazy Loading

- [ ] **S3-25** Lazy-load `pyodide`.
  - **File:** `frontend/src/lib/simulation/PythonInterpreter.ts` (atau `python.worker.ts`)
  - **Apa:** Ubah `import { loadPyodide } from 'pyodide'` ke dalam async function. Dynamic import: `const { loadPyodide } = await import('pyodide');`. Tambah loading state di UI.
  - **Verify:** `npm run build -- --webpack`. Check: `pyodide` tidak muncul di first-load chunks.

- [ ] **S3-26** Lazy-load `onnxruntime-web`.
  - **File:** Cari `import * as ort from 'onnxruntime-web'` di `src/lib/ai/` atau OnnxInferencePanel
  - **Apa:** Ubah ke `const ort = await import('onnxruntime-web');` dalam fungsi yang dipanggil saat panel aktif.
  - **Verify:** Build berhasil. onnxruntime tidak di initial bundle.

- [ ] **S3-27** Lazy-load `@mediapipe/tasks-vision`.
  - **File:** Cari usage di `src/`
  - **Apa:** Ubah ke dynamic import dalam fungsi aktivasi MediaPipe.
  - **Verify:** Build berhasil.

- [ ] **S3-28** Lazy-load `@dimforge/rapier3d-compat`.
  - **File:** Cari import di physics worker atau simulation worker
  - **Apa:** Jika di worker terpisah, verifikasi worker tidak di-instantiate di main bundle eagerly. Jika ada eager import, ubah ke dynamic.
  - **Verify:** Build berhasil.

- [ ] **S3-29** Verifikasi Monaco lazy.
  - **File:** `frontend/src/components/editor/MonacoWrapper.tsx`
  - **Apa:** Konfirmasi `@monaco-editor/react` hanya di-import via `next/dynamic`. Tidak ada static import di file lain.
  - **Verify:** `grep -r "@monaco-editor/react" src/ --include="*.ts" --include="*.tsx"` — hanya ada di MonacoWrapper.tsx.

- [ ] **S3-30** Route split `/lab`.
  - **File:** `frontend/src/app/lab/page.tsx` (baru), `frontend/src/app/WorkspacePage.tsx` (filter)
  - **Apa:** Buat `frontend/src/app/lab/page.tsx` yang render workspace dengan `NEXT_PUBLIC_SHOW_EXPERIMENTAL=true` atau direct import semua komponen. Di route `/` (WorkspacePage), filter hanya P0 components dan tidak load heavy panels. Ini memungkinkan route `/` dengan first-load lebih ringan.
  - **Verify:** `npm run build -- --webpack` berhasil. Dua routes exist.

### 3f. Validasi Akhir Sprint 3

- [ ] **S3-31** TypeScript clean.
  - **Verify:** `cd frontend && npx tsc --noEmit` — exit 0.

- [ ] **S3-32** ESLint clean.
  - **Verify:** `cd frontend && npx eslint --max-warnings 0` — exit 0.

- [ ] **S3-33** run.cjs 95+ checks.
  - **Verify:** `cd frontend && npm test` — `95 checks passed`.

- [ ] **S3-34** Vitest hijau.
  - **Verify:** `cd frontend && npm run test:unit` — all tests pass.

- [ ] **S3-35** E2E Playwright hijau.
  - **Verify:** `cd frontend && npm run test:e2e` — 5 tests pass.

- [ ] **S3-36** Next build berhasil.
  - **Verify:** `cd frontend && npm run build -- --webpack` — exit 0.

- [ ] **S3-37** Catat delta bundle.
  - **File:** `docs/BUNDLE-BASELINE.md`
  - **Apa:** Tambah seksi `## After Sprint 3` dengan ukuran first-load JS baru. Hitung delta: (baseline - after) / baseline × 100%. Catat apakah target ≥20% tercapai.
  - **Verify:** File updated.

- [ ] **S3-38** Git status bersih.
  - **Verify:** `cd frontend && git status` — tidak ada `.next/`, `public/pyodide/`, `node_modules/`, `https:/`. Tambah `https:` ke `.gitignore` jika belum ada.

---

## Item D — Dokumentasi Teknis

- [ ] **D1** `ASUMSI.md` untuk 7 V-HAL model.
  - **File:** `frontend/src/lib/simulation/vhal/docs/a4988.ASUMSI.md`, `l298n.ASUMSI.md`, `dc_motor.ASUMSI.md`, `battery.ASUMSI.md`, `charger_thermal.ASUMSI.md`, `dc_dc.ASUMSI.md`, `ssd1306.ASUMSI.md`
  - **Format per file (5 baris/seksi):**
    1. **Rumus utama** — formula yang dipakai (dari RENCANA.md addendum)
    2. **Parameter default** — nilai default dan asumsi
    3. **Sumber datasheet** — link atau "Asumsi edukasional (tidak ada datasheet spesifik)"
    4. **Yang TIDAK dimodelkan** — eksplisit list
    5. **Status validasi** — "Belum divalidasi terhadap hardware fisik" atau "Divalidasi: [referensi]"
  - **Verify:** 7 file exist. Setiap file ≥ 5 seksi.

- [ ] **D2** Link dari komponen ke INVENTARIS.md.
  - **File:** `frontend/src/components/panels/LibraryPanel.tsx` (atau page.tsx sebelum S3-2)
  - **Apa:** Badge fidelity dari S1-4 sudah ada tooltip. Tambah — di bagian bawah setiap component-card — link kecil `<a href="../docs/INVENTARIS.md">detail fidelity ↗</a>` atau tooltip yang menyebut "Lihat INVENTARIS.md untuk detail tingkat akurasi".
  - **Verify:** Badge punya informasi yang adequate tentang fidelity level.

- [ ] **D3** Dokumentasikan batas simulator di UI.
  - **File:** `frontend/src/app/page.tsx` atau `WorkspacePage.tsx` (Help modal baris ~546-596)
  - **Apa:** Tambah tab atau seksi "Batas Simulator" dalam Help modal:
    - Integer division C++ tidak dimodelkan
    - Overflow C++ tidak dimodelkan
    - Cycle-accurate ISR tidak dimodelkan
    - SPI fisik tidak dimodelkan
    - UART timing tidak dimodelkan
    - Switching PWM (hanya rata-rata duty cycle)
    - Winding R-L cycle (kuasistatik)
    - Bukan AVR instruction emulator
  - **Verify:** Help modal punya seksi batas. `npm test` 95. Build berhasil.

---

## Summary Metrics Target

| Metrik | Sebelum (Aktual) | Target | Validasi |
|--------|-----------------|--------|----------|
| `page.tsx` baris | **1210** | < 10 (re-export only) | `wc -l` |
| `simulation.worker.ts` baris | **1627** | < 300 (dispatcher) | `wc -l` |
| `useSimulatorStore.ts` baris | **471** | < 200/slice, store.ts < 50 | `wc -l` |
| `CodeEditorPanel.tsx` baris | **1240** | < 500 | `wc -l` |
| `run.cjs` checks | **95** | 95+ | `npm test` |
| E2E tests | **0** | 5 | `npm run test:e2e` |
| Unit tests (vitest) | **0** | ≥15 assertions | `npm run test:unit` |
| CI pipeline | **tidak ada** | hijau | `.github/workflows/ci.yml` |
| First-load JS/WASM | **? (ukur S1)** | -20% dari baseline | `docs/BUNDLE-BASELINE.md` |
| Komponen tanpa label fidelity | **semua (34)** | 0 | visual + `ComponentRegistry.ts` |
| V-HAL ASUMSI docs | **0** | 7 | `ls vhal/docs/` |
| README template lines | **36** | 0 | `grep "create-next-app"` |

---

## Catatan Risiko Per Sprint

| Sprint | Risiko Terbesar | Mitigasi |
|--------|----------------|---------|
| S0 | Keputusan tidak dieksekusi → tetap wacana | Tulis sebagai statement konkret dengan "apa yang berhenti dikerjakan" |
| S1-c | Copy scripts crash di CI | Guard `if (!fs.existsSync(src)) return` di copy scripts |
| S1-c | Turbopack di CI | Sudah di-handle dengan `--webpack` flag |
| S2 | Selector Playwright salah | Gunakan `data-testid` atau `data-tour` yang sudah ada di kode |
| S3-a/b | Regresi saat split | `npm test` 95 HARUS hijau di setiap commit — jangan big-bang |
| S3-c | Store compose type error | Test tsc --noEmit setelah setiap slice |
| S3-e | Dynamic import break pyodide worker | Pyodide sudah di worker terpisah — cek python.worker.ts import path |
| D1 | Nilai parameter salah | Ambil dari RENCANA.md addendum yang sudah detail |

---

## Urutan File Eksekusi FEAT

Implementasi dibagi menjadi 5 FEAT yang dieksekusi secara sequential:
1. **FEAT-001** (docs + labels + CI prep + bundle baseline) — Sprint 0, S1-1 hingga S1-13
2. **FEAT-002** (CI pipeline) — Sprint 1c, S1-7 hingga S1-9
3. **FEAT-003** (vitest + playwright + fuzz) — Sprint 2a/b/c
4. **FEAT-004** (UX + split god-files) — Sprint 2d/e/f + Sprint 3a/b/c/d
5. **FEAT-005** (bundle lazy + route /lab + validasi final + D1/D2/D3) — Sprint 3e/f + D items

FEAT artifacts lengkap ada di:
`/Users/user/Projects/simulate/.agents/tasks/task-ardusim-audit-08/`
