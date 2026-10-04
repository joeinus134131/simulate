# Rencana perombakan ArduSim

## Audit baseline (13 September 2026)
- Frontend Next 16 / React 19, Three/R3F, Zustand; backend Go/Fiber. Tidak ada AGENTS.md pada proyek.
- Katalog aktual: Uno R3, breadboard 30 kolom, LED merah, tombol 4 kaki, potensiometer, resistor 220Ω, jumper merah/hitam/biru/hijau/kuning (11 entri, 7 bentuk).
- Geometri awal tersedia untuk semua, tetapi header Uno berbeda dari definisi pin dan tidak punya target wiring. Breadboard menerapkan posisi pin dua kali dan lubang berada di tengah badan. Target pin beradius 1.5 jauh melebihi jarak antar-pin 0.5.
- Engine regex/new Function berjalan pada UI, memetakan D13 ke semua LED dan D2/A0 ke sensor pertama tanpa kabel. millis tidak mengikuti delay; stop tidak membatalkan delay. Serial hanya echo.
- Backend health tersedia, proyek dan AI adalah mock. Belum ada penyimpanan frontend atau tes.
- Perubahan pengguna sudah ada di frontend package/lock, layout, CSS, page dan tema; dipertahankan.

## Tahap dan kriteria penerimaan
| Tahap | Hasil yang dievaluasi | Penerimaan | Dependensi | Status |
|---|---|---|---|---|
| 1 Audit/baseline | Inventaris dan baseline TypeScript/build | Masalah aktual dicatat, perubahan pengguna dipertahankan | Kode awal | Selesai |
| 2 Kontrak & backend | Definisi pin tunggal, validasi proyek/kabel, API simpan/buka JSON berversi | Endpoint palsu dihapus; round-trip tidak kehilangan code/wiring; input invalid ditolak | 1 | Dikerjakan |
| 3 Model/interaksi | Uno proporsional dan header berlabel; semua target cocok geometri; wiring klik dua pin + daftar terminal | Rotasi mempertahankan ujung kabel; tidak ada pin tumpang tindih/dobel transform; hapus kabel | 2 | Menunggu |
| 4 Runtime | Interpreter subset .ino dalam worker; solver DC, input sensor, Serial RX/TX, stop/restart | LED harus punya rangkaian tertutup; PWM, pullup, ADC, echo serial; loop macet tidak membekukan UI | 2 | Menunggu |
| 5 Ringan/mudah | Render sesuai perubahan, batas log/ukuran, editor lokal, contoh siap pakai, simpan lokal/backend | UI responsif; tidak tergantung CDN untuk editor/HDR; status/error jelas | 3–4 | Menunggu |
| 6 Validasi | Tes solver/parser/API, build, pemeriksaan UI, panduan evaluasi Indonesia | Hasil aktual dan keterbatasan tercatat, bukan klaim lulus tanpa tes | 2–5 | Menunggu |

## Kontrak dukungan yang dituju
Simulasi pendidikan DC kuasistatik pada satu Uno virtual dengan catu ideal 5V/3.3V, digital/PWM rata-rata, analog 10-bit; resistor linear, LED model tegangan maju perkiraan, pot pembagi resistif, tombol sakelar ideal, breadboard/jumper kontinuitas. Tidak mencakup SPICE/transien, suhu/kerusakan, toleransi komponen, CPU AVR cycle-accurate, library C++ pihak ketiga, I2C/SPI/UART fisik, USB atau regulator Vin. Fitur yang tidak dimodelkan harus dijelaskan, bukan berpura-pura berjalan.

## Referensi fisik
Uno R3: https://docs.arduino.cc/hardware/uno-rev3/ dan https://content.arduino.cc/assets/A000066-full-pinout.pdf ; dimensi PCB 68.6 × 53.4 mm, pitch header 2.54 mm. Skala adegan 1 unit ≈ 5 mm. Komponen generik memiliki variasi kemasan; bukan CAD produksi bersertifikasi.
