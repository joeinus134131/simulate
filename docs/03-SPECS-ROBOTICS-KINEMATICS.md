# Spesifikasi Teknis: Kinematika & Dinamika Robotika Modern
## Penerapan Landasan Teori Lynch & Park (Modern Robotics) pada Web 3D

- **Kode Proyek**: `TridiArduSim-PhysicalAI`
- **Versi Dokumen**: 1.0.0
- **Rujukan Teori**: Kevin M. Lynch & Frank C. Park, *Modern Robotics: Mechanics, Planning, and Control*, Cambridge University Press.
- **Terkait dengan**: [01-PRD-PHYSICAL-AI-ROBOTICS.md](file:///Users/user/Projects/simulate/docs/01-PRD-PHYSICAL-AI-ROBOTICS.md), [02-ARCHITECTURE-SYSTEM-DESIGN.md](file:///Users/user/Projects/simulate/docs/02-ARCHITECTURE-SYSTEM-DESIGN.md)

---

## 1. Landasan Geometri: Aljabar & Grup Lie $SE(3)$

Dalam platform simulasi ini, representasi gerak benda kaku (*rigid-body motion*) tidak menggunakan parameter Euler angle yang rentan terhadap fenomena *Gimbal Lock*, melainkan memanfaatkan formulasi modern berbasis **Grup Lie $SO(3)$** untuk rotasi dan **Grup Lie $SE(3)$** untuk transformasi rigid 3D penuh.

### 1.1 Representasi Rotasi pada $SO(3)$
Matriks rotasi $R \in SO(3)$ memenuhi sifat ortogonalitas $R^T R = I$ dan $\det(R) = +1$.
Vektor kecepatan sudut $\omega = [\omega_x, \omega_y, \omega_z]^T \in \mathbb{R}^3$ dipetakan ke dalam aljabar Lie $so(3)$ menggunakan operator matriks skew-symmetric $[\omega]$:

$$
[\omega] = \begin{bmatrix} 
0 & -\omega_z & \omega_y \\ 
\omega_z & 0 & -\omega_x \\ 
-\omega_y & \omega_x & 0 
\end{bmatrix} \in so(3)
$$

Eksponensial matriks dari koordinat eksponensial $\hat{\omega}\theta$ menghasilkan matriks rotasi melalui **Formula Rodrigues**:

$$
\exp([\hat{\omega}]\theta) = I + \sin\theta [\hat{\omega}] + (1 - \cos\theta) [\hat{\omega}]^2
$$

### 1.2 Transformasi Rigid-Body pada $SE(3)$
Grup Euclidean Khusus $SE(3)$ mendefinisikan posisi dan orientasi gabungan:

$$
T = \begin{bmatrix} R & p \\ 0 & 1 \end{bmatrix} \in SE(3), \quad R \in SO(3), \; p \in \mathbb{R}^3
$$

Sebuah gerak ulir (*twist*) spasial $\mathcal{V} = \begin{bmatrix} \omega \\ v \end{bmatrix} \in \mathbb{R}^6$ memiliki elemen aljabar Lie $[\mathcal{V}] \in se(3)$:

$$
[\mathcal{V}] = \begin{bmatrix} 
[\omega] & v \\ 
0 & 0 
\end{bmatrix} = \begin{bmatrix} 
0 & -\omega_z & \omega_y & v_x \\ 
\omega_z & 0 & -\omega_x & v_y \\ 
-\omega_y & \omega_x & 0 & v_z \\ 
0 & 0 & 0 & 0 
\end{bmatrix}
$$

Matriks Adjoint $[Ad_T]$ mentransformasikan twist dari frame $\{\text{b}\}$ ke frame $\{\text{s}\}$:

$$
[Ad_T] = \begin{bmatrix} R & 0 \\ [p]R & R \end{bmatrix} \in \mathbb{R}^{6 \times 6}, \quad \mathcal{V}_s = [Ad_T] \mathcal{V}_b
$$

---

## 2. Forward Kinematics (FK): Formula Product of Exponentials (PoE)

### 2.1 Mengapa PoE Menggantikan Denavit-Hartenberg (D-H)?
1. **Bebas Singularitas Geometris**: Parameter D-H memiliki singularitas matematis ketika dua sumbu sendi berturut-turut hampir sejajar (pembagian dengan nol pada jarak link). PoE bersifat kontinu mulus di seluruh ruang konfigurasi.
2. **Tanpa Aturan Koordinat Buatan**: Tidak memerlukan penempatan sumbu $x_i$ ortogonal yang rumit pada setiap sambungan link. Cukup menentukan satu frame referensi dasar $\{\text{s}\}$ dan frame end-effector $\{\text{b}\}$.
3. **Formulasi Terpadu Sendi**:
   - Untuk sendi revolute (putar): $\mathcal{S} = \begin{bmatrix} \hat{\omega} \\ -\hat{\omega} \times q \end{bmatrix}$, di mana $q$ adalah sembarang titik pada sumbu putar.
   - Untuk sendi prismatic (geser): $\mathcal{S} = \begin{bmatrix} 0 \\ \hat{v} \end{bmatrix}$, di mana $\hat{v}$ adalah arah gerak linier.

### 2.2 Formulasi Matematis PoE Ruang (Space PoE)
Bila posisi robot pada sudut $\theta_1 = \dots = \theta_n = 0$ berada pada konfigurasi nol (*home configuration*) $M \in SE(3)$, maka posisi end-effector untuk sembarang sudut sendi $\theta$ adalah:

$$
T(\theta) = \exp([\mathcal{S}_1]\theta_1) \cdot \exp([\mathcal{S}_2]\theta_2) \dots \exp([\mathcal{S}_n]\theta_n) \cdot M
$$

Eksponensial dari $[\mathcal{S}]\theta$ dihitung secara analitik:

$$
\exp([\mathcal{S}]\theta) = \begin{bmatrix} 
\exp([\omega]\theta) & G(\theta)v \\ 
0 & 1 
\end{bmatrix}
$$

dengan:

$$
G(\theta) = I\theta + (1 - \cos\theta)[\omega] + (\theta - \sin\theta)[\omega]^2
$$

---

## 3. Inverse Kinematics (IK) & Solver Numerik

Menentukan sudut sendi $\theta = [\theta_1, \dots, \theta_n]^T$ untuk mencapai target pose $T_{sd} \in SE(3)$.

### 3.1 Jacobian Manipulator Ruang & Bodi
Jacobian bodi $J_b(\theta) \in \mathbb{R}^{6 \times n}$ mengaitkan kecepatan sudut sendi $\dot{\theta}$ dengan twist bodi $\mathcal{V}_b$:

$$
\mathcal{V}_b = J_b(\theta)\dot{\theta} = \begin{bmatrix} \mathcal{J}_{b1} & \mathcal{J}_{b2} & \dots & \mathcal{J}_{bn} \end{bmatrix} \dot{\theta}
$$

### 3.2 Algoritma Damped Least Squares (DLS / Levenberg-Marquardt)
Untuk mencegah ledakan kecepatan pada konfigurasi dekat singularitas ($\det(J J^T) \to 0$), platform menggunakan solver DLS dengan faktor redaman adaptif $\lambda$:

$$
\Delta\theta = J^T \left( J J^T + \lambda^2 I \right)^{-1} \mathcal{V}_e
$$

di mana $\mathcal{V}_e = \log(T_{sb}^{-1}(\theta_k) T_{sd})$ adalah twist galat (*error twist*).

**Algoritma Iterasi Numerik**:
```typescript
function solveIK_DLS(
  currentJoints: number[],
  targetPose: Matrix4,
  maxIterations = 50,
  tolerance = 1e-4,
  dampingLambda = 0.05
): { success: boolean; angles: number[] } {
  let theta = [...currentJoints];
  for (let iter = 0; iter < maxIterations; iter++) {
    const currentPose = computePoE(theta);
    const errorTwist = computeMatrixLogarithmSE3(currentPose.inverse().multiply(targetPose));
    
    // Cek konvergensi galat posisi dan orientasi
    if (twistNorm(errorTwist) < tolerance) {
      return { success: true, angles: theta };
    }
    
    const J = computeBodyJacobian(theta);
    const dlsInverse = computeDLSInverse(J, dampingLambda);
    const deltaTheta = multiplyMatrixVector(dlsInverse, errorTwist);
    
    // Perbarui sudut dengan batasan (joint limits clamping)
    for (let i = 0; i < theta.length; i++) {
      theta[i] = clamp(theta[i] + deltaTheta[i], jointLimits[i].min, jointLimits[i].max);
    }
  }
  return { success: false, angles: theta };
}
```

### 3.3 Redundansi Kinematika & Proyeksi Null-Space (Manipulator 7-DOF)
Untuk lengan dengan derajat kebebasan berlebih ($n > 6$, misal Franka Emika Panda atau bahu 3-DOF manusia), solusi DLS dilengkapi vektor kecepatan sekunder $\dot{\theta}_{null}$ yang diproyeksikan ke ruang nol (*null space*) Jacobian tanpa mengganggu gerak end-effector:

$$
\dot{\theta} = J^{\dagger} \mathcal{V} + \left( I - J^{\dagger} J \right) \nabla w(\theta)
$$

Fungsi potensi $w(\theta)$ dirancang untuk **menjauhkan sendi dari batas fisik maksimalnya**:

$$
w(\theta) = -\frac{1}{2n} \sum_{i=1}^n \left( \frac{\theta_i - \bar{\theta}_i}{\theta_{i,\max} - \theta_{i,\min}} \right)^2
$$

### 3.4 Algoritma FABRIK untuk Manipulasi Interaktif 60 FPS
Pada mode pratinjau interaktif (saat pengguna melakukan *drag-and-drop* end-effector menggunakan mouse pada viewport 3D), algoritma **FABRIK (Forward And Backward Reaching Inverse Kinematics)** digunakan sebagai pemecah posisi awal berkecepatan ultra-tinggi ($< 0.5 \text{ ms}$ per siklus), sebelum dihaluskan oleh solver DLS untuk koreksi orientasi $SO(3)$ penuh.

---

## 4. Perencanaan Lintasan (Trajectory Generation)

### 4.1 Polinomial Derajat Lima (Quintic Spline) pada Ruang Sendi
Untuk memastikan robot tidak bergetar saat mulai bergerak dan berhenti, profil lintasan dirancang dengan posisi, kecepatan, dan akselerasi kontinu ($C^2$ continuity):

$$
\theta(t) = a_0 + a_1 t + a_2 t^2 + a_3 t^3 + a_4 t^4 + a_5 t^5
$$

Dengan kondisi batas:
- Posisi: $\theta(0) = \theta_0, \; \theta(T) = \theta_f$
- Kecepatan: $\dot{\theta}(0) = 0, \; \dot{\theta}(T) = 0$
- Akselerasi (Jerk-Free): $\ddot{\theta}(0) = 0, \; \ddot{\theta}(T) = 0$

Solusi koefisien:
$$
a_0 = \theta_0, \quad a_1 = 0, \quad a_2 = 0
$$
$$
a_3 = \frac{10(\theta_f - \theta_0)}{T^3}, \quad a_4 = \frac{-15(\theta_f - \theta_0)}{T^4}, \quad a_5 = \frac{6(\theta_f - \theta_0)}{T^5}
$$

### 4.2 Interpolasi Gerakan Ruang Tugas (Task Space Screw Motion)
Untuk lintasan garis lurus end-effector di ruang 3D, interpolasi dilakukan langsung pada manifold $SE(3)$:

$$
T(s) = T_{start} \exp\left( \log(T_{start}^{-1} T_{end}) \cdot s \right), \quad s \in [0, 1]
$$

di mana $s(t)$ mengikuti fungsi percepatan trapesium (LSPB).

---

## 5. Model Robot Bawaan Platform (Catalog Specification)

| Model Robot | Konfigurasi Kinematika | Aplikasi Utama | Beban / Torsi Maksimum |
|---|---|---|---|
| **EduArm-3DOF** | 3 RRR (Rotasi Basis, Bahu, Siku) | Pembelajaran dasar trigonometri, FK/IK planar & spatial | 3x Micro Servo SG90 (Torsi 1.8 kg·cm) |
| **AeroArm-6DOF** | 6 RRRRRR (Articulated Spherical Wrist) | Simulasi industri (Pick and Place, Welding, Perakitan PCB) | 4x MG996R (11 kg·cm) + 2x SG90 |
| **RoverBot-4WD** | Differential Drive + 2-DOF Camera Gimbal | Navigasi otonom SLAM, deteksi objek LiDAR & Kamera AI | 4x TT Geared Motor + 2x SG90 Pan/Tilt |
| **DeltaBot-3DOF** | Paralel 3-RRR Kinematika Tertutup | Pengurutan berkecepatan tinggi (*high-speed delta sorter*) | 3x NEMA 17 Stepper Motor |
