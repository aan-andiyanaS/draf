# Rencana Pengujian Sistem — Sistem Bantu Navigasi Tunanetra

Dokumen ini menyajikan **rencana pengujian** sistem IoT bantu navigasi tunanetra menggunakan metode **Black Box Testing**. Setiap skenario pengujian didefinisikan berdasarkan fungsionalitas yang dirancang pada BAB 3.

Dokumen ini sesuai dengan **BAB 4 — Pengujian dan Analisis** pada skripsi.

---

## 4.1 Metode Pengujian

Pengujian yang digunakan pada penelitian ini adalah **Black Box Testing** (*Behavioral Testing*). Metode ini mengevaluasi fungsionalitas sistem **tanpa melihat struktur internal kode** — fokus pada apakah input menghasilkan output yang sesuai dengan spesifikasi.

**Alasan pemilihan metode:**
- Sesuai untuk menguji sistem embedded (IoT) di mana interaksi user bersifat fisik.
- Memvalidasi bahwa seluruh fitur berjalan sesuai diagram alur dan formula yang dirancang di BAB 3.
- Dapat dilakukan tanpa pengetahuan mendalam tentang implementasi kode.

**Cakupan pengujian:**

| No | Kategori Pengujian | Referensi BAB 3 | Jumlah Kasus |
|---|---|---|---|
| 1 | Provisioning & Koneksi | sub-bab 3.5.2 | 6 kasus |
| 2 | Penentuan Mode Sistem | sub-bab 3.5.3 | 5 kasus |
| 3 | Deteksi & Mapping Objek | sub-bab 3.5.4 (Flowchart 3a) | 8 kasus |
| 4 | Threshold Adaptif (Jalur A) | sub-bab 3.5.4 (Flowchart 3c) | 7 kasus |
| 5 | Deteksi Kendaraan Jauh (Jalur B) | sub-bab 3.5.4 (Flowchart 3d) | 4 kasus |
| 6 | Deteksi Medan (Jalur C) | sub-bab 3.5.4 (Flowchart 3e) | 5 kasus |
| 7 | Mode Darurat | sub-bab 3.5.5 | 5 kasus |
| 8 | Antarmuka Pengguna & TTS | sub-bab 3.5.4 | 5 kasus |
|   | **Total** |  | **45 kasus** |

---

## 4.2 Skenario Pengujian

### 4.2.1 Pengujian Provisioning & Koneksi (P)

Menguji proses setup awal perangkat — dari power ON hingga koneksi WiFi berhasil. Sesuai dengan **Flowchart 3.5.2** dan **SD-1** pada BAB 3.

| ID | Skenario Pengujian | Kondisi Awal | Masukan / Aksi | Keluaran yang Diharapkan | Status |
|---|---|---|---|---|---|
| P-01 | Perangkat dinyalakan pertama kali (NVS kosong) | ESP32 dalam keadaan mati, belum pernah terhubung WiFi | Tekan tombol multifungsi (GPIO39) | ESP32 boot, LED indikator (GPIO48) berkedip cepat (mode provisioning), BLE Advertising aktif | ☐ |
| P-02 | Aplikasi mendeteksi perangkat via BLE | ESP32 dalam mode BLE Advertising | Buka aplikasi Android, aktifkan Bluetooth | Perangkat muncul di daftar scan BLE pada aplikasi | ☐ |
| P-03 | Proses provisioning WiFi berhasil | Aplikasi terhubung ke ESP32 via BLE | Pendamping memilih SSID dan memasukkan password WiFi yang benar | ESP32 terhubung ke WiFi, kredensial tersimpan di NVS, BLE diputus otomatis | ☐ |
| P-04 | Provisioning dengan password WiFi salah | Aplikasi terhubung ke ESP32 via BLE | Pendamping memasukkan password WiFi yang salah | Aplikasi menampilkan pesan error, user diminta memasukkan ulang password | ☐ |
| P-05 | Auto-connect saat perangkat dinyalakan ulang | ESP32 pernah terhubung WiFi (NVS ada kredensial) | Tekan tombol multifungsi (GPIO39) | ESP32 boot, LED menyala tetap (Mode Smart), auto-connect ke WiFi tanpa provisioning ulang | ☐ |
| P-06 | WebSocket terhubung setelah WiFi aktif | ESP32 terhubung ke WiFi smartphone | Buka aplikasi Android dengan WiFi aktif | Koneksi WebSocket antara ESP32 dan aplikasi berhasil, data mulai mengalir (video + ToF) | ☐ |

---

### 4.2.2 Pengujian Penentuan Mode Sistem (M)

Menguji logika pemilihan mode otomatis berdasarkan kondisi WiFi dan cahaya. Sesuai dengan **Flowchart 3.5.3** dan **SM-1** pada BAB 3.

| ID | Skenario Pengujian | Kondisi Awal | Masukan / Aksi | Keluaran yang Diharapkan | Status |
|---|---|---|---|---|---|
| M-01 | Masuk Mode Smart (WiFi OK + terang) | ESP32 terhubung WiFi, lingkungan terang | Sistem melakukan pengecekan otomatis | Sistem masuk ke Mode Smart: kamera aktif, YOLO aktif, video streaming berjalan | ☐ |
| M-02 | Masuk Mode Gelap (WiFi OK + gelap) | ESP32 terhubung WiFi, lingkungan gelap ($B_{cam} < B_{threshold}$) | Sistem melakukan pengecekan otomatis | Sistem masuk ke Mode Gelap: kamera low FPS, streaming dihentikan, sensor + buzzer aktif | ☐ |
| M-03 | Masuk Mode Offline (WiFi putus > 5 detik) | ESP32 sedang dalam Mode Smart | Matikan WiFi Hotspot di smartphone selama > 5 detik | Sistem masuk ke Mode Offline: kamera dimatikan, sensor + buzzer aktif, ESP32 mandiri | ☐ |
| M-04 | Transisi Gelap → Smart (cahaya membaik) | Sistem dalam Mode Gelap | Nyalakan lampu sehingga lingkungan kembali terang | Sistem otomatis kembali ke Mode Smart: kamera aktif, YOLO aktif, streaming dilanjutkan | ☐ |
| M-05 | Transisi Offline → Smart (WiFi reconnect) | Sistem dalam Mode Offline | Nyalakan kembali WiFi Hotspot di smartphone | ESP32 reconnect WiFi, WebSocket terhubung, sistem kembali ke Mode Smart | ☐ |

---

### 4.2.3 Pengujian Deteksi & Mapping Objek (D)

Menguji proses deteksi objek oleh YOLO, perhitungan titik tengah, pemetaan arah jam, dan pengambilan data jarak dari sensor ToF. Sesuai dengan **Flowchart 3a**, **SD-2**, dan **Formula A–D** pada BAB 3.

| ID | Skenario Pengujian | Kondisi Awal | Masukan / Aksi | Keluaran yang Diharapkan | Status |
|---|---|---|---|---|---|
| D-01 | Deteksi objek di zona Jam 12 (tengah) | Mode Smart aktif, YOLO berjalan | Letakkan objek (misalnya botol) tepat di depan kamera (area piksel 240–399) | YOLO mendeteksi objek; $X_c$ berada di rentang 240–399; TTS mengucapkan "... di arah Jam 12" | ☐ |
| D-02 | Deteksi objek di zona Jam 11 (kiri) | Mode Smart aktif, YOLO berjalan | Letakkan objek di sisi kiri pandangan kamera (area piksel 80–239) | $X_c$ berada di rentang 80–239; TTS mengucapkan "... di arah Jam 11" | ☐ |
| D-03 | Deteksi objek di zona Jam 1 (kanan) | Mode Smart aktif, YOLO berjalan | Letakkan objek di sisi kanan pandangan kamera (area piksel 400–559) | $X_c$ berada di rentang 400–559; TTS mengucapkan "... di arah Jam 1" | ☐ |
| D-04 | Deteksi objek di dead zone kiri (Jam 10) | Mode Smart aktif, YOLO berjalan | Letakkan objek di tepi kiri pandangan kamera (area piksel 0–79) | $X_c < 80$; TTS mengucapkan "... di arah Jam 10" **tanpa informasi jarak** | ☐ |
| D-05 | Deteksi objek di dead zone kanan (Jam 2) | Mode Smart aktif, YOLO berjalan | Letakkan objek di tepi kanan pandangan kamera (area piksel 560–639) | $X_c \ge 560$; TTS mengucapkan "... di arah Jam 2" **tanpa informasi jarak** | ☐ |
| D-06 | Akurasi jarak sensor ToF untuk objek pada jarak 1 meter | Mode Smart aktif, objek terdeteksi dalam zona ToF | Letakkan objek pada jarak **1 meter** dari sensor (diukur dengan meteran) | Jarak yang dilaporkan TTS berada dalam rentang **0.8–1.2 meter** (toleransi ±20%) | ☐ |
| D-07 | Akurasi jarak sensor ToF untuk objek pada jarak 2 meter | Mode Smart aktif, objek terdeteksi dalam zona ToF | Letakkan objek pada jarak **2 meter** dari sensor (diukur dengan meteran) | Jarak yang dilaporkan TTS berada dalam rentang **1.7–2.3 meter** (toleransi ±15%) | ☐ |
| D-08 | Akurasi jarak sensor ToF untuk objek pada jarak 4 meter | Mode Smart aktif, objek terdeteksi dalam zona ToF | Letakkan objek pada jarak **4 meter** dari sensor (diukur dengan meteran) | Jarak yang dilaporkan TTS berada dalam rentang **3.5–4.5 meter** (toleransi ±12%) | ☐ |

---

### 4.2.4 Pengujian Threshold Adaptif — Jalur A (T)

Menguji mekanisme peringatan adaptif untuk objek dalam jangkauan ToF (≤ 4m). Sesuai dengan **Flowchart 3c**, **SD-3a**, dan **Formula E, F, J** pada BAB 3.

| ID | Skenario Pengujian | Kondisi Awal | Masukan / Aksi | Keluaran yang Diharapkan | Status |
|---|---|---|---|---|---|
| T-01 | User berjalan pelan mendekati objek statis | Mode Otonom aktif, objek statis di jarak 3 meter | User berjalan mendekati objek dengan kecepatan ~0.5 m/s | $T = \min(1 + 0.5 \times 2,\ 4) = 2$ m; peringatan TTS saat jarak < 2 meter | ☐ |
| T-02 | User berjalan cepat mendekati objek statis | Mode Otonom aktif, objek statis di jarak 4 meter | User berjalan cepat mendekati objek dengan kecepatan ~1.5 m/s | $T = \min(1 + 1.5 \times 2,\ 4) = 4$ m; peringatan TTS lebih awal (dari jarak ~4 meter) | ☐ |
| T-03 | Objek bergerak mendekati user yang diam | Mode Otonom aktif, user berdiri diam | Seseorang berjalan mendekati user dari jarak 3 meter | Accelerometer diam; $\Delta D > 0$; threshold adaptif berdasarkan kecepatan objek; TTS "AWAS, objek mendekat" | ☐ |
| T-04 | User dan objek diam, jarak < 1 meter | Mode Otonom aktif, user berdiri di dekat tiang (~0.5m) | User dan objek sama-sama tidak bergerak | TTS memberikan peringatan **satu kali**: "Objek Dekat di Jam X, 0.5 meter" lalu **diam** | ☐ |
| T-05 | Peringatan statis tidak diulang | Lanjutan dari T-04 (peringatan sudah pernah diberikan) | User tetap diam di posisi yang sama | Sistem **tidak mengulang** peringatan (flag aktif); buzzer dan TTS diam | ☐ |
| T-06 | Reset flag setelah user bergerak | Lanjutan dari T-05 (flag aktif) | User bergerak beberapa langkah lalu kembali ke posisi semula | Flag di-reset; jika kembali < 1 meter dari objek, peringatan diberikan **satu kali lagi** | ☐ |
| T-07 | User dan objek diam, jarak > 1 meter | Mode Otonom aktif, objek statis di jarak 2 meter | User berdiri diam | **Tidak ada peringatan** karena $\Delta D = 0$ dan $D_{objek} > 1$ m — kedua syarat tidak terpenuhi | ☐ |

---

### 4.2.5 Pengujian Deteksi Kendaraan Jauh — Jalur B (K)

Menguji deteksi objek di luar jangkauan sensor ToF (> 4m) menggunakan perubahan ukuran bounding box. Sesuai dengan **Flowchart 3d**, **SD-3b**, dan **Formula G** pada BAB 3.

| ID | Skenario Pengujian | Kondisi Awal | Masukan / Aksi | Keluaran yang Diharapkan | Status |
|---|---|---|---|---|---|
| K-01 | Kendaraan mendekat cepat (ΔA > 20%) | Mode Otonom aktif, motor terdeteksi YOLO di kejauhan (> 4m) | Motor bergerak mendekati user dengan kecepatan tinggi | $\Delta A > 20\text{\%}$; TTS "AWAS, Kendaraan Mendekat dari Jam X" **tanpa info jarak** | ☐ |
| K-02 | Kendaraan bergerak paralel (ΔA ≤ 20%) | Mode Otonom aktif, kendaraan terdeteksi YOLO | Kendaraan bergerak melintas secara paralel (tidak mendekati) | $\Delta A \le 20\text{\%}$; **tidak ada peringatan** karena objek stabil / tidak mendekati | ☐ |
| K-03 | Orang berjalan dari jauh mendekati user | Mode Otonom aktif, orang terdeteksi YOLO | Orang berjalan mendekati user dari jarak > 4 meter | $\Delta A$ meningkat gradual tapi < 20%; **tidak ada peringatan** sampai masuk jangkauan ToF | ☐ |
| K-04 | Transisi Jalur B ke Jalur A | Lanjutan dari K-03, objek terus mendekat | Orang terus berjalan hingga jarak ≤ 4 meter dari sensor | Sistem **otomatis beralih** ke Jalur A; peringatan berubah menjadi TTS dengan info jarak presisi | ☐ |

---

### 4.2.6 Pengujian Deteksi Medan — Jalur C (MD)

Menguji kemampuan sistem mendeteksi perubahan elevasi lantai menggunakan sensor ToF. Sesuai dengan **Flowchart 3e**, **SD-3c**, dan **Formula H** pada BAB 3.

| ID | Skenario Pengujian | Kondisi Awal | Masukan / Aksi | Keluaran yang Diharapkan | Status |
|---|---|---|---|---|---|
| MD-01 | Lantai datar — kondisi normal | Mode Otonom aktif, user berjalan di lantai datar | User berjalan normal di koridor / trotoar datar | $R < 0.7$ (normal); **tidak ada peringatan** medan | ☐ |
| MD-02 | Tangga turun terdeteksi + YOLO konfirmasi | Mode Otonom aktif, user mendekati tangga turun | User berjalan mendekati tangga turun yang terlihat jelas | $R > 0.8$ (anomali); pola gradual + $\sigma$ kecil; YOLO deteksi tangga → TTS: "Tangga di depan" (informatif, bukan peringatan) | ☐ |
| MD-03 | Penurunan terdeteksi + YOLO tidak konfirmasi | Mode Otonom aktif, user mendekati lereng turun | User berjalan mendekati area yang menurun gradual tapi bukan tangga visual | $R > 0.8$; pola gradual + $\sigma$ kecil; YOLO tidak deteksi tangga → TTS: "AWAS, penurunan di depan!" | ☐ |
| MD-04 | Lubang / parit terdeteksi | Mode Otonom aktif, user mendekati lubang di jalan | User berjalan mendekati lubang / parit di trotoar | $R > 0.8$; pola tiba-tiba + $\sigma$ besar → TTS: "AWAS, lubang di depan!" (langsung peringatan, tanpa konfirmasi YOLO) | ☐ |
| MD-05 | Permukaan bergelombang (bukan anomali) | Mode Otonom aktif, user berjalan di jalan bergelombang | User berjalan di permukaan yang sedikit tidak rata | $R$ berfluktuasi di sekitar 0.7; **tidak melebihi 0.8**; tidak ada peringatan | ☐ |

---

### 4.2.7 Pengujian Mode Darurat (E)

Menguji mekanisme fail-safe saat WiFi putus atau kondisi gelap. Sesuai dengan **Flowchart 3.5.5**, **SD-4a**, **SD-4b**, dan **Formula I** pada BAB 3.

| ID | Skenario Pengujian | Kondisi Awal | Masukan / Aksi | Keluaran yang Diharapkan | Status |
|---|---|---|---|---|---|
| E-01 | Mode Offline — buzzer aktif saat objek < 1m | Sistem dalam Mode Offline (WiFi putus) | Letakkan objek di depan sensor pada jarak < 1 meter | $D_{min} < 1$ m; buzzer berbunyi **langsung dari ESP32** (tanpa smartphone) | ☐ |
| E-02 | Mode Offline — buzzer diam saat objek ≥ 1m | Sistem dalam Mode Offline (WiFi putus) | Jauhkan semua objek dari sensor (jarak > 1 meter) | $D_{min} \ge 1$ m; buzzer **tidak berbunyi** | ☐ |
| E-03 | Mode Gelap — buzzer aktif saat objek < 1m | Sistem dalam Mode Gelap (cahaya kurang) | Letakkan objek di depan sensor pada jarak < 1 meter dalam kondisi gelap | $D_{min} < 1$ m; buzzer berbunyi; logika identik dengan Mode Offline | ☐ |
| E-04 | Mode Offline — reconnect otomatis | Sistem dalam Mode Offline, sensor loop berjalan | Nyalakan kembali WiFi Hotspot di smartphone | ESP32 reconnect WiFi; notifikasi ke App; sistem kembali ke Mode Smart | ☐ |
| E-05 | Mode Gelap — transisi saat cahaya membaik | Sistem dalam Mode Gelap, kamera low FPS | Nyalakan lampu penerangan di sekitar | $B_{cam} \ge B_{threshold}$; streaming dilanjutkan; sistem kembali ke Mode Smart | ☐ |

---

### 4.2.8 Pengujian Antarmuka Pengguna & TTS (U)

Menguji interaksi user dengan sistem — mode aplikasi, output suara, dan penghematan daya. Sesuai dengan **Flowchart 3b**, **SD-2**, dan **Use Case Diagram** pada BAB 3.

| ID | Skenario Pengujian | Kondisi Awal | Masukan / Aksi | Keluaran yang Diharapkan | Status |
|---|---|---|---|---|---|
| U-01 | Mode Otonom — peringatan hanya saat bahaya | Mode Otonom aktif, beberapa objek terdeteksi | User berjalan di lingkungan dengan objek jauh (> threshold) dan satu objek dekat | TTS **hanya** memperingatkan objek yang melebihi threshold; objek aman **tidak disebutkan** | ☐ |
| U-02 | Mode Tanya Jawab — lapor semua objek | Mode Tanya Jawab aktif | User menekan tombol 2× cepat (double press GPIO39) | TTS **menyebutkan semua objek** yang terdeteksi beserta arah jam dan jarak masing-masing | ☐ |
| U-03 | Ganti mode via tombol multifungsi | Sistem dalam Mode Otonom | User menekan tombol 1× singkat (GPIO39) | TTS mengkonfirmasi: "Mode Tanya Jawab aktif"; LED berubah pola (2× kedip + jeda); sistem beralih ke Mode Tanya Jawab | ☐ |
| U-04 | Auto-pause YOLO saat user diam > 10 detik | Mode Smart aktif, YOLO berjalan | User berdiri diam selama > 10 detik | YOLO dan video streaming **di-pause**; sensor ToF + buzzer **tetap aktif** untuk keamanan | ☐ |
| U-05 | Anti-tumpang-tindih TTS (callback) | Mode Otonom aktif, beberapa peringatan bersamaan | Sistem mendeteksi 2 objek berbahaya sekaligus | TTS mengucapkan peringatan **satu per satu** (pesan kedua menunggu callback dari pesan pertama); tidak ada suara yang bertumpuk | ☐ |

---

## 4.3 Rekapitulasi Hasil Pengujian

Tabel berikut diisi setelah seluruh pengujian dilaksanakan.

| No | Kategori Pengujian | Jumlah Kasus | Berhasil (✅) | Gagal (❌) | Persentase Keberhasilan |
|---|---|---|---|---|---|
| 1 | Provisioning & Koneksi | 6 | — | — | — |
| 2 | Penentuan Mode Sistem | 5 | — | — | — |
| 3 | Deteksi & Mapping Objek | 8 | — | — | — |
| 4 | Threshold Adaptif (Jalur A) | 7 | — | — | — |
| 5 | Deteksi Kendaraan Jauh (Jalur B) | 4 | — | — | — |
| 6 | Deteksi Medan (Jalur C) | 5 | — | — | — |
| 7 | Mode Darurat | 5 | — | — | — |
| 8 | Antarmuka Pengguna & TTS | 5 | — | — | — |
|   | **Total** | **45** | **—** | **—** | **—** |

### Formula Persentase Keberhasilan

$$P = \frac{\text{Jumlah Kasus Berhasil}}{\text{Total Kasus Uji}} \times 100\text{\%}$$

---

## 4.4 Pengujian Akurasi Jarak (Tambahan Kuantitatif)

Selain Black Box Testing, dilakukan pengujian kuantitatif untuk mengukur **akurasi jarak sensor VL53L5CX** terhadap jarak sebenarnya. Pengujian ini penting untuk memvalidasi Formula D.

### Prosedur Pengujian

1. Letakkan objek (papan datar) pada jarak yang sudah diukur dengan meteran.
2. Catat pembacaan sensor ToF (rata-rata baris 3-5, kolom tengah).
3. Hitung error: $\text{Error} = |D_{sensor} - D_{aktual}|$.
4. Ulangi untuk 5 jarak berbeda, masing-masing 3 kali pengulangan.

### Tabel Pengujian Akurasi Jarak

| No | Jarak Aktual (m) | Percobaan 1 (m) | Percobaan 2 (m) | Percobaan 3 (m) | Rata-rata (m) | Error (m) | Error (%) |
|---|---|---|---|---|---|---|---|
| 1 | 0.5 | — | — | — | — | — | — |
| 2 | 1.0 | — | — | — | — | — | — |
| 3 | 2.0 | — | — | — | — | — | — |
| 4 | 3.0 | — | — | — | — | — | — |
| 5 | 4.0 | — | — | — | — | — | — |

### Formula Error

$$\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |D_{sensor,i} - D_{aktual,i}|$$

$$\text{MAPE} = \frac{1}{n} \sum_{i=1}^{n} \frac{|D_{sensor,i} - D_{aktual,i}|}{D_{aktual,i}} \times 100\text{\%}$$

| Simbol | Keterangan |
|---|---|
| $\text{MAE}$ | *Mean Absolute Error* — rata-rata selisih absolut dalam satuan meter |
| $\text{MAPE}$ | *Mean Absolute Percentage Error* — rata-rata persentase error |
| $D_{sensor,i}$ | Jarak yang dilaporkan sensor pada pengujian ke-$i$ |
| $D_{aktual,i}$ | Jarak aktual (diukur meteran) pada pengujian ke-$i$ |
| $n$ | Jumlah total data pengujian |

---

## 4.5 Pengujian Akurasi Deteksi Objek YOLO (Tambahan Kuantitatif)

Pengujian ini mengukur kemampuan model YOLOv11 Nano (TFLite) dalam mendeteksi objek pada berbagai kondisi. Pengujian ini memvalidasi reliabilitas Formula A (centroid bounding box).

### Prosedur Pengujian

1. Siapkan **5 jenis objek** yang umum ditemui tunanetra (misalnya: orang, kursi, tiang, motor, tangga).
2. Untuk setiap objek, lakukan deteksi pada **3 kondisi cahaya** (terang, redup, gelap) dan **3 jarak** (dekat 1m, sedang 2m, jauh 4m).
3. Catat apakah YOLO berhasil mendeteksi objek (True Positive) atau gagal (False Negative).

### Tabel Pengujian Akurasi Deteksi

| No | Jenis Objek | Jarak (m) | Kondisi Cahaya | Terdeteksi? | Confidence (%) |
|---|---|---|---|---|---|
| 1 | Orang | 1 | Terang | — | — |
| 2 | Orang | 2 | Terang | — | — |
| 3 | Orang | 4 | Terang | — | — |
| 4 | Orang | 1 | Redup | — | — |
| 5 | Orang | 2 | Redup | — | — |
| ... | ... | ... | ... | — | — |

### Metrik Evaluasi

$$\text{Precision} = \frac{TP}{TP + FP}$$

$$\text{Recall} = \frac{TP}{TP + FN}$$

$$\text{F1-Score} = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}$$

| Simbol | Keterangan |
|---|---|
| $TP$ | *True Positive* — objek ada dan terdeteksi benar |
| $FP$ | *False Positive* — tidak ada objek tapi terdeteksi (deteksi palsu) |
| $FN$ | *False Negative* — objek ada tapi tidak terdeteksi (terlewat) |

---

## 4.6 Pengujian Latensi Sistem (Tambahan Non-Fungsional)

Pengujian ini mengukur waktu respons dari deteksi objek hingga output TTS. Penting untuk memastikan user mendapat peringatan **tepat waktu**.

### Prosedur Pengujian

1. Catat timestamp saat ESP32 mengirim frame video ke smartphone.
2. Catat timestamp saat TTS selesai mengucapkan peringatan.
3. Hitung latensi: $L = t_{TTS} - t_{frame}$.
4. Ulangi sebanyak 10 kali untuk mendapatkan rata-rata.

### Tabel Pengujian Latensi

| No | $t_{frame}$ (ms) | $t_{YOLO}$ (ms) | $t_{mapping}$ (ms) | $t_{TTS}$ (ms) | Latensi Total (ms) |
|---|---|---|---|---|---|
| 1 | — | — | — | — | — |
| 2 | — | — | — | — | — |
| ... | ... | ... | ... | ... | ... |
| 10 | — | — | — | — | — |
|  | | | | **Rata-rata:** | **— ms** |

### Breakdown Latensi yang Diharapkan

| Tahap | Target Latensi | Keterangan |
|---|---|---|
| Pengiriman frame (ESP32 → App) | < 100 ms | Tergantung kualitas WiFi |
| YOLO Inference (TFLite) | < 200 ms | YOLOv11 Nano pada HP menengah |
| Mapping + Threshold | < 10 ms | Perhitungan ringan |
| TTS (text-to-speech) | < 500 ms | Tergantung panjang kalimat |
| **Total** | **< 800 ms** | Target 1 detik end-to-end |
