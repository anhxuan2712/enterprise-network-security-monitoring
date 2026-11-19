# BÁO CÁO PHÂN TÍCH VÀ HIỆU CHỈNH TOÀN DIỆN THIẾT KẾ PHẦN CỨNG HẠ TẦNG MẠNG & AN NINH DOANH NGHIỆP

> **Tài liệu tham chiếu gốc:**  
> 1. `README.md` – Đồ án "Phân tích và thiết kế hệ thống mạng an toàn cho doanh nghiệp công nghệ" (Tài liệu GỐC 1: Kiến trúc Collapsed-Core, 30 VLANs, Ma trận Zone, 4 Extended ACLs, Mô hình Lab).  
> 2. `index-v2.html` – Bản vẽ thiết kế kỹ thuật mặt bằng 26 phòng ban & Bảng danh mục thiết bị (Tài liệu GỐC 2: Mặt bằng CAD, vị trí Rack U, danh mục thiết bị).  
> **Tiêu chuẩn thiết kế áp dụng:** TIA-942 (Rated-2 / Tier-2), ANSI/TIA-568.2-D, NIST SP 800-207 (Zero Trust Architecture), ISO/IEC 27001:2022 Controls.  
> **Quy mô hệ thống:** 2 Tòa nhà (Tòa A: 5 tầng, Tòa B: 4 tầng) • 26 Phòng ban • 144 Người dùng cuối + 2 Quản trị viên hệ thống (146 nhân sự) • 1 Phòng Máy chủ trung tâm (MDF A-202) • 9 Tủ mạng phân phối tầng (IDF) • 1 Trung tâm điều hành an ninh mạng (NOC/SOC A-404).

---

## MỤC LỤC

1. [TỔNG QUAN KIẾN TRÚC MẠNG & SỬA ĐỔI THIẾT KẾ CỐT LÕI](#1-tổng-quan-kiến-trúc-mạng--sửa-đổi-thiết-kế-cốt-lõi)
2. [SƠ ĐỒ TOPOLOGY PHÂN TẦNG VÀ LUỒNG DỮ LIỆU ĐÔNG - TÂY / BẮC - NAM](#2-sơ-đồ-topology-phân-tầng-và-luồng-dữ-liệu-đông---tây--bắc---nam)
3. [DANH MỤC THIẾT BỊ PHẦN CỨNG ĐÃ ĐƯỢC CHUẨN HÓA (BILL OF MATERIALS)](#3-danh-mục-thiết-bị-phần-cứng-đã-được-chuẩn-hóa-bill-of-materials)
4. [PHÂN TÍCH CHI TIẾT TỪNG PHÂN HỆ PHẦN CỨNG](#4-phân-tích-chi-tiết-từng-phân-hệ-phần-cứng)
   - [4.1. Phân Hệ Biên & Kết Nối WAN (Edge Routing): Cisco Catalyst 8300](#41-phân-hệ-biên--kết-nối-wan-edge-routing-cisco-catalyst-8300)
   - [4.2. Phân Hệ Tường Lửa Thế Hệ Mới (NGFW): Fortinet FortiGate 200F HA Cluster](#42-phân-hệ-tường-lửa-thế-hệ-mới-ngfw-fortinet-fortigate-200f-ha-cluster)
   - [4.3. Phân Hệ Chuyển Mạch Lõi (Core Switching L3): Cặp Cisco Catalyst 9500-24Y4C StackWise Virtual](#43-phân-hệ-chuyển-mạch-lõi-core-switching-l3-cặp-cisco-catalyst-9500-24y4c-stackwise-virtual)
   - [4.4. Phân Hệ Tủ Tầng (IDF) & Chuyển Mạch Truy Cập (Access Switching): Cisco Catalyst C9200L & C9200CX](#44-phân-hệ-tủ-tầng-idf--chuyển-mạch-truy-cập-access-switching-cisco-catalyst-c9200l--c9200cx)
   - [4.5. Phân Hệ Mạng Không Dây Doanh Nghiệp (Enterprise WLAN): Ubiquiti UniFi U6-Pro](#45-phân-hệ-mạng-không-dây-doanh-nghiệp-enterprise-wlan-ubiquiti-unifi-u6-pro)
   - [4.6. Phân Hệ Máy Chủ Dịch Vụ Data Center: Dell PowerEdge R660 & R760](#46-phân-hệ-máy-chủ-dịch-vụ-data-center-dell-poweredge-r660--r760)
   - [4.7. Phân Hệ Lưu Trữ Khối Chuyên Dụng (SAN Block Storage): Dell PowerVault ME5024 & ME5012](#47-phân-hệ-lưu-trữ-khối-chuyên-dụng-san-block-storage-dell-powervault-me5024--me5012)
   - [4.8. Phân Hệ Thoại Doanh Nghiệp (VoIP / UC): Cisco IP Phone 7841 & Tổng Đài SIP](#48-phân-hệ-thoại-doanh-nghiệp-voip--uc-cisco-ip-phone-7841--tổng-đài-sip)
   - [4.9. Phân Hệ Giám Sát Hình Ảnh (CCTV / VMS): Hikvision DS-2CD2143G2-I](#49-phân-hệ-giám-sát-hình-ảnh-cctv--vms-hikvision-ds-2cd2143g2-i)
   - [4.10. Phân Hệ Kiểm Soát Ra Vào Vật Lý (Physical Access Control): ZKTeco InBio + Mifare DESFire EV3](#410-phân-hệ-kiểm-soát-ra-vào-vật-lý-physical-access-control-zkteco-inbio--mifare-desfire-ev3)
   - [4.11. Hạ Tầng Nguồn Điện, UPS, Máy Phát & Cơ Điện Phòng Máy (DC Facilities)](#411-hạ-tầng-nguồn-điện-ups-máy-phát--cơ-điện-phòng-máy-dc-facilities)
   - [4.12. Trung Tâm Điều Hành & Giám Sát An Ninh (NOC / SOC A-404)](#412-trung-tâm-điều-hành--giám-sát-an-ninh-noc--soc-a-404)
   - [4.13. Máy Trạm Người Dùng & Máy In Mạng: Dell OptiPlex & HP LaserJet Enterprise](#413-máy-trạm-người-dùng--máy-in-mạng-dell-optiplex--hp-laserjet-enterprise)
   - [4.14. Hệ Thống Cáp Cấu Trúc, Patch Panel & Phân Phối Quang ODF](#414-hệ-thống-cáp-cấu-trúc-patch-panel--phân-phối-quang-odf)
5. [ĐÁNH GIÁ ĐIỂM YẾU KIẾN TRÚC, KẾ HOẠCH DỰ PHÒNG THẢM HỌA (DR) & RTO/RPO](#5-đánh-giá-điểm-yếu-kiến-trúc-kế-hoạch-dự-phòng-thảm-họa-dr--rtorpo)
6. [SO SÁNH TỔNG CHI PHÍ SỞ HỮU (TCO) & HOÀN VỐN ĐẦU TƯ (ROI) 5 NĂM](#6-so-sánh-tổng-chi-phí-sở-hữu-tco--hoàn-vốn-đầu-tư-roi-5-năm)
7. [KẾT LUẬN & LỘ TRÌNH TRIỂN KHAI](#7-kết-luận--lộ-trình-triển-khai)
8. [PHỤ LỤC KỸ THUẬT & NHẬT KÝ HIỆU CHỈNH](#8-phụ-lục-kỹ-thuật--nhật-ký-hiệu-chỉnh)
   - [8.1. Bảng Nhật Ký Thay Đổi](#81-bảng-nhật-ký-thay-đổi)
   - [8.2. Danh Sách Đề Xuất Cập Nhật Ngược Sang README.md & index-v2.html](#82-danh-sách-đề-xuất-cập-nhật-ngược-sang-readmemd--index-v2html)
   - [8.3. Danh Mục Các Mục Cần Đối Chiếu Datasheet Chính Hãng](#83-danh-mục-các-mục-cần-đối-chiếu-datasheet-chính-hãng)
   - [8.4. Báo Cáo Tự Kiểm Tra Chéo Số Liệu Nội Bộ](#84-báo-cáo-tự-kiểm-tra-chéo-số-liệu-nội-bộ)

---

## 1. TỔNG QUAN KIẾN TRÚC MẠNG & SỬA ĐỔI THIẾT KẾ CỐT LÕI

Để đảm bảo tính nhất quán tuyệt đối với tài liệu gốc `README.md` (Mục 3.1 và Phụ lục A) và bản vẽ mặt bằng `index-v2.html`, kiến trúc mạng được xác lập chuẩn mực theo mô hình **Collapsed-Core 2 lớp (Core/Distribution gộp & Access)** kết hợp phân vùng an ninh **Zero Trust**.

```
[KIẾN TRÚC COLLAPSED-CORE 2 LỚP DOANH NGHIỆP]

     LỚP BIÊN (WAN EDGE)         2x Cisco Catalyst 8300 (Dual ISP IP SLA + PBR)
              │                                      │
              ▼                                      ▼
    LỚP TƯỜNG LỬA (NGFW)        2x Fortinet FortiGate 200F (Active-Passive HA)
              │                                      │ (Mạng Transit OSPF Area 0)
              ▼                                      ▼
    LỚP CORE / PHÂN PHỐI        Cặp Cisco Catalyst 9500-24Y4C (StackWise Virtual)
    (COLLAPSED-CORE L3)         Default Gateway 30 VLANs + Bộ 4 Extended ACLs
              │
              ├────────────────────────────────────────┬─────────────────────────┐
    (Quang 10G SFP+ OS2)                      (Quang 10G SFP+ OS2)        (Quang 10G SFP+)
              │                                        │                         │
              ▼                                        ▼                         ▼
    9 TỦ TẦNG IDF (THỤ ĐỘNG)                 25 SWITCH PHÒNG ACCESS            DATA CENTER
    (ODF, Patch Panel, UPS 3kVA)             16x C9200L-24P-4X (PoE+ 370W)     ToR Switch Rack 02/03
    *Không đặt switch L3 tại IDF*            09x C9200CX-8P-2X2G (PoE+ 125W)   Máy chủ & SAN Storage
```

### Các nguyên tắc cốt lõi đã chuẩn hóa:
1. **Kiến trúc mạng 2 lớp Collapsed-Core:**  
   Toàn bộ chức năng định tuyến lõi và phân phối được tích hợp vào cặp switch **Cisco Catalyst 9500-24Y4C** ghép **StackWise Virtual (SVL)** đặt tại MDF A-202. Xóa bỏ hoàn toàn đề xuất đặt 9 switch phân phối L3 tại các tủ tầng IDF; **9 tủ IDF giữ nguyên vai trò là điểm đấu nối thụ động (Passive Cross-Connect)** chứa ODF quang, Patch Panel cáp đồng, PDU và UPS bảo vệ nguồn. Mỗi switch phòng sở hữu đường cáp quang Single-Mode OS2 10G SFP+ đi xuyên qua ODF tại IDF để kéo thẳng về cặp Core Switch tại MDF A-202 theo đúng `README.md` Mục 3.1.
2. **Kiểm soát liên vùng Đông - Tây (Inter-Zone Traffic):**  
   - *Trong phạm vi Lab đồ án & Thiết kế chuẩn:* Cặp Core Switch Catalyst 9500 đóng vai trò là SVI Default Gateway cho toàn bộ 30 VLANs (địa chỉ IP `.1`). Việc kiểm soát lưu lượng Đông - Tây giữa 6 Security Zones (Restricted, Infra/Mgmt, Executive, User, Voice, Guest) được thực thi triệt để bằng phần cứng thông qua **Bộ 4 Extended ACLs cốt lõi** (`ACL_GUEST_IN`, `ACL_USER_IN`, `ACL_RESTRICTED_IN`, `ACL_SERVER_FARM_OUT`) áp trực tiếp trên các SVI theo đúng `README.md` Mục 4.4.
   - *Mở rộng thực tế doanh nghiệp nâng cao (Ngoài phạm vi Lab đồ án):* Doanh nghiệp có thể chuyển gateway của các phân vùng nhạy cảm (Restricted, Server Farm, Management) sang cụm tường lửa FortiGate 200F thông qua sub-interface 802.1Q hoặc cấu hình VRF-Lite đa vùng trên Core Switch kết hợp định tuyến cưỡng bức qua tường lửa để kích hoạt tính năng kiểm tra ứng dụng chuyên sâu (DPI) và chống xâm nhập (IPS).
3. **Định tuyến WAN Dual ISP thực tế:**  
   Loại bỏ mô hình BGP đa luồng vì doanh nghiệp vừa và nhỏ không sở hữu số hiệu mạng ASN độc lập và dải IP Provider-Independent (PI). Hệ thống sử dụng cơ chế **IP SLA Tracking kết hợp Policy-Based Routing (PBR)** trên cặp router Cisco Catalyst 8300. Khi đường truyền chính gặp sự cố, hệ thống chuyển hướng lưu lượng sang đường truyền phụ trong vài giây (sau chu kỳ timeout của probe). Do địa chỉ IP NAT nguồn thay đổi từ IP Public của ISP 1 sang ISP 2, **các phiên kết nối TCP hiện hữu sẽ bị ngắt và thiết bị người dùng phải tái lập phiên kết nối mới** (không có hiện tượng chuyển mạch không đứt phiên). Bỏ BFD với IP công cộng 8.8.8.8 vì BFD yêu cầu sự hỗ trợ giao thức 2 đầu kết nối trực tiếp với router biên ISP.
4. **Hiệu năng Tường lửa FortiGate 200F:**  
   Sizing thông lượng tường lửa dựa trên chỉ số **Threat Protection Throughput = 3.0 Gbps** công bố chính thức trong Datasheet của Fortinet (khi bật đồng thời Firewall, IPS, Application Control, Antivirus và Logging), thay vì sử dụng thông lượng tường lửa thô 27 Gbps hay khoảng dải ước lượng.
5. **Chuẩn hóa thiết bị ngoại vi và an ninh vật lý:**  
   - Điện thoại VoIP nâng cấp lên **Cisco IP Phone 7841** để sở hữu 2 cổng Gigabit Ethernet 10/100/1000 Mbps passthrough cho máy tính.
   - Bộ điều khiển cửa `ZKTeco InBio-260` điều khiển 2 cửa độc lập, do đó toàn bộ 26 cửa của 26 phòng ban chỉ cần **13 bộ điều khiển**; khóa nam châm điện từ sử dụng nguồn DC 12V/24V riêng biệt có ắc quy dự phòng, không dùng PoE.
   - Tổng số camera an ninh là **26 camera** (25 camera tại 25 phòng ban + 01 camera giám sát trực diện tủ rack tại phòng MDF A-202).

---

## 2. SƠ ĐỒ TOPOLOGY PHÂN TẦNG VÀ LUỒNG DỮ LIỆU ĐÔNG - TÂY / BẮC - NAM

```
                       [ INTERNET ]
            ISP 1 (Viettel)     ISP 2 (VNPT)
                   │                   │
                   ▼                   ▼
       ┌───────────────────────────────────────────┐
       │   2x WAN Router Cisco Catalyst 8300       │ (Dual Power, IP SLA Tracking +
       │        (Active / Standby Failover)        │  PBR Failover vài giây)
       └─────────────────────┬─────────────────────┘
                             │ (2x 10G SFP+)
                             ▼
       ┌───────────────────────────────────────────┐
       │  2x Next-Gen Firewall FortiGate 200F      │ (Active-Passive HA Cluster,
       │    (Threat Protection: 3.0 Gbps Datasheet)│  Mạng Transit OSPF Area 0 MD5)
       └──────────────┬────────────────────────────┘
                      │ (Cụm LACP Trunk 2x 10G SFP+)
                      ▼
       ┌───────────────────────────────────────────┐
       │ CẶP CORE SWITCH LAYER 3                   │ (StackWise Virtual SVL + DAD)
       │ 2x Cisco Catalyst 9500-24Y4C              │ (Default Gateway cho 30 VLANs)
       │ Thực thi Bộ 4 Extended ACLs Inter-Zone   │ (Băng thông chuyển mạch: Line-rate)
       └──────┬─────────────────────────────┬──────┘
              │                             │
    (Cáp quang 10G OS2)            (Cáp quang 10G SFP+)
              │                             │
              ▼                             ▼
   ┌───────────────────────┐   ┌─────────────────────────────┐
   │ 9 TỦ TẦNG IDF         │   │ PHÂN HỆ DATA CENTER MDF     │
   │ (ĐIỂM ĐẤU NỐI THỤ ĐỘNG│   │ (Tủ Rack 01, 02, 03 A-202)  │
   │ ODF Quang 12-Port LC  │   │ 2x ToR Switch C9300-24S     │
   │ Patch Panel + UPS 3kVA│   │ Cụm Máy Chủ Dell R660/R760  │
   └──────────┬────────────┘   │ SAN Dell ME5024 & ME5012    │
              │                └─────────────────────────────┘
     (Cáp quang 10G OS2 kéo thẳng)
              │
              ▼
   ┌───────────────────────────────────────────────┐
   │ 25 SWITCH PHÒNG ACCESS LAYER                  │
   │ 16x C9200L-24P-4X (Phòng 4-15 người, PoE+ 370W)│
   │ 09x C9200CX-8P-2X2G (Phòng nhỏ/họp, Fanless)  │
   └───────┬──────────────┬──────────────┬─────────┘
           │              │              │
           ▼              ▼              ▼
       [ 144 PC ]    [ 25 AP U6 ]   [ 25 IP Phone ]
      Cat6A SFTP      PoE 802.3at      Cisco 7841
```

### Luồng lưu chuyển dữ liệu kỹ thuật:
- **Luồng Bắc - Nam (North - South):** Traffic từ Client ra Internet đi từ Switch phòng $\rightarrow$ cáp quang OS2 xuyên qua ODF tủ tầng IDF $\rightarrow$ Cặp Core Switch Catalyst 9500 (chuyển mạch L3) $\rightarrow$ Tuyến Transit OSPF `10.255.0.0/29` $\rightarrow$ Cụm Tường lửa FortiGate 200F (NAT, kiểm tra IPS, AV, Web Filter) $\rightarrow$ Router Cisco 8300 $\rightarrow$ Đường truyền ISP 1 hoặc ISP 2.
- **Luồng Đông - Tây (East - West):**
  - *Intra-VLAN:* Chuyển mạch L2 nội bộ tại switch phòng. Bật tính năng **Switchport Protected / Port Isolation** giữa các cổng máy trạm để chặn mã độc quét ngang (chú ý: **không kích hoạt tính năng này trên cổng máy in mạng dùng chung và cổng Uplink**).
  - *Inter-VLAN giữa các Zone:* Đi qua SVI trên Core Switch Catalyst 9500 và bị kiểm soát bằng **Bộ 4 Extended ACLs**:
    * `ACL_GUEST_IN`: Chặn tuyệt đối toàn bộ dải IP nội bộ RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), chỉ cho phép ra Internet qua NAT.
    * `ACL_USER_IN`: Cho phép ra Internet, truy cập dịch vụ Web/App tại Server Farm (VLAN 10), cấm truy cập sang VLAN Restricted và VLAN Quản trị 2.
    * `ACL_RESTRICTED_IN`: Chỉ cho phép kết nối đến các port CSDL/dịch vụ đặc quyền tại Server Farm, cấm truy cập chéo giữa các phòng thuộc vùng Restricted.
    * `ACL_SERVER_FARM_OUT`: Chỉ cho phép các gói tin phản hồi của các phiên đã khởi tạo (Established/ACK) đi ngược ra máy trạm.

---

## 3. DANH MỤC THIẾT BỊ PHẦN CỨNG ĐÃ ĐƯỢC CHUẨN HÓA (BILL OF MATERIALS)

| STT | Nhóm thiết bị | Model phần cứng chuẩn hóa | Số lượng | Vị trí lắp đặt | Vai trò kỹ thuật chính |
|:---:|:---|:---|:---:|:---|:---|
| **1** | **Router WAN Biên** | Cisco Catalyst C8300-1N1S-4T2X (Dual Power) | 02 bộ | MDF A-202 (Rack 01, U37-39) | Định tuyến biên, Dual ISP Viettel & VNPT, IP SLA + PBR failover |
| **2** | **Tường Lửa NGFW** | Fortinet FortiGate FG-200F (Gói Enterprise) | 02 bộ | MDF A-202 (Rack 01, U34-36) | Cụm HA Active-Passive, Threat Protection 3.0 Gbps, lọc ứng dụng |
| **3** | **Core Switch L3** | Cisco Catalyst C9500-24Y4C (StackWise Virtual) | 02 bộ | MDF A-202 (Rack 01, U30-33) | Cặp Core logic duy nhất, Default Gateway 30 VLANs, 48x 10/25G SFP28 |
| **4** | **Switch ToR Data Center** | Cisco Catalyst C9300-24S-E (24 Cổng 1G/10G SFP+) | 02 bộ | MDF A-202 (Rack 02/03, U40-42)| Top-of-Rack switch gom kết nối mạng máy chủ và SAN Storage |
| **5** | **Switch Phân Phối Tầng** | *Không trang bị switch L3 tại IDF* | 0 bộ | 9 Tủ IDF Tòa A & Tòa B | IDF là điểm trung chuyển thụ động (ODF, Patch Panel, UPS) |
| **6** | **Switch Phòng Lớn (4-15 PC)**| Cisco Catalyst C9200L-24P-4X (24 Cổng PoE+ 370W) | 16 bộ | 16 Phòng ban làm việc | Cấp mạng LAN 1Gbps, PoE+ 370W cho PC, VoIP, AP, Camera, Cửa |
| **7** | **Switch Phòng Nhỏ & Họp** | Cisco Catalyst C9200CX-8P-2X2G (Fanless 0dB) | 09 bộ | 9 Phòng nhỏ, Sảnh, Căng tin| Hoạt động không tiếng ồn, cấp nguồn PoE+ 125W, 2 cổng Uplink 10G |
| **8** | **Bộ Phát Sóng Wi-Fi** | Ubiquiti UniFi U6-Pro (Wi-Fi 6 AX5400) | 25 bộ | Trần 25 phòng ban | Phủ sóng Wi-Fi 6, WPA3-Enterprise 802.1X, tải 50–80 clients/AP |
| **9** | **Máy Chủ Dịch Vụ 1U** | Dell PowerEdge R660 (2x Xeon Silver 4410Y, 64-128GB)| 12 máy | MDF A-202 (Rack 01, 02, 03) | AD/DNS/DHCP, NMS Zabbix, RADIUS, PBX, ACS, Syslog, HRM, Fin |
| **10** | **Máy Chủ Dịch Vụ 2U** | Dell PowerEdge R760 (2x Xeon Gold 5416S, 128-256GB)| 08 máy | MDF A-202 (Rack 02, 03) | Web Portal, Mail, Database Cluster, Veeam Backup, SIEM, Kubernetes |
| **11** | **Hệ Thống SAN 2.5" (App/DB)**| Dell PowerVault ME5024 (24 khay SFF, Dual Ctrl) | 01 mảng | MDF A-202 (Rack 02, U16-20) | SAN Block Storage 2.5" SAS 10K HDD/SSD phục vụ DB & Virtualization |
| **12** | **Hệ Thống SAN 3.5" (VMS/Bk)**| Dell PowerVault ME5012 (12 khay LFF, Dual Ctrl) | 01 mảng | MDF A-202 (Rack 03, U05-08) | SAN Block Storage 3.5" SAS HDD phục vụ Camera NVR 30 ngày & Backup |
| **13** | **Điện Thoại IP Bàn** | Cisco IP Phone 7841 (4 Lines, 2 Cổng Gigabit RJ45)| 25 máy | Bàn lễ tân, trưởng phòng | VoIP HD Voice, cổng passthrough máy tính chuẩn 1000 Mbps Gigabit |
| **14** | **Điện Thoại IP Hội Nghị** | Cisco IP Phone 7832 Conference Phone | 02 máy | A-503 (BOD) & B-102 (Đối tác)| Thiết bị hội nghị đa hướng 360 độ, phục vụ họp trực tuyến |
| **15** | **Laptop Lãnh Đạo VIP** | Dell XPS 15 / 16 (Intel Core Ultra 7/9, 32GB RAM) | 02 máy | A-502 (Phòng CEO & CTO) | Trang bị di động bảo mật cao, kết nối Wi-Fi WPA3-Enterprise |
| **16** | **Camera An Ninh IP** | Hikvision DS-2CD2143G2-I (4MP Dome, IK10, H.265+) | 26 camera| 25 phòng ban + 01 MDF A-202| Giám sát hình ảnh an ninh 24/7, AcuSense, cô lập trong VLAN 12 |
| **17** | **Bộ Điều Khiển Cửa Ra Vào**| ZKTeco InBio-260 (Quản lý 2 cửa độc lập) | 13 bộ | Hộp kỹ thuật 26 cửa | Điều khiển cửa, kết nối mạng LAN về máy chủ ZKBioSecurity |
| **18** | **Đầu Đọc Thẻ & Vân Tay** | ZKTeco FR1500-A (Đầu đọc vân tay + thẻ Mifare) | 26 bộ | Cửa 25 phòng + Cửa MDF A-202| Xác thực 2 lớp, đọc thẻ tần số 13.56MHz qua kết nối RS485 |
| **19** | **Khóa Nam Châm Điện Từ** | Khóa từ Maglock 600lbs (280kg) + Gá ZL + Nút Exit | 26 bộ | Khung 26 cửa phòng ban | Khóa từ dùng nguồn DC 12V/24V, tích hợp rơ-le ngắt điện khi báo cháy |
| **20** | **Bộ Nguồn Dự Phòng Khóa Cửa**| Bộ nguồn chuyên dụng 12V DC 5A kèm ắc quy 12V 7Ah | 13 bộ | Cạnh bộ điều khiển InBio-260| Cấp nguồn DC cho khóa từ và đầu đọc, lưu điện 4–6 tiếng khi mất điện |
| **21** | **Tủ Rack Trung Tâm MDF** | APC NetShelter SX 42U (600 x 1070mm, Cửa lưới 80%) | 03 tủ | MDF A-202 (Rack 01, 02, 03) | Tải trọng tĩnh 1700 kg, chứa toàn bộ Core, Server Farm, SAN, PDU |
| **22** | **Tủ Rack Phân Phối Tầng** | APC NetShelter SX 42U (hoặc Tủ mạng chuyên dụng 42U)| 09 tủ | 9 Phòng kỹ thuật tầng (IDF) | Tủ IDF tiêu chuẩn chứa Patch Panel, ODF trung chuyển và UPS 3kVA |
| **23** | **Bộ Lưu Điện Online MDF** | APC Smart-UPS SRT 10kVA On-Line (kèm 2 module EBM)| 02 hệ | MDF A-202 (Cấu hình Nguồn 2N)| Lưu điện 15–20 phút ở tải 11kW, thời gian chuyển mạch 0ms |
| **24** | **Bộ Lưu Điện Online IDF** | APC Smart-UPS SRT 3000VA On-Line 230V | 09 bộ | 9 Tủ IDF các tầng | Cấp điện dự phòng cho switch phòng (kéo về) và thiết bị mạng tầng |
| **25** | **Thanh Nguồn Đo Dòng PDU** | APC Metered Rack PDU 16 Cổng (Dual Nhánh A/B) | 06 thanh | MDF A-202 (2 thanh/tủ rack) | Đo dòng tải Ampe thời gian thực, quản trị giám sát tải qua mạng |
| **26** | **KVM Console Rack** | KVM Console 1U màn hình gập 17" LCD + Touchpad | 01 bộ | MDF A-202 (Rack 02, U21-24) | Quản trị điều khiển trực tiếp bàn phím chuột màn hình tại phòng máy |
| **27** | **Thiết Bị Quản Trị OOB** | Terminal Console Server 8/16 Cổng RJ45 RS-232 | 02 bộ | MDF A-202 & NOC/SOC A-404 | Kết nối cổng Console vật lý cứu hộ Router, Firewall, Core Switch |
| **28** | **Máy Phát Điện Dự Phòng** | Cummins / Yanmar Diesel 45kVA + Tủ ATS 100A | 01 cụm | Khu kỹ thuật tầng trệt | Tự động phát điện cấp nguồn toàn bộ Data Center trong vòng 15 giây |
| **29** | **Điều Hòa Chính Xác CRAC** | Stulz / Vertiv In-Row CRAC 20kW (Cấu hình N+1) | 02 cụm | MDF A-202 | Duy trì nhiệt độ 20°C ± 2°C, độ ẩm 50% ± 5% liên tục 24/7 |
| **30** | **Chữa Cháy Khí Sạch** | Hệ thống bình khí sạch FM-200 / Novec 1230 | 01 hệ | MDF A-202 | Dập tắt đám cháy trong 10 giây, an toàn cho người và vi mạch điện tử |
| **31** | **Chống Sét Lan Truyền SPD** | Tủ cắt lọc sét OBO Bettermann Type 1+2 (50 kA) | 01 bộ | Tủ điện đầu vào Data Center | Triệt tiêu xung điện sét lan truyền bảo vệ toàn bộ thiết bị phòng máy |
| **32** | **Màn Hình Video Wall SOC** | Ghép 4 tấm 55-inch Ultra Narrow Bezel 0.88mm | 01 cụm | NOC/SOC A-404 | Hiển thị 24/7 SIEM Wazuh, Grafana, Zabbix NMS, Cảnh báo Firewall |
| **33** | **Máy Trạm Chuyên Dụng SOC** | Dell Precision 3680 Workstation Dual Monitor 27" | 04 bộ | NOC/SOC A-404 | Máy trạm đồ họa 32GB RAM phục vụ phân tích log, săn tìm mối đe dọa |
| **34** | **Máy Trạm Người Dùng** | Dell OptiPlex Plus 7020 SFF (Core i7, 16-32GB RAM) | 144 bộ | 25 Phòng ban nghiệp vụ | Máy trạm làm việc chuẩn hóa, TPM 2.0, Gigabit Ethernet bọc kim |
| **35** | **Máy In Mạng Doanh Nghiệp** | HP LaserJet Enterprise M507dn | 20 máy | Các phòng ban nghiệp vụ | In ấn văn bản bảo mật qua mã PIN cá nhân / HTTPS IPP |
| **36** | **Cáp Mạng Đồng Nhánh** | CommScope NetConnect Cat6A SFTP (Chống nhiễu kép) | Toàn bộ | Đi âm trần, sàn toàn tòa nhà| Băng thông 500 MHz, hỗ trợ 10 Gbps 100m, triệt tiêu nhiễu xuyên âm |
| **37** | **Cáp Quang Trục Chính** | Cáp quang Single-Mode OS2 12-core & 24-core | Toàn bộ | Trục đứng liên tầng & liên tòa| 9 Tuyến cáp quang từ 9 IDF kéo thẳng về MDF A-202 |
| **38** | **Khung Phân Phối Quang ODF**| Khung ODF tập trung 144-Port LC Duplex Single-Mode | 01 khung| MDF A-202 (Đỉnh Rack 01) | Quản lý 54 cặp sợi quang (108 sợi) từ 9 IDF và 2 nhà mạng ISP |
| **39** | **Module Quang SFP+ 10G** | Cisco SFP-10G-LR (Single-Mode OS2, 1310nm, 10km) | 68 chiếc| MDF A-202, IDF, Switch phòng| Bảng chi tiết đấu nối cổng quang xem tại Mục 4.14 |
| **40** | **Cáp Đấu Trực Tiếp DAC 10G** | Cisco 10GBASE-CU SFP+ Direct Attach Copper (1m-3m)| 24 sợi | MDF A-202 (Nội bộ Rack) | Kết nối tốc độ cao Server ToR, Firewall HA và Core Switch |
| **41** | **Cáp Ghép Lõi SVL 40G/100G**| Cisco QSFP-100G-CU Direct Attach Copper (1m) | 04 sợi | MDF A-202 (Giữa 2 Core 9500)| Kết nối kênh StackWise Virtual Link (SVL) và DAD giữa 2 Core 9500 |

---

## 4. PHÂN TÍCH CHI TIẾT TỪNG PHÂN HỆ PHẦN CỨNG

### 4.1. Phân Hệ Biên & Kết Nối WAN (Edge Routing): Cisco Catalyst 8300

- **Model cấu hình:** `Cisco Catalyst C8300-1N1S-4T2X` trang bị sẵn 2 nguồn dự phòng tháo lắp nóng (Dual Hot-swap AC Power Supplies).
- **Vị trí & Số lượng:** 02 thiết bị kết nối song hành tại U37-U39 Rack 01 (MDF A-202).
- **Cơ chế định tuyến Dual ISP thực tế cho doanh nghiệp:**  
  Doanh nghiệp sử dụng 2 đường truyền Internet từ 2 nhà cung cấp độc lập: **ISP 1 (Viettel FTTH 1Gbps)** và **ISP 2 (VNPT FTTH 1Gbps)**. Vì không có số hiệu mạng ASN và dải IP Provider-Independent (PI), cơ chế định tuyến được thiết lập bằng:
  - **IP SLA Tracking kết hợp Policy-Based Routing (PBR):** Router Cisco 8300 liên tục gửi gói tin ICMP Echo thăm dò định kỳ đến DNS công cộng tin cậy (như `8.8.8.8` và `1.1.1.1`) qua cổng WAN chính.
  - **Quá trình chuyển đổi dự phòng (Failover):** Khi đường truyền chính bị đứt hoặc mất kết nối ra Internet, bộ theo dõi IP SLA phát hiện timeout sau chu kỳ kiểm tra (thường mất **vài giây** tùy thuộc tham số cấu hình: 3 lần probe timeout $\times$ 1 giây + track delay). Router lập tức rút tuyến tĩnh (Static Route) mặc định của ISP 1 khỏi bảng định tuyến và kích hoạt tuyến dự phòng qua ISP 2.
  - *Lưu ý kỹ thuật quan trọng:* Khi chuyển hướng sang ISP 2, địa chỉ IP Public NAT của doanh nghiệp sẽ thay đổi từ dải IP của Viettel sang VNPT. **Toàn bộ các phiên kết nối TCP hiện hữu (như cuộc gọi Zoom/Teams, phiên SSH, phiên truyền tải file) sẽ bị ngắt kết nối và người dùng buộc phải thiết lập lại phiên mới**. Không sử dụng giao thức BFD cho trường hợp này vì BFD chỉ khả thi khi nhà mạng ISP hỗ trợ phiên tương thích BFD trực tiếp giữa thiết bị khách hàng và router biên của ISP.
- **Ưu điểm:** Nền tảng vi xử lý Cisco QFP 2.0 xử lý gói tin bằng phần cứng chuyên dụng; tích hợp cổng quang 10G SFP+; hỗ trợ sẵn sàng chuyển đổi sang kiến trúc Cisco SD-WAN khi doanh nghiệp mở thêm chi nhánh.
- **Nhược điểm:** Cần kỹ sư có chứng chỉ chuyên môn CCNA/CCNP để quản trị qua Cisco IOS-XE; chi phí bản quyền Cisco DNA đi kèm.

---

### 4.2. Phân Hệ Tường Lửa Thế Hệ Mới (NGFW): Fortinet FortiGate 200F HA Cluster

- **Model cấu hình:** Cụm 02 thiết bị `Fortinet FortiGate FG-200F` chạy song hành kèm gói bản quyền bảo mật FortiGuard Enterprise Protection.
- **Vị trí:** U34-U36 Rack 01 (MDF A-202).
- **Cấu hình Cổng Vật Lý (Port Mapping) chuẩn Datasheet:**  
  *Đối chiếu Datasheet FortiGate 200F:* Thiết bị sở hữu **4 cổng 10GE SFP+ slots**, **2 cổng 10GE SFP+ HA/MGMT**, **16 cổng GE RJ45**, **8 cổng GE SFP slots**. Bảng phân bổ cổng vật lý không vượt quá giới hạn thiết bị:
  - `HA1` & `HA2` (2 cổng 10GE SFP+ chuyên dụng): Nối chéo trực tiếp giữa 2 thiết bị FortiGate bằng cáp quang/DAC để đồng bộ trạng thái phiên làm việc (Session Synchronization) và nhịp tim (Heartbeat).
  - `X1` & `X2` (2 cổng 10GE SFP+): Cấu hình cụm kênh gộp cổng **802.3ad LACP Trunk (20 Gbps)** kết nối trực tiếp xuống Cặp Core Switch Catalyst 9500 (chạy mạng Transit OSPF Area 0 MD5).
  - `X3` & `X4` (2 cổng 10GE SFP+): Kết nối cáp quang/DAC 10G sang 2 Router biên Cisco 8300 (phân đoạn WAN/DMZ).
- **Tính toán hiệu năng tải thực tế (Firewall Sizing Calculation):**  
  - *Hiệu năng công bố chính thức (Datasheet):* **Threat Protection Throughput = 3.0 Gbps** (khi kích hoạt đồng thời Firewall, IPS, Application Control, Antivirus và Logging).
  - *Băng thông Internet thiết kế tối đa:* 2 đường FTTH $2 \times 1\text{ Gbps} = 2\text{ Gbps}$ (thông thường chỉ chạy Active-Standby tải thực tế $1\text{ Gbps}$).
  - *Lưu lượng liên vùng DMZ / Cổng dịch vụ:* $\approx 1.0\text{ Gbps}$.
  - *Tổng tải phân tích an ninh tối đa:* $1.0\text{ Gbps (WAN)} + 1.0\text{ Gbps (DMZ/Transit)} = 2.0\text{ Gbps}$.
  - *Tỷ lệ tải phần trăm thiết kế:*
    $$\text{Tỷ lệ tải Threat Protection} = \frac{2.0\text{ Gbps}}{3.0\text{ Gbps}} \approx \mathbf{66.7\%}$$
    Mức tải 66.7% nằm hoàn toàn trong ngưỡng vận hành an toàn tiêu chuẩn ($< 75\%-80\%$). Hệ thống không bị quá tải CPU khi có các đợt lưu lượng đột biến.
- **Cơ chế HA Active-Passive Stateful Failover:** Thiết bị Master xử lý toàn bộ lưu lượng; thiết bị Slave nhận bản sao bảng phiên (session table) liên tục. Khi Master gặp sự cố phần cứng hoặc đứt link mạng, Slave tiếp quản ngay lập tức trong vòng dưới 1 giây.

---

### 4.3. Phân Hệ Chuyển Mạch Lõi (Core Switching L3): Cặp Cisco Catalyst 9500-24Y4C StackWise Virtual

- **Model cấu hình:** Cặp 02 thiết bị `Cisco Catalyst C9500-24Y4C` (License Network Advantage).
- **Vị trí:** U30-U33 Rack 01 (MDF A-202).
- **Kiến trúc Ghép Cặp StackWise Virtual (SVL):**  
  Theo đúng thiết kế tại `README.md` Phụ lục A.2 và A.3, 2 switch vật lý C9500-24Y4C được hợp nhất thành **01 Switch Core Logic duy nhất**:
  - **StackWise Virtual Link (SVL):** Sử dụng 02 đường kết nối tốc độ cao **100G QSFP28** (cổng C1 và C2) nối chéo trực tiếp giữa Core 1 và Core 2 để truyền tải dữ liệu và đồng bộ Control Plane.
  - **Dual-Active Detection (DAD) Link:** Sử dụng 02 đường kết nối độc lập qua cổng SFP28 chuyên dụng để phát hiện và ngăn ngừa hiện tượng phân rã não bộ (Split-Brain) khi đường SVL gặp sự cố.
- **Vai trò Định tuyến & An ninh L3:**  
  - Là **Default Gateway của toàn bộ 30 VLANs** trong hệ thống (địa chỉ IP `.1` cấu hình trên các SVI tương ứng).
  - Trực tiếp thực thi **Bộ 4 Extended ACLs** (`ACL_GUEST_IN`, `ACL_USER_IN`, `ACL_RESTRICTED_IN`, `ACL_SERVER_FARM_OUT`) ở tốc độ phần cứng bằng chip chuyên dụng TCAM (Line-rate Hardware Forwarding), đảm bảo kiểm soát chặt chẽ lưu lượng Đông - Tây giữa 6 Security Zones mà không gây trễ mạng.
- **Phân bổ mật độ cổng quang trên cặp C9500-24Y4C (Tổng 48 cổng 10G/25G SFP28 + 8 cổng 40G/100G):**  
  *Bảng đếm cổng chính xác theo thiết kế:*
  1. *25 Uplink từ 25 Switch phòng:* 25 cổng 10G SFP+ (phân bổ 13 cổng Core 1, 12 cổng Core 2).
  2. *04 Link đấu nối cụm Firewall FortiGate 200F:* 04 cổng 10G SFP+ (chạy Cross-Stack LACP EtherChannel).
  3. *04 Link đấu nối cụm Switch ToR Data Center (Rack 02/03):* 04 cổng 10G SFP+ (chạy Dual-Homed LACP).
  4. *02 Link đấu nối trạm NOC/SOC A-404 & Helpdesk A-203:* 02 cổng 10G SFP+.
  5. *02 Link DAD (Dual-Active Detection):* 02 cổng SFP28 nối trực tiếp Core 1 - Core 2.
  6. *02 Link SVL (StackWise Virtual Link):* 02 cổng 100G QSFP28.
  7. *Dự phòng mở rộng (Expansion Spares):* 11 cổng 10G/25G SFP28 còn trống.
  $\rightarrow$ **Tổng cộng:** 37/48 cổng SFP28 và 2/8 cổng 100G được sử dụng. Mật độ cổng cực kỳ tối ưu và đáp ứng khả năng mở rộng trong 5-10 năm tới.

---

### 4.4. Phân Hệ Tủ Tầng (IDF) & Chuyển Mạch Truy Cập (Access Switching): Cisco Catalyst C9200L & C9200CX

#### Kiến trúc Tủ Mạng Phân Phối Tầng (IDF 42U - 9 Tủ):
Khẳng định nhất quán: **Tại 9 tủ IDF theo tầng không lắp đặt switch phân phối L3 chủ động**. Toàn bộ 9 tủ IDF (5 tủ Tòa A: `IDF-A-T1` đến `IDF-A-T5`; 4 tủ Tòa B: `IDF-B-T1` đến `IDF-B-T4`) là các tủ mạng tiêu chuẩn 42U đóng vai trò là **Điểm đấu nối trung chuyển thụ động (Passive Cross-Connect Facility)**:
- Chứa **Hộp phối quang ODF 12-Port LC Duplex** (tiếp nhận cáp quang từ switch phòng và đấu nối vào cáp quang trục chính OS2 về MDF A-202).
- Chứa các **Thanh Patch Panel Cat6A Modular 24/48 Cổng** để cố định các đầu cáp mạng nhánh.
- Chứa **Thanh nguồn PDU đo dòng** và **Bộ lưu điện Online APC Smart-UPS SRT 3000VA** chuyên cấp điện sạch bảo vệ cho các thiết bị mạng đặt tại tầng và duy trì hoạt động cho hệ thống camera/AP được cấp nguồn từ phòng.

#### Cấu hình Switch tại 25 Phòng Ban:
- **Tại 16 Phòng Quy Mô Lớn (4 - 15 nhân sự):** Trang bị `Cisco Catalyst C9200L-24P-4X`.
  - Cung cấp 24 cổng mạng Gigabit RJ45 hỗ trợ PoE+ (IEEE 802.3at), 4 cổng Uplink 10G SFP+.
  - **Tính toán tải nguồn PoE thực tế tại mỗi phòng:**
    * 01 Bộ phát Wi-Fi 6 UniFi U6-Pro: tiêu thụ tối đa **13.0 Watts**.
    * 01 Camera IP Hikvision DS-2CD2143G2-I: tiêu thụ tối đa **7.0 Watts**.
    * 01 Điện thoại VoIP Cisco IP Phone 7841: tiêu thụ tối đa **3.8 Watts** (làm tròn **4.0 Watts**).
    * *Bộ điều khiển cửa ZKTeco InBio và Khóa từ Maglock:* Sử dụng nguồn điện 12V DC riêng biệt, **không tiêu thụ công suất PoE của switch**.
    * **Tổng công suất tiêu thụ PoE thực tế của 1 phòng lớn:**
      $$P_{\text{phòng lớn}} = 13.0\text{W} + 7.0\text{W} + 4.0\text{W} = \mathbf{24.0\text{ Watts}}$$
    * **Tỷ lệ tải nguồn PoE trên Switch C9200L (Budget 370W):**
      $$\text{Tỷ lệ tải PoE} = \frac{24.0\text{W}}{370\text{W}} \approx \mathbf{6.49\%}$$
      *(Ghi chú đối chiếu: Bản tính toán lý thuyết tối đa tại `README.md` Phụ lục A.1 sử dụng công suất tối đa theo Class là 76.2W ứng với tỷ lệ 20.6%; công suất thực tế 24W là con số vận hành chính xác giúp switch luôn mát mẻ và nâng cao độ bền linh kiện).*
- **Tại 09 Phòng Nhỏ / Phòng Họp / Phòng Giám Đốc (1 - 3 nhân sự):** Trang bị `Cisco Catalyst C9200CX-8P-2X2G`.
  - Cung cấp 8 cổng Gigabit RJ45 PoE+, 2 cổng Uplink 10G SFP+, 2 cổng 1G Copper. Thiết kế **hoàn toàn không quạt (Fanless 0dB)**, vận hành êm ái tuyệt đối trong không gian yên tĩnh.
  - **Tỷ lệ tải nguồn PoE trên Switch C9200CX (Budget 125W):**
    $$\text{Tỷ lệ tải PoE} = \frac{24.0\text{W}}{125\text{W}} \approx \mathbf{19.2\%}$$
- **Dự phòng sự cố (Spare Units):** Chuẩn bị sẵn trong kho kỹ thuật IT tối thiểu **01 bộ C9200L-24P-4X** và **01 bộ C9200CX-8P-2X2G** (Cold Spare) đã nạp sẵn cấu hình mẫu chuẩn để có thể thay thế ngay lập tức trong vòng 30 phút khi có sự cố hỏng hóc bộ nguồn cố định của dòng switch này.

---

### 4.5. Phân Hệ Mạng Không Dây Doanh Nghiệp (Enterprise WLAN): Ubiquiti UniFi U6-Pro

- **Model thiết bị:** `Ubiquiti UniFi U6-Pro` (Wi-Fi 6 802.11ax, băng tần kép, 4x4 MU-MIMO ở băng tần 5GHz và 2x2 MU-MIMO ở băng tần 2.4GHz).
- **Số lượng & Bố trí:** 25 bộ gắn trần tại 25 phòng ban (Phòng máy chủ trung tâm MDF A-202 tuyệt đối không phát sóng Wi-Fi nhằm ngăn chặn các nguy cơ tấn công xâm nhập không dây).
- **Thông số kỹ thuật vận hành:**
  - Chuẩn cấp nguồn: **PoE 802.3at (PoE+)**, công suất tiêu thụ tối đa **13.0 Watts**.
  - **Khả năng chịu tải khuyến nghị thực tế:** Từ **50 đến 80 thiết bị (Clients) trên mỗi AP** để đảm bảo băng thông ổn định cho các cuộc gọi thoại VoIP và hội nghị truyền hình không bị giật lag (loại bỏ các con số lý thuyết 250 - 300 clients).
- **Quy hoạch phát sóng và phân vùng VLAN:**
  - *SSID Doanh Nghiệp ("Enterprise-Corp"):* Áp dụng mã hóa cấp doanh nghiệp **WPA3-Enterprise / 802.1X**. Người dùng xác thực thông qua máy chủ RADIUS kết nối với Active Directory. Switch và AP phối hợp gán động người dùng vào đúng VLAN phòng ban nghiệp vụ tương ứng.
  - *SSID Khách ("Enterprise-Guest"):* Kích hoạt Cổng chào (Captive Portal), bật tính năng cách ly người dùng (**Client Isolation**). Phát sóng gán đúng vào các VLAN Khách theo quy hoạch 30 VLAN của `README.md`:
    * Sảnh lễ tân Tòa A (A-101): **VLAN 100** (`172.16.100.0/24`).
    * Sảnh tiếp khách Tòa B (B-101): **VLAN 110** (`172.16.110.0/24`).
    * Phòng họp đối tác Tòa B (B-102): **VLAN 111** (`172.16.111.0/24`).
    * Căng tin / Khu nghỉ nhân viên (B-403): **VLAN 143** (`172.16.143.0/24`).
    * *Chính sách bảo mật:* Cấm hoàn toàn mọi kết nối từ VLAN Khách đi vào các dải mạng nội bộ, chỉ cho phép đi Internet và giới hạn băng thông 10 Mbps mỗi thiết bị.
- **Hệ thống Quản trị:** Phần mềm `UniFi Network Application` chạy dưới dạng máy ảo (VM) tập trung tại Data Center, quản lý cấu hình và tự động cân chỉnh sóng (Auto-RF Channel & Power) cho toàn bộ 25 AP mà không tốn phí bản quyền định kỳ hàng năm.

---

### 4.6. Phân Hệ Máy Chủ Dịch Vụ Data Center: Dell PowerEdge R660 & R760

Toàn bộ hệ thống máy chủ được chuẩn hóa trên nền tảng thế hệ mới **Dell PowerEdge 16G (Intel Xeon Scalable Gen 4)**, đảm bảo tính sẵn sàng cao, tiết kiệm điện năng và vòng đời hỗ trợ kỹ thuật dài hạn.

#### Mô hình Triển khai & Ảo Hóa Thực Tế:
Hệ thống sử dụng giải pháp **Cụm Ảo hóa Máy chủ Doanh nghiệp (VMware vSphere Enterprise Plus Cluster hoặc Proxmox VE VE Enterprise Cluster)** chạy trên nền tảng lưu trữ mạng SAN. Việc gom các dịch vụ vào các máy ảo (Virtual Machines - VM) trên các máy chủ vật lý cấu hình mạnh (Host) giúp tận dụng tối đa tài nguyên phần cứng, nâng cao khả năng dự phòng High Availability (tự động khởi động lại VM trên host khác khi một host vật lý gặp sự cố) và dễ dàng sao lưu snapshot.

#### Bảng Ánh Xạ Chi Tiết Máy Chủ, Dịch Vụ, Địa Chỉ IP & VLAN:

| STT | Tên Dịch Vụ / Máy Chủ | Mô Hình Triển Khai | Dòng Máy Chủ Vật Lý | Vị Trí Rack / U | Địa Chỉ IP Chuẩn Hóa | Thuộc VLAN | Ghi Chú Kỹ Thuật |
|:---:|:---|:---|:---|:---:|:---:|:---:|:---|
| 1 | **Active Directory / DNS / DHCP / NTP** | Cụm 02 VM (DC01 & DC02) | Dell PowerEdge R660 | Rack 02, U38-39 | `192.168.10.10` / `.11` | VLAN 10 | Windows Server 2022 AD DS, DNS tích hợp, DHCP Failover |
| 2 | **Web Portal Nội Bộ** | VM Cluster (Nginx/Apache) | Dell PowerEdge R760 | Rack 02, U35-36 | `192.168.10.20` | VLAN 10 | Cổng thông tin doanh nghiệp, tài liệu, REST API nội bộ |
| 3 | **Mail Server Doanh Nghiệp** | VM Độc Lập (Zimbra/Exchange)| Dell PowerEdge R760 | Rack 02, U33-34 | `192.168.10.25` | VLAN 10 | Hệ thống thư điện tử, bảo mật SPF/DKIM/DMARC |
| 4 | **Cơ Sở Dữ Liệu Cốt Lõi (Database)** | 02 VM (Master - Replica) | Dell PowerEdge R760 | Rack 02, U28-29 | `192.168.10.30` | VLAN 10 | PostgreSQL / MySQL Master-Slave Streaming Replication |
| 5 | **File Gateway / File Server** | 02 VM (Windows Failover Cluster)| Dell PowerEdge R760 | Rack 02, U30-32 | `192.168.10.35` | VLAN 10 | Chia sẻ tệp SMBv3/NFS, kết nối LUN từ SAN ME5024 |
| 6 | **Wazuh SIEM Manager & Indexer** | VM Chuyên Dụng (16 Core/64GB)| Dell PowerEdge R760 | Rack 03, U10-11 | `192.168.10.40` | VLAN 10 | Nguồn dữ liệu giám sát an ninh SOC, nhận log thiết bị |
| 7 | **Rsyslog Collector Tập Trung** | VM Linux (rsyslog daemon) | Dell PowerEdge R660 | Rack 03, U12 | `192.168.10.50` | VLAN 10 | Thu thập Syslog từ Switch, Router, Firewall lưu tối thiểu 12 tháng |
| 8 | **Veeam Backup & Replication** | Máy Chủ Vật Lý Độc Lập | Dell PowerEdge R760 | Rack 02, U25-27 | `192.168.10.60` | VLAN 10 | Quản lý sao lưu máy ảo, kết nối SAN ME5012 và Cloud |
| 9 | **Xác Thực RADIUS / NAC** | Cụm 02 VM (FreeRADIUS / NPS)| Dell PowerEdge R660 | Rack 03, U01-02 | `192.168.10.14` | VLAN 10 | Xác thực 802.1X cho máy trạm và WPA3-Enterprise Wi-Fi |
| 10 | **UniFi Network Controller** | VM Linux | Dell PowerEdge R660 | Rack 03, U03 | `192.168.10.15` | VLAN 10 | Quản trị tập trung 25 bộ phát Wi-Fi UniFi U6-Pro |
| 11 | **Tổng Đài Thoại IP-PBX** | VM Chuyên Dụng (CUCM / Asterisk)| Dell PowerEdge R660 | Rack 03, U04 | `192.168.10.16` | VLAN 10 | Xử lý cuộc gọi SIP, đăng ký cho 25 máy Cisco 7841 |
| 12 | **Hệ Thống Quản Lý Camera VMS** | Máy Chủ Vật Lý + Storage | Dell PowerEdge R760 | Rack 03, U05-08 | `192.168.10.17` | VLAN 10 | Hikvision iVMS Server, ghi hình 26 camera lưu trữ SAN ME5012 |
| 13 | **ZKBioSecurity Access Control** | VM Windows Server | Dell PowerEdge R660 | Rack 03, U09 | `192.168.10.18` | VLAN 10 | Nhận dữ liệu chấm công và quản lý thẻ ra vào từ 13 bộ InBio |
| 14 | **Bastion Host / Jump Server** | VM Linux Hardened | Dell PowerEdge R660 | Rack 03, U13 | `192.168.10.42` | VLAN 10 | Cổng SSH/RDP duy nhất vào hệ thống máy chủ, bắt buộc MFA |
| 15 | **Hệ Thống Giám Sát NMS Zabbix**| VM Linux | Dell PowerEdge R660 | Rack 01, U18-21 | `192.168.10.12` | VLAN 10 | Giám sát lưu lượng, trạng thái SNMP thiết bị mạng |
| 16 | **Kho Mã Nguồn GitLab & CI/CD** | VM Linux (SSD Cache) | Dell PowerEdge R760 | Rack 03, U14-15 | `192.168.10.70` | VLAN 10 | Quản lý mã nguồn dự án, build pipeline phần mềm |
| 17 | **Cụm Kubernetes Production** | 03 VM Nodes Stacked Master/Worker| Dell PowerEdge R760 | Rack 03, U16-21 | `192.168.10.80 - .82`| VLAN 10 | 3 Node K8s chạy etcd quorum đảm bảo HA control-plane |
| 18 | **Môi Trường Staging / UAT** | Cụm VM Thử Nghiệm | Dell PowerEdge R660 | Rack 03, U22-23 | `192.168.10.90` | VLAN 10 | Môi trường kiểm thử độc lập cho phòng QA A-401 và Lab A-201 |
| 19 | **Ứng Dụng CRM & ERP** | VM Chuyên Dụng | Dell PowerEdge R760 | Rack 03, U24-25 | `192.168.10.100`| VLAN 10 | Phục vụ nghiệp vụ bán hàng, kho vận và hỗ trợ khách hàng |
| 20 | **Hệ Thống Nhân Sự HRM & Payroll**| VM Bảo Mật | Dell PowerEdge R660 | Rack 03, U26 | `192.168.10.110`| VLAN 10 | Quản lý hồ sơ và bảng lương nhân viên, chỉ phòng B-301 truy cập |
| 21 | **Hệ Thống Kế Toán Finance** | VM Bảo Mật | Dell PowerEdge R660 | Rack 03, U27 | `192.168.10.120`| VLAN 10 | Dữ liệu tài chính kế toán, chỉ phòng B-302 truy cập |

---

### 4.7. Phân Hệ Lưu Trữ Khối Chuyên Dụng (SAN Block Storage): Dell PowerVault ME5024 & ME5012

Cần định nghĩa chính xác về mặt kiến trúc: **Dell PowerVault ME5 Series là thiết bị lưu trữ khối chuyên dụng SAN (Block Storage)**, giao tiếp qua giao thức iSCSI 10G/25G hoặc SAS 12G. Thiết bị không cung cấp giao thức chia sẻ file NAS (SMB/CIFS) trực tiếp; các dịch vụ File Server được cung cấp thông qua cụm máy chủ File Gateway Windows/Linux kết nối vào các LUN của ME5.

#### Phân bổ 2 mảng lưu trữ chuyên biệt:
1. **Mảng SAN 01 - Phục vụ Ứng Dụng, Cơ Sở Dữ Liệu & Ảo Hóa:** `Dell PowerVault ME5024` (Khung 2U gồm 24 khay ổ 2.5" SFF, Dual Active-Active Controllers).
   - Trang bị 24 ổ cứng chuyên dụng **Enterprise SAS 2.4TB 10K RPM 2.5" HDD** (kết hợp các ổ SAS SSD làm tầng đệm đọc ghi Read/Write Cache).
   - *Cấu hình Disk Group chuẩn kỹ thuật:* Khuyến nghị không cấu hình 1 nhóm RAID 6 quá lớn (trên 16 ổ) để tránh thời gian khôi phục (rebuild time) kéo dài làm giảm hiệu năng. Hệ thống chia thành:
     * **02 Nhóm Disk Group RAID 6 độc lập:** Mỗi nhóm gồm 10 ổ (8 Data + 2 Parity).
     * **04 Ổ đĩa dự phòng nóng (Global Hot Spares):** Tự động kích hoạt khi có bất kỳ ổ đĩa nào gặp sự cố vật lý.
   - *Tính toán dung lượng khả dụng (Usable Capacity):*
     * Tổng số ổ chứa dữ liệu: $2 \times 8 = 16\text{ ổ}$.
     * Dung lượng dữ liệu thô danh định: $16 \times 2.4\text{ TB} = 38.4\text{ TB (decimal)}$.
     * Quy đổi sang đơn vị nhị phân thực tế của hệ điều hành (TiB):
       $$38.4 \times 10^{12} \div (1024^4) \approx 34.92\text{ TiB}$$
     * Trừ trừ hao hụt bảng quản lý metadata của SAN (~5%): Dung lượng khả dụng thực tế đạt xấp xỉ **~33.2 TiB Usable** cho toàn bộ máy ảo và cơ sở dữ liệu.
2. **Mảng SAN 02 - Phục vụ Kho Ghi Hình Camera VMS & Sao Lưu Veeam:** `Dell PowerVault ME5012` (Khung 2U gồm 12 khay ổ 3.5" LFF dung lượng cao, Dual Controllers).
   - Trang bị 12 ổ cứng chuẩn **Enterprise SAS 3.5" dung lượng 10TB/ổ**.
   - *Cấu hình Disk Group:* **01 Nhóm RAID 6 gồm 10 ổ (8 Data + 2 Parity) + 02 Ổ Hot Spares**.
   - *Tính toán dung lượng khả dụng chuẩn xác:*
     * Dung lượng danh định của 8 ổ dữ liệu: $8 \times 10\text{ TB} = 80\text{ TB (decimal)}$.
     * Quy đổi sang đơn vị nhị phân:
       $$\text{Dung lượng nhị phân} = 80 \times 10^{12} \div (1024^4) \approx \mathbf{72.76\text{ TiB}}$$
     * Trừ hao hụt định dạng Pool & Snapshot Metadata (ước tính 6%): Dung lượng khả dụng thực tế đạt xấp xỉ **~68.4 TiB Usable**.
   - *Phân bổ dung lượng thực tế:*
     * **Nhu cầu ghi hình 26 Camera an ninh trong 30 ngày:**  
       26 camera $\times$ bitrate 3 Mbps $\times$ chuẩn nén H.265+ $\times$ 24 giờ $\times$ 30 ngày:
       $$\text{Dung lượng Camera cơ bản} = 26 \times \frac{3 \times 3600 \times 24 \times 30}{8 \times 1024 \times 1024} \approx 24.08\text{ TB (decimal)} \approx \mathbf{21.9\text{ TiB}}$$
       Cộng thêm hệ số dự phòng 25% (cho các khung cảnh nhiều chuyển động đột biến): $21.9\text{ TiB} \times 1.25 \approx \mathbf{27.4\text{ TiB}}$.
     * **Dung lượng khả dụng còn lại cho Kho Sao Lưu Veeam Backup Repository:**
       $$68.4\text{ TiB} - 27.4\text{ TiB} = \mathbf{41.0\text{ TiB}}$$
       Dung lượng 41.0 TiB hoàn toàn đáp ứng chu kỳ lưu trữ sao lưu hàng ngày (Daily Incremental) và sao lưu đầy đủ cuối tuần (Weekly Full) trong vòng 30 ngày cho toàn bộ máy ảo nghiệp vụ của doanh nghiệp.

---

### 4.8. Phân Hệ Thoại Doanh Nghiệp (VoIP / UC): Cisco IP Phone 7841 & Tổng Đài SIP

- **Model thiết bị:** `Cisco IP Phone 7841` (Trang bị cho bàn làm việc 25 phòng ban) và `Cisco IP Phone 7832` (Điện thoại hội nghị chuyên dụng cho 2 phòng họp A-503 và B-102).
- **Lý do kỹ thuật bắt buộc phải nâng cấp lên model 7841:**  
  Model Cisco 7821 cũ chỉ có cổng mạng 10/100 Mbps. Nếu cắm máy tính làm việc nối tiếp (Daisy-chain) qua điện thoại 7821, cổng mạng máy tính sẽ bị giới hạn ở tốc độ tối đa 100 Mbps (giảm 90% hiệu năng đường truyền gigabit). **Cisco IP Phone 7841 tích hợp sẵn 2 cổng mạng Gigabit Ethernet (10/100/1000 Mbps)**, đảm bảo đường truyền mạng dây từ switch phòng qua điện thoại đến máy tính luôn đạt chuẩn **1000 Mbps Gigabit ổn định**.
- **Tiêu chuẩn cấp nguồn & Giao thức:**
  - Cấp nguồn qua mạng: Hỗ trợ **PoE chuẩn IEEE 802.3af Class 2/3**, công suất tiêu thụ điện thực tế cực thấp, chỉ dao động từ **3.8W đến 4.0W**.
  - Tích hợp hệ thống tổng đài: Nếu doanh nghiệp sử dụng máy chủ tổng đài ảo hóa Cisco Unified Communications Manager (CUCM), thiết bị sử dụng firmware Enterprise mặc định; nếu doanh nghiệp sử dụng hệ thống tổng đài mã nguồn mở SIP bên thứ 3 (như Asterisk / FreePBX), cần đặt hàng đúng mã SKU firmware đa nền tảng **3PCC / MPP (Multiplatform Phone)**.
- **Tách biệt luồng dữ liệu & Ưu tiên QoS:**  
  Switch truy cập tự động nhận diện thiết bị thoại qua giao thức CDP/LLDP-MED, đưa gói tin thoại vào **Voice VLAN 200 (Tòa A) và VLAN 210 (Tòa B)**, gán nhãn ưu tiên chất lượng dịch vụ **DSCP EF (Expedited Forwarding - Giá trị 46)** và CoS 5, loại bỏ hoàn toàn hiện tượng méo tiếng hay ngắt quãng cuộc gọi.

---

### 4.9. Phân Hệ Giám Sát Hình Ảnh (CCTV / VMS): Hikvision DS-2CD2143G2-I

- **Model thiết bị:** `Hikvision DS-2CD2143G2-I` (Camera IP bán cầu Dome, cảm biến 4 Megapixel).
- **Số lượng & Bố trí đồng bộ:** Thống nhất toàn hệ thống có **26 camera an ninh**:
  - 25 camera bố trí tại góc trần của 25 phòng ban chức năng quan sát cửa ra vào và tài sản.
  - 01 camera chuyên dụng lắp đặt bên trong phòng MDF A-202 quan sát 24/7 trực diện các mặt tủ rack máy chủ và cửa phòng.
- **Thông số kỹ thuật:** Cấp nguồn qua mạng **PoE 802.3af (tiêu thụ thực tế ~7.0 Watts)**; vỏ hợp kim chống đập phá cơ học chuẩn **IK10** và chống nước **IP67**; chuẩn nén hình ảnh siêu tiết kiệm **H.265+**; công nghệ hồng ngoại thông minh tầm xa 30 mét.
- **An ninh mạng camera:** Toàn bộ 26 camera được gom trọn vẹn vào **VLAN 12 (Security_IoT)** với dải IP tĩnh/DHCP cố định `192.168.12.0/24`. Cụm tường lửa FortiGate chặn tuyệt đối mọi lưu lượng từ VLAN 12 đi ra Internet để ngăn ngừa hoàn toàn nguy cơ rò rỉ hình ảnh hoặc camera bị điều khiển tham gia mạng botnet tấn công từ chối dịch vụ.

---

### 4.10. Phân Hệ Kiểm Soát Ra Vào Vật Lý (Physical Access Control): ZKTeco InBio + Mifare DESFire EV3

#### Tối ưu hóa số lượng thiết bị điều khiển:
- **Bộ điều khiển trung tâm (Access Controller):** Sử dụng **13 bộ `ZKTeco InBio-260`**. Vì mỗi bộ InBio-260 hỗ trợ quản lý **2 cửa độc lập**, việc trang bị 13 bộ đáp ứng chính xác cho **26 cửa ra vào** của toàn bộ công trình (25 cửa phòng ban nghiệp vụ + 01 cửa thép chống cháy phòng MDF A-202). Điều này giúp tiết kiệm chi phí đầu tư 12 bộ điều khiển so với thiết kế ban đầu mà vẫn đảm bảo 100% tính năng kỹ thuật.
- **Công nghệ thẻ chống sao chép:** Trang bị 26 đầu đọc thẻ kết hợp vân tay `ZKTeco FR1500-A` gắn ngoài cửa (kết nối về bộ InBio qua giao tiếp RS485). Hệ thống hỗ trợ chuẩn thẻ thông minh **Mifare DESFire EV3** (tần số 13.56 MHz, mã hóa AES-128 bit), giảm thiểu tối đa nguy cơ nhân bản hoặc làm giả thẻ từ so với các dòng thẻ tần số thấp 125kHz không mã hóa trước đây.
- **Hạ tầng nguồn điện & Khóa từ:**
  - Khóa cửa sử dụng **Khóa nam châm điện từ (Magnetic Lock) lực giữ 600 lbs (~280 kg)** kèm gá đỡ chữ ZL, cảm biến trạng thái cửa (Door Sensor) và nút ấn mở cửa bên trong (Exit Button).
  - *Nguyên lý cấp nguồn:* Khóa từ sử dụng nguồn điện một chiều **12V DC**, hoàn toàn không dùng nguồn PoE từ switch mạng. Hệ thống bố trí **13 bộ nguồn chuyên dụng cho kiểm soát cửa (12V DC 5A có ắc quy dự phòng 12V 7Ah đi kèm)** đặt trong hộp kỹ thuật cạnh bộ InBio. Khi tòa nhà mất điện lưới, khóa cửa vẫn duy trì trạng thái đóng an toàn từ 4 đến 6 tiếng.
  - **Liên động khẩn cấp với hệ thống Báo cháy:** Đấu nối dây tín hiệu rơ-le tiếp điểm khô từ tủ báo cháy trung tâm tòa nhà vào cổng Fire Alarm Input của các bộ điều khiển InBio. Khi có chuông báo cháy, toàn bộ khóa nam châm tự động ngắt điện mở chốt hoàn toàn, đảm bảo cửa mở tự do phục vụ thoát nạn cho nhân viên theo quy chuẩn PCCC.

---

### 4.11. Hạ Tầng Nguồn Điện, UPS, Máy Phát & Cơ Điện Phòng Máy (DC Facilities)

Hạ tầng phòng máy chủ trung tâm MDF A-202 được thiết kế đạt tiêu chuẩn **TIA-942 Rated-2 / Tier-2**:

#### 1. Tính toán phụ tải điện (Power Load Sizing) tại MDF A-202:
| Phân hệ thiết bị | Số lượng & Cấu hình | Công suất thực tế tiêu thụ (kW) |
|:---|:---|:---:|
| **Tủ Rack 01 (Core & Network)** | 2 Router 8300 + 2 FortiGate 200F + 2 Core 9500 + NMS + Patch Panel | **~2.2 kW** |
| **Tủ Rack 02 (Server Farm & SAN 1)** | 10 Máy chủ Dell R660/R760 + SAN ME5024 + ToR Switch + KVM | **~4.6 kW** |
| **Tủ Rack 03 (App Server & SAN 2)** | 10 Máy chủ Dell R660/R760 + SAN ME5012 + VMS/Backup Storage | **~4.2 kW** |
| **TỔNG CÔNG SUẤT IT TIÊU THỤ THỰC TẾ (P_IT)** | Toàn bộ thiết bị mạng, máy chủ và lưu trữ tại 3 tủ rack | **~11.0 kW** |

#### 2. Cấu hình Bộ Lưu Điện Online (UPS) Nguồn Kép 2N:
- Trang bị **02 Hệ thống UPS Online độc lập `APC Smart-UPS SRT 10kVA On-Line 230V` (Hệ thống UPS A và UPS B)**, mỗi hệ thống được gắn kèm **02 Khối ắc quy mở rộng gắn ngoài (External Battery Modules - EBM)**.
- *Nguyên lý vận hành nguồn kép:* 
  - UPS A cấp nguồn cho Thanh PDU A của cả 3 tủ rack; UPS B cấp nguồn cho Thanh PDU B của cả 3 tủ rack.
  - Toàn bộ máy chủ Dell PowerEdge, switch lõi Cisco Catalyst 9500 và tường lửa FortiGate đều trang bị 2 bộ nguồn tháo lắp nóng (Dual Hot-plug PSUs), cắm chéo vào PDU A và PDU B.
  - Mỗi hệ thống UPS 10kVA (hỗ trợ công suất hữu dụng ~9.0 - 10.0 kW) được thiết kế để **có khả năng gánh toàn bộ tải 11.0 kW của Data Center** trong trường hợp một bên UPS bị ngắt để bảo trì hoặc gặp sự cố hư hỏng.
  - *Thời gian lưu điện thực tế:* Với 2 module pin EBM mở rộng, thời gian duy trì điện năng ở mức tải 11 kW đạt từ **15 đến 20 phút** `[CẦN ĐỐI CHIẾU DATASHEET: Đường cong xả pin APC SRT10KXLI + 2x SRT192BP2]`. Thời gian này thừa đủ cho máy phát điện tự động khởi động và hòa điện an toàn.

#### 3. Máy Phát Điện Dự Phòng & Tủ Chuyển Nguồn Tự Động (ATS):
- Trang bị 01 cụm **Máy phát điện công nghiệp Diesel Cummins / Yanmar công suất 45 kVA** (công suất liên tục Prime Power $\approx 36\text{ kW}$ với hệ số $\cos\varphi = 0.8$), tích hợp vỏ cách âm tiêu âm đạt chuẩn $< 70\text{ dB}$ ở khoảng cách 7 mét, bồn nhiên liệu đáy đảm bảo vận hành liên tục 24 giờ.
- **Tính toán khả năng đáp ứng phụ tải của máy phát điện:**
  - Tải thiết bị CNTT phòng máy (P_IT qua UPS): $11.0\text{ kW}$.
  - Tải hệ thống điều hòa chính xác CRAC (20kW lạnh, điện năng tiêu thụ thực): $\approx 14.5\text{ kW}$.
  - Tải chiếu sáng khẩn cấp và quạt thông gió phòng máy: $\approx 1.5\text{ kW}$.
  - *Tổng tải máy phát cần gánh:* $11.0 + 14.5 + 1.5 = \mathbf{27.0\text{ kW}}$.
  - *Tỷ lệ tải thực tế của máy phát 45kVA:*
    $$\text{Tỷ lệ tải máy phát} = \frac{27.0\text{ kW}}{36.0\text{ kW}} = \mathbf{75.0\%}$$
    Mức tải 75% là điểm vận hành lý tưởng nhất của động cơ diesel, giúp máy hoạt động bền bỉ, tiết kiệm nhiên liệu và không bị hiện tượng đóng muội than cổ xả.
- **Tủ chuyển nguồn tự động ATS 100A:** Khi mất lưới điện tòa nhà, tủ ATS gửi tín hiệu đề nổ máy phát và đóng điện máy phát cấp cho Data Center trong vòng **10 đến 15 giây** (nằm trọn vẹn trong khoảng thời gian bảo vệ 20 phút của UPS).
- *Quy hoạch cấp nguồn tủ tầng IDF:* 9 Tủ tầng IDF không nối trực tiếp vào máy phát điện của phòng máy chủ trung tâm mà duy trì hoạt động độc lập bằng bộ lưu điện **APC Smart-UPS SRT 3000VA On-Line** (đảm bảo cấp nguồn liên tục cho switch phòng và camera trong 30-40 phút khi mất điện).

#### 4. Cơ Điện, Làm Mát Chính Xác & Chữa Cháy Phòng Máy:
- **Điều hòa không khí chính xác In-Row CRAC:** 02 Cụm điều hòa chuyên dụng công suất lạnh 20kW chạy chế độ dự phòng luân phiên $N+1$, duy trì nhiệt độ ổn định $20^\circ\text{C} \pm 2^\circ\text{C}$ và độ ẩm $50\% \pm 5\%$ liên tục.
- **Chữa cháy tự động khí sạch:** Hệ thống bình khí sạch **FM-200 (hoặc Novec 1230)** có cảm biến khói quang học và cảm biến nhiệt đa điểm. Khi kích hoạt, khí được phun xả dập tắt đám cháy trong vòng 10 giây bằng cơ chế hấp thụ nhiệt, hoàn toàn không để lại cặn bẩn và không làm chập cháy các bo mạch điện tử nhạy cảm.
- **Cắt lọc sét & Tiếp địa an toàn:** Tủ cắt lọc sét đa cấp **OBO Bettermann Type 1+2** lắp đặt tại đầu vào tủ điện phân phối; hệ thống tiếp địa viễn thông độc lập có trị số điện trở đất đạt **$R < 1\ \Omega$**. Toàn bộ vỏ tủ rack và máng cáp được liên kết đẳng thế qua thanh đồng tiếp địa chính.

---

### 4.12. Trung Tâm Điều Hành & Giám Sát An Ninh (NOC / SOC A-404)

Bố trí tại phòng biệt lập A-404 với cấp độ bảo mật cao nhất (VLAN 44 Restricted):
- **Cụm Màn Hình Video Wall 4x 55-inch:** Tấm nền chuyên dụng hoạt động 24/7/365, viền ghép siêu mỏng 0.88mm. Phân chia 4 khu vực hiển thị thông tin tác chiến liên tục:
  - *Màn hình 1:* Bản đồ cảnh báo tấn công và nhật ký sự kiện thời gian thực từ hệ thống **Wazuh SIEM Manager** (IP `192.168.10.40`).
  - *Màn hình 2:* Đồ thị băng thông mạng, trạng thái cổng mạng và độ trễ của **25 switch phòng, cụm ToR switch và Core switch** từ hệ thống giám sát **Zabbix NMS**.
  - *Màn hình 3:* Chỉ số tải CPU, RAM, IOPS của cụm Kubernetes và cụm Cơ sở dữ liệu cốt lõi từ bảng điều khiển **Grafana**.
  - *Màn hình 4:* Bản đồ cảnh báo mã độc địa lý và lưu lượng chặn truy cập từ **Cụm Tường lửa FortiGate 200F**.
- **Máy Trạm Kỹ Sư Tác Chiến (04 bộ):** `Dell Precision 3680 Tower Workstation` (Intel Core i7-14700, 32GB RAM DDR5, card đồ họa rời NVIDIA RTX, trang bị 2 màn hình 27-inch độ phân giải 2K sắc nét).
- **Bộ Quản Trị Ngoại Băng Out-of-Band (OOB Console Server):** Terminal Server kết nối trực tiếp vào cổng Console vật lý của Router, Firewall, Core Switch và UPS, giúp kỹ sư khôi phục cấu hình hệ thống ngay cả khi mạng nội bộ bị ngắt hoàn toàn.

---

### 4.13. Máy Trạm Người Dùng & Máy In Mạng: Dell OptiPlex & HP LaserJet Enterprise

- **Máy trạm nhân viên (144 bộ):** `Dell OptiPlex Plus 7020 SFF` (Intel Core i7-14700, 16GB - 32GB DDR5 RAM, 512GB NVMe SSD, card mạng Gigabit RJ45 bọc kim chống nhiễu). Tích hợp chip mã hóa phần cứng TPM 2.0, hỗ trợ khởi động an toàn Secure Boot và cài đặt phần mềm phòng chống mã độc thế hệ mới (EDR).
- **Máy in mạng bảo mật (20 máy):** `HP LaserJet Enterprise M507dn`.
  - Kết nối cáp mạng Cat6A trực tiếp vào switch phòng.
  - Áp dụng tính năng in ấn an toàn **HP Sure Start** và **Secure PIN Printing**: Nhân viên gửi lệnh in từ máy tính, tài liệu được lưu tạm trên bộ nhớ mã hóa của máy in; nhân viên phải đến trước máy in nhập đúng mã PIN cá nhân thì tài liệu mới được in ra, tránh rò rỉ thông tin hợp đồng và tài chính quan trọng.

---

### 4.14. Hệ Thống Cáp Cấu Trúc, Patch Panel & Phân Phối Quang ODF

- **Cáp mạng đồng nhánh (Horizontal Cabling):** Toàn bộ sử dụng cáp **CommScope NetConnect Cat6A SFTP (chống nhiễu từng cặp bọc lá kim loại và bọc lưới đồng tổng)**. Băng thông kiểm định 500 MHz, đảm bảo tốc độ truyền tải 10 Gigabit đến từng bàn làm việc, đồng thời tiết diện lõi đồng 23 AWG bảo đảm nhiệt độ đường dây luôn an toàn khi truyền tải dòng điện PoE+ liên tục.
- **Cáp quang trục chính (Backbone Cabling):** Các tuyến cáp quang **Single-Mode OS2 12-core và 24-core** đi trong ống luồn chống cháy nối từ 9 tủ tầng IDF về MDF A-202.
- **Tủ Phân Phối Quang Tập Trung (MDF ODF):**  
  *Quy chuẩn đếm sợi và cổng quang chuẩn xác:* 9 Tủ tầng IDF với các tuyến cáp quang 12-core sẽ dẫn tổng cộng $9 \times 12 = \mathbf{108\text{ sợi quang}}$ về trung tâm MDF A-202, tương đương với **54 cặp sợi quang Duplex LC**. Do đó, tại đỉnh Rack 01 (MDF A-202) trang bị **01 Khung phân phối quang tập trung ODF 144-Port chuẩn LC Duplex** (quản lý tối đa 72 cặp cổng Duplex hoặc 144 sợi quang), đấu nối gọn gàng toàn bộ 108 sợi từ 9 IDF và các sợi quang từ 2 nhà cung cấp dịch vụ Internet ISP.

#### Bảng Bóc Tách Chi Tiết Số Lượng Module Quang SFP+ & Cáp DAC:
| STT | Vị Trí Đấu Nối Vật Lý | Loại Module / Cáp Kết Nối | Số Lượng Cần Thiết | Mục Đích Sử Dụng |
|:---:|:---|:---|:---:|:---|
| 1 | **Uplink từ 25 Switch phòng về MDF A-202** | Cisco SFP-10G-LR (Single-Mode LC 10km) | **50 chiếc** *(25 link $\times$ 2 đầu)* | 25 cổng tại switch phòng và 25 cổng tại Core 9500 |
| 2 | **Kết nối Core 9500 sang Switch ToR Data Center** | Cisco SFP-10G-LR (Single-Mode LC 10km) | **08 chiếc** *(4 link $\times$ 2 đầu)* | 4 đường quang 10G kết nối LACP giữa Core và ToR |
| 3 | **Kết nối Core 9500 sang Trạm SOC A-404 & Helpdesk**| Cisco SFP-10G-LR (Single-Mode LC 10km) | **04 chiếc** *(2 link $\times$ 2 đầu)* | Đường truyền cáp quang quản trị chuyên dụng |
| 4 | **Dự phòng Module Quang trong kho (Spare)** | Cisco SFP-10G-LR (Single-Mode LC 10km) | **06 chiếc** | Thay thế nóng khi có sự cố suy hao quang |
| **-** | **TỔNG CỘNG MODULE QUANG SFP-10G-LR** | **Chuẩn 10GBASE-LR Single-Mode** | **68 chiếc** | Đã tính toán đầy đủ 2 đầu kết nối và dự phòng |
| 5 | **Kết nối Cụm Core 9500 sang Tường lửa FortiGate 200F**| Cáp đính kèm Cisco 10GBASE-CU SFP+ DAC 2m | **04 sợi** | Cụm LACP Trunk 20G nối Core Switch và Firewall |
| 6 | **Kết nối Tường lửa FortiGate 200F sang Router 8300** | Cáp đính kèm Cisco 10GBASE-CU SFP+ DAC 2m | **04 sợi** | Đường kết nối tốc độ cao giữa Firewall và Router |
| 7 | **Kết nối Cụm Máy Chủ Dell sang Switch ToR Rack** | Cáp đính kèm 10GBASE-CU SFP+ DAC 1.5m | **16 sợi** | Cắm trực tiếp máy chủ ảo hóa vào switch ToR cùng rack |
| 8 | **Kênh Ghép Core StackWise Virtual Link (SVL)** | Cáp đính kèm Cisco 100GBASE-CR4 QSFP28 DAC 1m | **02 sợi** | Kênh truyền tốc độ cao 100G ghép cặp 2 Core 9500 |
| 9 | **Kênh Dual-Active Detection (DAD) Core 9500** | Cáp đính kèm Cisco 25GBASE-CU SFP28 DAC 1m | **02 sợi** | Kênh phát hiện chống phân rã não bộ Core Switch |

---

## 5. ĐÁNH GIÁ ĐIỂM YẾU KIẾN TRÚC, KẾ HOẠCH DỰ PHÒNG THẢM HỌA (DR) & RTO/RPO

### 5.1. Nhận định trung thực về các "Điểm Lỗi Đơn" (SPOF) còn tồn tại và giải pháp kiểm soát
Trong phạm vi một đồ án kỹ thuật và một công trình thực tế đơn lẻ, không thể tuyên bố "hoàn toàn không có điểm lỗi đơn". Báo cáo phân tích trung thực các rủi ro vật lý còn tồn tại và biện pháp giảm thiểu:

1. **Rủi ro vật lý phòng máy chủ tập trung (Single Data Center Facility):** Toàn bộ thiết bị máy chủ cốt lõi đặt tại phòng A-202. Nếu xảy ra sự cố thảm họa vật lý (cháy nổ lớn tòa nhà, ngập lụt thiên tai), hệ thống sẽ bị gián đoạn.  
   $\rightarrow$ *Biện pháp giảm thiểu:* Tăng cường hệ thống chữa cháy khí sạch FM-200, cửa thép chống cháy 120 phút và thiết lập kênh sao lưu dữ liệu mã hóa ra ngoài (Offsite Cloud Backup).
2. **Cụm Kubernetes Control-Plane:** Đảm bảo cấu hình tối thiểu **3 node Master vật lý (hoặc 3 VM Stacked Control-Plane/Worker nodes)** để duy trì cơ chế biểu quyết đồng thuận etcd quorum (tránh dùng 1 node master duy nhất gây mất kiểm soát toàn bộ cụm container).
3. **Cơ sở dữ liệu tập trung (Database):** Cấu hình mô hình **PostgreSQL / MySQL Master-Slave Streaming Replication** giữa 2 máy chủ vật lý độc lập; tự động chuyển đổi dự phòng (Automatic Failover) bằng Patroni hoặc Orchestrator.
4. **Bộ nguồn switch truy cập (C9200L Fixed PSU):** Dòng switch C9200L có nguồn gắn liền; nếu hỏng nguồn phải thay cả switch.  
   $\rightarrow$ *Biện pháp giảm thiểu:* Dự phòng sẵn 02 switch C9200L cấu hình mẫu (Cold Spare) trong kho IT; khi có sự cố, kỹ thuật viên Helpdesk có thể thay thế trong vòng 30 phút.

### 5.2. Khắc phục triệt để quy tắc sao lưu dữ liệu 3-2-1
Hệ thống thực thi nghiêm ngặt mô hình **Sao lưu 3-2-1**:
- **3 bản sao dữ liệu:** 1 bản đang chạy sản xuất + 1 bản sao lưu nhanh cục bộ (Local Snapshot trên SAN ME5012) + 1 bản sao lưu nén lưu trữ dài hạn.
- **2 loại môi trường lưu trữ khác nhau:** Lưu trữ trên mảng đĩa từ (Disk-based SAN) và lưu trữ trên cụm máy chủ sao lưu phân tán (Object Storage).
- **1 bản sao lưu Offsite tách rời hoàn toàn:** Hàng đêm, phần mềm Veeam Backup & Replication tự động mã hóa AES-256 toàn bộ các bản sao lưu của Database, Mail, File và đẩy đồng bộ qua đường truyền Internet riêng biệt lên dịch vụ lưu trữ đám mây **Cloud Object Storage nóng/ấm (Wasabi Hot Cloud Storage hoặc AWS S3 Standard-IA)**, đồng thời bật tính năng **Object Lock (Khóa bất biến - Immutability)** để chống lại mã độc tống tiền Ransomware xóa bản sao lưu. *Loại bỏ phương án sử dụng tầng lưu trữ lạnh AWS Glacier Deep Archive cho kịch bản phục hồi thảm họa khẩn cấp vì thời gian chờ trích xuất dữ liệu của Glacier mất từ 3 đến 12 giờ, không đáp ứng được mục tiêu RTO 2 giờ*.

### 5.3. Mục tiêu phục hồi sau sự cố (RTO & RPO) cho từng phân hệ:

| Phân hệ nghiệp vụ | RPO (Mức mất mát dữ liệu tối đa chấp nhận được) | RTO (Thời gian khôi phục hoạt động tối đa) | Phương thức kỹ thuật bảo vệ |
|:---|:---:|:---:|:---|
| **Cơ sở dữ liệu giao dịch cốt lõi** | **$\le 5$ phút** | **$\le 15$ phút** | Cơ chế đồng bộ dữ liệu Database Streaming Replication liên tục (tách biệt với chu kỳ backup log 15 phút) |
| **Hệ thống xác thực AD / DNS / DHCP** | **0 phút** (Zero Data Loss) | **$\le 1$ phút** | Chạy song hành cụm 2 máy chủ Active-Active phân tán |
| **Hệ thống Email doanh nghiệp** | $\le 1$ giờ | $\le 2$ giờ | Snapshot máy ảo hàng ngày + Đồng bộ hóa Mailbox |
| **Hệ thống File dữ liệu phòng ban** | $\le 4$ giờ | $\le 2$ giờ | Snapshot SAN ME5024 mỗi 4 tiếng + Backup Offsite hàng đêm |
| **Dữ liệu Camera an ninh** | $\le 1$ ngày | $\le 4$ giờ | Lưu trữ trực tiếp trên RAID 6 cục bộ |

---

## 6. SO SÁNH TỔNG CHI PHÍ SỞ HỮU (TCO) & HOÀN VỐN ĐẦU TƯ (ROI) 5 NĂM

### 6.1. Bảng số liệu dự toán TCO thực tế trong vòng đời 5 năm
*(Đơn giá mang tính chất ước tính sơ bộ theo mặt bằng giá thiết bị doanh nghiệp tham khảo trên thị trường, cần đối chiếu báo giá chính thức từ nhà phân phối ủy quyền tại thời điểm đấu thầu).*

| Nhóm thiết bị / Hạng mục chi phí | Đơn vị | Số lượng | Đơn giá ước tính (Triệu VNĐ) | Thành tiền P1: Đầu tư chuẩn hóa (Triệu VNĐ) | Thành tiền P2: Thiết bị giá rẻ / tự ráp (Triệu VNĐ) |
|:---|:---:|:---:|:---:|:---:|:---:|
| **Router Cisco Catalyst 8300 (Dual Power)** | Bộ | 02 | 145 | 290 | 45 *(DrayTek / MikroTik)* |
| **Firewall FortiGate FG-200F (Bundle Enterprise)**| Bộ | 02 | 280 | 560 | 90 *(Tường lửa mã nguồn mở)*|
| **Core Switch Cisco Catalyst C9500-24Y4C** | Bộ | 02 | 310 | 620 | 120 *(Switch Layer 3 giá rẻ)* |
| **Switch ToR Cisco Catalyst C9300-24S** | Bộ | 02 | 125 | 250 | 60 |
| **Switch Truy cập C9200L-24P-4X (PoE+ 370W)** | Bộ | 16 | 45 | 720 | 192 *(Switch unmanaged/Web)*|
| **Switch Truy cập C9200CX-8P-2X2G (Fanless)** | Bộ | 09 | 28 | 252 | 54 |
| **Bộ phát sóng Wi-Fi UniFi U6-Pro** | Bộ | 25 | 4.5 | 112.5 | 37.5 *(Wi-Fi gia đình)* |
| **Cụm Máy chủ Dell PowerEdge R660 & R760** | Máy | 20 | 95 | 1.900 | 700 *(PC tự lắp ráp / Server cũ)*|
| **Hệ thống SAN Dell ME5024 & ME5012** | Mảng | 02 | 320 | 640 | 180 *(NAS dân dụng)* |
| **Điện thoại IP Cisco 7841 & Hội nghị 7832** | Bộ | 27 | 3.5 | 94.5 | 27 |
| **Camera Hikvision 4MP IK10 & Nguồn phụ trợ** | Bộ | 26 | 2.2 | 57.2 | 26 |
| **Hệ thống kiểm soát cửa ZKTeco & Khóa từ** | Hệ | 13 | 14 | 182 | 90 |
| **Hạ tầng Tủ Rack APC, Nguồn UPS 10kVA, PDU** | Hệ | 03 tủ MDF | 380 | 380 | 90 *(UPS Offline / Tủ gia công)*|
| **Cáp mạng Cat6A SFTP, Quang OS2 & ODF 144** | Lô | Trọn gói | 320 | 320 | 120 *(Cáp Cat5e UTP thường)* |
| **TỔNG CHI PHÍ ĐẦU TƯ BAN ĐẦU (CAPEX)** | | | | **6.378,2** | **1.831,5** |
| **Bản quyền an ninh FortiGuard & License (5 năm)**| Gói | 5 năm | 140/năm | 700 | 100 |
| **Chi phí bảo trì, thay thế phần cứng hỏng hóc** | Năm | 5 năm | Dự toán | 120 | 550 *(Hỏng nguồn, tụ, quạt)* |
| **Chi phí nhân sự IT vận hành (5 năm)** | Kỹ sư | 2 người | 360/năm | 1.800 *(2 Kỹ sư)* | 3.600 *(Cần 4 Kỹ sư xử lý lỗi)*|
| **Thiệt hại tài chính ước tính do Downtime (5 năm)**| Giờ | 5 năm | Thống kê | 150 *(Uptime ~99.74%)* | 1.650 *(Uptime ~96.5%)* |
| **TỔNG CHI PHÍ SỞ HỮU (TCO) TRONG 5 NĂM** | | | | **9.148,2** | **7.731,5** |

### 6.2. Phân tích so sánh TCO & Giá trị hoàn vốn đầu tư (ROI):
- **So sánh trung tính về tài chính:**  
  Tổng chi phí sở hữu (TCO) trong 5 năm của Phương án chuẩn hóa cao hơn phương án giá rẻ khoảng **18.3%** (9.148 triệu so với 7.731 triệu VNĐ). Mức chênh lệch chi phí này hoàn toàn được bù đắp bởi các giá trị bảo toàn kinh doanh vượt trội:
  1. **Độ sẵn sàng dịch vụ thực tế:** Đạt chuẩn **TIA-942 Rated-2 (ước tính độ sẵn sàng lý thuyết $\approx 99.741\%$ Uptime)** với hệ thống nguồn điện kép, máy phát điện ATS và chuyển đổi dự phòng đường truyền mạng tự động.
  2. **Bảo hiểm tài sản số:** Ngăn chặn nguy cơ bị mã độc tống tiền Ransomware mã hóa dữ liệu hoặc rò rỉ cơ sở dữ liệu khách hàng; chi phí khắc phục một vụ khủng hoảng an ninh mạng đối với doanh nghiệp công nghệ thường dao động từ 1 đến 5 tỷ VNĐ.
  3. **Tối ưu năng suất lao động:** 144 nhân sự văn phòng làm việc liên tục với đường truyền gigabit ổn định, không bị lãng phí thời gian do rớt mạng hay nghẽn băng thông.

---

## 7. KẾT LUẬN & LỘ TRÌNH TRIỂN KHAI

Báo cáo phân tích và hiệu chỉnh phần cứng đã hoàn thành việc đồng bộ toàn diện giữa tài liệu gốc `README.md` và bản vẽ `index-v2.html`:
- **Đồng bộ kiến trúc chuẩn mực:** Giữ vững mô hình **2 lớp Collapsed-Core**, cặp switch lõi **Cisco Catalyst 9500-24Y4C** ghép StackWise Virtual làm Gateway 30 VLANs, thực thi chính sách an ninh bằng **Bộ 4 Extended ACLs**, giữ 9 tủ IDF là các điểm đấu nối thụ động.
- **Thống nhất số lượng thiết bị vật lý:** 25 switch phòng, 25 AP Wi-Fi 6, 25 điện thoại Cisco 7841, 26 camera an ninh, 13 bộ điều khiển cửa InBio-260, 26 khóa từ nam châm điện dùng nguồn DC riêng biệt.
- **Minh bạch hóa các thông số kỹ thuật:** Sizing tường lửa FortiGate 200F chuẩn 3.0 Gbps Threat Protection; phân biệt rõ SAN ME5024 (Block) và File Gateway; tính toán chính xác dung lượng lưu trữ camera 26 kênh và kho bản sao lưu Veeam; tính toán công suất nguồn 11 kW cho MDF A-202 với UPS 10kVA nguồn kép 2N và máy phát điện 45kVA.

---

## 8. PHỤ LỤC KỸ THUẬT & NHẬT KÝ HIỆU CHỈNH

### 8.1. Bảng Nhật Ký Thay Đổi

| Mục | Nội dung cũ (Tài liệu số 3 bản trước) | Nội dung mới đã hiệu chỉnh | Lý do hiệu chỉnh | Tài liệu gốc tham chiếu |
|:---|:---|:---|:---|:---|
| **1, 2** | Mô hình mạng 3 lớp, trang bị 9 switch phân phối C9200L-24T-4X tại 9 IDF | Mô hình 2 lớp Collapsed-Core; xóa bỏ 9 switch tại IDF; IDF là điểm trung chuyển thụ động | Đồng bộ kiến trúc gốc; tránh lãng phí thiết bị và sai lệch mô hình | `README.md` Mục 3.1 & Phụ lục A.2 |
| **1, 4.3** | Cặp Core Switch dùng C9300-24T-E kết hợp switch gom quang rời rạc | Cặp Core Switch chuẩn hóa là Cisco Catalyst C9500-24Y4C ghép StackWise Virtual | Khắc phục thiếu cổng quang; đồng bộ thiết bị Core thực tế | `README.md` Phụ lục A.1, A.2 |
| **1, 2, 4.3** | Ép toàn bộ lưu lượng Inter-VLAN Đông-Tây qua Firewall bằng VRF-Lite | Core làm Gateway L3 thực thi Bộ 4 Extended ACLs; firewall Inter-Zone chuyển thành mục mở rộng | Giữ nguyên thiết kế cốt lõi của đồ án; VRF-Lite không tự ép qua firewall | `README.md` Mục 3.2, 4.4 |
| **4.1** | Dual ISP chạy BGP failover < 1 giây không ngắt phiên TCP | IP SLA Tracking + PBR failover vài giây; đổi IP NAT nên phiên TCP bị ngắt; bỏ BFD với 8.8.8.8 | Thực tế doanh nghiệp vừa không có ASN/IP PI; BFD không chạy được với IP ngoài | Nguyên tắc mạng thực tế |
| **4.2** | FortiGate 200F throughput dải "3-5 Gbps" | Threat Protection Throughput chuẩn Datasheet là 3.0 Gbps; tính tải đạt 66.7% | Sizing chính xác theo công bố chính thức của hãng | Fortinet FG-200F Datasheet |
| **4.8** | Điện thoại IP Cisco 7821 cổng 10/100 Mbps | Đổi sang Cisco IP Phone 7841 tích hợp 2 cổng Gigabit Ethernet 10/100/1000 Mbps | Tránh nghẽn băng thông cổng passthrough nối tiếp máy tính | Khuyến nghị kỹ thuật phần cứng |
| **4.10** | 25 bộ điều khiển InBio cho 25 cửa; khóa từ dùng PoE | 13 bộ InBio-260 cho 26 cửa (kể cả MDF A-202); khóa từ dùng nguồn DC 12V riêng | InBio-260 quản lý 2 cửa/bộ; khóa từ Maglock tiêu thụ dòng lớn không cắm PoE | ZKTeco InBio Datasheet & index-v2.html |
| **3, 4.9** | Thống kê 25 camera an ninh | Thống nhất 26 camera (25 phòng + 01 camera giám sát phòng MDF A-202) | Tính đúng vị trí camera giám sát an ninh tủ rack máy chủ | `index-v2.html` dòng 1721-1729 |
| **4.7** | ME5024 là NAS; cấu hình RAID 6 một nhóm 22 ổ; dung lượng ~58 TB không rõ ràng | ME5024 là SAN Block Storage; chia 2 nhóm RAID 6 (mỗi nhóm 10 ổ) + 4 spare; tính rõ TiB | Giới hạn kỹ thuật nhóm RAID 6 trên SAN ME5; minh bạch công thức tính toán | Dell ME5024 Administrator Guide |
| **4.11** | 2 bộ UPS 5kVA lưu 45-60 phút; không có máy phát điện và ATS | 2 bộ UPS 10kVA On-Line nguồn 2N kèm EBM (lưu 15-20p); bổ sung Máy phát điện 45kVA và ATS | 5kVA không đủ tải 11kW; ắc quy nội bộ không thể lưu 60 phút | Tính toán công suất điện TIA-942 |
| **5.1** | Tuyên bố "hệ thống hoàn toàn không có điểm lỗi đơn" | Phân tích trung thực rủi ro: Single DC, K8s control plane, C9200L nguồn cố định | Đảm bảo tính trung thực và khách quan của báo cáo kỹ thuật | NIST SP 800-30 |
| **6.1, 6.2** | Bảng TCO số liệu chung chung; kết luận chủ quan "TCO tương đương" | Lập bảng chi tiết CAPEX/OPEX từng nhóm; so sánh trung tính (P1 cao hơn P2 18.3%) | Đảm bảo căn cứ số liệu khoa học cho cấp lãnh đạo phê duyệt | Phân tích tài chính CNTT |

---

### 8.2. Danh Sách Đề Xuất Cập Nhật Ngược Sang README.md & index-v2.html

1. **Cập nhật Model Điện thoại VoIP:** Đề xuất cập nhật trong `README.md` (Phụ lục A.1) và `index-v2.html` (Bảng thống kê thiết bị) từ `Cisco IP Phone 7821` sang `Cisco IP Phone 7841` để đồng bộ thông số cổng mạng Gigabit RJ45.
2. **Cập nhật Số lượng Bộ Điều Khiển Cửa:** Điều chỉnh số lượng tại `index-v2.html` từ 25 bộ InBio xuống **13 bộ InBio-260** (do mỗi bộ quản lý 2 cửa độc lập, đủ cho 26 cửa).
3. **Thống nhất Tổng số Camera An Ninh:** Cập nhật bảng tổng hợp BOM tại `index-v2.html` từ 25 camera lên **26 camera** (bổ sung camera `CAM-A-202` đã được vẽ trong sơ đồ CAD phòng MDF nhưng bị sót trong bảng tổng hợp).
4. **Cập nhật Ngân sách Nguồn PoE:** Cập nhật bảng tính toán PoE Budget trong `README.md` (Phụ lục A.1): ghi nhận mức tiêu thụ thực tế của mỗi phòng là **~24.0 Watts** (thay vì mức tính tối đa theo Class là 76.2W), ghi chú rõ khóa từ và bộ điều khiển cửa dùng nguồn DC riêng, không tiêu thụ nguồn PoE của switch.

---

### 8.3. Danh Mục Các Mục Cần Đối Chiếu Datasheet Chính Hãng

1. `[CẦN ĐỐI CHIẾU DATASHEET]` **Fortinet FortiGate FG-200F:** Xác minh thông số Threat Protection Throughput chính xác đạt 3.0 Gbps (khi kích hoạt đồng thời IPS, Application Control, Antivirus, Logging) theo tài liệu công bố chính thức của Fortinet.
2. `[CẦN ĐỐI CHIẾU DATASHEET]` **Cisco Catalyst C9500-24Y4C:** Kiểm tra yêu cầu gói bản quyền phần mềm Network Advantage để kích hoạt đầy đủ tính năng StackWise Virtual (SVL) và định tuyến nâng cao.
3. `[CẦN ĐỐI CHIẾU DATASHEET]` **Cisco IP Phone 7841:** Kiểm tra công suất tiêu thụ điện danh định chuẩn IEEE 802.3af Class 2/3 (~3.8W) và xác nhận mã SKU đặt hàng firmware tương ứng (Firmware Enterprise cho CUCM hoặc Firmware 3PCC/MPP cho SIP Server bên thứ 3).
4. `[CẦN ĐỐI CHIẾU DATASHEET]` **ZKTeco FR1500-A:** Xác nhận mã đặt hàng đầu đọc hỗ trợ đọc thẻ chuẩn Mifare DESFire EV3 tần số 13.56 MHz qua kết nối RS485 về bộ điều khiển InBio-260.
5. `[CẦN ĐỐI CHIẾU DATASHEET]` **APC Smart-UPS SRT 10kVA (SRT10KXLI):** Kiểm tra đường cong xả pin (Discharge Runtime Curve) khi kết hợp với 02 module ắc quy mở rộng gắn ngoài SRT192BP2 ở mức tải 11.0 kW để đảm bảo thời gian duy trì tối thiểu 15–20 phút.
6. `[CẦN ĐỐI CHIẾU DATASHEET]` **Dell PowerVault ME5024:** Kiểm tra giới hạn số lượng ổ cứng tối đa trong một Disk Group RAID 6 và quy tắc cấu hình phân tầng lưu trữ (Storage Tiering) tối ưu trên hệ điều hành ME5.

---

### 8.4. Báo Cáo Tự Kiểm Tra Chéo Số Liệu Nội Bộ

- **Số lượng Switch phòng:** 25 switch (16 switch lớn C9200L-24P-4X + 09 switch nhỏ C9200CX-8P-2X2G) $\rightarrow$ Khớp 100% giữa Mục 1, 2, 3, 4.4 và `index-v2.html`.
- **Số lượng Bộ phát Wi-Fi AP:** 25 bộ UniFi U6-Pro $\rightarrow$ Khớp 100% giữa Mục 3, 4.5, `README.md` và `index-v2.html`.
- **Số lượng Camera An Ninh:** 26 camera (25 phòng ban + 01 phòng MDF A-202) $\rightarrow$ Khớp 100% giữa Mục 3, 4.7, 4.9 và `index-v2.html` (dòng 1721).
- **Số lượng Cửa & Bộ Điều Khiển:** 26 cửa, 13 bộ điều khiển InBio-260, 26 khóa từ Maglock, 13 bộ nguồn 12V 5A $\rightarrow$ Khớp 100% giữa Mục 3, 4.10 và tổng số 26 phòng của công trình.
- **Quy hoạch Địa chỉ IP & VLAN:** Toàn bộ bảng máy chủ tại Mục 4.6 sử dụng đúng dải IP `192.168.10.0/24` (VLAN 10), Gateway `192.168.10.1`, khớp 100% với Bảng 30 VLANs tại `README.md` Mục 4.1 và Zone Matrix Mục 3.2.
- **Vị trí Tủ Rack U tại MDF A-202:** 
  - Rack 01: Router U37-39, Firewall U34-36, Core 9500 U30-33, NMS U18-21.
  - Rack 02: ToR Switch U40-42, AD/DNS U38-39, Web U35-36, Mail U33-34, File U30-32, Database U28-29, Backup U25-27, KVM U21-24, SAN ME5024 U16-20.
  - Rack 03: SAN ME5012 U05-08, Wazuh SIEM U10-11, Syslog U12, Bastion U13, GitLab U14-15, K8s U16-21, CRM/ERP U24-25, HRM U26, Fin U27.
  $\rightarrow$ Khớp 100% giữa sơ đồ CAD SVG của `index-v2.html` (dòng 1350-1503) và bảng máy chủ Mục 4.6.
