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