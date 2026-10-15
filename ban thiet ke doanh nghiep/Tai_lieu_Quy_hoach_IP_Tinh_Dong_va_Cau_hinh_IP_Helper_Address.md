# TÀI LIỆU QUY HOẠCH ĐỊA CHỈ IP TĨNH / ĐỘNG, PHÂN BỔ VLAN VÀ HƯỚNG DẪN CẤU HÌNH IP HELPER-ADDRESS TRÊN CISCO

> **Dự án:** Thiết Kế Hệ Thống Mạng An Toàn & Giám Sát An Ninh Doanh Nghiệp  
> **Kiến trúc áp dụng:** 2 Tòa nhà (Tòa A & B), 26 Phòng ban, Collapsed-Core 2 Lớp (Cặp Core Switch Cisco Catalyst 9500 SVL), 25 Switch phòng truy cập (C9200L & C9200CX), Server Farm MDF A-202.  
> **Tài liệu tham chiếu:**  
> - [Bang_thong_ke_thiet_bi_va_cap_mang.md](file:///c:/Users/ASUS/Downloads/enterprise-network-security-monitoring/ban%20thiet%20ke%20doanh%20nghiep/Bang_thong_ke_thiet_bi_va_cap_mang.md)  
> - [Phan_tich_toan_bo_phan_cung_doanh_nghiep.md](file:///c:/Users/ASUS/Downloads/enterprise-network-security-monitoring/ban%20thiet%20ke%20doanh%20nghiep/Phan_tich_toan_bo_phan_cung_doanh_nghiep.md)  
> - [index-v2.html](file:///c:/Users/ASUS/Downloads/enterprise-network-security-monitoring/ban%20thiet%20ke%20doanh%20nghiep/index-v2.html)  

---

## 1. TỔNG QUAN VỀ CHIẾN LƯỢC QUY HOẠCH IP (STATIC VS DYNAMIC)

Trong một hệ thống mạng doanh nghiệp quy mô chuẩn hóa (Enterprise Network), việc lựa chọn cấp phát địa chỉ IP tĩnh (**Static IP**) hay cấp phát tự động qua DHCP (**Dynamic IP**) đóng vai trò sống còn trong việc đảm bảo tính sẵn sàng, tính ổn định dịch vụ, khả năng truy vết an ninh và tối ưu hóa quản trị mạng.

```
                          ┌────────────────────────────────────────────────────────┐
                          │               HẠ TẦNG MẠNG DOANH NGHIỆP                │
                          └───────────────────────────┬────────────────────────────┘
                                                      │
                       ┌──────────────────────────────┴──────────────────────────────┐
                       ▼                                                             ▼
         ┌───────────────────────────┐                                 ┌───────────────────────────┐
         │     ĐỊA CHỈ IP TĨNH       │                                 │     ĐỊA CHỈ IP ĐỘNG       │
         │       (STATIC IP)         │                                 │     (DYNAMIC - DHCP)      │
         ├───────────────────────────┤                                 ├───────────────────────────┤
         │ • Router WAN, Firewall    │                                 │ • Máy trạm PC nhân viên   │
         │ • Core Switch, ToR Switch │                                 │ • Laptop nhân viên & VIP  │
         │ • Switch Access (SVI Mgmt)│                                 │ • Điện thoại IP (Option150│
         │ • Toàn bộ Cụm Máy Chủ     │                                 │ • Thiết bị Khách (Guest)  │
         │ • Camera An Ninh & NVR    │                                 │ • Thiết bị di động Wi-Fi  │
         │ • Bộ Điều Khiển Cửa Ra Vào│                                 └───────────────────────────┘
         │ • Máy In Mạng Văn Phòng   │                                               ▲
         │ • Máy Trạm Giám Sát SOC   │                                               │ (ip helper-address)
         └───────────────────────────┘                                 ┌─────────────┴─────────────┐
                                                                       │ DHCP Server Failover      │
                                                                       │ IP: 192.168.10.13/24      │
                                                                       │ Đặt tại VLAN 10 MDF A-202 │
                                                                       └───────────────────────────┘
```

### 1.1. Tiêu chí phân định thiết bị dùng IP Tĩnh (Static IP)
- **Thiết bị hạ tầng truyền dẫn mạng & an ninh biên:** Cần địa chỉ cố định để duy trì các phiên định tuyến (OSPF, BGP), liên kết dự phòng (IP SLA Tracking, HSRP, SVL), giám sát trạng thái qua SNMP và gửi cảnh báo Syslog.
- **Hệ thống Máy chủ Dịch vụ (Data Center Servers):** Cần IP tĩnh để các bản ghi phân giải tên miền nội bộ (Internal DNS Records) luôn trỏ chính xác, các dịch vụ phân quyền (Active Directory, Kerberos, RADIUS) không bị ngắt quãng.
- **Hệ thống An ninh vật lý & IoT đặc thù:** Camera IP truyền luồng RTSP về VMS Server, Đầu đọc thẻ và Bộ điều khiển cửa giao tiếp Socket cố định về ZKBioSecurity Server. Nếu thay đổi IP, luồng ghi hình và cơ chế mở cửa khẩn cấp sẽ bị gián đoạn.
- **Hệ thống Máy in mạng:** Giúp toàn bộ máy trạm người dùng kết nối in ấn qua giao thức IPP/RAW/LPR ổn định mà không bị mất kết nối máy in khi hết chu kỳ thuê IP (DHCP Lease Time).

### 1.2. Tiêu chí phân định thiết bị dùng IP Động (Dynamic IP - DHCP)
- **Máy trạm người dùng cuối (Dell OptiPlex), Laptop và Thiết bị di động:** Số lượng lớn (144+ máy tính, hàng trăm điện thoại cá nhân). Việc cấu hình thủ công sẽ tốn nhiều công sức, dễ xảy ra lỗi đụng độ IP (IP Conflict). DHCP tự động cung cấp IP, Subnet Mask, Default Gateway, DNS Servers và Domain Name.
- **Điện thoại IP VoIP (Cisco IP Phone 7841):** Cần nhận IP động từ Voice VLAN kèm theo **DHCP Option 150** (chứa địa chỉ IP của máy chủ tổng đài SIP/CUCM `192.168.10.16`) để tự động tải cấu hình (Firmware & XML Config).
- **Phân vùng Khách (Guest Wi-Fi):** Khách hàng vãng lai thường xuyên ra vào công ty, cần thời gian thuê IP ngắn (2–4 giờ) để thu hồi và tái sử dụng IP, tránh cạn kiệt tài nguyên mạng.

---

## 2. BẢNG PHÂN LOẠI CHI TIẾT THIẾT BỊ DOANH NGHIỆP: TĨNH VS ĐỘNG

Dưới đây là bảng thống kê toàn bộ thiết bị theo chuẩn kiến trúc của dự án, quy định rõ phương thức cấu hình IP, VLAN trực thuộc và dải mạng quy hoạch:

| STT | Nhóm thiết bị | Model phần cứng | Số lượng | Phương thức IP | VLAN trực thuộc | Dải mạng Subnet / Ghi chú |
|:---:|:---|:---|:---:|:---:|:---:|:---|
| **I** | **HẠ TẦNG MẠNG & AN NINH LÕI (MDF A-202)** | | | | | |
| 1 | Router WAN Biên | Cisco Catalyst C8300-1N1S-4T2X | 02 bộ | **IP TĨNH** | Transit WAN / OSPF | `10.255.0.0/29` (Transit Firewall) & Public IP từ ISP1/ISP2. |
| 2 | Tường Lửa NGFW | Fortinet FortiGate FG-200F | 02 bộ | **IP TĨNH** | Transit WAN / VLAN 2 | `10.255.0.2/29` (Transit OSPF), `192.168.2.254/24` (Quản trị). |
| 3 | Core Switch Layer 3 | Cisco Catalyst C9500-24Y4C (SVL) | 02 bộ | **IP TĨNH** | Toàn bộ 30 VLANs | Là Default Gateway (`.1`) của 30 VLANs. IP SVI Mgmt: `192.168.2.1/24`. |
| 4 | Switch ToR Data Center | Cisco Catalyst C9300-24S-E | 02 bộ | **IP TĨNH** | VLAN 2 (Mgmt) | `192.168.2.2/24` & `192.168.2.3/24` phục vụ quản trị OOB. |
| **II** | **CHUYỂN MẠCH TRUY CẬP (ACCESS SWITCHES PHÒNG BAN)** | | | | | |
| 5 | Switch 16 phòng lớn | Cisco Catalyst C9200L-24P-4X | 16 bộ | **IP TĨNH** | VLAN 2 (Mgmt) | Cấu hình IP tĩnh trên `interface Vlan2` (`192.168.2.11` đến `.26`) để Zabbix giám sát SNMP và SSH bảo mật. |
| 6 | Switch 09 phòng nhỏ & họp| Cisco Catalyst C9200CX-8P-2X2G | 09 bộ | **IP TĨNH** | VLAN 2 (Mgmt) | Cấu hình IP tĩnh trên `interface Vlan2` (`192.168.2.27` đến `.35`) để quản trị tập trung. |
| **III** | **MẠNG KHÔNG DÂY (ENTERPRISE WLAN)** | | | | | |
| 7 | Bộ phát sóng Wi-Fi AP | Ubiquiti UniFi U6-Pro | 25 bộ | **IP TĨNH (hoặc DHCP Reservation)** | VLAN 2 (Mgmt) | IP tĩnh thuộc VLAN 2 (`192.168.2.51` đến `.75`) kết nối về UniFi Controller `192.168.10.15`. |
| 8 | Người dùng Wi-Fi nội bộ | Laptop / Smartphone nhân viên | ~200 | **IP ĐỘNG (DHCP)** | Dynamic VLAN | Xác thực 802.1X/RADIUS, tự động gán vào VLAN phòng ban nghiệp vụ tương ứng qua DHCP. |
| 9 | Người dùng Wi-Fi Khách | Khách vãng lai, đối tác | Tùy biến | **IP ĐỘNG (DHCP)** | VLAN 100, 110, 111, 143 | Nhận IP động dải `172.16.x.0/24`, lease time 2 giờ, cô lập người dùng (Client Isolation). |
| **IV** | **CỤM MÁY CHỦ DỊCH VỤ DATA CENTER (MDF A-202)** | | | | | |
| 10 | AD / DNS Chính | Dell PowerEdge R660 (02 VM) | 02 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.10/24` (DC01) & `192.168.10.11/24` (DC02). |
| 11 | **DHCP Server Failover** | Dell PowerEdge R660 (02 VM) | 02 VM | **IP TĨNH** | **VLAN 10 (Server Farm)**| `192.168.10.13/24` **(ĐÍCH ĐẾN CỦA IP HELPER-ADDRESS)**. |
| 12 | Giám sát NMS Zabbix | Dell PowerEdge R660 (VM Linux) | 01 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.12/24`. |
| 13 | Xác thực RADIUS / NAC | Dell PowerEdge R660 (02 VM) | 02 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.14/24`. |
| 14 | UniFi Controller | Dell PowerEdge R660 (VM Linux) | 01 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.15/24`. |
| 15 | Tổng đài thoại IP-PBX SIP | Dell PowerEdge R660 (VM Asterisk)| 01 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.16/24` (Khai báo trong DHCP Option 150). |
| 16 | Máy chủ Camera VMS | Dell PowerEdge R760 (Vật lý) | 01 máy | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.17/24`. |
| 17 | Quản trị cửa ZKBioSecurity | Dell PowerEdge R660 (VM Windows)| 01 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.18/24`. |
| 18 | Cổng Web Portal nội bộ | Dell PowerEdge R760 (VM Cluster)| 02 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.20/24`. |
| 19 | Mail Server Doanh nghiệp | Dell PowerEdge R760 (VM Zimbra) | 01 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.25/24`. |
| 20 | Database Server Master/Rep| Dell PowerEdge R760 (02 VM) | 02 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.30/24` (Master) & `192.168.10.31/24` (Replica). |
| 21 | File Gateway Server | Dell PowerEdge R760 (02 VM) | 02 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.35/24`. |
| 22 | Wazuh SIEM Manager & Index| Dell PowerEdge R760 (VM Linux) | 01 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.40/24`. |
| 23 | Bastion Host (Jump Box) | Dell PowerEdge R660 (Hardened) | 01 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.42/24` (Bắt buộc xác thực MFA). |
| 24 | Rsyslog Server | Dell PowerEdge R660 (VM Linux) | 01 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.50/24`. |
| 25 | Veeam Backup Server | Dell PowerEdge R760 (Vật lý) | 01 máy | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.60/24`. |
| 26 | Cụm Kubernetes Production | Dell PowerEdge R760 (03 VM) | 03 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.80` đến `.82/24`. |
| 27 | Ứng dụng CRM, ERP, HRM | Dell PowerEdge R760/R660 | 03 VM | **IP TĨNH** | VLAN 10 (Server Farm) | `192.168.10.100`, `.110`, `.120/24`. |
| 28 | Mảng lưu trữ SAN Storage | Dell ME5024 & ME5012 | 02 mảng | **IP TĨNH** | VLAN 2 (Mgmt) & SAN iSCSI| Controller A/B IP tĩnh quản trị và IP iSCSI Data Path riêng. |
| **V** | **THIẾT BỊ AN NINH VẬT LÝ & IOT NGOẠI VI** | | | | | |
| 29 | Camera An Ninh IP Dome | Hikvision DS-2CD2143G2-I | 26 chiếc | **IP TĨNH (hoặc DHCP Reservation)** | VLAN 12 (Security_IoT) | `192.168.12.11` đến `.36/24`. Không cho ra Internet. |
| 30 | Bộ điều khiển cửa 2 cửa | ZKTeco InBio-260 | 13 bộ | **IP TĨNH** | VLAN 12 (Security_IoT) | `192.168.12.51` đến `.63/24` kết nối về máy chủ ZKBio. |
| 31 | Máy In Mạng Doanh Nghiệp | HP LaserJet Enterprise M507dn | 20 máy | **IP TĨNH** | Thuộc VLAN phòng ban | Gán IP tĩnh phần đầu dải của từng phòng (Ví dụ: `192.168.x.5/24`) hoặc Reservation. |
| **VI** | **THIẾT BỊ ĐẦU CUỐI NGƯỜI DÙNG & VĂN PHÒNG** | | | | | |
| 32 | Máy trạm nhân viên | Dell OptiPlex Plus 7020 SFF | 144 bộ | **IP ĐỘNG (DHCP)** | Thuộc VLAN từng phòng ban | Nhận IP từ Scope tương ứng qua Relay Agent trên Core Switch (`.50` đến `.240`). |
| 33 | Máy trạm NOC/SOC | Dell Precision 3680 Dual Screen| 04 bộ | **IP TĨNH** | VLAN 44 (NOC/SOC) | `192.168.44.11` đến `.14/24` (Định danh giám sát đặc quyền). |
| 34 | Trạm quản trị Out-of-Band | Dell OptiPlex 7020 Dual NIC | 01 bộ | **IP TĨNH** | VLAN 2 (Mgmt) | `192.168.2.200/24` tại bàn kỹ thuật MDF A-202. |
| 35 | Laptop Lãnh Đạo VIP | Dell XPS 15 / 16 (CEO & CTO) | 02 chiếc | **IP ĐỘNG (hoặc DHCP Reservation)** | VLAN 52 (Executive) | Nhận IP dải `192.168.52.0/24` kèm quyền truy cập ưu tiên. |
| 36 | Điện thoại IP Bàn | Cisco IP Phone 7841 | 25 máy | **IP ĐỘNG (DHCP)** | VLAN 200 (A) / VLAN 210 (B) | Nhận IP Voice tự động kèm Option 150 trỏ IP-PBX `192.168.10.16`. |
| 37 | Điện thoại Hội nghị IP | Cisco IP Phone 7832 | 02 máy | **IP ĐỘNG (DHCP)** | VLAN 200 (A) / VLAN 210 (B) | Nhận IP Voice tự động tại phòng họp A-503 và B-102. |

---

## 3. BẢNG TỔNG HỢP 30 VLANS, ZONE & PHƯƠNG THỨC CẤP PHÁT IP

Toàn bộ 30 VLAN được cấu hình SVI (Switch Virtual Interface) làm Default Gateway trên Cặp Core Switch Cisco Catalyst 9500 ghép StackWise Virtual:

| VLAN ID | Tên VLAN | Phân vùng (Zone) | Dải Subnet | Default Gateway (Core SVI) | Kiểu cấp IP chính | Cần cấu hình `ip helper-address`? |
|:---:|:---|:---|:---|:---:|:---:|:---:|
| **VLAN 2** | Network_Mgmt | Infra/Mgmt | `192.168.2.0/24` | `192.168.2.1` | **Static IP 100%** | **KHÔNG** (Thiết bị mạng đặt tĩnh) |
| **VLAN 10** | Server_Farm | Infra/Mgmt | `192.168.10.0/24` | `192.168.10.1` | **Static IP 100%** | **KHÔNG** (Chính là nơi đặt DHCP Server) |
| **VLAN 11** | Security_Guard (A-102)| Infra | `192.168.11.0/24` | `192.168.11.1` | DHCP + Static | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 12** | Security_IoT (Cam/Door)| Infra | `192.168.12.0/24` | `192.168.12.1` | Static / Reservation | **TÙY CHỌN** (Khuyến nghị đặt tĩnh) |
| **VLAN 21** | Lab_RD (A-201) | User | `192.168.21.0/24` | `192.168.21.1` | **DHCP** (8 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 23** | IT_Helpdesk (A-203) | Infra | `192.168.23.0/24` | `192.168.23.1` | **DHCP** (6 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 31** | Backend_Dev (A-301) | User | `192.168.31.0/24` | `192.168.31.1` | **DHCP** (12 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 32** | Data_Engineer (A-302)| Restricted | `192.168.32.0/24` | `192.168.32.1` | **DHCP** (8 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 33** | Frontend_Dev (A-303) | User | `192.168.33.0/24` | `192.168.33.1` | **DHCP** (10 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 41** | QA_Tester (A-401) | User | `192.168.41.0/24` | `192.168.41.1` | **DHCP** (10 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 42** | UI_UX_Design (A-402) | User | `192.168.42.0/24` | `192.168.42.1` | **DHCP** (8 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 43** | DevOps_Admin (A-403) | Restricted | `192.168.43.0/24` | `192.168.43.1` | **DHCP** (6 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 44** | NOC_SOC_Team (A-404) | Restricted | `192.168.44.0/24` | `192.168.44.1` | **Static IP** (4 SOC PC) | **TÙY CHỌN** (Khai báo dự phòng) |
| **VLAN 51** | Product_Mgmt (A-501) | User | `192.168.51.0/24` | `192.168.51.1` | **DHCP** (8 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 52** | Executive_BOD (A-502)| Executive | `192.168.52.0/24` | `192.168.52.1` | **DHCP / Reservation** | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 53** | Meeting_BOD (A-503) | Executive | `192.168.53.0/24` | `192.168.53.1` | **DHCP** (Họp nội bộ)| **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 100**| Guest_Lobby_A (A-101)| Guest | `172.16.100.0/24` | `172.16.100.1` | **DHCP** (Lease 2h)| **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 110**| Guest_Lobby_B (B-101)| Guest | `172.16.110.0/24` | `172.16.110.1` | **DHCP** (Lease 2h)| **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 111**| Guest_Meeting_B(B-102)| Guest | `172.16.111.0/24` | `172.16.111.1` | **DHCP** (Lease 2h)| **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 121**| Sales_Team (B-201) | User | `192.168.121.0/24`| `192.168.121.1` | **DHCP** (14 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 122**| Marketing_Team(B-202)| User | `192.168.122.0/24`| `192.168.122.1` | **DHCP** (10 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 131**| HR_Admin (B-301) | Restricted | `192.168.131.0/24`| `192.168.131.1` | **DHCP** (6 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 132**| Accounting_Fin(B-302)| Restricted | `192.168.132.0/24`| `192.168.132.1` | **DHCP** (8 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 133**| Legal_Admin (B-303) | User | `192.168.133.0/24`| `192.168.133.1` | **DHCP** (4 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 134**| Customer_Care (B-304)| User | `192.168.134.0/24`| `192.168.134.1` | **DHCP** (12 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 141**| Project_PM_BA (B-401)| User | `192.168.141.0/24`| `192.168.141.1` | **DHCP** (8 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 142**| Training_Room (B-402)| User | `192.168.142.0/24`| `192.168.142.1` | **DHCP** (6 PC) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 143**| Canteen_Staff (B-403)| Guest | `172.16.143.0/24` | `172.16.143.1` | **DHCP** (Lease 4h)| **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 200**| Voice_VoIP_A | Voice | `192.168.200.0/24`| `192.168.200.1` | **DHCP** (Option 150) | **CÓ** (`ip helper-address 192.168.10.13`) |
| **VLAN 210**| Voice_VoIP_B | Voice | `192.168.210.0/24`| `192.168.210.1` | **DHCP** (Option 150) | **CÓ** (`ip helper-address 192.168.10.13`) |

---

## 4. BẢN CHẤT & NGUYÊN LÝ HOẠT ĐỘNG CỦA LỆNH `ip helper-address` TRÊN CISCO

### 4.1. Vấn đề thực tế trong mạng đa VLAN (Multi-VLAN Architecture)
Quá trình xin cấp phát IP thông qua DHCP tuân theo tiến trình **DORA** (Discover $\rightarrow$ Offer $\rightarrow$ Request $\rightarrow$ Acknowledge):
1. Khi máy trạm (Client) khởi động, nó chưa có địa chỉ IP và chưa biết DHCP Server nằm ở đâu.
2. Client phát một gói tin **DHCP Discover** dưới dạng **Broadcast Layer 2** (`FF:FF:FF:FF:FF:FF`) và **Broadcast Layer 3** (`255.255.255.255`, UDP Source Port 68, Destination Port 67).
3. **Quy tắc cơ bản của mạng:** Router hoặc Switch Layer 3 ngăn chặn gói Broadcast qua lại giữa các VLAN để chống bão quảng bá (Broadcast Storm) và bảo vệ an ninh.
4. **Hậu quả:** Nếu doanh nghiệp có 30 VLANs nhưng chỉ đặt **01 Cụm DHCP Server tập trung tại VLAN 10 (IP `192.168.10.13`)**, thì các máy tính thuộc VLAN 21, 31, 121,... sẽ bị Core Switch chặn gói Broadcast Discover. **Kết quả là Client không thể xin được địa chỉ IP và sẽ tự gán dải vô hiệu APIPA (`169.254.x.x`)**.

```
[VLAN 31 CLIENT]                              [CORE SWITCH L3 C9500]                       [DHCP SERVER]
 (192.168.31.x)                                (SVI 31: 192.168.31.1)                  (VLAN 10: 192.168.10.13)
        │                                                │                                         │
        │─── 1. DHCP Discover (BROADCAST) ──────────────>│                                         │
        │    Src: 0.0.0.0:68                             │                                         │
        │    Dst: 255.255.255.255:67                     │ 2. Router chặn Broadcast!               │
        │                                                │    NHƯNG có lệnh:                       │
        │                                                │    "ip helper-address 192.168.10.13"    │
        │                                                │    Chuyển gói tin thành UNICAST         │
        │                                                │    Gán giaddr = 192.168.31.1            │
        │                                                │                                         │
        │                                                │─── 3. DHCP Discover (UNICAST) ─────────>│
        │                                                │    Src: 192.168.31.1                    │
        │                                                │    Dst: 192.168.10.13                   │
        │                                                │    giaddr: 192.168.31.1                 │
        │                                                │                                         │
        │                                                │                                4. Server đọc giaddr
        │                                                │                                   Biết Client thuộc
        │                                                │                                   Pool 192.168.31.0/24!
        │                                                │<── 5. DHCP Offer (UNICAST) ─────────────│
        │<── 6. DHCP Offer ──────────────────────────────│    Cấp IP: 192.168.31.55                │
        │                                                │                                         │
```

### 4.2. Cách thức giải quyết của `ip helper-address` (DHCP Relay Agent)
Khi kỹ sư cấu hình lệnh `ip helper-address <DHCP_Server_IP>` trên Interface VLAN (SVI) của Core Switch, Core Switch sẽ đóng vai trò là một **DHCP Relay Agent**:
1. **Lắng nghe & Bắt gói:** Core Switch lắng nghe các gói tin UDP Broadcast gửi đến cổng 67 trên SVI đó.
2. **Chuyển đổi từ Broadcast sang Unicast:** Thay vì thả gói (Drop), Core Switch biến đổi gói tin broadcast thành gói tin Unicast có:
   - **Source IP:** Địa chỉ IP của chính SVI đó (Ví dụ: `192.168.31.1`).
   - **Destination IP:** Địa chỉ IP của máy chủ DHCP Server đích (`192.168.10.13`).
3. **Chèn trường `giaddr` (Gateway IP Address):** Đây là bước **quan trọng nhất**. Core Switch ghi địa chỉ IP SVI của mình (ví dụ: `192.168.31.1`) vào trường `giaddr` trong phần tải trọng DHCP.
4. **Phân bổ đúng Pool trên Server:** Khi DHCP Server tại VLAN 10 nhận được gói tin, nó kiểm tra trường `giaddr`:
   - Thấy `giaddr = 192.168.31.1`, Server lập tức biết Client này đang kết nối vào VLAN 31.
   - Server tự động trích xuất địa chỉ IP trống từ **Scope/Pool `192.168.31.0/24`** để cấp phát, thay vì cấp nhầm IP của VLAN 10 hay các VLAN khác!
5. **Chuyển tiếp phản hồi (Offer/ACK):** DHCP Server gửi gói tin phản hồi Unicast về IP `giaddr`. Core Switch nhận được sẽ chuyển tiếp lại gói tin đến máy trạm ban đầu.

### 4.3. 8 Dịch vụ UDP mặc định mà lệnh `ip helper-address` chuyển tiếp
Khi kích hoạt `ip helper-address`, mặc định Cisco IOS sẽ chuyển tiếp 8 cổng UDP sau:
1. **TFTP** (Cổng 69)
2. **DNS** (Cổng 53)
3. **Time service** (Cổng 37)
4. **TACACS** (Cổng 49)
5. **BOOTP/DHCP Server** (Cổng 67)
6. **BOOTP/DHCP Client** (Cổng 68)
7. **NetBIOS Name Service** (Cổng 137)
8. **NetBIOS Datagram Service** (Cổng 138)

> [!TIP]
> **Khuyến nghị bảo mật:** Nếu hệ thống chỉ dùng DHCP, ta nên tắt các dịch vụ broadcast không cần thiết như NetBIOS để tránh lãng phí băng thông và phòng chống rò rỉ thông tin mạng bằng lệnh toàn cục:  
> `no ip forward-protocol udp 137`  
> `no ip forward-protocol udp 138`  
> `no ip forward-protocol udp 49`  

---

## 5. BỘ CÂU LỆNH CẤU HÌNH THỰC TẾ ĐẦY ĐỦ TRÊN CISCO (FULL CLI SCRIPT)

### 5.1. Kịch bản 1: Cấu hình `ip helper-address` trên Cặp Core Switch Cisco Catalyst 9500
*(Thực hiện cấu hình trên Core Switch L3 - Nơi đặt SVI Default Gateway cho các VLAN)*

```cisco
! ====================================================================
! BẬT ĐỊNH TUYẾN LAYER 3 VÀ BẢO MẬT GIAO THỨC CHUYỂN TIẾP UDP
! ====================================================================
configure terminal
ip routing

! Chỉ cho phép chuyển tiếp gói DHCP (Cổng 67/68), tắt chuyển tiếp các port UDP rác
no ip forward-protocol udp 137   ! NetBIOS Name Service
no ip forward-protocol udp 138   ! NetBIOS Datagram
no ip forward-protocol udp 49    ! TACACS
no ip forward-protocol udp 37    ! Time

! ====================================================================
! CẤU HÌNH SVI VLAN 10 (SERVER FARM - NƠI ĐẶT DHCP SERVER)
! Lưu ý: Trên VLAN này KHÔNG CẦN cấu hình ip helper-address vì
! DHCP Server 192.168.10.13 nằm trực tiếp cùng mạng nội bộ VLAN 10.
! ====================================================================
interface Vlan10
 description [INFRA] SERVER FARM DATA CENTER (MDF A-202)
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

! ====================================================================
! CẤU HÌNH IP HELPER-ADDRESS CHO CÁC PHÒNG BAN TÒA A
! ====================================================================
interface Vlan11
 description [INFRA] PHONG AN NINH / BAO VE (A-102)
 ip address 192.168.11.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan21
 description [USER] PHONG LAB / R&D (A-201)
 ip address 192.168.21.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan23
 description [INFRA] PHONG IT HELPDESK (A-203)
 ip address 192.168.23.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan31
 description [USER] PHONG LAP TRINH BACKEND (A-301)
 ip address 192.168.31.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan32
 description [RESTRICTED] PHONG DATABASE / DATA ENGINEER (A-302)
 ip address 192.168.32.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan33
 description [USER] PHONG LAP TRINH FRONTEND (A-303)
 ip address 192.168.33.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan41
 description [USER] PHONG KIEM THU QA/TESTER (A-401)
 ip address 192.168.41.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan42
 description [USER] PHONG THIET KE UI/UX (A-402)
 ip address 192.168.42.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan43
 description [RESTRICTED] PHONG DEVOPS / SYSADMIN (A-403)
 ip address 192.168.43.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan44
 description [RESTRICTED] TRUNG TAM DIEU HANH AN NINH NOC/SOC (A-404)
 ip address 192.168.44.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan51
 description [USER] PHONG QUAN LY SAN PHAM PRODUCT (A-501)
 ip address 192.168.51.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan52
 description [EXECUTIVE] PHONG BAN GIAM DOC CEO/CTO (A-502)
 ip address 192.168.52.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan53
 description [EXECUTIVE] PHONG HOP BAN GIAM DOC (A-503)
 ip address 192.168.53.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

! ====================================================================
! CẤU HÌNH IP HELPER-ADDRESS CHO CÁC PHÒNG BAN TÒA B
! ====================================================================
interface Vlan121
 description [USER] PHONG KINH DOANH (B-201)
 ip address 192.168.121.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan122
 description [USER] PHONG MARKETING (B-202)
 ip address 192.168.122.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan131
 description [RESTRICTED] PHONG NHAN SU HR (B-301)
 ip address 192.168.131.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan132
 description [RESTRICTED] PHONG KE TOAN / TAI CHINH (B-302)
 ip address 192.168.132.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan133
 description [USER] PHONG PHAP CHE / HANH CHINH (B-303)
 ip address 192.168.133.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan134
 description [USER] PHONG CHAM SOC KHACH HANG CSKH (B-304)
 ip address 192.168.134.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan141
 description [USER] PHONG TRIEN KHAI DU AN PM/BA (B-401)
 ip address 192.168.141.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan142
 description [USER] PHONG DAO TAO NOI BO (B-402)
 ip address 192.168.142.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

! ====================================================================
! CẤU HÌNH IP HELPER-ADDRESS CHO CÁC PHÂN VÙNG GUEST (KHÁCH VÃNG LAI)
! ====================================================================
interface Vlan100
 description [GUEST] SANH LE TAN TOA A (A-101)
 ip address 172.16.100.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan110
 description [GUEST] SANH TIEP KHACH TOA B (B-101)
 ip address 172.16.110.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan111
 description [GUEST] PHONG HOP DOI TAC TOA B (B-102)
 ip address 172.16.111.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan143
 description [GUEST] CANG TIN & KHU NGHI TOA B (B-403)
 ip address 172.16.143.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

! ====================================================================
! CẤU HÌNH IP HELPER-ADDRESS CHO PHÂN HỆ THOẠI VOICE VLAN (CISCO 7841)
! ====================================================================
interface Vlan200
 description [VOICE] VOICE VLAN TOA NHA A
 ip address 192.168.200.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit

interface Vlan210
 description [VOICE] VOICE VLAN TOA NHA B
 ip address 192.168.210.1 255.255.255.0
 ip helper-address 192.168.10.13
 no shutdown
exit
```

---

### 5.2. Kịch bản 2: Cấu hình bảo mật DHCP Snooping trên Access Switch (C9200L / C9200CX)
> [!CAUTION]
> **Nguy cơ Rogue DHCP Server:** Kẻ tấn công có thể cắm một chiếc Router Wi-Fi cá nhân hoặc phần mềm giả mạo DHCP vào một cổng mạng phòng ban để cấp phát Default Gateway giả mạo (tấn công Man-in-the-Middle) hoặc DNS giả mạo.  
> **Giải pháp:** Bắt buộc kích hoạt tính năng **DHCP Snooping** trên toàn bộ 25 switch phòng ban. Mặc định toàn bộ các cổng nối đến máy tính nhân viên bị coi là `Untrusted` (chặn gói DHCP Offer/ACK từ cổng này), chỉ duy nhất cổng Uplink quang 10G kết nối về Core Switch được cấu hình `Trusted`.

```cisco
! ====================================================================
! KÍCH HOẠT DHCP SNOOPING TOÀN CỤC TRÊN SWITCH PHÒNG
! ====================================================================
configure terminal

! 1. Bật tính năng DHCP Snooping
ip dhcp snooping

! 2. Áp dụng cho toàn bộ các VLAN người dùng trên switch
ip dhcp snooping vlan 11,21,23,31,32,33,41,42,43,44,51,52,53,100,110,111,121,122,131,132,133,134,141,142,143,200,210

! 3. Tắt chèn Option 82 nếu DHCP Server không hỗ trợ hoặc Core Switch từ chối
no ip dhcp snooping information option

! ====================================================================
! THIẾT LẬP CỔNG TIN CẬY (TRUSTED PORT) - CỔNG UPLINK QUANG 10G VỀ CORE
! ====================================================================
interface Te1/1/1
 description [UPLINK] 10G SFP+ TO CORE C9500 (MDF A-202)
 switchport mode trunk
 ip dhcp snooping trust
exit

! ====================================================================
! CẤU HÌNH CỔNG ACCESS NỐI MÁY TRẠM & CISCO IP PHONE (UNTRUSTED PORT)
! Giới hạn tốc độ gói DHCP để chống tấn công cạn kiệt tài nguyên (DHCP Starvation)
! ====================================================================
interface range Gi1/0/1 - 24
 switchport mode access
 switchport voice vlan 200     ! Hoặc VLAN 210 đối với Tòa B
 ip dhcp snooping limit rate 15 ! Tối đa 15 gói DHCP/giây trên mỗi cổng
exit
```

---

### 5.3. Kịch bản 3: Cấu hình Khai báo DHCP Server Pools (Ví dụ cấu hình Scope)

Hệ thống có thể chạy DHCP Server trên máy ảo Windows Server 2022 Failover Cluster hoặc Linux Kea / ISC-DHCP Server (IP `192.168.10.13`). Dưới đây là cấu hình tham chiếu:

#### Trường hợp 3A: Mẫu cấu hình Scope trên Linux ISC-DHCP Server (`/etc/dhcp/dhcpd.conf`)
```conf
# Cấu hình tham số toàn cục
option domain-name "corp.enterprise.local";
option domain-name-servers 192.168.10.10, 192.168.10.11;
default-lease-time 28800;    # 8 tiếng cho mạng người dùng
max-lease-time 86400;

# Khai báo Option 150 cho Cisco IP Phone
option cisco-tftp-servers code 150 = array of ip-address;

# 1. SCOPE CHO PHÒNG LẬP TRÌNH BACKEND (VLAN 31)
subnet 192.168.31.0 netmask 255.255.255.0 {
    range 192.168.31.50 192.168.31.240;
    option routers 192.168.31.1;             # SVI Gateway trên Core Switch
    option subnet-mask 255.255.255.0;
}

# 2. SCOPE CHO VOICE VLAN TÒA A (VLAN 200) - CẤP CHO CISCO IP PHONE 7841
subnet 192.168.200.0 netmask 255.255.255.0 {
    range 192.168.200.50 192.168.200.200;
    option routers 192.168.200.1;
    option subnet-mask 255.255.255.0;
    option cisco-tftp-servers 192.168.10.16; # Trỏ về IP của Tổng đài IP-PBX SIP
}

# 3. SCOPE CHO MẠNG GUEST SẢNH A (VLAN 100)
subnet 172.16.100.0 netmask 255.255.255.0 {
    range 172.16.100.50 172.16.100.250;
    option routers 172.16.100.1;
    option domain-name-servers 8.8.8.8, 1.1.1.1; # Guest dùng DNS công cộng
    default-lease-time 7200;                    # Thu hồi IP nhanh sau 2 giờ
    max-lease-time 14400;
}
```

#### Trường hợp 3B: Cấu hình trực tiếp trên Cisco IOS (Nếu Core Switch kiêm luôn DHCP Server dự phòng)
```cisco
! Loại trừ các địa chỉ IP tĩnh (Gateway, Máy in, Thiết bị đặt trước)
ip dhcp excluded-address 192.168.31.1 192.168.31.49
ip dhcp excluded-address 192.168.31.241 192.168.31.254

! Tạo Pool cấp phát cho VLAN 31
ip dhcp pool POOL_VLAN31_BACKEND
 network 192.168.31.0 255.255.255.0
 default-router 192.168.31.1
 dns-server 192.168.10.10 192.168.10.11
 domain-name corp.enterprise.local
 lease 0 8 0
exit

! Tạo Pool cấp phát cho Voice VLAN kèm Option 150
ip dhcp pool POOL_VOICE_A
 network 192.168.200.0 255.255.255.0
 default-router 192.168.200.1
 option 150 ip 192.168.10.16
 lease 7 0 0
exit
```

---

## 6. CÁC CÂU LỆNH KIỂM TRA, GIÁM SÁT VÀ XỬ LÝ SỰ CỐ (TROUBLESHOOTING)

### 6.1. Kiểm tra trên Core Switch Cisco Catalyst 9500
```cisco
! 1. Kiểm tra địa chỉ ip helper-address đã nhận trên các SVI hay chưa
show ip interface Vlan31 | include Helper
! Kết quả mong đợi: Helper address is 192.168.10.13

! 2. Xem thống kê lưu lượng gói DHCP chuyển tiếp qua Relay Agent
show ip dhcp relay information trusted-sources
show ip dhcp relay statistics

! 3. Bật debug gói tin DHCP chuyển tiếp thời gian thực (Chỉ bật khi cần gỡ rối)
debug ip dhcp server packet
terminal monitor
! Tắt debug sau khi kiểm tra xong:
undebug all
```

### 6.2. Kiểm tra trên Switch Truy Cập Access (C9200L / C9200CX)
```cisco
! 1. Kiểm tra trạng thái hoạt động của DHCP Snooping
show ip dhcp snooping

! 2. Xem bảng liên kết MAC - IP - Port - VLAN được DHCP Snooping ghi nhận
show ip dhcp snooping binding

! 3. Kiểm tra các gói tin DHCP trái phép bị drop trên cổng Untrusted
show ip dhcp snooping statistics
```

### 6.3. Bảng mã lỗi thường gặp và cách xử lý nhanh

| Hiện tượng lỗi | Nguyên nhân cốt lõi | Cách xử lý dứt điểm |
|:---|:---|:---|
| **Client nhận IP dải `169.254.x.x` (APIPA)** | SVI trên Core Switch quên cấu hình `ip helper-address`, hoặc DHCP Server `192.168.10.13` bị tắt / chặn cổng UDP 67 bởi Firewall. | 1. Kiểm tra `ip helper-address 192.168.10.13` trên SVI tương ứng.<br>2. Ping từ Core Switch đến `192.168.10.13`.<br>3. Kiểm tra service DHCP daemon trên máy chủ. |
| **DHCP Server nhận gói Discover nhưng không cấp IP** | DHCP Server thiếu Scope cho dải subnet tương ứng với IP SVI trong trường `giaddr`. | Mở trình quản lý DHCP trên Server, tạo Scope mới khớp chính xác với Network ID của VLAN xin cấp (Ví dụ: Scope `192.168.31.0/24`). |
| **Client bị từ chối cấp IP dù Server vẫn chạy** | Switch truy cập bật DHCP Snooping nhưng cổng Uplink nối về Core Switch chưa đặt lệnh `ip dhcp snooping trust`. | Truy cập cổng Uplink (ví dụ `Te1/1/1`) trên Access Switch, gõ: `ip dhcp snooping trust`. |
| **Điện thoại IP Cisco 7841 không đăng ký được SIP** | Voice VLAN thiếu cấu hình **Option 150** trỏ về máy chủ IP-PBX `192.168.10.16`. | Khai báo thêm `option 150 ip 192.168.10.16` vào DHCP Scope của VLAN 200 và VLAN 210. |

---

## 7. KẾT LUẬN & BẢN TÓM TẮT THỰC THI (EXECUTIVE SUMMARY)

1. **Phân định rõ ràng:** Toàn bộ thiết bị mạng (Router, Firewall, Core, Access Switch, ToR), toàn bộ Máy chủ Data Center (VLAN 10), Camera và Kiểm soát cửa (VLAN 12), Máy in văn phòng và Máy trạm SOC đều được **ấn định IP Tĩnh (Static IP)** để duy trì liên lạc và giám sát an ninh 24/7.
2. **IP Động hiệu quả:** 144+ máy tính người dùng, Laptop lãnh đạo, hàng trăm máy thoại Cisco 7841 và các phân vùng Wi-Fi Khách đều sử dụng **IP Động (DHCP)** từ máy chủ tập trung `192.168.10.13`.
3. **Cơ chế IP Helper-Address:** Là mắt xích trọng yếu tại tầng Core Switch (Cisco Catalyst 9500), biến đổi gói tin broadcast thành unicast, chèn trường `giaddr` tương ứng từng VLAN để cụm DHCP Server tại MDF A-202 nhận diện và cấp đúng dải địa chỉ IP cho từng phòng ban.
4. **Bảo mật toàn diện:** Kết hợp chặt chẽ giữa **`ip helper-address` trên Core Switch** và **`DHCP Snooping` trên 25 Access Switch**, tạo nên một hệ thống mạng phân phối IP hoàn toàn tự động, ngăn chặn triệt để tấn công Rogue DHCP Server và mã độc quét mạng.
