# HỆ THỐNG GIÁM SÁT VÀ QUẢN LÝ NHẬT KÝ AN TOÀN - TÒA E (ĐẠI HỌC ĐIỆN LỰC)

> **Môn học / Đồ án:** Phân tích và Thiết kế An toàn Mạng Máy tính  
> **Đơn vị áp dụng:** Tòa E - Trường Đại học Điện Lực (EPU)  
> **Kiến trúc cốt lõi:** 3-Tier Enterprise SIEM & Telemetry (Wazuh / ELK Stack, Suricata NIDS, Prometheus & Grafana, Syslog RFC 5424, SNMPv3, NetFlow).

---

## 📑 MỤC LỤC
1. [Tổng Quan Đề Tài & Bối Cảnh Tòa E](#1-tổng-quan-đề-tài--bối-cảnh-tòa-e)
2. [Mô Hình Kiến Trúc Hệ Thống (3 Tầng)](#2-mô-hình-kiến-trúc-hệ-thống-3-tầng)
3. [Quy Hoạch Phân Vùng Mạng (VLAN Segmentation)](#3-quy-hoạch-phân-vùng-mạng-vlan-segmentation)
4. [Phân Chia Nhiệm Vụ 5 Thành Viên](#4-phân-chia-nhiệm-vụ-5-thành-viên)
5. [Ma Trận Trách Nhiệm (RACI Matrix)](#5-ma-trận-trách-nhiệm-raci-matrix)
6. [Kế Hoạch Triển Khai & Kịch Bản Kiểm Thử (Timeline & Tests)](#6-kế-hoạch-triển-khai--kịch-bản-kiểm-thử)
7. [Tài Liệu Giáo Trình & Hướng Dẫn Kỹ Thuật (7 Chuyên Đề)](#7-tài-liệu-giáo-trình--hướng-dẫn-kỹ-thuật-7-chuyên-đề)

---

## 1. TỔNG QUAN ĐỀ TÀI & BỐI CẢNH TÒA E

Tòa E trường Đại học Điện Lực là khu phức hợp học tập và nghiên cứu quy mô lớn bao gồm:
* **Các phòng thực hành máy tính (Lab CNTT, Mạng máy tính, An toàn thông tin):** Môi trường sinh viên thực hành nhiều công cụ, nguy cơ phát tán mã độc, quét cổng mạng và tấn công nội bộ cao.
* **Văn phòng Khoa / Phòng làm việc Giảng viên:** Cần bảo mật tài liệu học thuật, đề thi, bảng điểm và các thông tin quản trị.
* **Hệ thống Wi-Fi sinh viên & khách vãng lai:** Lưu lượng lớn, nhiều thiết bị di động cá nhân không kiểm soát.
* **Phòng máy chủ & Tủ mạng phân tầng:** Chứa Core Switch, Router biên, Firewall, NVR Camera và các máy chủ ứng dụng nội bộ.

**Mục tiêu hệ thống:** Xây dựng giải pháp giám sát tập trung toàn diện, thu thập nhật ký (Log) từ toàn bộ thiết bị mạng và máy chủ, tự động phân tích tương quan sự kiện (Correlation) để phát hiện sớm các cuộc tấn công và gửi cảnh báo tức thời tới đội ngũ quản trị.

---

## 2. MÔ HÌNH KIẾN TRÚC HỆ THỐNG (CHUẨN HÓA CHO ĐỒ ÁN SINH VIÊN)

Hệ thống được thiết kế theo hướng **thực tế, tối ưu tài nguyên (chạy mượt trên máy cá nhân/máy ảo 8GB-16GB RAM)** với mô hình thu thập và giám sát nhật ký tập trung sử dụng **Wazuh All-in-One**:

```mermaid
flowchart TB
    subgraph LAN["HẠ TẦNG MẠNG TÒA E (Mô phỏng EVE-NG / Cisco Packet Tracer / VMware)"]
        SW["Switch Tòa E (VLAN 10, 20, 30, 40)"]
        SRV["Máy chủ Linux / Windows Server"]
        CLI["Máy trạm Client Phòng Lab"]
    end

    subgraph SIEM["MÁY CHỦ GIÁM SÁT TẬP TRUNG (Wazuh All-in-One Server)"]
        RSYS["Syslog Receiver (UDP 514)"]
        WZ_MGR["Wazuh Manager (Engine phân tích log & luật có sẵn)"]
        WZ_DB["Wazuh Dashboard (Giao diện Web SOC trực quan)"]
        BOT["Telegram Alerting (Bắn tin nhắn khi bị tấn công)"]
    end

    SW -->|1. Đẩy Syslog sự kiện mạng| RSYS
    SRV -->|2. Wazuh Agent (Gửi log hệ điều hành & dịch vụ)| WZ_MGR
    CLI -->|3. Thử nghiệm đăng nhập / Quét mạng| SW
    
    RSYS --> WZ_MGR
    WZ_MGR --> WZ_DB
    WZ_MGR -->|4. Cảnh báo khẩn cấp (Rule Level >= 8)| BOT
```

---

## 3. QUY HOẠCH PHÂN VÙNG MẠNG (VLAN SEGMENTATION)
*(Thiết kế gọn gàng, đúng chuẩn môn học mạng máy tính)*

| VLAN ID | Tên Phân Vùng | Mục Đích Sử Dụng | Dải Mạng IP | Thiết lập An ninh |
| :---: | :--- | :--- | :--- | :--- |
| **VLAN 10** | `Management` | Quản trị Switch, Router | `192.168.10.0/24` | Đặt mật khẩu mã hóa, chỉ cho phép IP quản trị SSH |
| **VLAN 20** | `Server_Zone` | Chứa Web Server, Wazuh SIEM Server | `192.168.20.0/24` | Mở port 80/443 (Web), port 514 (Syslog), 1514 (Agent) |
| **VLAN 30** | `GiangVien` | Máy tính phòng Giảng viên / Văn phòng | `192.168.30.0/24` | Được phép truy cập Server, ra Internet |
| **VLAN 40** | `PhongLab` | Máy tính thực hành của sinh viên | `192.168.40.0/24` | Bị ACL chặn truy cập sang VLAN 10 (Quản trị) |

---

## 4. PHÂN CHIA NHIỆM VỤ 5 THÀNH VIÊN (VỪA SỨC & RÕ RÀNG)

### 👑 1. NGUYỄN ANH XUÂN (Trưởng nhóm - Phụ trách Hạ tầng máy chủ & SIEM)
* **Mục tiêu:** Dựng thành công "trung tâm tiếp nhận và xử lý log".
* **Nhiệm vụ cụ thể:**
  1. **Cài đặt máy chủ Wazuh All-in-One:** Dùng file OVA có sẵn hoặc chạy bằng Docker Compose (cực nhanh, có sẵn Web UI đẹp mắt).
  2. **Cấu hình tiếp nhận Syslog:** Mở cổng UDP 514 trên Wazuh để sẵn sàng nhận log từ Switch/Router của Minh Trí.
  3. **Tích hợp cảnh báo Telegram:** Cấu hình tính năng Custom Alert có sẵn của Wazuh để tự động bắn tin nhắn báo động về nhóm Telegram khi phát hiện tấn công.
* **Sản phẩm nghiệm thu:** Máy chủ Wazuh hoạt động trơn tru; Bot Telegram nhận được tin nhắn test.

---

### 🌐 2. NGUYỄN PHÚC VƯỢNG (Kỹ sư Mạng 1 - Thiết kế Topology & Định tuyến)
* **Mục tiêu:** Xây dựng khung mạng thông suốt cho Tòa E.
* **Nhiệm vụ cụ thể:**
  1. **Vẽ & Cấu hình mô hình mạng:** Dựng sơ đồ Tòa E trên Cisco Packet Tracer hoặc EVE-NG.
  2. **Cấu hình VLAN & Định tuyến:** Tạo VLAN 10, 20, 30, 40 và cấu hình Inter-VLAN Routing (Router-on-a-Stick).
  3. **Cấu hình dịch vụ cơ bản:**
     - Cấu hình DHCP Server cấp IP tự động cho các phòng.
     - Cấu hình **NTP Server**: Đồng bộ thời gian giữa Router/Switch và Server *(đảm bảo log hiển thị đúng giờ)*.
* **Sản phẩm nghiệm thu:** File lab mô phỏng mạng chạy thông suốt, các máy trạm nhận đúng IP theo phòng và ping thấy Gateway.

---

### 🛡️ 3. NGUYỄN MINH TRÍ (Kỹ sư Mạng 2 - An toàn thiết bị mạng & Đẩy Syslog)
* **Mục tiêu:** Khóa an toàn các cổng mạng và đẩy dữ liệu sự kiện mạng về Wazuh.
* **Nhiệm vụ cụ thể:**
  1. **Bảo mật cơ bản Switch/Router:**
     - Cấu hình Port Security (giới hạn 1 địa chỉ MAC trên mỗi cổng cắm phòng Lab).
     - Viết Access Control List (ACL) chặn máy tính phòng Lab (VLAN 40) không được truy cập vào dải mạng Quản trị (VLAN 10).
  2. **Cấu hình đẩy Syslog về Wazuh:**
     - Dùng lệnh Cisco IOS: `logging host <IP_Wazuh>`, `logging trap informational`.
     - Test bật/tắt cổng (`shutdown`) để xem log có nhảy về Wazuh không.
* **Sản phẩm nghiệm thu:** Lệnh cấu hình ACL & Port Security; Bằng chứng log từ Switch/Router hiển thị trong Wazuh.

---

### 🔒 4. NGUYỄN THÀNH ĐẠT (Chuyên viên An toàn thông tin - Phụ trách Kịch bản tấn công & Luật)
* **Mục tiêu:** Thực hiện tấn công thử nghiệm để chứng minh hệ thống có bắt được sự cố.
* **Nhiệm vụ cụ thể:**
  1. **Cài đặt Agent giám sát:** Cài Wazuh Agent lên 1 máy chủ Linux (Ubuntu Server) và kích hoạt giám sát file cấu hình (FIM).
  2. **Thực hiện 3 kịch bản tấn công bằng Kali Linux:**
     - *Kịch bản 1 (Quét mạng):* Dùng lệnh `nmap -sS <IP_Server>` quét cổng mở.
     - *Kịch bản 2 (Dò mật khẩu):* Dùng công cụ `hydra` dò thử mật khẩu SSH/FTP sai liên tiếp.
     - *Kịch bản 3 (Tấn công SYN Flood):* Dùng `hping3` bắn gói tin SYN thử nghiệm.
  3. **Kiểm tra phát hiện:** Đối chiếu xem Wazuh có kích hoạt các Rule cảnh báo mặc định (Rule 5710, 5712, 31101...) hay không.
* **Sản phẩm nghiệm thu:** Ảnh chụp màn hình lệnh tấn công trên Kali Linux và sự kiện cảnh báo tương ứng trên Wazuh.

---

### 🔍 5. LÊ TUẤN ANH (Hỗ trợ An toàn thông tin, Thử nghiệm & Tổng hợp Báo cáo)
* **Mục tiêu:** Hỗ trợ Thành Đạt thử nghiệm các hành vi người dùng, trực quan hóa giao diện và làm tài liệu.
* **Nhiệm vụ cụ thể (Security mức cơ bản):**
  1. **Cài đặt Wazuh Agent trên máy Client Windows:** Chạy file cài đặt `.msi` trên máy tính phòng Lab, nhập IP Wazuh Server và kiểm tra trạng thái hiển thị "Active".
  2. **Chạy các thử nghiệm an ninh người dùng đơn giản:**
     - Thử đăng nhập sai mật khẩu Windows 5 lần liên tiếp.
     - Thử tạo 1 tài khoản User mới trên máy hoặc chỉnh sửa 1 file trong thư mục giám sát.
     - Kiểm tra xem trên Web Wazuh có ghi lại hành động đó không.
  3. **Tùy chỉnh Dashboard trên Web Wazuh:** Lọc và chụp ảnh các biểu đồ: Số lượng cảnh báo theo ngày, Top IP đăng nhập sai, Danh sách sự kiện mức High.
  4. **Tổng hợp Báo cáo Word & Slide thuyết trình:** Soạn thảo file báo cáo theo mẫu đồ án của Trường ĐH Điện Lực và làm slide PowerPoint trình chiếu cho cả nhóm.
* **Sản phẩm nghiệm thu:** File Word Báo cáo đồ án hoàn chỉnh; Slide thuyết trình đẹp mắt; Bảng ghi nhật ký thử nghiệm (Test Cases).

---

## 5. MA TRẬN TRÁCH NHIỆM (RACI MATRIX)

| Giai đoạn & Hạng mục công việc | Nguyễn Anh Xuân (Lead) | Nguyễn Phúc Vượng (Net 1) | Nguyễn Minh Trí (Net 2) | Nguyễn Thành Đạt (Security) | Lê Tuấn Anh (Sec Support) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| 1. Thiết kế sơ đồ mạng Tòa E & Cấu hình VLAN/IP | I | **R / A** | C | I | I |
| 2. Cấu hình ACLs, Port Security & Đẩy Syslog Switch | I | C | **R / A** | I | I |
| 3. Cài đặt Máy chủ Wazuh SIEM & Bot Telegram | **R / A** | I | C | C | I |
| 4. Cài Wazuh Agent (Linux Server) & Pentest Kali Linux | C | I | I | **R / A** | C |
| 5. Cài Wazuh Agent (Windows) & Test đăng nhập sai | I | I | I | **A** | **R** |
| 6. Tùy chỉnh Dashboard Web Wazuh & Chụp ảnh minh chứng | **A** | I | I | C | **R** |
| 7. Soạn thảo Báo cáo Word & Làm Slide thuyết trình | **A** | C | C | C | **R** |

> **Quy ước:** **R** = Người trực tiếp làm; **A** = Người kiểm tra/nghiệm thu; **C** = Người hỗ trợ; **I** = Người nắm thông tin.

---

## 6. KỊCH BẢN KIỂM THỬ DEMO BẢO VỆ ĐỒ ÁN (TEST CASES)

| Mã TC | Kịch Bản Thử Nghiệm | Công Cụ / Thao Tác | Kết Quả Hiển Thị Trên Wazuh SIEM | Người Phụ Trách |
| :---: | :--- | :--- | :--- | :---: |
| **TC-01** | Bật/Tắt cổng Switch | Gõ lệnh `shutdown` trên cổng Switch | Nhận log `LINK-3-UPDOWN` từ Switch gửi về | Minh Trí |
| **TC-02** | Chặn truy cập trái phép | Máy phòng Lab SSH sang IP Quản trị | Bị ACL chặn, Switch sinh log ACL Deny | Minh Trí |
| **TC-03** | Quét cổng dịch vụ Server | Dùng Kali Linux chạy `nmap -sS` | Wazuh ghi nhận nhiều kết nối bất thường liên tiếp | Thành Đạt |
| **TC-04** | Dò mật khẩu SSH Server | Dùng `hydra` thử 10 pass sai | Wazuh kích hoạt cảnh báo *SSH authentication failed* | Thành Đạt |
| **TC-05** | Đăng nhập sai trên Windows | Gõ sai mật khẩu Windows 5 lần | Wazuh ghi nhận Event ID 4625 (Logon Failure) | Tuấn Anh |
| **TC-06** | Nhận cảnh báo Telegram | Khi có cảnh báo mức cao (Level >= 8) | Bot Telegram tự động gửi tin nhắn báo động | Anh Xuân |
| **TC-07** | Trực quan hóa Dashboard | Mở trình duyệt Web Wazuh | Hiển thị biểu đồ phân tích trực quan toàn bộ sự cố | Tuấn Anh |

---

## 7. TÀI LIỆU GIÁO TRÌNH & HƯỚNG DẪN KỸ THUẬT (7 CHUYÊN ĐỀ)

Tất cả các tài liệu chi tiết, cấu hình mẫu và câu hỏi vấn đáp bảo vệ đồ án đã được chuẩn bị sẵn trong thư mục [Documents/](file:///d:/School/PTTK_AnToanMang/Documents):

1. [01_Network_Fundamentals.md](file:///d:/School/PTTK_AnToanMang/Documents/01_Network_Fundamentals.md)
   - **Nền tảng mạng & Kiến trúc giao thức:** Mô hình OSI/TCP-IP, Ánh xạ dữ liệu giám sát vào từng tầng, Subnetting & Quy hoạch VLSM.
2. [02_Network_Infrastructure.md](file:///d:/School/PTTK_AnToanMang/Documents/02_Network_Infrastructure.md)
   - **Kiến trúc hạ tầng thiết bị mạng:** Router (Control/Data Plane), Switch L2/L3 (Port Security, DHCP Snooping, DAI, SPAN Port), Firewall & Server Hardening.
3. [03_Network_Segmentation.md](file:///d:/School/PTTK_AnToanMang/Documents/03_Network_Segmentation.md)
   - **Phân đoạn mạng & Thiết kế VLAN giám sát:** Chuẩn 802.1Q, Inter-VLAN Routing, Cô lập VLAN Quản trị & VLAN Giám sát kèm Extended ACLs.
4. [04_Core_Network_Services.md](file:///d:/School/PTTK_AnToanMang/Documents/04_Core_Network_Services.md)
   - **Dịch vụ mạng cốt lõi:** DHCP Audit Trail, DNS Tunneling Detection, NAT/PAT Log Correlation, **NTP Synchronization** (Yêu cầu sống còn của SIEM).
5. [05_Routing_Protocols.md](file:///d:/School/PTTK_AnToanMang/Documents/05_Routing_Protocols.md)
   - **Giao thức định tuyến:** AD, Longest Prefix Match, Static Route, OSPF MD5 Authentication, Giám sát định tuyến qua Syslog/SIEM.
6. [06_Network_Security.md](file:///d:/School/PTTK_AnToanMang/Documents/06_Network_Security.md)
   - **An toàn mạng & Các kịch bản tấn công:** Firewall Zone-based, IPsec/SSL VPN, NIDS Suricata/Snort (SPAN vs Inline), Nhận diện tấn công (Brute-Force, ARP Poisoning, SYN Flood, Port Scan).
7. [07_Monitoring_And_Logging.md](file:///d:/School/PTTK_AnToanMang/Documents/07_Monitoring_And_Logging.md)
   - **TRỌNG TÂM ĐỀ TÀI - Hệ thống Giám sát & Quản lý Nhật ký tập trung:** SNMPv3, RFC 5424 Syslog, NetFlow/IPFIX, Pipeline ELK/Wazuh, Tập luật tương quan và Kịch bản kiểm thử giả lập tấn công (Hydra, Nmap, hping3).

---
> [!TIP]
> Mỗi chuyên đề đều có sẵn: **Lý thuyết học thuật chuẩn** + **Sơ đồ chu trình** + **Cấu hình mẫu dòng lệnh (Cisco IOS, Linux Rsyslog, Wazuh XML)** + **Bộ câu hỏi ôn tập & phản biện bảo vệ đồ án (Viva Q&A)**.
