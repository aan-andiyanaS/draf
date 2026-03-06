# State Machine Diagram - Sistem Bantu Navigasi Tunanetra

Dokumen ini menyajikan **State Machine Diagram (Statechart)** yang menggambarkan seluruh **state** (kondisi) yang mungkin dialami sistem dan **transisi** (perpindahan) antar kondisi tersebut beserta *trigger*-nya.

Dokumen ini sesuai dengan **sub-bab 3.5 Perancangan Logika dan Algoritma Sistem** pada BAB 3 skripsi, melengkapi flowchart di [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md).

---

## State Machine Gabungan: Keseluruhan Sistem

Diagram berikut menampilkan **seluruh state dan transisi** sistem dalam satu gambar:

```mermaid
stateDiagram-v2
    [*] --> Idle: Power ON

    Idle --> Provisioning: BLE Advertising

    Provisioning --> ModeSelection: WiFi Tersimpan di NVS
    Provisioning --> Provisioning: Koneksi Gagal

    state ModeSelection {
        [*] --> CekWiFi
        CekWiFi --> CekCahaya: WiFi Terhubung
        CekWiFi --> OfflineMode: WiFi Putus > 5 Detik
        CekCahaya --> SmartMode: Cahaya Terang
        CekCahaya --> GelapsMode: Cahaya Gelap
    }

    state SmartMode {
        [*] --> CekGerak
        CekGerak --> Paused: Diam > 10 Detik
        Paused --> CekGerak: User Bergerak Lagi
        CekGerak --> Processing: User Bergerak

        state Processing {
            [*] --> DataAcquisition
            DataAcquisition --> ParallelProcess
            state ParallelProcess {
                YOLOInference --> Mapping
                ToFProcessing --> Mapping
            }
            Mapping --> ModeCheck
        }

        ModeCheck --> Otonom: Mode Otonom
        ModeCheck --> TanyaJawab: Mode Tanya Jawab

        state Otonom {
            [*] --> ParallelDetection
            state ParallelDetection {
                state ObjekDeteksi {
                    CekJangkauan --> JalurA: Jarak ≤ 4m
                    CekJangkauan --> JalurB: Jarak > 4m
                }
                MedanDeteksi --> MedanAman: Normal
                MedanDeteksi --> AnalisisPola: Anomali
            }
        }

        TanyaJawab --> TTSOutput: Lapor Semua Objek
        Otonom --> TTSOutput: Ada Bahaya
        Otonom --> SimpanData: Aman (Diam)
        TTSOutput --> SimpanData: Audio Selesai
    }

    state OfflineMode {
        [*] --> SensorLoop
        SensorLoop --> BuzzerOn: Jarak < 1m
        SensorLoop --> BuzzerOff: Jarak ≥ 1m
        BuzzerOn --> TryReconnect
        BuzzerOff --> TryReconnect
        TryReconnect --> SensorLoop: Gagal
    }

    state GelapsMode {
        [*] --> StopStream
        StopStream --> SensorLoopGelap
        SensorLoopGelap --> BuzzerOnG: Jarak < 1m
        SensorLoopGelap --> BuzzerOffG: Jarak ≥ 1m
    }

    SmartMode --> ModeSelection: Siklus Selesai
    OfflineMode --> ModeSelection: Reconnect Berhasil
    GelapsMode --> ModeSelection: Cahaya Membaik

    SmartMode --> [*]: Power OFF
    OfflineMode --> [*]: Power OFF
    GelapsMode --> [*]: Power OFF
```

Diagram di atas terlalu besar untuk kertas A4. Berikut dipecah menjadi **tiga bagian**:

| Diagram | Fokus | State yang di-cover |
|---|---|---|
| **SM-1** | Siklus Hidup Sistem | Idle → Provisioning → Mode Selection |
| **SM-2** | Smart Mode (Detail) | Processing, Otonom, Tanya Jawab, Paused |
| **SM-3** | Mode Darurat | Offline Mode, Gelap Mode |

---

## SM-1. Siklus Hidup Sistem

Menampilkan state-state utama dari saat perangkat dinyalakan hingga masuk ke mode operasi.

```mermaid
stateDiagram-v2
    [*] --> Idle: Power ON

    state Idle {
        [*] --> HardwareBoot
        HardwareBoot --> BLEAdvertising: Boot Selesai
    }

    Idle --> Provisioning: App Terdeteksi → Koneksi BLE
    Idle --> ModeSelection: NVS Ada Kredensial → Auto Connect

    state Provisioning {
        [*] --> BLEConnected
        BLEConnected --> ScanWiFi: Request dari App
        ScanWiFi --> KirimKredensial: Pendamping Pilih WiFi
        KirimKredensial --> KoneksiWiFi
        KoneksiWiFi --> SimpanNVS: Berhasil
        KoneksiWiFi --> ScanWiFi: Gagal → Coba Lagi
    }

    Provisioning --> ModeSelection: Kredensial Tersimpan

    state ModeSelection {
        [*] --> CekWiFi
        CekWiFi --> CekCahaya: WiFi Terhubung
        CekWiFi --> OfflineMode: WiFi Putus > 5 Detik
        CekCahaya --> SmartMode: Cahaya Terang
        CekCahaya --> GelapMode: Cahaya Gelap
    }

    ModeSelection --> SmartMode
    ModeSelection --> OfflineMode
    ModeSelection --> GelapMode

    SmartMode --> [*]: Power OFF
    OfflineMode --> [*]: Power OFF
    GelapMode --> [*]: Power OFF
```

**Penjelasan State & Transisi:**

1. **Idle** — State awal setelah power ON. ESP32 melakukan boot hardware dan mengaktifkan BLE.
   - **Transisi ke Provisioning**: Jika belum pernah terhubung WiFi (NVS kosong), masuk ke mode provisioning untuk setup awal.
   - **Transisi ke Mode Selection**: Jika NVS sudah ada kredensial WiFi, langsung auto-connect dan masuk ke pemilihan mode.
2. **Provisioning** — State setup koneksi WiFi via BLE. Dilakukan oleh **pendamping**.
   - **Internal loop**: Jika koneksi WiFi gagal, kembali ke scan WiFi (tidak keluar dari state Provisioning).
   - **Transisi ke Mode Selection**: Setelah kredensial tersimpan di NVS, pendamping tidak lagi dibutuhkan.
3. **Mode Selection** — State penentuan mode operasi. Dilakukan otomatis setiap siklus.
   - **Cek WiFi**: Prioritas pertama. Jika putus > 5 detik → Offline Mode.
   - **Cek Cahaya**: Prioritas kedua (hanya jika WiFi OK). Gelap → Gelap Mode. Terang → Smart Mode.

> **Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — sub-bab 3.5.1 dan 3.5.2.

---

## SM-2. Smart Mode (Detail)

Menampilkan sub-state dalam Smart Mode — dari cek gerakan hingga output peringatan.

```mermaid
stateDiagram-v2
    [*] --> CekGerak

    state CekGerak {
        [*] --> BacaAccelerometer
        BacaAccelerometer --> UserBergerak: Gerakan Terdeteksi
        BacaAccelerometer --> Paused: Diam > 10 Detik
    }

    state Paused {
        [*] --> YOLOPaused
        YOLOPaused --> SensorAktif: VL53L5CX + Buzzer Tetap Jalan
    }
    Paused --> CekGerak: User Bergerak Lagi

    UserBergerak --> Processing

    state Processing {
        [*] --> KirimData
        KirimData --> ParallelProses

        state ParallelProses {
            state fork <<fork>>
            state join <<join>>
            fork --> YOLOInference
            fork --> ToFProcessing
            YOLOInference --> join
            ToFProcessing --> join
        }

        join --> MappingArahJam
        MappingArahJam --> CekModeApp
    }

    state CekModeApp <<choice>>
    CekModeApp --> ModeOtonom: Mode Otonom
    CekModeApp --> ModeTanyaJawab: Mode Tanya Jawab

    state ModeOtonom {
        [*] --> DeteksiParalel

        state DeteksiParalel {
            state fork2 <<fork>>
            state join2 <<join>>
            fork2 --> DeteksiObjek
            fork2 --> DeteksiMedan

            state DeteksiObjek {
                [*] --> CekJangkauan
                CekJangkauan --> JalurA: ≤ 4m (ToF Adaptif)
                CekJangkauan --> JalurB: > 4m (YOLO BBox)

                state JalurA {
                    [*] --> HitungDelta
                    HitungDelta --> CekAccel
                    CekAccel --> ThresholdAdaptif: User Bergerak
                    CekAccel --> ThresholdObjek: Objek Mendekat
                    CekAccel --> CekStatis: Kedua Diam
                }

                state JalurB {
                    [*] --> HitungBBox
                    HitungBBox --> BBoxMembesar: Area +20%
                    HitungBBox --> BBoxStabil: Stabil/Mengecil
                }
            }

            state DeteksiMedan {
                [*] --> CekBarisBawah
                CekBarisBawah --> MedanAman: Rasio Normal
                CekBarisBawah --> AnalisisPola: Anomali (Rasio > 0.8)
                AnalisisPola --> Tangga: Gradual + YOLO Konfirmasi
                AnalisisPola --> Lubang: Lokal / Tiba-tiba
                AnalisisPola --> Penurunan: Gradual + YOLO Tidak Deteksi
            }

            DeteksiObjek --> join2
            DeteksiMedan --> join2
        }
    }

    state ModeTanyaJawab {
        [*] --> LaporSemuaObjek
    }

    ModeOtonom --> OutputTTS: Ada Bahaya
    ModeOtonom --> SimpanData: Aman
    ModeTanyaJawab --> OutputTTS

    state OutputTTS {
        [*] --> KirimPesan
        KirimPesan --> PutarAudio
        PutarAudio --> AudioSelesai
    }

    OutputTTS --> SimpanData
    SimpanData --> [*]: Kembali ke Cek WiFi
```

**Penjelasan State & Transisi:**

1. **CekGerak** — Accelerometer smartphone dibaca setiap siklus.
   - **→ Paused**: User diam > 10 detik. YOLO di-pause, tapi VL53L5CX + buzzer tetap aktif.
   - **→ Processing**: User bergerak, pemrosesan data normal dimulai.
2. **Processing** — Proses paralel: YOLO inference (frame video) dan ToF processing (matriks jarak) berjalan bersamaan, lalu hasilnya di-mapping ke arah jam menggunakan titik tengah bounding box ($X_c$) dan formula kolom ToF $C_{index} = \lfloor (X_c - 80) / 60 \rfloor$.
3. **CekModeApp** — Pilihan user (via tombol multifungsi GPIO39 — tekan singkat 1× untuk ganti mode):
   - **Otonom**: Hanya peringatan saat bahaya.
   - **Tanya Jawab**: Lapor semua objek terdeteksi.
4. **ModeOtonom — DeteksiParalel** — Tiga jalur deteksi berjalan bersamaan:
   - **JalurA** ($D \le 4$ m): Threshold adaptif $T = \min(1 + v \times 2, \ 4)$ dengan 3 cabang accelerometer (user bergerak / objek mendekat / kedua statis).
   - **JalurB** ($D > 4$ m): Delta bounding box YOLO ($\Delta A > 20\text{\%}$) untuk kendaraan mendekat.
   - **DeteksiMedan**: Analisis rasio ToF $R = \bar{D}_{bawah} / \bar{D}_{tengah}$ untuk tangga/lubang/penurunan.
5. **OutputTTS** — Pesan dikirim ke TTS, audio diputar, callback diterima. Mencegah tumpang tindih suara.

> **Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — sub-bab 3.5.3 (Flowchart 3a–3e).

---

## SM-3. Mode Darurat (Offline & Gelap)

Menampilkan state-state saat kondisi non-ideal — ESP32 beroperasi mandiri.

```mermaid
stateDiagram-v2
    [*] --> OfflineMode: WiFi Putus > 5 Detik
    [*] --> GelapMode: Cahaya Gelap

    state OfflineMode {
        [*] --> MatikanKamera
        MatikanKamera --> SensorLoop

        state SensorLoop {
            [*] --> BacaSensor
            BacaSensor --> CekJarak

            state CekJarak <<choice>>
            CekJarak --> BuzzerOn: Jarak < 1m
            CekJarak --> BuzzerOff: Jarak ≥ 1m

            BuzzerOn --> CobaReconnect
            BuzzerOff --> CobaReconnect

            state CobaReconnect <<choice>>
            CobaReconnect --> BacaSensor: Gagal
        }
    }

    state GelapMode {
        [*] --> KameraLowFPS
        KameraLowFPS --> StopStreaming
        StopStreaming --> SensorLoopG

        state SensorLoopG {
            [*] --> BacaSensorG
            BacaSensorG --> CekJarakG

            state CekJarakG <<choice>>
            CekJarakG --> BuzzerOnG: Jarak < 1m
            CekJarakG --> BuzzerOffG: Jarak ≥ 1m

            BuzzerOnG --> BacaSensorG
            BuzzerOffG --> BacaSensorG
        }
    }

    OfflineMode --> ModeSelection: Reconnect Berhasil
    GelapMode --> ModeSelection: Cahaya Membaik (Brightness OK)

    state ModeSelection {
        [*] --> CekWiFi
        CekWiFi --> CekCahaya
    }
```

**Penjelasan State & Transisi:**

1. **Offline Mode** — Dipicu oleh WiFi putus > 5 detik.
   - **MatikanKamera**: Kamera dimatikan untuk hemat daya. Tidak ada video, tidak ada YOLO.
   - **SensorLoop**: ESP32 beroperasi mandiri — baca sensor VL53L5CX, buzzer jika $D_{min} < 1$ m, coba reconnect WiFi setiap siklus.
   - **Transisi keluar**: Hanya saat reconnect WiFi berhasil → kembali ke Mode Selection.
   - **Threshold tetap $T = 1$ m**: Karena tidak ada accelerometer/YOLO, kecepatan pendekatan ($v$) tidak dapat dihitung.
2. **Gelap Mode** — Dipicu oleh cahaya terlalu gelap ($B_{cam} < B_{threshold}$).
   - **KameraLowFPS**: Kamera tidak dimatikan total — masih digunakan untuk cek brightness ($B_{cam}$) secara periodik.
   - **StopStreaming**: Video streaming ke smartphone dihentikan.
   - **SensorLoopG**: Logika identik dengan Offline Mode (buzzer $D_{min} < 1$ m).
   - **Transisi keluar**: Saat $B_{cam} \ge B_{threshold}$ (cahaya membaik) → kembali ke Mode Selection, potensial masuk Smart Mode.

> **Referensi:** [alur-logika.md](file:///d:/Project/Skripsi/docs/alur-logika.md) — sub-bab 3.5.4.
