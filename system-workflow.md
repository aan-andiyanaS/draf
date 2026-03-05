# System Workflow & Architecture

This document explains the system architecture based on `diagram-block-sistem.png` and details the initial setup workflow.

## 1. System Block Diagram Overview

The system consists of three main units:

1.  **Wearable Head Unit (IoT Device)**
    *   **ESP32-S3 (N16R8):** The core controller.
    *   **OV2640 Camera:** Captures visual data.
    *   **VL53L5CX ToF Sensor:** Measures distance (depth sensing).
    *   **Connectivity:** Communicates via WiFi (WebSocket) and Bluetooth (Provisioning).

2.  **Processing Unit (Smartphone)**
    *   **Kotlin App:** Acts as the central hub and WebSocket Server.
    *   **AI Processing:** Uses NPU/GPU to run YOLOv11 Nano (TFLite) for object detection.
    *   **Logic Fusion:** Combines visual data and distance data to make decisions.

3.  **User Interaction Unit**
    *   **Audio Output:** Text-to-Speech feedback via Bluetooth earphones.
    *   **Audio Input:** Voice commands via microphone.

## 2. Initial Setup Workflow (Provisioning)

The following flowchart illustrates the process when the device is turned on for the first time.

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
    End([Selesai])

    Start --> ActivateBLE --> UserPrep --> ScanPilih
    ScanPilih --> ConnBLE --> ProvWiFi --> KirimKoneksi
    KirimKoneksi --> CheckConn
    CheckConn -- Ya --> SaveAktif --> End
    CheckConn -- Tidak --> ProvWiFi

    %% Styling
    classDef mobile fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef iot fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef user fill:#fff3e0,stroke:#e65100,stroke-width:2px;

    class ScanPilih,ProvWiFi,SaveAktif mobile;
    class Start,ActivateBLE,ConnBLE,KirimKoneksi,End iot;
    class UserPrep user;
    class CheckConn iot;
```

### Penjelasan Alur:

1.  **Memulai Program:**
    *   Nyalakan perangkat IoT.
    *   BLE pada perangkat IoT akan otomatis aktif (advertising).
2.  **Aksi User:**
    *   User menyalakan Bluetooth dan Hotspot pada smartphone.
    *   User membuka aplikasi Android.
3.  **Scanning & Koneksi BLE:**
    *   Aplikasi menampilkan menu scan BLE.
    *   User memilih perangkat IoT yang muncul di daftar scan.
4.  **Provisioning WiFi:**
    *   Setelah terkoneksi via BLE, perangkat IoT melakukan scanning WiFi di sekitar.
    *   Hasil scan dikirim ke aplikasi via BLE.
    *   User memilih WiFi (Hotspot Smartphone) dan perangkat IoT akan mencoba terhubung.
5.  **Aktivasi Sistem:**
    *   Jika koneksi WiFi berhasil, perangkat IoT menyimpan kredensial WiFi ke memori (NVS).
    *   Sistem pada blok Mobile (Processing Unit) akan aktif sepenuhnya (siap menerima stream data).
