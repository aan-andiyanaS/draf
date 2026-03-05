# Alur Penelitian — Metodologi Skripsi

Dokumen ini menjelaskan alur penelitian secara keseluruhan, dimulai dari flowchart garis besar lalu dipecah menjadi detail per tahapan.

---

## 0. Garis Besar Alur Penelitian

Flowchart berikut menunjukkan **tahapan utama** penelitian secara ringkas. Setiap tahapan memiliki flowchart detail di bagian selanjutnya.

```mermaid
flowchart TB
    START(["Mulai"]) --> T1["Tahap 1:<br/>Penelitian Awal"]
    T1 --> T2["Tahap 2:<br/>Analisis & Perancangan"]
    T2 --> T3["Tahap 3:<br/>Implementasi"]
    T3 --> T4{"Tahap 4:<br/>Pengujian"}
    T4 -- Gagal --> T3
    T4 -- Berhasil --> T5["Tahap 5:<br/>Evaluasi & Kesimpulan"]
    T5 --> SELESAI(["Selesai"])

    T1:::research
    T2:::design
    T3:::impl
    T4:::decision
    T5:::research

    classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
```

**Penjelasan garis besar:**

| Tahap | Nama | Isi Utama |
|---|---|---|
| 1 | Penelitian Awal | Identifikasi masalah, studi literatur, pengumpulan data |
| 2 | Analisis & Perancangan | Analisis kebutuhan HW/SW, desain arsitektur/mekanik/logika |
| 3 | Implementasi | Training AI, perakitan hardware, coding aplikasi |
| 4 | Pengujian | Testing sistem, debugging jika gagal |
| 5 | Evaluasi & Kesimpulan | Analisis hasil, penarikan kesimpulan & saran |

---

## 1. Tahap Penelitian Awal

Tahap awal yang mencakup identifikasi masalah, kajian literatur terkait, dan pengumpulan data yang dibutuhkan.

```mermaid
flowchart TB
    START(["Mulai"]) --> IDENTIFIKASI["Identifikasi Masalah:<br/>Keterbatasan Tongkat Putih<br/>& Risiko Cedera Kepala"]

    IDENTIFIKASI --> STUDI_1["Studi Literatur:<br/>Alat Bantu Tunanetra<br/>(Smart Stick / Smart Glasses)"]
    IDENTIFIKASI --> STUDI_2["Studi Literatur:<br/>Deep Learning<br/>YOLOv11 Object Detection"]
    IDENTIFIKASI --> STUDI_3["Studi Literatur:<br/>Sensor ToF VL53L5CX<br/>& IoT ESP32-S3"]

    STUDI_1 --> PENGUMPULAN
    STUDI_2 --> PENGUMPULAN
    STUDI_3 --> PENGUMPULAN

    PENGUMPULAN["Pengumpulan Data"]
    PENGUMPULAN --> DATA_1["Dataset Sekunder:<br/>COCO / Open Images<br/>(Pre-trained Base)"]
    PENGUMPULAN --> DATA_2["Dataset Primer:<br/>Foto Rintangan Lokal<br/>(Tiang, Lubang, Kendaraan)"]
    PENGUMPULAN --> DATA_3["Observasi Lapangan:<br/>Perilaku & Kebutuhan<br/>Tunanetra"]

    DATA_1 --> LANJUT(["Lanjut ke Tahap 2"])
    DATA_2 --> LANJUT
    DATA_3 --> LANJUT

    IDENTIFIKASI:::research
    STUDI_1:::research
    STUDI_2:::research
    STUDI_3:::research
    PENGUMPULAN:::research
    DATA_1:::research
    DATA_2:::research
    DATA_3:::research

    classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
```

**Penjelasan langkah demi langkah:**

1. **Identifikasi Masalah** — Penelitian dimulai dengan mengidentifikasi permasalahan utama: tongkat putih konvensional hanya mendeteksi rintangan di bawah pinggang, sehingga tunanetra rentan terhadap cedera kepala akibat rintangan setinggi kepala (tiang rambu, dahan pohon, kanopi toko).
2. **Studi Literatur** — Tiga arah kajian dilakukan secara paralel:
   - **Alat bantu tunanetra** yang sudah ada (smart stick, smart glasses), kelebihan dan kekurangannya.
   - **Deep learning YOLOv11** sebagai model object detection real-time yang ringan untuk perangkat mobile.
   - **Sensor ToF VL53L5CX** sebagai sensor jarak multizone 8×8 dan **ESP32-S3** sebagai mikrokontroler IoT dengan WiFi/BLE.
3. **Pengumpulan Data** — Tiga jenis data dikumpulkan:
   - **Dataset sekunder** (COCO/Open Images) sebagai basis pre-trained model.
   - **Dataset primer** berupa foto rintangan khas lingkungan lokal Indonesia.
   - **Observasi lapangan** untuk memahami perilaku navigasi tunanetra dan kebutuhan mereka.

---

## 2. Tahap Analisis & Perancangan

Tahap analisis kebutuhan sistem dan perancangan desain secara menyeluruh sebelum implementasi.

```mermaid
flowchart TB
    START(["Dari Tahap 1"]) --> ANALISIS["Analisis Kebutuhan Sistem"]

    ANALISIS --> HW["Kebutuhan Hardware"]
    ANALISIS --> SW["Kebutuhan Software"]

    HW --> HW_1["ESP32-S3 N16R8<br/>+ Kamera OV2640"]
    HW --> HW_2["Sensor VL53L5CX<br/>(ToF 8×8 Multizone)"]
    HW --> HW_3["Buzzer, Baterai,<br/>Casing Kacamata"]

    SW --> SW_1["Aplikasi Android<br/>(Kotlin + WebSocket)"]
    SW --> SW_2["Model YOLOv11 Nano<br/>(TFLite Custom)"]
    SW --> SW_3["Library: TensorFlow Lite,<br/>OkHttp, Android TTS"]

    HW_1 --> PERANCANGAN["Perancangan Sistem"]
    HW_2 --> PERANCANGAN
    HW_3 --> PERANCANGAN
    SW_1 --> PERANCANGAN
    SW_2 --> PERANCANGAN
    SW_3 --> PERANCANGAN

    PERANCANGAN --> DESAIN_A["Desain Arsitektur:<br/>Diagram Blok Sistem<br/>& Wiring Skematik"]
    PERANCANGAN --> DESAIN_M["Desain Mekanik:<br/>Casing Kacamata<br/>& Tata Letak Komponen"]
    PERANCANGAN --> DESAIN_L["Desain Logika:<br/>Flowchart, Sequence Diagram,<br/>Mapping Sensor 8×8"]

    DESAIN_A --> LANJUT(["Lanjut ke Tahap 3"])
    DESAIN_M --> LANJUT
    DESAIN_L --> LANJUT

    ANALISIS:::design
    HW:::design
    SW:::design
    HW_1:::design
    HW_2:::design
    HW_3:::design
    SW_1:::design
    SW_2:::design
    SW_3:::design
    PERANCANGAN:::design
    DESAIN_A:::design
    DESAIN_M:::design
    DESAIN_L:::design

    classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
```

**Penjelasan langkah demi langkah:**

1. **Analisis Kebutuhan Sistem** — Berdasarkan hasil studi literatur dan observasi, ditentukan kebutuhan hardware dan software secara spesifik.
2. **Kebutuhan Hardware** — Tiga kelompok komponen:
   - **ESP32-S3 N16R8 + Kamera OV2640**: Mikrokontroler utama dengan kamera 2MP untuk streaming video.
   - **VL53L5CX**: Sensor jarak Time-of-Flight multizone 8×8 untuk deteksi jarak akurat.
   - **Komponen pendukung**: Buzzer untuk peringatan darurat (fail-safe), baterai untuk portabilitas, dan casing kacamata sebagai wadah wearable.
3. **Kebutuhan Software** — Tiga komponen perangkat lunak:
   - **Aplikasi Android** berbasis Kotlin dengan WebSocket untuk komunikasi real-time.
   - **Model YOLOv11 Nano** yang dikonversi ke TFLite untuk inferensi ringan di smartphone.
   - **Library pendukung**: TensorFlow Lite (AI), OkHttp (WebSocket), Android TTS (suara).
4. **Perancangan Sistem** — Tiga desain dibuat secara paralel:
   - **Arsitektur**: Diagram blok sistem (wearable ↔ smartphone ↔ user) dan wiring skematik komponen elektronik.
   - **Mekanik**: Desain casing kacamata dan penempatan fisik komponen (kamera depan, sensor ToF, ESP32).
   - **Logika**: Flowchart alur kerja sistem, sequence diagram komunikasi antar aktor, dan logika mapping sensor 8×8 ke arah jam.

---

## 3. Tahap Implementasi

Tahap pembuatan dan pengembangan sistem berdasarkan desain yang telah dirancang.

```mermaid
flowchart TB
    START(["Dari Tahap 2"]) --> IMPL["Implementasi & Pembuatan Alat"]

    IMPL --> TRAINING["Training Model AI"]
    IMPL --> RAKIT["Perakitan Hardware<br/>& Coding"]

    %% Jalur Training AI
    TRAINING --> PREP_DATA["Persiapan Dataset:<br/>Labeling & Augmentasi"]
    PREP_DATA --> TRAIN_MODEL["Training YOLOv11 Nano<br/>(Google Colab / Local)"]
    TRAIN_MODEL --> EVAL_MODEL{"Evaluasi Model:<br/>mAP ≥ Target?"}
    EVAL_MODEL -- Tidak --> TUNE["Tuning Hyperparameter<br/>& Tambah Data"]
    TUNE --> TRAIN_MODEL
    EVAL_MODEL -- Ya --> CONVERT["Konversi ke TFLite<br/>(INT8 Quantized)"]
    CONVERT --> INTEGRASI["Integrasikan ke<br/>Aplikasi Android"]

    %% Jalur Hardware & Coding
    RAKIT --> RAKIT_HW["Rakit ESP32 + Kamera<br/>+ Sensor + Buzzer"]
    RAKIT --> CODING_ESP["Coding Firmware<br/>ESP32 (Arduino/ESP-IDF)"]
    RAKIT --> CODING_APP["Coding Aplikasi Android<br/>(Kotlin + WebSocket)"]

    RAKIT_HW --> INTEGRASI_FULL["Integrasi Penuh:<br/>Hardware + Software + AI"]
    CODING_ESP --> INTEGRASI_FULL
    CODING_APP --> INTEGRASI
    INTEGRASI --> INTEGRASI_FULL

    INTEGRASI_FULL --> LANJUT(["Lanjut ke Tahap 4"])

    IMPL:::impl
    TRAINING:::impl
    RAKIT:::impl
    PREP_DATA:::impl
    TRAIN_MODEL:::impl
    EVAL_MODEL:::decision
    TUNE:::impl
    CONVERT:::impl
    INTEGRASI:::impl
    RAKIT_HW:::impl
    CODING_ESP:::impl
    CODING_APP:::impl
    INTEGRASI_FULL:::impl

    classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
```

**Penjelasan langkah demi langkah:**

1. **Implementasi** — Tahap ini dibagi menjadi dua jalur paralel: training AI dan perakitan hardware.
2. **Jalur Training Model AI:**
   - **Persiapan dataset**: Labeling gambar rintangan menggunakan tools (Roboflow/CVAT), augmentasi data (rotasi, flip, brightness) untuk memperbanyak variasi.
   - **Training**: Melatih model YOLOv11 Nano menggunakan Google Colab (GPU) atau mesin lokal. Output berupa file weight model.
   - **Evaluasi**: Mengukur mAP (mean Average Precision). Jika belum mencapai target → tuning hyperparameter dan menambah data, lalu training ulang.
   - **Konversi TFLite**: Model yang lolos evaluasi dikonversi ke format TFLite dengan kuantisasi INT8 agar ringan di smartphone.
   - **Integrasi ke Android**: Model TFLite dimasukkan ke dalam aplikasi Android dan diuji inferensi dasar.
3. **Jalur Perakitan Hardware & Coding:**
   - **Rakit hardware**: Merakit ESP32-S3 dengan kamera OV2640, sensor VL53L5CX, dan buzzer ke dalam casing kacamata.
   - **Coding firmware ESP32**: Menulis kode untuk streaming video via WebSocket, membaca sensor I2C, dan mengendalikan buzzer.
   - **Coding aplikasi Android**: Membuat aplikasi Kotlin yang menerima stream video, menampilkan UI, dan menjalankan logika fusion.
4. **Integrasi Penuh** — Menggabungkan seluruh komponen: hardware yang sudah dirakit + firmware ESP32 + aplikasi Android + model AI TFLite menjadi satu sistem terintegrasi.

---

## 4. Tahap Pengujian

Tahap pengujian fungsionalitas dan performa sistem secara keseluruhan.

```mermaid
flowchart TB
    START(["Dari Tahap 3"]) --> PENGUJIAN{"Pengujian Sistem"}

    PENGUJIAN --> UJI_FUNG["Pengujian Fungsional"]
    PENGUJIAN --> UJI_PERF["Pengujian Performa"]
    PENGUJIAN --> UJI_USER["Pengujian Usability"]

    UJI_FUNG --> F1["Uji Koneksi:<br/>WiFi, BLE Provisioning"]
    UJI_FUNG --> F2["Uji Deteksi Objek:<br/>YOLO + Arah Jam"]
    UJI_FUNG --> F3["Uji Sensor Jarak:<br/>VL53L5CX Akurasi"]
    UJI_FUNG --> F4["Uji Mode Offline<br/>& Mode Gelap"]

    UJI_PERF --> P1["Uji Latensi:<br/>End-to-End Delay"]
    UJI_PERF --> P2["Uji FPS:<br/>Frame Rate Deteksi"]
    UJI_PERF --> P3["Uji Daya Tahan:<br/>Konsumsi Baterai"]

    UJI_USER --> U1["Uji Coba oleh<br/>Pengguna Tunanetra"]
    UJI_USER --> U2["Kuesioner<br/>Kepuasan & Keamanan"]

    F1 --> HASIL{"Semua Lolos?"}
    F2 --> HASIL
    F3 --> HASIL
    F4 --> HASIL
    P1 --> HASIL
    P2 --> HASIL
    P3 --> HASIL
    U1 --> HASIL
    U2 --> HASIL

    HASIL -- Gagal / Error --> DEBUG["Perbaikan & Debugging"]
    DEBUG --> KEMBALI(["Kembali ke Tahap 3"])
    HASIL -- Berhasil --> LANJUT(["Lanjut ke Tahap 5"])

    PENGUJIAN:::decision
    UJI_FUNG:::decision
    UJI_PERF:::decision
    UJI_USER:::decision
    F1:::research
    F2:::research
    F3:::research
    F4:::research
    P1:::research
    P2:::research
    P3:::research
    U1:::research
    U2:::research
    HASIL:::decision
    DEBUG:::impl

    classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
```

**Penjelasan langkah demi langkah:**

1. **Pengujian Fungsional** — Menguji apakah setiap fitur bekerja sesuai desain:
   - **Koneksi**: BLE provisioning pertama kali, WebSocket streaming setelah WiFi terhubung.
   - **Deteksi objek**: YOLO mendeteksi rintangan dengan benar dan mapping ke arah jam (10-2) akurat.
   - **Sensor jarak**: VL53L5CX menghasilkan data jarak yang akurat pada setiap zona 8×8.
   - **Mode fallback**: Mode Offline (buzzer saat WiFi putus) dan Mode Gelap (sensor only saat cahaya rendah) berfungsi.
2. **Pengujian Performa** — Mengukur metrik kuantitatif:
   - **Latensi**: Waktu dari capture frame hingga output suara TTS (end-to-end delay).
   - **FPS**: Berapa frame per detik yang berhasil diproses oleh YOLO secara real-time.
   - **Daya tahan**: Seberapa lama baterai bertahan dengan penggunaan normal.
3. **Pengujian Usability** — Menguji pengalaman pengguna sebenarnya:
   - **Uji coba** langsung oleh pengguna tunanetra dalam skenario navigasi nyata.
   - **Kuesioner** untuk mengukur tingkat kepuasan, rasa aman, dan kemudahan penggunaan.
4. **Hasil Pengujian** — Jika ada yang gagal, kembali ke Tahap 3 (Implementasi) untuk perbaikan dan debugging. Jika semua tes lolos, lanjut ke evaluasi akhir.

---

## 5. Tahap Evaluasi & Kesimpulan

Tahap akhir: menganalisis seluruh hasil pengujian, menarik kesimpulan, dan memberikan saran untuk pengembangan selanjutnya.

```mermaid
flowchart TB
    START(["Dari Tahap 4"]) --> ANALISIS_HASIL["Analisis Hasil Pengujian"]

    ANALISIS_HASIL --> A1["Analisis Akurasi:<br/>mAP Deteksi Objek<br/>& Akurasi Jarak"]
    ANALISIS_HASIL --> A2["Analisis Latensi:<br/>Rata-rata Delay<br/>& Bottleneck"]
    ANALISIS_HASIL --> A3["Analisis Usability:<br/>Feedback Pengguna<br/>& Skor Kuesioner"]

    A1 --> KOMPARASI["Komparasi dengan<br/>Penelitian Terdahulu"]
    A2 --> KOMPARASI
    A3 --> KOMPARASI

    KOMPARASI --> KESIMPULAN["Penarikan Kesimpulan:<br/>Menjawab Rumusan Masalah"]
    KESIMPULAN --> SARAN["Saran Pengembangan:<br/>Fitur Tambahan, Optimasi,<br/>Skala Produksi"]
    SARAN --> SELESAI(["Selesai"])

    ANALISIS_HASIL:::research
    A1:::research
    A2:::research
    A3:::research
    KOMPARASI:::research
    KESIMPULAN:::research
    SARAN:::design

    classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
```

**Penjelasan langkah demi langkah:**

1. **Analisis Hasil Pengujian** — Data dari pengujian Tahap 4 dianalisis dalam tiga dimensi:
   - **Akurasi**: Nilai mAP (mean Average Precision) model YOLO pada dataset custom, serta akurasi pengukuran jarak sensor VL53L5CX dibandingkan jarak aktual.
   - **Latensi**: Rata-rata waktu delay end-to-end dan identifikasi bottleneck sistem (apakah di streaming, inferensi, atau TTS).
   - **Usability**: Skor kuesioner dari pengguna tunanetra dan rangkuman feedback kualitatif.
2. **Komparasi** — Hasil analisis dibandingkan dengan penelitian terdahulu di bidang yang sama (smart stick, smart glasses) untuk menunjukkan kontribusi dan keunggulan/kekurangan sistem yang dibuat.
3. **Penarikan Kesimpulan** — Menjawab setiap rumusan masalah yang telah diidentifikasi di Tahap 1 berdasarkan data hasil pengujian.
4. **Saran Pengembangan** — Rekomendasi untuk penelitian selanjutnya, misalnya: penambahan fitur GPS navigasi, optimasi model AI ke versi lebih ringan, atau desain casing yang lebih ergonomis untuk produksi massal.
