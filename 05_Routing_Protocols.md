# CHUYÊN ĐỀ 5: ROUTING PROTOCOLS (GIAO THỨC ĐỊNH TUYẾN TRONG DOANH NGHIỆP)

> **Mục tiêu**: Nắm vững nguyên lý hoạt động của **Static Routing** (Định tuyến tĩnh) và **OSPF** (Open Shortest Path First), bảng tra cứu **Administrative Distance (AD)**, cơ chế bảo mật xác thực định tuyến, và cách thức giám sát trạng thái định tuyến qua Syslog/SNMP Trap.

---

## 1. ĐỊNH TUYẾN TĨNH (STATIC ROUTING)

### 1.1. Khái Niệm & Ưu Nhược Điểm
Định tuyến tĩnh là phương pháp do người quản trị mạng cấu hình thủ công từng tuyến đường vào bảng định tuyến của Router/Firewall.

- **Ưu điểm**:
  - Tiêu tốn cực ít tài nguyên phần cứng (CPU/RAM).
  - Bảo mật tuyệt đối vì Router không gửi bản tin quảng bá định tuyến ra ngoài môi trường mạng.
  - Dự đoán chính xác 100% đường đi của gói tin.
  - Phù hợp tối đa cho các kết nối Stub Network (mạng cụt) và các đồ án mạng quy mô vừa/nhỏ.
- **Nhược điểm**:
  - Không có khả năng tự động ứng phó hoặc tìm đường vòng khi một liên kết bị đứt (trừ khi kết hợp cơ chế IP SLA Tracking).
  - Tốn công quản trị khi mở rộng quy mô mạng lớn.

### 1.2. Các Dạng Cấu Hình Định Tuyến Tĩnh Điển Hình
```cisco
! 1. Tuyến cố định chỉ định rõ Next-Hop IP
Router(config)# ip route 192.168.50.0 255.255.255.224 10.0.0.2

! 2. Tuyến mặc định (Default Route - Gateway of Last Resort) trỏ ra Internet
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1

! 3. Tuyến dự phòng trôi nổi (Floating Static Route với AD = 10 để dự phòng đường chính)
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.5 10
```

---

## 2. GIAO THỨC ĐỊNH TUYẾN ĐỘNG OSPF (OPEN SHORTEST PATH FIRST)

### 2.1. Bản Chất Link-State & Thuật Toán Dijkstra
OSPF là giao thức định tuyến Link-State chuẩn mở (RFC 2328), sử dụng thuật toán Dijkstra (Shortest Path First - SPF) để tính toán đường đi ngắn nhất không bị vòng lặp dựa trên chi phí đường truyền (**Cost Metric** tỷ lệ nghịch với băng thông: $\text{Cost} = \frac{10^8}{\text{Bandwidth in bps}}$).

```
+-------------------------------------------------------------------------------+
|                       CHU TRÌNH THIẾT LẬP LÂN CẬN OSPF                        |
+-------------------------------------------------------------------------------+
|  Down  --->  Init  --->  2-Way  --->  ExStart  --->  Exchange  --->  Loading  --->  FULL
|   |           |           |              |              |              |              |
| Không có   Nhận Hello   Bầu chọn       Quyết định      Trao đổi       Gửi LSR /      Đồng bộ
| gói tin    chưa có ID   DR/BDR         Master/Slave    DBD packet     LSU chi tiết   bảng LSDB
+-------------------------------------------------------------------------------+
```

- **Router ID (RID)**: Địa chỉ IP 32-bit duy nhất định danh cho mỗi Router (Ưu tiên: Lệnh cấu hình thủ công `router-id` $\rightarrow$ IP Loopback cao nhất $\rightarrow$ IP Interface vật lý hoạt động cao nhất).
- **Phân chia Vùng (Area)**: Giảm thiểu kích thước bảng cơ sở dữ liệu trạng thái đường liên kết (LSDB):
  - **Backbone Area (Area 0)**: Khu vực trung tâm bắt buộc mọi Area khác phải kết nối trực tiếp vào.
  - **Standard Area (Area 1, Area 2...)**: Vùng phân nhánh chứa các mạng người dùng/máy chủ.

### 2.2. Mẫu Cấu Hình OSPF Kèm Xác Thực An Toàn (MD5/SHA)
```cisco
! ==============================================================
! CẤU HÌNH OSPF CƠ BẢN VÀ XÁC THỰC BẢO VỆ ĐỊNH TUYẾN
! ==============================================================
Router(config)# router ospf 1
 Router(config-router)# router-id 1.1.1.1
 ! Quảng bá mạng DMZ và VLAN User vào OSPF Area 0
 Router(config-router)# network 192.168.50.0 0.0.0.31 area 0
 Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
 
 ! Tắt gửi gói tin Hello OSPF ra cổng kết nối người dùng (Tránh rò rỉ topology)
 Router(config-router)# passive-interface GigabitEthernet0/0/0.10
 Router(config-router)# exit

! Bật xác thực MD5 trên liên kết giữa 2 Router chống giả mạo bảng định tuyến (Route Hijacking)
Router(config)# interface GigabitEthernet0/0/1
 Router(config-if)# ip ospf authentication message-digest
 Router(config-if)# ip ospf message-digest-key 1 md5 OSPF_Auth_Key_2026#
 Router(config-if)# exit
```

---

## 3. BẢNG SO SÁNH ADMINISTRATIVE DISTANCE (AD)

**Administrative Distance (AD)** là giá trị đo lường độ tin cậy của một nguồn thông tin định tuyến (giá trị càng nhỏ càng được ưu tiên đưa vào bảng định tuyến FIB/RIB):

| Nguồn Định Tuyến (Route Source) | Administrative Distance (AD) | Metric Sử Dụng |
| :--- | :--- | :--- |
| **Directly Connected** (Kết nối trực tiếp) | **`0`** | Không áp dụng |
| **Static Route** (Định tuyến tĩnh) | **`1`** | Do quản trị viên gán |
| **eBGP** (External BGP) | **`20`** | AS-Path, Local Pref, MED... |
| **EIGRP (Internal)** | **`90`** | Composite (Bandwidth + Delay) |
| **OSPF** (Open Shortest Path First) | **`110`** | Cost ($\propto \frac{1}{\text{Bandwidth}}$) |
| **IS-IS** | **`115`** | Cost mặc định |
| **RIP** (Routing Information Protocol) | **`120`** | Hop Count (Số bước nhảy, tối đa 15) |
| **iBGP** (Internal BGP) | **`200`** | BGP Path Attributes |

---

## 4. ĐÁNH GIÁ PHÙ HỢP CHO ĐỒ ÁN AN TOÀN & GIÁM SÁT MẠNG

> [!NOTE]
> **Định hướng tối ưu thời gian nghiên cứu**:
> - **Lý do chỉ tập trung Static Routing & OSPF**: Trong thực tế doanh nghiệp và đồ án giám sát an toàn thông tin, mạng doanh nghiệp thường áp dụng Static Routing tại biên (Firewall/Internet) và OSPF trong mạng lõi nội bộ (Core Network).
> - **Lý do bỏ qua RIP và EIGRP**: 
>   - `RIP`: Quá lỗi thời, metric giới hạn 15 hop, thời gian hội tụ rất chậm (30 giây), bảo mật kém.
>   - `EIGRP`: Là giao thức độc quyền lịch sử của Cisco, ít dùng trong các hệ thống mạng đa hãng hiện đại (Multi-vendor).
> - **Điểm nhấn giám sát định tuyến**: Hệ thống SIEM cần lắng nghe các bản tin Syslog OSPF như `%OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on Gi0/0/1 from FULL to DOWN` để cảnh báo tức thời sự cố đứt tuyến cáp hoặc dấu hiệu kẻ gian ngắt kết nối định tuyến.
