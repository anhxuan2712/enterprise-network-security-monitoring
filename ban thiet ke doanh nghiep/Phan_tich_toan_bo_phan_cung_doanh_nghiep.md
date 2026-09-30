# BÁO CÁO PHÂN TÍCH VÀ HIỆU CHỈNH TOÀN DIỆN THIẾT KẾ PHẦN CỨNG HẠ TẦNG MẠNG & AN NINH DOANH NGHIỆP

> **Tài liệu tham chiếu:** Bản vẽ thiết kế kỹ thuật hạ tầng `index-v2.html`  
> **Tiêu chuẩn thiết kế áp dụng:** TIA-942 (Telecommunications Infrastructure Standard for Data Centers), ANSI/TIA-568.2-D, NIST SP 800-207 (Zero Trust Architecture), ISO/IEC 27001:2022 Controls.  
> **Quy mô hệ thống:** 2 Tòa nhà (Tòa A: 5 tầng, Tòa B: 4 tầng) • 26 Phòng ban • 144 Người dùng cuối + 2 Quản trị viên hệ thống • 1 Phòng Máy chủ trung tâm (MDF A-202) • 9 Tủ mạng phân phối tầng (IDF) • 1 Trung tâm điều hành an ninh mạng (NOC/SOC A-404).

---

## MỤC LỤC

1. [TỔNG QUAN KIẾN TRÚC MẠNG & SỬA ĐỔI THIẾT KẾ CỐT LÕI](#1-tổng-quan-kiến-trúc-mạng--sửa-đổi-thiết-kế-cốt-lõi)
2. [SƠ ĐỒ TOPOLOGY PHÂN TẦNG VÀ LUỒNG DỮ LIỆU ĐÔNG - TÂY / BẮC - NAM](#2-sơ-đồ-topology-phân-tầng-và-luồng-dữ-liệu-đông---tây--bắc---nam)
3. [DANH MỤC THIẾT BỊ PHẦN CỨNG ĐÃ ĐƯỢC CHUẨN HÓA (BILL OF MATERIALS)](#3-danh-mục-thiết-bị-phần-cứng-đã-được-chuẩn-hóa-bill-of-materials)
4. [PHÂN TÍCH CHI TIẾT TỪNG PHÂN HỆ PHẦN CỨNG](#4-phân-tích-chi-tiết-từng-phân-hệ-phần-cứng)
   - [4.1. Phân Hệ Biên & Kết Nối WAN (Edge Routing): Cisco Catalyst 8300](#41-phân-hệ-biên--kết-nối-wan-edge-routing-cisco-catalyst-8300)
   - [4.2. Phân Hệ Tường Lửa Thế Hệ Mới (NGFW): Fortinet FortiGate 200F / 120G HA Cluster](#42-phân-hệ-tường-lửa-thế-hệ-mới-ngfw-fortinet-fortigate-200f--120g-ha-cluster)
   - [4.3. Phân Hệ Chuyển Mạch Lõi & Gom Tầng (Core / Aggregation Switching): Cisco Catalyst 9500 & 9300](#43-phân-hệ-chuyển-mạch-lõi--gom-tầng-core--aggregation-switching-cisco-catalyst-9500--9300)
   - [4.4. Phân Hệ Tủ Tầng (IDF) & Chuyển Mạch Truy Cập (Access Switching): Cisco Catalyst C9200L & C9200CX](#44-phân-hệ-tủ-tầng-idf--chuyển-mạch-truy-cập-access-switching-cisco-catalyst-c9200l--c9200cx)
   - [4.5. Phân Hệ Mạng Không Dây Doanh Nghiệp (Enterprise WLAN): Ubiquiti UniFi U6-Pro](#45-phân-hệ-mạng-không-dây-doanh-nghiệp-enterprise-wlan-ubiquiti-unifi-u6-pro)
   - [4.6. Phân Hệ Máy Chủ Dịch Vụ Data Center: Dell PowerEdge R660 & R760](#46-phân-hệ-máy-chủ-dịch-vụ-data-center-dell-poweredge-r660--r760)
   - [4.7. Phân Hệ Lưu Trữ Khối Chuyên Dụng (SAN Block Storage): Dell PowerVault ME5024 / ME5012](#47-phân-hệ-lưu-trữ-khối-chuyên-dụng-san-block-storage-dell-powervault-me5024--me5012)
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

---

## 1. TỔNG QUAN KIẾN TRÚC MẠNG & SỬA ĐỔI THIẾT KẾ CỐT LÕI

Dựa trên quá trình rà soát toàn diện bản vẽ kỹ thuật `index-v2.html`, tài liệu này hiệu chỉnh và làm rõ toàn bộ các bất cập về mật độ cổng, tính toán luồng dữ liệu, công suất nguồn, tính chất lưu trữ và ranh giới an ninh nhằm đảm bảo hệ thống có thể triển khai thực tế chính xác và an toàn.

### Các hiệu chỉnh kỹ thuật mang tính quyết định:
1. **Bổ sung Lớp Aggregation / Core SFP+ chuyên dụng:**  
   Cặp switch lõi `Cisco Catalyst 9300-24T-E` (chỉ có 24 cổng RJ45 và tối đa 8 cổng 10G uplink module) hoàn toàn không đủ cổng quang để đấu nối trực tiếp 9 tủ tầng IDF, 25 switch phòng, hơn 20 server, storage và firewall. Giải pháp: Kiến trúc bổ sung cặp **Switch Phân Phối / ToR Quang Cisco Catalyst 9500-24Y4C** (24 cổng 10G/25G SFP28) làm trung tâm gom cáp quang; hoặc phân cấp rõ ràng luồng uplink từ Switch phòng về Switch phân phối đặt tại các Tủ tầng IDF trước khi gom về MDF.
2. **Giải quyết triệt để mâu thuẫn định tuyến Đông - Tây (East-West Inspection):**  
   Trong bản vẽ cũ, nếu Core Switch đóng vai trò Default Gateway định tuyến L3 cho toàn bộ các VLAN, lưu lượng giữa các phòng ban (Inter-VLAN) sẽ được chuyển mạch trực tiếp tại Core Switch và hoàn toàn **không đi qua Firewall**, dẫn đến việc tường lửa không thể kiểm soát mã độc lây lan nội bộ. Kiến trúc mới áp dụng mô hình **VRF-Lite (Virtual Routing and Forwarding)** hoặc **Firewall-as-Gateway cho các vùng nhạy cảm**: Toàn bộ lưu lượng liên vùng (Cross-Zone) bắt buộc phải đi qua cụm Firewall FortiGate 200F để kiểm tra chính sách bảo mật, chống xâm nhập (IPS) và quét mã độc.
3. **Hiệu chỉnh tính toán công suất nguồn UPS & Máy phát điện:**  
   Tổng công suất tiêu thụ của 3 tủ rack tại MDF A-202 (hơn 20 máy chủ, hệ thống lưu trữ, switch lõi, router, firewall) ước tính đạt từ 10 kW đến 14 kW khi vận hành ổn định. Hai bộ UPS 5kVA (công suất thực khoảng 4.5 kW/bộ) không thể đảm bảo thời gian lưu điện 45–60 phút chỉ với khối ắc quy gắn trong. Bản thiết kế bổ sung:
   - Nâng cấp hệ thống lưu điện MDF lên **2x UPS 15kVA On-line** hoặc cấu hình UPS 10kVA kèm các **Khối ắc quy mở rộng gắn ngoài (External Battery Modules - EBM)**.
   - Bổ sung **Máy phát điện dự phòng Diesel 45kVA** có vỏ chống ồn và **Tủ chuyển nguồn tự động ATS (Automatic Transfer Switch)** với thời gian khởi động cấp điện dưới 15 giây.
4. **Chuẩn hóa thông số phần cứng đúng thực tế kỹ thuật:**
   - Điện thoại VoIP nâng cấp lên **Cisco IP Phone 7841** để hỗ trợ cổng mạng Gigabit 10/100/1000 Mbps passthrough cho máy tính (dòng 7821 chỉ có cổng 10/100 Mbps).
   - Hệ thống lưu trữ `Dell PowerVault ME5024` được làm rõ là **SAN (Block Storage)**; bổ sung máy chủ File Gateway để phục vụ chia sẻ tệp SMB/NFS; dung lượng được tính toán minh bạch giữa dung lượng thô (Raw) và dung lượng khả dụng thực tế (Usable) sau khấu hao RAID 6 và Hot Spare.
   - Bộ điều khiển cửa `ZKTeco InBio-260` là bộ điều khiển 2 cửa (Dual-door), do đó 25 cửa chỉ cần **13 bộ điều khiển** (thay vì 25 bộ gây lãng phí ngân sách). Khóa từ nam châm Maglock sử dụng nguồn 12V/24V DC tập trung kèm ắc quy dự phòng, không sử dụng nguồn PoE.

---

## 2. SƠ ĐỒ TOPOLOGY PHÂN TẦNG VÀ LUỒNG DỮ LIỆU ĐÔNG - TÂY / BẮC - NAM

Hệ thống tuân thủ mô hình mạng 3 lớp chuẩn hóa của Cisco kết hợp vùng kiểm soát Zero Trust:

```
                      [ INTERNET ]
            ISP 1 (Viettel)     ISP 2 (VNPT)
                   │                   │
                   ▼                   ▼
       ┌───────────────────────────────────────────┐
       │   2x WAN Router Cisco Catalyst 8300       │ (Dual Power, IP SLA Tracking +
       │        (Active / Standby Failover)        │  PBR + BFD, Failover < 1s)
       └─────────────────────┬─────────────────────┘
                             │ (2x 10G SFP+)
                             ▼
       ┌───────────────────────────────────────────┐
       │  2x Next-Gen Firewall FortiGate 200F      │ (Active-Passive HA Cluster,
       │    (Threat Protection: 3 - 5 Gbps)        │  DPI, IPS, Antivirus, SSL Proxy)
       └──────────────┬────────────────────────────┘
                      │ (4x 10G LACP Trunk)
                      ▼
       ┌───────────────────────────────────────────┐
       │ 2x Aggregation / Core Switch L3           │
       │ Cisco Catalyst 9500-24Y4C / 9300 Stack    │ (Core Switching: 480 Gbps StackWise)
       │ VRF-Lite: User-VRF, Server-VRF, Mgmt-VRF  │ (Cross-Stack EtherChannel)
       └──────┬─────────────────────────────┬──────┘
              │                             │
    (Cáp quang 10G OS2)            (Cáp quang 10G SFP+)
              │                             │
              ▼                             ▼
   ┌───────────────────────┐   ┌─────────────────────────────┐
   │ 9x Tủ Tầng (IDF)      │   │ Phân Hệ Máy Chủ & Lưu Trữ   │
   │ Switch Gom Tầng       │   │ Data Center MDF (Rack 02-03)│
   │ Patch Panel + UPS 3kVA│   │ ToR Switch 10G SFP+         │
   └──────────┬────────────┘   │ 20x Dell R660/R760 Servers  │
              │                │ Dell ME5024 SAN Storage     │
     (Cáp đồng Cat6A / Quang)  └─────────────────────────────┘
              │
              ▼
   ┌───────────────────────────────────────────────┐
   │ 25 Switch Phòng Ban (Access Layer)            │
   │ 16x C9200L-24P-4X (Phòng 4-15 người, PoE+ 370W)│
   │ 09x C9200CX-8P-2X2G (Phòng nhỏ/họp, Fanless)  │
   └───────┬──────────────┬──────────────┬─────────┘
           │              │              │
           ▼              ▼              ▼
       [ 144 PC ]    [ 25 AP U6 ]   [ 25 IP Phone ]
      Cat6A SFTP      PoE 802.3at      Cisco 7841
```

### Nguyên tắc phân luồng kiểm soát dữ liệu:
- **Luồng dữ liệu Bắc - Nam (North - South):** Traffic từ Client ra ngoài Internet bắt buộc đi từ Access Switch → Switch Tầng (IDF) → Core Switch (User-VRF) → FortiGate NGFW (thực thi NAT, lọc nội dung web, kiểm tra IPS, giải mã SSL) → Router 8300 → Đường truyền ISP.
- **Luồng dữ liệu Đông - Tây (East - West):**
  - *Cùng VLAN (Intra-VLAN):* Chuyển mạch L2 trực tiếp tại switch phòng hoặc switch phân phối tầng nhằm tối ưu độ trễ. Thiết lập tính năng **Port Isolation** trên switch truy cập để ngăn chặn các máy trạm trong cùng phòng lây lan mã độc ngang hàng.
  - *Khác VLAN / Khác Vùng (Inter-Zone):* Ví dụ từ VLAN Người dùng (User Zone) truy cập vào VLAN Máy chủ (Server Farm Zone) hoặc VLAN Quản trị (Management Zone), lưu lượng được Core Switch định tuyến sang cổng Firewall FortiGate thông qua các sub-interface/VRF riêng biệt. Firewall phân tích sâu gói tin theo bộ luật (Firewall Rules) và kiểm tra lỗ hổng IPS trước khi cho phép dữ liệu đi vào Server Farm.

---

## 3. DANH MỤC THIẾT BỊ PHẦN CỨNG ĐÃ ĐƯỢC CHUẨN HÓA (BILL OF MATERIALS)

| STT | Nhóm thiết bị | Model phần cứng chuẩn hóa | Số lượng | Vị trí lắp đặt | Vai trò kỹ thuật chính |
|:---:|:---|:---|:---:|:---|:---|
| **1** | **Router WAN Biên** | Cisco Catalyst C8300-1N1S-4T2X (Dual Power) | 02 bộ | MDF A-202 (Rack 01) | Định tuyến biên, Dual ISP Viettel & VNPT, IP SLA + PBR + BFD |
| **2** | **Tường Lửa NGFW** | Fortinet FortiGate FG-200F (Gói FortiGuard Enterprise) | 02 bộ | MDF A-202 (Rack 01) | Cụm HA Active-Passive, lọc gói tin chuyên sâu, kiểm soát truy cập liên vùng |
| **3** | **Switch Gom Quang Core/Agg** | Cisco Catalyst C9500-24Y4C (hoặc C9300X-24Y) | 02 bộ | MDF A-202 (Rack 01) | Trung tâm đấu nối cáp quang 10G/25G SFP28 từ 9 IDF, Server Farm và Firewall |
| **4** | **Switch Lõi Định Tuyến L3** | Cisco Catalyst C9300-24T-E (StackWise-480) | 02 bộ | MDF A-202 (Rack 01) | Cụm Gateway L3, thực thi VRF-Lite, phân phối bảng định tuyến nội bộ |
| **5** | **Switch Phân Phối Tầng (IDF)** | Cisco Catalyst C9200L-24T-4X | 09 bộ | 9 Tủ IDF các tầng | Gom đường truyền từ các switch phòng của tầng trước khi uplink quang về Core |
| **6** | **Switch Phòng Lớn (4-15 PC)**| Cisco Catalyst C9200L-24P-4X (24 Cổng PoE+ 370W) | 16 bộ | 16 Phòng ban làm việc | Cấp cổng mạng 1Gbps, cấp nguồn PoE+ cho AP, Camera, Điện thoại, Cửa |
| **7** | **Switch Phòng Nhỏ & Họp** | Cisco Catalyst C9200CX-8P-2X2G (Không quạt 0dB) | 09 bộ | 9 Phòng nhỏ, Sảnh, Căng tin| Hoạt động không tiếng ồn, cấp nguồn PoE+ 125W, 2 cổng Uplink 10G |
| **8** | **Bộ Phát Sóng Wi-Fi** | Ubiquiti UniFi U6-Pro (Wi-Fi 6 AX5400) | 25 bộ | Trần 25 phòng ban | Phủ sóng Wi-Fi 6, hỗ trợ WPA3-Enterprise 802.1X, tải khuyến nghị 50–80 clients/AP |
| **9** | **Máy Chủ Dịch Vụ 1U** | Dell PowerEdge R660 (2x Xeon Silver 4410Y, 64-128GB) | 12 máy | MDF A-202 (Rack 01, 02, 03)| Hạ tầng AD/DNS/DHCP, NMS Zabbix, RADIUS/NPS, PBX, ACS, Syslog, HRM, Fin |
| **10** | **Máy Chủ Dịch Vụ 2U** | Dell PowerEdge R760 (2x Xeon Gold 5416S, 128-256GB) | 08 máy | MDF A-202 (Rack 02, 03) | Ứng dụng Web, Mail, Database Cluster, Backup Veeam, SIEM Wazuh, Kubernetes |
| **11** | **Hệ Thống Lưu Trữ SAN** | Dell PowerVault ME5024 (24 khay SFF, Dual Controller) | 02 mảng | MDF A-202 (Rack 02) | Mảng lưu trữ khối (Block Storage) kết nối iSCSI 10G/25G cho Server Farm |
| **12** | **Hệ Thống Lưu Trữ 3.5" (VMS)**| Dell PowerVault ME5012 (12 khay LFF, Dual Controller) | 01 mảng | MDF A-202 (Rack 03) | Mảng lưu trữ dung lượng cao (HDD 3.5") phục vụ kho lưu trữ Camera 30 ngày |
| **13** | **Điện Thoại IP Doanh Nghiệp** | Cisco IP Phone 7841 (4 Lines, 2 Cổng Gigabit RJ45) | 25 máy | Bàn lễ tân, trưởng phòng | Đàm thoại VoIP chất lượng cao, cổng passthrough máy tính chuẩn 1000 Mbps |
| **14** | **Camera An Ninh IP** | Hikvision DS-2CD2143G2-I (4MP Dome, IK10, H.265+) | 25 camera | Góc trần 25 phòng & MDF | Giám sát hình ảnh an ninh, nhận diện người/xe AcuSense, cô lập trong VLAN 12 |
| **15** | **Bộ Điều Khiển Cửa Ra Vào**| ZKTeco InBio-260 (Hỗ trợ quản lý 2 cửa độc lập) | 13 bộ | Hộp kỹ thuật cửa các phòng| Điều khiển đóng mở cửa, nhận diện thẻ Mifare và vân tay sinh trắc học |
| **16** | **Đầu Đọc Thẻ & Vân Tay** | ZKTeco FR1500-A (Đầu đọc vân tay + thẻ Mifare DESFire)| 25 bộ | Cửa ra vào 25 phòng | Xác thực 2 yếu tố, mã hóa thẻ chuẩn AES-128 bit chống sao chép |
| **17** | **Khóa Nam Châm Điện Từ** | Khóa từ Magnetic Lock 600lbs + Bộ gá ZL + Nút Exit | 25 bộ | Khung cửa 25 phòng | Lực giữ 280 kg, cấp nguồn DC 12V/24V, tích hợp rơ-le ngắt điện tự động khi báo cháy |
| **18** | **Bộ Nguồn Dự Phòng Khóa Cửa**| Bộ nguồn kiểm soát cửa 12V DC 5A kèm ắc quy 7Ah | 13 bộ | Đặt cạnh bộ InBio-260 | Nuôi nguồn khóa từ độc lập, duy trì khóa cửa hoạt động từ 4–6 tiếng khi mất điện |
| **19** | **Tủ Rack Trung Tâm MDF** | APC NetShelter SX 42U (600 x 1070mm, Cửa lưới 80%) | 03 tủ | MDF A-202 | Chứa Core Network, Server Farm, SAN Storage và hệ thống an ninh |
| **20** | **Tủ Rack Phân Phối Tầng** | APC NetShelter SX 24U / 42U Tiêu chuẩn công nghiệp | 09 tủ | 9 Phòng kỹ thuật tầng (IDF)| Chứa Patch Panel tầng, Switch phân phối tầng, ODF quang tầng và UPS tầng |
| **21** | **Bộ Lưu Điện Online MDF** | APC Smart-UPS SRT 10kVA On-Line (kèm 2 module EBM) | 02 hệ | MDF A-202 (Đáy Rack 01-03) | Đảm bảo nguồn điện sạch 230V liên tục 15-20 phút cho toàn bộ tải MDF |
| **22** | **Bộ Lưu Điện Online IDF** | APC Smart-UPS SRT 3000VA On-Line 230V | 09 bộ | 9 Tủ IDF các tầng | Duy trì hoạt động cho Switch tầng, Camera PoE và Wi-Fi trong 30-40 phút |
| **23** | **Máy Phát Điện & Tủ ATS** | Máy phát điện Diesel 45kVA Cummins/Yanmar + ATS 100A | 01 cụm | Khu kỹ thuật mặt đất | Tự động khởi động phát điện trong vòng 15 giây sau khi mất lưới điện tòa nhà |
| **24** | **Điều Hòa Không Khí Chính Xác**| Stulz / Vertiv Liebert CRAC 20kW (N+1 In-Row) | 02 máy | MDF A-202 | Duy trì nhiệt độ phòng máy chủ 20°C ± 2°C, độ ẩm 45% - 55% liên tục 24/7 |
| **25** | **Chữa Cháy Khí Sạch** | Hệ thống chữa cháy khí sạch FM-200 (hoặc Novec 1230) | 01 cụm | MDF A-202 | Dập tắt đám cháy trong vòng 10 giây mà không làm hỏng vi mạch điện tử |
| **26** | **Chống Sét Lan Truyền (SPD)** | Thiết bị cắt lọc sét OBO Bettermann Type 1+2 | 01 bộ | Tủ điện tổng MDF A-202 | Triệt tiêu xung điện áp sét lan truyền từ lưới điện công nghiệp |
| **27** | **Màn Hình Video Wall SOC** | Cụm 4 màn hình 55-inch Ultra Narrow Bezel 0.88mm | 01 cụm | NOC/SOC A-404 | Hiển thị 24/7 SIEM Wazuh, Grafana Metrics, Zabbix NMS và bản đồ tấn công |
| **28** | **Máy Trạm Chuyên Dụng SOC** | Dell Precision 3680 Tower Workstation (Dual Monitor) | 04 bộ | NOC/SOC A-404 | Phục vụ 4 kỹ sư tác chiến điều tra chứng cứ số, săn lùng mối đe dọa (Threat Hunting) |
| **29** | **Máy Trạm Người Dùng** | Dell OptiPlex Plus 7020 SFF (Core i7, 16-32GB RAM) | 144 máy | 25 Phòng ban nghiệp vụ | Cấu hình máy trạm đồng bộ, TPM 2.0, cổng mạng Gigabit Ethernet bọc kim |
| **30** | **Máy In Mạng Doanh Nghiệp** | HP LaserJet Enterprise M507dn | 20 máy | Các phòng ban nghiệp vụ | In ấn văn bản an toàn qua xác thực mã PIN / HTTPS IPP |
| **31** | **Cáp Mạng Đồng Nhánh** | CommScope NetConnect Cat6A SFTP (Chống nhiễu kép) | Toàn bộ | Đi âm trần, sàn toàn tòa nhà| Đạt băng thông 500 MHz, hỗ trợ 10 Gbps khoảng cách 100m, an toàn nhiệt PoE+ |
| **32** | **Cáp Quang Trục Chính** | Cáp quang Single-Mode OS2 12-core & 24-core | Toàn bộ | Trục đứng liên tầng & liên tòa | Băng thông không giới hạn, kết nối 9 IDF về MDF A-202 |
| **33** | **Tủ Phân Phối Quang ODF** | Khung ODF tập trung 144-Port LC Duplex Single-Mode | 01 tủ | MDF A-202 (Đỉnh Rack 01) | Đấu nối và quản lý tập trung toàn bộ 108 core cáp quang từ 9 IDF và 2 nhà mạng ISP |

---

## 4. PHÂN TÍCH CHI TIẾT TỪNG PHÂN HỆ PHẦN CỨNG

### 4.1. Phân Hệ Biên & Kết Nối WAN (Edge Routing): Cisco Catalyst 8300

- **Model cấu hình:** `Cisco Catalyst C8300-1N1S-4T2X` trang bị sẵn 2 nguồn dự phòng tháo lắp nóng (Dual Hot-swap AC Power Supplies).
- **Vị trí & Số lượng:** 02 thiết bị chạy song hành tại Rack 01 (MDF A-202).
- **Cơ chế định tuyến Dual ISP thực tế cho doanh nghiệp:**  
  Thay vì giả định cấu hình BGP yêu cầu dải IP Public độc lập (Provider-Independent - PI) và số hiệu mạng ASN (vốn rất tốn kém và chỉ cấp cho các ISP hoặc tập đoàn rất lớn), hệ thống áp dụng giải pháp định tuyến kết hợp:
  - **IP SLA Tracking kết hợp Policy-Based Routing (PBR):** Router liên tục gửi các gói tin thăm dò (ICMP / DNS Probes) đến các máy chủ DNS công cộng tin cậy (8.8.8.8, 1.1.1.1) qua cả 2 cổng WAN Viettel và VNPT.
  - **Tăng tốc chuyển mạch dự phòng bằng BFD (Bidirectional Forwarding Detection):** Giúp phát hiện sự cố đứt tuyến cáp ngầm hoặc mất kết nối viễn thông chỉ trong vòng **300 – 500 mili-giây** (dưới 1 giây), ngay lập tức chuyển hướng toàn bộ lưu lượng sang đường truyền còn lại mà các phiên làm việc của nhân viên không bị đứt đoạn.
- **Ưu điểm:** Nền tảng vi xử lý Cisco QFP 2.0 xử lý định tuyến và mã hóa VPN bằng phần cứng; khả năng kích hoạt Cisco SD-WAN linh hoạt khi mở rộng chi nhánh.
- **Nhược điểm & Giới hạn:** Đòi hỏi kỹ sư quản trị nắm vững câu lệnh Cisco IOS-XE; chi phí đầu tư ban đầu cao.
- **Tại sao doanh nghiệp nên dùng:** Tránh hoàn toàn tình trạng mất kết nối Internet làm ngưng trệ quy trình kinh doanh, hóa đơn điện tử, giao dịch khách hàng và kết nối dịch vụ đám mây.

---

### 4.2. Phân Hệ Tường Lửa Thế Hệ Mới (NGFW): Fortinet FortiGate 200F / 120G HA Cluster

- **Model cấu hình:** Cụm 02 thiết bị `Fortinet FortiGate FG-200F` (hoặc thế hệ mới `FG-120G` trang bị chip bảo mật SP5) kèm gói bản quyền an ninh FortiGuard Enterprise Protection.
- **Vị trí:** U34-U36 Rack 01 (MDF A-202).
- **Nguyên lý tính toán hiệu năng (Sizing Throughput):**  
  *Lưu ý kỹ thuật then chốt:* Không thể sử dụng thông số Firewall thô (27 Gbps) để tính toán cho hệ thống doanh nghiệp. Khi kích hoạt toàn bộ các tính năng bảo vệ an ninh bao gồm: Kiểm soát ứng dụng (Application Control), Hệ thống chống xâm nhập (IPS), Quét virus luồng dữ liệu (Antivirus) và Giải mã kiểm tra nội dung SSL/TLS (Deep SSL Inspection), thông lượng thực tế (**Threat Protection Throughput**) của FortiGate 200F đạt mức ổn định từ **3.0 Gbps đến 5.0 Gbps**.
  - Băng thông Internet doanh nghiệp thuê từ 2 ISP: $2 \times 500\text{ Mbps} = 1\text{ Gbps}$.
  - Lưu lượng liên vùng Đông - Tây (Inter-Zone) cần kiểm tra: ~2.0 Gbps.
  - Tổng lưu lượng cần soi chiếu: ~3.0 Gbps $\rightarrow$ Nằm hoàn toàn trong ngưỡng an toàn tải của FortiGate 200F (hoạt động ở mức 60% – 75% công suất thiết kế).
- **Cơ chế sẵn sàng cao (High Availability):** 2 thiết bị kết nối trực tiếp qua 2 cổng chuyên dụng HA1 và HA2 bằng cáp quang DAC 10G, hoạt động ở chế độ **Active-Passive Stateful Failover**. Mọi bảng phiên làm việc (session table), bảng định tuyến, trạng thái kết nối VPN đều được đồng bộ thời gian thực theo từng micro-giây.
- **Ưu điểm:** Chip phần cứng chuyên biệt Network Processor (NP6XLite/SP5) và Content Processor (CP9) giải phóng tải cho CPU chính; kho tri thức mối đe dọa cập nhật liên tục từ FortiGuard Labs.
- **Nhược điểm:** Phí duy trì bản quyền bảo mật hàng năm (OPEX); cần quản lý việc phân phối chứng chỉ bảo mật nội bộ (Root CA) xuống các máy trạm khi bật tính năng Deep SSL Inspection.
- **Tại sao doanh nghiệp nên dùng:** Là chốt chặn quyết định ngăn chặn các cuộc tấn công tống tiền bằng Ransomware, các cuộc tấn công mã độc qua đường duyệt web và chặn đứng hành vi dò quét mạng nội bộ của kẻ tấn công.

---

### 4.3. Phân Hệ Chuyển Mạch Lõi & Gom Tầng (Core / Aggregation Switching): Cisco Catalyst 9500 & 9300

Để khắc phục hoàn toàn điểm thắt cổ chai về số lượng cổng quang trong thiết kế ban đầu, phân hệ Core được chia thành 2 khối phối hợp nhịp nhàng:

1. **Khối Gom Cáp Quang Phân Phối (Aggregation Layer):**  
   - Bổ sung cặp Switch quang chuyên dụng `Cisco Catalyst C9500-24Y4C` (24 cổng quang SFP28 10G/25G và 4 cổng 40G/100G).
   - Đóng vai trò là đầu mối tiếp nhận toàn bộ các đường cáp quang 10G SFP+ từ:
     - 09 đường cáp quang uplink từ 9 Tủ tầng IDF.
     - 04 đường cáp quang uplink từ cụm ToR Switch Server Farm (Rack 02 & 03).
     - 04 đường kết nối LACP 10G sang cụm Tường lửa FortiGate 200F.
     - 02 đường kết nối sang Router Cisco 8300.
2. **Khối Chuyển Mạch Lõi & Định Tuyến (Core Layer):**  
   - Cặp `Cisco Catalyst C9300-24T-E` ghép nối bằng cáp chuyên dụng **StackWise-480** (băng thông kênh ghép đạt 480 Gbps).
   - Đảm nhận xử lý bảng định tuyến L3 cho toàn bộ hệ thống thông qua công nghệ **VRF-Lite**:
     - *VRF-User:* Quản lý các dải mạng văn phòng.
     - *VRF-Server:* Quản lý dải mạng Server Farm.
     - *VRF-Mgmt:* Quản lý dải mạng quản trị thiết bị độc lập.
- **Ưu điểm:** Loại bỏ hoàn toàn việc thiếu hụt cổng quang kết nối; cấu trúc phân tầng rõ ràng giúp mở rộng thêm tầng lầu hoặc tòa nhà mới một cách dễ dàng.
- **Nhược điểm:** Chi phí đầu tư trang thiết bị switch quang ban đầu cao hơn so với giải pháp gộp cổng.
- **Tại sao doanh nghiệp nên dùng:** Đảm bảo tốc độ truyền tải nội bộ không có độ trễ, triệt tiêu nguy cơ nghẽn cổ chai khi hàng trăm thiết bị cùng truyền dữ liệu video, sao lưu dữ liệu và truy cập cơ sở dữ liệu.

---

### 4.4. Phân Hệ Tủ Tầng (IDF) & Chuyển Mạch Truy Cập (Access Switching): Cisco Catalyst C9200L & C9200CX

#### Kiến trúc Tủ Mạng Phân Phối Tầng (IDF 42U - 9 Tủ):
Mỗi tầng của Tòa nhà A (5 tầng) và Tòa nhà B (4 tầng) được bố trí 1 Tủ mạng tầng tiêu chuẩn (IDF). Trong mỗi tủ IDF bao gồm:
- **01 Switch Phân Phối Tầng (IDF Distribution Switch):** Model `Cisco Catalyst C9200L-24T-4X` (24 cổng RJ45 1G, 4 cổng quang 10G SFP+).
- **Hộp Phối Quang Tầng (ODF 12-Port LC):** Kết nối sợi quang Single-Mode OS2 dẫn trực tiếp về Tủ trung tâm MDF A-202.
- **Thanh Đấu Nối Cáp Mạng (Patch Panel Cat6A Modular 24/48 Cổng):** Cố định toàn bộ các đầu cáp đồng đi từ các phòng ban trong tầng về.
- **Bộ Lưu Điện Online Tủ Tầng:** `APC Smart-UPS SRT 3000VA On-Line` cấp điện dự phòng cho switch tầng và các thiết bị ngoại vi trong 30-40 phút khi mất điện.

#### Cấu hình Switch tại các phòng ban:
- **Tại 16 Phòng Quy Mô Lớn (4 - 15 nhân sự):** Trang bị `Cisco Catalyst C9200L-24P-4X`.
  - Cung cấp 24 cổng mạng 1Gbps, công suất cấp nguồn PoE+ tổng đạt **370 Watts**.
  - Nhu cầu tiêu thụ PoE thực tế của mỗi phòng: 1 AP (~13W) + 1 Camera (~7W) + 1 Điện thoại VoIP (~4W) = **~24 Watts** (chỉ chiếm ~6.5% tải nguồn PoE của switch). Khoảng trống công suất 346W còn lại cho phép mở rộng thêm hàng loạt thiết bị IoT, màn hình hiển thị hoặc máy chấm công trong tương lai.
  - Uplink: Cáp đồng Cat6A 1Gbps hoặc cáp quang 10G về Switch Phân Phối Tầng (IDF).
- **Tại 09 Phòng Nhỏ / Phòng Họp / Phòng Giám Đốc (1 - 3 nhân sự):** Trang bị `Cisco Catalyst C9200CX-8P-2X2G`.
  - Thiết kế **hoàn toàn không quạt (Fanless 0dB)**, mang lại không gian yên tĩnh tuyệt đối cho phòng điều hành và phòng họp cấp cao.
  - Cung cấp 8 cổng mạng PoE+ công suất 125W, tích hợp sẵn 2 cổng quang 10G SFP+.
- **Ưu điểm:** Tính năng bảo mật phần cứng Port Security (khóa cổng khi cắm máy lạ), DHCP Snooping (chống cấp phát IP giả mạo), Dynamic ARP Inspection (chống tấn công giả mạo Man-in-the-middle).
- **Nhược điểm:** Dòng C9200L sử dụng bộ nguồn cố định (Fixed Power Supply), cần kết nối với nguồn điện UPS ổn định tại phòng hoặc tủ tầng.

---

### 4.5. Phân Hệ Mạng Không Dây Doanh Nghiệp (Enterprise WLAN): Ubiquiti UniFi U6-Pro

- **Model thiết bị:** `Ubiquiti UniFi U6-Pro` (Chuẩn Wi-Fi 6 802.11ax, 4x4 MU-MIMO).
- **Vị trí & Số lượng:** 25 bộ gắn trần tại 25 phòng ban (Phòng MDF A-202 không triển khai Wi-Fi để đảm bảo an ninh vật lý cao nhất).
- **Thông số kỹ thuật chuẩn hóa:**
  - Tiêu chuẩn cấp nguồn: **PoE chuẩn 802.3at (PoE+)**, công suất tiêu thụ tối đa của mỗi AP là **13 Watts**.
  - Khả năng chịu tải thực tế khuyến nghị cho môi trường doanh nghiệp: Từ **50 đến 80 thiết bị (Clients) trên mỗi AP** để đảm bảo tốc độ mượt mà cho các ứng dụng hội nghị truyền hình, thoại không dây (tránh các con số lý thuyết 250 - 300 clients vốn chỉ áp dụng cho thiết bị IoT truyền dữ liệu ngắt quãng).
- **Quy hoạch phát sóng (SSID & An ninh):**
  - *SSID 1: "Enterprise-Internal"* $\rightarrow$ Áp dụng mã hóa an ninh cấp doanh nghiệp **WPA3-Enterprise / 802.1X**. Nhân viên đăng nhập bằng tài khoản Active Directory / RADIUS cá nhân. Khi nhân sự nghỉ việc, việc vô hiệu hóa tài khoản AD lập tức chặn quyền truy cập Wi-Fi mà không cần đổi mật khẩu chung của toàn công ty. Gán động vào VLAN phòng ban tương ứng.
  - *SSID 2: "Enterprise-Guest"* $\rightarrow$ Áp dụng xác thực Cổng chào (Captive Portal), gán vào **VLAN 100/110 (Guest Zone)**. Bật chế độ cách ly người dùng (Client Isolation), cấm hoàn toàn việc quét IP hoặc nhìn thấy các thiết bị khác trong mạng nội bộ, chỉ được phép đi Internet và giới hạn băng thông 10 Mbps/client.
- **Ưu điểm:** Chi phí đầu tư hợp lý, phần mềm điều khiển UniFi Network Application cài đặt tập trung trên máy chủ ảo hóa không tốn phí bản quyền định kỳ.
- **Nhược điểm:** Cần duy trì máy chủ Controller hoạt động liên tục để thu thập nhật ký kết nối và vận hành cổng Captive Portal.
- **Tại sao doanh nghiệp nên dùng:** Loại bỏ triệt để nguy cơ lộ mật khẩu Wi-Fi nội bộ; phủ sóng tốc độ cao không điểm chết trong toàn bộ không gian làm việc.

---

### 4.6. Phân Hệ Máy Chủ Dịch Vụ Data Center: Dell PowerEdge R660 & R760

Toàn bộ hệ thống máy chủ được chuẩn hóa trên nền tảng **Dell PowerEdge thế hệ 16G (Intel Xeon Scalable Gen 4)** đảm bảo vòng đời hỗ trợ linh kiện chính hãng đến năm 2032+:

#### Danh mục phân bổ chi tiết:
1. **Dell PowerEdge R660 (Dạng Rack 1U - 12 máy chủ):**
   - *Cấu hình:* 2x Intel Xeon Silver 4410Y (24 Cores/48 Threads), 64GB – 128GB ECC DDR5 RAM, 2x 480GB SSD NVMe RAID 1 (chứa OS), Dual 10G SFP+ Network Card, Dual Hot-plug Redundant PSU 800W Titanium.
   - *Phân bổ nghiệp vụ chuyên biệt:*
     - 02 máy chủ cụm **Active Directory Domain Services (DC01 & DC02)** chạy song hành đồng bộ dữ liệu.
     - 02 máy chủ **DNS nội bộ & DHCP Failover** phân giải tên miền và cấp phát IP tự động.
     - 02 máy chủ xác thực danh tính **FreeRADIUS / Microsoft NPS** (thay thế phần mềm Cisco ACS đã EoL).
     - 01 máy chủ **NMS Zabbix & Grafana** giám sát hiệu năng mạng và hạ tầng.
     - 01 máy chủ điều khiển tổng đài **IP-PBX SIP Server**.
     - 01 máy chủ an ninh vật lý **ZKBioSecurity Access Control Server**.
     - 01 máy chủ lưu trữ nhật ký tập trung **Syslog & NTP Server**.
     - 01 máy chủ cổng nhảy an toàn **Bastion / Jump Host** (tích hợp MFA FIDO2).
     - 01 máy chủ môi trường thử nghiệm **Dev / Staging / UAT Server**.
2. **Dell PowerEdge R760 (Dạng Rack 2U - 08 máy chủ):**
   - *Cấu hình:* 2x Intel Xeon Gold 5416S (32 Cores/64 Threads), 128GB – 256GB ECC DDR5 RAM, 4 cổng quang 10G/25G SFP28, Dual Redundant PSU 1100W Titanium.
   - *Phân bổ ứng dụng trọng yếu:*
     - 01 máy chủ Cổng thông tin doanh nghiệp & REST API (**Web Portal**).
     - 01 máy chủ Thư điện tử doanh nghiệp (**Mail Server Zimbra / Exchange**).
     - 02 máy chủ Cơ sở dữ liệu cốt lõi (**Database Cluster: PostgreSQL / MySQL Master-Slave**).
     - 03 máy chủ cụm điện toán đám mây nội bộ **Kubernetes Production Cluster (3 nodes control-plane & worker)**.
     - 01 máy chủ sao lưu và phục hồi dữ liệu chuyên nghiệp (**Veeam Backup & Replication Server**).
- **Ưu điểm:** Bộ nhớ RAM DDR5 hỗ trợ công nghệ tự sửa lỗi nâng cao (Advanced ECC); chip điều khiển từ xa **iDRAC9 Enterprise** chuyên dụng với cổng mạng riêng cho phép cài đặt, bật/tắt và xử lý sự cố từ xa mà không cần cắm màn hình bàn phím vật lý; tính năng Silicon Root of Trust chống lại mã độc can thiệp phần cứng.
- **Nhược điểm:** Yêu cầu phòng máy lạnh chính xác và nguồn điện ổn định.

---

### 4.7. Phân Hệ Lưu Trữ Khối Chuyên Dụng (SAN Block Storage): Dell PowerVault ME5024 / ME5012

Cần định nghĩa chuẩn xác: Dòng sản phẩm **Dell PowerVault ME5 bản chất là hệ thống lưu trữ khối chuyên dụng SAN (Block Storage)** kết nối qua giao thức iSCSI 10G/25G hoặc SAS 12G, **không phải là thiết bị chia sẻ file NAS**. Để phục vụ tính năng File Server (SMBv3/NFS), hệ thống sử dụng các máy chủ Windows Server / Linux làm File Gateway kết nối trực tiếp vào các LUN của ME5.

#### Phân bổ 2 mảng lưu trữ chuyên biệt:
1. **Mảng SAN 01 - Hiệu năng cao cho Ứng dụng & Dữ liệu:** `Dell PowerVault ME5024` (Khung 2U gồm 24 khay ổ 2.5" SFF, Dual Active-Active Controllers).
   - Trang bị 24 ổ đĩa chuẩn **Enterprise SAS SSD / HDD 2.4TB**.
   - Cấu hình mảng đĩa: **RAID 6 (20 Data Drives + 2 Parity Drives + 2 Hot Spare Drives)**.
   - *Tính toán dung lượng khả dụng (Usable Capacity):*
     $$\text{Dung lượng thô (Raw)} = 24 \times 2.4\text{ TB} = 57.6\text{ TB}$$
     $$\text{Dung lượng khả dụng sau RAID 6 \& Spare} = (24 - 2 - 2) \times 2.4\text{ TB} \times 0.909 \approx \mathbf{43.6\text{ TB Usable}}$$
   - Phục vụ cơ sở dữ liệu (Database), máy ảo ảo hóa (VMware/Proxmox datastores) và File Server dùng chung.
2. **Mảng SAN 02 - Dung lượng lớn cho Camera An Ninh & Backup:** `Dell PowerVault ME5012` (Khung 2U gồm 12 khay ổ 3.5" LFF dung lượng lớn).
   - Trang bị 12 ổ cứng chuẩn **Enterprise SAS 3.5" dung lượng 10TB/ổ**.
   - Cấu hình: **RAID 6 (8 Data + 2 Parity + 2 Hot Spare)** $\rightarrow$ Đạt dung lượng khả dụng xấp xỉ **~58 TB Usable**.
   - *Tính toán nhu cầu lưu trữ Camera thực tế:*
     - 25 camera IP độ phân giải 4MP, chuẩn nén tiên tiến H.265+, tốc độ khung hình 20 fps, bitrate trung bình đạt $3\text{ Mbps/camera}$.
     - Dung lượng ghi hình 24/7 trong 1 ngày cho 25 camera:
       $$25 \times \frac{3\text{ Mbps} \times 3600 \times 24}{8 \times 1024 \times 1024} \approx 772\text{ GB/ngày}$$
     - Dung lượng lưu trữ trọn vẹn trong 30 ngày:
       $$772\text{ GB} \times 30\text{ ngày} \approx \mathbf{23.16\text{ TB}}$$
     - Như vậy, mảng lưu trữ 58 TB Usable đáp ứng hoàn toàn nhu cầu lưu camera 30 ngày (~24 TB), phần dung lượng còn lại (~34 TB) được dành riêng cho các bản Snapshot sao lưu dữ liệu hệ thống cục bộ.

---

### 4.8. Phân Hệ Thoại Doanh Nghiệp (VoIP / UC): Cisco IP Phone 7841 & Tổng Đài SIP

- **Model thiết bị chuẩn hóa:** `Cisco IP Phone 7841` (Nâng cấp từ mã 7821 cũ).
- **Lý do kỹ thuật bắt buộc:** Model 7821 chỉ hỗ trợ 2 cổng Fast Ethernet (10/100 Mbps). Nếu cắm máy tính làm việc nối tiếp qua điện thoại 7821, tốc độ mạng của máy tính sẽ bị nghẽn ở mức 100 Mbps (giảm 90% hiệu năng đường truyền 1Gbps). Model **Cisco IP Phone 7841 tích hợp 2 cổng mạng chuẩn Gigabit Ethernet (10/100/1000 Mbps)**, cho phép máy tính passthrough giữ nguyên tốc độ 1 Gbps ổn định.
- **Cơ chế phân tách mạng & Ưu tiên dịch vụ:**
  - Switch tự động nhận diện thiết bị qua giao thức **LLDP-MED / CDP** và đẩy lưu lượng thoại vào **Voice VLAN 200 (Tòa A) và Voice VLAN 210 (Tòa B)**.
  - Gán nhãn ưu tiên chất lượng dịch vụ QoS phần cứng: **DSCP EF (Expedited Forwarding - Giá trị 46)** và CoS 5, bảo đảm gói tin âm thanh luôn được xử lý trước các gói tin tải file thông thường.
- **Ưu điểm:** Âm thanh đàm thoại băng rộng HD Voice trong trẻo; cấp nguồn trực tiếp qua cáp mạng (PoE 802.3af Class 1, tiêu thụ chỉ ~3.8W); đàm thoại nội bộ giữa các phòng ban và giữa 2 tòa nhà hoàn toàn miễn phí.

---

### 4.9. Phân Hệ Giám Sát Hình Ảnh (CCTV / VMS): Hikvision DS-2CD2143G2-I

- **Model thiết bị:** `Hikvision DS-2CD2143G2-I` (Camera IP dạng bán cầu Dome, độ phân giải 4.0 Megapixel).
- **Số lượng & Bố trí:** 25 camera tại các phòng làm việc, hành lang kỹ thuật và 01 camera giám sát an ninh trực diện tủ rack tại phòng MDF A-202.
- **Tiêu chuẩn công nghiệp & An ninh:**
  - Vỏ hợp kim đạt chuẩn chống va đập cơ học **IK10** và chống bụi nước **IP67**.
  - Thuật toán học sâu **AcuSense** phân biệt chính xác chuyển động của con người, hạn chế tối đa báo động giả.
  - **Chính sách an ninh mạng cô lập tuyệt đối:** Toàn bộ 25 camera được xếp vào **VLAN 12 (Infra / Camera Zone)** với dải IP riêng `192.168.12.0/24`. Cụm tường lửa FortiGate chặn toàn bộ chiều truy cập từ VLAN Camera đi ra ngoài Internet để ngăn chặn triệt để nguy cơ camera bị tấn công botnet hoặc gửi dữ liệu video trái phép ra máy chủ bên ngoài. Chỉ có máy chủ quản lý ghi hình Hikvision iVMS / VMS Server tại Data Center mới được phép truy xuất luồng RTSP của camera.

---

### 4.10. Phân Hệ Kiểm Soát Ra Vào Vật Lý (Physical Access Control): ZKTeco InBio + Mifare DESFire EV3

#### Chuẩn hóa thiết bị và số lượng:
- **Bộ điều khiển trung tâm (Controller):** Sử dụng **13 bộ ZKTeco InBio-260**. Do mỗi bộ InBio-260 hỗ trợ quản lý **2 cửa độc lập**, việc bố trí 13 bộ hoàn toàn đáp ứng trọn vẹn cho 25 cửa ra vào của toàn hệ thống (tiết kiệm chi phí đầu tư 12 bộ điều khiển so với bản vẽ ban đầu).
- **Công nghệ thẻ chống sao chép:** Đầu đọc thẻ hỗ trợ chuẩn thẻ thông minh **Mifare DESFire EV3** (tần số 13.56 MHz). Thẻ áp dụng thuật toán mã hóa tối tân **AES-128 bit**, tạo khóa ngẫu nhiên trong mỗi phiên quẹt thẻ, ngăn chặn hoàn toàn việc nhân bản thẻ từ trái phép bằng các thiết bị sao chép trôi nổi.
- **Khóa điện từ & Nguồn điện độc lập:**
  - Sử dụng **Khóa nam châm điện từ (Magnetic Lock) lực hút 600 lbs (~280 kg)**.
  - *Lưu ý kỹ thuật:* Khóa từ sử dụng nguồn điện một chiều **12V DC hoặc 24V DC**, hoàn toàn không dùng nguồn PoE từ switch. Hệ thống trang bị 13 tủ cấp nguồn chuyên dụng cho kiểm soát cửa (12V DC 5A có ắc quy dự phòng 12V 7Ah đi kèm). Khi tòa nhà mất điện lưới, khóa cửa vẫn giữ chặt trong vòng 4–6 tiếng.
  - **Liên động hệ thống Báo cháy (Fire Alarm Integration):** Đấu nối dây tín hiệu rơ-le tiếp điểm khô (Dry Contact) trực tiếp từ tủ báo cháy trung tâm của tòa nhà vào bộ điều khiển InBio. Khi có tín hiệu báo cháy khẩn cấp, rơ-le lập tức ngắt điện nguồn cấp cho khóa từ, toàn bộ các cửa tự động nhả mở hoàn toàn để nhân viên thoát nạn nhanh chóng theo quy chuẩn an toàn PCCC.

---

### 4.11. Hạ Tầng Nguồn Điện, UPS, Máy Phát & Cơ Điện Phòng Máy (DC Facilities)

Hạ tầng phòng máy chủ trung tâm MDF A-202 được thiết kế đạt tiêu chuẩn **TIA-942 Rated-2 / Tier-2**:

```
                    [ LƯỚI ĐIỆN TÒA NHÀ 3 PHA 380V ]
                                   │
                                   ▼
                       ┌───────────────────────┐
                       │  TỦ CẮT LỌC SÉT (SPD) │ (Type 1+2 OBO Bettermann,
                       │  VÀ TIẾP ĐỊA ĐỘC LẬP  │  Điện trở đất R < 1 Ohm)
                       └───────────┬───────────┘
                                   │
                                   ▼
 [ MÁY PHÁT ĐIỆN 45kVA ] ───► [ TỦ CHUYỂN NGUỒN TỰ ĐỘNG (ATS) ]
 (Diesel Cummins, bồn dầu 24h)     │ (Chuyển mạch tự động < 15 giây)
                                   ▼
                       ┌───────────────────────┐
                       │   TỦ ĐIỆN PHÂN PHỐI   │
                       │    DATA CENTER PDU    │
                       └─────┬───────────┬─────┘
                             │           │
                    (Nhánh A)│           │(Nhánh B)
                             ▼           ▼
        ┌───────────────────────┐     ┌───────────────────────┐
        │  UPS 1: APC SRT 10kVA │     │  UPS 2: APC SRT 10kVA │
        │  (Kèm 2 Khối Pin EBM) │     │  (Kèm 2 Khối Pin EBM) │
        └────────────┬──────────┘     └───────────┬───────────┘
                     │                            │
                     ▼                            ▼
        ┌───────────────────────┐     ┌───────────────────────┐
        │ Thanh Nguồn PDU A     │     │ Thanh Nguồn PDU B     │
        │ (Rack 01, 02, 03)     │     │ (Rack 01, 02, 03)     │
        └────────────┬──────────┘     └───────────┬───────────┘
                     │                            │
                     └─────────────┬──────────────┘
                                   ▼
                    [ THIẾT BỊ 2 NGUỒN (DUAL PSU) ]
                    (Server Dell, Core Switch, Firewall)
```

1. **Hệ Thống Lưu Điện Online (UPS):**  
   Trang bị 02 bộ lưu điện công nghệ chuyển đổi kép trực tuyến **APC Smart-UPS SRT 10kVA On-Line 230V**, mỗi bộ được gắn thêm 02 khối ắc quy mở rộng (External Battery Modules - EBM).
   - Khả năng cấp nguồn tải 10 kW liên tục trong **15 – 20 phút**, loại bỏ hoàn toàn độ trễ chuyển mạch (0ms).
   - Tích hợp card giám sát môi trường và quản trị mạng SNMP NMC3, tự động gửi lệnh tắt an toàn (Graceful Shutdown) các máy ảo qua mạng nếu nguồn điện cạn kiệt.
2. **Máy Phát Điện Dự Phòng & Tủ ATS:**  
   - 01 Máy phát điện Diesel công suất **45 kVA** có thùng cách âm, bình nhiên liệu đảm bảo vận hành liên tục 24 giờ.
   - Tủ chuyển đổi nguồn tự động **ATS 100A**: Khi mất điện lưới, ATS phát lệnh nổ máy phát và đóng điện hòa vào hệ thống trong vòng **10 – 15 giây** (nằm trọn vẹn trong khoảng thời gian 20 phút bảo vệ của UPS).
3. **Hệ Thống Cắt Lọc Sét & Tiếp Địa:**  
   - Tủ cắt lọc sét đa cấp **OBO Bettermann Type 1+2** lắp đặt ngay trước tủ điện phân phối của Data Center, triệt tiêu các xung sét lan truyền lên tới 50 kA.
   - Hệ thống bãi cọc tiếp địa viễn thông riêng biệt đạt điện trở đất **$R < 1\ \Omega$** (Ohm), toàn bộ vỏ tủ rack và máng cáp được liên kết thanh đồng tiếp địa bảo vệ an toàn cho thiết bị và con người.
4. **Hệ Thống Làm Mát Chính Xác & Chữa Cháy Khí Sạch:**  
   - 02 Cụm điều hòa chính xác **In-Row CRAC công suất 20kW** chạy cấu hình dự phòng luân phiên $N+1$, duy trì nhiệt độ buồng máy ổn định ở mức $20^\circ\text{C} \pm 2^\circ\text{C}$ và độ ẩm $50\% \pm 5\%$.
   - Hệ thống chữa cháy tự động bằng khí sạch **FM-200 (hoặc Novec 1230)**: Cảm biến khói quang học và cảm biến nhiệt độ cảnh báo 2 vùng độc lập; khi kích hoạt xả khí, đám cháy được dập tắt trong 10 giây bằng cơ chế hấp thụ nhiệt mà không gây ngạt cho người và hoàn toàn không làm hư hỏng thiết bị vi mạch điện tử.

---

### 4.12. Trung Tâm Điều Hành & Giám Sát An Ninh (NOC / SOC A-404)

Bố trí tại phòng biệt lập A-404 với cấp độ an ninh nghiêm ngặt (VLAN 44 Restricted):
- **Cụm Màn Hình Video Wall 4x 55-inch:** Sử dụng tấm nền IPS chuyên dụng hoạt động liên tục 24/7/365, viền ghép siêu mỏng 0.88mm. Phân chia 4 vùng hiển thị thông tin trọng yếu:
  - *Màn hình 1:* Bảng điều khiển phân tích sự kiện an ninh **SIEM Wazuh** (cảnh báo tấn công brute-force, thay đổi file hệ thống, phát hiện mã độc).
  - *Màn hình 2:* Đồ thị băng thông mạng, trạng thái cổng mạng 26 switch và đường truyền WAN từ **Zabbix NMS**.
  - *Màn hình 3:* Chỉ số tải tài nguyên CPU/RAM/IOPS của cụm Kubernetes và cụm Cơ sở dữ liệu từ **Grafana Dashboard**.
  - *Màn hình 4:* Bản đồ cảnh báo đe dọa địa lý và nhật ký chặn truy cập thời gian thực từ **Tường lửa FortiGate**.
- **Máy Trạm Kỹ Sư Tác Chiến (04 bộ):** `Dell Precision 3680 Tower Workstation` (Intel Core i7-14700, 32GB RAM DDR5, card đồ họa rời NVIDIA RTX, trang bị 2 màn hình 27-inch độ phân giải 2K sắc nét).
- **Bộ Quản Trị Ngoại Băng Out-of-Band (OOB Console Server):** Thiết bị Terminal Server 8 cổng kết nối trực tiếp vào cổng Console vật lý của Router, Firewall, Core Switch và UPS, cho phép kỹ sư khôi phục cấu hình hệ thống ngay cả khi mạng nội bộ bị sự cố sập hoàn toàn.

---

### 4.13. Máy Trạm Người Dùng & Máy In Mạng: Dell OptiPlex & HP LaserJet Enterprise

- **Máy trạm nhân viên (144 bộ):** `Dell OptiPlex Plus 7020 SFF` (Intel Core i7-14700, 16GB - 32GB DDR5 RAM, 512GB NVMe SSD, card mạng Gigabit RJ45 bọc kim chống nhiễu). Tích hợp chip mã hóa phần cứng TPM 2.0, hỗ trợ khởi động an toàn Secure Boot và cài đặt phần mềm phòng chống mã độc thế hệ mới (EDR).
- **Máy in mạng bảo mật (20 máy):** `HP LaserJet Enterprise M507dn`.
  - Kết nối mạng dây Ethernet trực tiếp vào switch phòng.
  - Áp dụng tính năng in ấn an toàn **HP Sure Start** và **Secure PIN Printing**: Nhân viên gửi lệnh in từ máy tính, tài liệu được lưu tạm trên bộ nhớ mã hóa của máy in; nhân viên phải đến trước máy in nhập đúng mã PIN cá nhân thì tài liệu mới được in ra, tránh rò rỉ thông tin hợp đồng và tài chính quan trọng.

---

### 4.14. Hệ Thống Cáp Cấu Trúc, Patch Panel & Phân Phối Quang ODF

- **Cáp đồng ngang (Horizontal Cabling):** Toàn bộ sử dụng cáp **Cat6A SFTP (chống nhiễu từng cặp bọc lá kim loại và bọc lưới đồng tổng)**. Băng thông kiểm định 500 MHz, đảm bảo tốc độ truyền tải 10 Gigabit đến từng bàn làm việc, đồng thời tiết diện lõi đồng 23 AWG bảo đảm nhiệt độ đường dây luôn an toàn khi truyền tải dòng điện PoE+ liên tục.
- **Cáp quang trục chính (Backbone Cabling):** Các tuyến cáp quang **Single-Mode OS2 12-core và 24-core** đi trong ống luồn chống cháy nối từ 9 tủ tầng IDF về MDF A-202.
- **Tủ Phân Phối Quang Tập Trung (MDF ODF):**  
  *Khắc phục lỗi thiếu cổng quang:* 9 Tủ tầng IDF với cáp quang 12-core sẽ tạo ra tổng cộng $9 \times 12 = 108\text{ sợi quang}$. Do đó, tại MDF A-202 trang bị **Khung phân phối quang tập trung ODF 144-Port chuẩn LC Duplex** dạng trượt 4U tại đỉnh Rack 01, kết hợp khay quản lý sợi quang giúp việc đấu nối gọn gàng, tránh gập gãy suy hao quang học.

---

## 5. ĐÁNH GIÁ ĐIỂM YẾU KIẾN TRÚC, KẾ HOẠCH DỰ PHÒNG THẢM HỌA (DR) & RTO/RPO

### 5.1. Nhận định trung thực về các "Điểm Lỗi Đơn" (SPOF) còn tồn tại và giải pháp kiểm soát
Trong thực tế kỹ thuật, không có một hệ thống nào trong phạm vi một công trình đơn lẻ có thể tuyên bố "hoàn toàn không có điểm lỗi đơn". Báo cáo làm rõ các rủi ro còn tồn tại và biện pháp giảm thiểu:

1. **Rủi ro vật lý phòng máy chủ tập trung (Single Data Center Facility):** Toàn bộ thiết bị máy chủ cốt lõi đặt tại phòng A-202. Nếu xảy ra sự cố thảm họa vật lý (cháy nổ lớn tòa nhà, ngập lụt thiên tai), hệ thống sẽ bị gián đoạn.  
   $\rightarrow$ *Biện pháp giảm thiểu:* Tăng cường hệ thống chữa cháy khí sạch FM-200, cửa thép chống cháy 120 phút và thiết lập kênh sao lưu dữ liệu mã hóa ra ngoài (Offsite Cloud Backup).
2. **Cụm Kubernetes Control-Plane:** Đảm bảo tối thiểu **3 node Master vật lý** để chạy cơ chế biểu quyết etcd quorum (tránh dùng 1 node master gây mất điều khiển toàn cụm container).
3. **Cơ sở dữ liệu tập trung (Database):** Cấu hình mô hình **PostgreSQL / MySQL Master-Slave Streaming Replication** giữa 2 máy chủ vật lý độc lập; tự động chuyển đổi dự phòng (Automatic Failover) bằng Patroni hoặc Orchestrator.
4. **Bộ nguồn switch truy cập (C9200L Fixed PSU):** Dòng switch C9200L có nguồn gắn liền; nếu hỏng nguồn phải thay cả switch.  
   $\rightarrow$ *Biện pháp giảm thiểu:* Dự phòng sẵn 02 switch C9200L cấu hình mẫu (Cold Spare) trong kho IT; khi có sự cố, kỹ thuật viên Helpdesk có thể thay thế trong vòng 30 phút.

### 5.2. Khắc phục triệt để quy tắc sao lưu dữ liệu 3-2-1
Trong bản thiết kế trước, cả 3 mảng lưu trữ đều đặt chung trong phòng A-202, nếu phòng máy chủ gặp thảm họa thì toàn bộ các bản sao lưu đều bị tiêu hủy. Kiến trúc chuẩn hóa thực thi nghiêm ngặt mô hình **Sao lưu 3-2-1**:
- **3 bản sao dữ liệu:** 1 bản đang chạy sản xuất + 1 bản sao lưu nhanh cục bộ (Local Snapshot trên SAN ME5012) + 1 bản sao lưu nén lưu trữ dài hạn.
- **2 loại môi trường lưu trữ khác nhau:** Lưu trữ trên ổ đĩa từ (Disk-based SAN) và lưu trữ trên cụm máy chủ sao lưu phân tán (Object Storage).
- **1 bản sao lưu Offsite tách rời hoàn toàn:** Hàng đêm, phần mềm Veeam Backup & Replication tự động mã hóa AES-256 toàn bộ các bản sao lưu của Database, Mail, File và đẩy đồng bộ qua đường truyền Internet riêng biệt lên dịch vụ lưu trữ đám mây **Cloud Object Storage (AWS S3 Glacier / Wasabi / Trung tâm dữ liệu dự phòng thứ 2)**.

### 5.3. Mục tiêu phục hồi sau sự cố (RTO & RPO) cho từng phân hệ:

| Phân hệ nghiệp vụ | RPO (Mức độ mất mát dữ liệu chấp nhận được) | RTO (Thời gian khôi phục hoạt động tối đa) | Phương thức kỹ thuật bảo vệ |
|:---|:---:|:---:|:---|
| **Cơ sở dữ liệu giao dịch cốt lõi** | **$\le 5$ phút** | **$\le 15$ phút** | Đồng bộ hóa dữ liệu liên tục (Replication), sao lưu Transaction Logs mỗi 15 phút |
| **Hệ thống xác thực AD / DNS / DHCP** | **0 phút** (Zero Data Loss) | **$\le 1$ phút** | Chạy song hành cụm 2 máy chủ Active-Active phân tán |
| **Hệ thống Email doanh nghiệp** | $\le 1$ giờ | $\le 2$ giờ | Snapshot máy ảo hàng ngày + Đồng bộ hóa Mailbox |
| **Hệ thống File dữ liệu phòng ban** | $\le 24$ giờ | $\le 2$ giờ | Snapshot SAN ME5024 mỗi 4 tiếng + Backup Offsite hàng đêm |
| **Dữ liệu Camera an ninh** | $\le 1$ ngày | $\le 4$ giờ | Lưu trữ trực tiếp trên RAID 6 cục bộ |

---

## 6. SO SÁNH TỔNG CHI PHÍ SỞ HỮU (TCO) & HOÀN VỐN ĐẦU TƯ (ROI) 5 NĂM

### 6.1. Bảng số liệu dự toán TCO thực tế trong vòng đời 5 năm (Đơn vị tính: Triệu VNĐ)

| Hạng mục chi phí | Phương án 1: Đầu tư chuẩn hóa theo thiết kế (Cisco / Fortinet / Dell / APC) | Phương án 2: Chắp vá thiết bị phổ thông giá rẻ (Switch unmanaged, Router dân dụng, PC tự ráp) | Phân tích chênh lệch & Lý do kỹ thuật |
|:---|:---:|:---:|:---|
| **1. Chi phí mua sắm thiết bị ban đầu (CAPEX)** | **3.850** | 1.450 | Phương án 1 cao hơn 2.400 triệu do toàn bộ là thiết bị công nghiệp chịu tải lớn và có dự phòng kép. |
| **2. Bản quyền phần mềm & License 5 năm** | **650** | 120 | Chi phí cập nhật tri thức bảo mật FortiGuard NGFW, bản quyền VMware/Veeam, Windows Server. |
| **3. Chi phí thay thế phần cứng hư hỏng vặt** | **80** | 450 | Thiết bị chuẩn doanh nghiệp có tỷ lệ hỏng hóc dưới 1%/năm; thiết bị phổ thông hỏng tụ nguồn, quạt sau 1-2 năm. |
| **4. Chi phí nhân sự IT vận hành & bảo trì** | **900** *(1-2 kỹ sư nhờ hệ thống tự động)* | 1.800 *(Cần 3-4 kỹ sư đi sửa lỗi thủ công)* | Tiết kiệm 50% chi phí nhân sự IT nhờ quản trị tập trung (iDRAC, NMS, Controller, Ansible). |
| **5. Ước tính thiệt hại tài chính do mạng gián đoạn (Downtime)** | **150** *(Độ sẵn sàng 99.99% ~ 52 phút gián đoạn/năm)* | 1.650 *(Độ sẵn sàng 97% ~ 11 ngày gián đoạn lũy kế/năm)* | 144 nhân sự ngồi chờ mạng, chậm trễ hợp đồng, phạt tiến độ dự án công nghệ thông tin. |
| **TỔNG CHI PHÍ SỞ HỮU (TCO) SAU 5 NĂM** | **5.630** | **5.470** | **Tổng chi phí sở hữu tương đương nhau, nhưng Phương án 1 đem lại hệ thống an toàn tuyệt đối và năng suất làm việc vượt trội!** |

### 6.2. Các giá trị hoàn vốn đầu tư (ROI) định tính và định lượng:
1. **Bảo toàn năng suất lao động:** Với 144 nhân viên văn phòng, việc loại bỏ tình trạng mạng chập chờn giúp tiết kiệm trung bình 15 phút lãng phí mỗi ngày cho mỗi nhân sự, tương đương với **hơn 8.500 giờ làm việc hữu ích mỗi năm** được thu hồi cho doanh nghiệp.
2. **Bảo hiểm rủi ro an ninh thông tin:** Ngăn chặn một vụ việc rò rỉ cơ sở dữ liệu khách hàng hoặc bị mã độc Ransomware tống tiền; chi phí chuộc dữ liệu và xử lý khủng hoảng truyền thông của một vụ tấn công mạng trung bình hiện nay dao động từ 1 đến 5 tỷ VNĐ.
3. **Giá trị thương hiệu & Khả năng trúng thầu:** Hạ tầng mạng và an ninh đạt chuẩn tạo điều kiện thuận lợi để doanh nghiệp vượt qua các vòng đánh giá an ninh thông tin độc lập của các đối tác lớn trong và ngoài nước (chứng chỉ ISO/IEC 27001, PCI-DSS).

---

## 7. KẾT LUẬN & LỘ TRÌNH TRIỂN KHAI

Hồ sơ thiết kế phần cứng sau khi hiệu chỉnh toàn diện đã khắc phục triệt để các hạn chế kỹ thuật:
- **Giải quyết triệt để số lượng cổng quang** bằng việc phân tầng cụ thể với switch phân phối và khung phối quang ODF 144 core.
- **Minh bạch hóa luồng định tuyến an ninh Đông - Tây**, bảo đảm mọi lưu lượng nhạy cảm đều chịu sự kiểm soát nghiêm ngặt của tường lửa thế hệ mới FortiGate.
- **Tính toán thực tế hạ tầng cơ điện phòng máy**, đảm bảo nguồn điện kép và hệ thống máy phát điện sẵn sàng cho mọi kịch bản mất điện lưới diện rộng.
- **Tối ưu hóa ngân sách đầu tư**, loại bỏ các thiết bị dư thừa (chuẩn hóa 13 bộ điều khiển cửa thay vì 25 bộ, điều chỉnh dung lượng lưu trữ camera chính xác, nâng cấp điện thoại VoIP lên Gigabit tránh nghẽn mạng).

Hệ thống sau khi hiệu chỉnh tạo nên một nền tảng công nghệ thông tin **vững chắc, bảo mật đa lớp, vận hành ổn định trong 7–10 năm tới** và đáp ứng hoàn hảo các mục tiêu tăng trưởng kinh doanh của doanh nghiệp.
