# TÀI LIỆU QUY HOẠCH ĐỊA CHỈ IP TĨNH / ĐỘNG, PHÂN BỔ VLAN VÀ TỔNG QUAN IP HELPER-ADDRESS

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

## 4. TỔNG QUAN VỀ LỆNH ip helper-address (KHÁI NIỆM & NGUYÊN LÝ HOẠT ĐỘNG)

### 4.1. Lệnh ip helper-address là gì?
- **Định nghĩa:** ip helper-address là câu lệnh được cấu hình trên giao diện mạng Layer 3 (SVI/Interface) của Router hoặc Switch L3 (Cisco Catalyst 9500), đóng vai trò như một **DHCP Relay Agent** (Đại lý chuyển tiếp DHCP).
- **Mục đích:** Giúp các thiết bị máy trạm (Client) ở các VLAN phòng ban khác nhau có thể nhận địa chỉ IP tự động từ **01 Cụm DHCP Server đặt tập trung** tại VLAN 10 Server Farm (IP 192.168.10.13).

### 4.2. Tại sao bắt buộc phải sử dụng ip helper-address?
1. **Xung đột cơ chế Broadcast:** Khi máy trạm khởi động và xin cấp IP, nó gửi gói tin **DHCP Discover** dưới dạng **Broadcast** (255.255.255.255).
2. **Quy tắc chặn Broadcast của Layer 3:** Router và Switch Layer 3 theo nguyên tắc mạng sẽ chặn hoàn toàn các gói Broadcast qua lại giữa các VLAN để chống bão quảng bá (Broadcast Storm) và bảo vệ an ninh.
3. **Thực tế hệ thống:** Toàn bộ doanh nghiệp có **30 VLANs**, nhưng chỉ duy trì **01 Cụm DHCP Server tập trung tại MDF A-202 (VLAN 10)**.
4. **Hậu quả nếu không có IP Helper:** Gói tin Broadcast Discover từ các VLAN người dùng (VLAN 21, 31, 121,...) sẽ bị Core Switch chặn lại, không thể tới được DHCP Server. Máy trạm không nhận được IP và bị rơi vào dải vô hiệu **APIPA (169.254.x.x)**, dẫn đến mất kết nối mạng toàn bộ.

### 4.3. Nguyên lý hoạt động (Chuyển Broadcast thành Unicast & gắn giaddr)
Khi kích hoạt lệnh ip helper-address 192.168.10.13 trên SVI của từng VLAN, Core Switch hoạt động như sau:
- **Chuyển đổi gói tin:** Bắt gói DHCP Discover broadcast từ máy trạm, chuyển đổi thành gói **Unicast** gửi thẳng đến IP của DHCP Server (192.168.10.13).
- **Gắn trường giaddr (Gateway IP Address):** Core Switch chèn địa chỉ IP SVI của VLAN đó (ví dụ VLAN 31 là 192.168.31.1) vào trường giaddr trong gói tin. Nhờ đó, DHCP Server biết chính xác Client đang thuộc VLAN nào để tự động cấp đúng dải IP (Scope/Pool) tương ứng.

`
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
`

---

## 5. KẾT LUẬN & BẢN TÓM TẮT THỰC THI (EXECUTIVE SUMMARY)

1. **Phân định rõ ràng:** Toàn bộ thiết bị mạng (Router, Firewall, Core, Access Switch, ToR), toàn bộ Máy chủ Data Center (VLAN 10), Camera và Kiểm soát cửa (VLAN 12), Máy in văn phòng và Máy trạm SOC đều được **ấn định IP Tĩnh (Static IP)** để duy trì liên lạc và giám sát an ninh 24/7.
2. **IP Động hiệu quả:** 144+ máy tính người dùng, Laptop lãnh đạo, hàng trăm máy thoại Cisco 7841 và các phân vùng Wi-Fi Khách đều sử dụng **IP Động (DHCP)** từ máy chủ tập trung 192.168.10.13.
3. **Cơ chế IP Helper-Address:** Là mắt xích trọng yếu tại tầng Core Switch (Cisco Catalyst 9500), biến đổi gói tin broadcast thành unicast, chèn trường giaddr tương ứng từng VLAN để cụm DHCP Server tại MDF A-202 nhận diện và cấp đúng dải địa chỉ IP cho từng phòng ban.
4. **Vận hành tối ưu:** Giúp quản trị tập trung toàn bộ dải IP của 30 VLAN tại một máy chủ DHCP duy nhất, dễ dàng kiểm soát, cấp phát và thu hồi địa chỉ mà không cần cấu hình phức tạp tại từng phân vùng truy cập.
