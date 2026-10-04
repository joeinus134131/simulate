# Inventaris dan tingkat dukungan

Audit awal: 11 entri, 7 bentuk fisik. Lima warna jumper berbagi geometri dan perilaku. Tidak ada ESP32, motor, LCD, sensor suhu atau komponen lain di katalog awal. Dokumen ini membedakan bentuk fisik, konektivitas, dan emulasi program.

| Entri katalog | Pin / model fisik saat ini | Perilaku sekarang | Batas yang masih ada |
|---|---|---|---|
| Arduino Uno R3 DIP, A000066 | PCB 68.6 × 53.4 × 1.6 mm; pitch 2.54 mm; header 8+10 digital dan 8+6 power/analog; USB-B, jack, DIP28, reset, regulator, kapasitor, dua ICSP. 44 terminal fisik: D0–13, A0–5, NC, IOREF, RESET, 3V3, 5V, GND1–3, VIN, AREF, SDA/SCL, 6 main ICSP, 6 USB ICSP. | Satu board, 5V/3.3V ideal; GPIO dengan resistansi output 25Ω, pullup 30kΩ; PWM rata-rata pada 3/5/6/9/10/11; ADC 0–1023. LED builtin D13. Tombol reset memulai ulang saat running. | Bukan AVR instruction/cycle emulator. Main ICSP signal hanya alias GPIO, tidak SPI. USB ICSP, AREF, NC, Vin, RESET eksternal hanya terminal fisik tanpa peripheral/regulator. USB/UART fisik, interrupt, I2C, SPI tidak diemulasi. Label mikro, trace PCB, logomark, lubang/cetakan konektor dan komponen kecil belum salinan CAD penuh. |
| Breadboard BB400 | 84 × 54.3 × 8.5 mm; 300 lubang kelompok 5, 100 lubang rail (4 ×25). Geometri lubang instanced; nomor/row dan rail berwarna. | a–e dalam kolom terhubung, f–j terpisah; tiap rail kontinu dan tidak otomatis diberi catu. | Audit awal 420 lubang dikoreksi ke 400 menurut varian BB400; ekspor eksperimen lama yang memakai rail indeks 25–29 perlu disambung ulang ke 0–24. Kontak pegas internal, pengunci samping dan cetakan logo belum lengkap. |
| Red LED, Kingbright WP7113ID | Lensa Ø5 mm, flange Ø5.9 mm, kaki A/C pitch 2.54 mm; panjang kaki nominal A27/C25.5 mm; sisi flange katoda. | Model DC piecewise: Vf 1.8V + resistansi dinamis 10Ω, brightness dari arus; polaritas dan return path wajib; arus >20mA diberi peringatan. | Ini perkiraan pendidikan, bukan kurva I–V/optik datasheet. Bentuk flat/fillet dan internal leadframe belum identik; tidak ada suhu, toleransi, breakdown atau kerusakan. |
| Push Button, Omron B3F-1000 | Body 6×6 mm, tinggi total 4.3 mm, plunger ivory Ø3.5 mm; pitch terminal 6.5×4.5 mm. 1a/1b satu kelompok, 2a/2b kelompok lain. | Tekan cap atau tombol properti: hubung antar kelompok. Lepas memutus. INPUT_PULLUP bekerja melalui rangkaian. | Nama pin simulator bukan nomor pabrikan. Tidak ada bounce, gaya tekan atau resistansi kontak; tekukan kaki dan emboss disederhanakan. |
| Potentiometer, Alps RK09K1130A6S | Body 9.8 mm, shaft datar 15 mm; tiga terminal 1/W/2; indikator putaran. | Dua resistor jumlah 10kΩ, wiper mengikuti slider 0–100%; harus disambung catu dan GND untuk ADC. | Tab snap-in, kaki, kontur shaft/casing belum CAD exact. Minimum setiap lengan 1Ω untuk stabilitas numerik; toleransi dan noise tidak dimodelkan. |
| Resistor 220Ω, acuan CFR-25 | Axial beige, pita merah-merah-cokelat-emas, L/R. Diameter body 2.4 mm, panjang model 6 mm. | Resistansi linear 220Ω; membatasi arus sesuai rangkaian. | Acuan nominal CFR-25 panjang 6.3 mm masih selisih 0.3 mm pada model; kaki ditampilkan terpotong 6 mm setiap sisi. Tidak ada toleransi/suhu/daya rusak. |
| Jumper Red | Male–male, housing hitam + kontak persegi, kabel merah; L/R. | Kontinuitas ideal dua ujung. | Acuan keluarga ZW-MM-10; rute fleksibel dan panjang kurva belum dikalibrasi persis 100 mm. |
| Jumper Black | Geometri sama, hitam; L/R. | Kontinuitas ideal. | Sama. |
| Jumper Blue | Geometri sama, biru; L/R. | Kontinuitas ideal. | Sama. |
| Jumper Green | Geometri sama, hijau; L/R. | Kontinuitas ideal. | Sama. |
| Jumper Yellow | Geometri sama, kuning; L/R. | Kontinuitas ideal. | Sama. |

## Dukungan .ino dan serial
Semua komponen memakai wiring solver yang sama; .ino berjalan pada Uno dan membaca/menulis pin yang benar-benar terhubung. Tidak ada pemetaan LED/sensor palsu menurut urutan komponen.

Parser terpisah (tanpa eval/new Function) mendukung deklarasi scalar, setup/loop, fungsi dengan parameter/return, if/else, while, for, break, assignment, postfix ++/--, operator aritmetika/logika/bit, literal string/char, komentar. API: pinMode, digitalWrite/Read, analogWrite/Read, delay, millis/micros, map, constrain, min/max/abs, Serial.begin/available/read/peek/print/println/write. Serial.read mengembalikan byte atau -1; write menulis byte, input UTF-8, pilihan tanpa akhiran/newline/CRLF. Serial.begin diperlukan untuk print/write; UI menghalangi RX ketika baud berbeda. Baud tidak mensimulasikan timing UART fisik.

Semantik scalar masih angka JavaScript: pembagian integer dan overflow C++ tidak dimodelkan; const belum readonly. Tidak mendukung #include/#define, array, pointer, kelas, struct, cast, switch, ternary, prefix increment, overload, variadic atau library. Sintaks/fungsi tidak didukung menghasilkan error, bukan diabaikan. Serial.print format basis 2/8/16 tersedia; presisi float ala Arduino belum diimplementasi. Parser bukan compiler C++ lengkap.

Worker bisa diterminasi langsung saat Stop/restart sehingga delay lama tidak menghidupkan kembali rangkaian. Jeda mempertahankan state program. Loop dibatasi 50.000 operasi per loop/delay, kedalaman panggilan 32, RX 4096 byte, sketch 64 KB. Log maksimal 500 chunk (tiap chunk maksimal 16 KB). Maksimum proyek 100 komponen/500 kabel; solver 256 net aktif. Floating net memakai referensi lemah deterministik, bukan noise fisik.

## Rujukan primer
- [Arduino Uno R3](https://docs.arduino.cc/hardware/uno-rev3/) dan [pinout A000066](https://content.arduino.cc/assets/A000066-full-pinout.pdf).
- [BB400 BusBoard](https://www.busboard.com/BB400): 400 titik, 84×54.3×8.5 mm. Menggantikan geometri katalog awal 420 titik.
- [Kingbright WP7113ID](https://www.kingbrightusa.com/images/catalog/SPEC/WP7113ID.pdf).
- [Omron B3F](https://components.omron.com/sites/default/files/datasheet_pdf/A070-E1.pdf).
- [Alps RK09K](https://tech.alpsalpine.com/e/products/category/potentiometers/sub/02/series/rk09k/).
- Yageo CFR-25: tautan katalog awal tidak dapat diambil; verifikasi drawing lengkap masih diperlukan sebelum mengklaim model identik.
- [BusBoard ZW-MM-10](https://www.busboard.com/ZW-MM-10), keluarga jumper male–male.

## Referensi Diode
[withdiode.com](https://www.withdiode.com/) ditemukan sebagai situs yang menjelaskan build/program/simulate hardware di browser. Saat diperiksa langsung 13 September 2026 situs menampilkan **Deployment Paused**. Karena itu fitur wiring/code/serial Diode tidak diuji dan tidak dijadikan bukti kemampuan runtime. Arah UX proyek tetap satu ruang kerja rangkaian + program + serial, tanpa menyalin aset situs itu.

## Penerimaan bentuk fisik
Ukuran utama dan kontrak pin telah diperbaiki, tetapi kriteria pengguna **sama dengan perangkat nyata** belum boleh dinyatakan selesai: detail yang disederhanakan di tabel masih terbuka. Model ini belum lulus overlay CAD/foto kalibrasi terhadap seluruh varian. Tampilan atas/depan/perspektif dan panel rujukan tersedia untuk evaluasi lanjutan.
