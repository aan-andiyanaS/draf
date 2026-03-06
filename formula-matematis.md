# Formula Matematis — Sistem Navigasi Bantu Tunanetra

Dokumen ini berisi **seluruh formula matematis** yang digunakan dalam sistem navigasi bantu tunanetra. Setiap formula disertai penjelasan **mengapa formula tersebut penting**, **apa arti setiap variabel**, dan **contoh perhitungan** agar mudah dipahami.

Dokumen ini sesuai dengan **sub-bab 3.5.1 Formulasi Matematis** pada BAB 3 skripsi.

---

## Konstanta Sistem

Berikut adalah nilai-nilai tetap yang digunakan di seluruh formula. Nilai-nilai ini ditentukan oleh spesifikasi hardware yang dipilih.

| Simbol | Nilai | Keterangan |
|---|---|---|
| $W_{cam}$ | 640 px | Lebar resolusi kamera OV2640 dalam mode VGA. Menentukan jumlah piksel horizontal yang tersedia untuk deteksi objek. |
| $H_{cam}$ | 480 px | Tinggi resolusi kamera OV2640 dalam mode VGA. |
| $N_{col}$ | 8 | Jumlah kolom sensor VL53L5CX. Sensor memiliki grid 8×8, sehingga secara horizontal terbagi menjadi 8 zona deteksi jarak. |
| $N_{row}$ | 8 | Jumlah baris sensor VL53L5CX. Setiap baris mewakili sudut vertikal yang berbeda. |
| $D_{left}$ | 80 px | *Dead zone* kiri — area di tepi kiri frame kamera yang **tidak ter-cover** oleh sensor jarak ToF, karena FoV kamera (~66°) lebih lebar dari FoV sensor (~45°). |
| $D_{right}$ | 80 px | *Dead zone* kanan — sama seperti di atas tetapi di sisi kanan frame. |
| $W_{tof}$ | 480 px | Lebar area kamera yang di-cover oleh sensor ToF. Dihitung: $W_{cam} - D_{left} - D_{right} = 640 - 80 - 80 = 480$ piksel. |
| $R_{col}$ | 60 px | Resolusi horizontal per kolom sensor ToF. Dihitung: $W_{tof} / N_{col} = 480 / 8 = 60$ piksel per kolom. Artinya setiap 1 kolom ToF mewakili 60 piksel di frame kamera. |

---

## A. Formulasi Titik Tengah Objek (Centroid Bounding Box)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3a

### Mengapa Formula Ini Penting?

Ketika YOLOv11 mendeteksi sebuah objek, output-nya berupa **bounding box** (kotak pembatas) yang menandai posisi objek di frame kamera. Bounding box ini didefinisikan oleh koordinat sudut kiri atas dan kanan bawah. Namun, untuk menentukan **di mana objek tersebut berada secara horizontal** (kiri, tengah, atau kanan dari sudut pandang user), kita memerlukan **satu titik referensi** — yaitu titik tengah horizontal.

Titik tengah inilah yang menjadi input utama untuk **semua formula selanjutnya**: menentukan arah jam (Formula B), kolom sensor jarak (Formula C), dan jarak objek (Formula D).

### Formula

$$X_c = x_{min} + \frac{x_{max} - x_{min}}{2}$$

### Keterangan Variabel

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $X_c$ | Output | Piksel (px) | **Titik tengah horizontal** objek. Nilai ini berkisar dari 0 (paling kiri frame) sampai 639 (paling kanan frame). Ini adalah "alamat horizontal" objek di kamera. |
| $x_{min}$ | Input | Piksel (px) | **Batas kiri** bounding box — koordinat X dari sisi kiri kotak pembatas yang dihasilkan YOLO. |
| $x_{max}$ | Input | Piksel (px) | **Batas kanan** bounding box — koordinat X dari sisi kanan kotak pembatas yang dihasilkan YOLO. |

### Contoh Perhitungan

YOLO mendeteksi sebuah tiang dengan bounding box: $x_{min} = 150$, $x_{max} = 230$.

$$X_c = 150 + \frac{230 - 150}{2} = 150 + \frac{80}{2} = 150 + 40 = 190$$

Artinya: titik tengah tiang berada di **piksel ke-190** dari kiri frame kamera.

---

## B. Formulasi Pemetaan Arah Jam

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3a
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-2

### Mengapa Formula Ini Penting?

Tunanetra tidak bisa memahami instruksi "objek berada di piksel 190". Mereka membutuhkan referensi spasial yang intuitif. **Sistem arah jam** (clock direction) adalah metode yang umum digunakan dalam orientasi mobilitas tunanetra:
- **Jam 12** = tepat di depan
- **Jam 11** = sedikit ke kiri
- **Jam 10** = jauh ke kiri
- **Jam 1** = sedikit ke kanan
- **Jam 2** = jauh ke kanan

Formula ini mengkonversi posisi piksel abstrak menjadi arah yang bisa langsung dipahami oleh tunanetra melalui TTS.

### Formula

$$\text{Arah Jam} = \begin{cases} 10, & \text{jika } X_c < 80 \\ 11, & \text{jika } 80 \le X_c < 240 \\ 12, & \text{jika } 240 \le X_c < 400 \\ 1, & \text{jika } 400 \le X_c < 560 \\ 2, & \text{jika } X_c \ge 560 \end{cases}$$

### Keterangan Variabel

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $X_c$ | Input | Piksel (px) | Titik tengah horizontal objek dari Formula A. |
| Arah Jam | Output | — | Arah relatif objek terhadap user, dalam notasi jam (10, 11, 12, 1, atau 2). |
| $80$ | Konstanta | Piksel | Batas antara dead zone kiri dan zona aktif sensor. Piksel 0–79 berada di luar jangkauan sensor, tapi YOLO masih bisa mendeteksi objek di sini. |
| $240, 400, 560$ | Konstanta | Piksel | Batas antar zona. Area tengah 480px dibagi 3 zona (masing-masing **160px**) untuk Jam 11, 12, dan 1. |

### Visualisasi

```
Lebar Kamera: 640 piksel
┌────────┬──────────────┬──────────────┬──────────────┬────────┐
│  80px  │   160px      │   160px      │   160px      │  80px  │
│ JAM 10 │   JAM 11     │   JAM 12     │   JAM 1      │ JAM 2  │
│ 0──79  │  80──239     │  240──399    │  400──559    │ 560─639│
│ ❌ ToF  │  ✅ ToF       │  ✅ ToF       │  ✅ ToF       │ ❌ ToF  │
└────────┴──────────────┴──────────────┴──────────────┴────────┘
```

### Alasan Pembagian Zona

- **Dead zone 80px di tiap sisi:** Kamera OV2640 memiliki FoV horizontal ~66°, sedangkan sensor VL53L5CX memiliki FoV ~45°. Selisih sudut (~21°) menyebabkan area di tepi kiri dan kanan frame kamera **tidak ter-cover** oleh sensor jarak. Objek terdeteksi YOLO di zona ini hanya dapat diberi informasi arah **tanpa jarak presisi**.
- **Zona 160px untuk Jam 11, 12, 1:** Area tengah 480px dibagi rata menjadi 3 zona yang simetris. Ini mencerminkan pembagian alami sudut pandang depan manusia.
- **Jam 10 dan 2 tanpa jarak:** YOLO masih bisa mendeteksi objek di sini, namun peringatan hanya menyebutkan arah tanpa jarak.

### Contoh

$X_c = 190$ → $80 \le 190 < 240$ → **Jam 11** (objek sedikit ke kiri dari depan user)

---

## C. Formulasi Pemetaan Indeks Kolom ToF (Grid Binning)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3a
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-2

### Mengapa Formula Ini Penting?

Setelah mengetahui posisi horizontal objek ($X_c$) dan arahnya (Formula B), kita perlu **mengukur jaraknya**. Jarak diukur oleh sensor ToF VL53L5CX yang memiliki **8 kolom**. Tapi kolom mana yang harus dibaca?

Formula ini menjembatani dunia **piksel kamera** (640px) dengan dunia **grid sensor ToF** (8 kolom). Tanpa formula ini, sistem tidak tahu kolom sensor mana yang sesuai dengan posisi objek yang dideteksi YOLO.

### Formula

$$C_{index} = \left\lfloor \frac{X_c - 80}{60} \right\rfloor$$

### Keterangan Variabel

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $C_{index}$ | Output | Integer (0-7) | **Indeks kolom** sensor ToF yang sesuai dengan posisi objek. Digunakan untuk mengambil data jarak dari matriks sensor. |
| $X_c$ | Input | Piksel (px) | Titik tengah horizontal objek dari Formula A. |
| $80$ | Konstanta | Piksel | Offset dead zone kiri. Dikurangi karena kolom ToF dimulai dari piksel ke-80 (bukan piksel 0). |
| $60$ | Konstanta | Piksel | Lebar setiap kolom ToF dalam satuan piksel kamera. Dihitung dari $W_{tof} / N_{col} = 480 / 8 = 60$. |
| $\lfloor \dots \rfloor$ | Operator | — | **Fungsi floor** (pembulatan ke bawah). Memastikan hasil selalu bilangan bulat (0-7). Piksel berapapun dalam blok 60 piksel yang sama → indeks kolom sama. |

> **Prasyarat:** Formula ini hanya berlaku jika $80 \le X_c < 560$. Jika $X_c$ berada di luar rentang tersebut (dead zone), tidak ada kolom ToF yang bisa diambil datanya.

### Tabel Mapping Kolom

| $C_{index}$ | Rentang Piksel | Zona Jam |
|---|---|---|
| 0 | 80 – 139 | Jam 11 |
| 1 | 140 – 199 | Jam 11 |
| 2 | 200 – 259 | Jam 11 / 12 |
| 3 | 260 – 319 | Jam 12 |
| 4 | 320 – 379 | Jam 12 |
| 5 | 380 – 439 | Jam 12 / 1 |
| 6 | 440 – 499 | Jam 1 |
| 7 | 500 – 559 | Jam 1 |

### Contoh Perhitungan

$X_c = 350$ (objek di area Jam 12):

$$C_{index} = \left\lfloor \frac{350 - 80}{60} \right\rfloor = \left\lfloor \frac{270}{60} \right\rfloor = \lfloor 4.5 \rfloor = 4$$

Artinya: data jarak untuk objek ini diambil dari **kolom ke-4** matriks sensor ToF.

---

## D. Formulasi Jarak Objek dari Sensor ToF

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3a

### Mengapa Formula Ini Penting?

Sensor VL53L5CX mengembalikan **64 titik jarak** (matriks 8×8). Tidak semua titik relevan untuk mengukur jarak objek. Baris atas sensor "melihat" ke arah langit, baris bawah "melihat" ke lantai. Hanya **baris tengah** (baris 3-5) yang melihat ke arah objek setinggi badan manusia.

Formula ini mengambil data jarak dari **kolom yang tepat** (dari Formula C) dan **baris yang relevan** (baris 3-5), lalu merata-ratakannya untuk mendapatkan jarak yang robust terhadap noise sensor.

### Formula

$$D_{objek} = \frac{1}{3} \sum_{r=3}^{5} \text{ToF}[r][C_{index}]$$

### Keterangan Variabel

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $D_{objek}$ | Output | Milimeter (mm) | **Jarak objek** yang terdeteksi. Dikonversi ke meter ($D_{objek} / 1000$) sebelum digunakan di Formula E dan diucapkan TTS. |
| $\text{ToF}[r][c]$ | Input | Milimeter (mm) | Nilai pembacaan sensor ToF pada **baris** $r$ dan **kolom** $c$. Matriks 8×8 = 64 nilai. |
| $r$ | Iterator | — | Indeks baris sensor (0-7). Formula ini hanya menggunakan $r = 3, 4, 5$ karena baris tersebut mengarah ke area objek setinggi badan. |
| $C_{index}$ | Input | Integer (0-7) | Kolom sensor yang sesuai dengan posisi objek, dihitung dari Formula C. |
| $\frac{1}{3}$ | Konstanta | — | Pembagi untuk rata-rata. Menggunakan 3 baris (baris 3, 4, 5) untuk mengurangi noise dan mendapatkan jarak representatif. |

### Pembagian Fungsi Baris Sensor

```
VL53L5CX dipasang miring ~15° ke bawah di frame kacamata:

Baris 0-2: Melihat ke atas    → Langit / plafon (tidak relevan untuk jarak objek)
Baris 3-5: Melihat ke tengah  → OBJEK setinggi badan ← DIPAKAI oleh Formula D
Baris 6-7: Melihat ke bawah   → Lantai / permukaan ← DIPAKAI oleh Formula H (Deteksi Medan)
```

### Mengapa Rata-rata 3 Baris?

- **Mengurangi noise:** Sensor ToF bisa mengalami fluktuasi ±5% antar pembacaan. Rata-rata 3 titik menghasilkan nilai lebih stabil.
- **Menangkap objek dengan ketinggian bervariasi:** Orang dewasa, anak-anak, dan objek statis (tiang) memiliki ketinggian berbeda. Baris 3-5 mencakup rentang vertikal yang cukup.
- **Menghindari false reading:** Jika satu baris terkena refleksi dari lantai atau langit, dua baris lainnya akan mengkompensasi.

### Contoh Perhitungan

$C_{index} = 4$ (dari Formula C), pembacaan sensor:

| Baris | $\text{ToF}[r][4]$ | Keterangan |
|---|---|---|
| Baris 3 | 2100 mm | Dada objek |
| Baris 4 | 2050 mm | Perut objek |
| Baris 5 | 2200 mm | Pinggang objek |

$$D_{objek} = \frac{2100 + 2050 + 2200}{3} = \frac{6350}{3} = 2117 \text{ mm} \approx 2.1 \text{ m}$$

Output TTS: *"Objek di arah Jam 12, jarak 2.1 meter"*

---

## E. Formulasi Threshold Adaptif (Jalur A — Objek ≤ 4m)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3c
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-3a

### Mengapa Formula Ini Penting?

Jika sistem selalu memperingatkan saat objek dalam jarak 1 meter (threshold tetap), user yang sedang berjalan cepat bisa **terlambat bereaksi** — karena 1 meter hanya memberikan waktu reaksi ~0.7 detik saat berjalan 1.5 m/s.

Threshold adaptif memecahkan masalah ini: **semakin cepat user/objek bergerak, semakin jauh threshold peringatan**. User yang berlari mendapat peringatan dari jarak 4 meter, sementara user yang diam hanya diperingatkan jika objek di bawah 1 meter. Ini meniru refleks alami manusia — saat berlari, kita secara instingtif melihat lebih jauh ke depan.

### Formula E.1 — Kecepatan Pendekatan

$$v = \frac{|\Delta D|}{{\Delta t}}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $v$ | Output | m/s | **Kecepatan pendekatan relatif** antara user dan objek. Entah user yang mendekat ke objek statis, atau objek yang mendekat ke user yang diam. |
| $\Delta D$ | Input | Meter (m) | **Selisih jarak** antara dua frame berturut-turut: $\Delta D = D_{lama} - D_{baru}$. Jika positif, objek mendekat. Jika negatif, menjauh. Menggunakan nilai absolut $|\Delta D|$ agar kecepatan selalu positif. |
| $\Delta t$ | Input | Detik (s) | **Selisih waktu** antara dua frame berturut-turut. Bergantung pada frame rate kamera (~0.1 detik untuk 10 FPS). |

**Siapa yang bergerak?** Ditentukan oleh Formula J (Accelerometer):
- **User bergerak** (accelerometer aktif): $v$ menunjukkan kecepatan user mendekati objek statis.
- **User diam + objek mendekat** ($\Delta D > 0$): $v$ menunjukkan kecepatan objek mendekati user (misalnya kendaraan lambat).

### Formula E.2 — Threshold Peringatan

$$T = \min\left(1 + v \times 2,\ 4\right)$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $T$ | Output | Meter (m) | **Threshold peringatan** — jarak di mana peringatan TTS mulai dikeluarkan. Semakin cepat kecepatan, semakin jauh threshold ini. |
| $1$ | Konstanta | Meter | **Jarak aman minimum tetap**. Meskipun kecepatan = 0, threshold tidak pernah kurang dari 1 meter. Ini jarak minimum agar user bisa berhenti atau berbelok. |
| $v \times 2$ | Komponen | Meter | **Jarak tambahan** yang didapatkan dari mengalikan kecepatan pendekatan dengan **waktu reaksi** (2 detik). Logikanya: jika user bergerak $v$ m/s, dalam 2 detik dia akan menempuh $v \times 2$ meter. Threshold harus cukup jauh agar user punya **2 detik** untuk bereaksi. |
| $4$ | Konstanta | Meter | **Batas maksimum threshold**. Tidak mungkin melebihi 4 meter karena sensor VL53L5CX memiliki jangkauan maksimum 4 meter. |
| $\min(\dots)$ | Fungsi | — | Memilih **nilai terkecil** antara $(1 + v \times 2)$ dan $4$. Mencegah threshold melebihi kemampuan sensor. |

### Mengapa Konstanta Tersebut Dipilih?

| Konstanta | Nilai | Alasan |
|---|---|---|
| Jarak minimum | 1 m | Jarak berhenti aman pejalan kaki. Rata-rata satu langkah penuh = ~0.75m. 1 meter memberi margin satu langkah untuk berhenti. |
| Waktu reaksi | 2 s | Rata-rata waktu reaksi manusia dewasa dalam kondisi waspada (bukan terkejut) adalah 1.5-2.5 detik (sumber: WHO, pedestrian safety). Dipilih 2 detik sebagai kompromi. |
| Maksimum | 4 m | Batas spesifikasi sensor VL53L5CX. Data jarak di atas 4m tidak akurat/tersedia. |

### Formula E.3 — Logika Peringatan

$$\text{Peringatan} = \begin{cases} \text{Ya (TTS aktif)}, & \text{jika } D_{objek} < T \\ \text{Tidak (diam)}, & \text{jika } D_{objek} \ge T \end{cases}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $D_{objek}$ | Input | Meter (m) | Jarak objek dari Formula D. |
| $T$ | Input | Meter (m) | Threshold dari Formula E.2. |

### Contoh Skenario Lengkap

| Skenario | $v$ | Perhitungan $T$ | $T$ | Objek di 2.5m | Peringatan? |
|---|---|---|---|---|---|
| User jalan pelan | 0.5 m/s | $\min(1 + 0.5 \times 2,\ 4) = \min(2, 4)$ | 2 m | 2.5m ≥ 2m | ❌ Aman |
| User jalan cepat | 1.5 m/s | $\min(1 + 1.5 \times 2,\ 4) = \min(4, 4)$ | 4 m | 2.5m < 4m | ✅ AWAS |
| Motor mendekat | 3 m/s | $\min(1 + 3 \times 2,\ 4) = \min(7, 4)$ | 4 m | 2.5m < 4m | ✅ AWAS |
| Kedua diam | 0 m/s | $\min(1 + 0 \times 2,\ 4) = \min(1, 4)$ | 1 m | 2.5m ≥ 1m | ❌ Aman |

---

## F. Formulasi Peringatan Objek Statis (Jalur A — Cabang Ketiga)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3c
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-3a

### Mengapa Formula Ini Penting?

Ketika user dan objek sama sekali tidak bergerak ($\Delta D = 0$), kecepatan pendekatan = 0, sehingga threshold adaptif (Formula E) hanya menjadi 1 meter. Tapi jika objek ada di jarak 0.5 meter dan sistem terus-menerus memperingatkan "Objek dekat... Objek dekat... Objek dekat..." — ini akan **sangat mengganggu** (spam suara).

Formula ini menyelesaikan masalah tersebut: peringatan **hanya diberikan satu kali**. Setelah itu, flag objek ditandai "sudah diperingatkan" dan peringatan tidak akan diulang sampai kondisi berubah.

### Formula

$$\text{Peringatan} = \begin{cases} \text{Ya (sekali)}, & \text{jika } D_{objek} < 1 \text{ m} \wedge \text{flag}_{objek} = \text{false} \\ \text{Tidak}, & \text{jika } D_{objek} \ge 1 \text{ m} \vee \text{flag}_{objek} = \text{true} \end{cases}$$

### Keterangan Variabel

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $D_{objek}$ | Input | Meter (m) | Jarak objek dari Formula D. |
| $1$ m | Konstanta | Meter | Threshold tetap (sama seperti minimum di Formula E). Jika diam, hanya objek **sangat dekat** yang diperingatkan. |
| $\text{flag}_{objek}$ | State | Boolean | **Flag per-objek** yang melacak apakah peringatan sudah diberikan. Setiap objek unik (berdasarkan tracking ID dari YOLO) memiliki flag sendiri. |
| $\wedge$ | Operator | — | **Logika AND**: kedua syarat harus terpenuhi bersamaan. |
| $\vee$ | Operator | — | **Logika OR**: salah satu syarat terpenuhi sudah cukup. |

### Mekanisme Reset Flag

```
flagObjek[id] = false  (default saat objek pertama kali terdeteksi)

Jika D_objek < 1m AND flagObjek[id] == false:
    → TTS: "Objek Dekat di Jam [X], [Jarak]m"
    → flagObjek[id] = true     ← tandai: sudah diperingatkan

Flag di-reset (kembali ke false) saat:
    → User mulai bergerak (accelerometer berubah) → masuk cabang E
    → Objek bergerak (ΔD ≠ 0) → masuk cabang E
    → Objek hilang dari frame YOLO → hapus flag
```

---

## G. Formulasi Delta Bounding Box (Jalur B — Objek > 4m)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3d
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-3b

### Mengapa Formula Ini Penting?

Sensor ToF VL53L5CX **hanya bisa mengukur jarak sampai 4 meter**. Tapi bahaya tidak berhenti di 4 meter — kendaraan bermotor bisa mendekat dari jarak 20+ meter dengan kecepatan tinggi. Kalau kita menunggu kendaraan masuk ke jangkauan 4 meter, user hanya punya < 1 detik untuk bereaksi.

Formula ini memberikan **peringatan dini** untuk objek yang jauh (> 4m): jika bounding box YOLO **membesar secara signifikan antar frame**, berarti objek tersebut **mendekati kamera** — meskipun kita tidak tahu persis jaraknya.

### Formula G.1 — Area Bounding Box

$$A = (x_{max} - x_{min}) \times (y_{max} - y_{min})$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $A$ | Output | Piksel² (px²) | **Area** (luas) bounding box dalam satuan piksel persegi. Objek yang dekat dengan kamera memiliki bounding box **lebih besar** di frame kamera. |
| $x_{max} - x_{min}$ | Komponen | Piksel (px) | **Lebar** bounding box. |
| $y_{max} - y_{min}$ | Komponen | Piksel (px) | **Tinggi** bounding box. |

### Formula G.2 — Delta Area Antar Frame

$$\Delta A = \frac{A_{baru} - A_{lama}}{A_{lama}} \times 100\text{\%}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $\Delta A$ | Output | Persen (%) | **Persentase perubahan area** bounding box. Positif = membesar (mendekat). Negatif = mengecil (menjauh). |
| $A_{baru}$ | Input | Piksel² (px²) | Area bounding box di **frame saat ini**. |
| $A_{lama}$ | Input | Piksel² (px²) | Area bounding box di **frame sebelumnya** untuk objek yang **sama** (matching by tracking ID YOLO). |

### Formula G.3 — Logika Peringatan

$$\text{Peringatan} = \begin{cases} \text{Ya}, & \text{jika } \Delta A > 20\text{\%} \\ \text{Tidak}, & \text{jika } \Delta A \le 20\text{\%} \end{cases}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $20\text{\%}$ | Konstanta | Persen | **Threshold signifikansi**. Dipilih berdasarkan pertimbangan berikut: perubahan < 5% = noise YOLO (jitter deteksi). Perubahan 5-20% = objek bergerak pelan atau bergerak paralel. Perubahan > 20% = objek **mendekati kamera dengan kecepatan tinggi**. |

### Perbandingan Jalur A vs Jalur B

| Aspek | Jalur A (Formula D+E) | Jalur B (Formula G) |
|---|---|---|
| Jangkauan | $D \le 4$ meter | $D > 4$ meter |
| Data jarak | ✅ Presisi (mm, dari ToF) | ❌ Tidak ada data jarak |
| Deteksi arah | ✅ Arah jam + jarak | ✅ Arah jam saja |
| Output TTS | "Objek di Jam X, Ym" | "Kendaraan Mendekat dari Jam X" |
| Transisi | — | **Otomatis beralih ke Jalur A** saat objek masuk ≤ 4m |

### Contoh Perhitungan

BBox motor: $A_{lama} = 5000\text{ px}^2$ (frame sebelumnya), $A_{baru} = 6500\text{ px}^2$ (frame sekarang):

$$\Delta A = \frac{6500 - 5000}{5000} \times 100\text{\%} = \frac{1500}{5000} \times 100\text{\%} = 30\text{\%}$$

$30\text{\%} > 20\text{\%}$ → **PERINGATAN**: *"AWAS, Kendaraan Mendekat dari Jam 12!"*

---

## H. Formulasi Deteksi Medan (Jalur C — Tangga/Lubang)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3e
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-3c

### Mengapa Formula Ini Penting?

Bahaya bagi tunanetra bukan hanya objek yang menghalangi — **perubahan elevasi lantai** (tangga turun, lubang, parit) adalah penyebab utama cedera serius. Tongkat putih konvensional bisa mendeteksi lubang, tapi perangkat kacamata tidak memiliki tongkat.

Formula ini memanfaatkan **baris bawah sensor ToF** (yang mengarah ke lantai) untuk mendeteksi perubahan mendadak di permukaan jalan. Jika lantai di depan kaki tiba-tiba "hilang" (jarak sensor membesar), berarti ada anomali medan.

### Formula H.1 — Rata-rata Jarak per Zona

$$\bar{D}_{bawah} = \frac{1}{N_{col}} \sum_{c=0}^{7} \text{ToF}[r_{bawah}][c] \quad \text{dengan } r_{bawah} \in \{6, 7\}$$

$$\bar{D}_{tengah} = \frac{1}{N_{col}} \sum_{c=0}^{7} \text{ToF}[r_{tengah}][c] \quad \text{dengan } r_{tengah} \in \{4, 5\}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $\bar{D}_{bawah}$ | Output | Milimeter (mm) | **Rata-rata jarak baris bawah** sensor (baris 6-7). Baris ini mengarah ke lantai **dekat kaki user** (~0.5-1m di depan). Pada lantai datar, jarak ini pendek (lantai dekat). |
| $\bar{D}_{tengah}$ | Output | Milimeter (mm) | **Rata-rata jarak baris tengah** sensor (baris 4-5). Baris ini mengarah ke lantai **lebih jauh** (~1.5-3m di depan). Pada lantai datar, jarak ini lebih panjang. |
| $\text{ToF}[r][c]$ | Input | Milimeter (mm) | Pembacaan sensor pada baris $r$, kolom $c$. |
| $N_{col} = 8$ | Konstanta | — | Jumlah kolom. Rata-rata semua 8 kolom per baris untuk memperhalus noise. |

### Formula H.2 — Rasio Anomali Medan

$$R = \frac{\bar{D}_{bawah}}{\bar{D}_{tengah}}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $R$ | Output | Rasio (0-∞) | **Rasio anomali medan**. Membandingkan jarak lantai dekat vs lantai jauh. Pada kondisi normal, $R < 0.7$. Pada anomali, $R > 0.8$. |

### Logika di Balik Rasio

```
LANTAI DATAR (Normal, R < 0.7):
Sensor di baris bawah → melihat lantai DEKAT → jarak PENDEK
Sensor di baris tengah → melihat lantai JAUH → jarak PANJANG
Rasio = pendek / panjang = KECIL

LUBANG (Anomali, R > 0.8):
Sensor di baris bawah → melihat ke LUBANG → tidak ada lantai → jarak JAUH
Sensor di baris tengah → melihat lantai setelah lubang → jarak PANJANG
Rasio = jauh / panjang = BESAR (mendekati 1)
```

### Formula H.3 — Logika Deteksi

$$\text{Anomali} = \begin{cases} \text{Ya (periksa pola)}, & \text{jika } R > 0.8 \\ \text{Normal (aman)}, & \text{jika } R < 0.7 \end{cases}$$

### Formula H.4 — Analisis Pola Anomali

Jika $R > 0.8$ (anomali terdeteksi), sistem perlu membedakan **jenis** anomali. Ini dilakukan dengan menganalisis **standar deviasi** per baris:

$$\sigma_{baris} = \sqrt{\frac{1}{N_{col}} \sum_{c=0}^{7} \left(\text{ToF}[r][c] - \bar{D}_{r}\right)^2}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $\sigma_{baris}$ | Output | Milimeter (mm) | **Standar deviasi** jarak dalam satu baris sensor. Mengukur seberapa **merata atau tidak merata** jarak di sepanjang baris. |
| $\bar{D}_{r}$ | Input | Milimeter (mm) | Rata-rata jarak pada baris $r$ (dari semua 8 kolom). |
| $\text{ToF}[r][c] - \bar{D}_{r}$ | Komponen | Milimeter (mm) | **Deviasi** setiap kolom terhadap rata-rata baris. Jika semua kolom mirip, deviasi kecil. |

| Pola yang Ditemukan | $\sigma$ | Artinya | Tindakan |
|---|---|---|---|
| Gradual + $\sigma$ kecil | Rendah | Jarak meningkat **merata** di semua kolom → permukaan turun secara gradual | **Tangga / Penurunan** → konfirmasi dengan YOLO |
| Tiba-tiba + $\sigma$ besar | Tinggi | Jarak melonjak hanya di **beberapa kolom** → ada "lubang" lokal | **Lubang / Parit** → langsung peringatan |

---

## I. Formulasi Mode Darurat (Offline & Gelap)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — sub-bab 3.5.5
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-4a & SD-4b

### Mengapa Formula Ini Penting?

Saat WiFi putus atau lingkungan gelap, smartphone tidak bisa membantu (tidak ada YOLO, TTS bermasalah). Mengandalkan **hanya** Formula E (threshold adaptif) akan gagal karena data accelerometer tidak tersedia di ESP32. Sistem harus punya **mekanisme keselamatan terakhir (fail-safe)** yang sepenuhnya mandiri.

Formula ini memberikan perlindungan dasar: **buzzer fisik pada perangkat wearable** dibunyikan langsung oleh ESP32 saat ada objek sangat dekat (< 1m). Sederhana tapi bisa menyelamatkan dari tabrakan.

### Formula I.1 — Threshold Tetap (Buzzer)

$$\text{Buzzer} = \begin{cases} \text{Aktif (bunyi)}, & \text{jika } D_{min} < 1 \text{ m} \\ \text{Diam}, & \text{jika } D_{min} \ge 1 \text{ m} \end{cases}$$

$$D_{min} = \min_{r,c} \left(\text{ToF}[r][c]\right) \quad \text{untuk } r \in \{0..7\},\ c \in \{0..7\}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $D_{min}$ | Output | Milimeter (mm) | **Jarak terdekat** dari seluruh 64 zona sensor. Tidak peduli arah atau posisi — jika **zona manapun** mendeteksi objek sangat dekat, buzzer berbunyi. Ini pendekatan konservatif (safety-first). |
| $\text{ToF}[r][c]$ | Input | Milimeter (mm) | Pembacaan sensor pada baris $r$, kolom $c$. Seluruh 8×8 = 64 zona diperiksa. |
| $\min_{r,c}$ | Fungsi | — | Mencari **nilai minimum** dari seluruh matriks 8×8. |
| $1$ m | Konstanta | Meter | Threshold tetap. Tidak adaptif karena tidak ada data kecepatan ($v$ tidak tersedia tanpa smartphone). |

### Mengapa $T = 1$ m Tetap (Bukan Adaptif)?

| Komponen | Mode Smart (Formula E) | Mode Darurat (Formula I) |
|---|---|---|
| Accelerometer | ✅ Aktif (di smartphone) | ❌ Tidak ada (ESP32 mandiri) |
| YOLO | ✅ Bisa identifikasi objek | ❌ Tidak ada (kamera off/gelap) |
| Kecepatan ($v$) | ✅ Bisa dihitung | ❌ Tidak bisa dihitung |
| Threshold | Adaptif: $1$–$4$ m | Tetap: $1$ m |
| Output | TTS (detail) | Buzzer (bunyi saja) |

### Formula I.2 — Kondisi Masuk Mode Darurat

$$\text{Mode} = \begin{cases} \text{Offline}, & \text{jika WiFi putus} > 5 \text{ detik} \\ \text{Gelap}, & \text{jika WiFi OK} \wedge B_{cam} < B_{threshold} \\ \text{Smart}, & \text{jika WiFi OK} \wedge B_{cam} \ge B_{threshold} \end{cases}$$

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $B_{cam}$ | Input | Integer (0-255) | **Rata-rata brightness** (kecerahan) frame kamera. 0 = gelap total, 255 = terang penuh. Dihitung dari rata-rata nilai piksel grayscale frame kamera. |
| $B_{threshold}$ | Konstanta | Integer (0-255) | **Threshold kecerahan minimum** agar YOLO bisa bekerja efektif. Ditentukan secara empirik saat pengujian. |
| $5$ detik | Konstanta | Detik | **Toleransi WiFi putus**. Tidak langsung masuk mode offline saat WiFi sempat terputus sesaat (jitter). Hanya jika putus **berkelanjutan** > 5 detik. |

---

## J. Formulasi Deteksi Sumber Gerakan (Accelerometer)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3c

### Mengapa Formula Ini Penting?

Formula E (threshold adaptif) memerlukan input kecepatan ($v$), tapi sebelum menghitung kecepatan, sistem harus tahu **siapa yang bergerak**: apakah user yang berjalan mendekati objek statis, atau objek yang bergerak mendekati user yang diam, atau kedua-duanya diam.

Ini penting karena **respons sistem berbeda** untuk setiap kasus:
- **User bergerak**: Peringatkan tentang objek statis di depannya — user bisa berhenti atau berbelok.
- **Objek mendekat**: Peringatkan bahwa ada objek mendekati user — user sebaiknya menghindar.
- **Kedua diam**: Gunakan Formula F (peringatan sekali saja) — user sudah di posisi tetap.

### Formula

$$\text{Sumber Gerakan} = \begin{cases} \text{User bergerak}, & \text{jika } a_{acc} > a_{threshold} \\ \text{Objek mendekat}, & \text{jika } a_{acc} \le a_{threshold} \wedge \Delta D > 0 \\ \text{Kedua diam}, & \text{jika } a_{acc} \le a_{threshold} \wedge \Delta D = 0 \end{cases}$$

### Keterangan Variabel

| Variabel | Tipe | Satuan | Deskripsi |
|---|---|---|---|
| $a_{acc}$ | Input | g (gravitasi) | **Magnitude accelerometer** smartphone. Mengukur percepatan total perangkat. Jika HP ada di saku user yang berjalan, nilai akan berfluktuasi. Jika user diam, nilai mendekati 0 (hanya gravitasi). |
| $a_{threshold}$ | Konstanta | g | **Threshold gerakan** — nilai empirik (~0.3 g) yang membedakan "user bergerak" dari "user diam". Ditentukan melalui pengujian: berjalan normal menghasilkan ~0.5-1.0 g, diam menghasilkan ~0-0.1 g. |
| $\Delta D$ | Input | Meter (m) | **Delta jarak** dari sensor ToF: $D_{lama} - D_{baru}$. Jika positif ($\Delta D > 0$), berarti **jarak mengecil** = objek mendekat. Jika nol, tidak ada perubahan jarak. |
| $\wedge$ | Operator | — | **Logika AND**: kedua syarat harus terpenuhi bersamaan. |

---

## Ringkasan Seluruh Formula

| # | Formula | Input | Output | Kegunaan | Dipakai di |
|---|---|---|---|---|---|
| **A** | $X_c = x_{min} + \frac{x_{max} - x_{min}}{2}$ | BBox YOLO | Titik tengah (px) | Menentukan "alamat" horizontal objek | Flowchart 3a, SD-2 |
| **B** | Piecewise 5 zona | $X_c$ | Arah Jam (10-2) | Mengkonversi posisi piksel ke arah intuitif | Flowchart 3a, SD-2 |
| **C** | $C_{index} = \lfloor (X_c - 80) / 60 \rfloor$ | $X_c$ | Kolom ToF (0-7) | Menjembatani dunia kamera ke dunia sensor | Flowchart 3a, SD-2 |
| **D** | $D = \frac{1}{3}\sum_{r=3}^{5} \text{ToF}[r][C]$ | Kolom ToF | Jarak (m) | Mengukur jarak presisi dari objek | Flowchart 3a, SD-2 |
| **E** | $T = \min(1 + v \times 2, 4)$ | Kecepatan | Threshold (m) | Peringatan dini saat bergerak cepat | Flowchart 3c, SD-3a |
| **F** | $D < 1\text{m} \wedge \neg flag$ | Jarak + flag | Peringatan sekali | Mencegah spam suara saat diam | Flowchart 3c, SD-3a |
| **G** | $\Delta A = (A_b - A_l) / A_l \times 100\text{\%}$ | BBox area | Peringatan kendaraan | Deteksi kendaraan di luar jangkauan ToF | Flowchart 3d, SD-3b |
| **H** | $R = \bar{D}_{bawah} / \bar{D}_{tengah}$ | ToF baris 6-7 vs 4-5 | Anomali medan | Deteksi tangga, lubang, parit | Flowchart 3e, SD-3c |
| **I** | $D_{min} < 1\text{m}$ → Buzzer | ToF semua zona | Buzzer ON/OFF | Keselamatan terakhir tanpa HP | Flowchart 3.5.5, SD-4 |
| **J** | $a_{acc}$ vs $\Delta D$ | Accel + jarak | Sumber gerakan | Menentukan cabang mana di Formula E/F | Flowchart 3c, SD-3a |

### Alur Ketergantungan Antar Formula

```
YOLO → [A] Centroid → [B] Arah Jam → Output TTS
                    → [C] Kolom ToF → [D] Jarak → [J] Cek Gerakan
                                                         ↓
                                              ┌─── User Bergerak ──→ [E] Threshold Adaptif → Peringatan?
                                              ├─── Objek Mendekat ──→ [E] Threshold Adaptif → Peringatan?
                                              └─── Kedua Diam ─────→ [F] Peringatan Sekali → Peringatan?

Objek > 4m (tanpa ToF) ─────────────────────→ [G] Delta BBox → Peringatan?

ToF baris bawah ────────────────────────────→ [H] Rasio Medan → Anomali?

Mode Darurat (tanpa HP) ───────────────────→ [I] Buzzer Tetap → Bunyi?
```
