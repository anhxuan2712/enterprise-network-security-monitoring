# CHUYÊN ĐỀ 00: TỔNG QUAN ĐỀ TÀI & PHÂN CHIA NHIỆM VỤ DỰ ÁN
## HỆ THỐNG GIÁM SÁT VÀ QUẢN LÝ NHẬT KÝ AN TOÀN - TÒA E (ĐẠI HỌC ĐIỆN LỰC)

> **Môn học:** Phân tích và Thiết kế An toàn Mạng Máy tính  
> **Đơn vị ứng dụng:** Tòa E - Trường Đại học Điện Lực (EPU)  
> **Số lượng thành viên:** 05 thành viên  

---

## 📑 MỤC LỤC
1. [Bối Cảnh & Mục Tiêu Đề Tài](#1-bối-cảnh--mục-tiêu-đề-tài)
2. [Kiến Trúc Tổng Thể 3 Tầng](#2-kiến-trúc-tổng-thể-3-tầng)
3. [Quy Hoạch Phân Đoạn VLAN Tòa E](#3-quy-hoạch-phân-đoạn-vlan-tòa-e)
4. [Bảng Phân Công Nhiệm Vụ 5 Thành Viên (Chi Tiết & KPI)](#4-bảng-phân-công-nhiệm-vụ-5-thành-viên-chi-tiết--kpi)
5. [Ma Trận Trách Nhiệm RACI](#5-ma-trận-trách-nhiệm-raci)
6. [Kế Hoạch Triển Khai 4 Tuần (Gantt Chart)](#6-kế-hoạch-triển-khai-4-tuần-gantt-chart)
7. [Bảng Kịch Bản Kiểm Thử & Nghiệm Thu (Test Case Sheet Mẫu)](#7-bảng-kịch-bản-kiểm-thử--nghiệm-thu-test-case-sheet-mẫu)

---

## 1. BỐI CẢNH & MỤC TIÊU ĐỀ TÀI

### 1.1. Hiện trạng Tòa E - Đại học Điện Lực
Tòa E là tòa nhà đào tạo trọng điểm với lưu lượng mạng dày đặc:
- **Phòng Lab máy tính (Tầng 2 - Tầng 5):** Hàng trăm máy trạm sinh viên thực hành, dễ bị cài cắm mã độc, chạy tool quét mạng, tấn công nội bộ.
- **Khu vực làm việc Giảng viên & Văn phòng Khoa:** Chứa dữ liệu điểm, đề thi, thông tin cán bộ cần cách ly nghiêm ngặt.
- **Hệ thống Wi-Fi sinh viên & khách:** Số lượng truy cập đồng thời lớn, tiềm ẩn nguy cơ nghe lén (Sniffing) và giả mạo Gateway (ARP Poisoning).
- **Phòng Server & Tủ kỹ thuật:** Chứa Core Switch, Router, Firewall, NVR Camera, Server nội bộ.

### 1.2. Mục tiêu hệ thống
1. **Thu thập nhật ký tập trung:** Thu thập toàn bộ log sự kiện từ Switch, Router, Firewall, Server qua Syslog (RFC 5424), SNMPv3, NetFlow và Wazuh Agent.
2. **Đồng bộ thời gian chính xác (NTP):** Đảm bảo log từ mọi nguồn khớp nhau �## 2. KIẾN TRÚC TỔNG THỂ HỆ THỐNG (MÔ HÌNH SINH VIÊN NĂM 3)

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

## 3. QUY HOẠCH PHÂN ĐOẠN VLAN TÒA E

| VLAN ID | Tên Phân Vùng | Mục Đích Sử Dụng | Dải Mạng IP | Thiết lập An ninh |
| :---: | :--- | :--- | :--- | :--- |
| **VLAN 10** | `Management` | Quản trị Switch, Router | `192.168.10.0/24` | Đặt mật khẩu mã hóa, chỉ cho phép IP quản trị SSH |
| **VLAN 20** | `Server_Zone` | Chứa Web Server, Wazuh SIEM Server | `192.168.20.0/24` | Mở port 80/443 (Web), port 514 (Syslog), 1514 (Agent) |
| **VLAN 30** | `GiangVien` | Máy tính phòng Giảng viên / Văn phòng | `192.168.30.0/24` | Được phép truy cập Server, ra Internet |
| **VLAN 40** | `PhongLab` | Máy tính thực hành của sinh viên | `192.168.40.0/24` | Bị ACL chặn truy cập sang VLAN 10 (Quản trị) |

---

## 4. BẢNG PHÂN CÔNG NHIỆM VỤ 5 THÀNH VIÊN (VỪA SỨC & RÕ RÀNG)

### 4.1. NGUYỄN ANH XUÂN (Trưởng nhóm - Phụ trách Hạ tầng máy chủ & SIEM)
* **Mục tiêu:** Dựng thành công "trung tâm tiếp nhận và xử lý log".
* **Nhiệm vụ cụ thể:**
  1. Cài đặt máy chủ Wazuh All-in-One (bằng file OVA hoặc Docker Compose).
  2. Mở cổng tiếp nhận Syslog (UDP 514) trên Wazuh Server.
  3. Cấu hình Bot Telegram tích hợp tự động bắn cảnh báo khi có sự cố an ninh mức cao.
* **Sản phẩm bàn giao:** Máy chủ Wazuh hoạt động trơn tru; Bot Telegram nhận được tin nhắn test.

---

### 4.2. NGUYỄN PHÚC VƯỢNG (Kỹ sư Mạng 1 - Thiết kế Topology & Định tuyến)
* **Mục tiêu:** Xây dựng khung mạng thông suốt cho Tòa E.
* **Nhiệm vụ cụ thể:**
  1. Vẽ sơ đồ mạng Tòa E trên Cisco Packet Tracer / EVE-NG.
  2. Cấu hình VLAN 10, 20, 30, 40 và Inter-VLAN Routing (Router-on-a-Stick).
  3. Cấu hình DHCP Server cấp IP động và **NTP Server** đồng bộ giờ giữa các thiết bị.
* **Sản phẩm bàn giao:** File mô phỏng mạng hoạt động thông suốt; bảng phân chia IP.

---

### 4.3. NGUYỄN MINH TRÍ (Kỹ sư Mạng 2 - An toàn thiết bị mạng & Đẩy Syslog)
* **Mục tiêu:** Khóa an toàn cổng mạng và đẩy dữ liệu sự kiện mạng về Wazuh.
* **Nhiệm vụ cụ thể:**
  1. Cấu hình Port Security (giới hạn 1 MAC/cổng) và cấu hình ACL chặn VLAN Phòng Lab vào VLAN Quản trị.
  2. Bật tính năng gửi Syslog từ Cisco Switch/Router về IP máy chủ Wazuh (`logging host <ip_wazuh>`).
* **Sản phẩm bàn giao:** File cấu hình Cisco IOS (ACL, Port Security, Syslog); bằng chứng log mạng trên Wazuh.

---

### 4.4. NGUYỄN THÀNH ĐẠT (Chuyên viên An toàn thông tin - Phụ trách Kịch bản tấn công & Luật)
* **Mục tiêu:** Thực hiện tấn công thử nghiệm để chứng minh hệ thống bắt được sự cố.
* **Nhiệm vụ cụ thể:**
  1. Cài đặt Wazuh Agent lên 1 máy chủ Linux (Ubuntu Server) và kích hoạt giám sát file FIM.
  2. Dùng Kali Linux chạy 3 kịch bản tấn công thử nghiệm: Quét cổng bằng Nmap, dò mật khẩu SSH bằng Hydra, bắn gói tin SYN Flood bằng Hping3.
  3. Kiểm tra các sự kiện cảnh báo tương ứng nhảy về trên Wazuh.
* **Sản phẩm bàn giao:** Ảnh chụp màn hình lệnh tấn công trên Kali Linux và sự kiện cảnh báo tương ứng trên Wazuh.

---

### 4.5. LÊ TUẤN ANH (Hỗ trợ An toàn thông tin, Thử nghiệm & Tổng hợp Báo cáo)
* **Mục tiêu:** Hỗ trợ Thành Đạt thử nghiệm các hành vi người dùng, trực quan hóa giao diện và làm tài liệu.
* **Nhiệm vụ cụ thể (Security mức cơ bản):**
  1. Cài đặt Wazuh Agent lên máy Client Windows phòng Lab (chạy file `.msi`).
  2. Thử nghiệm hành vi người dùng: Đăng nhập sai mật khẩu Windows 5 lần, tạo user mới, chỉnh sửa file mẫu xem Wazuh có ghi log không.
  3. Tùy chỉnh Dashboard trên giao diện Web Wazuh (lọc biểu đồ số lượng cảnh báo, Top IP đăng nhập sai).
  4. Soạn thảo file Word Báo cáo đồ án theo chuẩn mẫu của Trường ĐH Điện Lực và thiết kế Slide trình chiếu.
* **Sản phẩm bàn giao:** File Word Báo cáo đồ án hoàn chỉnh; Slide thuyết trình đẹp mắt; Bảng ghi nhật ký thử nghiệm (Test Cases).

---

## 5. MA TRẬN TRÁCH NHIỆM RACI

| Giai đoạn & Hạng mục công việc | Nguyễn Anh Xuân (Lead) | Nguyễn Phúc Vượng (Net 1) | Nguyễn Minh Trí (Net 2) | Nguyễn Thành Đạt (Security) | Lê Tuấn Anh (Sec Support) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| 1. Thiết kế sơ đồ mạng Tòa E & Cấu hình VLAN/IP | I | **R / A** | C | I | I |
| 2. Cấu hình ACLs, Port Security & Đẩy Syslog Switch | I | C | **R / A** | I | I |
| 3. Cài đặt Máy chủ Wazuh SIEM & Bot Telegram | **R / A** | I | C | C | I |
| 4. Cài Wazuh Agent (Linux Server) & Pentest Kali Linux | C | I | I | **R / A** | C |
| 5. Cài Wazuh Agent (Windows) & Test đăng nhập sai | I | I | I | **A** | **R** |
| 6. Tùy chỉnh Dashboard Web Wazuh & Chụp ảnh minh chứng | **A** | I | I | C | **R** |
| 7. Soạn thảo Báo cáo Word & Làm Slide thuyết trình | **A** | C | C | C | **R** |

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
| **TC-07** | Trực quan hóa Dashboard | Mở trình duyệt Web Wazuh | Hiển thị biểu đồ phân tích trực quan toàn bộ sự cố | Tuấn Anh | (Anh Xuân)
        Cài đặt Suricata NIDS (Thành Đạt) + Cài Wazuh Agent máy trạm (Tuấn Anh)

Tuần 3: Đẩy Syslog/SNMP từ Switch/Router về cụm SIEM (Minh Trí + Anh Xuân)
        Viết tập luật tương quan (Correlation Rules) phát hiện tấn công (Thành Đạt)
        Dựng các Widget đồ thị trên SOC Dashboard (Tuấn Anh)

Tuần 4: Diễn tập giả lập tấn công nâng cao & Test cảnh báo Telegram (Thành Đạt + Anh Xuân)
        Chạy toàn bộ bảng Test Cases an ninh & chụp ảnh minh chứng (Tuấn Anh + Team)
        Hoàn thiện Báo cáo Word + Slide + Video Demo (Tuấn Anh + Anh Xuân tổng duyệt)
```

---

## 7. BẢNG KỊCH BẢN KIỂM THỬ & NGHIỆM THU (TEST CASE SHEET MẪU)

| Mã TC | Phân Loại | Mô Tả Hành Động Kiểm Thử | Kết Quả Kỳ Vọng | Trạng Thái | Người Thực Hiện |
| :---: | :--- | :--- | :--- | :---: | :---: |
| **TC-01** | NTP Sync | Kiểm tra đồng bộ giờ giữa Switch, Server và SIEM | Lệch thời gian < 100ms | Đạt | Nguyễn Phúc Vượng |
| **TC-02** | Syslog Net | Tắt/Bật một cổng mạng trên Switch (`shutdown`) | SIEM nhận log `LINK-3-UPDOWN` trong vòng 3s | Đạt | Nguyễn Minh Trí |
| **TC-03** | Port Sec | Cắm một thiết bị có MAC lạ vào cổng phòng Lab | Cổng bị khóa (`err-disable`), gửi SNMP Trap | Đạt | Nguyễn Minh Trí |
| **TC-04** | Nmap Scan | Dùng máy phòng Lab quét cổng Core Switch/Server | Suricata kích hoạt cảnh báo `ET SCAN Nmap` | Đạt | Nguyễn Thành Đạt |
| **TC-05** | SSH Brute | Chạy Hydra thử 10 mật khẩu sai vào SSH Server | Wazuh kích hoạt Rule ID 5712 (Level 10) | Đạt | Nguyễn Thành Đạt |
| **TC-06** | Agent FIM | Chỉnh sửa file cấu hình mẫu `/etc/passwd` trên Agent | Wazuh cảnh báo thay đổi toàn vẹn file (FIM) | Đạt | Lê Tuấn Anh |
| **TC-07** | Alert Bot | Tấn công thử nghiệm kích hoạt Rule Severity >= 8 | Bot Telegram nhận tin nhắn cảnh báo trong 5s | Đạt | Nguyễn Anh Xuân |
| **TC-08** | Dashboard | Mở bảng điều khiển Kibana/Grafana trên trình duyệt | Đồ thị cập nhật thời gian thực không lỗi | Đạt | Lê Tuấn Anh |
| **TC-09** | ACL Check | Máy từ VLAN 40 Ping/SSH sang VLAN 10 (Management) | Bị chặn hoàn toàn (`Host Unreachable`), có log ACL | Đạt | Lê Tuấn Anh |
