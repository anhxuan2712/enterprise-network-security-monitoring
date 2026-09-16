# Enterprise Network Security & Monitoring Architecture

Tài liệu toàn tập về Phân tích Thiết kế Hạ tầng An toàn Mạng Doanh nghiệp và Hệ thống Giám sát & Quản lý Nhật ký Tập trung (SIEM / SOC / Syslog / SNMP / NetFlow).

---

## 📚 MỤC LỤC BỘ GIÁO TRÌNH HỌC TẬP (7 CHUYÊN ĐỀ)

Tất cả các tài liệu chi tiết đã được lưu trữ trong thư mục [Documents/](file:///d:/School/PTTK_AnToanMang/Documents):

1. [01_Network_Fundamentals.md](file:///d:/School/PTTK_AnToanMang/Documents/01_Network_Fundamentals.md)
   - **Nền tảng mạng & Kiến trúc giao thức**: Mô hình OSI 7 tầng, Mô hình TCP/IP 4 tầng, Cấu trúc chi tiết TCP/UDP Header, Ánh xạ dữ liệu giám sát an ninh vào từng tầng, Địa chỉ IPv4, Phép toán Bitwise AND, Dải mạng riêng RFC 1918, Bảng tính nhẩm Subnetting nhanh và quy hoạch mạng con VLSM.
2. [02_Network_Infrastructure.md](file:///d:/School/PTTK_AnToanMang/Documents/02_Network_Infrastructure.md)
   - **Kiến trúc hạ tầng thiết bị mạng**: Vai trò trong hệ thống giám sát của Router (Control/Data Plane, CoPP, uRPF), Switch L2/L3 (CAM vs TCAM, Port Security, DHCP Snooping, DAI, SPAN Port Mirroring), Next-Gen Firewall (State Table, DPI, App-ID, IPS) và Enterprise Server (Auditd, Windows Event IDs 4624/4625/4720/1102, Hardening).
3. [03_Network_Segmentation.md](file:///d:/School/PTTK_AnToanMang/Documents/03_Network_Segmentation.md)
   - **Phân đoạn mạng & Thiết kế VLAN giám sát**: Phân loại LAN/WAN/MAN, Chuẩn gắn thẻ 802.1Q (4 Bytes Tag), Inter-VLAN Routing (Router-on-a-Stick), và kiến trúc phân đoạn mạng cô lập **Management OOB (VLAN 99)** & **Monitoring / SIEM (VLAN 100)** kèm bộ Extended ACL bảo vệ.
4. [04_Core_Network_Services.md](file:///d:/School/PTTK_AnToanMang/Documents/04_Core_Network_Services.md)
   - **Các dịch vụ mạng cốt lõi & Giám sát an ninh**: DHCP & kỹ thuật điều tra số (Audit Trail), DNS & các nguy cơ DNS Tunneling / C2 Domain DGA, NAT/PAT & giải pháp đối chiếu log IP thực, NTP (Đồng bộ thời gian chuẩn - Yêu cầu sống còn của hệ thống SIEM).
5. [05_Routing_Protocols.md](file:///d:/School/PTTK_AnToanMang/Documents/05_Routing_Protocols.md)
   - **Giao thức định tuyến trong doanh nghiệp**: Bảng tra cứu Administrative Distance (AD), Longest Prefix Match, Định tuyến tĩnh (Static Routing, Default Route, Floating Static Route), Giao thức Link-State OSPF (Dijkstra, 7 trạng thái láng giềng, DR/BDR, Xác thực MD5), và giám sát định tuyến qua Syslog / Rule Wazuh SIEM.
6. [06_Network_Security.md](file:///d:/School/PTTK_AnToanMang/Documents/06_Network_Security.md)
   - **An toàn mạng & Các kịch bản tấn công**: Firewall & ACLs (Standard vs Extended), Zone-based Policy, VPN (Site-to-Site IPsec & SSL-VPN), IDS/IPS (Suricata/Snort - Passive SPAN vs Active Inline, EVE JSON), Bảng nhận diện các dạng tấn công phổ biến (Brute-Force, ARP Poisoning, MAC Flooding, DoS/DDoS, Port Scan, Thay đổi config).
7. [07_Monitoring_And_Logging.md](file:///d:/School/PTTK_AnToanMang/Documents/07_Monitoring_And_Logging.md)
   - **TRỌNG TÂM ĐỀ TÀI - Hệ thống Giám sát & Quản lý Nhật ký tập trung**: Giao thức SNMP (v1/v2c/v3 USM authPriv, MIB/OID, Polling vs Traps/Informs), Chuẩn Syslog (RFC 5424, Priority, 8 mức Severity, Facilities, UDP 514 vs TCP/TLS 6514), Flexible NetFlow / IPFIX, Kiến trúc SIEM Pipeline (ELK Stack / Wazuh / Graylog), Tập luật tương quan sự kiện (Correlation Rules) và Kịch bản kiểm thử giả lập tấn công (Hydra, Nmap, hping3, KPIs).

---

> [!TIP]
> Mỗi tập giáo trình đều bao gồm: Khung lý thuyết chuẩn học thuật, sơ đồ chu trình (Mermaid / ASCII), bảng so sánh đa chiều, mẫu cấu hình dòng lệnh (Cisco IOS, Linux Rsyslog, Wazuh XML Rules), và **Bộ câu hỏi ôn tập & phản biện bảo vệ đồ án (Viva Q&A)**.
