   # Alur Logika - Sistem Kerja Perangkat

   Dokumen ini menjelaskan seluruh alur kerja perangkat IoT bantu navigasi tunanetra, mulai dari koneksi pertama kali (provisioning) hingga logika pemrosesan di setiap mode operasi. Dokumen ini sesuai dengan **sub-bab 3.5 Perancangan Logika dan Algoritma Sistem** pada BAB 3 skripsi.

   Terdapat **empat algoritma utama** yang masing-masing direpresentasikan dalam flowchart:

   | Sub-bab | Algoritma | Jumlah Flowchart |
   |---|---|---|
   | 3.5.1 | Koneksi Awal (Provisioning) | 1 flowchart |
   | 3.5.2 | Penentuan Mode Operasi | 1 flowchart |
   | 3.5.3 | Pemrosesan Cerdas (Smart Mode) | 5 flowchart (3a–3e) |
   | 3.5.4 | Mode Darurat (Fail-Safe) | 1 flowchart |

   ---

   ## 3.5.1. Algoritma Koneksi Awal (Provisioning)

   Algoritma ini menjelaskan proses setup awal saat perangkat IoT dinyalakan untuk pertama kali. Perangkat menggunakan **BLE (Bluetooth Low Energy)** untuk melakukan provisioning kredensial WiFi dari smartphone ke perangkat IoT, sehingga kedua perangkat dapat berkomunikasi via WiFi (WebSocket) secara otomatis di sesi-sesi berikutnya.

   ```mermaid
   flowchart TD
      Start([Mulai: Nyalakan Perangkat IoT])
      ActivateBLE[IoT: BLE Aktif - Advertising]
      UserPrep[/User: Nyalakan BT, Hotspot & Buka App/]
      ScanPilih[App: Scan BLE → User Pilih Perangkat]
      ConnBLE[App ↔ IoT: Terkoneksi via BLE]
      ProvWiFi[IoT: Scan WiFi → App Tampilkan Daftar → User Pilih Hotspot]
      KirimKoneksi[App → IoT: Kirim Kredensial → IoT Koneksi WiFi]
      CheckConn{Terkoneksi?}
      SaveAktif[IoT: Simpan WiFi ke NVS → Sistem Aktif]
      End([Selesai → Lanjut ke 3.5.2])

      Start --> ActivateBLE --> UserPrep --> ScanPilih
      ScanPilih --> ConnBLE --> ProvWiFi --> KirimKoneksi
      KirimKoneksi --> CheckConn
      CheckConn -- Ya --> SaveAktif --> End
      CheckConn -- Tidak --> ProvWiFi

      %% Styling
      classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
      classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;

      class ScanPilih,ProvWiFi,SaveAktif research;
      class Start,ActivateBLE,ConnBLE,KirimKoneksi design;
      class UserPrep impl;
      class CheckConn decision;
   ```

   **Penjelasan langkah demi langkah:**

   1. **Nyalakan Perangkat IoT** — User menekan tombol power pada perangkat wearable (head unit). ESP32-S3 melakukan boot dan menginisialisasi semua modul hardware.
   2. **BLE Aktif (Advertising)** — Setelah boot, ESP32 langsung mengaktifkan BLE dalam mode *advertising*, artinya perangkat mengirimkan sinyal agar bisa ditemukan oleh smartphone di sekitarnya.
   3. **Aksi User di Smartphone** — User perlu menyalakan Bluetooth dan Hotspot WiFi pada smartphone, lalu membuka aplikasi Android yang sudah terinstall.
   4. **Scan BLE & Pilih Perangkat** — Aplikasi menampilkan daftar perangkat BLE yang terdeteksi di sekitar. User memilih perangkat IoT dari daftar tersebut.
   5. **Terkoneksi via BLE** — Smartphone dan perangkat IoT membentuk koneksi BLE dua arah. Koneksi ini digunakan hanya untuk proses provisioning (bukan untuk transfer data utama).
   6. **Provisioning WiFi** — Perangkat IoT melakukan scanning jaringan WiFi di sekitarnya, lalu mengirimkan hasilnya ke aplikasi via BLE. User memilih jaringan WiFi yang sesuai (biasanya Hotspot smartphone itu sendiri).
   7. **Kirim Kredensial & Koneksi** — Aplikasi mengirimkan SSID dan password WiFi yang dipilih ke perangkat IoT. IoT kemudian mencoba terhubung ke jaringan WiFi tersebut.
   8. **Cek Koneksi** — Jika koneksi WiFi berhasil, lanjut ke langkah berikutnya. Jika gagal, kembali ke proses provisioning WiFi untuk mencoba jaringan lain.
   9. **Simpan & Aktifkan** — Kredensial WiFi yang berhasil disimpan ke memori NVS (Non-Volatile Storage) agar perangkat dapat auto-connect di sesi berikutnya tanpa provisioning ulang. Sistem dinyatakan aktif dan siap masuk ke algoritma penentuan mode.

   ---

   ## Flowchart Gabungan: Keseluruhan Sistem Setelah Provisioning

   Flowchart berikut menampilkan **keseluruhan alur kerja sistem** setelah provisioning selesai, mencakup penentuan mode (3.5.2), pemrosesan cerdas/Smart Mode (3.5.3), dan mode darurat/fail-safe (3.5.4) dalam satu gambar besar.

   ```mermaid
   flowchart TD
      %% ════ 3.5.2: PENENTUAN MODE OPERASI ════
      START([Mulai: Sistem Aktif]) --> INIT[Inisialisasi Hardware:<br/>ESP32, Kamera, VL53L5CX]
      INIT --> CEK_WIFI{Cek Koneksi WiFi?}

      CEK_WIFI -- Putus > 5 Detik --> MODE_SAFETY[MODE SAFETY / OFFLINE]
      CEK_WIFI -- Terhubung --> CEK_CAHAYA{Cek Kecerahan?}

      CEK_CAHAYA -- Gelap --> MODE_LOW[MODE LOW-LIGHT]
      CEK_CAHAYA -- Terang --> MODE_SMART([Mode Smart / AI])

      %% ════ 3.5.4: MODE DARURAT — OFFLINE ════
      MODE_SAFETY --> CAM_OFF[Matikan Kamera]
      MODE_SAFETY --> BACA_SENSOR[Baca Sensor Jarak VL53L5CX]

      BACA_SENSOR --> LOGIKA_BUZZER{Jarak < 1 Meter?}
      LOGIKA_BUZZER -- Ya --> BUZZ_ON[Bunyikan Buzzer]
      LOGIKA_BUZZER -- Tidak --> BUZZ_OFF[Buzzer Diam]
      BUZZ_ON --> RETRY[Coba Reconnect WiFi]
      BUZZ_OFF --> RETRY
      RETRY --> CEK_ULANG{Reconnect Berhasil?}
      CEK_ULANG -- Ya --> CEK_CAHAYA
      CEK_ULANG -- Tidak --> BACA_SENSOR

      %% ════ 3.5.4: MODE DARURAT — GELAP ════
      MODE_LOW --> CAM_LOW[Kamera Low FPS]
      MODE_LOW --> STOP_STREAM[Stop Video Streaming]
      MODE_LOW --> BACA_SENSOR

      %% ════ 3.5.3 FLOWCHART 3a: PEMROSESAN DATA & MAPPING ════
      MODE_SMART --> CEK_GERAK{Accelerometer:<br/>User Bergerak?}

      CEK_GERAK -- Diam > 10 Detik --> PAUSE_YOLO[Pause YOLO & Streaming]
      PAUSE_YOLO --> SENSOR_ONLY[Sensor VL53L5CX Tetap Aktif + Buzzer]
      SENSOR_ONLY -- User Bergerak Lagi --> CEK_GERAK

      CEK_GERAK -- Ya --> KIRIM_DATA[Kirim Video + Data Jarak ke HP]

      KIRIM_DATA --> YOLO[Proses Citra AI YOLOv11]
      KIRIM_DATA --> ToF[Proses Matriks Jarak VL53L5CX]

      YOLO --> MAPPING{Logika Mapping<br/>Arah Jam 10-2}
      ToF --> MAPPING

      MAPPING -- X < 20% / X > 80% --> LUAR[Set: Arah Jam 10 atau 2<br/>Status: Jarak Unknown]
      MAPPING -- X = 20% - 80% --> DALAM[Set: Arah Jam 11, 12, atau 1]
      DALAM --> HITUNG_GRID[Hitung Grid Pixel/60<br/>Ambil Data Array Jarak]

      %% ════ 3.5.3 FLOWCHART 3b: PERCABANGAN MODE APLIKASI ════
      LUAR --> CEK_MODE{Cek Mode Aplikasi?}
      HITUNG_GRID --> CEK_MODE

      CEK_MODE -- Mode Tanya Jawab --> TTS_INFO[Suara Info:<br/>Objek + Arah + Jarak]

      CEK_MODE -- Mode Otonom --> PARALEL[Proses Paralel:<br/>Deteksi Objek + Deteksi Medan]

      PARALEL --> CEK_JANGKAUAN{Objek dalam<br/>Jangkauan ToF?<br/>≤ 4 Meter}
      PARALEL --> CEK_BAWAH[Baca ToF Baris 7-8<br/>Bandingkan dengan Baris 4-5]

      %% ════ 3.5.3 FLOWCHART 3c: JALUR A — DETEKSI OBJEK DEKAT ════
      CEK_JANGKAUAN -- Ya: ≤ 4m --> DELTA_TOF[Hitung Delta Jarak ToF:<br/>ΔJarak = Jarak Lama − Jarak Baru]
      DELTA_TOF --> CEK_ACCEL{Cek Accelerometer<br/>Smartphone}

      CEK_ACCEL -- User Bergerak --> SPEED_U[Kecepatan Pendekatan =<br/>ΔJarak / Waktu Antar Frame]
      CEK_ACCEL -- User Diam +<br/>ΔJarak Positif --> SPEED_O[Kecepatan Objek =<br/>ΔJarak / Waktu Antar Frame]
      CEK_ACCEL -- User Diam +<br/>ΔJarak = 0 --> CEK_STATIS{Jarak < 1m?}

      SPEED_U --> TH_U[Threshold Adaptif =<br/>1m + Kecepatan × 2 detik<br/>Max: 4m]
      SPEED_O --> TH_O[Threshold Adaptif =<br/>1m + Kecepatan × 2 detik<br/>Max: 4m]

      TH_U --> F_U{Jarak < Threshold?}
      TH_O --> F_O{Jarak < Threshold?}

      F_U -- Ya --> TTS_WARN[Suara: AWAS<br/>Objek + Jam X + Jarak]
      F_U -- Tidak --> SILENT[Diam / Tidak Ada Suara]

      F_O -- Ya --> TTS_OBJ[Suara: AWAS Objek<br/>Mendekat + Jam X + Jarak]
      F_O -- Tidak --> SILENT

      CEK_STATIS -- Ya + Belum Pernah<br/>Diperingatkan --> TTS_STATIS[Suara: Objek Dekat<br/>di Jam X, Jarak Y]
      CEK_STATIS -- Tidak / Sudah<br/>Diperingatkan --> SILENT

      %% ════ 3.5.3 FLOWCHART 3d: JALUR B — DETEKSI KENDARAAN JAUH ════
      CEK_JANGKAUAN -- Tidak: > 4m --> BBOX[Hitung Delta Bounding Box:<br/>ΔBBox = BBox Baru − BBox Lama]
      BBOX --> CEK_BBOX{BBox Membesar<br/>Signifikan?<br/>Area +20% Antar Frame}
      CEK_BBOX -- Ya: Objek Mendekat --> TTS_FAR[Suara: AWAS<br/>Kendaraan Mendekat<br/>dari Arah Jam X]
      CEK_BBOX -- Tidak: Stabil/Mengecil --> SILENT

      %% ════ 3.5.3 FLOWCHART 3e: JALUR C — DETEKSI MEDAN ════
      CEK_BAWAH --> CEK_ANOMALI{Anomali Medan?<br/>Rasio Bawah/Tengah<br/>> 0.8}
      CEK_ANOMALI -- Tidak: Normal --> MEDAN_AMAN[Medan Datar: Aman]
      CEK_ANOMALI -- Ya: Anomali --> ANALISIS[Analisis Pola<br/>Seluruh Matriks 8×8]
      ANALISIS --> CEK_POLA{Pola Jarak<br/>Antar Baris?}
      CEK_POLA -- Gradual Naik +<br/>Merata Semua Kolom --> CEK_YOLO_T{YOLO Deteksi<br/>Tangga?}
      CEK_POLA -- Tiba-tiba Besar<br/>di Zona Tertentu --> TTS_LUBANG[Suara: AWAS<br/>Lubang di Depan!]
      CEK_YOLO_T -- Ya --> TTS_TANGGA[Suara: Tangga<br/>di Depan]
      CEK_YOLO_T -- Tidak --> TTS_DROP[Suara: AWAS<br/>Penurunan di Depan!]

      %% ════ AKHIR SIKLUS ════
      TTS_INFO --> SIMPAN((Simpan Data<br/>Frame))
      TTS_WARN --> SIMPAN
      TTS_OBJ --> SIMPAN
      TTS_STATIS --> SIMPAN
      TTS_FAR --> SIMPAN
      TTS_TANGGA --> SIMPAN
      TTS_DROP --> SIMPAN
      TTS_LUBANG --> SIMPAN
      SILENT --> SIMPAN
      MEDAN_AMAN --> SIMPAN
      SIMPAN --> CEK_WIFI

      %% Styling
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
      classDef adaptive fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
      classDef yolo fill:#fff9c4,stroke:#f57f17,stroke-width:2px;
      classDef terrain fill:#e0f2f1,stroke:#00695c,stroke-width:2px;
      classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
      classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
      classDef safety fill:#ffebee,stroke:#b71c1c,stroke-width:2px;

      class TTS_INFO,TTS_WARN,TTS_OBJ,TTS_STATIS,TTS_FAR,TTS_TANGGA,TTS_DROP,TTS_LUBANG,SILENT,BUZZ_ON,BUZZ_OFF impl;
      class CEK_WIFI,CEK_CAHAYA,CEK_GERAK,MAPPING,CEK_MODE,CEK_JANGKAUAN,CEK_ACCEL,F_U,F_O,CEK_STATIS,CEK_BBOX,CEK_ANOMALI,CEK_POLA,CEK_YOLO_T,LOGIKA_BUZZER,CEK_ULANG decision;
      class DELTA_TOF,SPEED_U,SPEED_O,TH_U,TH_O adaptive;
      class BBOX yolo;
      class CEK_BAWAH,ANALISIS,MEDAN_AMAN terrain;
      class PARALEL,SIMPAN,LUAR,DALAM,HITUNG_GRID,PAUSE_YOLO,SENSOR_ONLY,INIT design;
      class BACA_SENSOR,CAM_OFF,CAM_LOW,STOP_STREAM research;
      class MODE_SAFETY,MODE_LOW,RETRY safety;
   ```

   Flowchart di atas menampilkan **keseluruhan sistem dalam satu gambar**. Karena terlalu besar untuk dicetak pada A4

, sistem dipecah menjadi **bagian-bagian yang lebih kecil** pada sub-bab berikut:

   | Sub-bab | Bagian | Area pada Gambar Besar |
   |---|---|---|
   | **3.5.2** | Penentuan Mode Operasi | Bagian paling atas (Sistem Aktif → 3 mode) |
   | **3.5.3** | Smart Mode (Flowchart 3a–3e) | Bagian tengah dan bawah |
   | **3.5.4** | Mode Darurat (Offline & Gelap) | Cabang kiri atas |

   ---

   ## 3.5.2. Algoritma Penentuan Mode Operasi

   Algoritma ini menentukan mode operasi perangkat setelah provisioning selesai. Pengecekan dilakukan secara berurutan: **koneksi WiFi** terlebih dahulu, kemudian **kondisi cahaya**. Hasil percabangan mengarahkan sistem ke salah satu dari tiga mode operasi.

   ```mermaid
   flowchart TD
      START([Mulai: Sistem Aktif]) --> INIT[Inisialisasi Hardware:  ESP32, Kamera, VL53L5CX]
      INIT --> CEK_WIFI{Cek Koneksi WiFi?}

      CEK_WIFI -- Putus > 5 Detik --> GOTO_OFFLINE([3.5.4:  Mode Offline])
      CEK_WIFI -- Terhubung --> CEK_CAHAYA{Cek Kecerahan?}

      CEK_CAHAYA -- Gelap --> GOTO_GELAP([3.5.4:  Mode Gelap])
      CEK_CAHAYA -- Terang --> GOTO_SMART([3.5.3:  Mode Smart / AI])

      %% Styling
      classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
      classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;

      class INIT research;
      class CEK_WIFI,CEK_CAHAYA decision;
   ```

   **Penjelasan langkah demi langkah:**

   1. **Sistem Aktif** — Setelah provisioning berhasil (atau auto-connect dari NVS), sistem memulai operasi normalnya.
   2. **Inisialisasi Hardware** — ESP32-S3 menginisialisasi semua sensor: kamera OV2640, sensor jarak VL53L5CX, dan mempersiapkan koneksi WebSocket.
   3. **Cek Koneksi WiFi** — Sistem memonitor status koneksi WiFi secara terus-menerus.
      - **Putus > 5 detik**: Jika WiFi terputus lebih dari 5 detik (bukan hanya jitter sesaat), sistem masuk ke **Mode Offline/Safety** (3.5.4) untuk menjaga keamanan user.
      - **Terhubung**: Jika WiFi aktif, lanjut ke pengecekan cahaya.
   4. **Cek Kecerahan** — Kamera mengambil frame dan sistem menghitung rata-rata brightness.
      - **Gelap**: Jika lingkungan terlalu gelap untuk YOLO memproses gambar secara akurat, sistem masuk ke **Mode Gelap** (3.5.4) yang mengandalkan sensor jarak saja.
      - **Terang**: Kondisi ideal — sistem masuk ke **Mode Smart/AI** (3.5.3) yang memanfaatkan seluruh kemampuan AI.

   ---

   ## 3.5.3. Algoritma Pemrosesan Cerdas (Smart Mode)

   Mode utama sistem yang memanfaatkan seluruh kemampuan perangkat: kamera + YOLO AI + sensor jarak + accelerometer. Mode ini hanya aktif saat WiFi terhubung DAN kondisi cahaya cukup terang.

   Berikut adalah **flowchart gabungan** yang menampilkan keseluruhan algoritma Smart Mode dalam satu gambar:

   ```mermaid
   flowchart TD
      %% ════ FLOWCHART 3a: PEMROSESAN DATA & MAPPING ════
      MODE_SMART([Mode Smart / AI]) --> CEK_GERAK{Accelerometer:<br/>User Bergerak?}

      CEK_GERAK -- Diam > 10 Detik --> PAUSE_YOLO[Pause YOLO & Streaming]
      PAUSE_YOLO --> SENSOR_ONLY[Sensor VL53L5CX Tetap Aktif + Buzzer]
      SENSOR_ONLY -- User Bergerak Lagi --> CEK_GERAK

      CEK_GERAK -- Ya --> KIRIM_DATA[Kirim Video + Data Jarak ke HP]

      KIRIM_DATA --> YOLO[Proses Citra AI YOLOv11]
      KIRIM_DATA --> ToF[Proses Matriks Jarak VL53L5CX]

      YOLO --> MAPPING{Logika Mapping<br/>Arah Jam 10-2}
      ToF --> MAPPING

      MAPPING -- X < 20% / X > 80% --> LUAR[Set: Arah Jam 10 atau 2<br/>Status: Jarak Unknown]
      MAPPING -- X = 20% - 80% --> DALAM[Set: Arah Jam 11, 12, atau 1]
      DALAM --> HITUNG_GRID[Hitung Grid Pixel/60<br/>Ambil Data Array Jarak]

      %% ════ FLOWCHART 3b: PERCABANGAN MODE APLIKASI ════
      LUAR --> CEK_MODE{Cek Mode Aplikasi?}
      HITUNG_GRID --> CEK_MODE

      CEK_MODE -- Mode Tanya Jawab --> TTS_INFO[Suara Info:<br/>Objek + Arah + Jarak]

      CEK_MODE -- Mode Otonom --> PARALEL[Proses Paralel:<br/>Deteksi Objek + Deteksi Medan]

      PARALEL --> CEK_JANGKAUAN{Objek dalam<br/>Jangkauan ToF?<br/>≤ 4 Meter}
      PARALEL --> CEK_BAWAH[Baca ToF Baris 7-8<br/>Bandingkan dengan Baris 4-5]

      %% ════ FLOWCHART 3c: JALUR A — DETEKSI OBJEK DEKAT ════
      CEK_JANGKAUAN -- Ya: ≤ 4m --> DELTA_TOF[Hitung Delta Jarak ToF:<br/>ΔJarak = Jarak Lama − Jarak Baru]
      DELTA_TOF --> CEK_ACCEL{Cek Accelerometer<br/>Smartphone}

      CEK_ACCEL -- User Bergerak --> SPEED_U[Kecepatan Pendekatan =<br/>ΔJarak / Waktu Antar Frame]
      CEK_ACCEL -- User Diam +<br/>ΔJarak Positif --> SPEED_O[Kecepatan Objek =<br/>ΔJarak / Waktu Antar Frame]
      CEK_ACCEL -- User Diam +<br/>ΔJarak = 0 --> CEK_STATIS{Jarak < 1m?}

      SPEED_U --> TH_U[Threshold Adaptif =<br/>1m + Kecepatan × 2 detik<br/>Max: 4m]
      SPEED_O --> TH_O[Threshold Adaptif =<br/>1m + Kecepatan × 2 detik<br/>Max: 4m]

      TH_U --> F_U{Jarak < Threshold?}
      TH_O --> F_O{Jarak < Threshold?}

      F_U -- Ya --> TTS_WARN[Suara: AWAS<br/>Objek + Jam X + Jarak]
      F_U -- Tidak --> SILENT[Diam / Tidak Ada Suara]

      F_O -- Ya --> TTS_OBJ[Suara: AWAS Objek<br/>Mendekat + Jam X + Jarak]
      F_O -- Tidak --> SILENT

      CEK_STATIS -- Ya + Belum Pernah<br/>Diperingatkan --> TTS_STATIS[Suara: Objek Dekat<br/>di Jam X, Jarak Y]
      CEK_STATIS -- Tidak / Sudah<br/>Diperingatkan --> SILENT

      %% ════ FLOWCHART 3d: JALUR B — DETEKSI KENDARAAN JAUH ════
      CEK_JANGKAUAN -- Tidak: > 4m --> BBOX[Hitung Delta Bounding Box:<br/>ΔBBox = BBox Baru − BBox Lama]
      BBOX --> CEK_BBOX{BBox Membesar<br/>Signifikan?<br/>Area +20% Antar Frame}
      CEK_BBOX -- Ya: Objek Mendekat --> TTS_FAR[Suara: AWAS<br/>Kendaraan Mendekat<br/>dari Arah Jam X]
      CEK_BBOX -- Tidak: Stabil/Mengecil --> SILENT

      %% ════ FLOWCHART 3e: JALUR C — DETEKSI MEDAN ════
      CEK_BAWAH --> CEK_ANOMALI{Anomali Medan?<br/>Rasio Bawah/Tengah<br/>> 0.8}
      CEK_ANOMALI -- Tidak: Normal --> MEDAN_AMAN[Medan Datar: Aman]
      CEK_ANOMALI -- Ya: Anomali --> ANALISIS[Analisis Pola<br/>Seluruh Matriks 8×8]
      ANALISIS --> CEK_POLA{Pola Jarak<br/>Antar Baris?}
      CEK_POLA -- Gradual Naik +<br/>Merata Semua Kolom --> CEK_YOLO_T{YOLO Deteksi<br/>Tangga?}
      CEK_POLA -- Tiba-tiba Besar<br/>di Zona Tertentu --> TTS_LUBANG[Suara: AWAS<br/>Lubang di Depan!]
      CEK_YOLO_T -- Ya --> TTS_TANGGA[Suara: Tangga<br/>di Depan]
      CEK_YOLO_T -- Tidak --> TTS_DROP[Suara: AWAS<br/>Penurunan di Depan!]

      %% ════ AKHIR SIKLUS ════
      TTS_INFO --> SIMPAN((Simpan Data<br/>Frame))
      TTS_WARN --> SIMPAN
      TTS_OBJ --> SIMPAN
      TTS_STATIS --> SIMPAN
      TTS_FAR --> SIMPAN
      TTS_TANGGA --> SIMPAN
      TTS_DROP --> SIMPAN
      TTS_LUBANG --> SIMPAN
      SILENT --> SIMPAN
      MEDAN_AMAN --> SIMPAN
      SIMPAN --> KEMBALI([Kembali ke 3.5.2:<br/>Cek WiFi])

      %% Styling
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
      classDef adaptive fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
      classDef yolo fill:#fff9c4,stroke:#f57f17,stroke-width:2px;
      classDef terrain fill:#e0f2f1,stroke:#00695c,stroke-width:2px;
      classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;

      class TTS_INFO,TTS_WARN,TTS_OBJ,TTS_STATIS,TTS_FAR,TTS_TANGGA,TTS_DROP,TTS_LUBANG,SILENT impl;
      class CEK_GERAK,MAPPING,CEK_MODE,CEK_JANGKAUAN,CEK_ACCEL,F_U,F_O,CEK_STATIS,CEK_BBOX,CEK_ANOMALI,CEK_POLA,CEK_YOLO_T decision;
      class DELTA_TOF,SPEED_U,SPEED_O,TH_U,TH_O adaptive;
      class BBOX yolo;
      class CEK_BAWAH,ANALISIS,MEDAN_AMAN terrain;
      class PARALEL,SIMPAN,LUAR,DALAM,HITUNG_GRID,PAUSE_YOLO,SENSOR_ONLY design;
   ```

   Flowchart di atas menampilkan keseluruhan alur secara ringkas. Karena ukurannya terlalu besar untuk dicetak pada kertas A4 dengan detail yang jelas, algoritma ini **dipecah menjadi 5 flowchart** yang lebih detail:

   | Flowchart | Fungsi | Area pada Gambar Besar |
   |---|---|---|
   | **3a** | Pemrosesan data & mapping arah jam | Bagian atas (Mode Smart → Mapping) |
   | **3b** | Percabangan mode aplikasi | Bagian tengah (Mode Aplikasi → 3 jalur) |
   | **3c** | Jalur A — Deteksi objek dekat (ToF ≤ 4m) | Cabang kiri (ΔJarak → Threshold Adaptif) |
   | **3d** | Jalur B — Deteksi kendaraan jauh (YOLO > 4m) | Cabang tengah (ΔBBox → Peringatan) |
   | **3e** | Jalur C — Deteksi medan (tangga / lubang) | Cabang kanan (ToF Baris Bawah → Analisis) |

   Berikut penjelasan detail masing-masing flowchart:



   **Flowchart 3a — Pemrosesan Data & Mapping**

   Tahap pertama: sistem mengecek gerakan user via accelerometer, kemudian memproses data video (YOLO) dan data jarak (ToF) secara paralel, lalu menggabungkannya di logika mapping arah jam.

   ```mermaid
   flowchart TD
      MODE_SMART([Mode Smart / AI]) --> CEK_GERAK{Accelerometer:  User Bergerak?}

      %% User Diam - Pause YOLO, Sensor Tetap Jalan
      CEK_GERAK -- Diam > 10 Detik --> PAUSE_YOLO[Pause YOLO & Streaming]
      PAUSE_YOLO --> SENSOR_ONLY[Sensor VL53L5CX Tetap Aktif + Buzzer]
      SENSOR_ONLY -- User Bergerak Lagi --> CEK_GERAK

      %% User Bergerak - Proses Normal
      CEK_GERAK -- Ya --> KIRIM_DATA[Kirim Video + Data Jarak ke HP]

      %% Fork - Paralel Processing
      KIRIM_DATA --> YOLO[Proses Citra AI YOLOv11]
      KIRIM_DATA --> ToF[Proses Matriks Jarak VL53L5CX]

      %% Join - Penggabungan
      YOLO --> MAPPING{Logika Mapping  Arah Jam 10-2}
      ToF --> MAPPING

      %% Logika Percabangan Arah Jam
      MAPPING -- X < 20% / X > 80% --> LUAR[Set: Arah Jam 10 atau 2  Status: Jarak Unknown]
      MAPPING -- X = 20% - 80% --> DALAM[Set: Arah Jam 11, 12, atau 1]

      DALAM --> HITUNG_GRID[Hitung Grid Pixel/60  Ambil Data Array Jarak]

      LUAR --> GOTO_OUTPUT([Lanjut ke  Flowchart 3b])
      HITUNG_GRID --> GOTO_OUTPUT

      %% Styling
      classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
      classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;

      class KIRIM_DATA,YOLO,ToF,HITUNG_GRID research;
      class LUAR,DALAM,PAUSE_YOLO,SENSOR_ONLY design;
      class CEK_GERAK,MAPPING decision;
   ```

   **Penjelasan langkah demi langkah:**

   1. **Cek Accelerometer** — Sistem membaca data accelerometer smartphone untuk mendeteksi apakah user sedang bergerak (berjalan) atau diam (berdiri/duduk).
      - **Diam > 10 detik**: Jika user tidak bergerak selama lebih dari 10 detik, YOLO dan video streaming di-pause untuk menghemat baterai dan sumber daya. Namun sensor jarak VL53L5CX **tetap aktif** beserta buzzer untuk menjaga keamanan.
      - **User bergerak lagi**: Saat accelerometer mendeteksi gerakan, YOLO otomatis di-resume.
      - **Ya (bergerak)**: Lanjut ke pemrosesan normal.
   2. **Kirim Data ke HP** — ESP32 mengirimkan dua jenis data ke smartphone via WebSocket secara bersamaan: frame video dari kamera OV2640 dan matriks jarak 8×8 dari sensor VL53L5CX.
   3. **Proses Paralel** — Smartphone memproses kedua data secara paralel:
      - **YOLO (AI)**: Model YOLOv11 Nano (TFLite) mendeteksi objek pada frame video. Output berupa label objek dan koordinat bounding box (posisi X).
      - **ToF (Jarak)**: Matriks jarak 8×8 dari VL53L5CX diproses menjadi data jarak per zona.
   4. **Mapping Arah Jam** — Hasil deteksi YOLO (posisi X objek) digabungkan dengan data jarak ToF. Posisi horizontal objek dipetakan ke arah jam (10, 11, 12, 1, 2):
      - **X < 20% atau X > 80%**: Objek berada di tepi frame (luar jangkauan ToF). Arah jam di-set ke 10 atau 2 dengan status jarak *unknown* karena sensor tidak meng-cover area tersebut.
      - **X = 20% - 80%**: Objek berada dalam jangkauan sensor. Arah jam dipetakan ke 11, 12, atau 1.
   5. **Hitung Grid** — Untuk objek dalam jangkauan, sistem menghitung zona grid (pixel ÷ 60) untuk menentukan kolom sensor ToF yang sesuai, lalu mengambil data jarak dari array sensor pada kolom tersebut.
   6. **Lanjut ke Flowchart 3b** — Data hasil mapping (objek + arah + jarak) diteruskan untuk keputusan output.

   ---

   **Flowchart 3b — Percabangan Mode Aplikasi**

   Tahap kedua: menentukan mode aplikasi dan memulai **tiga jalur deteksi paralel**. Pada Mode Otonom, sistem menjalankan deteksi objek (Flowchart 3c/3d) dan deteksi medan (Flowchart 3e) secara bersamaan. Pada Mode Tanya Jawab, sistem langsung melaporkan semua objek.

   ```mermaid
   flowchart TD
      START_OUTPUT([Mulai: Data Hasil Mapping]) --> CEK_MODE{Cek Mode Aplikasi?}

      %% Mode Tanya Jawab
      CEK_MODE -- Mode Tanya Jawab --> TTS_INFO[Suara Info:<br/>Objek + Arah + Jarak]

      %% Mode Otonom
      CEK_MODE -- Mode Otonom --> PARALEL[Proses Paralel:<br/>Deteksi Objek + Deteksi Medan]

      PARALEL --> CEK_JANGKAUAN{Objek dalam<br/>Jangkauan ToF?<br/>≤ 4 Meter}
      PARALEL --> GOTO_MEDAN([Flowchart 3e:<br/>Deteksi Medan])

      CEK_JANGKAUAN -- Ya: ≤ 4m --> GOTO_TOF([Flowchart 3c:<br/>Deteksi Objek Dekat])
      CEK_JANGKAUAN -- Tidak: > 4m --> GOTO_YOLO([Flowchart 3d:<br/>Deteksi Kendaraan Jauh])

      %% Akhir
      TTS_INFO --> SIMPAN((Simpan Data<br/>Frame))
      SIMPAN --> KEMBALI([Kembali ke 3.5.2:<br/>Cek WiFi])

      %% Styling
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
      classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;

      class TTS_INFO impl;
      class CEK_MODE,CEK_JANGKAUAN decision;
      class PARALEL,SIMPAN design;
   ```

   **Penjelasan langkah demi langkah:**

   1. **Data Hasil Mapping** — Menerima data dari Flowchart 3a berupa: label objek, arah jam (10-2), dan jarak (jika tersedia).
   2. **Cek Mode Aplikasi** — Aplikasi memiliki dua mode yang dapat dipilih user:
      - **Mode Otonom**: Sistem hanya memperingatkan user jika ada bahaya. Masuk ke proses paralel tiga jalur deteksi.
      - **Mode Tanya Jawab**: Sistem langsung menyebutkan semua objek yang terdeteksi beserta arah dan jarak.
   3. **Proses Paralel** — Pada mode otonom, dua proses berjalan bersamaan:
      - **Deteksi Objek**: Cek jangkauan ToF untuk menentukan Jalur A (≤4m, Flowchart 3c) atau Jalur B (>4m, Flowchart 3d).
      - **Deteksi Medan**: Analisis anomali pola ToF untuk tangga/lubang (Flowchart 3e).
   4. **Simpan Data Frame** — Di akhir siklus, data jarak dan bounding box disimpan untuk perbandingan di frame berikutnya.

   > **Catatan:** Pada Mode Otonom, alur kembali ke node "Simpan Data" dilakukan melalui masing-masing flowchart 3c/3d/3e — setiap jalur memiliki node kembali sendiri yang mengarah ke proses simpan data sebelum kembali ke pengecekan WiFi (3.5.2).

   ---

   **Flowchart 3c — Jalur A: Deteksi Objek Dekat (ToF ≤ 4m, Threshold Adaptif)**

   Jalur ini aktif saat objek terdeteksi **dalam jangkauan sensor ToF (≤ 4 meter)**. Threshold peringatan bersifat **adaptif** — semakin cepat objek mendekat, semakin jauh threshold-nya.

   ```mermaid
   flowchart TD
      START_A([Dari Flowchart 3b:<br/>Objek ≤ 4m]) --> DELTA_TOF[Hitung Delta Jarak ToF:<br/>ΔJarak = Jarak Lama − Jarak Baru]

      DELTA_TOF --> CEK_ACCEL{Cek Accelerometer<br/>Smartphone}

      CEK_ACCEL -- User Bergerak --> SPEED_U[Kecepatan Pendekatan =<br/>ΔJarak / Waktu Antar Frame]
      CEK_ACCEL -- User Diam +<br/>ΔJarak Positif --> SPEED_O[Kecepatan Objek =<br/>ΔJarak / Waktu Antar Frame]
      CEK_ACCEL -- User Diam +<br/>ΔJarak = 0 --> CEK_STATIS{Jarak < 1m?}

      SPEED_U --> TH_U[Threshold Adaptif =<br/>1m + Kecepatan × 2 detik<br/>Max: 4m]
      SPEED_O --> TH_O[Threshold Adaptif =<br/>1m + Kecepatan × 2 detik<br/>Max: 4m]

      TH_U --> F_U{Jarak < Threshold?}
      TH_O --> F_O{Jarak < Threshold?}

      F_U -- Ya --> TTS_WARN[Suara: AWAS<br/>Objek + Jam X + Jarak]
      F_U -- Tidak --> SILENT[Diam / Tidak Ada Suara]

      F_O -- Ya --> TTS_OBJ[Suara: AWAS Objek<br/>Mendekat + Jam X + Jarak]
      F_O -- Tidak --> SILENT

      CEK_STATIS -- Ya + Belum Pernah<br/>Diperingatkan --> TTS_STATIS[Suara: Objek Dekat<br/>di Jam X, Jarak Y]
      CEK_STATIS -- Tidak / Sudah<br/>Diperingatkan --> SILENT

      TTS_WARN --> KEMBALI([Kembali ke<br/>Flowchart 3b: Simpan Data])
      TTS_OBJ --> KEMBALI
      TTS_STATIS --> KEMBALI
      SILENT --> KEMBALI

      %% Styling
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
      classDef adaptive fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;

      class TTS_WARN,TTS_OBJ,TTS_STATIS,SILENT impl;
      class CEK_ACCEL,F_U,F_O,CEK_STATIS decision;
      class DELTA_TOF,SPEED_U,SPEED_O,TH_U,TH_O adaptive;
   ```

   **Penjelasan langkah demi langkah:**

   1. **Hitung Delta Jarak** — Menghitung selisih jarak objek antara frame sekarang dan frame sebelumnya: `ΔJarak = Jarak Lama − Jarak Baru`. Jika positif, objek mendekat.
   2. **Cek Accelerometer Smartphone** — Menentukan siapa yang bergerak, dengan tiga kemungkinan:
      - **User bergerak**: User berjalan mendekati objek statis (misalnya tiang, tembok).
      - **User diam + ΔJarak positif**: Objek bergerak mendekati user yang diam (misalnya kendaraan).
      - **User diam + ΔJarak = 0**: Baik user maupun objek tidak bergerak (misalnya user berdiri di dekat tiang). Cek apakah jarak < 1 meter.
   3. **Hitung Kecepatan Pendekatan** — `Kecepatan = ΔJarak / Waktu Antar Frame` (dalam m/s). Sumber data: sensor jarak ToF VL53L5CX.
   4. **Threshold Adaptif** — `Threshold = 1m + (Kecepatan × 2 detik)`, dengan batas maksimum 4 meter (jangkauan sensor). Contoh: user jalan 1 m/s → threshold 3 meter; objek & user diam → threshold tetap 1 meter.
   5. **Filter & Output** — Jika jarak saat ini kurang dari threshold → keluarkan peringatan TTS dengan info lengkap (nama objek, arah jam, jarak). Jika tidak → diam.
   6. **Peringatan Objek Statis** — Jika user dan objek sama-sama diam, dan jarak < 1 meter, sistem memberikan peringatan **satu kali** (tidak diulang-ulang). Peringatan akan di-reset jika user bergerak atau objek berubah posisi. Hal ini mencegah spam suara saat user sengaja berdiri di dekat objek.

   ---

   **Flowchart 3d — Jalur B: Deteksi Kendaraan Jauh (YOLO > 4m)**

   Jalur ini aktif saat objek terdeteksi YOLO tetapi **di luar jangkauan sensor ToF (> 4 meter)**. Sistem menggunakan perubahan ukuran **bounding box YOLO** antar frame untuk mendeteksi objek yang mendekat cepat (misalnya kendaraan).

   ```mermaid
   flowchart TD
      START_B([Dari Flowchart 3b:<br/>Objek > 4m]) --> BBOX[Hitung Delta Bounding Box:<br/>ΔBBox = BBox Baru − BBox Lama]

      BBOX --> CEK_BBOX{BBox Membesar<br/>Signifikan?<br/>Area +20% Antar Frame}

      CEK_BBOX -- Ya: Objek Mendekat --> TTS_FAR[Suara: AWAS<br/>Kendaraan Mendekat<br/>dari Arah Jam X]
      CEK_BBOX -- Tidak: Stabil/Mengecil --> SILENT[Diam / Tidak Ada Suara]

      TTS_FAR --> KEMBALI([Kembali ke<br/>Flowchart 3b: Simpan Data])
      SILENT --> KEMBALI

      %% Styling
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
      classDef yolo fill:#fff9c4,stroke:#f57f17,stroke-width:2px;

      class TTS_FAR,SILENT impl;
      class CEK_BBOX decision;
      class BBOX yolo;
   ```

   **Penjelasan langkah demi langkah:**

   1. **Hitung Delta Bounding Box** — Membandingkan ukuran (area) bounding box objek yang sama antara frame saat ini dan frame sebelumnya. Jika bbox membesar, objek mendekat ke kamera.
   2. **Cek Signifikansi** — Jika area bounding box bertambah lebih dari 20% dalam satu interval frame, objek dianggap **mendekati dengan kecepatan tinggi** (misalnya kendaraan bermotor).
      - **Ya**: Keluarkan peringatan mendesak via TTS. Peringatan **tanpa info jarak** karena ToF tidak bisa mengukur di luar 4 meter.
      - **Tidak**: BBox stabil atau mengecil → objek diam atau menjauh → tidak ada peringatan.
   3. **Catatan Penting**: Jalur ini hanya bisa menginformasikan bahwa objek **mendekat**, tetapi tidak bisa memberikan jarak pasti. Saat objek akhirnya masuk jangkauan ToF (≤ 4m), sistem otomatis berpindah ke Jalur A (Flowchart 3c) yang memberikan jarak akurat.

   ---

   **Flowchart 3e — Jalur C: Deteksi Medan (Tangga / Lubang)**

   Jalur ini berjalan **paralel** dengan deteksi objek (Jalur A/B) setiap frame. Sistem menganalisis pola jarak pada **baris bawah matriks ToF 8×8** untuk mendeteksi perubahan medan (tangga turun, lubang, parit).

   ```mermaid
   flowchart TD
      START_C([Dari Flowchart 3b:<br/>Deteksi Medan]) --> CEK_BAWAH[Baca ToF Baris 7-8<br/>Bandingkan dengan Baris 4-5]

      CEK_BAWAH --> CEK_ANOMALI{Anomali Medan?<br/>Rasio Bawah/Tengah<br/> > 0.8}

      CEK_ANOMALI -- Tidak: Normal --> MEDAN_AMAN[Medan Datar: Aman]

      CEK_ANOMALI -- Ya: Anomali --> ANALISIS[Analisis Pola<br/>Seluruh Matriks 8×8]

      ANALISIS --> CEK_POLA{Pola Jarak<br/>Antar Baris?}

      CEK_POLA -- Gradual Naik +<br/>Merata Semua Kolom --> CEK_YOLO{YOLO Deteksi<br/>Tangga?}
      CEK_POLA -- Tiba-tiba Besar<br/>di Zona Tertentu --> TTS_LUBANG[Suara: AWAS<br/>Lubang di Depan!]

      CEK_YOLO -- Ya --> TTS_TANGGA[Suara: Tangga<br/>di Depan]
      CEK_YOLO -- Tidak --> TTS_DROP[Suara: AWAS<br/>Penurunan di Depan!]

      TTS_TANGGA --> KEMBALI([Kembali ke<br/>Flowchart 3b: Simpan Data])
      TTS_DROP --> KEMBALI
      TTS_LUBANG --> KEMBALI
      MEDAN_AMAN --> KEMBALI

      %% Styling
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;
      classDef terrain fill:#e0f2f1,stroke:#00695c,stroke-width:2px;

      class TTS_TANGGA,TTS_DROP,TTS_LUBANG impl;
      class CEK_ANOMALI,CEK_POLA,CEK_YOLO decision;
      class CEK_BAWAH,ANALISIS,MEDAN_AMAN terrain;
   ```

   **Penjelasan langkah demi langkah:**

   1. **Baca ToF Baris Bawah** — Setiap frame, sistem membaca rata-rata jarak baris 7-8 (area paling bawah FoV, melihat lantai dekat) dan membandingkan dengan baris 4-5 (area tengah FoV, melihat lantai jauh).
   2. **Cek Anomali Medan** — Pada lantai datar normal, baris bawah harusnya **lebih dekat** dari baris tengah (rasio < 0.7). Jika rasio > 0.8 (baris bawah hampir sama atau lebih jauh dari tengah), berarti ada **perubahan medan** — lantai di depan kaki user "hilang" atau berubah ketinggian.
   3. **Analisis Pola Matriks** — Jika anomali terdeteksi, sistem menganalisis pola seluruh matriks 8×8:
      - **Gradual naik + merata semua kolom**: Jarak bertambah bertahap dan merata di semua kolom (std deviasi kecil) → pola khas **tangga** atau **penurunan bertingkat**. Konfirmasi dengan YOLO.
      - **Tiba-tiba besar di zona tertentu**: Jarak melonjak hanya di beberapa kolom (std deviasi besar) → pola khas **lubang** atau **parit/selokan**.
   4. **Konfirmasi YOLO** — Untuk pola gradual:
      - **YOLO deteksi tangga**: Informasi deskriptif *"Tangga di depan"* (bukan peringatan bahaya).
      - **YOLO tidak deteksi tangga**: Peringatan *"AWAS, penurunan di depan!"* (bisa tangga turun yang tidak terlihat secara visual).

   ---

   ## 3.5.4. Algoritma Mode Darurat (Fail-Safe)

   Algoritma ini mencakup dua mode fallback untuk kondisi non-ideal. Kedua mode disatukan dalam satu flowchart karena sama-sama hanya mengandalkan **sensor jarak VL53L5CX + buzzer** tanpa pemrosesan AI (YOLO tidak aktif).

   > **Catatan:** Pada mode darurat, threshold jarak tetap menggunakan **1 meter** (bukan threshold adaptif seperti di Flowchart 3c). Hal ini karena pada mode darurat, ESP32 beroperasi mandiri tanpa smartphone — tidak ada data accelerometer maupun YOLO, sehingga kecepatan pendekatan tidak dapat dihitung. Threshold tetap 1 meter dipilih sebagai jarak aman minimum untuk pejalan kaki.

   ```mermaid
   flowchart TD
      %% LOGIKA OFFLINE / FAIL-SAFE
      ENTRY_OFF([WiFi Putus > 5 Detik]) --> MODE_SAFETY[MODE SAFETY / OFFLINE]
      MODE_SAFETY --> CAM_OFF[Matikan Kamera]
      MODE_SAFETY --> BACA_SENSOR[Baca Sensor Jarak VL53L5CX]

      BACA_SENSOR --> LOGIKA_BUZZER{Jarak < 1 Meter?}
      LOGIKA_BUZZER -- Ya --> BUZZ_ON[Bunyikan Buzzer]
      LOGIKA_BUZZER -- Tidak --> BUZZ_OFF[Buzzer Diam]
      BUZZ_ON --> RETRY[Coba Reconnect WiFi]
      BUZZ_OFF --> RETRY
      RETRY --> CEK_ULANG{Reconnect Berhasil?}
      CEK_ULANG -- Ya --> KEMBALI_UTAMA([Kembali ke 3.5.2:  Cek Kecerahan])
      CEK_ULANG -- Tidak --> BACA_SENSOR

      %% LOGIKA GELAP
      ENTRY_GELAP([Cahaya Gelap]) --> MODE_LOW[MODE LOW-LIGHT]
      MODE_LOW --> CAM_LOW[Kamera Low FPS]
      MODE_LOW --> STOP_STREAM[Stop Video Streaming]
      MODE_LOW --> BACA_SENSOR

      %% Styling
      classDef research fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
      classDef design fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
      classDef impl fill:#fff3e0,stroke:#e65100,stroke-width:2px;
      classDef decision fill:#fce4ec,stroke:#c62828,stroke-width:2px;

      class BACA_SENSOR,CAM_OFF,CAM_LOW,STOP_STREAM research;
      class MODE_SAFETY,MODE_LOW,RETRY design;
      class BUZZ_ON,BUZZ_OFF impl;
      class LOGIKA_BUZZER,CEK_ULANG decision;
   ```

   **Penjelasan langkah demi langkah:**

   **Mode Offline / Safety (WiFi Putus):**

   1. **WiFi Putus > 5 Detik** — Jika koneksi WiFi terputus lebih dari 5 detik, sistem mengasumsikan koneksi benar-benar hilang (bukan hanya fluktuasi sesaat).
   2. **Mode Safety Aktif** — Sistem masuk ke mode darurat. Dua aksi dilakukan secara bersamaan:
      - **Matikan Kamera**: Kamera dimatikan untuk menghemat daya, karena tanpa WiFi, video tidak bisa dikirim ke smartphone untuk diproses YOLO.
      - **Baca Sensor Jarak**: Sensor VL53L5CX tetap aktif dan terus membaca data jarak secara langsung di ESP32.
   3. **Cek Jarak < 1 Meter** — Jika ada objek yang terdeteksi pada jarak kurang dari 1 meter:
      - **Ya**: Buzzer pada ESP32 dibunyikan sebagai peringatan langsung tanpa melalui smartphone.
      - **Tidak**: Buzzer diam.
   4. **Coba Reconnect** — Setelah setiap siklus baca sensor, ESP32 mencoba menghubungkan ulang WiFi.
      - **Berhasil**: Kembali ke 3.5.2 (Cek Kecerahan) untuk menentukan mode selanjutnya.
      - **Gagal**: Kembali ke loop baca sensor dan coba lagi.

   **Mode Gelap (Low-Light):**

   1. **Cahaya Gelap** — Kamera mendeteksi bahwa lingkungan terlalu gelap. YOLO tidak dapat memproses gambar dengan akurasi memadai pada kondisi ini.
   2. **Mode Low-Light Aktif** — Dua penyesuaian dilakukan:
      - **Kamera Low FPS**: FPS kamera diturunkan untuk mengurangi konsumsi daya (kamera tidak sepenuhnya dimatikan karena masih digunakan untuk pengecekan brightness secara periodik).
      - **Stop Video Streaming**: Pengiriman frame video ke smartphone dihentikan karena tidak berguna untuk YOLO.
   3. **Baca Sensor Jarak** — Sama seperti Mode Offline, sistem mengandalkan sensor VL53L5CX + buzzer untuk deteksi rintangan dasar. Logika buzzer identik dengan mode offline.
