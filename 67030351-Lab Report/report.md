# 07-Labsheet-05-1-Wi-Fi-Scan-Phase
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

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **การกำหนดค่าในโครงสร้าง `wifi_scan_config_t` สำหรับสแกนเจาะจงเฉพาะช่องความถี่ (ข้อ 5.1.2) ช่วยลดเวลาในการสแกนเมื่อเทียบกับการสแกนทุกช่องความถี่ (ข้อ 5.1.1) อย่างไร และมีข้อจำกัดอย่างไร?**
- **Ans**
	- **การลดเวลาในการสแกน:** เมื่อกำหนด `.channel = 1` ตัวชิป Wi-Fi ของ ESP32 จะส่ง Probe Request และดักฟังสัญญาณอยู่บน Channel 1 เพียงช่องเดียว โดยไม่ต้องสลับความถี่ (Channel Switching) และไม่ต้องรอเวลา Scan Time ประจำแต่ละช่องสัญญาณครบทั้ง 13 ช่อง (Channels 1–13) ส่งผลให้เวลาสแกนลดลงอย่างมาก จาก **1,395 ms** เหลือเพียง **200 ms** (เร็วขึ้นเกือบ 7 เท่า)
	- **ข้อจำกัด:** จะค้นพบเฉพาะ Access Point ที่กระจายสัญญาณอยู่บนช่องความถี่ที่กำหนดเท่านั้น หากมี AP อื่นๆ อยู่บน Channel อื่น (เช่น Channel 6 หรือ 11) หรือหากไม่ทราบ Channel ที่แน่นอนของ AP เป้าหมาย จะไม่สามารถค้นพบได้

2. **เมื่อสังเกตผล Forensic Log ในข้อ 5.1.4 (สแกนหา SSID ที่ไม่มีอยู่จริง) ฟังก์ชัน `esp_wifi_scan_start()`, `esp_wifi_scan_get_ap_num()` และ `esp_wifi_scan_get_ap_records()` ส่งคืนค่าอย่างไร?
- **Ans**
   - **`esp_wifi_scan_start()`:** ส่งคืนค่า **`ESP_OK (0x0)`** เนื่องจากกระบวนการสแกนคลื่นวิทยุทำงานสำเร็จสมบูรณ์ตามคำสั่ง ไม่เกิดข้อผิดพลาดระดับฮาร์ดแวร์/ไดรเวอร์
   - **`esp_wifi_scan_get_ap_num()`:** ส่งคืนค่า **`ESP_OK (0x0)`** และคืนค่าตัวแปร `ap_count = 0` ซึ่งแสดงว่าไม่พบ AP ที่ตรงกับ SSID ที่ค้นหา
   - **`esp_wifi_scan_get_ap_records()`:** ในโปรแกรมมีเงื่อนไขตรวจสอบ `if (ap_count == 0)` จึงทำให้ฟังก์ชันนี้ **ไม่ถูกเรียกใช้งาน (Skipped)** เนื่องจากไม่มีรายการ AP ให้ดึงข้อมูลมาแสดงผล

3. **ค่าระดับความแรงสัญญาณ (RSSI) ที่แสดงเป็นตัวเลขติดลบ (เช่น -45 dBm กับ -80 dBm) ค่าใดแสดงถึงสัญญาณที่มีความแรงและความเสถียรมากกว่ากัน?
- **Ans**
   - ค่า **-45 dBm** มีความแรงและความเสถียรมากกว่า **-80 dBm**
   - **เหตุผล:** RSSI มีหน่วยเป็น dBm ซึ่งเป็นค่าลอการิทึมเทียบกับกำลังงาน 1 มิลลิวัตต์ ค่าที่เป็นลบน้อยกว่า (มีค่าเข้าใกล้ 0 มากกว่า หรือค่าทางคณิตศาสตร์มากกว่า เช่น \(-45 > -80\)) แสดงถึงกำลังของคลื่นวิทยุที่ภาครับของ ESP32 ได้รับมีความเข้มสูงกว่า มีอัตราส่วนสัญญาณต่อสัญญาณรบกวน (Signal-to-Noise Ratio: SNR) ที่ดีกว่า ทำให้การเชื่อมต่อมีความเสถียรและโอกาสเกิด Packet Loss ต่ำกว่า

4. เหตุใดการดึงค่า `authmode` (`wifi_auth_mode_t`) จากโครงสร้าง `wifi_ap_record_t` จึงมีความสำคัญต่อการเตรียมการในเฟสถัดไป (Authentication & Association Phase)?
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
# 08-Labsheet-05-2-Wi-Fi-Connection-Phase
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

## Log
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
--- 
# 09-Labsheet-05-3-Wi-Fi-Auth-Assoc-Phase
## 6. ตารางบันทึกผลการทดลอง (Experiment Results)

ให้นักศึกษาบันทึกผลลัพธ์จากการสังเกตใน Serial Console ลงในตารางต่อไปนี้:

### 6.1 ตารางสรุปเปรียบเทียบผลการทดลองในระดับ Link Layer

| ข้อการทดลอง | สถานการณ์ทดสอบ                           | Event ที่ได้รับ | ผลการผูกสัมพันธ์ Link Layer | ค่า Association ID (AID) ที่ได้ | Reason Code (ถ้ามี) |
| :---------: | :--------------------------------------- | :-------------: | :-------------------------: | :-----------------------------: | :------------------ |
|  **5.3.1**  | ร้องขอ Auth & Assoc กับ AP มีอยู่จริง    | `WIFI_EVENT_STA_CONNECTED` | ผ่าน (Passed / Link-Layer Established) | 1 | - |
|  **5.3.2**  | ร้องขอ Auth & Assoc กับ AP ไม่มีอยู่จริง | `WIFI_EVENT_STA_DISCONNECTED` | ไม่ผ่าน (Failed / ไม่พบ AP ใน Scan Phase) | - | 201 / 0xC9 (`WIFI_REASON_NO_AP_FOUND`) |

### 6.2 บันทึกข้อมูล Link Layer จาก Event `WIFI_EVENT_STA_CONNECTED` (ข้อ 5.3.1)

| พารามิเตอร์ Link Layer | ค่าที่อ่านได้จริงจาก Forensic Log |
| :--- | :--- |
| **SSID** | Test-WiFi |
| **BSSID (MAC Address)** | 0A:8A:B4:BB:12:61 |
| **Channel** | 6 |
| **Auth Mode Enum** | 6 (WIFI_AUTH_WPA3_PSK / WPA3-SAE) |
| **Association ID (AID)** | 1 |

## 7. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **Association ID (AID)** คืออะไร มีบทบาทอย่างไรใน Phase 3 และส่งคืนมาในโครงสร้างข้อมูลตัวแปรใด?
- **Ans**
  - **ความหมายและบทบาท:** **Association ID (AID)** คือหมายเลขระบุตัวตนประจำสถานีลูกข่าย (1 ถึง 2007) ที่ Access Point (AP) กำหนดและส่งมอบให้แก่ ESP32 Station ผ่านเฟรม *802.11 Association Response* ในช่วง Phase 3 (Association Phase)
  - **หน้าที่สำคัญ:**
    1. ใช้เป็น Unique Identifier เพื่อระบุและอ้างอิง Station ใน Client Management Table และตารางจัดสรรทรัพยากรบน AP
    2. ใช้ในกระบวนการประหยัดพลังงาน (Power Saving Mode): AP จะใช้ค่า AID ในการระบุบิตบนแผนผัง **Traffic Indication Map (TIM)** ภายใน Beacon Frame เพื่อแจ้งเตือน Station ที่หลับอยู่ (Sleep Mode) ให้ตื่นขึ้นมารับเฟรมข้อมูลที่ค้างอยู่ใน Buffer
  - **ตัวแปรที่ส่งคืนมา:** ส่งคืนมาในสมาชิกตัวแปร `aid` (ประเภทข้อมูล `uint16_t`) ภายในโครงสร้างข้อมูล `wifi_event_sta_connected_t` เมื่อเกิด Event `WIFI_EVENT_STA_CONNECTED`

2. **เหตุใดการเชื่อมต่อ Wi-Fi ความปลอดภัยแบบ WPA2-PSK จึงสามารถผ่าน Phase 2 (Authentication) และ Phase 3 (Association) จนเกิด Event `WIFI_EVENT_STA_CONNECTED` ได้สำเร็จ แม้ผู้ใช้จะป้อนรหัสผ่าน (Password) ผิด?**
- **Ans**
  - ตามมาตรฐาน IEEE 802.11 ใน **Phase 2 (Link-Layer Authentication)** จะใช้กลไก *Open System Authentication* ซึ่งเป็นการยืนยันตัวตนแบบเปิด (แลกเปลี่ยนเพียง Request/Response เปล่าเพื่อตกลงเริ่มการเชื่อมต่อระดับสัญญาณวิทยุ) โดยยังไม่มีการตรวจสอบ Pre-Shared Key (PSK/Password) ในเฟสนี้
  - ต่อมาใน **Phase 3 (Association Phase)** เป็นเพียงการเจรจาขีดความสามารถทางกายภาพ (Supported Data Rates, Capability Information) และรับหมายเลข AID
  - ดังนั้น การเชื่อมต่อจึงถือว่าผ่านระดับ Link Layer สำเร็จและสร้าง Event `WIFI_EVENT_STA_CONNECTED`
  - การตรวจสอบรหัสผ่านจริงจะเกิดขึ้นใน **Phase 4: EAPOL 4-Way Handshake** ซึ่งทำหลังจบ Phase 3 โดยนำ Password ไปคำนวณสร้างกุญแจเข้ารหัส (PMK/PTK) หากรหัสผ่านผิด Handshake จะล้มเหลวและส่ง Disconnect Event ตามมาในภายหลัง

3. **หาก Router มีการตั้งค่า MAC Address Filtering (อนุญาตเฉพาะ MAC ที่ลงทะเบียน) ESP32 จะล้มเหลวในเฟสใด และจะส่ง Disconnect Reason Code ใดออกมา?**
- **Ans**
  - **เฟสที่ล้มเหลว:** จะล้มเหลวตั้งแต่ **Phase 2: Authentication Phase** (หรือ Phase 3: Association Phase แล้วแต่ผู้ผลิต Router) โดยเมื่อ ESP32 ส่งเฟรม *802.11 Authentication Request* ที่มี MAC Address นอกเหนือจาก Whitelist ทาง Router จะปฏิเสธการเชื่อมต่อโดยส่ง Auth Response ที่มี Status Code เป็นปฏิเสธ (Deny) ทันที
  - **Disconnect Reason Code ที่ส่งออกมา:** จะได้รับ Event `WIFI_EVENT_STA_DISCONNECTED` พร้อม Reason Code:
    - **`WIFI_REASON_AUTH_FAIL` (Reason Decimal `202` / Hex `0xCA`)** หรือ
    - **`WIFI_REASON_UNSPECIFIED` (Reason Decimal `1` / Hex `0x01`)** / **`WIFI_REASON_AUTH_EXPIRE` (Reason Decimal `2` / Hex `0x02`)**

4. **สรุปความแตกต่างสำคัญระหว่างจุดสิ้นสุดของ Phase 3 (Link-Layer Connected) กับจุดสิ้นสุดของ Phase 5 (IP Address Assigned)**
- **Ans**

| มิติการเปรียบเทียบ          | จุดสิ้นสุด Phase 3 (Link-Layer Connected)                                              | จุดสิ้นสุด Phase 5 (IP Address Assigned)                                                    |
| :-------------------------- | :------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------ |
| **OSI Layer**               | **Layer 2 (Data Link Layer)**                                                          | **Layer 3 (Network Layer)**                                                                 |
| **Event ประจำเฟส**          | `WIFI_EVENT_STA_CONNECTED`                                                             | `IP_EVENT_STA_GOT_IP`                                                                       |
| **สถานะการเชื่อมต่อ**       | จับคู่คลื่นวิทยุ 802.11 กับ AP สำเร็จ, ได้รับ AID                                      | ได้รับ IP Address, Subnet Mask, Gateway จาก DHCP Server                                     |
| **ความปลอดภัย (Security)**  | ได้รับการผูกสัมพันธ์ระดับ Link แต่กระบวนการ Handshake กุญแจอาจยังไม่เสร็จสิ้น          | แลกเปลี่ยนคีย์เข้ารหัสเสร็จสิ้น และข้อมูลผ่านการเข้ารหัสแล้ว                                |
| **ความพร้อมในการส่งข้อมูล** | ส่งได้เฉพาะเฟรมระดับ Link Layer (802.11 / EAPOL) **ยังใช้งาน Internet/TCP/UDP ไม่ได้** | **พร้อมสื่อสารระดับเครือข่ายเต็มรูปแบบ** สามารถส่ง HTTP, MQTT, Socket ออกสู่อินเทอร์เน็ตได้ |

## Log
```
I (27) boot: ESP-IDF v6.0.2 2nd stage bootloader
I (27) boot: compile time Aug  3 2026 10:38:37
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
I (79) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=1a638h (108088) map
I (125) esp_image: segment 1: paddr=0002a660 vaddr=3ffb0000 size=04528h ( 17704) load
I (132) esp_image: segment 2: paddr=0002eb90 vaddr=40080000 size=01488h (  5256) load
I (134) esp_image: segment 3: paddr=00030020 vaddr=400d0020 size=86208h (549384) map
I (332) esp_image: segment 4: paddr=000b6230 vaddr=40081488 size=14180h ( 82304) load
I (366) esp_image: segment 5: paddr=000ca3b8 vaddr=50000000 size=00028h (    40) load
I (377) boot: Loaded app from partition at offset 0x10000
I (377) boot: Disabling RNG early entropy source...
I (387) cpu_start: Multicore app
I (396) cpu_start: GPIO 3 and 1 are used as console UART I/O pins
I (396) cpu_start: Pro cpu start user code
I (396) cpu_start: cpu freq: 160000000 Hz
I (398) app_init: Application information:
I (402) app_init: Project name:     wifi_auth_assoc_phase
I (407) app_init: App version:      e011982-dirty
I (411) app_init: Compile time:     Aug  3 2026 10:38:32
I (416) app_init: ELF file SHA256:  3f19deb77...
I (421) app_init: ESP-IDF:          v6.0.2
I (424) efuse_init: Min chip rev:     v0.0
I (428) efuse_init: Max chip rev:     v3.99 
I (432) efuse_init: Chip rev:         v3.1
I (436) heap_init: Initializing. RAM available for dynamic allocation:
I (442) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
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
I (502) LAB_AUTH_ASSOC: [FORENSIC]: Call nvs_flash_init()
I (542) LAB_AUTH_ASSOC: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (542) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_netif_init()
I (542) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_event_loop_create_default()
I (552) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (562) LAB_AUTH_ASSOC: [FORENSIC]: esp_netif_create_default_wifi_sta() returned 0x3ffbddfc
I (562) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_init(&cfg)
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
I (662) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_event_handler_instance_register(WIFI_EVENT)
I (662) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (672) LAB_AUTH_ASSOC: ==================================================================
I (682) LAB_AUTH_ASSOC:   Lab 5.3: Wi-Fi Authentication & Association Phase (ESP-IDF Forensic)
I (692) LAB_AUTH_ASSOC: ==================================================================
I (702) LAB_AUTH_ASSOC: 

I (702) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (712) LAB_AUTH_ASSOC: >>> Experiment 5.3.1: Link-Layer Auth & Assoc Phase Test
I (712) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (722) LAB_AUTH_ASSOC:   Target SSID    : "Test-WiFi"
I (732) LAB_AUTH_ASSOC:   Target Password: "0954276527"
I (732) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_stop()
I (742) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (742) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (772) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (772) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_start()
I (772) phy_init: phy_version 4863,a3a4459,Oct 28 2025,14:30:06
I (862) phy_init: Saving new calibration data due to checksum failure or outdated calibration data, mode(0)
I (932) wifi:mode : sta (88:57:21:ae:44:84)
I (942) wifi:enable tsf
I (942) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (942) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (942) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (952) LAB_AUTH_ASSOC: [FORENSIC]: Initiating 802.11 Link-Layer Connection (Auth & Assoc)...
I (962) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_connect()
I (962) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (1592) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1, snd_ch_cfg:0x0
I (1592) wifi:state: init -> auth (0xb0)
I (1592) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (2202) wifi:state: auth -> assoc (0x0)
I (2252) wifi:state: assoc -> run (0x10)
E (8972) LAB_AUTH_ASSOC: [RESULT]: TEST TIMEOUT - Response timeout from AP.
I (10972) LAB_AUTH_ASSOC: 

I (10972) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (10972) LAB_AUTH_ASSOC: >>> Experiment 5.3.2: Link-Layer Test - Non-Existent AP
I (10972) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (10982) LAB_AUTH_ASSOC:   Target SSID    : "NON_EXISTENT_AP_9999"
I (10992) LAB_AUTH_ASSOC:   Target Password: "12345678"
I (10992) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_stop()
I (11002) wifi:state: run -> init (0x0)
W (11012) LAB_AUTH_ASSOC: =======================================================
W (11012) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (11012) LAB_AUTH_ASSOC:   -> Target SSID          : Test-WiFi
W (11022) LAB_AUTH_ASSOC:   -> Reason Code (Decimal): 8
W (11022) LAB_AUTH_ASSOC:   -> Reason Code (Hex)    : 0x08
W (11032) LAB_AUTH_ASSOC:   -> Reason Diagnosis     : OTHER_DISCONNECT_REASON
W (11042) LAB_AUTH_ASSOC: =======================================================
I (11042) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (11102) wifi:flush txq
I (11102) wifi:stop sw txq
I (11102) wifi:lmac stop hw txq
I (11102) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (11102) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (11132) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (11132) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_start()
I (11142) wifi:mode : sta (88:57:21:ae:44:84)
I (11142) wifi:enable tsf
I (11142) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (11152) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (11152) LAB_AUTH_ASSOC: [FORENSIC]: Initiating 802.11 Link-Layer Connection (Auth & Assoc)...
I (11162) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_connect()
I (11172) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (11152) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
W (11182) LAB_AUTH_ASSOC: [RESULT]: TEST FAILED - Disconnected event captured in Link-Layer.
I (11192) LAB_AUTH_ASSOC: ==================================================================
I (11202) LAB_AUTH_ASSOC:   [Phase 2 & Phase 3 Completed: Link-Layer Auth & Assoc Finished]
I (11212) LAB_AUTH_ASSOC: ==================================================================
I (11212) main_task: Returned from app_main()
W (12502) LAB_AUTH_ASSOC: =======================================================
W (12502) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (12512) LAB_AUTH_ASSOC:   -> Target SSID          : NON_EXISTENT_AP_9999
W (12512) LAB_AUTH_ASSOC:   -> Reason Code (Decimal): 201
W (12522) LAB_AUTH_ASSOC:   -> Reason Code (Hex)    : 0xC9
W (12522) LAB_AUTH_ASSOC:   -> Reason Diagnosis     : WIFI_REASON_NO_AP_FOUND (201) [Phase 1: SSID Not Found]
W (12532) LAB_AUTH_ASSOC: =======================================================
```
---
# 10-Labsheet-05-4-Wi-Fi-Handshake-IP-Phase
## ใบงานที่ 5.4: กระบวนการแลกเปลี่ยนคีย์ความปลอดภัยและการจัดสรรหมายเลข IP Address (4-Way Handshake & IP Assignment Phase)

---

## 6. ตารางบันทึกผลการทดลอง (Experiment Results)

### 6.1 ตารางสรุปเปรียบเทียบผลการทดลองใน Handshake & IP Phase

| ข้อการทดลอง | สถานการณ์ทดสอบ | Event `WIFI_EVENT_STA_CONNECTED` (เกิด/ไม่เกิด) | Event `IP_EVENT_STA_GOT_IP` (เกิด/ไม่เกิด) | ผลการทดลอง | Disconnect Reason Code (ถ้ามี) |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.4.1** | Password ถูกต้อง (`0954276527`) | เกิดขึ้น | เกิดขึ้น | **Passed** | N/A (เชื่อมต่อและรับ IP สำเร็จสมบูรณ์) |
| **5.4.2** | Password ผิด (`WRONG_PASSWORD_1234`) | ไม่เกิด (เนื่องจากเป็น WPA3-SAE) | ไม่เกิด | **Failed** | Decimal `202` / Hex `0xCA` (`WIFI_REASON_AUTH_FAIL`) |

### 6.2 บันทึกข้อมูล IP Network จาก Event `IP_EVENT_STA_GOT_IP` (ข้อ 5.4.1)

| พารามิเตอร์ Network Layer | ค่าที่จัดสรรได้จริงจาก DHCP Server |
| :--- | :--- |
| **IP Address** | 10.20.17.118 |
| **Subnet Mask** | 255.255.255.0 |
| **Default Gateway** | 10.20.17.253 |
## 7. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **เหตุใดกระบวนการ 4-Way Handshake จึงพิสูจน์ทราบรหัสผ่าน Wi-Fi ได้โดยไม่ต้องส่งรหัสผ่าน (Passphrase) ลอยไปในอากาศเลยแม้แต่แพ็กเกจเดียว?**
- **Ans**
  - กระบวนการ WPA2 4-Way Handshake ใช้หลักการ **Zero-Knowledge Proof** ร่วมกับฟังก์ชันแฮชทางเดียวแบบกุญแจสมมาตร (HMAC-SHA1):
  - ทั้งฝั่ง Station (ESP32) และ Access Point (AP) จะนำรหัสผ่าน (Passphrase) และชื่อ SSID ไปคำนวณสร้างกุญแจหลัก **Pairwise Master Key (PMK)** เก็บไว้ภายในหน่วยความจำของตนเอง โดยไม่มีการส่ง PMK หรือ Passphrase ผ่านคลื่นวิทยุ
  - ในระหว่างกระบวนการ Handshake ทั้งสองฝั่งจะสุ่มสร้างค่าตัวเลขเฉพาะกิจ (Nonce) คือ **ANonce** (จาก AP) และ **SNonce** (จาก Station) แล้วนำ `{PMK, ANonce, SNonce, MAC_AP, MAC_STA}` ไปคำนวณสร้าง **Pairwise Transient Key (PTK)** 
  - ภายใน PTK จะมีส่วนย่อยเรียกว่า **Key Confirmation Key (KCK)** ซึ่ง Station จะนำ KCK นี้ไปคำนวณสร้างค่าลายเซ็นตรวจสอบ **Message Integrity Code (MIC)** แนบไปในเฟรม 2/4
  - เมื่อ AP ได้รับเฟรม 2/4 จะนำ SNonce ไปคำนวณ PTK/KCK ของตนเองและหาค่า MIC หากค่า MIC ตรงกัน ย่อมพิสูจน์ได้ว่าทั้งสองฝ่ายถือครอง Password/PMK เดียวกันอย่างแน่นอน 100% โดยไม่ต้องส่งรหัสผ่านจริงออกไปในอากาศ

2. **อธิบายบทบาทและที่มาของคีย์ PMK (Pairwise Master Key) และ PTK (Pairwise Transient Key) ว่ามีความสัมพันธ์กันอย่างไรในการเข้ารหัสเฟรมข้อมูล?**
- **Ans**
  - **PMK (Pairwise Master Key):**
    - **ที่มา:** ในโหมด WPA2-Personal (PSK) ค่า PMK ขนาด 256 บิต (32 ไบต์) ถูกแปลงมาจาก Passphrase และ SSID ผ่านฟังก์ชัน PBKDF2 (Password-Based Key Derivation Function 2) ทำการแฮชซ้ำ 4,096 รอบ
    - **บทบาท:** ทำหน้าที่เป็น "กุญแจแม่บท (Master Secret)" ที่คงที่ตลอดการใช้งาน ทำหน้าที่เป็นวัตถุดิบตั้งต้น (Seed) ในการนำไปคำนวณสร้าง PTK
  - **PTK (Pairwise Transient Key):**
    - **ที่มา:** เป็น "กุญแจชั่วคราวแบบสุ่ม" ขนาด 384 หรือ 512 บิต ที่คำนวณแบบพลวัต (Dynamic) ผ่านฟังก์ชัน PRF (Pseudo-Random Function) โดยนำ `{PMK, ANonce, SNonce, AP MAC, Station MAC}` มารวมกัน
    - **บทบาทและความสัมพันธ์:** PTK จะถูกแบ่งออกเป็น 3 ส่วนย่อยเพื่อใช้งานจริง:
      1. **KCK (Key Confirmation Key):** ใช้ตรวจสอบความถูกต้องและพิสูจน์ตัวตนผ่านค่า MIC ในเฟรม EAPOL-Key
      2. **KEK (Key Encryption Key):** ใช้เข้ารหัสกุญแจกลุ่ม (Group Temporal Key - GTK) ที่ AP ส่งมอบให้ Station ในเฟรม 3/4
      3. **TK (Temporal Key):** เป็นกุญแจที่ถูกนำไปติดตั้งลงในตัวถอดรหัสระดับฮาร์ดแวร์ (Hardware Crypto Engine) เพื่อใช้เข้ารหัสและถอดรหัสข้อมูลจริง (Unicast Data Traffic) ระดับ Layer 2 ตลอดช่วงเวลาที่เชื่อมต่อ

3. **เหตุใดเมื่อเราพิมพ์ Password ผิด (ข้อ 5.4.2) ESP32 จึงยังคงได้รับ Event `WIFI_EVENT_STA_CONNECTED` ก่อนที่จะเกิด Event `WIFI_EVENT_STA_DISCONNECTED` ตามมาในภายหลัง?**
- **Ans**
  - **ในกรณี WPA2-PSK (Legacy):**
    - **Phase 2 (Authentication) & Phase 3 (Association):** ใช้กลไก *Open System Authentication* เพื่อตกลงขีดความสามารถทางกายภาพของคลื่นวิทยุและรับ Association ID (AID) ซึ่งในขั้นตอนนี้ **ยังไม่มีการตรวจสอบรหัสผ่าน (PSK)** เมื่อผูกสัมพันธ์ระดับ Link สำเร็จ Wi-Fi Driver จึงปล่อย Event `WIFI_EVENT_STA_CONNECTED`
    - **Phase 4 (4-Way Handshake):** เกิดขึ้น *หลังจาก* ได้รับ `WIFI_EVENT_STA_CONNECTED` แล้ว เมื่อเริ่มกระบวนการคำนวณ PTK ด้วยรหัสผ่านที่ผิด ค่า MIC ในเฟรม 2/4 จะไม่ตรงกับที่ AP คาดหวัง AP จึงไม่ยอมรับและส่งผลให้เกิด Handshake Timeout ปล่อย Event `WIFI_EVENT_STA_DISCONNECTED` พร้อม Reason Code `15` หรือ `204` ตามมาในภายหลัง
  - **ในกรณี WPA3-SAE (ผลการทดลองจริง):**
    - หาก Access Point ทำงานด้วยระบบความปลอดภัย WPA3-SAE (Simultaneous Authentication of Equals) การตรวจสอบรหัสผ่านจะทำตั้งแต่ Phase 2 (SAE Authentication) หากรหัสผ่านผิด การยืนยันตัวตนจะล้มเหลวทันทีและส่ง Disconnect Event ด้วย Reason Code `202` (`WIFI_REASON_AUTH_FAIL`) โดยจะไม่ผ่านไปยัง Phase 3 และจะไม่เกิด Event `WIFI_EVENT_STA_CONNECTED`

4. **หากเครือข่าย Wi-Fi ไม่มี DHCP Server (ไม่มีการแจก IP อัตโนมัติ) ผลการทดลองในข้อ 5.4.1 จะหยุดอยู่ที่ขั้นตอนใด และจะไม่เกิด Event ใดขึ้น?**
- **Ans**
  - **ขั้นตอนที่หยุด:** การเชื่อมต่อจะผ่าน Phase 1 ถึง Phase 4 สำเร็จสมบูรณ์ โดย ESP32 และ AP สามารถตกลงคีย์ PTK/GTK และเปิดใช้งาน Hardware Encryption ได้ตามปกติ แต่จะมาหยุดค้างที่ **Phase 5 (IP Assignment Phase)** เนื่องจาก LwIP DHCP Client ส่งคำขอ *DHCP Discover / Request* ออกไปแล้วไม่ได้รับการตอบกลับ (DHCP Timeout)
  - **Event ที่เกิดขึ้น:** จะได้รับ Event `WIFI_EVENT_STA_CONNECTED`
  - **Event ที่จะไม่เกิดขึ้น:** **จะไม่เกิด Event `IP_EVENT_STA_GOT_IP` อย่างแน่นอน**
  - **ผลกระทบ:** ESP32 จะไม่มี IP Address, Subnet Mask และ Gateway ทำให้อินเทอร์เฟซไม่สามารถส่งแพ็กเก็ต TCP/IP หรือเชื่อมต่ออินเทอร์เน็ตได้ เว้นแต่ผู้พัฒนาจะกำหนดค่า IP แบบคงที่ (Static IP) ให้กับอินเทอร์เฟซด้วยตนเองผ่านคำสั่ง `esp_netif_set_ip_info()`

## Log
```
I (27) boot: ESP-IDF v6.0.2 2nd stage bootloader
I (27) boot: compile time Aug  3 2026 11:05:54
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
I (79) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=1a868h (108648) map
I (125) esp_image: segment 1: paddr=0002a890 vaddr=3ffb0000 size=04528h ( 17704) load
I (132) esp_image: segment 2: paddr=0002edc0 vaddr=40080000 size=01258h (  4696) load
I (134) esp_image: segment 3: paddr=00030020 vaddr=400d0020 size=86360h (549728) map
I (332) esp_image: segment 4: paddr=000b6388 vaddr=40081258 size=143b0h ( 82864) load
I (366) esp_image: segment 5: paddr=000ca740 vaddr=50000000 size=00028h (    40) load
I (377) boot: Loaded app from partition at offset 0x10000
I (377) boot: Disabling RNG early entropy source...
I (388) cpu_start: Multicore app
I (396) cpu_start: GPIO 3 and 1 are used as console UART I/O pins
I (397) cpu_start: Pro cpu start user code
I (397) cpu_start: cpu freq: 160000000 Hz
I (398) app_init: Application information:
I (402) app_init: Project name:     wifi_handshake_ip_phase
I (407) app_init: App version:      508986f-dirty
I (412) app_init: Compile time:     Aug  3 2026 11:05:50
I (417) app_init: ELF file SHA256:  5b8332c20...
I (421) app_init: ESP-IDF:          v6.0.2
I (425) efuse_init: Min chip rev:     v0.0
I (429) efuse_init: Max chip rev:     v3.99 
I (433) efuse_init: Chip rev:         v3.1
I (437) heap_init: Initializing. RAM available for dynamic allocation:
I (443) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (448) heap_init: At 3FFB8A40 len 000275C0 (157 KiB): DRAM
I (453) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (459) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (464) heap_init: At 40095608 len 0000A9F8 (42 KiB): IRAM
W (471) spi_flash: Detected boya flash chip but using generic driver. For optimal functionality, enable `SPI_FLASH_SUPPORT_BOYA_CHIP` in menuconfig
I (483) spi_flash: detected chip: generic
I (486) spi_flash: flash io: dio
W (489) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (503) main_task: Started on CPU0
I (503) main_task: Calling app_main()
I (503) LAB_HANDSHAKE_IP: [FORENSIC]: Call nvs_flash_init()
I (543) LAB_HANDSHAKE_IP: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (543) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_netif_init()
I (553) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_event_loop_create_default()
I (553) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (563) LAB_HANDSHAKE_IP: [FORENSIC]: esp_netif_create_default_wifi_sta() returned 0x3ffbddfc
I (573) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_init(&cfg)
I (583) wifi:wifi driver task: 3ffc04e8, prio:23, stack:6656, core=0
I (593) wifi:wifi firmware version: 00ad238
I (593) wifi:wifi certification version: v7.0
I (593) wifi:config NVS flash: enabled
I (593) wifi:config nano formatting: disabled
I (603) wifi:Init data frame dynamic rx buffer num: 32
I (603) wifi:Init static rx mgmt buffer num: 5
I (613) wifi:Init management short buffer num: 32
I (613) wifi:Init dynamic tx buffer num: 32
I (623) wifi:Init static rx buffer size: 1600
I (623) wifi:Init static rx buffer num: 10
I (623) wifi:Init dynamic rx buffer num: 32
I (633) wifi_init: rx ba win: 6
I (633) wifi_init: accept mbox: 6
I (633) wifi_init: tcpip mbox: 32
I (643) wifi_init: udp mbox: 6
I (643) wifi_init: tcp mbox: 6
I (643) wifi_init: tcp tx win: 5760
I (653) wifi_init: tcp rx win: 5760
I (653) wifi_init: tcp mss: 1440
I (653) wifi_init: WiFi IRAM OP enabled
I (663) wifi_init: WiFi RX IRAM OP enabled
I (663) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_event_handler_instance_register(WIFI_EVENT)
I (673) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_event_handler_instance_register(IP_EVENT)
I (673) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (683) LAB_HANDSHAKE_IP: ==================================================================
I (693) LAB_HANDSHAKE_IP:   Lab 5.4: 4-Way Handshake & IP Assignment Phase (ESP-IDF Forensic)
I (703) LAB_HANDSHAKE_IP: ==================================================================
I (713) LAB_HANDSHAKE_IP: 

I (713) LAB_HANDSHAKE_IP: ------------------------------------------------------------------
I (723) LAB_HANDSHAKE_IP: >>> Experiment 5.4.1: Handshake & IP Test - Correct Password
I (723) LAB_HANDSHAKE_IP: ------------------------------------------------------------------
I (733) LAB_HANDSHAKE_IP:   Target SSID    : "Test-WiFi"
I (743) LAB_HANDSHAKE_IP:   Target Password: "0954276527"
I (743) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_stop()
I (1753) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
I (1773) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (1773) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_start()
I (1773) phy_init: phy_version 4863,a3a4459,Oct 28 2025,14:30:06
I (1863) phy_init: Saving new calibration data due to checksum failure or outdated calibration data, mode(0)
I (1943) wifi:mode : sta (88:57:21:ae:44:84)
I (1943) wifi:enable tsf
I (1943) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (1943) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (1943) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (1953) LAB_HANDSHAKE_IP: [FORENSIC]: Initiating Wi-Fi connection...
I (1963) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_connect()
I (1973) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (3193) wifi:new:<11,0>, old:<1,0>, ap:<255,255>, sta:<11,0>, prof:1, snd_ch_cfg:0x0
I (3203) wifi:state: init -> auth (0xb0)
I (3203) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (3783) wifi:state: auth -> assoc (0x0)
I (3823) wifi:state: assoc -> run (0x10)
I (3863) wifi:connected with Test-WiFi, aid = 1, channel 11, BW20, bssid = 0a:8a:b4:bb:12:61
I (3873) wifi:security: WPA3-SAE HUNT_AND_PECK, phy: bgn, rssi: -65, cipher(pairwise:0x3, group:0x3), pmf:1
I (3883) wifi:pm start, type: 1

I (3883) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us
I (3893) wifi:dp: 2, bi: 102400, li: 4, scale listen interval from 307200 us to 409600 us
I (3893) wifi:AP's beacon interval = 102400 us, DTIM period = 2
I (3903) LAB_HANDSHAKE_IP: =======================================================
I (3903) wifi:<ba-add>idx:0 (ifx:0, 0a:8a:b4:bb:12:61), tid:0, ssn:0, winSize:64
I (3903) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_CONNECTED received!
I (3923) LAB_HANDSHAKE_IP:   -> Phase 2 (Auth) & Phase 3 (Assoc) PASSED
I (3923) LAB_HANDSHAKE_IP:   -> Connected SSID  : Test-WiFi
I (3933) LAB_HANDSHAKE_IP:   -> BSSID           : 0A:8A:B4:BB:12:61
I (3943) LAB_HANDSHAKE_IP:   -> Channel         : 11
I (3943) LAB_HANDSHAKE_IP:   -> Association ID  : 34680
I (3953) LAB_HANDSHAKE_IP: [FORENSIC]: Entering Phase 4: 4-Way EAPOL Key Exchange...
I (3953) LAB_HANDSHAKE_IP: =======================================================
I (5133) esp_netif_handlers: sta ip: 10.20.17.118, mask: 255.255.255.0, gw: 10.20.17.253
I (5133) LAB_HANDSHAKE_IP: =======================================================
I (5133) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: IP_EVENT_STA_GOT_IP received!
I (5143) LAB_HANDSHAKE_IP:   [SUCCESS]: Phase 4 (4-Way Handshake) & Phase 5 (DHCP IP) COMPLETED!
I (5153) LAB_HANDSHAKE_IP:   -> Allocated IP Address : 10.20.17.118
I (5153) LAB_HANDSHAKE_IP:   -> Subnet Mask          : 255.255.255.0
I (5163) LAB_HANDSHAKE_IP:   -> Default Gateway      : 10.20.17.253
I (5163) LAB_HANDSHAKE_IP: =======================================================
I (5173) LAB_HANDSHAKE_IP: [RESULT]: TEST PASSED - 4-Way Handshake & DHCP IP Assignment Successful!
I (8183) LAB_HANDSHAKE_IP: 

I (8183) LAB_HANDSHAKE_IP: ------------------------------------------------------------------
I (8183) LAB_HANDSHAKE_IP: >>> Experiment 5.4.2: Handshake Test - Incorrect Password
I (8183) LAB_HANDSHAKE_IP: ------------------------------------------------------------------
I (8193) LAB_HANDSHAKE_IP:   Target SSID    : "Test-WiFi"
I (8203) LAB_HANDSHAKE_IP:   Target Password: "WRONG_PASSWORD_1234"
I (8203) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_stop()
I (8213) wifi:state: run -> init (0x0)
I (8223) wifi:pm stop, total sleep time: 3422487 us / 4335980 us

I (8223) wifi:<ba-del>idx:0, tid:0
W (8223) LAB_HANDSHAKE_IP: =======================================================
W (8233) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (8233) LAB_HANDSHAKE_IP:   -> Target SSID          : Test-WiFi
W (8243) LAB_HANDSHAKE_IP:   -> Reason Code (Decimal): 8
W (8253) LAB_HANDSHAKE_IP:   -> Reason Code (Hex)    : 0x08
W (8253) LAB_HANDSHAKE_IP:   -> Reason Diagnosis     : OTHER_DISCONNECT_REASON
W (8263) LAB_HANDSHAKE_IP: =======================================================
I (8273) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (8293) wifi:flush txq
I (8293) wifi:stop sw txq
I (8293) wifi:lmac stop hw txq
I (9293) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
I (9313) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (9313) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_start()
I (9323) wifi:mode : sta (88:57:21:ae:44:84)
I (9323) wifi:enable tsf
I (9323) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (9333) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (9333) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (9343) LAB_HANDSHAKE_IP: [FORENSIC]: Initiating Wi-Fi connection...
I (9353) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_connect()
I (9353) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (9433) wifi:new:<11,0>, old:<1,0>, ap:<255,255>, sta:<11,0>, prof:1, snd_ch_cfg:0x0
I (9433) wifi:state: init -> auth (0xb0)
I (9443) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (10043) wifi:state: auth -> init (0x600)
W (10063) LAB_HANDSHAKE_IP: =======================================================
W (10073) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (10073) LAB_HANDSHAKE_IP:   -> Target SSID          : Test-WiFi
W (10073) LAB_HANDSHAKE_IP:   -> Reason Code (Decimal): 202
W (10083) LAB_HANDSHAKE_IP:   -> Reason Code (Hex)    : 0xCA
W (10093) LAB_HANDSHAKE_IP:   -> Reason Diagnosis     : WIFI_REASON_AUTH_FAIL (1/202) [Phase 2: Auth Rejected / MAC Filter / SAE Auth Fail]
W (10103) LAB_HANDSHAKE_IP: =======================================================
W (10103) LAB_HANDSHAKE_IP: [RESULT]: TEST FAILED - Disconnected during Handshake or Auth.
I (10113) LAB_HANDSHAKE_IP: ==================================================================
I (10123) LAB_HANDSHAKE_IP:   [Phase 4 & Phase 5 Completed: Wi-Fi Handshake & IP Lab Finished]
I (10133) LAB_HANDSHAKE_IP: ==================================================================
I (10143) main_task: Returned from app_main()
```