# 08 — Audit Kritis Pedas & Rencana Perbaikan (Frontend)

> Sumber: bedah langsung repo `simulate/frontend` (6 Okt 2026).
> Sifat: jujur, berlawanan arah, dan terverifikasi baris-per-baris. Bukan opini tanpa bukti.
> Status: ~80% temuan berdasar kuat, ~20% hiperbola yang dikoreksi di §8.
> **Revisi 2 (6 Okt 2026)**: ditambahkan verifikasi independen + koreksi lanjutan (§10) dan **Rencana Implementasi FINAL 3-sprint (§11) yang menggantikan checklist §9** — urutan eksekusi dikoreksi: harness test & CI duluan, split god-file belakangan. Batas verifikasi jujur di §12.

## 0. Ringkasan Eksekutif

Project `IDN MAKERSPACE LAB 3D` itu berani: simulator Arduino/ESP32 + robotik + AI 3D di browser.
Masalahnya: mau jadi Tinkercad + Wokwi + Gazebo + Arduino IDE sekaligus.

Akibatnya:
- Katalog 30+ komponen tapi banyak model "edukasional ideal", bukan ekuivalen hardware.
- 4 god-file >1200 baris, 1 store campur semua urusan.
- README masih template create-next-app.
- Bundle gabungan three + rapier + pyodide + onnx + mediapipe + monaco = berat, mobile menyerah.
- Test logic kuat, tapi test sistem/UI/E2E nol.
- Strategi lawan 3 raksasa sekaligus = kalah di semua front.

Dokumen ini daftar yang **wajib diperbaiki**, dengan bukti file:baris + acceptance criteria.

## 1. Metodologi Audit

File yang dibedah:
- `frontend/package.json` — deps & scripts
- `frontend/README.md:1-47` — masih template
- `frontend/next.config.ts:1-7` — `distDir: ARDUSIM_BUILD_DIR`
- `frontend/src/app/page.tsx:1210` — workspace monolit
- `frontend/src/store/useSimulatorStore.ts:471` — store monolit
- `frontend/src/lib/simulation/simulation.worker.ts:1627` — worker monolit
- `frontend/src/components/panels/CodeEditorPanel.tsx:1240`
- `frontend/src/app/globals.css:1603`
- `frontend/src/lib/components/ComponentRegistry.ts:690` — 30+ tipe
- `frontend/tests/run.cjs` — harness custom 900+ baris
- `frontend/.env.example`, `.env.local` — `NEXT_PUBLIC_API_URL=http://localhost:8080`
- `frontend/src/lib/simulation/ArduinoInterpreter.ts`, `SketchRuntime.ts`, `CircuitSolver.ts`, `SimulationClock.ts` — custom, tanpa avr8js

Verifikasi: `wc -l`, `grep`, `ls -la`, baca langsung potongan kode.

## 2. Temuan P0 — Wajib Beres Duluan

### P0-1. Scope Creep: Katalog RS Components KW
**Bukti:**
- `src/lib/components/ComponentRegistry.ts:188` — Siglent SPD3303X-E: "solver menyediakan satu keluaran virtual 0–24V"
- `src/lib/components/ComponentRegistry.ts:220` — LM2596: "solver memakai regulator ideal edukasional... bukan switching LM2596"
- `src/lib/components/ComponentRegistry.ts:243` — TP4056: "solver charger edukasional... bukan ekuivalen IC TP4056"
- `src/lib/components/ComponentRegistry.ts:396` — LED: "pilihan warna adalah opsi visual simulator, bukan LED RGB fisik"

Lu sendiri jujur di deskripsi, bagus. Tapi user lihat nama "LM2596 / TP4056 / Siglent" dan mengira akurat.

**Perbaikan:**
- [ ] Tambah label jujur di UI: `Edukasi-Ideal` vs `Tervalidasi-Datasheet` per komponen. Jangan biarkan nama dagang menipu.
- [ ] Sunat katalog aktif jadi 10 tipe P0: `arduino_uno, breadboard, led_red, resistor_220, push_button, potentiometer, dht11, hcsr04, servo_sg90, jumper_*`. Sisanya pindah ke `Experimental / Hidden behind flag`.
- [ ] Buat `docs/INVENTARIS.md` kolom baru: `Tingkat Fidelity: Ideal / Quasi-static / Datasheet`. Sinkron dengan `ComponentRegistry.ts`.
- [ ] Acceptance: tidak ada komponen berlabel nama IC komersial tanpa disclaimer di card + datasheet modal.

### P0-2. God-File & Store Monolit
**Bukti:**
- `src/app/page.tsx:1210` — menubar + toolbar + library + wiring + 3D + inspector + code + serial + footer + tour + modal dalam 1 file.
- `src/components/panels/CodeEditorPanel.tsx:1240`
- `src/lib/simulation/simulation.worker.ts:1627` — import semua: `vhal/pwm, vhal/a4988, vhal/dcMotor, vhal/battery, vhal/brownout, vhal/l298n, vhal/chargerThermal, vhal/i2c, vhal/ssd1306`, dst (`simulation.worker.ts:1-21`).
- `src/store/useSimulatorStore.ts:18-96` — workspace + wiring + editor + sim + serial + `rgbdFrames/lidarFrames/imuFrames` campur.
- `src/app/globals.css:1603`

**Perbaikan:**
- [ ] Pecah `page.tsx` minimal jadi: `WorkspacePage.tsx` (orkestrasi) + `LibraryPanel.tsx` + `WiringPanel.tsx` + `InspectorPanel.tsx` + `SceneSection.tsx` + `useWorkspaceShortcuts.ts` (pindahan `page.tsx:209-271`).
- [ ] Pecah `simulation.worker.ts` per domain: `circuitStep.ts`, `motorStep.ts`, `batteryPowerStep.ts`, `i2cSensorStep.ts`, `roboticsStep.ts`. Worker tinggal dispatcher.
- [ ] Pecah `useSimulatorStore.ts` jadi slice: `useWorkspaceStore`, `useWiringStore`, `useEditorStore`, `useSimRuntimeStore`, `useSensorFrameStore`. Atau minimal split file dengan `create` terpisah, jangan 1 interface 471 baris.
- [ ] Pecah `CodeEditorPanel.tsx` jadi: `CodeEditorShell.tsx` + `LibraryCatalog.ts` + `HardwareFlashPanel.tsx` + `MonacoWrapper.tsx`.
- [ ] Acceptance: tidak ada file `src/` >500 baris kecuali generated; `eslint --max-warnings 0` + `tsc --noEmit` lolos; `npm test` tetap hijau.

### P0-3. README Basi = Afterthought
**Bukti:**
- `README.md:1-36` murni template `create-next-app`: "Learn More about Next.js", "Deploy on Vercel".
- Hanya `README.md:38-47` yang asli (kinematik reference check).

**Perbaikan:**
- [ ] Tulis ulang `frontend/README.md`: apa ini (1 paragraf), cara jalan (`npm run dev`, butuh backend Go di `:8080` atau bisa jalan tanpa backend?), seakurat apa (link ke `docs/INVENTARIS.md` + disclaimer ideal), cara reproduksi test (`npm test`, `test:kinematics-reference`), batas mobile.
- [ ] Acceptance: kontributor baru bisa setup <10 menit tanpa tanya lu.

## 3. Temuan P1 — Performa & Arsitektur Runtime

### P1-1. Bundle Bunuh Diri
**Bukti:**
- `package.json dependencies`: `next, react, three, @react-three/fiber, @react-three/drei, @dimforge/rapier3d-compat, pyodide, onnxruntime-web@1.30.0, @mediapipe/tasks-vision, @monaco-editor/react, esptool-js, zustand`.
- `package.json scripts`: `predev/prebuild/prestart` selalu `copy-pyodide-assets + copy-onnx-assets`.
- `src/app/page.tsx:1207` selalu mount `<MobileWarningBanner />` = ngaku gagal mobile.

**Perbaikan:**
- [ ] Pasang `next/bundle-analyzer`, catat baseline JS/WASM first-load. Target awal: potong 30%.
- [ ] Lazy-load serius: `pyodide`, `onnxruntime-web`, `mediapipe`, `rapier`, `monaco` hanya saat fitur dipakai (dynamic import + loading state), bukan di bundle awal.
- [ ] Pisah route: `/` (rangkaian dasar ringan) vs `/lab` (robotik+AI berat). Jangan paksa anak SMP download fisika 6DOF buat blink LED.
- [ ] Acceptance: Lighthouse mobile tidak merah; halaman blink bisa dibuka di HP mid-range tanpa warning.

### P1-2. Simulator Custom Tanpa Pembanding Proven
**Bukti:**
- Tidak ada `avr8js/simulavr` di `package.json`. Yang ada `ArduinoInterpreter.ts + SketchRuntime.ts + CircuitSolver.ts + SimulationClock.ts` bikinan sendiri.
- Klaim akurasi kinematik kuat (`README.md:47`: 10.000 FK/SE(3)/Jacobian + 250 IK, error FK `1.9e-16m` di `docs/README.md:84`), tapi itu kinematik, bukan timing interrupt/PWM/I2C/brownout.

**Perbaikan:**
- [ ] Bekukan semantik: dokumentasikan yang TIDAK dimodelkan (integer division C++, overflow, cycle-accurate ISR, SPI, UART timing — sebagian sudah di `docs/INVENTARIS.md:24-28`, tinggal jujur di UI).
- [ ] Tambah differential-test vs hardware nyata untuk 5 sketch kanonik (blink timing, PWM-analogRead, button pullup, DHT11, HC-SR04). Catat deviasi ms/V di docs, bukan klaim "realtime".
- [ ] Tandai `brownout/thermal/SoC/safetyTimer` sebagai `model edukasi kuasistatik`, bukan proteksi silikon. Sudah tersirat di kode, tinggal eksplisit di panel.
- [ ] Acceptance: tiap model V-HAL punya `ASUMSI.md` 5 baris: rumus, parameter default, sumber datasheet (jika ada), dan apa yang diabaikan.

## 4. Temuan P1 — Testing & Kualitas

**Bukti:**
- `tests/run.cjs` compile ~20 file TS lalu `assert` manual. Isinya sebenarnya lumayan (I2C ACK/NACK, SSD1306, PoE 10k pose, DLS IK, FABRIK 200 sampel, PWM, encoder, solver LED/Ohm, dsb).
- Tapi: bukan Vitest/Jest/Playwright, tidak cover UI/3D/worker-threading/E2E sensor-Python-I2C-gateway.

**Perbaikan:**
- [ ] Pertahankan `run.cjs` sebagai unit-logic gate, tapi tambah: `vitest` untuk lib murni + `playwright` untuk 5 alur E2E (load example blink → run → serial ON/OFF; wiring invalid ditolak; save/load JSON roundtrip; flash panel tidak crash tanpa serial).
- [ ] Tambah contract-test `parseProject`/`validWire`/`buildElectricalConnectivity` fuzz ringan (pin MISSING, wire duplikat — sebagian sudah ada di `run.cjs:592-606`, tinggal otomatisasi CI).
- [ ] Acceptance: `npm test` + `npx tsc --noEmit` + `eslint` + 5 E2E hijau di CI. Browser E2E yang disebut "belum terverifikasi" di `docs/README.md:9` jadi terverifikasi.

## 5. Temuan P2 — UX Panel Neraka

**Bukti:**
- `src/app/page.tsx:909` — `<CloudAIGatewayPanel />` hampir selalu mounted (selama bukan soldering mode).
- `src/app/page.tsx:1200` — `<OnnxInferencePanel />` selalu mounted tanpa kondisi.
- `src/app/page.tsx:972-1022` — 10+ panel kondisional by `typeId`.
- Shortcut global `src/app/page.tsx:210-271`: `Delete/Backspace, Arrow, R, H, O`, dengan guard `INPUT/TEXTAREA/contenteditable` di `:213-220` (sudah benar, tapi tetap fragile + tidak discoverable).

**Perbaikan:**
- [ ] Virtualisasi inspector: hanya mount panel yang aktif (`React.lazy` per `typeId`), unmount sisanya. Jangan mount ONNX + gateway kalau tidak dipakai.
- [ ] Pindah shortcut ke `useWorkspaceShortcuts.ts` + tampilkan di Help modal (`page.tsx:546-596`) + hindari single-letter (`R/H/O`) tanpa modifier saat editor fokus. Pertimbangkan `Ctrl/Cmd+` atau hanya saat canvas fokus.
- [ ] Tambah `aria-label`, focus-trap modal, dan uji keyboard-only 1x.
- [ ] Acceptance: buka inspector servo tidak ikut mount LiDAR/ONNX (cek React DevTools); tidak ada shortcut nyasar saat ngetik di Monaco.

## 6. Temuan P2 — Backend Coupling & Build Membingungkan

**Bukti:**
- `.env.example` + `.env.local`: `NEXT_PUBLIC_API_URL=http://localhost:8080`.
- `src/app/page.tsx:315-335,381-394`: fetch `/api/projects` dengan `AbortSignal.timeout(5000)`, error cuma jadi `notice`.
- `next.config.ts:4`: `distDir: ARDUSIM_BUILD_DIR || ".next"`, dan di working dir ada `.next/` + `.next-production/` dua-duanya.
- Folder `https:/cdn.jsdelivr.net/pyodide/...` — BUKAN typo lu, ini cache artifact script copy-pyodide. Koreksi dari audit sebelumnya.

**Perbaikan:**
- [ ] Dokumentasikan 2 mode jalan: `frontend-only (localStorage)` vs `dengan backend Go :8080`. Jangan biarkan user mengira backend wajib.
- [ ] Rapikan build: satu `distDir` baku, `.gitignore` untuk `.next*`, `https:/` cache, dan `node_modules`. Hapus folder build yang ke-commit.
- [ ] Perbaiki error UX backend: bedakan `offline`, `timeout 5s`, `rejected 4xx/5xx` dengan pesan + tombol retry, bukan sekadar `notice`.
- [ ] Acceptance: `npm run build` bersih tanpa env khusus; `git status` tidak menampilkan folder build/cache.

## 7. Pilihan Strategi (View Berlawanan Arah)

Lu tidak bisa menang lawan ketiganya sekaligus:
- Tinkercad (Autodesk, duit infinite, fokus anak sekolah),
- Wokwi (komunitas + akurasi AVR/ESP32),
- Gazebo/Webots (fisika robotik beneran).

**Pilih satu untuk 3 bulan ke depan:**
- **Opsi A — Alat ajar (recommended):** fokus blink/button/breadboard super polished. Bekukan robotik/AI sebagai eksperimen. Menang di kurikulum Indonesia.
- **Opsi B — Lab robotik mahasiswa:** buang Arduino blink, fokus EduArm/AeroArm/Rover + Python + Colab. Menang di kinematik yang memang sudah kuat.
- **Opsi C — Menyerah elegan:** embed Wokwi untuk AVR akurat, lu fokus ke 3D + kurikulum + bahasa Indonesia.

Tanpa pilihan ini, semua perbaikan P0-P2 di atas cuma nambal kapal bocor.

## 8. Koreksi Hiperbola Audit Sebelumnya (Biar Fair)

1. `tests/run.cjs` ternyata thorough untuk logic — kritik "nol test serius" salah. Yang benar: "test logic kuat, test sistem/E2E nol".
2. Folder `https:/` bukan typo — itu cache CDN pyodide. Kritik "typo" ditarik.
3. "Slideshow di HP" belum diukur profiler — butuh `bundle-analyzer` + Lighthouse sebelum klaim pasti.
4. "Maintainability nol" itu subjektif — pemisahan worker/store sudah ada niat baik, cuma kebablasan monolit.
5. `docs/` lu sebenarnya niat (01-07 + INVENTARIS + RENCANA revisi 1.16). Masalahnya `frontend/README.md` tidak mencerminkan kedewasaan itu.

## 9. Checklist Prioritas 2 Minggu (Minimal Selamat) — **SUPERSEDED, lihat §11**

> Catatan Revisi 2: urutan di bawah salah (split god-file diletakkan sebelum E2E/CI), estimasi 2 minggu terlalu agresif, dan item "rapikan `.gitignore`" sebagian ternyata sudah beres (lihat §10.2). Simpan sebagai rekam jejak, jangan dieksekusi apa adanya.

- [ ] P0: label fidelity di UI + sunat katalog default ke 10 tipe
- [ ] P0: tulis ulang `frontend/README.md`
- [ ] P0: pecah `page.tsx` → 5 komponen + 1 hook shortcut
- [ ] P1: baseline bundle-analyzer + lazy-load pyodide/onnx/mediapipe/monaco
- [ ] P1: tambah 5 E2E Playwright + CI hijau
- [ ] P2: lazy inspector panel + rapikan shortcut
- [ ] P2: rapikan `.gitignore` build/cache + dokumentasikan mode offline vs backend
- [ ] Strategi: pilih Opsi A/B/C dan tulis di `docs/RENCANA.md` sebagai keputusan, bukan wacana

## 10. Verifikasi Independen & Koreksi Tambahan (Revisi 2, 6 Okt 2026)

Diperiksa ulang langsung ke repo (statis: `wc -l`, `git ls-files`, `git show`, baca baris per baris).

**Klaim awal yang terkonfirmasi persis:**
- Semua angka baris cocok: `page.tsx` 1210, `simulation.worker.ts` 1627, `useSimulatorStore.ts` 471, `CodeEditorPanel.tsx` 1240, `globals.css` 1603, `ComponentRegistry.ts` 690.
- Disclaimer di `ComponentRegistry.ts:188/220/243/396` (Siglent, LM2596, TP4056, LED) ada di baris yang disebut, redaksi cocok.
- `package.json` deps cocok persis (termasuk `onnxruntime-web@1.30.0`, `pyodide`, `@mediapipe/tasks-vision`, `rapier3d-compat`, `@monaco-editor/react`, `esptool-js`); `predev/prebuild/prestart` memang selalu copy aset pyodide+onnx.
- `OnnxInferencePanel` (`page.tsx:1199`) dan `MobileWarningBanner` (`:1207`) di-mount tanpa kondisi; guard shortcut di `:213-220`; fetch backend `AbortSignal.timeout(5000)` dengan error cuma jadi `notice`; worker import semua `vhal/*` di baris 1-21.
- `docs/INVENTARIS.md` (rev 14) ternyata sudah matang: 34 tipe + kolom "Batas yang masih ada" per komponen. Pekerjaan dokumen P0-1 lebih kecil dari estimasi awal — gap-nya di UI, bukan di docs.

**Koreksi (menambah daftar §8):**
1. `tests/run.cjs` itu **1779 baris**, bukan "900+" — test logic lebih tebal dari yang diakui.
2. Temuan P2-2 sebagian salah: `.gitignore` frontend **sudah ada** dan sudah cover `.next/`, `.next-production/`, `node_modules`, `public/pyodide|onnxruntime`, `*.tsbuildinfo`, `next-env.d.ts`. `git ls-files` di submodule: **nol build artifact ter-commit**. Sisa masalah: folder kosong `frontend/https:` (0 byte, untracked) + commit parent `9408385` berpesan misleading ("add frontend dependencies and build artifacts" padahal bump pointer submodule + docs). P2-2 jadi masalah commit-hygiene, bukan repo-hygiene.
3. §4 bilang fuzz "tinggal otomatisasi CI" — **tidak ada CI sama sekali** (tak ada `.github/`). Itu pipeline greenfield; item ini di-underestimate di checklist lama.
4. `frontend` adalah **git submodule** (149 file ter-track, status bersih) — berdampak ke strategi CI & commit; audit awal memperlakukannya sebagai bagian repo utama.

## 11. Rencana Implementasi FINAL (menggantikan §9)

Prinsip: **guardrail sebelum refactoring.** Split god-file adalah item paling berisiko; tanpa E2E itu medan ranjau. Gate-nya harus `95 checks (run.cjs) + 5 E2E`, bukan cuma `npm test` hijau — `run.cjs` tidak menyentuh UI/worker-threading.

### Sprint 0 — Keputusan (belum ada baris kode)
- [ ] Pilih Opsi A/B/C (§7), tulis sebagai **keputusan** di `docs/RENCANA.md`: satu paragraf "keputusan + alasan + apa yang berhenti dikerjakan".
- [ ] **Rewrite** status tabel lama di `RENCANA.md` (baseline 13 Sept; addendum-nya sendiri ngaku "tidak menggambarkan branch terbaru") — jangan nambah di belakang.
- [ ] Catatan Opsi C: engine simulator AVR Wokwi dipublikasikan di npm (`wokwi-arduino`, open source) — Opsi C feasible di level **engine**, bukan embed situs; artinya menerima model komponen & scope akurasi orang lain. Verifikasi lisensi dulu sebelum dijadikan keputusan.
- Acceptance: `docs/RENCANA.md` berisi keputusan yang bisa langsung dicetak di onboarding kontributor.

### Sprint 1 — Guardrail & Quick Win (sebelum split)
- [ ] Tulis ulang `frontend/README.md` (P0-3).
- [ ] Fidelity katalog: tambah field `fidelity: "ideal" | "quasi-static" | "datasheet"` + `experimental: true` di `ComponentRegistry`; label `Edukasi-Ideal` / `Tervalidasi-Datasheet` di card komponen; non-P0 (di luar 10 tipe) disembunyikan di balik flag. **Flag, bukan delete** — reversible, tidak buang kerja.
- [ ] `docs/INVENTARIS.md`: kolom eksplisit `Tingkat Fidelity` — kontennya sudah ada di kolom "Batas yang masih ada", tinggal sinkronkan, jangan tulis ulang.
- [ ] CI GitHub Actions **dari nol**: `npm ci` → `tsc --noEmit` → `eslint --max-warnings 0` → `npm test` → `next build`.
- [ ] `next/bundle-analyzer`: baseline JS/WASM first-load dicatat di docs. Target "-30%" baru sah setelah baseline ada.
- Acceptance: CI hijau di branch; kartu 10 tipe katalog default semua punya label fidelity; file baseline bundle ter-commit.

### Sprint 2 — Test & UX
- [ ] `vitest` untuk lib murni (solver, parser, kinematika, vhal). `run.cjs` (1779 baris) dipertahankan sebagai legacy logic gate, migrasi bertahap.
- [ ] `playwright`: 5 E2E — load example blink → run → serial ON/OFF; wiring invalid ditolak; save/load JSON roundtrip; flash panel tanpa serial tidak crash; 1 alur tambahan pilihan.
- [ ] Fuzz ringan `parseProject`/`validWire`/`buildElectricalConnectivity` (pin MISSING, wire duplikat — materialnya sudah di `run.cjs:592-606`) → otomatis di CI.
- [ ] Inspector lazy: `React.lazy` per `typeId`, panel non-aktif di-unmount (cek DevTools: buka inspector servo tidak ikut mount ONNX/gateway).
- [ ] Shortcut pindah ke `useWorkspaceShortcuts.ts` + tampil di Help modal; single-letter tanpa modifier mati saat editor/input fokus.
- Acceptance: 95 checks + 5 E2E hijau di CI; acceptance §5 terpenuhi (tidak ada shortcut nyasar saat ngetik di Monaco).

### Sprint 3 — Split God-File & Bundle
- [ ] `page.tsx` → `WorkspacePage.tsx` + `LibraryPanel.tsx` + `WiringPanel.tsx` + `InspectorPanel.tsx` + `SceneSection.tsx` + `useWorkspaceShortcuts.ts`.
- [ ] `simulation.worker.ts` → per domain (`circuitStep`, `motorStep`, `batteryPowerStep`, `i2cSensorStep`, `roboticsStep`); worker tinggal dispatcher.
- [ ] `useSimulatorStore.ts` → slice; `CodeEditorPanel.tsx` → `CodeEditorShell` + `LibraryCatalog` + `HardwareFlashPanel` + `MonacoWrapper`.
- [ ] **Disiplin: extract-and-move murni, satu ekstrak per commit, test hijau di setiap commit.** Potong di batas domain (panel/step), bukan di batas 500 baris.
- [ ] Lazy-load: `pyodide`, `onnxruntime-web`, `mediapipe`, `rapier`, `monaco` via dynamic import + loading state (hanya saat fitur dipakai); pisah route `/` (rangkaian dasar) vs `/lab` (robotik+AI).
- Acceptance: tidak ada file `src/` >500 baris kecuali generated (**budget**, bukan goal — kriteria sesungguhnya: tiap potongan bisa dites mandiri); `tsc` + `eslint` + 95 checks + 5 E2E hijau; delta bundle vs baseline dicatat di docs.

## 12. Batas Verifikasi (Klausul Jujur)

- Revisi 2 memverifikasi yang **statis**: angka baris, isi file, git, `package.json`, konfigurasi.
- **Terverifikasi dinamis (6 Okt 2026, mesin lokal)**: `npm test` → **95 checks passed** (cakupan nyata: A4988/UVLO, DC-DC, baterai SoC, charger CC/CV, balancing LiPo, proteksi reverse-polarity, termal RC, V-HAL RGB-D/LiDAR/IMU, parser sketch, kapasitor RC, rigid body & fixed joint Rapier); `npx tsc --noEmit` → bersih (exit 0); `npm run build` → sukses (Next 16.2.6). Catatan: build Next 16 ini jalan di **Turbopack**, sedangkan `docs/README.md` rev 1.16 menyebut "production build Webpack lulus" — klaim Webpack-nya belum dicek ulang; route yang ada saat ini cuma `/` (belum ada `/lab`).
- **Belum diverifikasi**: Lighthouse/mobile di perangkat nyata (butuh browser + HP; sesuai §8.3), dan klaim kinematik (FK `1.9e-16m`) — `npm run test:kinematics-reference` butuh `pip install -r requirements-kinematics-reference.txt` yang belum ada di environment ini.
- "-30% bundle" dan "Lighthouse mobile tidak merah" adalah **stretch goal**: ukur dulu, jangan dijanjikan dulu.
- Differential-test vs hardware (P1-2) **tidak masuk 3 sprint**: butuh hardware nyata. Scope minimal blink/PWM/tombol (logic analyzer/MCU referensi); DHT11/HC-SR04 hanya jika sensor tersedia; jika tidak, sitasi datasheet + label "belum diukur". Jangan dipalsukan jadi hasil ukur.

---
*File ini adalah cermin, bukan piagam. Kalau rencana final di §11 tidak dikerjakan, kritik pedas ini akan tetap valid 6 bulan dari sekarang.*
