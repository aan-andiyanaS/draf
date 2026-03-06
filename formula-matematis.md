# Formula Matematis — Sistem Navigasi Bantu Tunanetra

Dokumen ini berisi **seluruh formula matematis** yang digunakan dalam sistem navigasi bantu tunanetra. Setiap formula dijelaskan secara detail agar mudah dipahami dan dapat diterapkan langsung ke kode implementasi.

Dokumen ini sesuai dengan **sub-bab 3.7 Formulasi Matematis** pada BAB 3 skripsi.

**Konstanta Sistem:**

| Simbol | Nilai | Keterangan |
|---|---|---|
| $W_{cam}$ | 640 px | Lebar resolusi kamera OV2640 (VGA) |
| $H_{cam}$ | 480 px | Tinggi resolusi kamera OV2640 (VGA) |
| $N_{col}$ | 8 | Jumlah kolom sensor VL53L5CX |
| $N_{row}$ | 8 | Jumlah baris sensor VL53L5CX |
| $D_{left}$ | 80 px | Dead zone kiri (luar FoV ToF) |
| $D_{right}$ | 80 px | Dead zone kanan (luar FoV ToF) |
| $W_{tof}$ | 480 px | Lebar area yang di-cover ToF ($W_{cam} - D_{left} - D_{right}$) |
| $R_{col}$ | 60 px | Resolusi per kolom ToF ($W_{tof} / N_{col}$) |

---

## A. Formulasi Titik Tengah Objek (Centroid Bounding Box)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3a

Sistem menentukan posisi objek berdasarkan **titik tengah sumbu horizontal** ($X_c$) dari Bounding Box yang dihasilkan oleh YOLOv11. Titik tengah ini adalah pusat visual dari objek yang terdeteksi dan digunakan untuk semua perhitungan selanjutnya.

$$X_c = x_{min} + \frac{x_{max} - x_{min}}{2}$$

| Simbol | Keterangan |
|---|---|
| $X_c$ | Posisi horizontal titik tengah objek (piksel) |
| $x_{min}$ | Batas kiri bounding box (piksel) |
| $x_{max}$ | Batas kanan bounding box (piksel) |

**Contoh:** YOLO mendeteksi objek dengan bounding box $x_{min} = 150$, $x_{max} = 230$:
$$X_c = 150 + \frac{230 - 150}{2} = 150 + 40 = 190$$

---

## B. Formulasi Pemetaan Arah Jam

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3a
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-2

Sistem membagi lebar layar 640 piksel menjadi **5 zona arah jam**. Area tengah selebar 480 piksel (dari piksel 80 hingga 559) dibagi menjadi 3 zona utama (Jam 11, 12, 1) dengan lebar masing-masing **160 piksel**. Sisa piksel di tepi luar diklasifikasikan sebagai area luar jangkauan sensor (Jam 10 dan Jam 2).

$$\text{Arah Jam} = \begin{cases} 10, & \text{jika } X_c < 80 \\ 11, & \text{jika } 80 \le X_c < 240 \\ 12, & \text{jika } 240 \le X_c < 400 \\ 1, & \text{jika } 400 \le X_c < 560 \\ 2, & \text{jika } X_c \ge 560 \end{cases}$$

**Visualisasi pembagian zona:**

```
Lebar Kamera: 640 piksel
┌────────┬──────────────┬──────────────┬──────────────┬────────┐
│  80px  │   160px      │   160px      │   160px      │  80px  │
│ JAM 10 │   JAM 11     │   JAM 12     │   JAM 1      │ JAM 2  │
│ 0──79  │  80──239     │  240──399    │  400──559    │ 560─639│
│ ❌ ToF  │  ✅ ToF       │  ✅ ToF       │  ✅ ToF       │ ❌ ToF  │
└────────┴──────────────┴──────────────┴──────────────┴────────┘
```

**Alasan pembagian:**
- **Dead zone (80px tiap sisi):** Kamera OV2640 memiliki FoV horizontal ~66°, sedangkan sensor VL53L5CX memiliki FoV ~45°. Selisih sudut (~21°) menyebabkan area di tepi kiri dan kanan frame kamera **tidak ter-cover** oleh sensor jarak.
- **Zona 160px:** Area tengah 480px dibagi rata menjadi 3 zona (masing-masing 160px) untuk representasi arah jam 11, 12, dan 1 secara simetris.
- **Jam 10 dan 2:** Meskipun tidak memiliki data jarak, YOLO masih bisa mendeteksi objek di zona ini. Informasi arah tetap diberikan ke user, namun tanpa jarak presisi.

**Contoh:** $X_c = 190$ → $80 \le 190 < 240$ → **Jam 11**

---

## C. Formulasi Pemetaan Indeks Kolom ToF (Grid Binning)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3a
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-2

Untuk objek yang berada di dalam jangkauan sensor ToF ($80 \le X_c < 560$), sistem harus menentukan **indeks kolom matriks ToF** ($C_{index}$). Karena sensor VL53L5CX membagi area 480 piksel menjadi 8 kolom, maka setiap 1 titik (kolom) sensor mewakili tepat **60 piksel** layar kamera.

$$C_{index} = \left\lfloor \frac{X_c - 80}{60} \right\rfloor$$

> Fungsi floor $\lfloor \dots \rfloor$ membulatkan hasil ke bawah, sehingga piksel berapapun di dalam blok 60 piksel tersebut akan menghasilkan indeks integer yang sama.

| $C_{index}$ | Pixel Range | Zona Jam |
|---|---|---|
| 0 | 80 – 139 | Jam 11 |
| 1 | 140 – 199 | Jam 11 |
| 2 | 200 – 259 | Jam 11 / 12 |
| 3 | 260 – 319 | Jam 12 |
| 4 | 320 – 379 | Jam 12 |
| 5 | 380 – 439 | Jam 12 / 1 |
| 6 | 440 – 499 | Jam 1 |
| 7 | 500 – 559 | Jam 1 |

**Contoh:** $X_c = 350$ → $C_{index} = \lfloor (350 - 80) / 60 \rfloor = \lfloor 4.5 \rfloor = 4$ → **Kolom 4** (Jam 12)

---

## D. Formulasi Jarak Objek dari Sensor ToF

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3a

Setelah kolom ToF ditentukan, **jarak objek** diambil dari rata-rata pembacaan sensor pada baris tengah (baris 3, 4, dan 5) di kolom tersebut. Baris tengah dipilih karena sesuai dengan ketinggian badan objek (bukan langit di atas atau lantai di bawah).

$$D_{objek} = \frac{1}{3} \sum_{r=3}^{5} \text{ToF}[r][C_{index}]$$

| Simbol | Keterangan |
|---|---|
| $D_{objek}$ | Jarak objek yang terdeteksi (milimeter, dikonversi ke meter) |
| $\text{ToF}[r][c]$ | Pembacaan sensor pada baris $r$, kolom $c$ (mm) |
| $r = 3, 4, 5$ | Baris tengah sensor (area objek setinggi badan) |
| $C_{index}$ | Kolom sensor dari Formula C |

**Pembagian baris sensor VL53L5CX:**
```
Baris 0-2: Melihat ke atas    → Langit / objek tinggi (tidak dipakai)
Baris 3-5: Melihat ke tengah  → OBJEK setinggi badan ← dipakai untuk jarak
Baris 6-7: Melihat ke bawah   → Lantai ← dipakai untuk deteksi medan (Formula H)
```

**Contoh:** $C_{index} = 4$, pembacaan ToF:
$$D_{objek} = \frac{ToF[3][4] + ToF[4][4] + ToF[5][4]}{3} = \frac{2100 + 2050 + 2200}{3} = 2117 \text{ mm} \approx 2.1 \text{ m}$$

---

## E. Formulasi Threshold Adaptif (Jalur A)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3c
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-3a

Threshold peringatan bersifat **adaptif** — semakin cepat objek mendekat, semakin jauh threshold-nya sehingga user mendapat peringatan lebih awal. Formula ini berlaku untuk objek dalam jangkauan ToF (≤ 4 meter).

### E.1 Kecepatan Pendekatan

$$v = \frac{|\Delta D|}{{\Delta t}}$$

| Simbol | Keterangan |
|---|---|
| $v$ | Kecepatan pendekatan (m/s) |
| $\Delta D$ | Selisih jarak antar frame: $D_{lama} - D_{baru}$ (meter) |
| $\Delta t$ | Selisih waktu antar frame (detik) |

Kecepatan ini dihitung dari **salah satu** sumber:
- **User bergerak** (accelerometer aktif): $v = v_{user}$ — kecepatan user mendekati objek statis.
- **User diam + objek mendekat** ($\Delta D > 0$): $v = v_{objek}$ — kecepatan objek mendekati user.

### E.2 Threshold Peringatan

$$T = \min\left(1 + v \times 2,\ 4\right)$$

| Simbol | Keterangan |
|---|---|
| $T$ | Threshold peringatan (meter) |
| $1$ | Jarak aman minimum (1 meter) |
| $v \times 2$ | Jarak tambahan berdasarkan kecepatan × waktu reaksi (2 detik) |
| $4$ | Batas maksimum (jangkauan ToF = 4 meter) |

**Logika peringatan:**

$$\text{Peringatan} = \begin{cases} \text{Ya}, & \text{jika } D_{objek} < T \\ \text{Tidak}, & \text{jika } D_{objek} \ge T \end{cases}$$

**Contoh skenario:**

| Skenario | $v$ | $T$ | Objek di 2.5m | Peringatan? |
|---|---|---|---|---|
| User jalan pelan | 0.5 m/s | $\min(1+1, 4)$ = 2m | 2.5m ≥ 2m | ❌ Aman |
| User jalan cepat | 1.5 m/s | $\min(1+3, 4)$ = 4m | 2.5m < 4m | ✅ AWAS |
| Motor mendekat | 3 m/s | $\min(1+6, 4)$ = 4m | 2.5m < 4m | ✅ AWAS |
| Kedua diam | 0 m/s | 1m (tetap) | 2.5m ≥ 1m | ❌ Aman |

**Alasan konstanta:**
- **1 meter (minimum):** Jarak aman minimum saat user atau objek statis. Cukup untuk berhenti atau berbelok.
- **2 detik (reaction time):** Rata-rata waktu reaksi manusia dalam kondisi waspada. Dibutuhkan agar ada jarak cukup untuk bereaksi.
- **4 meter (maximum):** Batas jangkauan sensor VL53L5CX. Threshold tidak mungkin melampaui kemampuan sensor.

---

## F. Formulasi Peringatan Objek Statis (Jalur A — Cabang Ketiga)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3c
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-3a

Saat user diam DAN objek diam ($\Delta D = 0$), threshold adaptif tidak berlaku (kecepatan = 0). Peringatan hanya diberikan **satu kali** jika objek berada dalam jarak sangat dekat.

$$\text{Peringatan} = \begin{cases} \text{Ya (sekali)}, & \text{jika } D_{objek} < 1 \text{ m} \wedge \text{belum diperingatkan} \\ \text{Tidak}, & \text{jika } D_{objek} \ge 1 \text{ m} \vee \text{sudah diperingatkan} \end{cases}$$

**Mekanisme flag:**
```
flagObjek[id] = false  (default)

Jika D < 1m AND flagObjek[id] == false:
    → TTS: "Objek Dekat di Jam [X], [Jarak]m"
    → flagObjek[id] = true

Reset flag saat:
    → User bergerak (accelerometer berubah)
    → Objek bergerak (ΔD ≠ 0)
    → Objek hilang dari frame YOLO
```

Tujuannya: mencegah **spam suara** — user tidak perlu mendengar peringatan berulang tentang objek statis yang sudah diketahui.

---

## G. Formulasi Delta Bounding Box (Jalur B)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3d
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-3b

Untuk objek di luar jangkauan ToF (> 4 meter atau di dead zone), sistem menggunakan **perubahan ukuran bounding box** antar frame untuk mendeteksi objek yang mendekat secara cepat (terutama kendaraan).

### G.1 Area Bounding Box

$$A = (x_{max} - x_{min}) \times (y_{max} - y_{min})$$

### G.2 Delta Area Antar Frame

$$\Delta A = \frac{A_{baru} - A_{lama}}{A_{lama}} \times 100\%$$

### G.3 Logika Peringatan

$$\text{Peringatan} = \begin{cases} \text{Ya}, & \text{jika } \Delta A > 20\% \\ \text{Tidak}, & \text{jika } \Delta A \le 20\% \end{cases}$$

| Simbol | Keterangan |
|---|---|
| $A$ | Area (luas piksel) bounding box |
| $A_{baru}$ | Area bbox di frame saat ini |
| $A_{lama}$ | Area bbox di frame sebelumnya (objek yang sama) |
| $\Delta A$ | Persentase perubahan area |
| $20\%$ | Threshold signifikansi |

**Alasan threshold 20%:**
- Perubahan < 5%: noise normal (jitter YOLO, perubahan angle).
- Perubahan 5-20%: objek bergerak pelan atau paralel.
- Perubahan > 20%: objek **mendekati kamera dengan kecepatan tinggi** (kemungkinan besar kendaraan).

**Karakteristik Jalur B vs Jalur A:**

| Aspek | Jalur A (ToF) | Jalur B (BBox) |
|---|---|---|
| Jangkauan | ≤ 4 meter | > 4 meter |
| Presisi jarak | ✅ Presisi (mm) | ❌ Tidak ada jarak |
| Deteksi arah | ✅ Jam + jarak | ✅ Jam saja |
| Info peringatan | "Objek di Jam X, Ym" | "Kendaraan Mendekat dari Jam X" |
| Transisi | — | Saat objek ≤ 4m → beralih ke Jalur A |

**Contoh:** BBox motor: $A_{lama} = 5000\text{px}^2$, $A_{baru} = 6500\text{px}^2$:
$$\Delta A = \frac{6500 - 5000}{5000} \times 100\% = 30\% > 20\% \rightarrow \text{PERINGATAN}$$

---

## H. Formulasi Deteksi Medan (Jalur C)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3e
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-3c

Sistem mendeteksi perubahan medan (tangga, lubang, penurunan) dengan membandingkan pembacaan ToF antara baris bawah (lantai dekat kaki) dan baris tengah (lantai lebih jauh).

### H.1 Rata-rata Jarak per Zona

$$\bar{D}_{bawah} = \frac{1}{8} \sum_{c=0}^{7} \text{ToF}[r_{bawah}][c] \quad \text{dengan } r_{bawah} \in \{6, 7\}$$

$$\bar{D}_{tengah} = \frac{1}{8} \sum_{c=0}^{7} \text{ToF}[r_{tengah}][c] \quad \text{dengan } r_{tengah} \in \{4, 5\}$$

### H.2 Rasio Anomali Medan

$$R = \frac{\bar{D}_{bawah}}{\bar{D}_{tengah}}$$

### H.3 Logika Deteksi

$$\text{Anomali} = \begin{cases} \text{Ya}, & \text{jika } R > 0.8 \\ \text{Normal}, & \text{jika } R < 0.7 \end{cases}$$

**Penjelasan rasio:**
- **Lantai datar (normal):** Baris bawah (dekat kaki) lebih dekat → $\bar{D}_{bawah} << \bar{D}_{tengah}$ → $R < 0.7$
- **Anomali (lubang/tangga):** Baris bawah "hilang" (lantai menjauh/tidak ada) → $\bar{D}_{bawah} \approx \bar{D}_{tengah}$ → $R > 0.8$

### H.4 Analisis Pola (Saat Anomali Terdeteksi)

Saat $R > 0.8$, sistem menganalisis **standar deviasi per baris** untuk membedakan jenis anomali:

$$\sigma_{baris} = \sqrt{\frac{1}{8} \sum_{c=0}^{7} \left(\text{ToF}[r][c] - \bar{D}_{r}\right)^2}$$

| Pola | $\sigma$ | Interpretasi | Output |
|---|---|---|---|
| Gradual + $\sigma$ kecil | < threshold | Merata di semua kolom → **Tangga / Penurunan** | Cek YOLO untuk konfirmasi |
| Tiba-tiba + $\sigma$ besar | > threshold | Lokal di beberapa kolom → **Lubang / Parit** | Langsung peringatan |

---

## I. Formulasi Mode Darurat (Offline & Gelap)

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — sub-bab 3.5.4
**Referensi:** [sequence-diagram.md](file:///d:/Project/Skripsi/docs/sequence-diagram.md) — SD-4a & SD-4b

Pada mode darurat, ESP32 beroperasi **mandiri tanpa smartphone**. Semua formula adaptif (E, F, G, H) **tidak berlaku** karena tidak ada data accelerometer, YOLO, atau TTS.

### I.1 Threshold Tetap (Buzzer)

$$\text{Buzzer} = \begin{cases} \text{Aktif}, & \text{jika } D_{min} < 1 \text{ m} \\ \text{Diam}, & \text{jika } D_{min} \ge 1 \text{ m} \end{cases}$$

$$D_{min} = \min_{r,c} \left(\text{ToF}[r][c]\right) \quad \text{untuk } r \in \{0..7\},\ c \in \{0..7\}$$

**Alasan threshold tetap 1 meter:**
- Tidak ada accelerometer → kecepatan pendekatan tidak bisa dihitung.
- Tidak ada YOLO → objek tidak bisa diidentifikasi.
- 1 meter = jarak aman minimum untuk pejalan kaki berhenti atau berbelok.

### I.2 Kondisi Masuk Mode Darurat

$$\text{Mode} = \begin{cases} \text{Offline}, & \text{jika WiFi putus} > 5 \text{ detik} \\ \text{Gelap}, & \text{jika WiFi OK} \wedge B_{cam} < B_{threshold} \\ \text{Smart}, & \text{jika WiFi OK} \wedge B_{cam} \ge B_{threshold} \end{cases}$$

| Simbol | Keterangan |
|---|---|
| $B_{cam}$ | Rata-rata brightness frame kamera (0-255) |
| $B_{threshold}$ | Threshold kecerahan minimum untuk YOLO |

---

## J. Formulasi Kecepatan Pendekatan Relatif

**Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — Flowchart 3c

Formula tambahan untuk menghitung **siapa yang mendekat** berdasarkan kombinasi accelerometer dan delta jarak:

$$\text{Sumber Gerakan} = \begin{cases} \text{User bergerak}, & \text{jika } a_{acc} > a_{threshold} \\ \text{Objek mendekat}, & \text{jika } a_{acc} \le a_{threshold} \wedge \Delta D > 0 \\ \text{Kedua diam}, & \text{jika } a_{acc} \le a_{threshold} \wedge \Delta D = 0 \end{cases}$$

| Simbol | Keterangan |
|---|---|
| $a_{acc}$ | Magnitude accelerometer smartphone |
| $a_{threshold}$ | Threshold gerakan (empirik, ±0.3 g) |
| $\Delta D$ | Delta jarak: $D_{lama} - D_{baru}$ (positif = mendekat) |

---

## Ringkasan Seluruh Formula

| # | Formula | Input | Output | Dipakai di |
|---|---|---|---|---|
| **A** | $X_c = x_{min} + \frac{x_{max} - x_{min}}{2}$ | BBox YOLO | Titik tengah (px) | Flowchart 3a, SD-2 |
| **B** | Piecewise 5 zona ($<80, <240, <400, <560, \ge560$) | $X_c$ | Arah Jam (10-2) | Flowchart 3a, SD-2 |
| **C** | $C_{index} = \lfloor (X_c - 80) / 60 \rfloor$ | $X_c$ | Kolom ToF (0-7) | Flowchart 3a, SD-2 |
| **D** | $D = \frac{1}{3}\sum_{r=3}^{5} \text{ToF}[r][C]$ | Kolom ToF | Jarak (m) | Flowchart 3a, SD-2 |
| **E** | $T = \min(1 + v \times 2, 4)$ | Kecepatan | Threshold (m) | Flowchart 3c, SD-3a |
| **F** | $D < 1\text{m} \wedge \neg flag$ | Jarak + flag | Peringatan sekali | Flowchart 3c, SD-3a |
| **G** | $\Delta A = (A_{baru} - A_{lama}) / A_{lama} \times 100\%$ | BBox area | Peringatan kendaraan | Flowchart 3d, SD-3b |
| **H** | $R = \bar{D}_{bawah} / \bar{D}_{tengah}$ | ToF baris 6-7 vs 4-5 | Anomali medan | Flowchart 3e, SD-3c |
| **I** | $D_{min} < 1\text{m}$ → Buzzer | ToF semua zona | Buzzer ON/OFF | Flowchart 3.5.4, SD-4 |
| **J** | $a_{acc}$ vs $\Delta D$ | Accel + jarak | Sumber gerakan | Flowchart 3c, SD-3a |
