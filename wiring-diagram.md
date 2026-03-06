# Wiring Diagram — Perangkat Kacamata Pintar

Dokumen ini menjelaskan skema koneksi (wiring) seluruh komponen hardware pada perangkat wearable berdasarkan diagram blok sistem.

**Board yang digunakan:** ESP32-S3 WROOM N16R8 + Kamera OV2640 2MP (Freenove / Generic)

---

## 1. Pin Mapping — Kamera OV2640 (Built-in)

Kamera OV2640 sudah **terhubung langsung** ke board ESP32-S3 WROOM melalui konektor FPC 24-pin. **Tidak perlu wiring manual.** Pin-pin berikut sudah digunakan oleh kamera dan **tidak boleh dipakai** untuk komponen lain.

| Fungsi Kamera | GPIO | Keterangan |
|---|---|---|
| XCLK | GPIO15 | External clock |
| SIOD (SDA) | GPIO4 | I2C SDA (kontrol kamera) |
| SIOC (SCL) | GPIO5 | I2C SCL (kontrol kamera) |
| D0 (Y2) | GPIO11 | Data bit 0 |
| D1 (Y3) | GPIO9 | Data bit 1 |
| D2 (Y4) | GPIO8 | Data bit 2 |
| D3 (Y5) | GPIO10 | Data bit 3 |
| D4 (Y6) | GPIO12 | Data bit 4 |
| D5 (Y7) | GPIO18 | Data bit 5 |
| D6 (Y8) | GPIO17 | Data bit 6 |
| D7 (Y9) | GPIO16 | Data bit 7 |
| VSYNC | GPIO6 | Vertical sync |
| HREF | GPIO7 | Horizontal reference |
| PCLK | GPIO13 | Pixel clock |

> **Total GPIO dipakai kamera: 14 pin** — sisa GPIO tersedia untuk sensor, buzzer, dan tombol.

---

## 2. Pin Mapping — Sensor VL53L5CX (I2C)

Sensor jarak Time-of-Flight VL53L5CX terhubung ke ESP32 via **I2C**. Menggunakan GPIO yang **berbeda** dari I2C kamera (kamera pakai GPIO4/GPIO5).

```mermaid
flowchart LR
    subgraph VL53L5CX ["VL53L5CX Breakout Board"]
        VIN["VIN"]
        GND_S["GND"]
        SCL_S["SCL"]
        SDA_S["SDA"]
        INT_S["INT"]
        LPN_S["LPn"]
    end

    subgraph ESP32 ["ESP32-S3 WROOM"]
        P3V3["3.3V"]
        GND_E["GND"]
        G2["GPIO2 (SCL)"]
        G1["GPIO1 (SDA)"]
        G3["GPIO3 (INT)"]
        G14["GPIO14 (LPn)"]
    end

    VIN ---|Merah| P3V3
    GND_S ---|Hitam| GND_E
    SCL_S ---|Hijau| G2
    SDA_S ---|Biru| G1
    INT_S ---|Kuning| G3
    LPN_S ---|Putih| G14

    classDef sensor fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef mcu fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    class VIN,GND_S,SCL_S,SDA_S,INT_S,LPN_S sensor;
    class P3V3,GND_E,G2,G1,G3,G14 mcu;
```

| Pin VL53L5CX | Pin ESP32 | Warna Kabel | Keterangan |
|---|---|---|---|
| **VIN** | **3.3V** | 🔴 Merah | Power supply 3.3V dari board |
| **GND** | **GND** | ⚫ Hitam | Ground |
| **SCL** | **GPIO2** | 🟢 Hijau | I2C Clock |
| **SDA** | **GPIO1** | 🔵 Biru | I2C Data |
| **INT** | **GPIO3** | 🟡 Kuning | Interrupt (data ready) — opsional |
| **LPn** | **GPIO14** | ⚪ Putih | Low Power enable — opsional |

**Catatan penting:**
- GPIO1 dan GPIO2 dipilih karena **tidak dipakai** oleh kamera dan merupakan pin I2C kedua yang umum dipakai di ESP32-S3.
- Kamera menggunakan I2C pada GPIO4 (SDA) dan GPIO5 (SCL) — ini adalah **bus I2C terpisah**, sehingga tidak ada konflik.
- Pin **INT** dan **LPn** bersifat opsional. INT digunakan jika ingin interrupt-driven reading (lebih efisien). LPn digunakan untuk mengontrol mode low-power sensor.

---

## 3. Pin Mapping — Buzzer Aktif (Fail-Safe)

Buzzer aktif terhubung langsung ke GPIO ESP32. Buzzer aktif hanya membutuhkan sinyal HIGH/LOW (tidak perlu PWM frekuensi tertentu).

```mermaid
flowchart LR
    subgraph BUZZER ["Buzzer Aktif 3.3V"]
        BZ_P["+ (Positif)"]
        BZ_N["- (Negatif)"]
    end

    subgraph ESP32 ["ESP32-S3 WROOM"]
        G38["GPIO38"]
        GND_E["GND"]
    end

    BZ_P ---|Merah| G38
    BZ_N ---|Hitam| GND_E

    classDef output fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef mcu fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    class BZ_P,BZ_N output;
    class G38,GND_E mcu;
```

| Pin Buzzer | Pin ESP32 | Warna Kabel | Keterangan |
|---|---|---|---|
| **+ (Positif)** | **GPIO38** | 🔴 Merah | Sinyal kontrol (HIGH = bunyi) |
| **- (Negatif)** | **GND** | ⚫ Hitam | Ground |

**Catatan:**
- Gunakan **buzzer aktif 3.3V** (bukan pasif) agar bisa dikontrol hanya dengan `digitalWrite(38, HIGH)`.
- GPIO38 dipilih karena tersedia dan aman untuk output digital.
- Jika arus buzzer > 12mA (batas GPIO ESP32), tambahkan **transistor NPN** (misalnya 2N2222) sebagai driver.

---

## 4. Wiring — Manajemen Daya (Li-Po + TP4056 + MT3608)

Karena board ESP32-S3 WROOM menggunakan regulator **AMS1117-3.3V** (dropout ~1.1V, minimum input 4.4V), baterai Li-Po 3.7V **tidak bisa langsung** masuk ke pin 5V. Diperlukan **boost converter MT3608** untuk menaikkan tegangan ke 5V.

### Jalur Daya

```
Li-Po 3.7V → TP4056 (Charging + Proteksi) → MT3608 (Boost 3.7V → 5V) → Pin 5V ESP32 → AMS1117 (5V → 3.3V)
```

```mermaid
flowchart LR
    subgraph BATERAI ["Sumber Daya"]
        LIPO["Baterai Li-Po<br/>3.7V 1000mAh"]
    end

    subgraph CHARGING ["Modul Charging"]
        TP4056["TP4056<br/>+ Proteksi"]
    end

    subgraph BOOST ["Boost Converter"]
        MT3608["MT3608<br/>3.7V → 5V"]
    end

    subgraph ESP32 ["ESP32-S3 WROOM"]
        PIN5V["Pin 5V"]
        AMS["AMS1117<br/>(Built-in)"]
        P3V3["3.3V Rail"]
    end

    LIPO -->|"B+ / B-"| TP4056
    TP4056 -->|"OUT+ / OUT-"| MT3608
    MT3608 -->|"5V Output"| PIN5V
    PIN5V --> AMS
    AMS -->|"3.3V"| P3V3

    classDef power fill:#fce4ec,stroke:#c62828,stroke-width:2px;
    classDef mcu fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    class LIPO,TP4056,MT3608 power;
    class PIN5V,AMS,P3V3 mcu;

    linkStyle 0,1,2,3,4 stroke:#f44336,stroke-width:2px;
```

### Tabel Koneksi Daya

| Dari | Pin | Ke | Pin | Warna Kabel | Keterangan |
|---|---|---|---|---|---|
| **Li-Po +** | Kabel Merah | **TP4056** | B+ | 🔴 Merah | Positif baterai |
| **Li-Po -** | Kabel Hitam | **TP4056** | B- | ⚫ Hitam | Negatif baterai |
| **TP4056** | OUT+ | **MT3608** | IN+ (VIN) | 🔴 Merah | Output baterai → input boost |
| **TP4056** | OUT- | **MT3608** | IN- (GND) | ⚫ Hitam | Ground |
| **MT3608** | OUT+ (VOUT) | **ESP32** | Pin 5V | 🔴 Merah | 5V output → board |
| **MT3608** | OUT- (GND) | **ESP32** | GND | ⚫ Hitam | Ground bersama |

### Catatan Penting

- **MT3608 harus di-set ke 5V** sebelum dihubungkan ke ESP32. Putar trimpot pada modul MT3608 sambil mengukur output dengan multimeter hingga tepat **5.0V**.
- **TP4056 berfungsi ganda**: (1) mengisi baterai saat USB terhubung ke TP4056, (2) proteksi over-discharge/over-charge baterai.
- **Charging**: Untuk mengisi baterai, hubungkan kabel Micro-USB ke **TP4056** (bukan ke ESP32). ESP32 tetap menyala saat baterai di-charge.
- **AMS1117** bawaan board menangani konversi 5V → 3.3V. Semua komponen (ESP32, kamera, sensor) mendapat daya 3.3V dari regulator ini.

### ⚠️ Peringatan: Jangan Hubungkan USB-C dan MT3608 Bersamaan

Pin 5V pada board ESP32-S3 WROOM bersifat **bidirectional** (bisa input dan output). Hal ini menimbulkan risiko jika dua sumber daya terhubung secara bersamaan:

| Kondisi | Aman? | Keterangan |
|---|---|---|
| Hanya MT3608 → Pin 5V (tanpa USB) | ✅ Aman | Mode operasi normal (wearable) |
| Hanya USB-C (tanpa MT3608) | ✅ Aman | Mode programming / debugging |
| USB-C + MT3608 bersamaan | ❌ Bahaya | Dua sumber tegangan bertabrakan → risiko kerusakan board/komponen |

**Prosedur aman saat upload kode:**
1. **Lepas** kabel MT3608 dari pin 5V ESP32
2. **Colok** USB-C ke ESP32 untuk upload/debug
3. **Cabut** USB-C setelah selesai
4. **Pasang kembali** kabel MT3608 ke pin 5V

**Solusi permanen (opsional):** Pasang **dioda Schottky** (misalnya 1N5817) di jalur output MT3608 → Pin 5V. Dioda ini mencegah arus dari USB back-feed ke MT3608, sehingga kedua sumber bisa terhubung bersamaan dengan aman.

---

## 5. Pin Mapping — Tombol Multifungsi Eksternal

Satu tombol push button eksternal yang dipasang di **posisi mudah dijangkau** pada frame kacamata (misalnya di gagang kacamata dekat telinga). Tombol ini menangani **tiga fungsi** berdasarkan pola tekanan.

```mermaid
flowchart LR
    subgraph BUTTON ["Push Button Eksternal"]
        B1["Pin 1"]
        B2["Pin 2"]
    end

    subgraph ESP32 ["ESP32-S3 WROOM"]
        G39["GPIO39"]
        GND_E["GND"]
    end

    B1 ---|Kuning| G39
    B2 ---|Hitam| GND_E

    classDef input fill:#fff9c4,stroke:#f9a825,stroke-width:2px;
    classDef mcu fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    class B1,B2 input;
    class G39,GND_E mcu;
```

| Pin Tombol | Pin ESP32 | Warna Kabel | Keterangan |
|---|---|---|---|
| **Pin 1** | **GPIO39** | 🟡 Kuning | Input digital dengan pull-up internal |
| **Pin 2** | **GND** | ⚫ Hitam | Ground |

### Pola Tekanan dan Fungsi

| Pola Tekanan | Durasi | Fungsi | Feedback |
|---|---|---|---|
| **Tekan singkat** (1×) | < 1 detik | **Ganti mode** (Otonom ↔ Tanya Jawab) | TTS: "Mode Otonom Aktif" / "Mode Tanya Jawab Aktif" |
| **Tekan 2× cepat** (double press) | 2× dalam 0.5 detik | **Trigger tanya** (hanya di Mode Tanya Jawab) | TTS menyebutkan semua objek yang terdeteksi |
| **Tekan panjang** | 3–5 detik | **Matikan perangkat** (deep sleep) | Buzzer: 2× beep pendek, LED mati |
| **Tekan sangat panjang** | > 8 detik | **Reset WiFi** (hapus kredensial NVS) | Buzzer: 3× beep panjang, LED berkedip cepat, lalu reboot ke mode provisioning |

**Catatan:**
- Menggunakan **pull-up internal** (`pinMode(39, INPUT_PULLUP)`). Saat tombol ditekan, GPIO39 = LOW.
- GPIO39 dipilih karena tersedia, aman untuk input, dan tidak ada fungsi strapping.
- **Tidak perlu resistor eksternal** — pull-up internal ESP32-S3 (~45kΩ) sudah cukup.
- Tombol dipasang di **gagang kacamata** agar mudah dijangkau oleh tunanetra tanpa perlu melihat.

---

## 6. Pin Mapping — LED Indikator

Satu LED eksternal yang menunjukkan status sistem. Dipasang di **sisi luar frame kacamata** agar terlihat oleh pendamping atau orang di sekitar.

```mermaid
flowchart LR
    subgraph LED_EXT ["LED + Resistor"]
        LA["Anoda (+)"]
        R220["Resistor 220Ω"]
        LK["Katoda (-)"]
    end

    subgraph ESP32 ["ESP32-S3 WROOM"]
        G48["GPIO48"]
        GND_E["GND"]
    end

    G48 ---|Hijau| LA
    LA --- R220
    R220 --- LK
    LK ---|Hitam| GND_E

    classDef led fill:#c8e6c9,stroke:#388e3c,stroke-width:2px;
    classDef mcu fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    class LA,R220,LK led;
    class G48,GND_E mcu;
```

| Pin LED | Pin ESP32 | Warna Kabel | Keterangan |
|---|---|---|---|
| **Anoda (+)** | **GPIO48** (via resistor 220Ω) | 🟢 Hijau | Output digital (HIGH = nyala) |
| **Katoda (-)** | **GND** | ⚫ Hitam | Ground |

### Pola LED dan Status Sistem

| Pola LED | Kecepatan | Status Sistem |
|---|---|---|
| **Menyala tetap** (solid) | — | Sistem aktif — Mode Smart berjalan normal |
| **Berkedip lambat** | 1× per 2 detik | Mode Darurat (Offline / Gelap) — ESP32 mandiri |
| **Berkedip cepat** | 3× per detik | BLE Provisioning aktif — menunggu koneksi dari app |
| **Berkedip 2× lalu jeda** | 2× cepat, jeda 1 detik | Mode Tanya Jawab aktif |
| **Mati** | — | Perangkat mati (deep sleep) atau tidak ada daya |

**Catatan:**
- **Resistor 220Ω wajib** — tanpa resistor, LED akan rusak. Perhitungan: $(3.3\text{V} - 2.0\text{V}) / 220\Omega \approx 6\text{mA}$ (aman untuk GPIO ESP32, maks 12mA).
- GPIO48 dipilih karena beberapa board ESP32-S3 sudah memiliki **LED built-in** di GPIO48. Jika board sudah punya LED built-in, **tidak perlu wiring tambahan**.
- Jika board tidak punya LED built-in di GPIO48, wiring manual diperlukan (LED + resistor 220Ω).

---

## 7. Ringkasan GPIO — Seluruh Komponen

### GPIO yang Digunakan

| GPIO | Fungsi | Komponen |
|---|---|---|
| GPIO4 | SIOD (I2C SDA) | Kamera OV2640 |
| GPIO5 | SIOC (I2C SCL) | Kamera OV2640 |
| GPIO6 | VSYNC | Kamera OV2640 |
| GPIO7 | HREF | Kamera OV2640 |
| GPIO8 | Y4 (Data 2) | Kamera OV2640 |
| GPIO9 | Y3 (Data 1) | Kamera OV2640 |
| GPIO10 | Y5 (Data 3) | Kamera OV2640 |
| GPIO11 | Y2 (Data 0) | Kamera OV2640 |
| GPIO12 | Y6 (Data 4) | Kamera OV2640 |
| GPIO13 | PCLK | Kamera OV2640 |
| GPIO15 | XCLK | Kamera OV2640 |
| GPIO16 | Y9 (Data 7) | Kamera OV2640 |
| GPIO17 | Y8 (Data 6) | Kamera OV2640 |
| GPIO18 | Y7 (Data 5) | Kamera OV2640 |
| GPIO1 | I2C SDA | VL53L5CX |
| GPIO2 | I2C SCL | VL53L5CX |
| GPIO3 | INT (opsional) | VL53L5CX |
| GPIO14 | LPn (opsional) | VL53L5CX |
| GPIO38 | Buzzer Output | Buzzer Aktif |
| **GPIO39** | **Input Tombol** | **Tombol Multifungsi Eksternal** |
| **GPIO48** | **Output LED** | **LED Indikator Eksternal** |

### GPIO yang Masih Tersedia

| GPIO | Status | Catatan |
|---|---|---|
| GPIO0 | ⚠️ Strapping | Tombol BOOT bawaan board |
| GPIO19 | ✅ Tersedia | USB D- (jangan pakai jika pakai USB native) |
| GPIO20 | ✅ Tersedia | USB D+ (jangan pakai jika pakai USB native) |
| GPIO21 | ✅ Tersedia | General purpose |
| GPIO40 | ✅ Tersedia | General purpose |
| GPIO41 | ✅ Tersedia | General purpose |
| GPIO42 | ✅ Tersedia | General purpose |
| GPIO43 | ✅ Tersedia | TX (default Serial) |
| GPIO44 | ✅ Tersedia | RX (default Serial) |
| GPIO45 | ⚠️ Strapping | Boot mode select |
| GPIO46 | ⚠️ Strapping | Boot mode select |
| GPIO47 | ✅ Tersedia | General purpose |

---

## 8. Skema Wiring Lengkap

Diagram koneksi seluruh komponen ke ESP32-S3 WROOM:

```mermaid
flowchart TB
    subgraph POWER_CHAIN ["Manajemen Daya"]
        LIPO["Baterai Li-Po<br/>3.7V 1000mAh"]
        TP4056["TP4056<br/>Charging + Proteksi"]
        MT3608["MT3608<br/>Boost 3.7V → 5V"]
    end

    subgraph ESP32_BOARD ["ESP32-S3 WROOM N16R8 + OV2640"]
        ESP32["ESP32-S3<br/>Mikrokontroler"]
        CAM["Kamera OV2640<br/>(Built-in via FPC)"]
        AMS["AMS1117<br/>(Built-in 5V→3.3V)"]
    end

    subgraph I2C_BUS ["I2C Bus (GPIO1 SDA / GPIO2 SCL)"]
        TOF["VL53L5CX<br/>Sensor Jarak ToF 8×8"]
    end

    subgraph OUTPUT ["Output"]
        BUZZ["Buzzer Aktif 3.3V<br/>(GPIO38)"]
        LED["LED Indikator<br/>(GPIO48 + R220Ω)"]
    end

    subgraph INPUT ["Input"]
        BTN["Tombol Multifungsi<br/>(GPIO39 + Pull-Up)"]
    end

    %% Jalur Daya
    LIPO --> TP4056
    TP4056 --> MT3608
    MT3608 -->|"5V"| AMS
    AMS -->|"3.3V"| ESP32

    %% Jalur Data
    CAM ---|FPC 24-pin<br/>Built-in| ESP32
    TOF ---|I2C: SDA=GPIO1<br/>SCL=GPIO2| ESP32
    ESP32 ---|GPIO38| BUZZ
    ESP32 ---|GPIO48| LED
    BTN ---|GPIO39| ESP32

    ESP32 <-.->|WiFi WebSocket<br/>& BLE Provisioning| PHONE["📱 Smartphone"]

    classDef mcu fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef sensor fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef output fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef power fill:#fce4ec,stroke:#c62828,stroke-width:2px;
    classDef input fill:#fff9c4,stroke:#f9a825,stroke-width:2px;
    classDef external fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,stroke-dasharray: 5 5;

    class ESP32,CAM,AMS mcu;
    class TOF sensor;
    class BUZZ,LED output;
    class BTN input;
    class LIPO,TP4056,MT3608 power;
    class PHONE external;

    linkStyle 0,1,2,3 stroke:#f44336,stroke-width:2px;
```

---

## 9. Ringkasan Komponen dan Kabel

| No | Komponen | Koneksi ke ESP32 | Jumlah Kabel | Keterangan |
|---|---|---|---|---|
| 1 | Kamera OV2640 | FPC 24-pin (built-in) | 0 (built-in) | Tidak perlu wiring manual |
| 2 | Sensor VL53L5CX | I2C (GPIO1, GPIO2) + opsional (GPIO3, GPIO14) | 4-6 kabel | VIN, GND, SDA, SCL + INT, LPn |
| 3 | Buzzer Aktif | GPIO38 + GND | 2 kabel | Positif ke GPIO, negatif ke GND |
| 4 | Tombol Multifungsi | GPIO39 + GND | 2 kabel | Pull-up internal, tidak perlu resistor |
| 5 | LED Indikator | GPIO48 + GND (via R220Ω) | 2 kabel + resistor | Anoda via resistor ke GPIO, katoda ke GND |
| 6 | Li-Po + TP4056 + MT3608 | Pin 5V + GND | 2 kabel ke board | Output MT3608 5V ke pin 5V board |
|   | **Total kabel manual** | | **~14 kabel** | |
