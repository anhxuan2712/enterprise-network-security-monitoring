# Enterprise Network Security & Monitoring Architecture

Tài liệu toàn tập về Phân tích Thiết kế Hạ tầng An toàn Mạng Doanh nghiệp và Hệ thống Giám sát & Quản lý Nhật ký Tập trung (SIEM / SOC / Syslog / SNMP / NetFlow).

---

## 📚 MỤC LỤC BỘ TÀI LIỆU (7 CHUYÊN ĐỀ)

1. [01_Network_Fundamentals.md](file:///d:/School/PTTK_AnToanMang/01_Network_Fundamentals.md)
   - **Nền tảng mạng căn bản**: Mô hình OSI 7 tầng, Mô hình TCP/IP 4 tầng, Ánh xạ dữ liệu giám sát an ninh vào từng tầng, Địa chỉ IPv4, Dải mạng riêng RFC 1918, Kỹ thuật CIDR và phân bổ mạng con VLSM.
2. [02_Network_Infrastructure.md](file:///d:/School/PTTK_AnToanMang/02_Network_Infrastructure.md)
   - **Hạ tầng thiết bị mạng**: Vai trò trong hệ thống giám sát của Router (CoPP, uRPF), Switch L2/L3 (CAM Table, Port Security, DHCP Snooping, SPAN Port Mirroring), Next-Gen Firewall (State Table, DPI, App-ID, IPS) và Enterprise Server (Auditd, Event ID, Hardening).
3. [03_Network_Segmentation.md](file:///d:/School/PTTK_AnToanMang/03_Network_Segmentation.md)
   - **Phân đoạn mạng & Thiết kế VLAN**: Phân loại LAN/WAN/MAN, Chuẩn gắn thẻ 802.1Q, Inter-VLAN Routing, và kiến trúc phân đoạn mạng cô lập **Management (VLAN 99)** & **Monitoring / SIEM (VLAN 100)** kèm bộ Extended ACL bảo vệ.
4. [04_Core_Network_Services.md](file:///d:/School/PTTK_AnToanMang/04_Core_Network_Services.md)
   - **Các dịch vụ mạng cốt lõi**: DHCP & kỹ thuật điều tra số (Audit Trail), DNS & các nguy cơ DNS Tunneling / C2 Domain, NAT/PAT & giải pháp đối chiếu log IP thực, NTP (Đồng bộ thời gian chuẩn - Yêu cầu sống còn của hệ thống SIEM).
5. [05_Routing_Protocols.md](file:///d:/School/PTTK_AnToanMang/05_Routing_Protocols.md)
   - **Giao thức định tuyến**: Định tuyến tĩnh (Static Routing, Default Route, Floating Static Route), Giao thức Link-State OSPF (Dijkstra, Router ID, Area 0, Xác thực MD5), Bảng tra cứu Administrative Distance (AD), và giám sát định tuyến qua Syslog.
6. [06_Network_Security.md](file:///d:/School/PTTK_AnToanMang/06_Network_Security.md)
   - **An toàn mạng & Kịch bản tấn công**: Firewall & ACLs (Standard vs Extended), Zone-based Policy, VPN (Site-to-Site IPsec & SSL-VPN), IDS/IPS (Suricata/Snort - Passive SPAN vs Active Inline), Bảng nhận diện các dạng tấn công phổ biến (Brute-Force, ARP Poisoning, MAC Flooding, DoS/DDoS, Port Scan, Thay đổi config).
7. [07_Monitoring_And_Logging.md](file:///d:/School/PTTK_AnToanMang/07_Monitoring_And_Logging.md)
   - **TRỌNG TÂM ĐỀ TÀI - Giám sát & Quản lý Nhật ký tập trung**: Giao thức SNMP (v1/v2c/v3 USM, MIB/OID, Polling vs Traps/Informs), Chuẩn Syslog (RFC 5424, Priority, 8 mức Severity, Facilities), Flexible NetFlow / IPFIX, Kiến trúc SIEM Pipeline (ELK Stack / Wazuh / Graylog), Tập luật tương quan sự kiện (Correlation Rules) và Kịch bản kiểm thử giả lập tấn công (Hydra, Nmap, hping3).

---

> [!TIP]
> Tất cả các tài liệu đều được định dạng chi tiết, có sơ đồ chu trình (Mermaid / ASCII), bảng so sánh đa chiều, và mẫu cấu hình dòng lệnh (Cisco IOS, Linux Rsyslog, Wazuh XML Rules).
