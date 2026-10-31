# THIẾT KẾ THIẾT BỊ CHO TỪNG PHÒNG – BẢN ĐÃ RÀ SOÁT

**Quy mô:** 2 tòa nhà, 5 tầng/tòa, **26 phòng**.

## 1. Nhận xét quan trọng sau khi rà soát
- File gốc có **26 phòng**, phù hợp với số thẻ phòng thực tế trong tài liệu.
- Các phòng làm việc thông thường đang dùng bộ thiết bị khá thống nhất: **Switch + PC + Wi‑Fi AP + máy in + điện thoại IP + camera + kiểm soát cửa**.
- **A-202** đang được gọi là phòng máy chủ nhưng danh sách hiện tại chỉ có switch, AP, điện thoại IP, camera và kiểm soát cửa. Nếu đây thực sự là **phòng tủ mạng/IDF**, nên bổ sung **tủ rack, patch panel, PDU, UPS, quản lý cáp và uplink quang**. Nếu đây là **phòng máy chủ**, cần bổ sung thêm máy chủ, lưu trữ và thiết bị nguồn/điều hòa phù hợp.
- NOC/SOC nên ưu tiên thiết bị giám sát; không nên coi đây là phòng văn phòng bình thường.
- Phòng họp và khu nghỉ không nhất thiết phải có máy in và nhiều PC cố định.

## 2. Danh sách từng phòng và thiết bị

| Mã | Phòng | Thiết bị chính nên có |
|---|---|---|
| **A-101** | Sảnh lễ tân | Switch, 2 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-102** | Phòng bảo vệ / Kiểm soát ra vào | Switch, 2 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-201** | Phòng Lab / Phát triển & Kiểm thử | Switch, 10 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-202** | Phòng Máy Chủ Trung Tâm & Data Center (MDF) | 2 Router WAN (Dual ISP), 2 Firewall HA, 2 Core/L3 Switch, 2 Tủ Rack 42U, 2 UPS Online 5000VA, Cụm Server Farm (AD, DNS, DHCP, Web, Mail, File, Database, NMS, Backup), Camera IP, Kiểm soát cửa vân tay/thẻ, Điều hòa chính xác |
| **A-203** | Phòng hỗ trợ IT (Helpdesk) | Switch, 4 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-301** | Phòng Lập trình Backend | Switch, 15 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-302** | Phòng Cơ sở dữ liệu / Kỹ sư dữ liệu | Switch, 8 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-303** | Phòng Lập trình Frontend | Switch, 15 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-401** | Phòng Kiểm thử (QA / Tester) | Switch, 10 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-402** | Phòng Thiết kế UI/UX | Switch, 6 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-403** | Phòng DevOps / Quản trị hệ thống | Switch, 8 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-404** | Phòng NOC/SOC (Giám sát & An ninh mạng) | Switch, 4 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa, NOC/SOC workstation, màn hình giám sát, thiết bị/console quản trị |
| **A-501** | Phòng Quản lý sản phẩm | Switch, 5 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-502** | Phòng CEO / CTO | Switch, 2 PC, Wi‑Fi AP, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **A-503** | Phòng họp Ban Giám đốc | Switch, Wi‑Fi AP, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-101** | Sảnh tiếp khách hàng / Đối tác | Switch, 2 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-102** | Phòng họp đối tác | Switch, Wi‑Fi AP, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-201** | Phòng Kinh doanh | Switch, 12 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-202** | Phòng Marketing | Switch, 8 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-301** | Phòng Nhân sự | Switch, 5 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-302** | Phòng Kế toán / Tài chính | Switch, 6 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-303** | Phòng Pháp chế / Hành chính | Switch, 3 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-304** | Phòng Chăm sóc khách hàng | Switch, 10 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-401** | Phòng Triển khai dự án (PM/BA) | Switch, 7 PC, Wi‑Fi AP, Máy in mạng, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-402** | Phòng Đào tạo nội bộ | Switch, Wi‑Fi AP, Điện thoại IP, Camera IP, Kiểm soát cửa |
| **B-403** | Căng tin / Khu nghỉ nhân viên | Switch, Wi‑Fi AP, Camera IP, Kiểm soát cửa, Điện thoại IP |

## 3. Thiết bị nên bố trí theo loại phòng

### Phòng làm việc nhân viên
- 1 switch phòng (số cổng tùy số PC).
- Mỗi nhân viên: 1 PC/laptop có dây hoặc Wi‑Fi.
- 1 Wi‑Fi AP cho khu vực.
- 1 máy in mạng nếu phòng có nhu cầu in chung.
- 1 điện thoại IP nếu doanh nghiệp dùng VoIP.
- Camera tại cửa/khu vực cần giám sát.
- Kiểm soát cửa đối với phòng có dữ liệu hoặc thiết bị quan trọng.

### Phòng họp
- Wi‑Fi AP.
- 1–2 cổng mạng cho thiết bị trình chiếu/họp.
- Camera hội nghị nếu có họp trực tuyến.
- Điện thoại IP nếu cần.
- Không bắt buộc phải đặt máy in và PC cố định.

### Phòng NOC/SOC
- Workstation giám sát.
- Nhiều màn hình hiển thị dashboard.
- Hệ thống giám sát mạng/NMS.
- Hệ thống SIEM/IDS/IPS hoặc console quản trị tùy mô hình.
- Switch quản lý, UPS và đường mạng dự phòng.

### Phòng tủ mạng/IDF
- Tủ rack.
- Patch panel.
- Switch access/distribution.
- ODF/khay phối quang nếu có uplink quang.
- PDU.
- UPS.
- Thanh quản lý cáp.
- Tiếp địa, cảm biến nhiệt độ/độ ẩm nếu cần.

## 4. Chữ cần dùng trên bản vẽ để dễ đọc

| Chữ trên bản vẽ | Nên ghi |
|---|---|
| Switch Phòng | **Switch phòng** |
| AP | **Wi‑Fi AP** |
| PRN | **Máy in mạng** |
| VOIP | **Điện thoại IP** |
| CAM | **Camera IP** |
| DOOR | **Kiểm soát cửa** |
| IDF | **Tủ mạng tầng (IDF)** |
| NOC/SOC | **Giám sát mạng & an ninh mạng** |
| PC | **Máy trạm** |

## 5. Kết luận
Bản thiết kế nên giữ bộ thiết bị chung cho phòng làm việc, nhưng điều chỉnh theo chức năng phòng. Quan trọng nhất là sửa **A-202** thành **phòng tủ mạng/IDF** hoặc đổi tên thành **phòng máy chủ** rồi bổ sung thiết bị tương ứng; đồng thời giảm thiết bị cố định ở phòng họp/khu nghỉ và bổ sung thiết bị giám sát chuyên dụng cho NOC/SOC.

> Lưu ý: Đây là bản rà soát và chuẩn hóa cách trình bày từ file gốc; các thiết bị bổ sung cho IDF/NOC/SOC là đề xuất thiết kế để bản vẽ sát với mô hình doanh nghiệp hơn.