# Daftar Materi BAB 2 — Tinjauan Pustaka

Dokumen ini berisi **daftar topik** yang perlu dijelaskan di BAB 2 skripsi, disusun berdasarkan seluruh rancangan sistem di BAB 3 dan pengujian di BAB 4.

---

## 2.1 Metodologi Penelitian

1. **Research and Development (R&D)**
   - Definisi dan karakteristik metode R&D
   - Tahapan umum R&D (Borg & Gall atau Sugiyono)
   - Alasan pemilihan R&D untuk penelitian ini

2. **Model Pengembangan Prototyping**
   - Definisi prototyping
   - Tahapan prototyping: pengumpulan kebutuhan → desain cepat → evaluasi → perbaikan → produk akhir
   - Kelebihan dan kekurangan prototyping
   - Alasan pemilihan prototyping dibanding model lain (Waterfall, Agile, dll.)

3. **Black Box Testing**
   - Definisi dan konsep black box testing
   - Perbedaan dengan white box testing
   - Teknik yang digunakan (equivalence partitioning, boundary value analysis)

---

## 2.2 Tunanetra & Orientasi Mobilitas

4. **Tunanetra (Visual Impairment)**
   - Definisi dan klasifikasi tunanetra (buta total vs low vision)
   - Data statistik tunanetra di Indonesia (BPS / WHO)
   - Tantangan mobilitas tunanetra di lingkungan perkotaan

5. **Orientasi dan Mobilitas (O&M)**
   - Definisi orientasi dan mobilitas
   - Teknik konvensional: tongkat putih, guide dog, pendamping
   - Sistem arah jam (*clock direction*) dalam O&M — Jam 10, 11, 12, 1, 2
   - 📐 *Jelaskan konsep pembagian zona sudut pandang menjadi arah jam → mendukung **Formula B** (Pemetaan Arah Jam) di BAB 3*

6. **Assistive Technology untuk Tunanetra**
   - Perkembangan assistive technology
   - Penelitian terdahulu / produk sejenis (Be My Eyes, Seeing AI, WeWalk Smart Cane)
   - Positioning penelitian ini terhadap penelitian terdahulu

---

## 2.3 Internet of Things (IoT)

7. **Konsep Internet of Things (IoT)**
   - Definisi IoT
   - Arsitektur IoT (perception layer, network layer, application layer)
   - Penerapan IoT dalam assistive technology

8. **Wearable Device**
   - Definisi wearable device
   - Jenis-jenis wearable (smartwatch, smartglasses, dll.)
   - Penerapan wearable untuk tunanetra

---

## 2.4 Hardware

9. **ESP32-S3 WROOM N16R8**
   - Arsitektur dan spesifikasi (dual-core, clock, RAM, flash)
   - Fitur konektivitas: WiFi + Bluetooth 5 (BLE)
   - GPIO dan antarmuka komunikasi (I2C, SPI, UART)
   - Alasan pemilihan ESP32-S3 dibanding mikrokontroler lain

10. **Kamera OV2640**
    - Spesifikasi (resolusi max, FoV, output format JPEG)
    - Mode VGA (640×480) yang digunakan
    - Kelebihan: ukuran kecil, konsumsi daya rendah, didukung langsung oleh ESP32 camera driver
    - 📐 *Jelaskan konsep FoV (Field of View) horizontal kamera ~66° → mendukung **Formula B** (dead zone 80px) dan **Formula C** (grid binning) di BAB 3*

11. **Sensor VL53L5CX (Time-of-Flight)**
    - Prinsip kerja sensor Time-of-Flight (ToF)
    - Spesifikasi: grid 8×8 (64 zona), jangkauan 4 meter, FoV 45°
    - Komunikasi I2C
    - Perbedaan dengan sensor ultrasonik (HC-SR04) — alasan pemilihan ToF
    - 📐 *Jelaskan konsep matriks 8×8 dan FoV 45° → mendukung **Formula C** (indeks kolom), **Formula D** (jarak dari baris 3-5), dan **Formula H** (rasio baris bawah vs tengah) di BAB 3*

12. **Buzzer (Piezoelectric)**
    - Prinsip kerja buzzer
    - Peran buzzer sebagai mekanisme fail-safe (output non-visual, non-TTS)

13. **Baterai Li-Po & TP4056**
    - Manajemen daya untuk wearable device
    - Charging module TP4056 dan perlindungan short circuit

---

## 2.5 Protokol Komunikasi

14. **Bluetooth Low Energy (BLE)**
    - Prinsip kerja BLE
    - Perbedaan BLE dengan Bluetooth Classic
    - Peran BLE dalam provisioning WiFi (setup awal)

15. **WiFi (IEEE 802.11)**
    - Prinsip kerja WiFi
    - Mode station (ESP32 sebagai client ke hotspot HP)

16. **WebSocket**
    - Definisi dan prinsip kerja WebSocket
    - Perbedaan dengan HTTP (full-duplex vs request-response)
    - Peran WebSocket: streaming video + data sensor secara real-time dari ESP32 ke smartphone

17. **I2C (Inter-Integrated Circuit)**
    - Prinsip kerja protokol I2C (SDA, SCL)
    - Peran I2C: komunikasi ESP32 dengan sensor VL53L5CX

---

## 2.6 Computer Vision & AI

18. **Object Detection**
    - Definisi object detection
    - Perbedaan: classification vs detection vs segmentation
    - Konsep bounding box (x_min, y_min, x_max, y_max, confidence, class)
    - 📐 *Jelaskan konsep centroid (titik tengah) bounding box → mendukung **Formula A** ($X_c$) di BAB 3*
    - 📐 *Jelaskan konsep area bounding box → mendukung **Formula G** ($\Delta A$, delta BBox) di BAB 3*

19. **YOLO (You Only Look Once)**
    - Arsitektur dan prinsip kerja YOLO (single-stage detector)
    - Evolusi YOLO (v1 → v11)
    - YOLOv11 Nano — varian paling ringan untuk edge/mobile
    - Perbandingan YOLO dengan SSD, Faster R-CNN

20. **TensorFlow Lite (TFLite)**
    - Definisi dan tujuan TFLite (AI di perangkat mobile)
    - Proses konversi model: PyTorch → ONNX → TFLite
    - GPU Delegate dan NNAPI untuk akselerasi inferensi di Android
    - Alasan pemilihan TFLite dibanding ONNX Runtime atau PyTorch Mobile

---

## 2.7 Android Development

21. **Android Platform**
    - Arsitektur sistem Android (kernel, libraries, framework, apps)
    - Bahasa pemrograman: Kotlin / Java

22. **Text-to-Speech (TTS)**
    - Prinsip kerja TTS di Android (TextToSpeech API)
    - Bahasa Indonesia pada TTS Android
    - Mekanisme callback (OnUtteranceCompleted) untuk anti-tumpang-tindih suara

23. **Accelerometer (Sensor Smartphone)**
    - Prinsip kerja sensor accelerometer
    - Android Sensor API (SensorManager, SensorEvent)
    - Peran accelerometer: mendeteksi apakah user bergerak atau diam
    - 📐 *Jelaskan konsep magnitude percepatan dan threshold gerakan → mendukung **Formula J** (deteksi sumber gerakan) dan **Formula E** (kecepatan pendekatan) di BAB 3*

---

## 2.8 Pemodelan Sistem (UML)

24. **Unified Modeling Language (UML)**
    - Definisi UML
    - Jenis-jenis diagram UML yang digunakan:

25. **Use Case Diagram**
    - Definisi, komponen (aktor, use case, relasi include/extend)

26. **Sequence Diagram**
    - Definisi, komponen (lifeline, message, alt/loop/par fragment)

27. **State Machine Diagram**
    - Definisi, komponen (state, transition, trigger, guard)

28. **Flowchart**
    - Definisi, simbol-simbol standar (process, decision, I/O, terminator)

---

## 2.9 Dasar Teori Matematis

Bagian ini menjelaskan **teori dasar** yang menjadi landasan formula di BAB 3. Formula spesifik yang dirancang penulis ada di BAB 3 (sub-bab 3.7).

29. **Fungsi Floor (Pembulatan ke Bawah)**
    - Definisi $\lfloor x \rfloor$
    - 📐 *Dasar teori untuk **Formula C** ($C_{index} = \lfloor (X_c - 80) / 60 \rfloor$) di BAB 3*

30. **Rata-rata Aritmetika (Mean)**
    - Definisi dan formula umum $\bar{x} = \frac{1}{n}\sum x_i$
    - 📐 *Dasar teori untuk **Formula D** (rata-rata 3 baris sensor) dan **Formula H** (rata-rata jarak per zona) di BAB 3*

31. **Standar Deviasi**
    - Definisi dan formula umum $\sigma = \sqrt{\frac{1}{n}\sum(x_i - \bar{x})^2}$
    - 📐 *Dasar teori untuk **Formula H.4** (analisis pola anomali medan) di BAB 3*

32. **Fungsi Minimum**
    - Definisi $\min(a, b)$
    - 📐 *Dasar teori untuk **Formula E** ($T = \min(1 + v \times 2, 4)$) dan **Formula I** ($D_{min}$) di BAB 3*

33. **Persentase Perubahan (Rate of Change)**
    - Definisi formula persentase: $\Delta = \frac{\text{baru} - \text{lama}}{\text{lama}} \times 100\text{\%}$
    - 📐 *Dasar teori untuk **Formula G** (delta bounding box) di BAB 3*

34. **Kecepatan (Jarak / Waktu)**
    - Definisi dasar kinematika: $v = \Delta d / \Delta t$
    - 📐 *Dasar teori untuk **Formula E.1** (kecepatan pendekatan) di BAB 3*

35. **Mean Absolute Error (MAE) & MAPE**
    - Definisi dan formula
    - 📐 *Dasar teori untuk **pengujian akurasi jarak** di BAB 4*

36. **Precision, Recall, F1-Score**
    - Definisi TP, FP, FN
    - Formula Precision, Recall, F1
    - 📐 *Dasar teori untuk **pengujian akurasi YOLO** di BAB 4*

---

## Checklist Penulisan BAB 2

> 📐 = topik yang perlu menyertakan **dasar teori formula** (sebagai pondasi untuk formula di BAB 3)

| No | Topik | Sub-bab | Formula BAB 3 | Status |
|---|---|---|---|---|
| 1 | Research and Development (R&D) | 2.1 | — | ☐ |
| 2 | Model Prototyping | 2.1 | — | ☐ |
| 3 | Black Box Testing | 2.1 | — | ☐ |
| 4 | Tunanetra | 2.2 | — | ☐ |
| 5 | Orientasi dan Mobilitas | 2.2 | 📐 Formula B | ☐ |
| 6 | Assistive Technology | 2.2 | — | ☐ |
| 7 | Internet of Things | 2.3 | — | ☐ |
| 8 | Wearable Device | 2.3 | — | ☐ |
| 9 | ESP32-S3 | 2.4 | — | ☐ |
| 10 | Kamera OV2640 | 2.4 | 📐 Formula B, C | ☐ |
| 11 | Sensor VL53L5CX (ToF) | 2.4 | 📐 Formula C, D, H | ☐ |
| 12 | Buzzer | 2.4 | — | ☐ |
| 13 | Baterai & TP4056 | 2.4 | — | ☐ |
| 14 | Bluetooth Low Energy | 2.5 | — | ☐ |
| 15 | WiFi | 2.5 | — | ☐ |
| 16 | WebSocket | 2.5 | — | ☐ |
| 17 | I2C | 2.5 | — | ☐ |
| 18 | Object Detection | 2.6 | 📐 Formula A, G | ☐ |
| 19 | YOLO / YOLOv11 | 2.6 | — | ☐ |
| 20 | TensorFlow Lite | 2.6 | — | ☐ |
| 21 | Android Platform | 2.7 | — | ☐ |
| 22 | Text-to-Speech | 2.7 | — | ☐ |
| 23 | Accelerometer | 2.7 | 📐 Formula J, E | ☐ |
| 24 | UML | 2.8 | — | ☐ |
| 25 | Use Case Diagram | 2.8 | — | ☐ |
| 26 | Sequence Diagram | 2.8 | — | ☐ |
| 27 | State Machine Diagram | 2.8 | — | ☐ |
| 28 | Flowchart | 2.8 | — | ☐ |
| 29 | Fungsi Floor | 2.9 | 📐 Formula C | ☐ |
| 30 | Rata-rata (Mean) | 2.9 | 📐 Formula D, H | ☐ |
| 31 | Standar Deviasi | 2.9 | 📐 Formula H.4 | ☐ |
| 32 | Fungsi Minimum | 2.9 | 📐 Formula E, I | ☐ |
| 33 | Persentase Perubahan | 2.9 | 📐 Formula G | ☐ |
| 34 | Kecepatan (Kinematika) | 2.9 | 📐 Formula E.1 | ☐ |
| 35 | MAE & MAPE | 2.9 | 📐 BAB 4 (akurasi jarak) | ☐ |
| 36 | Precision, Recall, F1 | 2.9 | 📐 BAB 4 (akurasi YOLO) | ☐ |
