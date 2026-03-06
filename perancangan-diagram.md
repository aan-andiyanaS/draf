# Perancangan Diagram Sistem — Sistem Bantu Navigasi Tunanetra

Dokumen ini menyajikan **seluruh diagram perancangan sistem** secara terpadu dalam satu alur naratif, mulai dari Use Case Diagram, Sequence Diagram, State Machine Diagram, hingga potongan Flowchart yang relevan. Setiap bagian menjelaskan aspek yang **berbeda** tanpa pengulangan — Use Case menjawab *siapa melakukan apa*, Sequence menjawab *urutan komunikasi antar komponen*, State Machine menjawab *kapan status sistem berubah*, dan Flowchart menjawab *bagaimana logika keputusannya*.

---

## 3.6 Use Case Diagram

### 3.6.1 Use Case Provisioning — Peran Pendamping

Diagram berikut menampilkan interaksi **Pendamping** (sighted companion) saat melakukan setup awal perangkat. Proses provisioning **hanya dilakukan satu kali** — setelah berhasil, perangkat akan auto-connect di setiap penggunaan selanjutnya.

```mermaid
graph LR
    TN(("👤 Tunanetra"))
    PD(("👁️ Pendamping"))

    subgraph PROV ["🔧 Provisioning (Satu Kali)"]
        direction TB
        UC1["UC-1<br/>Nyalakan Perangkat<br/>(Tombol GPIO39)"]
        UC2["UC-2<br/>Setup Koneksi BLE"]
        UC3["UC-3<br/>Provisioning WiFi"]
        UC3a["UC-3a<br/>Scan Jaringan WiFi"]
        UC3b["UC-3b<br/>Pilih & Kirim Kredensial"]
        UC3c["UC-3c<br/>Simpan ke NVS"]
    end

    TN --- UC1
    PD --- UC2
    PD --- UC3

    UC3 -.->|"≪include≫"| UC3a
    UC3 -.->|"≪include≫"| UC3b
    UC3 -.->|"≪include≫"| UC3c

    classDef actor fill:#e3f2fd,stroke:#1565c0,stroke-width:2px;
    classDef uc fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    class TN,PD actor;
    class UC1,UC2,UC3,UC3a,UC3b,UC3c uc;
```

**Penjelasan:**

- **UC-1 — Nyalakan Perangkat**: Tunanetra menekan tombol multifungsi (GPIO39) pada gagang kacamata. ESP32-S3 boot, LED indikator (GPIO48) berkedip cepat menandakan mode BLE Provisioning aktif. Ini **satu-satunya aksi tunanetra** dalam seluruh proses provisioning.
- **UC-2 — Setup Koneksi BLE**: Pendamping menyalakan Bluetooth & WiFi Hotspot di smartphone, membuka aplikasi Android, melakukan scan BLE, dan memilih perangkat IoT. Langkah ini memerlukan **interaksi visual** dengan layar sehingga harus dilakukan pendamping.
- **UC-3 — Provisioning WiFi**: Pendamping memilih jaringan WiFi dan mengirim kredensial ke ESP32 via BLE. Terdiri dari tiga sub-proses (≪include≫):
  - **UC-3a — Scan Jaringan WiFi**: ESP32 memindai SSID sekitar dan mengirim daftar ke aplikasi via BLE.
  - **UC-3b — Pilih & Kirim Kredensial**: Pendamping memilih SSID, masukkan password, lalu kirim ke ESP32.
  - **UC-3c — Simpan ke NVS**: Setelah WiFi terhubung, kredensial disimpan ke NVS (Non-Volatile Storage) agar auto-connect di sesi berikutnya. **Setelah langkah ini, pendamping tidak lagi dibutuhkan.**

---

### 3.6.2 Use Case Operasi Harian — Interaksi Tunanetra

Diagram berikut menampilkan interaksi **Tunanetra** saat penggunaan sehari-hari. Semua interaksi melalui **satu tombol multifungsi** (GPIO39), output **suara TTS**, **LED indikator** (GPIO48), dan **buzzer** — tidak perlu melihat layar.

```mermaid
graph LR
    TN(("👤 Tunanetra"))

    subgraph NAV ["🚶 Operasi Harian"]
        direction TB
        UC4["UC-4<br/>Mulai Navigasi<br/>(Auto-Connect WiFi)"]
        UC5["UC-5<br/>Mode Otonom<br/>(Peringatan saat bahaya)"]
        UC6["UC-6<br/>Mode Tanya Jawab<br/>(Lapor semua objek)"]
        UC7["UC-7<br/>Ganti Mode<br/>(Tekan 1× GPIO39)"]
        UC8["UC-8<br/>Matikan Perangkat<br/>(Tekan panjang 3-5s)"]
        UC9["UC-9<br/>Reset WiFi<br/>(Tekan >8s GPIO39)"]
    end

    subgraph DET ["🎯 Deteksi Otomatis"]
        direction TB
        UC10["UC-10<br/>Deteksi Objek Dekat<br/>Jalur A: D ≤ 4m"]
        UC11["UC-11<br/>Deteksi Kendaraan Jauh<br/>Jalur B: D > 4m"]
        UC12["UC-12<br/>Deteksi Medan<br/>Tangga / Lubang"]
        UC13["UC-13<br/>Peringatan TTS"]
    end

    subgraph MODE ["⚙️ Mode Sistem (Otomatis)"]
        direction TB
        UC14["UC-14<br/>Mode Smart / AI"]
        UC15["UC-15<br/>Mode Offline"]
        UC16["UC-16<br/>Mode Gelap"]
        UC17["UC-17<br/>Buzzer Darurat"]
    end

    TN --- UC4
    TN --- UC5
    TN --- UC6
    TN --- UC7
    TN --- UC8
    TN --- UC9

    UC5 -.->|"≪include≫"| UC10
    UC5 -.->|"≪include≫"| UC11
    UC5 -.->|"≪include≫"| UC12
    UC5 -.->|"≪include≫"| UC13
    UC6 -.->|"≪include≫"| UC13

    UC4 -.->|"≪extend≫"| UC14
    UC4 -.->|"≪extend≫"| UC15
    UC4 -.->|"≪extend≫"| UC16
    UC15 -.->|"≪include≫"| UC17
    UC16 -.->|"≪include≫"| UC17

    classDef actor fill:#e3f2fd,stroke:#1565c0,stroke-width:2px;
    classDef operasi fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef deteksi fill:#fce4ec,stroke:#c62828,stroke-width:2px;
    classDef mode fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
    classDef alert fill:#ffebee,stroke:#b71c1c,stroke-width:3px;

    class TN actor;
    class UC4,UC5,UC6,UC7,UC8,UC9 operasi;
    class UC10,UC11,UC12,UC13 deteksi;
    class UC14,UC15,UC16 mode;
    class UC17 alert;
```

**Penjelasan Interaksi Tombol Multifungsi (GPIO39):**

| Pola Tekanan | Durasi | Use Case | Feedback |
|---|---|---|---|
| Tekan singkat 1× | < 1 detik | UC-7 Ganti Mode (Otonom ↔ Tanya Jawab) | TTS konfirmasi + LED berubah pola |
| Double press 2× | 2× dalam 0.5 detik | UC-6 Trigger Tanya (lapor semua objek) | TTS menyebutkan semua objek |
| Tekan panjang | 3–5 detik | UC-8 Matikan Perangkat (deep sleep) | Buzzer 2× beep + LED mati |
| Tekan sangat panjang | > 8 detik | UC-9 Reset WiFi (hapus NVS) | Buzzer 3× beep + reboot |

**Penjelasan Use Case Operasi:**

- **UC-4 — Mulai Navigasi**: Tunanetra menekan tombol → ESP32 auto-connect WiFi via NVS → LED menyala tetap → sistem otomatis memilih mode berdasarkan kondisi:
  - WiFi OK + terang → **UC-14 Mode Smart** (≪extend≫)
  - WiFi putus > 5s → **UC-15 Mode Offline** (≪extend≫)
  - WiFi OK + gelap → **UC-16 Mode Gelap** (≪extend≫)
- **UC-5 — Mode Otonom** (default): Sistem **diam saat aman**, hanya memberi peringatan saat ada bahaya melalui 3 jalur deteksi paralel (UC-10, UC-11, UC-12).
- **UC-6 — Mode Tanya Jawab**: Tunanetra menekan 2× cepat → TTS menyebutkan **semua objek** beserta arah jam dan jarak.
- **UC-17 — Buzzer Darurat**: Di Mode Offline/Gelap, buzzer ESP32 berbunyi langsung saat $D_{min} < 1$ m — mekanisme fail-safe tanpa TTS.

---

## 3.7 Sequence Diagram

### 3.7.1 SD-1: Alur Komunikasi Provisioning

Sequence diagram berikut menunjukkan **urutan komunikasi** antar aktor saat provisioning. Tunanetra hanya menekan tombol, semua langkah visual dilakukan pendamping.

```mermaid
sequenceDiagram
    participant TN as 👤 Tunanetra
    participant IoT as 🔌 ESP32-S3
    participant App as 📱 Aplikasi Android
    participant PD as 👁️ Pendamping

    TN->>IoT: Nyalakan Perangkat (Tombol GPIO39)
    Note over IoT: Boot → BLE Advertising<br/>LED berkedip cepat

    PD->>App: Buka App, Aktifkan BT & Hotspot
    App->>IoT: Scan BLE → Pilih Perangkat
    IoT-->>App: BLE Connected

    PD->>App: Pilih SSID WiFi
    App->>IoT: Kirim Kredensial WiFi via BLE
    IoT->>IoT: Coba Connect ke WiFi

    alt WiFi Berhasil
        IoT->>IoT: Simpan ke NVS
        IoT-->>App: Status: Connected ✅
        Note over IoT: BLE Diputus<br/>LED menyala tetap
        Note over PD: Pendamping tidak lagi<br/>dibutuhkan setelah ini
    else WiFi Gagal
        IoT-->>App: Status: Gagal ❌
        PD->>App: Masukkan ulang password
    end
```

**Poin penting yang tidak dibahas di Use Case:**
1. **BLE diputus setelah WiFi berhasil** — kanal BLE hanya dipakai saat provisioning, setelah itu ESP32 berkomunikasi penuh via WiFi (WebSocket).
2. **Kredensial disimpan permanen di NVS** — sehingga pada boot berikutnya, ESP32 langsung auto-connect tanpa BLE.
3. **Jika WiFi gagal**, pendamping bisa langsung memasukkan ulang password tanpa restart perangkat.

---

### 3.7.2 SD-2: Alur Data Real-Time (Mode Smart)

Setelah provisioning berhasil dan sistem masuk Mode Smart, berikut adalah **alur data per-frame video** dari kamera hingga output TTS.

```mermaid
sequenceDiagram
    participant IoT as 🔌 ESP32-S3
    participant App as 📱 Aplikasi Android
    participant AI as 🧠 YOLO Engine

    loop Setiap Frame
        IoT->>App: Kirim Frame JPEG via WebSocket
        IoT->>App: Kirim Data ToF 8×8 via WebSocket

        App->>AI: Frame → YOLO Inference
        AI-->>App: Bounding Box (class, x, y, w, h, conf)

        App->>App: Hitung Xc (titik tengah horizontal)
        App->>App: Mapping Arah Jam (Formula B)
        App->>App: Mapping Kolom ToF (Formula C)
        App->>App: Ambil Jarak dari ToF (Formula D)

        Note over App: Hasil: "Objek [X] di Jam [Y], [Z] meter"
    end
```

**Poin penting yang berbeda dari Use Case:**
1. **Dua jenis data dikirim bersamaan** dari ESP32: frame video (JPEG) dan matriks sensor ToF (8×8) — keduanya via WebSocket.
2. **Proses mapping sepenuhnya di smartphone**: Titik tengah bounding box ($X_c$) dipetakan ke arah jam (Formula B), lalu indeks kolom ToF (Formula C) digunakan untuk mengambil jarak presisi (Formula D).
3. **YOLO inference berjalan di HP** menggunakan TFLite GPU Delegate — bukan di ESP32. ESP32 hanya mengirim data mentah.

---

### 3.7.3 SD-3a: Alur Peringatan Jalur A (Objek Dekat, D ≤ 4m)

Saat objek terdeteksi dalam jangkauan sensor ToF, sistem menghitung threshold adaptif untuk menentukan apakah peringatan perlu diberikan.

```mermaid
sequenceDiagram
    participant App as 📱 Aplikasi
    participant TTS as 🔊 TTS Engine
    participant TN as 👤 Tunanetra

    App->>App: Hitung kecepatan pendekatan (v)
    App->>App: Threshold T = min(1 + v×2, 4)

    alt D < T (Bahaya)
        alt User bergerak ATAU Objek mendekat
            App->>TTS: "Awas, [Objek] di Jam [X], [Y] meter"
            TTS->>TN: 🔊 Output suara
            TTS-->>App: Callback: selesai bicara
        else Keduanya diam DAN D < 1m DAN belum pernah diperingatkan
            App->>TTS: "Objek Dekat di Jam [X], [Y] meter"
            TTS->>TN: 🔊 Output suara (sekali saja)
            Note over App: Set flag = true<br/>Tidak diulang sampai user bergerak
        end
    else D ≥ T (Aman)
        Note over App: Tidak ada peringatan — sistem diam
    end
```

**Poin penting:**
1. **Threshold dinamis** — Semakin cepat user/objek bergerak, semakin jauh jarak peringatan. User berjalan 1.5 m/s → $T = 4$ m (peringatan dari 4 meter). User diam → $T = 1$ m (tidak mengganggu).
2. **Anti-spam saat statis** — Jika user dan objek sama-sama diam pada jarak < 1 meter, peringatan diberikan **satu kali saja** (flag). Flag reset saat user bergerak.
3. **Callback mechanism** — TTS mengirim callback setelah selesai bicara. Peringatan berikutnya menunggu callback ini — mencegah suara bertumpuk.

---

### 3.7.4 SD-3b: Alur Peringatan Jalur B (Kendaraan Jauh, D > 4m)

Untuk objek di luar jangkauan ToF, sistem menggunakan perubahan ukuran bounding box YOLO sebagai indikator pendekatan.

```mermaid
sequenceDiagram
    participant App as 📱 Aplikasi
    participant TTS as 🔊 TTS Engine
    participant TN as 👤 Tunanetra

    App->>App: Bandingkan BBox frame ini vs sebelumnya
    App->>App: ΔA = (A_baru - A_lama) / A_lama × 100%

    alt ΔA > 20% (Objek mendekat cepat)
        App->>TTS: "AWAS, Kendaraan Mendekat dari Jam [X]!"
        TTS->>TN: 🔊 Output suara
        Note over App: Tanpa info jarak<br/>(ToF tidak menjangkau)
    else ΔA ≤ 20% (Stabil/menjauh)
        Note over App: Tidak ada peringatan
    end

    Note over App: Saat objek masuk D ≤ 4m<br/>→ otomatis pindah ke Jalur A
```

**Poin penting:**
1. **Tidak ada jarak** — Peringatan hanya menyebutkan arah jam, bukan jarak, karena sensor ToF tidak menjangkau > 4 meter.
2. **Transisi otomatis** — Saat objek terus mendekat dan masuk jangkauan ToF ($D \le 4$ m), sistem otomatis beralih ke Jalur A yang memberikan jarak presisi.

---

### 3.7.5 SD-3c: Alur Deteksi Medan (Tangga/Lubang)

Sensor ToF menganalisis perbedaan jarak antar zona untuk mendeteksi perubahan elevasi lantai di depan user.

```mermaid
sequenceDiagram
    participant IoT as 🔌 ESP32-S3
    participant App as 📱 Aplikasi
    participant AI as 🧠 YOLO Engine
    participant TTS as 🔊 TTS Engine
    participant TN as 👤 Tunanetra

    IoT->>App: Data ToF 8×8

    App->>App: Hitung R = D̄_bawah / D̄_tengah

    alt R > 0.8 (Anomali)
        App->>App: Analisis pola σ (standar deviasi)

        alt σ kecil (pola gradual — mungkin tangga)
            App->>AI: Konfirmasi visual: apakah tangga?
            alt YOLO deteksi tangga
                App->>TTS: "Tangga di depan" (informatif)
            else YOLO tidak deteksi tangga
                App->>TTS: "AWAS, penurunan di depan!" (peringatan)
            end
        else σ besar (pola tiba-tiba — lubang/parit)
            App->>TTS: "AWAS, lubang di depan!" (langsung peringatan)
        end

        TTS->>TN: 🔊 Output suara
    else R < 0.7 (Normal — lantai datar)
        Note over App: Tidak ada peringatan
    end
```

**Poin penting:**
1. **Dua langkah analisis** — Rasio $R$ mendeteksi anomali, lalu standar deviasi $\sigma$ membedakan pola gradual (tangga) vs tiba-tiba (lubang).
2. **Konfirmasi visual** — Untuk pola gradual, YOLO diminta konfirmasi. Jika YOLO bilang tangga → informasi netral ("Tangga di depan"). Jika bukan tangga → peringatan ("AWAS, penurunan!").
3. **Lubang langsung peringatan** — Pola tiba-tiba ($\sigma$ besar) dianggap berbahaya tanpa perlu konfirmasi YOLO.

---

### 3.7.6 SD-4: Alur Mode Darurat (Offline & Gelap)

Saat WiFi putus atau cahaya kurang, sistem beralih ke mekanisme fail-safe menggunakan buzzer fisik pada ESP32.

```mermaid
sequenceDiagram
    participant TN as 👤 Tunanetra
    participant IoT as 🔌 ESP32-S3

    Note over IoT: WiFi putus > 5 detik<br/>ATAU cahaya < threshold

    loop Sensor Loop Mandiri
        IoT->>IoT: Baca VL53L5CX (64 zona)
        IoT->>IoT: Cari D_min dari semua zona

        alt D_min < 1 m
            IoT->>IoT: Bunyikan Buzzer (GPIO38)
            Note over TN: Mendengar buzzer<br/>dari perangkat wearable
        else D_min ≥ 1 m
            Note over IoT: Buzzer diam
        end

        IoT->>IoT: Coba reconnect WiFi
        alt WiFi berhasil
            Note over IoT: Kembali ke Mode Smart<br/>LED menyala tetap
        end
    end
```

**Poin penting:**
1. **ESP32 beroperasi mandiri** — Tidak ada smartphone, tidak ada YOLO, tidak ada TTS. Hanya sensor ToF + buzzer.
2. **Threshold tetap 1 meter** — Tidak adaptif, karena accelerometer di HP tidak terjangkau. Angka 1 meter dipilih sebagai jarak aman minimum universal.
3. **Reconnect otomatis** — Setiap siklus sensor, ESP32 mencoba reconnect WiFi. Jika berhasil → kembali ke Mode Smart.

---

## 3.8 State Machine Diagram

### 3.8.1 SM-1: Transisi Mode Keseluruhan Sistem

State machine menunjukkan **semua state** yang mungkin dialami sistem dan **trigger** yang menyebabkan perpindahan antar state.

```mermaid
stateDiagram-v2
    [*] --> Idle: Power ON (GPIO39)

    Idle --> Provisioning: NVS Kosong
    Idle --> AutoConnect: NVS Ada Kredensial

    Provisioning --> AutoConnect: WiFi Berhasil<br/>(Simpan ke NVS)

    AutoConnect --> CekMode: WiFi Terhubung

    state CekMode {
        [*] --> CekWiFi
        CekWiFi --> SmartMode: WiFi OK
        CekWiFi --> OfflineMode: WiFi Putus > 5s

        SmartMode --> CekCahaya
        CekCahaya --> Aktif: Terang
        CekCahaya --> GelapMode: Gelap
    }

    state Aktif {
        [*] --> Processing
        Processing --> ModeOtonom: Tombol 1× → Otonom
        Processing --> ModeTanyaJawab: Tombol 1× → Tanya Jawab
    }

    OfflineMode --> CekWiFi: WiFi Reconnect
    GelapMode --> CekCahaya: Cahaya Membaik

    Aktif --> [*]: Tombol GPIO39 panjang (3-5s)
    OfflineMode --> [*]: Power OFF
    GelapMode --> [*]: Power OFF
```

**Penjelasan transisi yang belum dibahas di diagram sebelumnya:**
1. **Idle → Provisioning vs AutoConnect**: Saat power ON, ESP32 memeriksa NVS. Jika kosong → BLE Provisioning. Jika sudah ada kredensial → langsung auto-connect WiFi.
2. **Transisi antar Mode bersifat otomatis**: User **tidak memilih** Mode Smart/Offline/Gelap — sistem menentukan sendiri berdasarkan status WiFi dan cahaya.
3. **Transisi mode aplikasi (Otonom ↔ Tanya Jawab)**: Ini satu-satunya transisi yang **dikendalikan user** lewat tombol GPIO39 (tekan 1× singkat).
4. **Feedback LED** menunjukkan state saat ini: solid (Smart), kedip lambat (Darurat), kedip cepat (Provisioning), 2× kedip + jeda (Tanya Jawab).

---

### 3.8.2 SM-2: State Detail Mode Smart (Pemrosesan Paralel)

Saat sistem dalam Mode Smart, tiga jalur deteksi berjalan **bersamaan secara paralel**.

```mermaid
stateDiagram
    state SmartMode {
        [*] --> CekModeApp

        state CekModeApp {
            [*] --> Otonom
            Otonom --> TanyaJawab: Tekan 1× GPIO39
            TanyaJawab --> Otonom: Tekan 1× GPIO39
        }

        state ModeOtonom {
            state DeteksiParalel {
                JalurA: Jalur A\nObjek Dekat (D ≤ 4m)\nThreshold Adaptif
                JalurB: Jalur B\nKendaraan Jauh (D > 4m)\nDelta BBox YOLO
                JalurC: Jalur C\nDeteksi Medan\nRasio ToF R
            }
        }

        Otonom --> DeteksiParalel
        TanyaJawab --> LaporSemua: Tekan 2× GPIO39
        LaporSemua --> TanyaJawab: TTS selesai
    }

    SmartMode --> Paused: User diam > 10 detik
    Paused --> SmartMode: User bergerak
    Note right of Paused: YOLO di-pause\nToF + Buzzer tetap aktif
```

**Poin penting:**
1. **User diam > 10 detik → YOLO di-pause** untuk hemat daya baterai dan CPU HP. Namun sensor ToF + buzzer **tetap aktif** di ESP32 untuk keamanan.
2. **Tiga jalur paralel** tidak saling mengganggu — Jalur A, B, dan C berjalan bersamaan di setiap frame.

---

### 3.8.3 SM-3: State Detail Mode Darurat

```mermaid
stateDiagram-v2
    state OfflineMode {
        [*] --> SensorLoop
        SensorLoop --> BuzzerAktif: D_min < 1m
        SensorLoop --> BuzzerDiam: D_min ≥ 1m
        BuzzerAktif --> CobaReconnect
        BuzzerDiam --> CobaReconnect
        CobaReconnect --> SensorLoop: WiFi masih putus
        CobaReconnect --> KembaliSmart: WiFi berhasil
    }

    state GelapMode {
        [*] --> SensorLoopGelap
        SensorLoopGelap --> BuzzerAktifG: D_min < 1m
        SensorLoopGelap --> BuzzerDiamG: D_min ≥ 1m
        BuzzerAktifG --> CekCahaya
        BuzzerDiamG --> CekCahaya
        CekCahaya --> SensorLoopGelap: Masih gelap
        CekCahaya --> KembaliSmartG: Cahaya membaik
    }
```

**Perbedaan antara Mode Offline dan Mode Gelap:**

| Aspek | Mode Offline | Mode Gelap |
|---|---|---|
| **Trigger masuk** | WiFi putus > 5 detik | Cahaya < threshold |
| **Kamera** | Dimatikan total | Low FPS (cek brightness berkala) |
| **Cara keluar** | WiFi reconnect | Cahaya membaik ($B_{cam} \ge B_{threshold}$) |
| **Logika buzzer** | Identik ($D_{min} < 1$ m) | Identik ($D_{min} < 1$ m) |
| **LED indikator** | Berkedip lambat | Berkedip lambat |

---

## 3.9 Ringkasan Relasi Antar Diagram

Tabel berikut menunjukkan bagaimana setiap fitur diwakilkan di masing-masing diagram, sehingga pembaca tahu **di mana mencari detail** tanpa harus membaca ulang penjelasan yang sama.

| Fitur Sistem | Use Case (3.6) | Sequence (3.7) | State Machine (3.8) | Flowchart (3.5) | Formula (3.5.1) |
|---|---|---|---|---|---|
| **Provisioning** | UC-1, UC-2, UC-3 | SD-1 (3.7.1) | SM-1: Idle→Provisioning | Flowchart 3.5.2 | — |
| **Penentuan Mode** | UC-4 (extend) | — | SM-1: CekMode | Flowchart 3.5.3 | — |
| **Mapping Objek** | — | SD-2 (3.7.2) | — | Flowchart 3a (3.5.4) | Formula A, B, C, D |
| **Deteksi Dekat (Jalur A)** | UC-10 | SD-3a (3.7.3) | SM-2: JalurA | Flowchart 3c (3.5.4) | Formula E, F, J |
| **Deteksi Kendaraan (Jalur B)** | UC-11 | SD-3b (3.7.4) | SM-2: JalurB | Flowchart 3d (3.5.4) | Formula G |
| **Deteksi Medan (Jalur C)** | UC-12 | SD-3c (3.7.5) | SM-2: JalurC | Flowchart 3e (3.5.4) | Formula H |
| **Mode Darurat** | UC-15, UC-16, UC-17 | SD-4 (3.7.6) | SM-3 | Flowchart 3.5.5 | Formula I |
| **Ganti Mode** | UC-7 | — | SM-2: CekModeApp | — | — |
| **Matikan Perangkat** | UC-8 | — | SM-1: →[*] | — | — |
| **Reset WiFi** | UC-9 | — | SM-1: →Provisioning | — | — |
| **Buzzer Darurat** | UC-17 | SD-4 | SM-3: BuzzerAktif | Flowchart 3.5.5 | Formula I |
