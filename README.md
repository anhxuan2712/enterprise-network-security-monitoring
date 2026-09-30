# ĐỒ ÁN: PHÂN TÍCH VÀ THIẾT KẾ HỆ THỐNG MẠNG AN TOÀN CHO DOANH NGHIỆP CÔNG NGHỆ
*(Enterprise Network Security Architecture, Zone-based Segmentation, Scaled-Down Lab Simulation & Centralized Wazuh SIEM Monitoring)*

> **Học phần:** Phân tích và Thiết kế An toàn Mạng Máy tính  
> **Chủ đề lựa chọn:** Mô hình Doanh nghiệp Công nghệ & Dịch vụ Số (Tech Enterprise - 2 Tòa nhà A & B, 26 Phòng ban, 144 Nhân sự, ~288 Endpoints, 30 VLANs)  
> **Người soạn thảo:** Nguyễn Phúc Vượng  
> **Người duyệt / Chủ trì đề tài:** Nguyễn Anh Xuân  
> **Kiến trúc cốt lõi:** Phân vùng an ninh Zone-based Segmentation (Zero Trust Principles), Mô hình 2 lớp Collapsed-Core, Trục cáp quang Single-Mode OS2 10Gbps qua 9 Tủ trung chuyển quang IDF & Trung tâm MDF A-202, Mô hình Thực nghiệm Lab thu nhỏ (Core L3 Switch, 6 VLAN đại diện, Wazuh SIEM All-in-One, Rsyslog Server, 4 Extended ACLs cốt lõi, Port Security, Đồng bộ NTP tập trung, Telegram Alerting & SOC 24/7).  
> **Tài liệu thiết kế mặt bằng chi tiết:** [ban thiet ke doanh nghiep/index-v2.html](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/ban%20thiet%20ke%20doanh%20nghiep/index-v2.html) *(Phiên bản 3.1)*

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
   - [3.1 Kiến trúc mạng 2 lớp Collapsed-Core & Trục trung chuyển quang OS2](#31-kiến-trúc-mạng-2-lớp-collapsed-core--trục-trung-chuyển-quang-os2)
   - [3.2 Phân vùng an ninh (Security Zones) & Ma trận kiểm soát truy cập (Zone Matrix)](#32-phân-vùng-an-ninh-security-zones--ma-trận-kiểm-soát-truy-cập-zone-matrix)
   - [3.3 Sơ đồ kiến trúc logic & Luồng dữ liệu giám sát SIEM / SOC](#33-sơ-đồ-kiến-trúc-logic--luồng-dữ-liệu-giám-sát-siem--soc)
4. [CHƯƠNG 4. TRIỂN KHAI MÔ HÌNH THỰC NGHIỆM LAB & KIỂM THỬ AN NINH](#chương-4-triển-khai-mô-hình-thực-nghiệm-lab--kiểm-thử-an-ninh)
   - [4.1 Quy hoạch Địa chỉ IP & Phân chia 30 VLAN toàn hệ thống (IP & VLAN Plan)](#41-quy-hoạch-địa-chỉ-ip--phân-chia-30-vlan-toàn-hệ-thống-ip--vlan-plan)
   - [4.2 Mô hình Demo Lab Thu Nhỏ & Yêu cầu Tài nguyên Máy ảo](#42-mô-hình-demo-lab-thu-nhỏ--yêu-cầu-tài-nguyên-máy-ảo)
   - [4.3 Cấu hình Dịch vụ Mạng Cốt lõi trên Lab (VLAN, SVI, DHCP, NTP)](#43-cấu-hình-dịch-vụ-mạng-cốt-lõi-trên-lab-vlan-svi-dhcp-ntp)
   - [4.4 Thiết kế An toàn Access & Bộ 4 Extended ACLs Cốt Lõi](#44-thiết-kế-an-toàn-access--bộ-4-extended-acls-cốt-lõi)
   - [4.5 Thiết kế Hệ thống Giám sát Wazuh SIEM, Thu thập Log & Cảnh báo Telegram](#45-thiết-kế-hệ-thống-giám-sát-wazuh-siem-thu-thập-log--cảnh-báo-telegram)
   - [4.6 Kịch bản Kiểm thử & Nghiệm thu Thực nghiệm Lab (Test Cases)](#46-kịch-bản-kiểm-thử--nghiệm-thu-thực-nghiệm-lab-test-cases)
5. [CHƯƠNG 5. TỔ CHỨC DỰ ÁN, PHÂN CÔNG NHIỆM VỤ & MA TRẬN RACI](#chương-5-tổ-chức-dự-án-phân-công-nhiệm-vụ--ma-trận-raci)
6. [CHƯƠNG 6. DANH MỤC SẢN PHẨM BÀN GIAO & TÀI LIỆU CHUYÊN ĐỀ](#chương-6-danh-mục-sản-phẩm-bàn-giao--tài-liệu-chuyên-đề)
7. [PHỤ LỤC A. MỞ RỘNG CHO MÔI TRƯỜNG DOANH NGHIỆP THỰC TẾ](#phụ-lục-a-mở-rộng-cho-môi-trường-doanh-nghiệp-thực-tế)
   - [A.1 Danh mục Thiết bị Phần cứng & Ngân sách Nguồn cấp PoE (PoE Budget)](#a1-danh-mục-thiết-bị-phần-cứng--ngân-sách-nguồn-cấp-poe-poe-budget)
   - [A.2 Quy hoạch Tủ mạng IDF/MDF & Tính toán Cổng quang SFP+ trên Core Switch](#a2-quy-hoạch-tủ-mạng-idfmdf--tính-toán-cổng-quang-sfp-trên-core-switch)
   - [A.3 Phương án Tính Sẵn Sàng Cao (HA) & Dự Phòng Thực Tế](#a3-phương-án-tính-sẵn-sàng-cao-ha--dự-phòng-thực-tế)
   - [A.4 Mạng Transit Core–Firewall Thực Tế (OSPF Area 0 MD5)](#a4-mạng-transit-corefirewall-thực-tế-ospf-area-0-md5)

---

## CHƯƠNG 1. GIỚI THIỆU DOANH NGHIỆP & PHÂN TÍCH NHU CẦU

### 1.1 Giới thiệu quy mô & Mô hình hoạt động
* **Tên doanh nghiệp:** Công ty Cổ phần Công nghệ & Dịch vụ Số TechCorp (Doanh nghiệp công nghệ mẫu).
* **Lĩnh vực hoạt động:** Phát triển phần mềm, gia công ứng dụng di động, giải pháp đám mây, trí tuệ nhân tạo (AI/ML) và phân tích dữ liệu lớn.
* **Cơ sở hạ tầng vật lý:** Khuôn viên gồm **2 Tòa nhà cao tầng** kết nối trục cáp quang nội bộ:
  - **Tòa A (Trung tâm Nghiên cứu & Vận hành Kỹ thuật - 5 Tầng):** 91 nhân sự làm việc, 15 phòng ban chức năng, bao gồm Trung tâm Dữ liệu MDF (A-202) và Trung tâm NOC/SOC (A-404).
  - **Tòa B (Trung tâm Kinh doanh & Dịch vụ Hỗ trợ - 4 Tầng):** 53 nhân sự nghiệp vụ, 11 phòng ban chức năng.
  - **Tổng nhân sự:** **144 nhân sự chính thức** (+ 2 quản trị viên hệ thống tại MDF A-202 = 146 người).
* **Quy mô Endpoints & Hạ tầng thiết kế:**
  - **144 Máy trạm PC / Workstation** (Dell OptiPlex 7000 / Precision).
  - **25 Switch truy cập phòng** (Cisco Catalyst C9200L và C9200CX).
  - **25 Bộ phát sóng Wi-Fi 6** (UniFi U6-Pro) cấp nguồn PoE+.
  - **25 Điện thoại IP Phone** (Cisco 7821 VoIP) hỗ trợ Voice VLAN & QoS.
  - **25 Camera IP an ninh Dome** (Hikvision DS-2CD2143G2-I PoE) cô lập VLAN Giám sát.
  - **25 Hệ thống kiểm soát ra vào cửa** (ZKTeco InBio + Khóa từ + Thẻ Mifare DESFire EV3).
  - **20 Máy in mạng** (HP LaserJet Enterprise M507dn) xác thực in bảo mật.
  - **9 Tủ trung chuyển quang IDF theo tầng** + **1 Trung tâm Dữ liệu MDF (A-202)** kết nối cáp quang Single-Mode OS2 10Gbps SFP+.

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
* **Dịch vụ CNTT nòng cốt:** Cụm Server Farm tập trung tại MDF A-202 gồm Active Directory/DNS/DHCP/NTP Server, Web Portal & Git CI/CD, Database Server (PostgreSQL), File Storage NAS/SAN, ERP & CRM nội bộ, Hệ thống Tổng đài VoIP IP-PBX, Wazuh SIEM Server All-in-One & Rsyslog Server.
* **Mục tiêu an toàn thông tin cốt lõi (CIA Triad):**
  - **Confidentiality (Tính bí mật):** Phân vùng mạng chi tiết theo 6 Security Zones (`Guest`, `User`, `Restricted`, `Executive`, `Infra/Mgmt`, `Voice`); bảo vệ mã nguồn phần mềm, CSDL khách hàng và dữ liệu tài chính kế toán theo nguyên tắc đặc quyền tối thiểu (*Least Privilege*).
  - **Integrity (Tính toàn vẹn):** Đảm bảo tính toàn vẹn của mã nguồn, file cấu hình hệ thống máy chủ và nhật ký an ninh không bị chỉnh sửa trái phép (giám sát tự động qua *Wazuh FIM - File Integrity Monitoring*).
  - **Availability (Tính sẵn sàng):** Đảm bảo mạng hoạt động liên tục với định tuyến Inter-VLAN tối ưu, trục cáp quang OS2 liên tầng, nguồn lưu điện UPS và hệ thống giám sát cảnh báo sớm sự cố qua Telegram Bot.

---

## CHƯƠNG 2. KHẢO SÁT HIỆN TRẠNG & ĐÁNH GIÁ RỦI RO HỆ THỐNG

### 2.1 Hiện trạng hạ tầng mạng truyền thống
* **Hạ tầng mạng phẳng (Flat Network):** Mạng nội bộ sử dụng dải IP phẳng chung (`192.168.1.0/24`), không chia VLAN theo phòng ban, thiếu kiểm soát phân quyền.
* **Hệ thống định tuyến & tường lửa yếu kém:** Chỉ có 1 Router Gateway nhà mạng cơ bản, thiếu kiểm soát phân vùng, không có bộ lọc ACLs kiểm soát truy cập giữa các khối nghiệp vụ và khối máy chủ.
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
| **RSK-01** | **Xâm nhập & Đánh cắp CSDL Tối Mật:** Kẻ tấn công từ mạng User/Guest truy cập trái phép cổng CSDL (PostgreSQL 5432) | `Data_Zone` (VLAN 32 / VLAN 10) | 3 *(Có thể)* | 5 *(Nghiêm trọng)* | **15 - High / Critical** | Cô lập VLAN 32 & VLAN 10 bằng Extended ACLs; chỉ cho phép Backend Server kết nối cổng DB 5432; kích hoạt giám sát FIM và audit log CSDL trên Wazuh. |
| **RSK-02** | **Tấn công Dò quét Mật khẩu (Brute-force SSH/RDP):** Quét dò mật khẩu máy chủ quản trị / DB Server | `Server Farm` (VLAN 10) | 5 *(Gần như chắc chắn)* | 4 *(Cao)* | **20 - Critical** | Khóa SSH chỉ cho phép Key-based Auth từ VLAN 44 (NOC/SOC) và VLAN 43 (DevOps); cấu hình Wazuh cảnh báo tức thời khi sai pass liên tiếp. |
| **RSK-03** | **Lây lan Mã độc từ Khách & Wi-Fi Vãng lai:** Thiết bị cá nhân mang mã độc kết nối vào mạng sảnh, phòng họp | `Guest_Zone` (VLAN 100, 110, 111, 143) | 4 *(Rất có thể)* | 4 *(Cao)* | **16 - Critical** | Cô lập hoàn toàn các VLAN Guest (Client Isolation + chỉ ra Internet qua NAT, cấm định tuyến sang các dải IP nội bộ). |
| **RSK-04** | **Tấn công Từ chối Dịch vụ (SYN Flood / DoS):** Gửi lượng lớn gói tin TCP SYN làm tê liệt Web Server nội bộ hoặc cổng WAN | `App / Server Farm` (VLAN 10) | 4 *(Rất có thể)* | 4 *(Cao)* | **16 - Critical** | Giới hạn kết nối và cảnh báo qua Telegram Bot khi có lưu lượng bất thường. |
| **RSK-05** | **Cắm thiết bị lạ vào cổng mạng Văn phòng:** Kẻ xấu cắm laptop cá nhân/thiết bị lạ vào Switch sảnh/phòng ban | `User_Zone` / `Infra` | 3 *(Có thể)* | 3 *(Vừa)* | **9 - Medium** | Bật **Port Security** (tối đa 2 MAC/cổng PC, vi phạm `restrict`/`shutdown`), bật **DHCP Snooping** bảo vệ cổng cấp phát IP. |
| **RSK-06** | **Quét thăm dò cổng dịch vụ (Port Scanning):** Dò quét phát hiện các cổng mở, lỗ hổng dịch vụ Web/App | `Toàn mạng` | 5 *(Gần như chắc chắn)* | 2 *(Thấp)* | **10 - High** | Thiết lập Custom Wazuh Rule (ID 100100) phát hiện quét cổng từ log ACL Deny; phân quyền ACL chặn quét giữa các VLAN. |
| **RSK-07** | **Mất mát & Thiếu nhất quán Nhật ký An toàn:** Sự cố xảy ra nhưng log nằm rải rác, sai lệch thời gian không điều tra được | `Toàn hệ thống` | 4 *(Rất có thể)* | 4 *(Cao)* | **16 - Critical** | Triển khai **Wazuh SIEM + Rsyslog tập trung**, bắt buộc đồng bộ thời gian toàn bộ thiết bị qua **máy chủ NTP nội bộ (192.168.10.10)**. |

---

### 2.3 Yêu cầu chuyển đổi sang hệ thống bảo mật thế hệ mới
1. **Phân vùng an ninh 6 Zones độc lập:** Chia tách toàn bộ hệ thống thành **30 VLANs** độc lập (gồm 25 VLAN phòng ban chức năng + 5 VLAN hạ tầng dùng chung; phòng Máy chủ Trung tâm A-202 sử dụng đồng thời VLAN 2 Management và VLAN 10 Server Farm).
2. **Kiểm soát truy cập phân vùng bằng Extended ACLs nghiêm ngặt:** Chặn tuyệt đối luồng dữ liệu từ mạng Guest và User thông thường truy cập trực tiếp vào vùng Database (VLAN 32), Quản trị (VLAN 2) và Server Farm (VLAN 10).
3. **Bảo vệ Lớp truy cập (Access Layer Security):** Kích hoạt Port Security, DHCP Snooping trên các Switch phòng; phân biệt rõ cổng Access máy trạm và cổng AP Trunk.
4. **Xây dựng Trung tâm Giám sát SIEM tập trung:** Triển khai **Wazuh All-in-One** kết hợp **Rsyslog**, phân tích log thời gian thực và cảnh báo tức thời qua **Telegram Bot**.

---

## CHƯƠNG 3. THIẾT KẾ TỔNG THỂ HỆ THỐNG MẠNG & BẢO MẬT

### 3.1 Kiến trúc mạng 2 lớp Collapsed-Core & Trục trung chuyển quang OS2
* **Mô hình Kiến trúc Collapsed-Core (Core/Distribution gộp):**
  - Hệ thống áp dụng mô hình 2 lớp Collapsed-Core (Core/Distribution gộp & Access) nhằm tối ưu hóa chi phí đầu tư và giảm độ trễ chuyển mạch (Latency).
  - Thiết bị Core Switch L3 đặt tại Trung tâm Dữ liệu MDF A-202, đóng vai trò là Lõi chuyển mạch trung tâm kiêm Định tuyến phân phối (Inter-VLAN Routing) cho toàn bộ 30 VLANs.
* **Lớp Trung chuyển Quang IDF (9 Tủ IDF theo tầng):**
  - Gồm 9 Tủ mạng phân phối tầng (5 IDF Tòa A: `IDF-A-T1` đến `IDF-A-T5`; 4 IDF Tòa B: `IDF-B-T1` đến `IDF-B-T4`).
  - Các tủ IDF đóng vai trò là điểm đấu nối trung chuyển vật lý (chứa khay phối quang ODF, Patch Panel Cat6A, PDU đo dòng và UPS), **không lắp đặt Switch phân phối L3 chủ động**.
  - Mỗi switch phòng sử dụng **1 đường cáp quang Single-Mode OS2 10Gbps SFP+** dẫn qua ODF tại IDF và kéo thẳng về Core Switch tại MDF A-202 (Mô hình kết nối điểm-điểm đơn giản hóa).
* **Lớp Truy cập (Access Layer - 25 Switch Phòng):**
  - Đặt tại từng phòng ban: Switch phòng cung cấp kết nối mạng có dây Gigabit cho máy trạm, máy in và cấp nguồn PoE+ (IEEE 802.3at) cho Wi-Fi 6 AP, IP Phone, Camera IP và Bộ kiểm soát cửa.

---

### 3.2 Phân vùng an ninh (Security Zones) & Ma trận kiểm soát truy cập (Zone Matrix)

Hệ thống được chia thành **6 Security Zones** tiêu chuẩn:
1. 🔴 **Restricted Zone (Risk: Critical / High):** Phòng CSDL A-302 (VLAN 32), DevOps A-403 (VLAN 43), NOC/SOC A-404 (VLAN 44), Nhân sự B-301 (VLAN 131), Kế toán B-302 (VLAN 132).
2. 🟠 **Infra / Management Zone (Risk: Critical / High):** Quản trị Switch/Router/FW (VLAN 2), Cụm Server Farm MDF A-202 (VLAN 10), Bảo vệ A-102 (VLAN 11), Helpdesk A-203 (VLAN 23), Camera & Khóa cửa IoT (VLAN 12).
3. 🟣 **Executive Zone (Risk: High / Medium):** Phòng CEO/CTO A-502 (VLAN 52), Phòng họp Ban Giám đốc A-503 (VLAN 53).
4. 🟢 **User Zone (Risk: Medium / Low):** Các phòng kỹ thuật R&D (A-201, A-301, A-303, A-401, A-402, A-501) và khối kinh doanh/hỗ trợ (B-201, B-202, B-303, B-304, B-401, B-402).
5. 🔵 **Voice Zone (Risk: Medium):** Phân vùng mạng thoại riêng biệt Tòa A (VLAN 200) và Tòa B (VLAN 210) ưu tiên chất lượng thoại VoIP.
6. ⚪ **Guest Zone (Risk: Untrusted):** Sảnh lễ tân (VLAN 100, 110), Phòng họp đối tác (VLAN 111), Căng tin (VLAN 143).

#### 🔐 BẢNG MA TRẬN TRUY CẬP GIỮA CÁC ZONE (ZONE ACCESS MATRIX)

| Zone Nguồn (Source) | Được Phép Truy Cập Đến (Destination) | Giới Hạn & Ràng Buộc Bảo Mật |
| :--- | :--- | :--- |
| **Guest Zone** | Internet Only (NAT Gateway) | Chặn tuyệt đối mọi VLAN nội bộ (ACL Deny `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`). Bật Client Isolation trên AP. |
| **User Zone** | Internet, Server Farm (VLAN 10 theo port dịch vụ Web/App), Voice VLAN | Chặn truy cập trực tiếp vào VLAN Restricted (32, 43, 44, 131, 132) và Management (VLAN 2). |
| **Restricted Zone** | Server Farm (VLAN 10 cổng chuyên biệt), Internet qua Secure Proxy | Tiếp nhận kết nối quản trị từ NOC/SOC (A-404) và DevOps (A-403). Cô lập lẫn nhau giữa các phòng. |
| **Executive Zone** | Internet, Server Farm (VLAN 10 theo dịch vụ), Mail/ERP | Được bảo vệ nghiêm ngặt, chặn truy cập trái phép từ User và Guest Zone. |
| **Infra / Mgmt Zone** | Toàn bộ thiết bị mạng & máy chủ | Cho phép truy cập từ Trạm NOC/SOC (A-404), DevOps (A-403) & Helpdesk (A-203), bắt buộc xác thực MFA & SSHv2/HTTPS. |
| **Voice Zone** | Máy chủ IP-PBX (VLAN 10) | Tự động nhận Voice VLAN qua LLDP-MED / CDP; cô lập hoàn toàn với dữ liệu máy trạm Data VLAN. |

---

### 3.3 Sơ đồ kiến trúc logic & Luồng dữ liệu giám sát SIEM / SOC

```mermaid
flowchart TB
    subgraph WAN_EDGE["🌐 LỚP BIÊN MẠNG & INTERNET GATEWAY"]
        ISP1["ISP 1: Viettel / FTTH 1Gbps"]
        ISP2["ISP 2: VNPT / Backup"]
        EDGE_ROUTER["Router / Firewall Biên (NAT / Gateway Ra Internet)"]
        ISP1 === EDGE_ROUTER
        ISP2 === EDGE_ROUTER
    end

    subgraph MDF_CENTER["🏢 TRUNG TÂM DỮ LIỆU MDF (A-202 - TÒA A TẦNG 2)"]
        CORE_SW["Core Switch Layer 3 (Định Tuyến Inter-VLAN / SVI Gateway .1)"]
        subgraph SERVER_FARM["Cụm Server Farm (VLAN 10: 192.168.10.0/24)"]
            SRV_AD["192.168.10.10: AD / DNS / DHCP / NTP"]
            SRV_APP["192.168.10.20: Web Portal (80/443)"]
            SRV_DB["192.168.10.30: Database PostgreSQL (5432)"]
            SRV_SIEM["192.168.10.40: 🛡️ Wazuh SIEM Server"]
            SRV_RSYS["192.168.10.50: Rsyslog Collector (UDP 514)"]
        end
        MGMT_VLAN["VLAN 2: Management (192.168.2.0/24)"]
        EDGE_ROUTER === CORE_SW
        CORE_SW --- SERVER_FARM
        CORE_SW --- MGMT_VLAN
    end

    subgraph TOA_A_IDFS["🏢 TRỤC TRUNG CHUYỂN QUANG TÒA A (5 TẦNG)"]
        IDF_A1["IDF-A-T1: A-101 (Guest 100), A-102 (Infra 11)"]
        IDF_A2["IDF-A-T2: A-201 (User 21), A-203 (Infra 23)"]
        IDF_A3["IDF-A-T3: A-301 (User 31), A-302 (Restricted 32), A-303 (User 33)"]
        IDF_A4["IDF-A-T4: A-401 (User 41), A-402 (User 42), A-403 (DevOps 43), A-404 (SOC 44)"]
        IDF_A5["IDF-A-T5: A-501 (User 51), A-502 (Exec 52), A-503 (Exec 53)"]
    end

    subgraph TOA_B_IDFS["🏢 TRỤC TRUNG CHUYỂN QUANG TÒA B (4 TẦNG)"]
        IDF_B1["IDF-B-T1: B-101 (Guest 110), B-102 (Guest 111)"]
        IDF_B2["IDF-B-T2: B-201 (User 121), B-202 (User 122)"]
        IDF_B3["IDF-B-T3: B-301 (HR 131), B-302 (Kế toán 132), B-303 (Pháp chế 133), B-304 (CSKH 134)"]
        IDF_B4["IDF-B-T4: B-401 (PM/BA 141), B-402 (Đào tạo 142), B-403 (Canteen 143)"]
    end

    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_A1
    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_A2
    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_A3
    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_A4
    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_A5

    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_B1
    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_B2
    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_B3
    CORE_SW ==="Quang OS2 10G SFP+"=== IDF_B4

    subgraph SOC_MONITORING["🛡️ LUỒNG GIÁM SÁT AN NINH SIEM / SOC (WAZUH)"]
        TEL_BOT["Telegram Alert Bot (Rule Level ≥ 8)"]
        SOC_WS["Trạm Giám sát SOC (A-404) Video Wall / Dashboard"]
        EDGE_ROUTER -.->|"1. Syslog Traffic / NAT"| SRV_RSYS
        CORE_SW -.->|"1. Syslog Port Sec / ACL Drop"| SRV_RSYS
        SRV_APP & SRV_DB -.->|"2. Wazuh Agent (TCP 1514/1515)"| SRV_SIEM
        SRV_RSYS -->|"Forward Syslog"| SRV_SIEM
        SRV_SIEM -->|"3. Gửi Cảnh Báo Khẩn (HTTPS 443)"| TEL_BOT
        SRV_SIEM -->|"4. Trực Quan Hóa Dashboard"| SOC_WS
    end
```

---

## CHƯƠNG 4. TRIỂN KHAI MÔ HÌNH THỰC NGHIỆM LAB & KIỂM THỬ AN NINH

### 4.1 Quy hoạch Địa chỉ IP & Phân chia 30 VLAN toàn hệ thống (IP & VLAN Plan)

Hệ thống tài liệu thiết kế quy hoạch đầy đủ **30 VLANs** độc lập (5 VLAN hạ tầng dùng chung + 25 VLAN phòng ban chức năng):

| STT | VLAN ID | Tên VLAN | Phân Vùng Zone | Phạm Vi Áp Dụng / Phòng Ban | Dải Mạng IP / Subnet | Default Gateway | Dải Cấp Phát DHCP | Mức Rủi Ro |
| :---: | :---: | :--- | :---: | :--- | :---: | :---: | :---: | :---: |
| **A** | **HẠ TẦNG DÙNG CHUNG (SHARED INFRASTRUCTURE - 5 VLANS)** | | | | | | |
| 1 | **VLAN 2** | `Management` | **Infra/Mgmt** | Quản trị Switch phòng, Core Switch, Router, PDU, UPS | `192.168.2.0/24` | `192.168.2.1` | IP Tĩnh (.101 - .126) | `Critical` |
| 2 | **VLAN 10** | `Server_Farm` | **Infra** | Cụm máy chủ MDF A-202 (AD .10, Web .20, DB .30, Wazuh .40, Syslog .50) | `192.168.10.0/24` | `192.168.10.1` | IP Tĩnh (.10 - .50) | `Critical` |
| 3 | **VLAN 12** | `Security_IoT` | **Infra** | 25 Camera IP & 25 Bộ kiểm soát cửa toàn tòa nhà | `192.168.12.0/24` | `192.168.12.1` | `192.168.12.10 - .200` | `High` |
| 4 | **VLAN 200** | `Voice_Toa_A` | **Voice** | Điện thoại IP Cisco 7821 các phòng Tòa A | `192.168.200.0/24` | `192.168.200.1` | `192.168.200.10 - .200` | `Medium` |
| 5 | **VLAN 210** | `Voice_Toa_B` | **Voice** | Điện thoại IP Cisco 7821 các phòng Tòa B | `192.168.210.0/24` | `192.168.210.1` | `192.168.210.10 - .200` | `Medium` |
| **B** | **TÒA A - VẬN HÀNH KỸ THUẬT (14 PHÒNG BAN CHỨC NĂNG)** | | | | | | |
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
| **C** | **TÒA B - KINH DOANH & HỖ TRỢ (11 PHÒNG BAN CHỨC NĂNG)** | | | | | | |
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

### 4.2 Mô hình Thực Nghiệm Lab Thu Nhỏ, Quy Hoạch IP Máy Ảo & Môi Trường Mô Phỏng

Để đồ án có tính khả thi cao và vừa sức thực hiện cho nhóm 5 sinh viên trên máy tính cá nhân (16GB RAM), nhóm triển khai một **Mô hình Thực nghiệm Thu nhỏ (Scaled-down Lab Topology)**. Mô hình này đại diện đầy đủ cho 6 phân vùng an ninh (Security Zones) của thiết kế 30 VLAN toàn doanh nghiệp:

```
                  [ Internet / NAT Gateway ]
                              │
                    [ Edge Router / FW ]
                              │
                    [ Core Switch L3 ]
          (Inter-VLAN Routing / SVI .1 / 4 ACLs Cốt Lõi)
         ┌────────────┬────────────┬────────────┬────────────┐
         │            │            │            │            │
    [ VLAN 10 ]  [ VLAN 21 ]  [ VLAN 32 ]  [ VLAN 44 ]  [ VLAN 100 ]
    Server Farm    User Lab    Data / DB     NOC/SOC      Guest Sảnh
    (Wazuh/Web)   (Win/Kali)  (PostgreSQL)  (Kali/SOC)     (Khách)
```

#### 📊 1. 6 VLAN đại diện được kích hoạt trong Lab:
1. **`VLAN 10` (Server Farm - Infra Zone):** Subnet `192.168.10.0/24` - SVI `192.168.10.1`. Chứa Cụm máy chủ dịch vụ cốt lõi: DHCP/DNS/NTP (`.10`), Web Portal (`.20`), Database (`.30`), Wazuh SIEM (`.40`), Rsyslog Collector (`.50`).
2. **`VLAN 21` (User Lab - User Zone):** Subnet `192.168.21.0/24` - SVI `192.168.21.1`. Đại diện cho các phòng làm việc nhân sự R&D/Kinh doanh.
3. **`VLAN 32` (Data Restricted - Restricted Zone):** Subnet `192.168.32.0/24` - SVI `192.168.32.1`. Đại diện phòng Cơ sở dữ liệu và dữ liệu tối mật.
4. **`VLAN 44` (NOC/SOC - Restricted Zone):** Subnet `192.168.44.0/24` - SVI `192.168.44.1`. Đại diện trạm quản trị và giám sát an ninh mạng (được phép SSH/RDP quản trị máy chủ).
5. **`VLAN 100` (Guest Sảnh - Guest Zone):** Subnet `172.16.100.0/24` - SVI `172.16.100.1`. Đại diện mạng khách vãng lai chỉ ra Internet.
6. **`VLAN 2` (Management - Infra/Mgmt Zone):** Subnet `192.168.2.0/24` - SVI `192.168.2.1`. Quản trị thiết bị mạng.

---

#### 💻 2. Bảng phân bổ máy ảo (VM) tối ưu tài nguyên & Kỹ thuật Gán IP phụ (IP Aliasing):

Để tiết kiệm RAM và CPU mà vẫn đảm bảo đầy đủ địa chỉ IP cho mọi dịch vụ trong kịch bản (DHCP, NTP, Web, DB, Wazuh, Syslog), nhóm áp dụng giải pháp **Gán IP Phụ (IP Aliasing / Virtual IP)** trên Linux Server:

| Tên Máy Ảo / Node | Hệ Điều Hành | Vai Trò & Dịch Vụ Cài Đặt | Địa Chỉ IP Gán (Primary & Alias)* | Cấu Hình Tối Thiểu (vCPU / RAM / Disk) |
| :--- | :--- | :--- | :--- | :---: |
| **VM 1: Wazuh SIEM All-in-One** | Ubuntu Server 22.04 LTS | • Wazuh Server + Indexer + Dashboard (Port 443, 1514, 1515)<br>• Rsyslog Server tiếp nhận Syslog mạng (UDP 514)<br>• Script Python đẩy cảnh báo Telegram Bot | • **IP Chính:** `192.168.10.40` (Wazuh SIEM)<br>• **IP Phụ (`eth0:1`):** `192.168.10.50` (Rsyslog Receiver) | 2 vCPU, **4GB RAM**, 30GB Disk |
| **VM 2: Linux Multi-Service Server** | Ubuntu Server 22.04 LTS | • ISC-DHCP-Server / Dnsmasq (cấp IP cho các VLAN qua Relay)<br>• Chrony NTP Server (đồng bộ thời gian cho Switch & Clients)<br>• Nginx Web Portal (HTTP 80, HTTPS 443)<br>• Database PostgreSQL (TCP 5432)<br>• Wazuh Agent | • **IP Chính:** `192.168.10.10` (DHCP / DNS / NTP Server)<br>• **IP Phụ 1 (`eth0:1`):** `192.168.10.20` (Web Portal / SSH)<br>• **IP Phụ 2 (`eth0:2`):** `192.168.10.30` (PostgreSQL DB) | 1-2 vCPU, **2GB RAM**, 20GB Disk |
| **VM 3: Windows Client** | Windows 10/11 LTSC | • Máy trạm người dùng văn phòng / Kế toán<br>• Cài đặt Wazuh Agent giám sát Endpoint<br>• Kiểm thử hành vi đăng nhập sai (Event ID 4625) | • **IP:** `192.168.21.50` (VLAN 21)<br>*(Có thể gán sang VLAN 32 `192.168.32.50` khi test truy cập CSDL)* | 1-2 vCPU, **2.5GB RAM**, 30GB Disk |
| **VM 4: Kali Linux (Multi-Role)** | Kali Linux 2023/2024 | • Máy diễn tập tấn công an ninh mạng (Pentest Node):<br>  - Test Nmap Port Scan (TC-02)<br>  - Test Hydra Brute-force SSH (TC-03)<br>  - Test Cô lập mạng Guest (TC-01) | • **VLAN 44:** `192.168.44.100` *(Dùng khi test Hydra SSH tới .20)*<br>• **VLAN 21:** `192.168.21.100` *(Dùng khi test Nmap Scan tới .30)*<br>• **VLAN 100:** `172.16.100.50` *(Dùng khi test chặn mạng Khách)* | 1-2 vCPU, **1.5 - 2GB RAM**, 20GB Disk |
| **TỔNG CỘNG TÀI NGUYÊN** | | **4 Máy Ảo Đồng Thời** | | **~10.5 GB RAM** *(Rất an toàn trên máy 16GB RAM)* |

> [!TIP]
> **Hướng dẫn cấu hình IP Aliasing nhanh trên VM2 (Ubuntu Server):**
> Trong file `/etc/netplan/00-installer-config.yaml`:
> ```yaml
> network:
>   version: 2
>   ethernets:
>     eth0:
>       addresses:
>         - 192.168.10.10/24  # IP Chinh (DHCP/NTP/DNS)
>         - 192.168.10.20/24  # IP Phu 1 (Web Portal/SSH)
>         - 192.168.10.30/24  # IP Phu 2 (PostgreSQL DB)
>       gateway4: 192.168.10.1
>       nameservers:
>         addresses: [8.8.8.8, 1.1.1.1]
> ```
> Chạy `sudo netplan apply` để nhận đồng thời cả 3 địa chỉ IP trên một card mạng duy nhất.

---

#### 🛠️ 3. Lựa chọn Công cụ Mô phỏng & Phương án Triển khai Thực tế:

Báo cáo xác định rõ 2 phương án công cụ mô phỏng để phù hợp với điều kiện phòng lab:

* **Phương án 1: EVE-NG hoặc GNS3 kết hợp VMware Workstation (Môi trường Khuyến nghị Tốt nhất):**
  - Sử dụng Router/Switch Cisco chạy Image Cisco IOL hoặc IOSv/IOSvL2.
  - Switch Core và Access trong EVE-NG/GNS3 được đấu nối ra các máy ảo VMware thông qua **Cloud Interface (vnet/tap bridge)**.
  - Toàn bộ gói tin mạng, phiên DHCP Relay, gói tin ACL Drop và thông điệp Syslog (`%SEC-6-IPACCESSLOGP`, `%PORT_SECURITY-2-PSECURE_VIOLATION`) được switch Cisco thật đẩy trực tiếp qua UDP 514 tới máy chủ Rsyslog/Wazuh trong thời gian thực.
* **Phương án 2: Cisco Packet Tracer kết hợp VMware Workstation (Phương án Mô phỏng Độc lập):**
  - Do Cisco Packet Tracer không hỗ trợ đấu nối card mạng trực tiếp ra máy ảo VMware bên ngoài hệ điều hành máy chủ, nhóm phân định rõ phạm vi triển khai:
    1. **Phần Hạ tầng Mạng & ACLs:** Triển khai trên Cisco Packet Tracer để kiểm tra bảng định tuyến Inter-VLAN, cấp phát DHCP qua `ip helper-address`, kiểm tra bảng chặn `show ip access-lists` và kiểm tra Port Security.
    2. **Phần Giám sát An ninh SIEM & SOC:** Triển khai trên cụm máy ảo VMware Workstation (Wazuh SIEM + Linux + Windows + Kali). Để kiểm tra luật tương quan (Rule Correlation) của các sự kiện Cisco trong Wazuh, nhóm sử dụng lệnh `logger -n 192.168.10.50 -d -u /dev/log` từ máy Linux/Kali hoặc dán trực tiếp chuỗi log Cisco vào công cụ `/var/ossec/bin/wazuh-logtest` trên Wazuh Server để chứng minh luật nổ chính xác.

---

### 4.3 Cấu hình Dịch vụ Mạng Cốt lõi trên Lab (VLAN, SVI, DHCP, NTP)

Cấu hình mẫu dòng lệnh chuẩn (thực hiện trên Core Switch L3 Cisco):

```cisco
! === 1. KHOI TAO 6 VLAN DAI DIEN TRONG LAB ===
vlan 2
 name Management
vlan 10
 name Server_Farm
vlan 21
 name User_Lab
vlan 32
 name Data_Restricted
vlan 44
 name NOC_SOC_Zone
vlan 100
 name Guest_Zone
exit

! === 2. CAU HINH DINH TUYEN INTER-VLAN (SVI GATEWAY) ===
ip routing

interface Vlan2
 description GATEWAY_MGMT
 ip address 192.168.2.1 255.255.255.0
 no shutdown

interface Vlan10
 description GATEWAY_SERVER_FARM
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface Vlan21
 description GATEWAY_USER_LAB
 ip address 192.168.21.1 255.255.255.0
 ip helper-address 192.168.10.10
 no shutdown

interface Vlan32
 description GATEWAY_DATA_RESTRICTED
 ip address 192.168.32.1 255.255.255.0
 ip helper-address 192.168.10.10
 no shutdown

interface Vlan44
 description GATEWAY_NOC_SOC
 ip address 192.168.44.1 255.255.255.0
 ip helper-address 192.168.10.10
 no shutdown

interface Vlan100
 description GATEWAY_GUEST
 ip address 172.16.100.1 255.255.255.0
 ip helper-address 192.168.10.10
 no shutdown

! === 3. XU LY DHCP RELAY TRUST OPTION 82 TREN CORE SWITCH ===
! Tranh viec Core Switch drop goi DHCP chua Option 82 do Access Switch chen vao
ip dhcp relay information trust-all

! === 4. DONG BO THOI GIAN NTP VA FORMAT LOG ===
clock timezone ICT 7 0
service timestamps log datetime msec show-timezone
ntp server 192.168.10.10
```

---

### 4.4 Thiết kế An toàn Access & Bộ 4 Extended ACLs Cốt Lõi

#### 🛡️ 1. Cấu hình Bảo vệ Cổng Access (Port Security & DHCP Snooping):
Áp dụng trên Switch Access kết nối máy trạm:
```cisco
! Bat DHCP Snooping tren cac VLAN cap phat dong
ip dhcp snooping
ip dhcp snooping vlan 21,32,44,100

! Xu ly Option 82 tren Access Switch de tranh bi Core Switch drop goi tin Relay
no ip dhcp snooping information option

! Cong Uplink noi ve Core Switch la cong Trusted
interface GigabitEthernet0/1
 description UPLINK_TO_CORE
 ip dhcp snooping trust
 switchport mode trunk

! Cong Access noi may tram User (Bat Port Security gioi han 2 MAC)
interface FastEthernet0/1
 description PC_USER_WORKSTATION
 switchport mode access
 switchport access vlan 21
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
```

---

#### 🔐 2. Bộ 4 Extended ACLs Cốt Lõi (Stateless Handling & Đúng Chiều SVI):

> [!IMPORTANT]
> **Nguyên lý Kỹ thuật Quan trọng về Stateless ACL:**  
> ACL trên Cisco Router/Switch L3 là **Stateless** (không tự động ghi nhớ phiên Stateful như Tường lửa thế hệ mới). Do đó:
> 1. Dòng `permit tcp any any established` được đặt ở đầu ACL_USER_IN, ACL_RESTRICTED_IN, ACL_SERVER_FARM_OUT để cho phép các gói tin TCP có cờ **ACK** hoặc **RST** (gói tin phản hồi của các phiên hợp lệ đã được thiết lập trước đó) đi qua trơn tru mà không bị chặn nhầm.
> 2. Mọi kết nối mới khởi tạo (gói tin có cờ **SYN**) sẽ lần lượt duyệt qua các dòng quy tắc bên dưới. Nếu không có dòng `permit` đích danh, gói tin sẽ rơi vào dòng `deny ... log`, bị loại bỏ ngay lập tức và sinh bản tin Syslog `%SEC-6-IPACCESSLOGP` gửi về Wazuh SIEM.

##### a) `ACL_GUEST_IN` (Gắn chiều `in` trên SVI `interface Vlan100` - Cách ly hoàn toàn Khách):
```cisco
ip access-list extended ACL_GUEST_IN
 remark === 1. Cho phep DHCP & DNS ===
 permit udp any eq bootpc any eq bootps
 permit udp any host 192.168.10.10 eq domain
 permit tcp any host 192.168.10.10 eq domain
 permit udp any any eq domain
 permit tcp any any eq domain
 remark === 2. Chan tuyet doi moi dai IP Private noi bo (RFC 1918) ===
 deny   ip any 10.0.0.0 0.255.255.255 log
 deny   ip any 172.16.0.0 0.15.255.255 log
 deny   ip any 192.168.0.0 0.0.255.255 log
 remark === 3. Cho phep ra Internet qua NAT Gateway ===
 permit ip any any
!
interface Vlan100
 ip access-group ACL_GUEST_IN in
```
* **Lệnh kiểm tra:** `show ip access-lists ACL_GUEST_IN`.
* **Bước thử Pass:** Từ VM Guest (`172.16.100.50`) ping ra IP Internet (ví dụ `8.8.8.8` hoặc Gateway WAN) -> Thành công.
* **Bước thử Block:** Từ VM Guest ping sang máy chủ Database `192.168.10.30` -> Bị Drop ngay lập tức, Core tăng match count dòng deny.

---

##### b) `ACL_USER_IN` (Gắn chiều `in` trên SVI `interface Vlan21` - Phân vùng Người dùng):
```cisco
ip access-list extended ACL_USER_IN
 remark === 1. Cho phep goi TCP tra loi cua cac phien da thiet lap (Established) ===
 permit tcp any any established
 remark === 2. Dich vu mang cot loi: DHCP, DNS, NTP toi 192.168.10.10 ===
 permit udp any eq bootpc any eq bootps
 permit udp any host 192.168.10.10 eq domain
 permit tcp any host 192.168.10.10 eq domain
 permit udp any host 192.168.10.10 eq 123
 remark === 3. Luong giam sat: Wazuh Agent (1514/1515) & Syslog (514) ===
 permit tcp any host 192.168.10.40 eq 1514
 permit tcp any host 192.168.10.40 eq 1515
 permit udp any host 192.168.10.50 eq 514
 remark === 4. Cho phep truy cap Web Portal noi bo (Nginx 80/443) ===
 permit tcp any host 192.168.10.20 eq 80
 permit tcp any host 192.168.10.20 eq 443
 remark === 5. Chan ket noi truc tiep toi CSDL Database (PostgreSQL 5432) ===
 deny   tcp any host 192.168.10.30 eq 5432 log
 deny   ip any host 192.168.10.30 log
 remark === 6. Chan ket noi toi Mang Quan tri (VLAN 2) va Vung CSDL (VLAN 32) ===
 deny   ip any 192.168.2.0 0.0.0.255 log
 deny   ip any 192.168.32.0 0.0.0.255 log
 remark === 7. Chan toan bo cac dai mang noi bo con lai truoc khi ra Internet ===
 deny   ip any 10.0.0.0 0.255.255.255 log
 deny   ip any 172.16.0.0 0.15.255.255 log
 deny   ip any 192.168.0.0 0.0.255.255 log
 remark === 8. Cho phep ra Internet qua NAT ===
 permit ip any any
!
interface Vlan21
 ip access-group ACL_USER_IN in
```
* **Lệnh kiểm tra:** `show ip access-lists ACL_USER_IN`.
* **Bước thử Pass:** Từ VM Windows (`192.168.21.50`) mở trình duyệt truy cập Web Portal `http://192.168.10.20` -> Mở trang thành công.
* **Bước thử Block (TC-02):** Từ Kali ở VLAN 21 (`192.168.21.100`) chạy lệnh quét cổng `nmap -sS -p 5432 192.168.10.30` -> Bị chặn ngay tại dòng deny mục 5, sinh log `%SEC-6-IPACCESSLOGP`.

---

##### c) `ACL_RESTRICTED_IN` (Gắn chiều `in` trên SVI `interface Vlan32` - Vùng CSDL):
```cisco
ip access-list extended ACL_RESTRICTED_IN
 remark === 1. Cho phep goi TCP tra loi cua cac phien da thiet lap ===
 permit tcp any any established
 remark === 2. Dich vu mang cot loi: DNS, NTP, DHCP ===
 permit udp any host 192.168.10.10 eq 53
 permit tcp any host 192.168.10.10 eq 53
 permit udp any host 192.168.10.10 eq 123
 permit udp any eq bootpc any eq bootps
 remark === 3. Luong giam sat Wazuh Agent & Syslog ===
 permit tcp any host 192.168.10.40 eq 1514
 permit tcp any host 192.168.10.40 eq 1515
 permit udp any host 192.168.10.50 eq 514
 remark === 4. Cho phep Data Engineer ket noi Database Server (PostgreSQL 5432) ===
 permit tcp 192.168.32.0 0.0.0.255 host 192.168.10.30 eq 5432
 remark === 5. Chan truy cap sang cac dai Private khac ===
 deny   ip any 10.0.0.0 0.255.255.255 log
 deny   ip any 172.16.0.0 0.15.255.255 log
 deny   ip any 192.168.0.0 0.0.255.255 log
 permit ip any any
!
interface Vlan32
 ip access-group ACL_RESTRICTED_IN in
```
* **Lệnh kiểm tra:** `show ip access-lists ACL_RESTRICTED_IN`.
* **Bước thử Pass:** Từ máy thuộc VLAN 32 kết nối cổng 5432 của DB `192.168.10.30` -> Kết nối thành công.
* **Bước thử Block:** Từ máy thuộc VLAN 32 ping sang máy trạm VLAN 21 (`192.168.21.50`) -> Bị Drop.

---

##### d) `ACL_SERVER_FARM_OUT` (Gắn chiều `out` trên SVI `interface Vlan10` - Bảo vệ Cụm Máy Chủ):
```cisco
ip access-list extended ACL_SERVER_FARM_OUT
 remark === 1. Cho phep goi TCP tra loi cua Server ra ngoai (Established) ===
 permit tcp any any established
 remark === 2. Cho phep SOC (VLAN 44) & DevOps quan tri Server (SSH 22, RDP 3389, Web SSL 443) ===
 permit tcp 192.168.44.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 22
 permit tcp 192.168.44.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 3389
 permit tcp 192.168.44.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 443
 remark === 3. Cho phep Backend (VLAN 31) & Data Engineer (VLAN 32) truy van PostgreSQL ===
 permit tcp 192.168.31.0 0.0.0.255 host 192.168.10.30 eq 5432
 permit tcp 192.168.32.0 0.0.0.255 host 192.168.10.30 eq 5432
 remark === 4. Cho phep moi VLAN ket noi dich vu Web Portal (80/443) ===
 permit tcp any host 192.168.10.20 eq 80
 permit tcp any host 192.168.10.20 eq 443
 remark === 5. Cho phep moi VLAN hoi DNS, NTP, DHCP toi 192.168.10.10 ===
 permit udp any host 192.168.10.10 eq domain
 permit tcp any host 192.168.10.10 eq domain
 permit udp any host 192.168.10.10 eq 123
 permit udp any host 192.168.10.10 eq bootps
 remark === 6. Cho phep Wazuh Agent (1514/1515) & Syslog (514) gui log vao Server Farm ===
 permit tcp any host 192.168.10.40 eq 1514
 permit tcp any host 192.168.10.40 eq 1515
 permit udp any host 192.168.10.50 eq 514
 remark === 7. Chan toan bo luu luong trai phep con lai vao Vung Server Farm ===
 deny   ip any 192.168.10.0 0.0.0.255 log
 remark === 8. Cho phep cac luu luong khac di qua (neu co) ===
 permit ip any any
!
interface Vlan10
 ip access-group ACL_SERVER_FARM_OUT out
```
* **Lệnh kiểm tra:** `show ip access-lists ACL_SERVER_FARM_OUT`.
* **Bước thử Pass (TC-03):** Từ máy Kali đặt tại **VLAN 44 (`192.168.44.100`)** SSH vào máy chủ Linux `192.168.10.20` (`ssh admin@192.168.10.20`) -> Kết nối TCP 22 được phép đi qua, tới tầng ứng dụng SSH của Linux Server để ghi nhận log thử mật khẩu.
* **Bước thử Block:** Từ máy Kali đặt tại **VLAN 21 (`192.168.21.100`)** SSH vào `192.168.10.20` -> Bị Drop ngay tại SVI Vlan10 chiều out, ghi nhận log `%SEC-6-IPACCESSLOGP` (do VLAN 21 không có quyền SSH vào Server).

---

### 4.5 Thiết kế Hệ thống Giám sát Wazuh SIEM, Thu thập Log & Cảnh báo Telegram

#### 1. Cấu hình Chuyển tiếp Syslog Thiết bị Mạng về Rsyslog:
* Trên Switch Core Cisco (sử dụng định dạng Syslog BSD RFC 3164 mặc định được hỗ trợ trên 100% các phiên bản IOS/Packet Tracer/EVE-NG):
  ```cisco
  logging host 192.168.10.50
  logging trap informational
  logging facility local6
  ```
* Trên máy chủ Rsyslog Server (`192.168.10.50` tích hợp trên Ubuntu Wazuh Server): Mở cổng UDP 514 trong `/etc/rsyslog.conf` và chuyển tiếp log vào `/var/log/cisco.log` để Wazuh Agent/Server đọc trực tiếp.

---

#### 2. Định nghĩa 3 Rule Tùy Chỉnh Chạy Thật Trên Wazuh (`/var/ossec/etc/rules/local_rules.xml`):

```xml
<group name="cisco_ios,custom_security,">
  <!-- Rule 1: Phat hien Port Scanning dua tren log ACL Deny lap lai tu cung 1 IP nguon -->
  <rule id="100100" level="10" frequency="6" timeframe="10">
    <same_source_ip />
    <match>%SEC-6-IPACCESSLOGP</match>
    <description>Canh bao SOC: Phat hien hanh vi Do Quet Cong (Port Scan) tu IP nguon bat thuong</description>
    <mitre>
      <id>T1046</id>
    </mitre>
  </rule>

  <!-- Rule 2: Phat hien Vi pham Port Security tren Switch Access -->
  <rule id="100101" level="10">
    <match>%PORT_SECURITY-2-PSECURE_VIOLATION</match>
    <description>Canh bao An ninh Vat ly: Vi pham Port Security - Phat hien cam thiet bi mang/MAC la vao Switch</description>
    <mitre>
      <id>T1200</id>
    </mitre>
  </rule>

  <!-- Rule 3: Phat hien Dang nhap sai Windows lien tiep tren 5 lan trong 60 giay (Brute-force) -->
  <rule id="100102" level="10" frequency="5" timeframe="60">
    <if_sid>60122</if_sid>
    <same_field>win.eventdata.targetUserName</same_field>
    <description>Canh bao An ninh May tram: Phat hien chuoi dang nhap sai Windows lien tiep cung tai khoan</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>
```

> [!TIP]
> **Phương pháp kiểm tra tính chính xác của Rule qua `wazuh-logtest`:**  
> Kỹ sư SOC chạy công cụ `/var/ossec/bin/wazuh-logtest` trên Ubuntu Server và dán chuỗi log mẫu để xác minh:
> - **Kiểm tra Rule 100100 (Port Scan):** Dán liên tiếp 6 dòng log ACL:  
>   `Feb 10 10:15:30 Core-SW %SEC-6-IPACCESSLOGP: list ACL_USER_IN denied tcp 192.168.21.100(54321) -> 192.168.10.30(5432), 1 packet`  
>   -> Sau 6 lần dán trong vòng 10 giây, `wazuh-logtest` sẽ kích hoạt **Rule 100100 (Level 10)**.
> - **Kiểm tra Rule 100101 (Port Security):** Dán chuỗi log:  
>   `%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC 0050.7966.6800 on port FastEthernet0/1.`  
>   -> `wazuh-logtest` sẽ kích hoạt ngay **Rule 100101 (Level 10)**.
> - **Kiểm tra Rule 100102 (Windows Logon):** Dán log Windows Event 4625 có trường `<Data Name='TargetUserName'>ktoan01</Data>` 5 lần trong 60s -> Kích hoạt **Rule 100102**.

---

#### 3. Tự động hóa cảnh báo qua Telegram Bot:
* Trên Wazuh Server, cấu hình tích hợp trong `/var/ossec/etc/ossec.conf`:
  ```xml
  <integration>
    <name>custom-telegram</name>
    <hook_url>https://api.telegram.org/bot<TOKEN>/sendMessage</hook_url>
    <level>8</level>
    <alert_format>json</alert_format>
  </integration>
  ```
  *(Ghi chú: Thẻ `<name>` trong Wazuh được đặt là `custom-telegram` tương ứng với file thực thi `/var/ossec/integrations/custom-telegram` - không có phần mở rộng `.py` trong thẻ name)*.
* Mọi sự kiện an ninh kích hoạt Rule có **`level >= 8`** (Bao gồm Rule 100100 Port Scan, Rule 100101 Port Security, Rule 100102 Windows Logon Fail, Rule 5712 SSH Brute-force) sẽ tự động đẩy tin nhắn báo động tức thời vào nhóm Telegram của đội ngũ SOC < 2 giây.

---

### 4.6 Kịch bản Kiểm thử & Nghiệm thu Thực nghiệm Lab (Test Cases)

| Mã TC | Tên Kịch Bản Thử Nghiệm | Điều Kiện Tiên Quyết | Thao Tác Thực Hiện | Bằng Chứng Nghiệm Thu (Lệnh Show / Log / Alert) | Ảnh Chụp Báo Cáo Cần Có | Người Phụ Trách |
| :---: | :--- | :--- | :--- | :--- | :--- | :---: |
| **TC-01** | **Chặn truy cập trái phép Guest/User vào Database** | Đã gán `ACL_GUEST_IN` trên SVI 100 và `ACL_USER_IN` trên SVI 21 | Từ máy Guest (`172.16.100.50`) hoặc User ping/telnet cổng 5432 của DB `192.168.10.30` | Lệnh `show ip access-lists` hiển thị match count tăng; log console `%SEC-6-IPACCESSLOGP` | Ảnh màn hình Command Prompt báo Request Timed Out & Ảnh log ACL Drop trên Switch | Minh Trí |
| **TC-02** | **Dò quét cổng (Nmap Port Scan) & Bắn Bot Telegram** | Kali Linux đặt tại VLAN 21 (`192.168.21.100`), Core Switch đẩy Syslog về `.50` | Từ Kali chạy `nmap -sS -p 1-100 192.168.10.30` quét cổng Database | Log `%SEC-6-IPACCESSLOGP` sinh ra liên tục; `/var/ossec/logs/alerts/alerts.log` kích hoạt Rule 100100 (Level 10) | Ảnh terminal Kali chạy Nmap & Ảnh tin nhắn cảnh báo Rule 100100 trên Telegram | Thành Đạt |
| **TC-03** | **Tấn công dò quét mật khẩu SSH máy chủ Linux** | Kali Linux đặt tại **VLAN 44 (`192.168.44.100`)**, Wazuh Agent chạy trên Server Linux `192.168.10.20` | Từ Kali chạy `hydra -l root -P pass.txt ssh://192.168.10.20` | File `/var/log/auth.log` ghi nhận `Failed password`; Wazuh kích hoạt Rule ID `5712` (Level 10) | Ảnh Hydra chạy brute-force & Ảnh thông báo cảnh báo SSH Brute-force trên Telegram | Thành Đạt |
| **TC-04** | **Đăng nhập sai Windows liên tiếp (Logon Failure)** | Wazuh Agent đã chạy trên Windows Client `192.168.21.50` | Nhập sai mật khẩu Windows 5 lần liên tiếp trong 60 giây | Windows Event Viewer ghi nhận 5 Event ID 4625; Wazuh kích hoạt Custom Rule ID `100102` | Ảnh màn hình khóa Windows & Ảnh Dashboard Wazuh ghi nhận chuỗi Event ID 4625 | Tuấn Anh |
| **TC-05** | **Vi phạm An ninh cổng mạng (Port Security Violation)** | Cổng Switch Access đã bật `switchport port-security` | Rút cáp PC cắm vào máy tính lạ mang địa chỉ MAC thứ 2 | `show port-security interface Fa0/1` hiển thị Violation Count = 1; Switch sinh log `%PORT_SECURITY-2-PSECURE_VIOLATION` | Ảnh cấu hình Port Security & Ảnh log vi phạm trên console Switch gửi về Rsyslog | Minh Trí |
| **TC-06** | **Kiểm tra Đồng bộ Thời gian Mạng (NTP Offset)** | Server Linux `.10` chạy dịch vụ Chrony NTP, Switch trỏ `ntp server 192.168.10.10` | Chạy lệnh kiểm tra đồng bộ trên Core Switch và Linux Client | Lệnh `show ntp status` và `show ntp associations` trên Switch; lệnh `chronyc tracking` trên Linux | Ảnh chụp `show ntp status` hiển thị `Clock is synchronized` (Độ lệch Offset < 10ms) | Phúc Vượng |
| **TC-07** | **Nghiệm thu Luồng Giám sát Wazuh Agent & Dashboard SOC** | Wazuh Server `.40` mở cổng 1514/1515, Agent đã cài | Chạy `systemctl status wazuh-agent`, chỉnh sửa file `/etc/shadow` để test FIM | Giao diện Wazuh Dashboard hiển thị Agent `Active`; mục Integrity Monitoring bắt sự kiện sửa file | Ảnh chụp giao diện tổng quan Wazuh Security Events Dashboard và FIM Alert | Anh Xuân |




### 4.6 Kịch bản Kiểm thử & Nghiệm thu Thực nghiệm Lab (Test Cases)

| Mã TC | Tên Kịch Bản Thử Nghiệm | Điều Kiện Tiên Quyết | Thao Tác Thực Hiện | Bằng Chứng Nghiệm Thu (Lệnh Show / Log / Alert) | Ảnh Chụp Báo Cáo Cần Có | Người Phụ Trách |
| :---: | :--- | :--- | :--- | :--- | :--- | :---: |
| **TC-01** | **Chặn truy cập trái phép Guest/User vào Database** | Đã gán `ACL_GUEST_IN` trên SVI 100 và `ACL_USER_IN` trên SVI 21 | Từ máy Guest (`172.16.100.x`) hoặc User ping/telnet cổng 5432 của DB `192.168.10.30` | Lệnh `show ip access-lists` hiển thị match count tăng; log console `%SEC-6-IPACCESSLOGP` | Ảnh màn hình Command Prompt báo Request Timed Out & Ảnh log ACL Drop trên Switch | Minh Trí |
| **TC-02** | **Dò quét cổng (Nmap Port Scan) & Bắn Bot Telegram** | Kali Linux (`192.168.21.100`), Core Switch đẩy Syslog về `.50` | Từ Kali chạy `nmap -sS -p 1-100 192.168.10.30` quét cổng Database | Log `%SEC-6-IPACCESSLOGP` sinh ra liên tục; `/var/ossec/logs/alerts/alerts.log` kích hoạt Rule 100100 | Ảnh terminal Kali chạy Nmap & Ảnh tin nhắn cảnh báo Rule 100100 trên Telegram | Thành Đạt |
| **TC-03** | **Tấn công dò quét mật khẩu SSH máy chủ Linux** | Wazuh Agent đã chạy trên Server Linux `192.168.10.20` | Từ Kali chạy `hydra -l root -P pass.txt ssh://192.168.10.20` | File `/var/log/auth.log` ghi nhận `Failed password`; Wazuh kích hoạt Rule ID `5712` (Level 10) | Ảnh Hydra chạy brute-force & Ảnh thông báo cảnh báo SSH Brute-force trên Telegram | Thành Đạt |
| **TC-04** | **Đăng nhập sai Windows liên tiếp (Logon Failure)** | Wazuh Agent đã chạy trên Windows Client `192.168.21.50` | Nhập sai mật khẩu Windows 5 lần liên tiếp trong 60 giây | Windows Event Viewer ghi nhận 5 Event ID 4625; Wazuh kích hoạt Custom Rule ID `100102` | Ảnh màn hình khóa Windows & Ảnh Dashboard Wazuh ghi nhận chuỗi Event ID 4625 | Tuấn Anh |
| **TC-05** | **Vi phạm An ninh cổng mạng (Port Security Violation)** | Cổng Switch Access đã bật `switchport port-security` | Rút cáp PC cắm vào máy tính lạ mang địa chỉ MAC thứ 2 | `show port-security interface Fa0/1` hiển thị Violation Count = 1; Switch sinh log PSECUREVIOLATION | Ảnh cấu hình Port Security & Ảnh log vi phạm trên console Switch gửi về Rsyslog | Minh Trí |
| **TC-06** | **Kiểm tra Đồng bộ Thời gian Mạng (NTP Offset)** | Server Linux `.10` chạy dịch vụ NTP, Switch trỏ `ntp server` | Chạy lệnh kiểm tra đồng bộ trên Core Switch và Linux Client | Lệnh `show ntp status` và `show ntp associations` trên Switch; lệnh `chronyc tracking` trên Linux | Ảnh chụp `show ntp status` hiển thị `Clock is synchronized` (Độ lệch Offset < 10ms) | Phúc Vượng |
| **TC-07** | **Nghiệm thu Luồng Giám sát Wazuh Agent & Dashboard SOC** | Wazuh Server `.40` mở cổng 1514/1515, Agent đã cài | Chạy `systemctl status wazuh-agent`, chỉnh sửa file `/etc/shadow` để test FIM | Giao diện Wazuh Dashboard hiển thị Agent `Active`; mục Integrity Monitoring bắt sự kiện sửa file | Ảnh chụp giao diện tổng quan Wazuh Security Events Dashboard và FIM Alert | Anh Xuân |

---

## CHƯƠNG 5. TỔ CHỨC DỰ ÁN, PHÂN CÔNG NHIỆM VỤ & MA TRẬN RACI

### 👑 1. Phân công nhiệm vụ chi tiết từng thành viên:
1. **Nguyễn Anh Xuân (Trưởng nhóm - SIEM & SOC Lead / Người duyệt đề tài):**
   - Chủ trì kiến trúc an toàn hệ thống, nghiệm thu tài liệu thiết kế; triển khai Máy chủ Wazuh All-in-One (`192.168.10.40`), cấu hình tiếp nhận Syslog RFC 3164, viết tập luật tương quan XML (Ruleset) và cấu hình tích hợp Bot Telegram báo động tự động.
2. **Nguyễn Phúc Vượng (Kỹ sư Mạng 1 - Topology & Routing / Người soạn thảo):**
   - Thiết kế sơ đồ mạng 2 tòa nhà A & B trên Packet Tracer / EVE-NG theo mô hình Collapsed-Core; cấu hình Inter-VLAN Routing trên SVI, dịch vụ DHCP Relay theo 30 phân vùng VLAN và máy chủ đồng bộ thời gian NTP (`192.168.10.10`).
3. **Nguyễn Minh Trí (Kỹ sư Mạng 2 - Bảo mật Thiết bị & Syslog Forwarding):**
   - Cấu hình Port Security, DHCP Snooping trên Switch truy cập; thiết lập bộ 4 Extended ACLs cốt lõi cô lập Data/Server Farm/Guest; cấu hình chuyển tiếp Syslog thiết bị mạng về Rsyslog Server (`192.168.10.50`).
4. **Nguyễn Thành Đạt (Chuyên viên ATTT 1 - Giám sát Máy chủ & Pentest):**
   - Cài đặt Wazuh Agent trên máy chủ Linux (Web/DB); thực hiện các kịch bản diễn tập tấn công bằng Kali Linux (Nmap Port Scan, Hydra Brute-force SSH); đối soát Rule ID cảnh báo trên SIEM.
5. **Lê Tuấn Anh (Chuyên viên ATTT 2 - An ninh Máy trạm & Báo cáo):**
   - Cài đặt Wazuh Agent trên máy trạm Windows khối văn phòng; thử nghiệm các hành vi bất thường trên Endpoint (đăng nhập sai 5 lần); thiết kế Dashboard trực quan hóa SOC và hoàn thiện tài liệu Báo cáo Word & Slide bảo vệ.

---

### 📊 2. Ma trận Trách nhiệm (RACI Matrix)

| Giai đoạn & Hạng mục công việc | Nguyễn Anh Xuân (Lead) | Nguyễn Phúc Vượng (Net 1) | Nguyễn Minh Trí (Net 2) | Nguyễn Thành Đạt (Security 1) | Lê Tuấn Anh (Security 2) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| 1. Khảo sát phân vùng 26 phòng ban 2 Tòa A-B & Thiết kế Sơ đồ Collapsed-Core | C | **R / A** | C | I | I |
| 2. Cấu hình Inter-VLAN Routing, DHCP theo 30 VLAN & Đồng bộ NTP | C | **R / A** | C | I | I |
| 3. Cấu hình Bộ 4 Extended ACLs, Port Security & Đẩy Syslog từ Switch/Core | C | C | **R / A** | I | I |
| 4. Cài đặt Máy chủ Wazuh SIEM & Tích hợp Bot Cảnh báo Telegram | **R / A** | I | C | C | I |
| 5. Cài Wazuh Agent Máy chủ Linux & Diễn tập Tấn công (Nmap, Hydra) | C | I | I | **R / A** | C |
| 6. Cài Wazuh Agent Máy trạm Windows & Test hành vi nội bộ (Logon Fail) | C | I | I | **A** | **R** |
| 7. Thiết kế Dashboard SOC trực quan hóa & Phân tích Sự cố | **A** | I | I | C | **R** |
| 8. Soạn thảo Báo cáo Đồ án Tổng thể & Thiết kế Slide Bảo vệ | **A** | C | C | C | **R** |

> **Quy ước:** **R** (*Responsible*) = Người trực tiếp thực hiện; **A** (*Accountable*) = Người kiểm tra, nghiệm thu; **C** (*Consulted*) = Người hỗ trợ, cố vấn; **I** (*Informed*) = Người theo dõi tiến độ.

---

## CHƯƠNG 6. DANH MỤC SẢN PHẨM BÀN GIAO & TÀI LIỆU CHUYÊN ĐỀ

### 📦 1. Sản phẩm bàn giao nghiệm thu đồ án:
1. **Báo cáo chuyên đề hoàn chỉnh (Đóng quyển 50 ÷ 70 trang):** Soạn thảo chuẩn cấu trúc theo đề cương môn học.
2. **Slide thuyết trình bảo vệ (10 ÷ 15 slide):** Trình bày trực quan, đầy đủ luận điểm kiến trúc và kết quả thử nghiệm.
3. **Bản vẽ thiết kế mặt bằng & sơ đồ chi tiết 26 phòng ban:** File [ban thiet ke doanh nghiep/index-v2.html](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/ban%20thiet%20ke%20doanh%20nghiep/index-v2.html) *(Phiên bản 3.1)* kèm bản in PDF kỹ thuật.
4. **Bảng địa chỉ IP & VLAN Plan:** Bảng quy hoạch IP chi tiết 30 VLANs kèm dải cấp phát DHCP.
5. **Danh sách chính sách an toàn (Security Policy Catalog):** Tập hợp bộ 4 Extended ACLs, cấu hình Port Security và Rule cảnh báo Wazuh XML.
6. **File mô phỏng thực nghiệm:** File Cisco Packet Tracer / EVE-NG sẵn sàng chạy demo 7 kịch bản kiểm thử.
7. **Nhật ký làm việc & Video Demo:** Video ghi hình diễn tập tấn công và phản ứng tức thời của hệ thống SIEM & Bot Telegram.

---

### 📚 2. Tài liệu Giáo trình Kỹ thuật Chuyên sâu (7 Chuyên đề):
Toàn bộ hướng dẫn lý thuyết và cấu hình mẫu chi tiết được lưu trữ tại thư mục [Documents/](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents):
1. [01_Network_Fundamentals.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/01_Network_Fundamentals.md): Nền tảng mạng & Kiến trúc giao thức (30 VLANs, VLSM).
2. [02_Network_Infrastructure.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/02_Network_Infrastructure.md): Kiến trúc hạ tầng thiết bị mạng (Collapsed-Core, Access Security).
3. [03_Network_Segmentation.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/03_Network_Segmentation.md): Phân đoạn mạng & Ma trận kiểm soát truy cập (Extended ACLs).
4. [04_Core_Network_Services.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/04_Core_Network_Services.md): Dịch vụ mạng cốt lõi & Giám sát (DHCP Relay, NTP Synchronization).
5. [05_Routing_Protocols.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/05_Routing_Protocols.md): Giao thức định tuyến & Bảo mật mạng lõi.
6. [06_Network_Security.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/06_Network_Security.md): An toàn mạng & Phòng thủ theo chiều sâu (Port Security, DAI).
7. [07_Monitoring_And_Logging.md](file:///d:/Academy/Computer%20Network%20Safety%20Analysis%20and%20Design/Documents/07_Monitoring_And_Logging.md): Hệ thống Giám sát & Quản lý Nhật ký An toàn Tập trung (Wazuh SIEM, Rsyslog, Telegram Alerting).

---

## PHỤ LỤC A. MỞ RỘNG CHO MÔI TRƯỜNG DOANH NGHIỆP THỰC TẾ
*(Nội dung kỹ thuật nâng cao mở rộng phục vụ tham khảo thiết kế doanh nghiệp thực tế – **KHÔNG mô phỏng trong bài lab đồ án**)*

### A.1 Danh mục Thiết bị Phần cứng & Ngân sách Nguồn cấp PoE (PoE Budget)

#### 📦 1. Tổng hợp thiết bị phần cứng đề xuất cho môi trường thực tế:
1. **Cặp Core Switch Cisco Catalyst 9500:** Đề xuất model Cisco Catalyst C9500-24Y4C (24 cổng 1/10/25G SFP28 + 4 cổng 40/100G QSFP28) hoặc C9500-40X. *(Cần xác minh với datasheet Cisco)*.
2. **Switch Access Cisco Catalyst C9200L-24P-4X (24 cổng PoE+ 370W, 4x 10G SFP+):** 16 bộ (cho 16 phòng quy mô vừa và lớn).
3. **Switch Access Cisco Catalyst C9200CX-8P-2X2G (8 cổng PoE+ 125W, 2x 10G SFP+):** 9 bộ (cho 9 phòng quy mô nhỏ).
4. **Bộ phát Wi-Fi 6 UniFi U6-Pro (PoE+ 30W Class 4):** 25 bộ.
5. **Điện thoại IP Cisco 7821 VoIP (PoE 15.4W Class 3):** 25 bộ.
6. **Camera IP Hikvision DS-2CD2143G2-I (PoE 15.4W Class 3):** 25 bộ.
7. **Bộ điều khiển kiểm soát cửa ZKTeco InBio (PoE 15.4W Class 3):** 25 bộ.
8. **Máy trạm PC Dell OptiPlex 7000 / Precision:** 144 bộ.
9. **Máy in mạng HP LaserJet Enterprise M507dn:** 20 bộ.

#### ⚡ 2. Bảng tính toán tải nguồn PoE tại từng phòng ban (PoE Budget):
* **Giả định công suất tiêu thụ tối đa (Max Power Draw) của thiết bị PoE:**
  - 1x Wi-Fi 6 AP: **30.0W** (802.3at PoE+ Class 4).
  - 1x Điện thoại IP: **15.4W** (802.3af PoE Class 3).
  - 1x Camera IP: **15.4W** (802.3af PoE Class 3).
  - 1x Bộ điều khiển cửa: **15.4W** (802.3af PoE Class 3).
* **Tổng công suất tải ước tính tối đa 1 phòng (4 thiết bị PoE):**
  $$P_{\text{phòng}} = 30.0\text{W} + 15.4\text{W} + 15.4\text{W} + 15.4\text{W} = \mathbf{76.2\text{W}}$$
* **Ngưỡng an toàn kỹ thuật (80% Budget Switch):** C9200CX 125W là **100W**; C9200L 370W là **296W**.

| Nhóm Phòng Ban | Model Switch Trang Bị | PoE Budget Switch* | Tải PoE Ước Tính Tối Đa | Tỷ Lệ Sử Dụng (%) | Đánh Giá Trạng Thái |
| :--- | :--- | :---: | :---: | :---: | :---: |
| `A-101`, `A-102`, `A-502`, `A-503`, `B-101`, `B-102`, `B-303`, `B-402`, `B-403` (9 phòng nhỏ) | Cisco Catalyst C9200CX-8P-2X2G | **125W** | **76.2W** | **61.0%** | `An toàn (< 80%)` |
| `A-201`, `A-203`, `A-301..303`, `A-401..404`, `A-501`, `B-201..202`, `B-301..302`, `B-304`, `B-401` (16 phòng lớn) | Cisco Catalyst C9200L-24P-4X | **370W** | **76.2W** | **20.6%** | `Rất an toàn (< 80%)` |

> [!WARNING]
> *\* Ghi chú:* Thông số PoE Budget (125W / 370W) cần được xác minh chính xác với Datasheet Cisco theo mã nguồn phần cứng cụ thể trước khi mua sắm.

---

### A.2 Quy hoạch Tủ mạng IDF/MDF & Tính toán Cổng quang SFP+ trên Core Switch

#### 🏢 1. Quy hoạch 9 Tủ Trung Chuyển Quang IDF theo tầng:
* Mỗi tủ IDF trang bị Rack 42U, Patch Panel Cat6A, ODF khay phối quang Single-Mode OS2 và UPS Online 3000VA.
* Toàn bộ cáp quang 10G từ switch phòng được đấu nối qua ODF tại IDF và kéo thẳng về Cặp Core Switch tại Trung tâm MDF A-202.

#### 🔌 2. Bảng đếm cổng quang SFP+ (10Gbps) cần có trên Cặp Core Catalyst 9500 (Môi trường thực tế):

| STT | Mục Đích Kết Nối Vật Lý | Số Đường Link Thực Tế | Chuẩn Giao Tiếp | Ghi Chú Kỹ Thuật |
| :---: | :--- | :---: | :---: | :--- |
| 1 | **Uplink từ 25 Switch phòng (Access Switches)** | 25 | 10G SFP+ OS2 | Đấu nối qua ODF trung chuyển tại 9 IDF về Core |
| 2 | **Kết nối Cặp Next-Gen Firewall (HA Active/Standby)** | 4 | 10G SFP+ | 2 link 10G cho mỗi Firewall (LACP MEC) |
| 3 | **Kết nối Cụm Server Farm / Data Center MDF A-202** | 8 | 10G SFP+ | Dual-homed LACP MEC tới cụm máy chủ ảo hóa / vật lý |
| 4 | **Trạm Giám sát NOC/SOC (A-404) & Helpdesk (A-203)** | 2 | 10G SFP+ | Cổng trunking quản trị và SPAN Mirroring |
| 5 | **Dự phòng mở rộng (Expansion Spares)** | 5 | 10G SFP+ | Dự phòng nâng cấp thêm thiết bị |
| 6 | **StackWise Virtual Link (SVL) + DAD Link** | 2 - 4 | 40G / 100G QSFP28 *(hoặc 25G)* | Đấu chéo trực tiếp giữa Core 1 và Core 2 |
| **TỔNG** | **Tổng cổng cần thiết trên Cặp Core Switch** | **44 Cổng 10G** | **+ 2-4 Cổng 40G/100G** | *Khuyến nghị C9500-24Y4C (tổng 48x 10/25G + 8x 40/100G)* |

---

### A.3 Phương án Tính Sẵn Sàng Cao (HA) & Dự Phòng Thực Tế

* **Công nghệ Cisco StackWise Virtual (SVL):**
  - Trong môi trường thực tế, 2 switch Catalyst 9500 được ghép cặp bằng công nghệ Cisco StackWise Virtual tạo thành một thực thể chuyển mạch logic duy nhất với 1 Control Plane và 2 Data Plane Active/Active.
  - Default Gateway của các VLAN (địa chỉ `.1`) được gán trực tiếp trên SVI của switch logic.
* **Phương án HSRP / VRRP Dự phòng Thay thế:**
  - Nếu doanh nghiệp không sử dụng StackWise Virtual mà tách thành 2 Core Switch L3 độc lập, giao thức HSRP/VRRP sẽ được cấu hình với Virtual IP `.1` (Core 1 `.2` Active, Core 2 `.3` Standby).
* **Bảo vệ ARP cho Thiết bị IP Tĩnh (Static ARP Inspection):**
  - Đối với VLAN 2 (Mgmt) và VLAN 10 (Server Farm) sử dụng IP tĩnh, trên thiết bị thật cần cấu hình ARP ACL hoặc `ip source binding` tĩnh để tránh việc DAI drop nhầm gói ARP hợp lệ:
    ```cisco
    ip arp access-list STATIC_ARP_MGMT
     permit ip host 192.168.2.101 mac host 0011.2233.4401
    !
    ip arp inspection filter STATIC_ARP_MGMT vlan 2
    ```

---

### A.4 Mạng Transit Core–Firewall Thực Tế (OSPF Area 0 MD5)

* **Dải mạng Transit:** `10.255.0.0/29` (Subnet mask `255.255.255.248`):
  - `10.255.0.1`: Interface Transit trên Cặp Core Switch.
  - `10.255.0.2`: Inside Interface trên Firewall 1 (Active).
  - `10.255.0.3`: Inside Interface trên Firewall 2 (Standby).
  - `10.255.0.4`: Virtual IP (VIP) của Cụm Firewall HA.
* **Định tuyến:** Chạy OSPF Area 0 với xác thực MD5 giữa Core Switch và Firewall; Core Switch quảng bá dải 30 VLANs lên Firewall và nhận Default Route `0.0.0.0/0` từ Firewall.
