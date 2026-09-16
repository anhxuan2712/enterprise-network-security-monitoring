# CHUYÊN ĐỀ 2: NETWORK INFRASTRUCTURE (HẠ TẦNG MẠNG DOANH NGHIỆP)

> **Mục tiêu**: Phân tích kiến trúc phần cứng, nguyên lý hoạt động và vai trò an ninh của 4 trụ cột hạ tầng mạng: **Router (Bộ định tuyến)**, **Switch (Bộ chuyển mạch)**, **Next-Gen Firewall (Tường lửa thế hệ mới)** và **Enterprise Server (Máy chủ doanh nghiệp)** trong bức tranh tổng thể về Giám sát An toàn Thông tin (SIEM/SOC).

---

## 1. TỔNG QUAN VAI TRÒ CÁC THIẾT BỊ TRONG HỆ THỐNG GIÁM SÁT

| Thiết bị | Vị trí Tầng OSI | Vai trò Chức năng Chính | Vai trò Trong Hệ Thống Giám Sát & An Ninh (SIEM) | Nguồn Dữ liệu Log & Telemetry |
| :--- | :--- | :--- | :--- | :--- |
| **Router** | Layer 3 (Network) | Định tuyến liên mạng, chia tách Broadcast domain, NAT, Inter-VLAN | Cảnh báo tấn công Control Plane (CoPP), phát hiện giả mạo IP nguồn (uRPF), thống kê lưu lượng mạng diện rộng | • Syslog (Link state, Auth, Config)<br>• SNMP (CPU/RAM, Traffic OID)<br>• Flexible NetFlow (Flow records) |
| **Switch (L2/L3)** | Layer 2 & Layer 3 | Kết nối điểm cuối (End-hosts), tạo VLAN, chống vòng lặp (STP), gộp cổng (LACP) | Phát hiện tấn công nội bộ (MAC Flooding, ARP Spoofing, Rogue DHCP), nhân bản lưu lượng phục vụ phân tích mạng (SPAN Port) | • Syslog (Port-sec violation, STP change)<br>• SNMP (Port status, Error counters)<br>• SPAN/RSPAN Mirroring sang IDS/IPS |
| **Next-Gen Firewall** | Layer 3 - Layer 7 | Phân tách Security Zones, kiểm soát truy cập Stateful, kiểm tra sâu gói tin (DPI), chống xâm nhập (IPS) | Nguồn sinh log giá trị nhất: Ghi nhận vi phạm chính sách, chặn mã độc, cảnh báo khai thác lỗ hổng, phiên VPN truy cập | • Traffic Logs (Chấp nhận / Từ chối)<br>• Threat / IPS Alert Logs<br>• User Authentication & VPN Logs<br>• Syslog RFC 5424 / CEF Format |
| **Enterprise Server** | Layer 4 - Layer 7 | Chạy dịch vụ mạng (DNS, DHCP, AD, Web, DB) và lưu trữ dữ liệu tập trung | Vừa là mục tiêu bảo vệ trọng yếu, vừa là nền tảng máy chủ vận hành SIEM, Logstash, Wazuh Manager, Elasticsearch | • Linux Auditd / Syslog (`auth.log`, `syslog`)<br>• Windows Event Logs (Security Event ID)<br>• Application Logs (Web access/error, DB) |

---

## 2. BỘ ĐỊNH TUYẾN (ROUTER)

```
+-----------------------------------------------------------------------------------+
|                            KIẾN TRÚC PHẦN CỨNG ROUTER                             |
+-----------------------------------------------------------------------------------+
|  CONTROL PLANE (Xử lý thông minh)  <--->  DATA PLANE (Chuyển tiếp tốc độ cao)     |
|  - Chạy trên CPU chính                   - Chip chuyên dụng ASIC / NPU            |
|  - Trao đổi OSPF, BGP, SSH, SNMP         - Tra cứu bảng FIB (Forwarding Base)     |
|  - Xây dựng bảng định tuyến (RIB)        - Bảng quan hệ lân cận (Adjacency Table) |
|  - Được bảo vệ bởi cơ chế CoPP           - Chuyển mạch Cisco Express Forwarding   |
+-----------------------------------------------------------------------------------+
```

### 2.1. Chu Trình Xử Lý Gói Tin Tại Router
1. **Bóc tách Header L2 (De-encapsulation)**: Kiểm tra MAC đích có khớp với MAC cổng Router không; kiểm tra lỗi Frame Check Sequence (FCS).
2. **Kiểm tra IP Header & TTL**: Giảm `TTL` đi 1. Nếu `TTL = 0`, hủy gói và gửi bản tin `ICMP Time Exceeded` về máy nguồn (chống vòng lặp vô tận).
3. **Tra cứu Bảng định tuyến (Longest Prefix Match)**: So khớp IP đích với bảng FIB/RIB, ưu tiên tuyến có Subnet Mask dài nhất (cụ thể nhất).
4. **Đóng gói lại Header L2 mới (Re-encapsulation)**: Tra bảng ARP tìm địa chỉ MAC của Next-Hop, gán MAC nguồn mới (MAC cổng ra của router) và MAC đích mới (MAC Next-Hop).
5. **Chuyển tiếp (Forwarding)**: Đẩy gói tin qua cổng mạng tương ứng.

### 2.2. Cơ Chế Bảo Mật & Giám Sát Cốt Lõi Trên Router
- **Control Plane Policing (CoPP)**: Sử dụng chính sách Modular QoS CLI (MQC) để phân loại và giới hạn tốc độ lưu lượng gửi đến CPU Router, ngăn chặn triệt để tấn công DoS làm treo thiết bị mạng.
- **Unicast Reverse Path Forwarding (uRPF)**: Chống giả mạo IP nguồn (IP Spoofing) bằng cách kiểm tra đường quay về của gói tin trong bảng định tuyến trước khi chấp nhận chuyển tiếp.
- **Flexible NetFlow (FnF) / IPFIX**: Trích xuất 7 thông số nhận dạng luồng (`Source IP`, `Dest IP`, `Source Port`, `Dest Port`, `L4 Protocol`, `Ingress Interface`, `ToS/CoS`) gửi về Flow Collector.

---

## 3. BỘ CHUYỂN MẠCH (SWITCH LAYER 2 & LAYER 3)

### 3.1. Nguyên Lý Học & Chuyển Tiếp Địa Chỉ MAC
Switch hoạt động dựa trên **CAM Table (Content Addressable Memory Table)** thông qua 4 cơ chế:
- **Learning**: Đọc địa chỉ MAC nguồn của frame đi vào cổng để lưu cặp `(MAC Address, Ingress Port, VLAN ID)` vào bảng CAM.
- **Forwarding**: Nếu MAC đích đã có trong CAM Table, switch chỉ chuyển tiếp frame ra đúng cổng tương ứng (Unicast).
- **Flooding**: Nếu MAC đích chưa có trong CAM Table (Unknown Unicast) hoặc là địa chỉ Broadcast (`FF:FF:FF:FF:FF:FF`), switch đẩy frame ra tất cả các cổng trong cùng VLAN (ngoại trừ cổng nhận vào).
- **Filtering**: Không đẩy frame sang các cổng thuộc VLAN khác hoặc cổng bị chặn bởi Spanning Tree.

### 3.2. Cơ Chế Bảo Mật Tầng 2 (Layer 2 Security)
- **Port Security**: Giới hạn số lượng MAC address trên một cổng vật lý, tự động `shutdown` cổng khi phát hiện vi phạm (ngăn chặn MAC Flooding).
- **DHCP Snooping**: Phân loại cổng `Trusted` (kết nối máy chủ DHCP thật) và cổng `Untrusted` (kết nối người dùng), ngăn chặn tấn công dựng máy chủ DHCP giả mạo (Rogue DHCP Server).
- **Dynamic ARP Inspection (DAI)**: Sử dụng cơ sở dữ liệu của DHCP Snooping để xác thực tính hợp lệ của các gói tin ARP Reply, triệt tiêu tấn công giả mạo ARP (ARP Poisoning / Man-In-The-Middle).
- **SPAN / RSPAN (Switch Port Analyzer)**: Nhân bản toàn bộ lưu lượng của một hoặc nhiều cổng gửi về cổng kết nối với máy chủ IDS/IPS (như Snort/Suricata) phục vụ phân tích chuyên sâu.

```
+-----------------------------------------------------------------------------+
|               CƠ CHẾ MIRRORING LƯU LƯỢNG MẠNG (SPAN PORT)                   |
+-----------------------------------------------------------------------------+
|  [ User Port Gi/0/1 ] ---> Traffic bình thường ---> [ Core Switch / Router ] |
|            |                                                                |
|            +---> (SPAN Mirroring bản sao dữ liệu) ---> [ IDS/IPS Sensor ]    |
|                                                        (Suricata / Snort)   |
|                                                               |             |
|                                                    Đẩy cảnh báo Syslog/JSON |
|                                                               v             |
|                                                       [ SIEM SERVER ]       |
+-----------------------------------------------------------------------------+
```

---

## 4. TƯỜNG LỬA THẾ HỆ MỚI (NEXT-GENERATION FIREWALL - NGFW)

### 4.1. Kiến Trúc Kiểm Soát Trạng Thái & Vùng An Ninh (Security Zones)
- Tường lửa phân chia mạng thành các vùng logic có mức độ tin cậy khác nhau: **WAN/Untrust (Internet)**, **DMZ (Máy chủ công khai)**, **LAN/Trust (Nội bộ)**, **Management/Monitoring (Giám sát)**.
- Mọi kết nối đi qua Firewall đều được kiểm soát bởi **State Table (Bảng phiên)**, theo dõi chu trình bắt tay 3 bước TCP (SYN $\rightarrow$ SYN-ACK $\rightarrow$ ACK) và chỉ cho phép gói tin phản hồi hợp lệ quay trở lại.

### 4.2. Các Công Nghệ Cốt Lõi Trên NGFW
- **Deep Packet Inspection (DPI)**: Kiểm tra toàn bộ phần tải dữ liệu (Payload) đến Layer 7 thay vì chỉ xem xét IP/Port ở Layer 3/4.
- **Application Identification (App-ID)**: Nhận diện chính xác ứng dụng (VD: Facebook, TeamViewer, BitTorrent, SSH) bất kể ứng dụng đó sử dụng cổng chuẩn hay cổng ngụy trang.
- **Intrusion Prevention System (IPS)**: Quét luồng dữ liệu theo thời gian thực để đối chiếu với hàng chục nghìn chữ ký tấn công (Signatures), tự động Drop gói tin và gửi cảnh báo về SIEM.
- **SSL/TLS Decryption**: Giải mã lưu lượng HTTPS để kiểm tra mã độc ẩn giấu bên trong các luồng dữ liệu được mã hóa.

---

## 5. MÁY CHỦ DOANH NGHIỆP (ENTERPRISE SERVER)

### 5.1. Vai Trò Máy Chủ Trong Đề Tài An Toàn & Giám Sát Mạng
1. **Nền tảng Vận hành Hệ thống Giám sát**:
   - Máy chủ Linux (Ubuntu Server / Rocky Linux) triển khai **Wazuh Manager / Indexer**, **Elasticsearch / Logstash / Kibana**, **rsyslog-ng**.
   - Máy chủ Windows Server triển khai **Active Directory Domain Services (AD DS)** quản lý xác thực tập trung.
2. **Đối tượng Giám sát Trọng yếu**:
   - Ghi nhận nhật ký đăng nhập thất bại liên tiếp (Event ID 4625 trên Windows hoặc `Failed password` trong `/var/log/auth.log` trên Linux).
   - Giám sát tính toàn vẹn của tệp cấu hình hệ thống (FIM - File Integrity Monitoring trên các file `/etc/passwd`, `/etc/shadow`, `C:\Windows\System32\drivers\etc\hosts`).
   - Cung cấp dữ liệu SNMP Host MIB để theo dõi tỷ lệ chiếm dụng CPU, RAM, Disk I/O.
