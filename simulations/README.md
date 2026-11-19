# NETWORK SIMULATIONS (CISCO PACKET TRACER)

Thư mục chứa các file mô phỏng sơ đồ mạng doanh nghiệp TechCorp xây dựng trên **Cisco Packet Tracer (v8.2+)**.

---

## 📂 Danh mục tệp mô phỏng

1. **`phase-1-2-topology.pkt`**:
   - **Giai đoạn:** Giai đoạn 1 & 2 (Khảo sát & Thiết kế logic sơ bộ).
   - **Nội dung:** Sơ đồ kết nối phân cấp Collapsed-Core, các tủ mạng phân phối tầng IDF, phòng Server MDF A-202 và các switch truy cập tại 26 phòng ban.

2. **`phase-3-configured-security.pkt`**:
   - **Giai đoạn:** Giai đoạn 3 (Cấu hình bảo mật & Dịch vụ an ninh).
   - **Nội dung cấu hình:**
     - Quy hoạch IP quản trị trên **VLAN 2** (`192.168.2.0/24`).
     - Cấu hình gửi Syslog tập trung về máy chủ **`192.168.10.50`** (Server Farm).
     - Triển khai **Port Security** (Sticky MAC, vi phạm `shutdown`/`restrict`) trên các cổng Access.
     - Cấu hình phòng chống tấn công ARP Spoofing (Dynamic ARP Inspection) và DHCP Snooping.
     - 4 Extended ACLs kiểm soát luồng giao tiếp giữa các phân vùng (Security Zones).

---

## 📌 Hướng dẫn mở file
- Mở bằng phần mềm **Cisco Packet Tracer 8.0 trở lên**.
- Để kiểm tra Syslog: mở giao diện CLI trên Switch / Router và thực hiện thay đổi cấu hình hoặc kích hoạt vi phạm Port Security để quan sát gói log gửi đến Syslog Server.
