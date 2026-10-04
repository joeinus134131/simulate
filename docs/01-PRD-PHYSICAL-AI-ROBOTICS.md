# Product Requirements Document (PRD)
## Platform Simulasi 3D Physical AI & Robotika Kerakyatan

- **Kode Proyek**: `TridiArduSim-PhysicalAI`
- **Versi Dokumen**: 1.0.0
- **Status**: Disetujui untuk Implementasi
- **Target Branch**: `feature/physical-ai-robotics-simulation`
- **Terkait dengan**: [RENCANA.md](file:///Users/user/Projects/simulate/docs/RENCANA.md), [INVENTARIS.md](file:///Users/user/Projects/simulate/docs/INVENTARIS.md)

---

## 1. Latar Belakang & Pernyataan Masalah

### 1.1 Kesenjangan Aksesibilitas Hardware di Indonesia & Negara Berkembang
Perkembangan pesat **Physical AI** (Embodied AI, robotika otonom, Vision-Language-Action models) saat ini terkunci di balik dinding biaya hardware yang sangat tinggi:
- **NVIDIA Jetson Orin Nano / Nano Dev Kit**: Dibanderol berkisar **Rp 8.000.000 – Rp 15.000.000+** (belum termasuk pajak impor, carrier board, dan aksesoris kamera MIPI).
- **Raspberry Pi 5 / 4 (8GB)**: Berkisar **Rp 1.500.000 – Rp 2.500.000** (hanya board, belum sensor LiDAR/Depth Camera).
- **Sensor Industri & Aktuator Presisi**: Smart servo Dynamixel berkisar Rp 800.000 – Rp 3.000.000 per unit; kamera Intel RealSense Depth berkisar Rp 6.000.000 – Rp 10.000.000; LiDAR 2D RPLiDAR berkisar Rp 1.800.000 – Rp 3.500.000.

Bagi mahasiswa, pelajar SMK, dan insinyur perintis dari kalangan menengah ke bawah di Indonesia, harga-harga di atas merupakan penghalang absolut. Akibatnya, kurikulum dan eksperimen lokal kerap tertinggal pada proyek sederhana (LED berkedip, sensor jarak statis) tanpa pernah menyentuh esensi robotika modern: **Kinematika Ruang $SE(3)$, Inverse Kinematics, Kontrol Loop Tertutup Sensorimotor, dan Kebijakan AI Fisik**.

### 1.2 Keterbatasan Alat Simulasi Konvensional
- **NVIDIA Isaac Sim / Isaac Gym**: Membutuhkan PC workstation dengan GPU NVIDIA RTX kelas desktop (minimal RTX 3070/4070 VRAM 8–16GB, harga PC > Rp 25.000.000), instalasi Linux kompleks, dan tidak ramah web.
- **Gazebo / ROS 2**: Sangat membebani CPU, instalasi dependensi rumit bagi pemula, dan visualisasi GUI kurang intuitif untuk rekayasa elektronik tingkat komponen.
- **Wokwi / Tinkercad**: Bagus untuk pengenalan dasar mikrokontroler, tetapi **tidak memiliki** simulasi fisika robot multi-joint, kinematika modern, virtual kamera/LiDAR untuk AI, maupun ekspor perakitan mekanik 3D fungsional.

---

## 2. Visi & Misi Produk

> **Visi**: Membuka gerbang riset dan rekayasa Physical AI serta Robotika Modern untuk setiap anak bangsa tanpa diskriminasi ekonomi.

> **Misi**:
> 1. Membangun platform simulasi 3D berbasis Web (Next.js, Three.js/R3F, Rapier3D WASM) yang dapat berjalan lancar di browser laptop standar, Chromebook, maupun tablet.
> 2. Menyediakan **Virtual Hardware Abstraction Layer (V-HAL)** yang mensimulasikan lingkungan komputasi Single Board Computer (ala Jetson Nano / Raspberry Pi) dan mikrokontroler (Arduino / ESP32) secara virtual.
> 3. Mengintegrasikan algoritma robotika modern berbasis teks standar industri (*Lynch & Park - Modern Robotics*): Formula Product of Exponentials (PoE), Jacobian Damped Least Squares (DLS), dan FABRIK.
> 4. Menyediakan **Pipeline Sim-to-Real Kerakyatan**: Menjembatani desain virtual menuju realisasi fisik berbiaya super-murah (< Rp 300.000) menggunakan ESP32/Pico + servo mikro + cetak 3D lokal, dilengkapi standar soldering & assembly mechanical best-practices.

---

## 3. Profil Pengguna (User Personas)

| Persona | Profil & Kendala | Nilai yang Didapatkan dari Platform |
|---|---|---|
| **Budi (20 thn)**<br>Mahasiswa Teknik Elektro / Mekatronika | Hanya memiliki laptop kantor/kuliah bekas (Intel i3/i5 iGPU). Tidak mampu membeli kit robotika atau Jetson Nano seharga UKT 2 semester. | Dapat menjalankan simulasi lengan robot 6-DOF, memprogram Inverse Kinematics via JavaScript/Python, dan melihat output visual real-time 60 FPS tanpa PC gaming. |
| **Rian (28 thn)**<br>Maker & Insinyur Hardware Perintis | Ingin merancang lengan robot perakitan PCB mandiri. Sering gagal karena salah solder, salah memilih driver motor, atau salah hitung torsi sehingga servo terbakar. | Melakukan *virtual prototyping*: uji coba rangkaian driver L298N/A4988, simulasi panas/solder joints, validasi kinematika, lalu ekspor file STL 3D print dan Gerber PCB siap cetak. |
| **Siti (25 thn)**<br>Peneliti / AI Engineer Pemula | Tertarik pada Embodied AI (Policy Learning & Computer Vision untuk Robotika), tidak memiliki GPU lokal untuk training model. | Memanfaatkan arsitektur hybrid: Platform 3D di browser mengirim streaming virtual camera ke **Google Colab (GPU T4 gratis)** via WebSocket, dan menerima kembali trajectory prediksi AI ke robot virtual. |

---

## 4. Ruang Lingkup Fitur & Matriks Prioritas

### 4.1 Prioritas P0 (Fondasi Kritis - Milestone 1)
- **Kinematika Robotika Modern (Lynch & Park Standard)**:
  - Implementasi aljabar grup Lie $SE(3)$ dan $SO(3)$ di TypeScript/WASM.
  - Forward Kinematics (FK) berbasis Product of Exponentials (PoE).
  - Inverse Kinematics (IK) Numerik: Damped Least Squares (DLS) & Jacobian Pseudo-Inverse dengan batasan sudut sendi (*joint limits*).
  - Algoritma FABRIK (*Forward And Backward Reaching Inverse Kinematics*) untuk kontrol drag-and-drop end-effector interaktif di canvas 3D.
  - Model robot referensi: 3-DOF Arm (Planar/Spatial), 6-DOF Articulated Arm (ala Universal Robots UR5 / Puma), dan 4-Wheel Differential Drive Mobile Robot.
- **Virtual Hardware Abstraction Layer (V-HAL)**:
  - Abstraksi pin I/O virtual untuk motor kontrol: PWM resolusi 8-bit & 16-bit, A/B Encoder feedback (quadrature pulses), STEP/DIR signals untuk stepper.
  - Runtime interpreter motor multi-axis terintegrasi dengan loop simulasi fisika.
- **Sensor Sintetis Dasar**:
  - Virtual RGB-D Camera (Color frame + Float32 depth texture buffer).
  - Virtual 2D Planar LiDAR (Raycasting berbasis Three-Mesh-BVH 360°, 360–720 sampel per putaran).
  - Virtual 6-Axis IMU (Akselerometer 3D + Giroskop 3D dengan model noise Gauss deterministik).

### 4.2 Prioritas P1 (Physical AI Hybrid & Advanced Electronics - Milestone 2)
- **Hybrid AI Inference Gateway (Demokratisasi Komputasi)**:
  - In-Browser Edge Inference: Integrasi **ONNX Runtime Web** dengan akselerasi WebGPU/WebAssembly untuk model klasifikasi/deteksi objek ringan (MobileNetV4, YOLOv8-nano INT8).
  - Zero-Cost Cloud Offloading: Konektor WebSocket/WebRTC 1-klik menuju **Google Colab / Kaggle Notebook** (free GPU T4/P100), memungkinkan eksekusi Vision-Language-Action (VLA) atau Reinforcement Learning di cloud sementara dunia 3D berjalan di browser lokal.
- **Simulasi Elektronika Daya & Aktuator Lanjut**:
  - Driver Motor DC H-Bridge L298N (Drop tegangan $V_{sat}$, disipasi termal, logika arah/enable).
  - Stepper Driver A4988 / DRV8825 (Microstepping 1/16 & 1/32, kalkulasi arus referensi $V_{ref}$).
  - Model Elektromekanik DC Motor (Back-EMF, resistansi kumparan, inersia rotor, torsi stall vs beban aktual).
  - Simulasi Baterai (Penurunan tegangan / *voltage sag* saat motor menarik arus puncak/stall, mencegah brownout mikrokontroler).
- **Modul Virtual Soldering & Breadboard-to-PCB Transition**:
  - Simulasi solder interaktif: transfer panas ujung solder ke pad, lelehan timah, pembentukan meniscus wetting, dan deteksi otomatis kesalahan solder (*cold joint*, *solder bridge*, *overheated PCB pad*).

### 4.3 Prioritas P2 (Fabrikasi & Sim-to-Real Pipeline - Milestone 3)
- **Ekspor Desain Fungsional 3D & Fabrikasi Nyata**:
  - Ekspor model mekanik assembly ke format **STL / STEP** siap cetak 3D di jasa cetak lokal atau printer 3D komunitas.
  - Ekspor netlist sirkuit ke format skematik KiCad / Gerber PCB untuk pemesanan fabrikasi PCB lokal atau JLCPCB.
  - Bill of Materials (BOM) otomatis dengan estimasi harga real-time berbasis marketplace Indonesia (Tokopedia / Shopee).
- **Physical Sim-to-Real Blueprint (< Rp 300.000)**:
  - Panduan perangkat keras nyata menggunakan ESP32-WROOM / ESP32-CAM + servo mikro SG90 / MG996R.
  - Sinkronisasi firmware langsung melalui WebSerial API (tanpa install Arduino IDE manual).

---

## 5. Kebutuhan Non-Fungsional (Non-Functional Requirements)

1. **Performa Rendering Web**:
   - Target frame rate: **60 FPS** pada resolusi Full HD (1080p) di GPU terintegrasi (Intel Iris Xe, AMD Radeon Vega 7/8, Apple Silicon M1/M2/M3).
   - Penggunaan memori browser (Heap Memory) tidak melebihi **450 MB** pada scene kompleks (robot 6-DOF + sirkuit penuh + 100 komponen).
2. **Keterpisahan Komputasi (Architecture Decoupling)**:
   - Loop simulasi fisika & kinematika berjalan pada **Web Worker** terpisah (dedicated thread) agar perhitungan numerik berat tidak memblokir antarmuka UI (Zero UI Freeze).
3. **Determinisme & Stabilitas Numerik**:
   - Solver kinematika IK wajib konvergen dalam waktu $< 5 \text{ ms}$ per target pose.
   - Penanganan singularitas harus mulus menggunakan DLS tanpa ledakan kecepatan sendi (*no wild joint flipping*).
4. **Portabilitas & Aksesibilitas**:
   - Tidak memerlukan instalasi software lokal (Zero Local Install); dapat diakses 100% via peramban modern (Chrome, Edge, Firefox, Safari) yang mendukung WebGL 2.0 / WebGPU dan WebAssembly.

---

## 6. Metrik Keberhasilan (Success Metrics)

1. **User Empowerment**: Pengguna dapat merancang, mensimulasikan, dan memprogram lengan robot 6-DOF lengkap dengan kendali IK dalam waktu $< 15$ menit sejak membuka browser.
2. **Cost Reduction**: Mengurangi biaya barrier-to-entry riset Physical AI dari Rp 15.000.000 (biaya Jetson + robot fisik komersial) menjadi **Rp 0** (simulasi penuh) atau **< Rp 300.000** (versi fisik mikrokontroler kerakyatan).
3. **Sim-to-Real Parity**: Kode kontrol gerak dan trajectory yang diuji pada simulator virtual dapat diekspor langsung ke mikrokontroler fisik tanpa perubahan arsitektur logika.
