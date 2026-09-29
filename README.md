# ĐỒ ÁN: PHÂN TÍCH VÀ THIẾT KẾ HỆ THỐNG MẠNG AN TOÀN CHO DOANH NGHIỆP CÔNG NGHỆ
*(Enterprise Network Security Architecture, Zone-based Segmentation, Physical Room Blueprint & Centralized Wazuh SIEM/SOC Monitoring)*

> **Học phần:** Phân tích và Thiết kế An toàn Mạng Máy tính  
> **Chủ đề lựa chọn:** Mô hình Doanh nghiệp Công nghệ & Dịch vụ Số (Tech Enterprise - 2 Tòa nhà A & B, 26 Phòng ban, 144 Nhân sự, ~288 Endpoints)  
> **Kiến trúc cốt lõi:** Phân vùng an ninh Zone-based Segmentation (Zero Trust Principles), 3-Tier Core-Distribution-Access, Trục cáp quang Single-Mode OS2 10Gbps, 9 Tủ IDF & 1 Trung tâm Dữ liệu MDF (A-202), Wazuh SIEM All-in-One, Rsyslog RFC 5424, Extended ACLs, Port Security, 802.1X, Telegram Alerting & SOC 24/7.  
> **Tài liệu thiết kế mặt bằng chi tiết:** [ban thiet ke doanh nghiep/index-v2.html](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/ban%20thiet%20ke%20doanh%20nghiep/index-v2.html) *(Phiên bản 3.0)*

---

## 📑 MỤC LỤC TỔNG THỂ

1. [CHƯƠNG 1. GIỚI THIỆU DOANH NGHIỆP & PHÂN TÍCH NHU CẦU](#chương-1-giới-thiệu-doanh-nghiệp--phân-tích-nhu-cầu)
   - [1.1 Giới thiệu quy mô & Mô hình hoạt động](#11-giới-thiệu-quy-mô--mô-hình-hoạt-động)
   - [1.2 Cơ cấu tổ chức & Bảng phân bổ 26 phòng ban chi tiết (Tòa A & B)](#12-cơ-cấu-tổ-chức--bảng-phân-bổ-26-phòng-ban-chi-tiết-tòa-a--b)
   - [1.3 Dịch vụ CNTT hiện hành & Mục tiêu an toàn thông tin (CIA Triad)](#13-dịch-vụ-cntt-hiện-hành--mục-tiêu-an-toàn-thông-tin-cia-triad)
2. [CHƯƠNG 2. KHẢO SÁT HIỆN TRẠNG & ĐÁNH GIÁ RỦI RO HỆ THỐNG](#chương-2-khảo-sát-hiện-trạng--đánh-giá-rủi-ro-hệ-thống)
   - [2.1 Hiện trạng hạ tầng mạng truyền thống](#21-hiện-trạng-hạ-tầng-mạng-truyền-thống)
   - [2.2 Phân tích Ma trận Rủi ro & Lỗ hổng (Risk Matrix NIST SP 800-30)](#22-phân-tích-ma-trận-rủi-ro--lỗ-hổng-risk-matrix-nist-sp-800-30)
   - [2.3 Yêu cầu chuyển đổi sang hệ thống bảo mật thế hệ mới](#23-yêu-cầu-chuyển-đổi-sang-hệ-thống-bảo-mật-thế-hệ-mới)
3. [CHƯƠNG 3. THIẾT KẾ TỔNG THỂ HỆ THỐNG MẠNG & BẢO MẬT](#chương-3-thiết-kế-tổng-thể-hệ-thống-mạng--bảo-mật)
   - [3.1 Kiến trúc mạng phân tầng 3 lớp (Core - Distribution - Access) & Trục quang OS2](#31-kiến-trúc-mạng-phân-tầng-3-lớp-core---distribution---access--trục-quang-os2)
   - [3.2 Phân vùng an ninh (Security Zones) & Ma trận kiểm soát truy cập (Zone Matrix)](#32-phân-vùng-an-ninh-security-zones--ma-trận-kiểm-soát-truy-cập-zone-matrix)
   - [3.3 Sơ đồ kiến trúc logic & Luồng dữ liệu giám sát SIEM / SOC](#33-sơ-đồ-kiến-trúc-logic--luồng-dữ-liệu-giám-sát-siem--soc)
4. [CHƯƠNG 4. THIẾT KẾ CHI TIẾT & KẾ HOẠCH TRIỂN KHAI](#chương-4-thiết-kế-chi-tiết--kế-hoạch-triển-khai)
   - [4.1 Quy hoạch Địa chỉ IP & Phân chia VLAN toàn hệ thống (IP & VLAN Plan)](#41-quy-hoạch-địa-chỉ-ip--phân-chia-vlan-toàn-hệ-thống-ip--vlan-plan)
   - [4.2 Quy hoạch Tủ mạng phân phối IDF theo tầng & Trung tâm Data Center MDF](#42-quy-hoạch-tủ-mạng-phân-phối-idf-theo-tầng--trung-tâm-data-center-mdf)
   - [4.3 Bảng thống kê thiết bị phần cứng & Ngân sách nguồn cấp PoE (PoE Budget)](#43-bảng-thống-kê-thiết-bị-phần-cứng--ngân-sách-nguồn-cấp-poe-poe-budget)
   - [4.4 Thiết kế Định tuyến & Dịch vụ Mạng cốt lõi (Routing, DHCP, NTP)](#44-thiết-kế-định-tuyến--dịch-vụ-mạng-cốt-lõi-routing-dhcp-ntp)
   - [4.5 Thiết kế An toàn Lớp biên, Switch Access & Ma trận Extended ACLs](#45-thiết-kế-an-toàn-lớp-biên-switch-access--ma-trận-extended-acls)
   - [4.6 Thiết kế Hệ thống Giám sát SIEM Wazuh & Cảnh báo Telegram](#46-thiết-kế-hệ-thống-giám-sát-siem-wazuh--cảnh-báo-telegram)
   - [4.7 Tính sẵn sàng cao & Dự phòng (High Availability & Redundancy)](#47-tính-sẵn-sàng-cao--dự-phòng-high-availability--redundancy)
   - [4.8 Kịch bản Kiểm thử & Diễn tập An ninh Thực nghiệm (Test Cases)](#48-kịch-bản-kiểm-thử--diễn-tập-an-ninh-thực-nghiệm-test-cases)
5. [TỔ CHỨC DỰ ÁN, PHÂN CÔNG NHIỆM VỤ & MA TRẬN RACI](#tổ-chức-dự-án-phân-công-nhiệm-vụ--ma-trận-raci)
6. [DANH MỤC SẢN PHẨM BÀN GIAO & TÀI LIỆU CHUYÊN ĐỀ](#danh-mục-sản-phẩm-bàn-giao--tài-liệu-chuyên-đề)

---

## CHƯƠNG 1. GIỚI THIỆU DOANH NGHIỆP & PHÂN TÍCH NHU CẦU

### 1.1 Giới thiệu quy mô & Mô hình hoạt động
* **Tên doanh nghiệp:** Công ty Cổ phần Công nghệ & Dịch vụ Số TechCorp (Doanh nghiệp công nghệ mẫu).
* **Lĩnh vực hoạt động:** Phát triển phần mềm, gia công ứng dụng di động, giải pháp đám mây, trí tuệ nhân tạo (AI/ML) và phân tích dữ liệu lớn.
* **Cơ sở hạ tầng vật lý:** Khuôn viên gồm **2 Tòa nhà cao tầng** kết nối trục cáp quang nội bộ:
  - **Tòa A (Trung tâm Nghiên cứu & Vận hành Kỹ thuật - 5 Tầng):** 91 nhân sự làm việc, 15 phòng ban chức năng, bao gồm Trung tâm Dữ liệu MDF (A-202) và Trung tâm NOC/SOC (A-404).
  - **Tòa B (Trung tâm Kinh doanh & Dịch vụ Hỗ trợ - 4 Tầng):** 53 nhân sự nghiệp vụ, 11 phòng ban chức năng.
  - **Tổng nhân sự:** **144 nhân sự chính thức** (+ 2 quản trị viên hệ thống tại A-202 = 146 người).
* **Quy mô Endpoints & Hạ tầng:**
  - **144 Máy trạm PC / Workstation** (Dell OptiPlex 7000 / Precision).
  - **25 Switch truy cập phòng** (16 switch Cisco Catalyst C9200L-24P-4X và 9 switch Cisco Catalyst C9200CX-8P-2X2G).
  - **25 Bộ phát sóng Wi-Fi 6** (UniFi U6-Pro) cấp nguồn PoE+.
  - **25 Điện thoại IP Phone** (Cisco 7821 VoIP) hỗ trợ Voice VLAN & QoS.
  - **25 Camera IP an ninh Dome** (Hikvision DS-2CD2143G2-I PoE) cô lập VLAN Giám sát.
  - **25 Hệ thống kiểm soát ra vào cửa** (ZKTeco InBio + Khóa từ + Thẻ Mifare DESFire EV3).
  - **20 Máy in mạng** (HP LaserJet Enterprise M507dn) xác thực in bảo mật.
  - **9 Tủ mạng phân phối tầng (IDF)** + **1 Trung tâm Dữ liệu MDF (A-202)** kết nối cáp quang Single-Mode OS2 10Gbps SFP+.

---

### 1.2 Cơ cấu tổ chức & Bảng phân bổ 26 phòng ban chi tiết (Tòa A & B)
*(Đồng bộ 100% với Bản vẽ thiết kế mặt bằng kỹ thuật [index-v2.html](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/ban%20thiet%20ke%20doanh%20nghiep/index-v2.html))*

#### 🏢 TÒA A - VẬN HÀNH KỸ THUẬT & TRUNG TÂM R&D (5 TẦNG - 15 PHÒNG - 91 NHÂN SỰ)

| STT | Tầng | Mã phòng | Tên bộ phận / Phòng ban | Nhân sự | Phân vùng Zone | Risk Level | VLAN | Dải IP / Subnet | Model Switch Phòng |
| :---: | :---: | :---: | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| 1 | **T1** | `A-101` | Sảnh lễ tân Tòa A | 2 | **Guest** | `Untrusted` | VLAN 100 | `172.16.100.0/24` | Cisco C9200CX-8P-2X2G |
| 2 | **T1** | `A-102` | Phòng bảo vệ / Kiểm soát ra vào | 2 | **Infra** | `High` | VLAN 11 | `192.168.11.0/24` | Cisco C9200CX-8P-2X2G |
| 3 | **T2** | `A-201` | Phòng Lab / Phát triển & Kiểm thử | 10 | **User** | `Medium/High` | VLAN 21 | `192.168.21.0/24` | Cisco C9200L-24P-4X |
| 4 | **T2** | `A-202` | Phòng Máy Chủ Trung Tâm & Data Center (MDF) | 0 *(2 Admin)* | **Infra** | `Critical` | VLAN 2 & 10 | `192.168.10.0/24` *(Srv)*<br>`192.168.2.0/24` *(Mgmt)* | Core Switch / Dual C9500 / FW HA / 2x Rack 42U |
| 5 | **T2** | `A-203` | Phòng hỗ trợ IT (Helpdesk) | 4 | **Infra** | `High` | VLAN 23 | `192.168.23.0/24` | Cisco C9200L-24P-4X |
| 6 | **T3** | `A-301` | Phòng Lập trình Backend | 15 | **User** | `High` | VLAN 31 | `192.168.31.0/24` | Cisco C9200L-24P-4X |
| 7 | **T3** | `A-302` | Phòng Cơ sở dữ liệu / Kỹ sư dữ liệu | 8 | **Restricted** | `Critical` | VLAN 32 | `192.168.32.0/24` | Cisco C9200L-24P-4X |
| 8 | **T3** | `A-303` | Phòng Lập trình Frontend | 15 | **User** | `Medium` | VLAN 33 | `192.168.33.0/24` | Cisco C9200L-24P-4X |
| 9 | **T4** | `A-401` | Phòng Kiểm thử (QA / Tester) | 10 | **User** | `Medium` | VLAN 41 | `192.168.41.0/24` | Cisco C9200L-24P-4X |
| 10 | **T4** | `A-402` | Phòng Thiết kế UI/UX | 6 | **User** | `Medium` | VLAN 42 | `192.168.42.0/24` | Cisco C9200L-24P-4X |
| 11 | **T4** | `A-403` | Phòng DevOps / Quản trị hệ thống | 8 | **Restricted** | `Critical` | VLAN 43 | `192.168.43.0/24` | Cisco C9200L-24P-4X |
| 12 | **T4** | `A-404` | Phòng NOC/SOC (Giám sát & An ninh mạng) | 4 | **Restricted** | `Critical` | VLAN 44 | `192.168.44.0/24` | Cisco C9200L-24P-4X |
| 13 | **T5** | `A-501` | Phòng Quản lý sản phẩm (Product) | 5 | **User** | `Medium` | VLAN 51 | `192.168.51.0/24` | Cisco C9200L-24P-4X |
| 14 | **T5** | `A-502` | Phòng CEO / CTO (Ban Điều hành) | 2 | **Executive** | `High` | VLAN 52 | `192.168.52.0/24` | Cisco C9200CX-8P-2X2G |
| 15 | **T5** | `A-503` | Phòng họp Ban Giám đốc | 0 | **Executive** | `Medium` | VLAN 53 | `192.168.53.0/24` | Cisco C9200CX-8P-2X2G |
| **CỘNG** | | | **Tổng Tòa A (15 phòng)** | **91** | | | | | |

---

#### 🏢 TÒA B - TRUNG TÂM KINH DOANH & DỊCH VỤ KHÁCH HÀNG (4 TẦNG - 11 PHÒNG - 53 NHÂN SỰ)

| STT | Tầng | Mã phòng | Tên bộ phận / Phòng ban | Nhân sự | Phân vùng Zone | Risk Level | VLAN | Dải IP / Subnet | Model Switch Phòng |
| :---: | :---: | :---: | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| 16 | **T1** | `B-101` | Sảnh tiếp khách hàng / Đối tác | 2 | **Guest** | `Untrusted` | VLAN 110 | `172.16.110.0/24` | Cisco C9200CX-8P-2X2G |
| 17 | **T1** | `B-102` | Phòng họp đối tác | 0 | **Guest** | `Untrusted` | VLAN 111 | `172.16.111.0/24` | Cisco C9200CX-8P-2X2G |
| 18 | **T2** | `B-201` | Phòng Kinh doanh (Sales) | 12 | **User** | `Medium` | VLAN 121 | `192.168.121.0/24` | Cisco C9200L-24P-4X |
| 19 | **T2** | `B-202` | Phòng Marketing | 8 | **User** | `Medium` | VLAN 122 | `192.168.122.0/24` | Cisco C9200L-24P-4X |
| 20 | **T3** | `B-301` | Phòng Nhân sự (HR) | 5 | **Restricted** | `High` | VLAN 131 | `192.168.131.0/24` | Cisco C9200L-24P-4X |
| 21 | **T3** | `B-302` | Phòng Kế toán / Tài chính | 6 | **Restricted** | `High` | VLAN 132 | `192.168.132.0/24` | Cisco C9200L-24P-4X |
| 22 | **T3** | `B-303` | Phòng Pháp chế / Hành chính | 3 | **User** | `Medium` | VLAN 133 | `192.168.133.0/24` | Cisco C9200CX-8P-2X2G |
| 23 | **T3** | `B-304` | Phòng Chăm sóc khách hàng (CSKH) | 10 | **User** | `Medium` | VLAN 134 | `192.168.134.0/24` | Cisco C9200L-24P-4X |
| 24 | **T4** | `B-401` | Phòng Triển khai dự án (PM/BA) | 7 | **User** | `Medium` | VLAN 141 | `192.168.141.0/24` | Cisco C9200L-24P-4X |
| 25 | **T4** | `B-402` | Phòng Đào tạo nội bộ | 0 | **User** | `Low` | VLAN 142 | `192.168.142.0/24` | Cisco C9200CX-8P-2X2G |
| 26 | **T4** | `B-403` | Căng tin / Khu nghỉ nhân viên | 0 | **Guest** | `Untrusted` | VLAN 143 | `172.16.143.0/24` | Cisco C9200CX-8P-2X2G |
| **CỘNG** | | | **Tổng Tòa B (11 phòng)** | **53** | | | | | |
| **TỔNG** | | | **TOÀN DOANH NGHIỆP (26 PHÒNG)** | **144** | *(+ 2 Admin Data Center A-202 = 146 nhân sự)* | | | | |

---

### 1.3 Dịch vụ CNTT hiện hành & Mục tiêu an toàn thông tin (CIA Triad)
* **Dịch vụ CNTT nòng cốt:** Cụm Server Farm tập trung tại MDF A-202 gồm Active Directory/DNS/DHCP, Web Portal, Git/CI-CD Server, Database Server (PostgreSQL/MySQL), File Storage NAS/SAN, ERP & CRM nội bộ, Hệ thống Tổng đài VoIP IP-PBX, Hệ thống Wazuh SIEM & Rsyslog Server.
* **Mục tiêu an toàn thông tin cốt lõi (CIA Triad):**
  - **Confidentiality (Tính bí mật):** Phân vùng mạng chi tiết theo 6 Security Zones (`Guest`, `User`, `Restricted`, `Executive`, `Infra/Mgmt`, `Voice`); bảo vệ mã nguồn phần mềm, CSDL khách hàng và dữ liệu tài chính kế toán theo nguyên tắc đặc quyền tối thiểu (*Least Privilege*).
  - **Integrity (Tính toàn vẹn):** Đảm bảo tính toàn vẹn của mã nguồn, file cấu hình hệ thống máy chủ và nhật ký an ninh không bị chỉnh sửa trái phép (giám sát tự động qua *Wazuh FIM - File Integrity Monitoring*).
  - **Availability (Tính sẵn sàng):** Đảm bảo mạng hoạt động 24/7 với đường truyền dự phòng Dual ISP, Firewall HA, Core Switch Stacking, 9 tuyến cáp quang OS2 liên tầng, UPS Online 5000VA và hệ thống cảnh báo sớm tấn công DoS/DDoS qua Telegram Bot.

---

## CHƯƠNG 2. KHẢO SÁT HIỆN TRẠNG & ĐÁNH GIÁ RỦI RO HỆ THỐNG

### 2.1 Hiện trạng hạ tầng mạng truyền thống
* **Hạ tầng mạng phẳng (Flat Network):** Trước khi nâng cấp, mạng nội bộ sử dụng dải IP phẳng chung (`192.168.1.0/24`), không chia VLAN theo phòng ban, thiếu kiểm soát phân quyền.
* **Hệ thống định tuyến & tường lửa yếu kém:** Chỉ có 1 Router Gateway nhà mạng cơ bản, thiếu Next-Gen Firewall (NGFW) chuyên dụng, không có bộ lọc ACLs kiểm soát truy cập giữa các khối nghiệp vụ và khối máy chủ.
* **Quản lý nhật ký phân tán:** Nhật ký an ninh nằm rải rác trên từng máy tính, không có máy chủ lưu trữ Syslog tập trung, thiếu đồng bộ thời gian NTP dẫn đến sai lệch dữ liệu đối soát khi có sự cố.

### 2.2 Phân tích Ma trận Rủi ro & Lỗ hổng (Risk Matrix NIST SP 800-30)

Đánh giá rủi ro an toàn thông tin hệ thống mạng doanh nghiệp theo phương pháp chuẩn **NIST SP 800-30 / ISO 27005**:
$$\text{Mức Độ Rủi Ro (Risk Score)} = \text{Khả Năng Xảy Ra (Likelihood: 1-5)} \times \text{Mức Độ Tác Động (Impact: 1-5)}$$

#### 📊 Bảng Ma Trận Phân Cấp Rủi Ro 5x5

| Khả Năng Xảy Ra (Likelihood) \ Mức Tác Động (Impact) | 1 - Rất Thấp (Negligible) | 2 - Thấp (Minor) | 3 - Vừa (Moderate) | 4 - Cao (Major) | 5 - Nghiêm Trọng (Catastrophic) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **5 - Gần như chắc chắn (Almost Certain)** | Medium (5) | High (10) | High (15) | **Critical (20)** | **Critical (25)** |
| **4 - Rất có thể (Likely)** | Low (4) | Medium (8) | High (12) | High (16) | **Critical (20)** |
| **3 - Có thể xảy ra (Possible)** | Low (3) | Medium (6) | Medium (9) | High (12) | High (15) |
| **2 - Ít có khả năng (Unlikely)** | Low (2) | Low (4) | Medium (6) | Medium (8) | High (10) |
| **1 - Rất hiếm khi (Rare)** | Low (1) | Low (2) | Low (3) | Low (4) | Medium (5) |

---

#### 📋 Bảng Định Danh & Đánh Giá Chi Tiết Rủi Ro Mạng Doanh Nghiệp (Risk Register)

| Mã Rủi Ro | Mối Đe Dọa & Lỗ Hổng Hiện Hữu | Vùng Bị Ảnh Hưởng | Khả Năng (L: 1-5) | Tác Động (I: 1-5) | Điểm & Cấp Độ Rủi Ro | Biện Pháp Kiểm Soát & Giải Pháp Kỹ Thuật |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- |
| **RSK-01** | **Xâm nhập & Đánh cắp CSDL Tối Mật:** Kẻ tấn công từ mạng User/Guest truy cập trái phép cổng CSDL (MySQL/Postgres) | `Data_Zone` (VLAN 32 / VLAN 10) | 3 *(Có thể)* | 5 *(Nghiêm trọng)* | **15 - High / Critical** | Cô lập VLAN 32 & VLAN 10 bằng Extended ACLs; chỉ cho phép Backend Server kết nối cổng DB; kích hoạt giám sát FIM và audit log CSDL trên Wazuh. |
| **RSK-02** | **Tấn công Dò quét Mật khẩu (Brute-force SSH/RDP):** Quét dò mật khẩu máy chủ quản trị / DB Server | `Server Farm` (VLAN 10) | 5 *(Gần như chắc chắn)* | 4 *(Cao)* | **20 - Critical** | Khóa SSH chỉ cho phép Key-based Auth từ VLAN 44 (NOC/SOC) và VLAN 2 (Mgmt); cấu hình Wazuh Active Response tự động chặn IP qua Firewall khi sai pass > 5 lần. |
| **RSK-03** | **Lây lan Mã độc từ Khách & Wi-Fi Vãng lai:** Thiết bị cá nhân mang mã độc kết nối vào mạng sảnh, phòng họp | `Guest_Zone` (VLAN 100, 110, 111, 143) | 4 *(Rất có thể)* | 4 *(Cao)* | **16 - Critical** | Cô lập hoàn toàn các VLAN Guest (Client Isolation + chỉ ra Internet qua NAT, cấm định tuyến sang các VLAN nội bộ). |
| **RSK-04** | **Tấn công Từ chối Dịch vụ (SYN Flood / DoS):** Gửi lượng lớn gói tin TCP SYN làm tê liệt Web Server nội bộ hoặc cổng WAN | `App / Server Farm` (VLAN 10) | 4 *(Rất có thể)* | 4 *(Cao)* | **16 - Critical** | Bật TCP Intercept / SYN Flood Protection trên Firewall; giám sát lưu lượng và cảnh báo qua Telegram Bot khi có lưu lượng bất thường. |
| **RSK-05** | **Cắm thiết bị lạ vào cổng mạng Văn phòng:** Kẻ xấu cắm laptop cá nhân/thiết bị lạ vào Switch sảnh/phòng ban | `User_Zone` / `Infra` | 3 *(Có thể)* | 3 *(Vừa)* | **9 - Medium** | Bật **Port Security** (giới hạn tối đa 2 MAC/cổng, vi phạm sẽ `shutdown`/`restrict`), bật **DHCP Snooping** và **Dynamic ARP Inspection (DAI)**. |
| **RSK-06** | **Quét thăm dò cổng dịch vụ (Port Scanning):** Dò quét phát hiện các cổng mở, lỗ hổng dịch vụ Web/App | `Toàn mạng` | 5 *(Gần như chắc chắn)* | 2 *(Thấp)* | **10 - High** | Thiết lập Wazuh Rule phát hiện kết nối đa cổng bất thường trong thời gian ngắn; phân quyền ACL chặn quét giữa các VLAN. |
| **RSK-07** | **Mất mát & Thiếu nhất quán Nhật ký An toàn:** Sự cố xảy ra nhưng log nằm rải rác, sai lệch thời gian không điều tra được | `Toàn hệ thống` | 4 *(Rất có thể)* | 4 *(Cao)* | **16 - Critical** | Triển khai **Wazuh SIEM + Rsyslog tập trung (RFC 5424)**, bắt buộc đồng bộ thời gian toàn bộ thiết bị qua **máy chủ NTP**. |

---

### 2.3 Yêu cầu chuyển đổi sang hệ thống bảo mật thế hệ mới
1. **Phân vùng an ninh 6 Zones độc lập:** Chia tách chi tiết từng phòng ban thành các VLAN riêng biệt (tổng cộng 26 VLAN phòng ban + 4 VLAN hạ tầng dùng chung).
2. **Triển khai Tường lửa HA & Ma trận Extended ACLs nghiêm ngặt:** Chặn tuyệt đối luồng dữ liệu từ mạng Guest và User thông thường truy cập trực tiếp vào vùng Database (VLAN 32), Quản trị (VLAN 2) và Server Farm (VLAN 10).
3. **Bảo vệ toàn diện Lớp truy cập (Access Layer Security):** Kích hoạt Port Security, DHCP Snooping, DAI, Storm Control và 802.1X trên toàn bộ 25 Switch phòng.
4. **Xây dựng Trung tâm Giám sát SIEM tập trung:** Triển khai **Wazuh All-in-One** kết hợp **Rsyslog (RFC 5424)**, phân tích log thời gian thực và cảnh báo tức thời qua **Telegram Bot**.

---

## CHƯƠNG 3. THIẾT KẾ TỔNG THỂ HỆ THỐNG MẠNG & BẢO MẬT

### 3.1 Kiến trúc mạng phân tầng 3 lớp (Core - Distribution - Access) & Trục quang OS2
* **Core Layer:** Cặp Core Switch Cisco Catalyst 9500 Stacking đặt tại Trung tâm Dữ liệu MDF A-202, kết nối với Cặp Next-Gen Firewall (HA Active/Standby) và 2 Router WAN (Dual ISP Viettel & VNPT) chạy BGP/Static Route.
* **Distribution Layer (9 Tủ IDF theo tầng):**
  - Gồm 9 Tủ mạng phân phối tầng (5 IDF Tòa A: `IDF-A-T1` đến `IDF-A-T5`; 4 IDF Tòa B: `IDF-B-T1` đến `IDF-B-T4`).
  - Mỗi tủ IDF kết nối về Trung tâm Data Center MDF A-202 qua **trục cáp quang Single-Mode OS2 (10G SFP+)** với khay phối quang ODF, Patch Panel Cat6A, PDU đo dòng và UPS Online 3000VA.
* **Access Layer (25 Switch Phòng):**
  - Đặt tại từng phòng ban: Sử dụng **Cisco Catalyst C9200L-24P-4X** cho các phòng đông nhân sự (10-15 máy trạm) và **Cisco Catalyst C9200CX-8P-2X2G** cho các phòng nhỏ (2-5 máy trạm).
  - Cung cấp nguồn PoE+ (IEEE 802.3at) cho Wi-Fi 6 AP, IP Phone, Camera IP và Bộ kiểm soát cửa.

---

### 3.2 Phân vùng an ninh (Security Zones) & Ma trận kiểm soát truy cập (Zone Matrix)

Hệ thống được chia thành **6 Security Zones** tiêu chuẩn:
1. 🔴 **Restricted Zone (Risk: Critical / High):** Phòng CSDL A-302 (VLAN 32), DevOps A-403 (VLAN 43), NOC/SOC A-404 (VLAN 44), Nhân sự B-301 (VLAN 131), Kế toán B-302 (VLAN 132).
2. 🟠 **Infra / Management Zone (Risk: Critical / High):** Quản trị Switch/Router/FW (VLAN 2), Cụm Server Farm MDF A-202 (VLAN 10), Bảo vệ A-102 (VLAN 11), Helpdesk A-203 (VLAN 23), Camera & Khóa cửa IoT (VLAN 12).
3. 🟣 **Executive Zone (Risk: High / Medium):** Phòng CEO/CTO A-502 (VLAN 52), Phòng họp Ban Giám đốc A-503 (VLAN 53).
4. 🟢 **User Zone (Risk: Medium / Low):** Các phòng kỹ thuật R&D (A-201, A-301, A-303, A-401, A-402, A-501) và khối kinh doanh/hỗ trợ (B-201, B-202, B-303, B-304, B-401, B-402).
5. 🔵 **Voice Zone (Risk: Medium):** Phân vùng mạng thoại riêng biệt Tòa A (VLAN 200) và Tòa B (VLAN 210) ưu tiên chất lượng QoS CoS 5 / DSCP EF.
6. ⚪ **Guest Zone (Risk: Untrusted):** Sảnh lễ tân (VLAN 100, 110), Phòng họp đối tác (VLAN 111), Căng tin (VLAN 143).

#### 🔐 BẢNG MA TRẬN TRUY CẬP GIỮA CÁC ZONE (ZONE ACCESS MATRIX)

| Zone Nguồn (Source) | Được Phép Truy Cập Đến (Destination) | Giới Hạn & Ràng Buộc Bảo Mật |
| :--- | :--- | :--- |
| **Guest Zone** | Internet Only (NAT Gateway) | Chặn tuyệt đối mọi VLAN nội bộ (ACL Deny `192.168.0.0/16`, `172.16.0.0/12`). Bật Client Isolation trên AP. |
| **User Zone** | Internet, Server Farm (VLAN 10 theo port dịch vụ Web/App), Voice VLAN | Chặn truy cập trực tiếp vào VLAN Restricted (32, 43, 44, 131, 132) và Management (VLAN 2). |
| **Restricted Zone** | Server Farm (VLAN 10 cổng chuyên biệt), Internet qua Secure Proxy | Chỉ tiếp nhận kết nối quản trị từ Bastion Host / NOC-SOC (A-404). Cô lập lẫn nhau giữa các phòng. |
| **Executive Zone** | Internet, Server Farm (VLAN 10 theo dịch vụ), Mail/ERP | Được bảo vệ nghiêm ngặt, chặn truy cập trái phép từ User và Guest Zone. |
| **Infra / Mgmt Zone** | Toàn bộ thiết bị mạng & máy chủ | Chỉ cho phép truy cập từ Console OOB / Trạm NOC-SOC (A-404) & Helpdesk (A-203), bắt buộc xác thực MFA & SSHv2/HTTPS. |
| **Voice Zone** | Máy chủ IP-PBX (VLAN 10) | Tự động nhận Voice VLAN qua LLDP-MED / CDP; cô lập với dữ liệu máy trạm Data VLAN. |

---

### 3.3 Sơ đồ kiến trúc logic & Luồng dữ liệu giám sát SIEM / SOC

```mermaid
flowchart TB
    subgraph WAN_EDGE["🌐 LỚP BIÊN MẠNG & INTERNET GATEWAY"]
        ISP1["ISP 1: Viettel (FTTH 1Gbps / BGP)"]
        ISP2["ISP 2: VNPT (FTTH 1Gbps / Backup)"]
        EDGE_FW["Cặp Next-Gen Firewall (HA Active/Standby)"]
        ISP1 === EDGE_FW
        ISP2 === EDGE_FW
    end

    subgraph MDF_CENTER["🏢 TRUNG TÂM DỮ LIỆU MDF (A-202 - TÒA A TẦNG 2)"]
        CORE_SW["Cặp Core Switch Cisco Catalyst 9500 (Stacking / Inter-VLAN Routing)"]
        subgraph SERVER_FARM["Cụm Server Farm (VLAN 10: 192.168.10.0/24)"]
            SRV_AD["AD / DNS / DHCP / NTP"]
            SRV_APP["Web Portal & Git CI/CD"]
            SRV_DB["Database (Postgres / MySQL)"]
            SRV_PBX["VoIP IP-PBX Server"]
            SRV_SIEM["🛡️ Wazuh SIEM All-in-One"]
            SRV_RSYS["Rsyslog Server (UDP 514)"]
        end
        MGMT_VLAN["VLAN 2: Management (192.168.2.0/24)"]
        IOT_VLAN["VLAN 12: Camera & Door Controller (192.168.12.0/24)"]
        EDGE_FW === CORE_SW
        CORE_SW --- SERVER_FARM
        CORE_SW --- MGMT_VLAN
        CORE_SW --- IOT_VLAN
    end

    subgraph TOA_A_IDFS["🏢 TRỤC IDF TÒA A (5 TẦNG - CÁP QUANG SINGLE-MODE OS2 10G)"]
        IDF_A1["IDF-A-T1 (Tầng 1): A-101 (Guest 100), A-102 (Infra 11)"]
        IDF_A2["IDF-A-T2 (Tầng 2): A-201 (User 21), A-202 (MDF), A-203 (Infra 23)"]
        IDF_A3["IDF-A-T3 (Tầng 3): A-301 (User 31), A-302 (Restricted 32), A-303 (User 33)"]
        IDF_A4["IDF-A-T4 (Tầng 4): A-401 (User 41), A-402 (User 42), A-403 (DevOps 43), A-404 (SOC 44)"]
        IDF_A5["IDF-A-T5 (Tầng 5): A-501 (User 51), A-502 (Exec 52), A-503 (Exec 53)"]
    end

    subgraph TOA_B_IDFS["🏢 TRỤC IDF TÒA B (4 TẦNG - CÁP QUANG SINGLE-MODE OS2 10G)"]
        IDF_B1["IDF-B-T1 (Tầng 1): B-101 (Guest 110), B-102 (Guest 111)"]
        IDF_B2["IDF-B-T2 (Tầng 2): B-201 (User 121), B-202 (User 122)"]
        IDF_B3["IDF-B-T3 (Tầng 3): B-301 (HR 131), B-302 (Kế toán 132), B-303 (Pháp chế 133), B-304 (CSKH 134)"]
        IDF_B4["IDF-B-T4 (Tầng 4): B-401 (PM/BA 141), B-402 (Đào tạo 142), B-403 (Canteen 143)"]
    end

    CORE_SW ==="Quang OS2 10G"=== IDF_A1
    CORE_SW ==="Quang OS2 10G"=== IDF_A2
    CORE_SW ==="Quang OS2 10G"=== IDF_A3
    CORE_SW ==="Quang OS2 10G"=== IDF_A4
    CORE_SW ==="Quang OS2 10G"=== IDF_A5

    CORE_SW ==="Quang OS2 10G"=== IDF_B1
    CORE_SW ==="Quang OS2 10G"=== IDF_B2
    CORE_SW ==="Quang OS2 10G"=== IDF_B3
    CORE_SW ==="Quang OS2 10G"=== IDF_B4

    subgraph SOC_MONITORING["🛡️ LUỒNG GIÁM SÁT AN NINH SIEM / SOC (WAZUH)"]
        TEL_BOT["Telegram Alert Bot (Rule Level ≥ 8)"]
        SOC_WS["Trạm Giám sát SOC (A-404) Video Wall 4x 55'"]
        EDGE_FW -.->|"1. Syslog ACL Drop / Traffic"| SRV_RSYS
        CORE_SW -.->|"1. Syslog Port Sec / UpDown"| SRV_RSYS
        IDF_A1 & IDF_A3 & IDF_A4 & IDF_B3 -.->|"1. Switch Syslog"| SRV_RSYS
        SRV_APP & SRV_DB -.->|"2. Wazuh Agent (FIM, Auth Log)"| SRV_SIEM
        SRV_RSYS --> SRV_SIEM
        SRV_SIEM -->|"3. Gửi Cảnh Báo Khẩn"| TEL_BOT
        SRV_SIEM -->|"4. Trực Quan Hóa Dashboard"| SOC_WS
    end
```

---

## CHƯƠNG 4. THIẾT KẾ CHI TIẾT & KẾ HOẠCH TRIỂN KHAI

### 4.1 Quy hoạch Địa chỉ IP & Phân chia VLAN toàn hệ thống (IP & VLAN Plan)

Hệ thống được quy hoạch theo chuẩn phân đoạn chuyên nghiệp với **30 VLANs** độc lập:

| STT | VLAN ID | Tên VLAN | Phân Vùng Zone | Phạm Vi Áp Dụng / Phòng Ban | Dải Mạng IP / Subnet | Default Gateway | Dải Cấp Phát DHCP | Mức Rủi Ro |
| :---: | :---: | :--- | :---: | :--- | :---: | :---: | :---: | :---: |
| **A** | **HẠ TẦNG DÙNG CHUNG (SHARED INFRASTRUCTURE)** | | | | | | |
| 1 | **VLAN 2** | `Management` | **Infra/Mgmt** | Quản trị 25 Switch phòng, Core Switch, Router, FW, PDU, UPS | `192.168.2.0/24` | `192.168.2.1` | IP Tĩnh (.101 - .126) | `Critical` |
| 2 | **VLAN 10** | `Server_Farm` | **Infra** | Cụm máy chủ MDF A-202 (AD, Web, DB, Mail, Wazuh, Rsyslog) | `192.168.10.0/24` | `192.168.10.1` | IP Tĩnh (.10 - .50) | `Critical` |
| 3 | **VLAN 12** | `Security_IoT` | **Infra** | 25 Camera IP Hikvision & 25 Bộ kiểm soát cửa ZKTeco toàn tòa nhà | `192.168.12.0/24` | `192.168.12.1` | `192.168.12.10 - .200` | `High` |
| 4 | **VLAN 200** | `Voice_Toa_A` | **Voice** | Điện thoại IP Cisco 7821 các phòng Tòa A | `192.168.200.0/24` | `192.168.200.1` | `192.168.200.10 - .200` | `Medium` |
| 5 | **VLAN 210** | `Voice_Toa_B` | **Voice** | Điện thoại IP Cisco 7821 các phòng Tòa B | `192.168.210.0/24` | `192.168.210.1` | `192.168.210.10 - .200` | `Medium` |
| **B** | **TÒA A - VẬN HÀNH KỸ THUẬT (15 PHÒNG)** | | | | | | |
| 6 | **VLAN 100** | `Guest_A101` | **Guest** | Sảnh lễ tân Tòa A (A-101 - 2 Nhân sự) | `172.16.100.0/24` | `172.16.100.1` | `172.16.100.10 - .250` | `Untrusted` |
| 7 | **VLAN 11** | `Security_A102`| **Infra** | Phòng bảo vệ / Kiểm soát ra vào (A-102 - 2 Nhân sự) | `192.168.11.0/24` | `192.168.11.1` | `192.168.11.10 - .50` | `High` |
| 8 | **VLAN 21** | `Lab_DevTest` | **User** | Phòng Lab / Phát triển & Kiểm thử (A-201 - 10 Nhân sự) | `192.168.21.0/24` | `192.168.21.1` | `192.168.21.10 - .100` | `Medium/High` |
| 9 | **VLAN 23** | `IT_Helpdesk` | **Infra** | Phòng hỗ trợ IT nội bộ (A-203 - 4 Nhân sự) | `192.168.23.0/24` | `192.168.23.1` | `192.168.23.10 - .50` | `High` |
| 10 | **VLAN 31** | `Dev_Backend` | **User** | Phòng Lập trình Backend (A-301 - 15 Nhân sự) | `192.168.31.0/24` | `192.168.31.1` | `192.168.31.10 - .100` | `High` |
| 11 | **VLAN 32** | `Data_Engineer`| **Restricted** | Phòng Cơ sở dữ liệu / Kỹ sư dữ liệu (A-302 - 8 Nhân sự) | `192.168.32.0/24` | `192.168.32.1` | `192.168.32.10 - .50` | `Critical` |
| 12 | **VLAN 33** | `Dev_Frontend` | **User** | Phòng Lập trình Frontend (A-303 - 15 Nhân sự) | `192.168.33.0/24` | `192.168.33.1` | `192.168.33.10 - .100` | `Medium` |
| 13 | **VLAN 41** | `QA_Tester` | **User** | Phòng Kiểm thử QA / Tester (A-401 - 10 Nhân sự) | `192.168.41.0/24` | `192.168.41.1` | `192.168.41.10 - .100` | `Medium` |
| 14 | **VLAN 42** | `UI_UX_Design` | **User** | Phòng Thiết kế UI/UX (A-402 - 6 Nhân sự) | `192.168.42.0/24` | `192.168.42.1` | `192.168.42.10 - .50` | `Medium` |
| 15 | **VLAN 43** | `DevOps_Admin` | **Restricted** | Phòng DevOps / SysAdmin (A-403 - 8 Nhân sự) | `192.168.43.0/24` | `192.168.43.1` | `192.168.43.10 - .50` | `Critical` |
| 16 | **VLAN 44** | `NOC_SOC_Zone` | **Restricted** | Phòng NOC/SOC Giám sát & An ninh (A-404 - 4 Nhân sự) | `192.168.44.0/24` | `192.168.44.1` | `192.168.44.10 - .50` | `Critical` |
| 17 | **VLAN 51** | `Product_PM` | **User** | Phòng Quản lý sản phẩm (A-501 - 5 Nhân sự) | `192.168.51.0/24` | `192.168.51.1` | `192.168.51.10 - .50` | `Medium` |
| 18 | **VLAN 52** | `Executive_CEO`| **Executive** | Phòng CEO / CTO (A-502 - 2 Nhân sự) | `192.168.52.0/24` | `192.168.52.1` | `192.168.52.10 - .30` | `High` |
| 19 | **VLAN 53** | `Board_Meeting`| **Executive** | Phòng họp Ban Giám đốc (A-503 - Phòng họp) | `192.168.53.0/24` | `192.168.53.1` | `192.168.53.10 - .50` | `Medium` |
| **C** | **TÒA B - KINH DOANH & HỖ TRỢ (11 PHÒNG)** | | | | | | |
| 20 | **VLAN 110** | `Guest_B101` | **Guest** | Sảnh tiếp khách hàng / Đối tác (B-101 - 2 Nhân sự) | `172.16.110.0/24` | `172.16.110.1` | `172.16.110.10 - .250` | `Untrusted` |
| 21 | **VLAN 111** | `Meeting_B102` | **Guest** | Phòng họp đối tác (B-102 - Phòng họp) | `172.16.111.0/24` | `172.16.111.1` | `172.16.111.10 - .100` | `Untrusted` |
| 22 | **VLAN 121** | `Sales_Dept` | **User** | Phòng Kinh doanh (B-201 - 12 Nhân sự) | `192.168.121.0/24` | `192.168.121.1` | `192.168.121.10 - .100` | `Medium` |
| 23 | **VLAN 122** | `Marketing_Dept`| **User** | Phòng Marketing (B-202 - 8 Nhân sự) | `192.168.122.0/24` | `192.168.122.1` | `192.168.122.10 - .100` | `Medium` |
| 24 | **VLAN 131** | `HR_Dept` | **Restricted** | Phòng Nhân sự (B-301 - 5 Nhân sự) | `192.168.131.0/24` | `192.168.131.1` | `192.168.131.10 - .50` | `High` |
| 25 | **VLAN 132** | `Finance_Acc` | **Restricted** | Phòng Kế toán / Tài chính (B-302 - 6 Nhân sự) | `192.168.132.0/24` | `192.168.132.1` | `192.168.132.10 - .50` | `High` |
| 26 | **VLAN 133** | `Legal_Admin` | **User** | Phòng Pháp chế / Hành chính (B-303 - 3 Nhân sự) | `192.168.133.0/24` | `192.168.133.1` | `192.168.133.10 - .50` | `Medium` |
| 27 | **VLAN 134** | `CS_Support` | **User** | Phòng Chăm sóc khách hàng (B-304 - 10 Nhân sự) | `192.168.134.0/24` | `192.168.134.1` | `192.168.134.10 - .100` | `Medium` |
| 28 | **VLAN 141** | `Project_PM_BA`| **User** | Phòng Triển khai dự án (B-401 - 7 Nhân sự) | `192.168.141.0/24` | `192.168.141.1` | `192.168.141.10 - .50` | `Medium` |
| 29 | **VLAN 142** | `Training_Room`| **User** | Phòng Đào tạo nội bộ (B-402 - Phòng đào tạo) | `192.168.142.0/24` | `192.168.142.1` | `192.168.142.10 - .100` | `Low` |
| 30 | **VLAN 143** | `Canteen_Rest` | **Guest** | Căng tin / Khu nghỉ nhân viên (B-403) | `172.16.143.0/24` | `172.16.143.1` | `172.16.143.10 - .250` | `Untrusted` |

---

### 4.2 Quy hoạch Tủ mạng phân phối IDF theo tầng & Trung tâm Data Center MDF

Toàn bộ hệ thống cáp mạng ngang (Cat6A SFTP) từ các phòng ban được gom về **9 Tủ IDF theo tầng** và kết nối về **Trung tâm MDF A-202** qua các tuyến cáp quang Single-Mode OS2 10G SFP+:

| Mã Tủ Mạng | Vị Trí Lắp Đặt | Loại Tủ & Nguồn Cấp | Uplink Về Trung Tâm MDF | Số Phòng Phụ Trách | Danh Sách Phòng Phụ Trách |
| :---: | :--- | :--- | :--- | :---: | :--- |
| **MDF A-202** | Tòa A - Tầng 2 (Phòng A-202) | 2x Rack 42U 1000mm + 2x UPS Online 5kVA | Trung tâm điều phối lõi (Core) | 1 | Phòng Data Center Trung tâm |
| **IDF-A-T1** | Tòa A - Tầng 1 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 2 | `A-101`, `A-102` |
| **IDF-A-T2** | Tòa A - Tầng 2 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 3 | `A-201`, `A-202`, `A-203` |
| **IDF-A-T3** | Tòa A - Tầng 3 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 3 | `A-301`, `A-302`, `A-303` |
| **IDF-A-T4** | Tòa A - Tầng 4 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 4 | `A-401`, `A-402`, `A-403`, `A-404` |
| **IDF-A-T5** | Tòa A - Tầng 5 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 3 | `A-501`, `A-502`, `A-503` |
| **IDF-B-T1** | Tòa B - Tầng 1 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 2 | `B-101`, `B-102` |
| **IDF-B-T2** | Tòa B - Tầng 2 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 2 | `B-201`, `B-202` |
| **IDF-B-T3** | Tòa B - Tầng 3 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 4 | `B-301`, `B-302`, `B-303`, `B-304` |
| **IDF-B-T4** | Tòa B - Tầng 4 (Hộp kỹ thuật tầng) | Rack 42U + Patch Panel Cat6A + UPS 3kVA | Cáp quang OS2 LC-LC 10G SFP+ | 3 | `B-401`, `B-402`, `B-403` |

---

### 4.3 Bảng thống kê thiết bị phần cứng & Ngân sách nguồn cấp PoE (PoE Budget)

#### 📦 Tổng hợp thiết bị toàn hệ thống:
1. **Switch Cisco Catalyst C9200L-24P-4X (24 cổng PoE+ 370W, 4x 10G SFP+):** **16 bộ** (dành cho 16 phòng quy mô vừa và lớn).
2. **Switch Cisco Catalyst C9200CX-8P-2X2G (8 cổng PoE+ 125W, 2x 10G SFP+):** **9 bộ** (dành cho 9 phòng quy mô nhỏ).
3. **Bộ phát Wi-Fi 6 UniFi U6-Pro (PoE+ 30W Class 4):** **25 bộ**.
4. **Điện thoại IP Cisco 7821 VoIP (PoE 15.4W Class 3):** **25 bộ**.
5. **Camera IP Hikvision DS-2CD2143G2-I (PoE 15.4W Class 3):** **25 bộ**.
6. **Bộ điều khiển kiểm soát cửa ZKTeco InBio (PoE 15.4W Class 3):** **25 bộ**.
7. **Máy trạm PC / Workstation Dell OptiPlex 7000 / Precision:** **144 bộ**.
8. **Máy in mạng HP LaserJet Enterprise M507dn:** **20 bộ**.

#### ⚡ Bảng tính toán tải nguồn PoE tại từng phòng ban (PoE Budget):

| Mã Phòng | Model Switch Phòng Được Trang Bị | Tổng Công Suất PoE Switch (Budget) | Tải PoE Ước Tính (1x AP + 1x IP Phone + 1x CAM + 1x Door) | Tỷ Lệ Sử Dụng Nguồn PoE (%) |
| :---: | :--- | :---: | :---: | :---: |
| `A-101` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W (30W + 3x 10.3W) | 48.8% *(An toàn)* |
| `A-102` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W | 48.8% *(An toàn)* |
| `A-201` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-203` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-301` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-302` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-303` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-401` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-402` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-403` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-404` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-501` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `A-502` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W | 48.8% *(An toàn)* |
| `A-503` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W | 48.8% *(An toàn)* |
| `B-101` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W | 48.8% *(An toàn)* |
| `B-102` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W | 48.8% *(An toàn)* |
| `B-201` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `B-202` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `B-301` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `B-302` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `B-303` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W | 48.8% *(An toàn)* |
| `B-304` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `B-401` | Cisco Catalyst C9200L-24P-4X | **370W** | ~61W | 16.5% *(Rất an toàn)* |
| `B-402` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W | 48.8% *(An toàn)* |
| `B-403` | Cisco Catalyst C9200CX-8P-2X2G | **125W** | ~61W | 48.8% *(An toàn)* |

---

### 4.4 Thiết kế Định tuyến & Dịch vụ Mạng cốt lõi (Routing, DHCP, NTP)
1. **Định tuyến:** Triển khai Inter-VLAN Routing trên Core Switch Cisco 9500 kết hợp định tuyến OSPF (Process ID 1, Area 0 với MD5 Authentication) trên mạng lõi và Static Route dự phòng kép ra cổng Next-Gen Firewall / Dual WAN.
2. **Dịch vụ DHCP:** Cấu hình DHCP Server tập trung trên Windows Server / Linux Server tại MDF A-202 (`192.168.10.10`), chuyển tiếp gói tin bằng `ip helper-address 192.168.10.10` trên từng SVI VLAN interface. Cấu hình Lease Time: 2 giờ cho Guest VLAN và 24 giờ cho Office VLAN.
3. **Đồng bộ Thời gian Mạng NTP (Network Time Protocol):**
   > [!IMPORTANT]
   > Tất cả 25 Switch phòng, Core Switch, Router, Firewall, Server Linux/Windows và Wazuh SIEM bắt buộc đồng bộ thời gian từ máy chủ NTP nội bộ (`192.168.10.10` - Stratum 2) để đảm bảo chuỗi thời gian (Timestamps) của Log an ninh trên Wazuh SIEM đồng nhất 100%.

---

### 4.5 Thiết kế An toàn Lớp biên, Switch Access & Ma trận Extended ACLs
* **Port Security:** Áp dụng trên toàn bộ cổng Access kết nối PC và Máy in:
  ```cisco
  switchport mode access
  switchport port-security
  switchport port-security maximum 2
  switchport port-security violation restrict
  switchport port-security mac-address sticky
  ```
* **DHCP Snooping & Dynamic ARP Inspection (DAI):**
  ```cisco
  ip dhcp snooping
  ip dhcp snooping vlan 2,10-53,100-143,200,210
  ip arp inspection vlan 2,10-53,100-143,200,210
  ```
* **Ma trận Extended ACLs tiêu biểu trên Core/Distribution:**
  - `ACL_GUEST`:
    ```cisco
    ip access-list extended ACL_GUEST
     permit udp any eq bootpc any eq bootps
     permit udp any any eq domain
     deny   ip 172.16.0.0 0.0.255.255 192.168.0.0 0.0.255.255
     deny   ip 172.16.0.0 0.0.255.255 172.16.0.0 0.0.255.255
     permit ip 172.16.0.0 0.0.255.255 any
    ```
  - `ACL_RESTRICTED_DATA (VLAN 32 / VLAN 10)`:
    ```cisco
    ip access-list extended ACL_PROTECT_DATA
     permit tcp 192.168.31.0 0.0.0.255 192.168.10.30 0.0.0.0 eq 5432
     permit tcp 192.168.32.0 0.0.0.255 192.168.10.30 0.0.0.0 eq 5432
     permit tcp 192.168.44.0 0.0.0.255 192.168.10.30 0.0.0.0 eq 22
     deny   ip any 192.168.10.30 0.0.0.0 log
     deny   ip any 192.168.32.0 0.0.0.255 log
     permit ip any any
    ```

---

### 4.6 Thiết kế Hệ thống Giám sát SIEM Wazuh & Cảnh báo Telegram
1. **Thu thập Syslog Thiết bị mạng (RFC 5424):** Cấu hình toàn bộ Switch Cisco IOS, Firewall đẩy Syslog về Rsyslog Server (`192.168.10.50:514 UDP`):
   ```cisco
   service timestamps log datetime msec show-timezone
   logging host 192.168.10.50 transport udp port 514
   logging trap informational
   logging facility local6
   ```
2. **Wazuh Agent trên Máy chủ & Trạm nhạy cảm:** Cài đặt Wazuh Agent trên các máy chủ Web/App, Database (VLAN 10), máy trạm Kế toán (VLAN 132), Nhân sự (VLAN 131) và DevOps (VLAN 43) để kích hoạt FIM, Rootcheck và giám sát xác thực.
3. **Tự động hóa cảnh báo qua Telegram Bot:**
   - Tạo Script tích hợp `custom-telegram.py` trong `/var/ossec/integrations/` trên Wazuh Server.
   - Khi Wazuh kích hoạt Rule có `level >= 8` (Phát hiện Port Scanning, Brute-force SSH, Vi phạm Port Security, ACL Drop truy cập Database), hệ thống tự động gửi tin nhắn báo động định dạng Markdown kèm IP nguồn, IP đích, mã phòng vi phạm về nhóm Telegram của đội ngũ SOC.

---

### 4.7 Tính sẵn sàng cao & Dự phòng (High Availability & Redundancy)
* **Dự phòng Gateway & Core:** Chạy HSRP / VRRP giữa cặp Switch Core Cisco 9500 với địa chỉ Virtual IP đóng vai trò Default Gateway cho từng VLAN.
* **Dự phòng liên kết LACP (802.3ad):** Gộp 2x đường quang 10G SFP+ (EtherChannel 20Gbps) giữa Core Switch và các IDF phân phối tầng.
* **Hệ thống Nguồn điện dự phòng:** Trang bị 2 Bộ lưu điện UPS Online 5000VA đấu song song N+1 tại MDF A-202 và máy phát điện dự phòng tự động ATS chuyển mạch < 10 giây khi mất điện lưới.

---

### 4.8 Kịch bản Kiểm thử & Diễn tập An ninh Thực nghiệm (Test Cases)

| Mã TC | Phân Vùng Mục Tiêu | Kịch Bản Thử Nghiệm Chi Tiết | Công Cụ & Thao Tác | Kết Quả Ghi Nhận Trên Wazuh SIEM & Telegram | Người Phụ Trách |
| :---: | :--- | :--- | :--- | :--- | :---: |
| **TC-01** | `Management (VLAN 2)` | Trạng thái cổng Switch thay đổi bất thường (Up/Down) | `shutdown` / `no shutdown` cổng trên Switch SW-A-101 | Wazuh SIEM nhận Syslog `LINK-3-UPDOWN` tức thời | Minh Trí |
| **TC-02** | `Guest` ➡️ `Data_Zone` | Chặn truy cập trái phép từ VLAN 100/110 sang Database | Máy từ VLAN 100 ping hoặc scan cổng DB `192.168.10.30` | Bị ACL Drop; Switch sinh log `SEC-6-IPACCESSLOGP` gửi về SIEM | Minh Trí |
| **TC-03** | `Server Farm (VLAN 10)` | Quét dò tìm cổng dịch vụ máy chủ Web/App | Dùng Kali Linux chạy `nmap -sS -p 1-1000 192.168.10.20` | Wazuh SIEM kích hoạt Rule ID `100100` (Port Scan detected) | Thành Đạt |
| **TC-04** | `Server Farm (VLAN 10)` | Tấn công dò quét mật khẩu SSH máy chủ quản trị | Dùng `hydra -l root -P pass.txt ssh://192.168.10.20` | Kích hoạt cảnh báo *SSH authentication failed / Brute-force* | Thành Đạt |
| **TC-05** | `Kế toán (VLAN 132)` | Đăng nhập sai liên tiếp trên máy tính Kế toán B-302 | Nhập sai mật khẩu Windows 5 lần liên tiếp trên PC Kế toán | Wazuh Agent thu thập Windows Event ID 4625 (Logon Failure) | Tuấn Anh |
| **TC-06** | `Toàn hệ thống` | Tự động bắn cảnh báo sự cố nghiêm trọng qua Bot | Tấn công tạo sự kiện kích hoạt Rule Level ≥ 8 | Bot Telegram lập tức đẩy tin nhắn báo động vào nhóm SOC | Anh Xuân |
| **TC-07** | `Phòng NOC/SOC (A-404)` | Trực quan hóa dữ liệu giám sát an ninh toàn doanh nghiệp | Truy cập Wazuh Dashboard trên Video Wall 4x 55' | Hiển thị bản đồ phân bổ sự cố, Top IP vi phạm, trạng thái 25 Switch | Tuấn Anh |

---

## TỔ CHỨC DỰ ÁN, PHÂN CÔNG NHIỆM VỤ & MA TRẬN RACI

### 👑 Phân công nhiệm vụ chi tiết từng thành viên:
1. **Nguyễn Anh Xuân (Trưởng nhóm - SIEM & SOC Lead):**
   - Chủ trì kiến trúc an toàn hệ thống, triển khai Máy chủ Wazuh All-in-One, cấu hình tiếp nhận Syslog RFC 5424, viết tập luật tương quan và cấu hình tích hợp Bot Telegram báo động.
2. **Nguyễn Phúc Vượng (Kỹ sư Mạng 1 - Topology & Routing):**
   - Thiết kế sơ đồ mạng 2 tòa nhà A & B trên Packet Tracer / EVE-NG, cấu hình Inter-VLAN Routing, định tuyến Core, cấu hình dịch vụ DHCP theo 30 phân vùng VLAN và NTP Server đồng bộ thời gian.
3. **Nguyễn Minh Trí (Kỹ sư Mạng 2 - Bảo mật Thiết bị & Syslog Forwarding):**
   - Cấu hình Port Security, DHCP Snooping, Dynamic ARP Inspection trên 25 Switch truy cập; thiết lập bộ lọc phân vùng Extended ACLs cô lập Data/Infra; cấu hình đẩy Syslog thiết bị mạng về Wazuh.
4. **Nguyễn Thành Đạt (Chuyên viên ATTT 1 - Giám sát Máy chủ & Pentest):**
   - Cài đặt Wazuh Agent trên máy chủ Linux (Web/DB), bật FIM; thực hiện các kịch bản diễn tập tấn công bằng Kali Linux (Nmap Port Scan, Hydra Brute-force SSH, SYN Flood DoS); đối soát Rule ID cảnh báo trên SIEM.
5. **Lê Tuấn Anh (Chuyên viên ATTT 2 - An ninh Máy trạm & Báo cáo):**
   - Cài đặt Wazuh Agent trên máy trạm Windows khối văn phòng; thử nghiệm các hành vi bất thường trên Endpoint (đăng nhập sai, tạo user); thiết kế Dashboard trực quan hóa SOC và hoàn thiện tài liệu Báo cáo Word & Slide bảo vệ.

---

### 📊 Ma trận Trách nhiệm (RACI Matrix)

| Giai đoạn & Hạng mục công việc | Nguyễn Anh Xuân (Lead) | Nguyễn Phúc Vượng (Net 1) | Nguyễn Minh Trí (Net 2) | Nguyễn Thành Đạt (Security 1) | Lê Tuấn Anh (Security 2) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| 1. Khảo sát phân vùng 26 phòng ban 2 Tòa A-B & Thiết kế Sơ đồ mạng | C | **R / A** | C | I | I |
| 2. Cấu hình Inter-VLAN Routing, DHCP theo 30 VLAN & Đồng bộ NTP | C | **R / A** | C | I | I |
| 3. Cấu hình Extended ACLs, Port Security & Đẩy Syslog từ 25 Switch/Core | C | C | **R / A** | I | I |
| 4. Cài đặt Máy chủ Wazuh SIEM & Tích hợp Bot Cảnh báo Telegram | **R / A** | I | C | C | I |
| 5. Cài Wazuh Agent Máy chủ Linux & Diễn tập Tấn công (Nmap, Hydra) | C | I | I | **R / A** | C |
| 6. Cài Wazuh Agent Máy trạm Windows & Test hành vi nội bộ (Logon Fail) | C | I | I | **A** | **R** |
| 7. Thiết kế Dashboard SOC trực quan hóa & Phân tích Sự cố | **A** | I | I | C | **R** |
| 8. Soạn thảo Báo cáo Đồ án Tổng thể & Thiết kế Slide Bảo vệ | **A** | C | C | C | **R** |

> **Quy ước:** **R** (*Responsible*) = Người trực tiếp thực hiện; **A** (*Accountable*) = Người kiểm tra, nghiệm thu; **C** (*Consulted*) = Người hỗ trợ, cố vấn; **I** (*Informed*) = Người theo dõi tiến độ.

---

## DANH MỤC SẢN PHẨM BÀN GIAO & TÀI LIỆU CHUYÊN ĐỀ

### 📦 Sản phẩm bàn giao nghiệm thu đồ án:
1. **Báo cáo chuyên đề hoàn chỉnh (Đóng quyển 50 ÷ 70 trang):** Soạn thảo chuẩn cấu trúc 4 Chương theo đề cương môn học.
2. **Slide thuyết trình bảo vệ (10 ÷ 15 slide):** Trình bày trực quan, đầy đủ luận điểm kiến trúc và kết quả thử nghiệm.
3. **Bản vẽ thiết kế mặt bằng & sơ đồ chi tiết 26 phòng ban:** File [ban thiet ke doanh nghiep/index-v2.html](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/ban%20thiet%20ke%20doanh%20nghiep/index-v2.html) kèm bản in PDF kỹ thuật.
4. **Bảng địa chỉ IP & VLAN Plan:** Bảng quy hoạch IP chi tiết 30 VLANs kèm dải cấp phát DHCP và PoE Budget.
5. **Danh sách chính sách an toàn (Security Policy Catalog):** Tập hợp bảng ACLs, cấu hình Port Security và Rule cảnh báo Wazuh.
6. **File mô phỏng thực nghiệm:** File Cisco Packet Tracer / EVE-NG / Docker Compose sẵn sàng chạy demo.
7. **Nhật ký làm việc & Video Demo:** Video ghi hình diễn tập tấn công và phản ứng tức thời của hệ thống SIEM & Bot Telegram.

---

### 📚 Tài liệu Giáo trình Kỹ thuật Chuyên sâu (7 Chuyên đề):
Toàn bộ hướng dẫn lý thuyết và cấu hình mẫu dòng lệnh chi tiết được lưu trữ tại thư mục [Documents/](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents):

1. [01_Network_Fundamentals.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/01_Network_Fundamentals.md)
   - **Nền tảng mạng & Kiến trúc giao thức:** Mô hình OSI/TCP-IP trong môi trường doanh nghiệp, Ánh xạ dữ liệu giám sát vào từng tầng mạng, Subnetting & Quy hoạch VLSM cho 2 tòa nhà (26 phòng ban).
2. [02_Network_Infrastructure.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/02_Network_Infrastructure.md)
   - **Kiến trúc hạ tầng thiết bị mạng:** Router (Control/Data Plane), Switch L2/L3 phân tầng, Kỹ thuật bảo vệ Access Layer (Port Security, DHCP Snooping, DAI, SPAN Port, PoE Budget).
3. [03_Network_Segmentation.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/03_Network_Segmentation.md)
   - **Phân đoạn mạng & Thiết kế VLAN phân vùng rủi ro:** Chuẩn 802.1Q Trunking, Inter-VLAN Routing, Ma trận kiểm soát truy cập (ACLs) cô lập vùng tối mật (Data Zone, Infra Zone, Restricted Zone).
4. [04_Core_Network_Services.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/04_Core_Network_Services.md)
   - **Dịch vụ mạng cốt lõi & Giám sát:** DHCP Audit Trail, DNS Security, NAT/PAT Log Correlation, **NTP Synchronization** (Đồng bộ thời gian chuẩn cho SIEM).
5. [05_Routing_Protocols.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/05_Routing_Protocols.md)
   - **Giao thức định tuyến & Bảo mật:** Static Route, Dynamic Routing OSPF với xác thực MD5, Giám sát và phát hiện bất thường định tuyến qua Syslog/SIEM.
6. [06_Network_Security.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/06_Network_Security.md)
   - **An toàn mạng & Phòng thủ theo chiều sâu:** Firewall Zone-based, IPsec/SSL VPN, NIDS Suricata/Snort, Nhận diện và ngăn chặn các kỹ thuật tấn công phổ biến.
7. [07_Monitoring_And_Logging.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/07_Monitoring_And_Logging.md)
   - **TRỌNG TÂM ĐỀ TÀI - Hệ thống Giám sát & Quản lý Nhật ký An toàn Tập trung:** Wazuh SIEM Architecture, Syslog RFC 5424, Wazuh Ruleset, Kịch bản diễn tập an ninh và Tích hợp Telegram Alerting.

---

> [!TIP]
> Mỗi chuyên đề đều cung cấp đầy đủ: **Cơ sở lý thuyết học thuật** ➔ **Sơ đồ chu trình xử lý** ➔ **Mẫu cấu hình dòng lệnh chuẩn thực tế (Cisco IOS, Linux Rsyslog, Wazuh Rules)** ➔ **Bộ câu hỏi ôn tập & phản biện bảo vệ đồ án (Viva Q&A)**.
