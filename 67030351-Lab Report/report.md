# Lab5-1-Wi-Fi-Phase1
## 6. ตารางบันทึกผลการทดลอง (Experiment Results)

ให้นักศึกษาบันทึกผลลัพธ์จากการสังเกตใน Serial Console ลงในตารางต่อไปนี้:

### 6.1 ตารางสรุปเปรียบเทียบการสแกนทั้ง 4 กรณี

| ข้อการทดลอง | เงื่อนไขการสแกน | สถานะ (Success/Error Code) | จำนวน AP ที่พบ (เครือข่าย) | เวลาที่ใช้ในการสแกน (ms) |
| :---: | :--- | :---: | :---: | :---: |
| **5.1.1** | สแกนทั่วไปทุก Channel | ESP_OK (0x0) / Success | 7 | 1395 |
| **5.1.2** | กำหนดสแกนเฉพาะ Channel 1 | ESP_OK (0x0) / Success | 6 | 200 |
| **5.1.3** | กำหนดสแกน SSID ที่มีจริง ("KMITL-Legacy") | ESP_OK (0x0) / Success | 2 | 1399 |
| **5.1.4** | กำหนดสแกน SSID ที่ไม่มีจริง ("NON_EXISTENT_AP_9999") | ESP_OK (0x0) / Success | 0 | 1398 |

### 6.2 ตารางรายละเอียด AP ที่พบจากการสแกนทั่วไป (ข้อ 5.1.1)

| ลำดับ | ชื่อเครือข่าย (SSID) | MAC Address (BSSID) | ความแรงสัญญาณ (RSSI: dBm) | ช่องความถี่ (Channel) | ประเภทการเข้ารหัส (Encryption Type) |
| :---: | :--- | :--- | :---: | :---: | :--- |
| 1 | KMITL-Legacy | 78:17:BE:C0:7D:A0 | -53 dBm | 1 | WPA2_ENTERPRISE |
| 2 | KMITL-IoT | 78:17:BE:C0:7D:A2 | -53 dBm | 1 | WPA2_PSK |
| 3 | KMITL-WIFI | 78:17:BE:C0:7D:A1 | -58 dBm | 1 | OPEN (No Password) |
| 4 | KMITL-Legacy | 78:17:BE:C0:66:20 | -85 dBm | 1 | WPA2_ENTERPRISE |
| 5 | KMITL-IoT | 78:17:BE:C0:66:22 | -85 dBm | 1 | WPA2_PSK |
| 6 | KMITL-IoT | 78:17:BE:A9:94:E2 | -85 dBm | 6 | WPA2_PSK |
| 7 | KMITL-WIFI | 78:17:BE:C0:66:21 | -86 dBm | 1 | OPEN (No Password) |

---

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **การกำหนดค่าในโครงสร้าง `wifi_scan_config_t` สำหรับสแกนเจาะจงเฉพาะช่องความถี่ (ข้อ 5.1.2) ช่วยลดเวลาในการสแกนเมื่อเทียบกับการสแกนทุกช่องความถี่ (ข้อ 5.1.1) อย่างไร และมีข้อจำกัดอย่างไร?**
- **Ans**
	- **การลดเวลาในการสแกน:** เมื่อกำหนด `.channel = 1` ตัวชิป Wi-Fi ของ ESP32 จะส่ง Probe Request และดักฟังสัญญาณอยู่บน Channel 1 เพียงช่องเดียว โดยไม่ต้องสลับความถี่ (Channel Switching) และไม่ต้องรอเวลา Scan Time ประจำแต่ละช่องสัญญาณครบทั้ง 13 ช่อง (Channels 1–13) ส่งผลให้เวลาสแกนลดลงอย่างมาก จาก **1,395 ms** เหลือเพียง **200 ms** (เร็วขึ้นเกือบ 7 เท่า)
	- **ข้อจำกัด:** จะค้นพบเฉพาะ Access Point ที่กระจายสัญญาณอยู่บนช่องความถี่ที่กำหนดเท่านั้น หากมี AP อื่นๆ อยู่บน Channel อื่น (เช่น Channel 6 หรือ 11) หรือหากไม่ทราบ Channel ที่แน่นอนของ AP เป้าหมาย จะไม่สามารถค้นพบได้

1. **เมื่อสังเกตผล Forensic Log ในข้อ 5.1.4 (สแกนหา SSID ที่ไม่มีอยู่จริง) ฟังก์ชัน `esp_wifi_scan_start()`, `esp_wifi_scan_get_ap_num()` และ `esp_wifi_scan_get_ap_records()` ส่งคืนค่าอย่างไร?
- **Ans**
   - **`esp_wifi_scan_start()`:** ส่งคืนค่า **`ESP_OK (0x0)`** เนื่องจากกระบวนการสแกนคลื่นวิทยุทำงานสำเร็จสมบูรณ์ตามคำสั่ง ไม่เกิดข้อผิดพลาดระดับฮาร์ดแวร์/ไดรเวอร์
   - **`esp_wifi_scan_get_ap_num()`:** ส่งคืนค่า **`ESP_OK (0x0)`** และคืนค่าตัวแปร `ap_count = 0` ซึ่งแสดงว่าไม่พบ AP ที่ตรงกับ SSID ที่ค้นหา
   - **`esp_wifi_scan_get_ap_records()`:** ในโปรแกรมมีเงื่อนไขตรวจสอบ `if (ap_count == 0)` จึงทำให้ฟังก์ชันนี้ **ไม่ถูกเรียกใช้งาน (Skipped)** เนื่องจากไม่มีรายการ AP ให้ดึงข้อมูลมาแสดงผล

1. **ค่าระดับความแรงสัญญาณ (RSSI) ที่แสดงเป็นตัวเลขติดลบ (เช่น -45 dBm กับ -80 dBm) ค่าใดแสดงถึงสัญญาณที่มีความแรงและความเสถียรมากกว่ากัน?
- **Ans**
   - ค่า **-45 dBm** มีความแรงและความเสถียรมากกว่า **-80 dBm**
   - **เหตุผล:** RSSI มีหน่วยเป็น dBm ซึ่งเป็นค่าลอการิทึมเทียบกับกำลังงาน 1 มิลลิวัตต์ ค่าที่เป็นลบน้อยกว่า (มีค่าเข้าใกล้ 0 มากกว่า หรือค่าทางคณิตศาสตร์มากกว่า เช่น \(-45 > -80\)) แสดงถึงกำลังของคลื่นวิทยุที่ภาครับของ ESP32 ได้รับมีความเข้มสูงกว่า มีอัตราส่วนสัญญาณต่อสัญญาณรบกวน (Signal-to-Noise Ratio: SNR) ที่ดีกว่า ทำให้การเชื่อมต่อมีความเสถียรและโอกาสเกิด Packet Loss ต่ำกว่า

1. **เหตุใดการดึงค่า `authmode` (`wifi_auth_mode_t`) จากโครงสร้าง `wifi_ap_record_t` จึงมีความสำคัญต่อการเตรียมการในเฟสถัดไป (Authentication & Association Phase)?
-**Ans**
   - **ความสำคัญ:** เนื่องจากในเฟส Authentication และ 4-Way Handshake ตัว ESP32 จำเป็นต้องทราบประเภทระบบความปลอดภัยของ AP เป้าหมายล่วงหน้า (เช่น `WIFI_AUTH_OPEN`, `WIFI_AUTH_WPA2_PSK`, หรือ `WIFI_AUTH_WPA2_ENTERPRISE`) เพื่อ:
     1. กำหนดโครงสร้าง `wifi_config_t` และเลือกโปรโตคอลการยืนยันตัวตนให้ตรงกับ AP
     2. ทราบว่าต้องเตรียม Password/PSK หรือข้อมูลยืนยันตัวตน Enterprise เพื่อใช้สร้างคีย์เข้ารหัส (PTK/GTK) ในขั้นตอน 4-Way Handshake หรือไม่ หากค่า `authmode` ไม่ตรงกัน จะทำให้การเชื่อมต่อล้มเหลวตั้งแต่ขั้นตอน Auth/Assoc ทันที

## Log
```
I (27) boot: ESP-IDF v6.0.2 2nd stage bootloader
I (27) boot: compile time Aug  3 2026 09:17:25
I (28) boot: Multicore bootloader
I (29) boot: chip revision: v3.1
I (32) boot.esp32: SPI Speed      : 40MHz
I (35) boot.esp32: SPI Mode       : DIO
I (39) boot.esp32: SPI Flash Size : 2MB
I (42) boot: Enabling RNG early entropy source...
I (47) boot: Partition Table:
I (49) boot: ## Label            Usage          Type ST Offset   Length
I (56) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (62) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (69) boot:  2 factory          factory app      00 00 00010000 00100000
I (75) boot: End of partition table
I (79) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=1a40ch (107532) map
I (124) esp_image: segment 1: paddr=0002a434 vaddr=3ffb0000 size=04528h ( 17704) load
I (132) esp_image: segment 2: paddr=0002e964 vaddr=40080000 size=016b4h (  5812) load
I (134) esp_image: segment 3: paddr=00030020 vaddr=400d0020 size=85fa4h (548772) map
I (331) esp_image: segment 4: paddr=000b5fcc vaddr=400816b4 size=13f54h ( 81748) load
I (365) esp_image: segment 5: paddr=000c9f28 vaddr=50000000 size=00028h (    40) load
I (376) boot: Loaded app from partition at offset 0x10000
I (376) boot: Disabling RNG early entropy source...
I (387) cpu_start: Multicore app
I (395) cpu_start: GPIO 3 and 1 are used as console UART I/O pins
I (395) cpu_start: Pro cpu start user code
I (395) cpu_start: cpu freq: 160000000 Hz
I (397) app_init: Application information:
I (401) app_init: Project name:     wifi_scan_phase
I (406) app_init: App version:      37de524-dirty
I (410) app_init: Compile time:     Aug  3 2026 09:17:21
I (415) app_init: ELF file SHA256:  a81a36b9f...
I (419) app_init: ESP-IDF:          v6.0.2
I (423) efuse_init: Min chip rev:     v0.0
I (427) efuse_init: Max chip rev:     v3.99 
I (431) efuse_init: Chip rev:         v3.1
I (435) heap_init: Initializing. RAM available for dynamic allocation:
I (441) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (446) heap_init: At 3FFB8A20 len 000275E0 (157 KiB): DRAM
I (452) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (457) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (462) heap_init: At 40095608 len 0000A9F8 (42 KiB): IRAM
W (469) spi_flash: Detected boya flash chip but using generic driver. For optimal functionality, enable `SPI_FLASH_SUPPORT_BOYA_CHIP` in menuconfig
I (481) spi_flash: detected chip: generic
I (484) spi_flash: flash io: dio
W (487) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (501) main_task: Started on CPU0
I (501) main_task: Calling app_main()
I (501) LAB_WIFI_SCAN: [FORENSIC]: Call nvs_flash_init()
I (541) LAB_WIFI_SCAN: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (541) LAB_WIFI_SCAN: [FORENSIC]: Call esp_netif_init()
I (541) LAB_WIFI_SCAN: [FORENSIC]: esp_netif_init() returned ESP_OK (0x0)
I (551) LAB_WIFI_SCAN: [FORENSIC]: Call esp_event_loop_create_default()
I (551) LAB_WIFI_SCAN: [FORENSIC]: esp_event_loop_create_default() returned ESP_OK (0x0)
I (561) LAB_WIFI_SCAN: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (571) LAB_WIFI_SCAN: [FORENSIC]: esp_netif_create_default_wifi_sta() returned pointer 0x3ffbddb8
I (581) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_init(&cfg)
I (591) wifi:wifi driver task: 3ffc04a4, prio:23, stack:6656, core=0
I (601) wifi:wifi firmware version: 00ad238
I (601) wifi:wifi certification version: v7.0
I (611) wifi:config NVS flash: enabled
I (611) wifi:config nano formatting: disabled
I (611) wifi:Init data frame dynamic rx buffer num: 32
I (611) wifi:Init static rx mgmt buffer num: 5
I (621) wifi:Init management short buffer num: 32
I (621) wifi:Init dynamic tx buffer num: 32
I (631) wifi:Init static rx buffer size: 1600
I (631) wifi:Init static rx buffer num: 10
I (631) wifi:Init dynamic rx buffer num: 32
I (641) wifi_init: rx ba win: 6
I (641) wifi_init: accept mbox: 6
I (641) wifi_init: tcpip mbox: 32
I (651) wifi_init: udp mbox: 6
I (651) wifi_init: tcp mbox: 6
I (651) wifi_init: tcp tx win: 5760
I (661) wifi_init: tcp rx win: 5760
I (661) wifi_init: tcp mss: 1440
I (661) wifi_init: WiFi IRAM OP enabled
I (671) wifi_init: WiFi RX IRAM OP enabled
I (671) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_init() returned ESP_OK (0x0)
I (681) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (681) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_set_mode() returned ESP_OK (0x0)
I (691) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_start()
I (691) phy_init: phy_version 4863,a3a4459,Oct 28 2025,14:30:06
I (781) wifi:mode : sta (88:57:21:ae:44:84)
I (791) wifi:enable tsf
I (791) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (791) LAB_WIFI_SCAN: ==================================================================
I (791) LAB_WIFI_SCAN:   Lab 5.1: Wi-Fi Connection and Scanning Phase (ESP-IDF Forensic)
I (801) LAB_WIFI_SCAN: ==================================================================
I (811) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (821) LAB_WIFI_SCAN: >>> Experiment 5.1.1: General AP Scan (All Channels)
I (821) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (831) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_start(scan_config, block=true)
I (2241) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (0x0) [Duration: 1395 ms]
I (2241) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (2241) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_num() returned ESP_OK (0x0), ap_count=7
I (2251) LAB_WIFI_SCAN: [STATUS]: Scan SUCCESS
I (2251) LAB_WIFI_SCAN: [AP COUNT]: 7 network(s) found
I (2261) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_records(&number, ap_info)
I (2271) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_records() returned ESP_OK (0x0), records=7

--------------------------------------------------------------------------------------------------
No.  | SSID                     | MAC Address (BSSID) | RSSI   | Chan | Encryption Type     
--------------------------------------------------------------------------------------------------
1    | KMITL-Legacy             | 78:17:BE:C0:7D:A0 | -53  dBm | 1    | WPA2_ENTERPRISE     
2    | KMITL-IoT                | 78:17:BE:C0:7D:A2 | -53  dBm | 1    | WPA2_PSK            
3    | KMITL-WIFI               | 78:17:BE:C0:7D:A1 | -58  dBm | 1    | OPEN (No Password)  
4    | KMITL-Legacy             | 78:17:BE:C0:66:20 | -85  dBm | 1    | WPA2_ENTERPRISE     
5    | KMITL-IoT                | 78:17:BE:C0:66:22 | -85  dBm | 1    | WPA2_PSK            
6    | KMITL-IoT                | 78:17:BE:A9:94:E2 | -85  dBm | 6    | WPA2_PSK            
7    | KMITL-WIFI               | 78:17:BE:C0:66:21 | -86  dBm | 1    | OPEN (No Password)  
--------------------------------------------------------------------------------------------------

I (3371) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (3371) LAB_WIFI_SCAN: >>> Experiment 5.1.2: Channel-Specific Scan (Channel 1)
I (3371) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (3381) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_start(scan_config, block=true)
I (3591) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (0x0) [Duration: 200 ms]
I (3591) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (3591) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_num() returned ESP_OK (0x0), ap_count=6
I (3601) LAB_WIFI_SCAN: [STATUS]: Scan SUCCESS
I (3601) LAB_WIFI_SCAN: [AP COUNT]: 6 network(s) found
I (3611) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_records(&number, ap_info)
I (3621) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_records() returned ESP_OK (0x0), records=6

--------------------------------------------------------------------------------------------------
No.  | SSID                     | MAC Address (BSSID) | RSSI   | Chan | Encryption Type     
--------------------------------------------------------------------------------------------------
1    | KMITL-Legacy             | 78:17:BE:C0:7D:A0 | -55  dBm | 1    | WPA2_ENTERPRISE     
2    | KMITL-IoT                | 78:17:BE:C0:7D:A2 | -55  dBm | 1    | WPA2_PSK            
3    | KMITL-WIFI               | 78:17:BE:C0:7D:A1 | -59  dBm | 1    | OPEN (No Password)  
4    | KMITL-Legacy             | 78:17:BE:C0:66:20 | -84  dBm | 1    | WPA2_ENTERPRISE     
5    | KMITL-IoT                | 78:17:BE:C0:66:22 | -85  dBm | 1    | WPA2_PSK            
6    | KMITL-WIFI               | 78:17:BE:C0:66:21 | -86  dBm | 1    | OPEN (No Password)  
--------------------------------------------------------------------------------------------------

I (4711) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (4711) LAB_WIFI_SCAN: >>> Experiment 5.1.3: Targeted SSID Scan - Existing ("KMITL-Legacy")
I (4711) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (4721) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_start(scan_config, block=true)
I (6131) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (0x0) [Duration: 1399 ms]
I (6131) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (6131) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_num() returned ESP_OK (0x0), ap_count=2
I (6141) LAB_WIFI_SCAN: [STATUS]: Scan SUCCESS
I (6141) LAB_WIFI_SCAN: [AP COUNT]: 2 network(s) found
I (6151) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_records(&number, ap_info)
I (6161) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_records() returned ESP_OK (0x0), records=2

--------------------------------------------------------------------------------------------------
No.  | SSID                     | MAC Address (BSSID) | RSSI   | Chan | Encryption Type     
--------------------------------------------------------------------------------------------------
1    | KMITL-Legacy             | 78:17:BE:C0:7D:A0 | -56  dBm | 1    | WPA2_ENTERPRISE     
2    | KMITL-Legacy             | 78:17:BE:C0:72:60 | -77  dBm | 11   | WPA2_ENTERPRISE     
--------------------------------------------------------------------------------------------------

I (7211) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (7211) LAB_WIFI_SCAN: >>> Experiment 5.1.4: Targeted SSID Scan - Non-Existent ("NON_EXISTENT_AP_9999")
I (7211) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (7221) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_start(scan_config, block=true)
I (8631) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (0x0) [Duration: 1398 ms]
I (8631) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (8631) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_num() returned ESP_OK (0x0), ap_count=0
I (8641) LAB_WIFI_SCAN: [STATUS]: Scan SUCCESS
I (8641) LAB_WIFI_SCAN: [AP COUNT]: 0 network(s) found
W (8651) LAB_WIFI_SCAN: [NOTE]: No Access Point found matching the criteria.
I (8651) LAB_WIFI_SCAN: ==================================================================
I (8661) LAB_WIFI_SCAN:   [Phase 1 Completed: Wi-Fi Scan Finished]
I (8671) LAB_WIFI_SCAN:   Program stopped after scanning. Auth/Assoc Phase not started.
I (8681) LAB_WIFI_SCAN: ==================================================================
I (8681) main_task: Returned from app_main()
```

---
# Lab5-2-Wi-Fi-Phase2
## 6. ตารางบันทึกผลการทดลอง (Experiment Results)

ให้นักศึกษาบันทึกผลลัพธ์จากการสังเกตใน Serial Console ลงในตารางต่อไปนี้:

### 6.1 ตารางสรุปเปรียบเทียบผลการทดลองทั้ง 3 สถานการณ์

| ข้อการทดลอง | สถานการณ์ทดสอบ                     |    Event สุดท้ายที่ได้รับ     | ผลลัพธ์ (Passed/Failed) | Reason Code (Decimal / Hex) | คำอธิบาย Reason Code                                                     |
| :---------: | :--------------------------------- | :---------------------------: | :---------------------: | :-------------------------: | :----------------------------------------------------------------------- |
|  **5.2.1**  | SSID และ Password ถูกต้อง          |     `IP_EVENT_STA_GOT_IP`     |         Passed          |              -              | เชื่อมต่อสำเร็จและได้รับ IP Address จาก DHCP Server                      |
|  **5.2.2**  | ระบุ SSID ผิด (ไม่มีในระบบ)        | `WIFI_EVENT_STA_DISCONNECTED` |         Failed          |         201 / 0xC9          | WIFI_REASON_NO_AP_FOUND (ไม่พบ Access Point ที่ตรงกับ SSID)              |
|  **5.2.3**  | ระบุ SSID ถูกต้อง แต่ Password ผิด | `WIFI_EVENT_STA_DISCONNECTED` |         Failed          |         202 / 0xCA          | WIFI_REASON_AUTH_FAIL (การยืนยันตัวตนล้มเหลวเนื่องจากรหัสผ่านไม่ถูกต้อง) |

### 6.2 บันทึกข้อมูลเครือข่ายจากการเชื่อมต่อสำเร็จ (ข้อ 5.2.1)

| พารามิเตอร์เครือข่าย | ค่าที่ได้รับจริงจาก DHCP |
| :--- | :--- |
| **SSID** | Test-WiFi |
| **BSSID (MAC Address)** | 0A:8A:B4:BB:12:61 |
| **Channel** | 6 |
| **IP Address** | 10.20.17.118 |
| **Subnet Mask** | 255.255.255.0 |
| **Default Gateway** | 10.20.17.253 |

---

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **เหตุใดการระบุ SSID ผิด (ข้อ 5.2.2) จึงส่งผลให้เกิด Disconnect Event ด้วย Reason Code `201` (`WIFI_REASON_NO_AP_FOUND`) ตั้งแต่เฟส Scan?**
- **Ans**
	- ในขั้นตอนการทำงานของ Wi-Fi Station เมื่อเรียกฟังก์ชัน `esp_wifi_connect()` ชิป ESP32 จะต้องเริ่มกระบวนการค้นหา (Scanning) ช่องสัญญาณ 1–13 เพื่อตรวจหา Beacon Frame หรือ Probe Response จาก Access Point ที่มีชื่อ SSID และระดับความปลอดภัยตรงตามที่กำหนดไว้ใน `wifi_config_t` ก่อนเป็นลำดับแรก (Phase 1: Scan Phase) 
	- หากสแกนครบทุกช่องความถี่แล้วไม่พบ AP ที่มีชื่อตรงกัน ระบบ Wi-Fi Driver จะไม่สามารถเริ่มขั้นตอนส่งเฟรม Authentication หรือ Association (Phase 2 & 3) ได้ และจะสร้าง Event `WIFI_EVENT_STA_DISCONNECTED` ทันที พร้อมแนบ Reason Code `201` (`WIFI_REASON_NO_AP_FOUND`) เพื่อแจ้งให้แอปพลิเคชันทราบว่าไม่พบเครือข่ายเป้าหมาย

2. **เหตุใดการพิมพ์ Password ผิด (ข้อ 5.2.3) จึงผ่านเฟส Auth และ Assoc มาได้ แต่มาล้มเหลวในเฟส 4-Way Handshake (Reason Code `15` หรือ `204`)? (หรือ Reason Code `202` ใน WPA3-SAE)**
- **Ans**
	- **ในระบบ WPA2-PSK (Legacy):** การยืนยันตัวตนในระดับ 802.11 Link-Layer (Phase 2 & 3: Open System Authentication และ Association) เป็นเพียงการจับคู่ระดับฮาร์ดแวร์เพื่อสร้าง Session เท่านั้น โดยยังไม่มีการตรวจสอบรหัสผ่าน (Pre-Shared Key: PSK) จนกระทั่งเข้าสู่ Phase 4: EAPOL 4-Way Handshake ตัว AP และ ESP32 จะนำรหัสผ่านไปคำนวณสร้าง Pairwise Master Key (PMK) และ Pairwise Transient Key (PTK) หากรหัสผ่านผิด ค่า Message Integrity Code (MIC) ในข้อความ Handshake จะไม่ตรงกัน ทำให้เกิด Timeout หรือปฏิเสธการเชื่อมต่อด้วย Reason Code `15` / `204` (`WIFI_REASON_4WAY_HANDSHAKE_TIMEOUT` / `WIFI_REASON_HANDSHAKE_TIMEOUT`)
	- **ในระบบ WPA3-SAE (ผลการทดลองจริง):** กระบวนการตรวจสอบรหัสผ่านจะถูกย้ายขึ้นมาทำตั้งแต่ขั้นตอน **Simultaneous Authentication of Equals (SAE)** ใน Authentication Phase (Phase 2) โดยใช้รหัสผ่านสร้างรหัสลับร่วม (Commit & Confirm) หากรหัสผ่านไม่ถูกต้อง การยืนยันตัวตน SAE จะล้มเหลวทันที ส่งผลให้เกิด Disconnect Event ด้วย Reason Code `202` (`WIFI_REASON_AUTH_FAIL`) ตั้งแต่จบเฟส Auth โดยไม่สามารถไปต่อยังเฟส Assoc ได้

3. **ลำดับการเกิด Event ระหว่าง `WIFI_EVENT_STA_CONNECTED` กับ `IP_EVENT_STA_GOT_IP` Event ใดเกิดขึ้นก่อนกัน และมีความหมายทางกายภาพของ Layer Network ต่างกันอย่างไร?**
- **Ans**
	- **ลำดับการเกิด:** **`WIFI_EVENT_STA_CONNECTED` เกิดขึ้นก่อน `IP_EVENT_STA_GOT_IP` เสมอ**
	- **ความหมายทาง Layer Network:**
		- **`WIFI_EVENT_STA_CONNECTED` (OSI Layer 2 - Data Link Layer):** บ่งบอกว่า ESP32 เชื่อมต่อกับ Access Point ในระดับสัญญาณวิทยุ 802.11 สำเร็จเรียบร้อยแล้ว (ผ่านทั้ง Scan, Auth, Assoc และ 4-Way Handshake) ได้รับ Association ID (AID) และช่องทางการส่งเฟรมข้อมูลเปิดใช้งานแล้ว แต่ **ยังไม่มี IP Address** จึงยังไม่สามารถส่งแพ็กเก็ต TCP/IP หรือเข้าสู่อินเทอร์เน็ตได้
		- **`IP_EVENT_STA_GOT_IP` (OSI Layer 3 - Network Layer):** บ่งบอกว่าโปรโตคอลสแต็ก TCP/IP (LwIP) ของ ESP32 ได้ส่ง DHCP Request ข้าม Layer 2 ไปยัง DHCP Server บน Router และได้รับค่าพารามิเตอร์เครือข่าย (IP Address, Subnet Mask, Gateway, DNS) มาจัดสรรให้อินเทอร์เฟซเรียบร้อยแล้ว ณ จุดนี้ ESP32 จึงพร้อมสำหรับการสื่อสารผ่านโปรโตคอลเครือข่าย เช่น HTTP, MQTT, Socket

4. **สมาชิกตัวแปร `reason` ในโครงสร้าง `wifi_event_sta_disconnected_t` มีประโยชน์อย่างไรต่อการออกแบบระบบค้นหาสาเหตุและกู้คืนการเชื่อมต่อ (Auto-Reconnection Mechanism) ในแอปพลิเคชัน IoT?**
- **Ans**
	- ตัวแปร `reason` (ประเภท `uint8_t` หรือ `wifi_err_reason_t`) ให้ข้อมูลที่เฉพาะเจาะจงเกี่ยวกับสาเหตุที่ทำให้การเชื่อมต่อหลุด ซึ่งมีประโยชน์อย่างยิ่งในการตัดสินใจเชิงตรรกะสำหรับการกู้คืนระบบ (State Machine / Retry Strategy):
		- **กรณีสาเหตุชั่วคราว (Transient Errors):** เช่น `WIFI_REASON_BEACON_TIMEOUT (200)`, `WIFI_REASON_CONNECTION_FAIL (205)` หรือ `WIFI_REASON_AUTH_EXPIRE (2)` (เกิดจากสัญญาณอ่อนชั่วคราว) $\rightarrow$ ระบบควรสั่ง **Exponential Backoff Reconnect** (ลองเชื่อมต่อใหม่โดยค่อยๆ เพิ่มระยะเวลาหน่วง เช่น 1s, 2s, 4s, 8s เพื่อลดภาระเครือข่าย)
		- **กรณี AP ปิดตัวหรืออยู่นอกระยะ:** เช่น `WIFI_REASON_NO_AP_FOUND (201)` $\rightarrow$ สั่งสลับไปสแกนหา Access Point สำรอง (Fallback / Secondary AP) หรือเปิดโหมด AP Config (Captive Portal)
		- **กรณีข้อผิดพลาดถาวร (Fatal Configuration Errors):** เช่น `WIFI_REASON_AUTH_FAIL (202)` หรือ `WIFI_REASON_HANDSHAKE_TIMEOUT (204)` (รหัสผ่านผิด) $\rightarrow$ ระบบควรงดการ Retry ซ้ำๆ ทันทีเพื่อไม่ให้สิ้นเปลืองพลังงาน และส่งแจ้งเตือน Error ไปยังผู้ใช้งาน หรือเข้าสู่ Wi-Fi Provisioning Mode เพื่อรอรับรหัสผ่านใหม่

---

## 8. Forensic Log
```
I (27) boot: ESP-IDF v6.0.2 2nd stage bootloader
I (27) boot: compile time Aug  3 2026 10:05:36
I (28) boot: Multicore bootloader
I (29) boot: chip revision: v3.1
I (32) boot.esp32: SPI Speed      : 40MHz
I (35) boot.esp32: SPI Mode       : DIO
I (39) boot.esp32: SPI Flash Size : 2MB
I (42) boot: Enabling RNG early entropy source...
I (47) boot: Partition Table:
I (49) boot: ## Label            Usage          Type ST Offset   Length
I (56) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (62) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (69) boot:  2 factory          factory app      00 00 00010000 00100000
I (75) boot: End of partition table
I (79) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=1a628h (108072) map
I (125) esp_image: segment 1: paddr=0002a650 vaddr=3ffb0000 size=04528h ( 17704) load
I (132) esp_image: segment 2: paddr=0002eb80 vaddr=40080000 size=01498h (  5272) load
I (134) esp_image: segment 3: paddr=00030020 vaddr=400d0020 size=86324h (549668) map
I (332) esp_image: segment 4: paddr=000b634c vaddr=40081498 size=14170h ( 82288) load
I (366) esp_image: segment 5: paddr=000ca4c4 vaddr=50000000 size=00028h (    40) load
I (377) boot: Loaded app from partition at offset 0x10000
I (377) boot: Disabling RNG early entropy source...
I (388) cpu_start: Multicore app
I (396) cpu_start: GPIO 3 and 1 are used as console UART I/O pins
I (396) cpu_start: Pro cpu start user code
I (396) cpu_start: cpu freq: 160000000 Hz
I (398) app_init: Application information:
I (402) app_init: Project name:     wifi_connection_phase
I (407) app_init: App version:      3707e88-dirty
I (411) app_init: Compile time:     Aug  3 2026 10:09:34
I (416) app_init: ELF file SHA256:  b5bacdcbc...
I (421) app_init: ESP-IDF:          v6.0.2
I (424) efuse_init: Min chip rev:     v0.0
I (428) efuse_init: Max chip rev:     v3.99 
I (432) efuse_init: Chip rev:         v3.1
I (436) heap_init: Initializing. RAM available for dynamic allocation:
I (443) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (447) heap_init: At 3FFB8A40 len 000275C0 (157 KiB): DRAM
I (453) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (458) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (464) heap_init: At 40095608 len 0000A9F8 (42 KiB): IRAM
W (470) spi_flash: Detected boya flash chip but using generic driver. For optimal functionality, enable `SPI_FLASH_SUPPORT_BOYA_CHIP` in menuconfig
I (482) spi_flash: detected chip: generic
I (485) spi_flash: flash io: dio
W (488) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (502) main_task: Started on CPU0
I (502) main_task: Calling app_main()
I (502) LAB_WIFI_CONN: [FORENSIC]: Call nvs_flash_init()
I (542) LAB_WIFI_CONN: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (542) LAB_WIFI_CONN: [FORENSIC]: Call esp_netif_init()
I (542) LAB_WIFI_CONN: [FORENSIC]: Call esp_event_loop_create_default()
I (552) LAB_WIFI_CONN: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (562) LAB_WIFI_CONN: [FORENSIC]: esp_netif_create_default_wifi_sta() returned 0x3ffbddfc
I (562) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_init(&cfg)
I (582) wifi:wifi driver task: 3ffc04e8, prio:23, stack:6656, core=0
I (592) wifi:wifi firmware version: 00ad238
I (592) wifi:wifi certification version: v7.0
I (592) wifi:config NVS flash: enabled
I (592) wifi:config nano formatting: disabled
I (602) wifi:Init data frame dynamic rx buffer num: 32
I (602) wifi:Init static rx mgmt buffer num: 5
I (612) wifi:Init management short buffer num: 32
I (612) wifi:Init dynamic tx buffer num: 32
I (612) wifi:Init static rx buffer size: 1600
I (622) wifi:Init static rx buffer num: 10
I (622) wifi:Init dynamic rx buffer num: 32
I (632) wifi_init: rx ba win: 6
I (632) wifi_init: accept mbox: 6
I (632) wifi_init: tcpip mbox: 32
I (632) wifi_init: udp mbox: 6
I (642) wifi_init: tcp mbox: 6
I (642) wifi_init: tcp tx win: 5760
I (642) wifi_init: tcp rx win: 5760
I (652) wifi_init: tcp mss: 1440
I (652) wifi_init: WiFi IRAM OP enabled
I (652) wifi_init: WiFi RX IRAM OP enabled
I (662) LAB_WIFI_CONN: [FORENSIC]: Call esp_event_handler_instance_register(WIFI_EVENT)
I (672) LAB_WIFI_CONN: [FORENSIC]: Call esp_event_handler_instance_register(IP_EVENT)
I (672) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (682) LAB_WIFI_CONN: ==================================================================
I (692) LAB_WIFI_CONN:   Lab 5.2: Wi-Fi Connection & IP Assignment (ESP-IDF Forensic)
I (692) LAB_WIFI_CONN: ==================================================================
I (702) LAB_WIFI_CONN: 

I (702) LAB_WIFI_CONN: ------------------------------------------------------------------
I (712) LAB_WIFI_CONN: >>> Experiment 5.2.1: Connection Test - Correct Credentials
I (722) LAB_WIFI_CONN: ------------------------------------------------------------------
I (732) LAB_WIFI_CONN:   Target SSID: "Test-WiFi"
I (732) LAB_WIFI_CONN:   Target Password: "0954276527"
I (742) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_stop()
I (742) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (752) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (782) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (782) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_start()
I (782) phy_init: phy_version 4863,a3a4459,Oct 28 2025,14:30:06
I (872) wifi:mode : sta (88:57:21:ae:44:84)
I (872) wifi:enable tsf
I (872) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (882) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (882) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (892) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_connect()
I (902) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (1632) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1, snd_ch_cfg:0x0
I (1632) wifi:state: init -> auth (0xb0)
I (1932) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (2432) wifi:state: auth -> assoc (0x0)
I (2442) wifi:state: assoc -> run (0x10)
I (2482) wifi:connected with Test-WiFi, aid = 1, channel 6, BW20, bssid = 0a:8a:b4:bb:12:61
I (2482) wifi:security: WPA3-SAE HUNT_AND_PECK, phy: bgn, rssi: -54, cipher(pairwise:0x3, group:0x3), pmf:1
I (2502) wifi:pm start, type: 1

I (2502) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us
I (2512) wifi:dp: 2, bi: 102400, li: 4, scale listen interval from 307200 us to 409600 us
I (2512) wifi:AP's beacon interval = 102400 us, DTIM period = 2
I (2522) LAB_WIFI_CONN: =======================================================
I (2522) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_CONNECTED received!
I (2532) LAB_WIFI_CONN:   -> Connected to SSID : Test-WiFi
I (2532) LAB_WIFI_CONN:   -> BSSID            : 0A:8A:B4:BB:12:61
I (2542) LAB_WIFI_CONN:   -> Channel          : 6
I (2542) LAB_WIFI_CONN:   -> Auth Mode        : 6
I (2552) LAB_WIFI_CONN: =======================================================
I (2522) wifi:<ba-add>idx:0 (ifx:0, 0a:8a:b4:bb:12:61), tid:0, ssn:0, winSize:64
I (3592) esp_netif_handlers: sta ip: 10.20.17.118, mask: 255.255.255.0, gw: 10.20.17.253
I (3592) LAB_WIFI_CONN: =======================================================
I (3592) LAB_WIFI_CONN: [EVENT FORENSIC]: IP_EVENT_STA_GOT_IP received!
I (3602) LAB_WIFI_CONN:   -> IP Address : 10.20.17.118
I (3602) LAB_WIFI_CONN:   -> Netmask    : 255.255.255.0
I (3612) LAB_WIFI_CONN:   -> Gateway    : 10.20.17.253
I (3612) LAB_WIFI_CONN: =======================================================
I (3622) LAB_WIFI_CONN: [RESULT]: TEST PASSED - Connected to AP successfully!
I (5622) LAB_WIFI_CONN: 

I (5622) LAB_WIFI_CONN: ------------------------------------------------------------------
I (5622) LAB_WIFI_CONN: >>> Experiment 5.2.2: Connection Test - Wrong SSID (No AP Found)
I (5622) LAB_WIFI_CONN: ------------------------------------------------------------------
I (5632) LAB_WIFI_CONN:   Target SSID: "NON_EXISTENT_SSID_9999"
I (5642) LAB_WIFI_CONN:   Target Password: "12345678"
I (5642) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_stop()
I (5652) wifi:state: run -> init (0x0)
I (5662) wifi:pm stop, total sleep time: 621374 us / 3156259 us

I (5662) wifi:<ba-del>idx:0, tid:0
W (5662) LAB_WIFI_CONN: =======================================================
W (5672) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (5672) LAB_WIFI_CONN:   -> Target SSID          : Test-WiFi
W (5682) LAB_WIFI_CONN:   -> Reason Code (Decimal): 8
W (5682) LAB_WIFI_CONN:   -> Reason Code (Hex)    : 0x08
W (5692) LAB_WIFI_CONN:   -> Reason Description   : OTHER_DISCONNECT_REASON
W (5692) LAB_WIFI_CONN: =======================================================
I (5702) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (5712) wifi:flush txq
I (5712) wifi:stop sw txq
I (5712) wifi:lmac stop hw txq
I (5722) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (5722) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (5752) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (5752) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_start()
I (5762) wifi:mode : sta (88:57:21:ae:44:84)
I (5762) wifi:enable tsf
I (5762) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (5772) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (5772) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (5782) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_connect()
I (5792) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
W (5792) LAB_WIFI_CONN: [RESULT]: TEST FAILED - Disconnected event captured.
W (7122) LAB_WIFI_CONN: =======================================================
W (7122) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (7122) LAB_WIFI_CONN:   -> Target SSID          : NON_EXISTENT_SSID_9999
W (7132) LAB_WIFI_CONN:   -> Reason Code (Decimal): 201
W (7142) LAB_WIFI_CONN:   -> Reason Code (Hex)    : 0xC9
W (7142) LAB_WIFI_CONN:   -> Reason Description   : WIFI_REASON_NO_AP_FOUND (201)
W (7152) LAB_WIFI_CONN: =======================================================
I (7802) LAB_WIFI_CONN: 

I (7802) LAB_WIFI_CONN: ------------------------------------------------------------------
I (7802) LAB_WIFI_CONN: >>> Experiment 5.2.3: Connection Test - Wrong Password (Auth/Handshake Fail)
I (7802) LAB_WIFI_CONN: ------------------------------------------------------------------
I (7812) LAB_WIFI_CONN:   Target SSID: "Test-WiFi"
I (7822) LAB_WIFI_CONN:   Target Password: "WRONG_PASS_9999"
I (7822) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_stop()
I (7832) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (7832) wifi:flush txq
I (7832) wifi:stop sw txq
I (7842) wifi:lmac stop hw txq
I (7842) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (7852) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (7922) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (7922) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_start()
I (7922) wifi:mode : sta (88:57:21:ae:44:84)
I (7922) wifi:enable tsf
I (7932) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (7932) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_connect()
I (7942) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (7932) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (7952) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1, snd_ch_cfg:0x0
I (7962) wifi:state: init -> auth (0xb0)
I (7962) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (8542) wifi:state: auth -> init (0x600)
W (8552) LAB_WIFI_CONN: =======================================================
W (8552) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (8562) LAB_WIFI_CONN:   -> Target SSID          : Test-WiFi
W (8562) LAB_WIFI_CONN:   -> Reason Code (Decimal): 202
W (8572) LAB_WIFI_CONN:   -> Reason Code (Hex)    : 0xCA
W (8572) LAB_WIFI_CONN:   -> Reason Description   : WIFI_REASON_AUTH_FAIL (202)
W (8582) LAB_WIFI_CONN: =======================================================
W (8592) LAB_WIFI_CONN: [RESULT]: TEST FAILED - Disconnected event captured.
I (8592) LAB_WIFI_CONN: ==================================================================
I (8602) LAB_WIFI_CONN:   [Phase 2/3/4/5 Completed: Wi-Fi Connection Lab Finished]
I (8612) LAB_WIFI_CONN: ==================================================================
I (8622) main_task: Returned from app_main()
```