# BÁO CÁO TIẾN ĐỘ – GIAI ĐOẠN 3

**Triển khai giải pháp phòng chống tấn công cho hệ thống mạng Cisco (Packet Tracer)**

| Hạng mục | Nội dung |
|---|---|
| Đề tài | Phân tích và thiết kế hệ thống giám sát và quản lý nhật ký an toàn cho mạng doanh nghiệp |
| Giai đoạn | Giai đoạn 3 – Phòng chống tấn công bên ngoài và bên trong nội bộ |
| Phạm vi | Hệ thống mạng mô phỏng gồm các switch tầng/khu vực, switch trung tâm và máy chủ Syslog |

## MỤC LỤC

**PHẦN I – Cấu hình địa chỉ IP, Syslog và Port Security**

- [I.1. Mục tiêu giai đoạn 3](#i1-mục-tiêu-giai-đoạn-3)
- [I.2. Kế hoạch chia địa chỉ IP](#i2-kế-hoạch-chia-địa-chỉ-ip)
- [I.3. Cấu hình Syslog](#i3-cấu-hình-syslog)
- [I.4. Cấu hình Port Security](#i4-cấu-hình-port-security)
- [I.5. Kiểm tra và đánh giá kết quả](#i5-kiểm-tra-và-đánh-giá-kết-quả)
- [I.6. Tiến độ thực hiện giai đoạn 3](#i6-tiến-độ-thực-hiện-giai-đoạn-3)
- **PHẦN II – Các giải pháp phòng chống tấn công**

- [II.1. Tổng quan tiến độ](#ii1-tổng-quan-tiến-độ)
- [II.2. Bảng ma trận phân chia nhiệm vụ](#ii2-bảng-ma-trận-phân-chia-nhiệm-vụ)
- [II.3. Giải pháp bảo vệ dịch vụ ARP khi bị tấn công](#ii3-giải-pháp-bảo-vệ-dịch-vụ-arp-khi-bị-tấn-công)
- [II.4. Giải pháp thật sự cần thiết cho từng kiểu tấn công](#ii4-giải-pháp-thật-sự-cần-thiết-cho-từng-kiểu-tấn-công)
- [II.5. Giải pháp tấn công nội bộ – RBAC (Xuân)](#ii5-giải-pháp-tấn-công-nội-bộ--rbac-xuân)
- [II.6. Khó khăn và hướng xử lý](#ii6-khó-khăn-và-hướng-xử-lý)

---

# PHẦN I – CẤU HÌNH ĐỊA CHỈ IP, SYSLOG VÀ PORT SECURITY

## I.1. Mục tiêu giai đoạn 3

- Hoàn thiện kế hoạch địa chỉ IP quản trị cho các thiết bị mạng.
- Cấu hình Syslog để thiết bị gửi nhật ký về máy chủ tập trung `192.168.10.50`.
- Triển khai Port Security trên các cổng access kết nối thiết bị đầu cuối.
- Kiểm tra trạng thái cấu hình, MAC sticky và số lần vi phạm Port Security.

## I.2. Kế hoạch chia địa chỉ IP

| Thiết bị/Thành phần | VLAN/Mạng | Địa chỉ IP | Subnet mask | Gateway | Trạng thái/Ghi chú |
|---|---|---|---|---|---|
| Gateway quản trị VLAN 2 | VLAN 2 | `192.168.2.1` | `255.255.255.0` | `192.168.2.1` | Gateway trong kế hoạch quản trị |
| SW-Tang1 | VLAN 10 | `192.168.10.2` | `255.255.255.0` | `192.168.10.1` | Đã ghi nhận cấu hình quản trị |
| SW-Tang2 | VLAN 10 | `192.168.10.3` | `255.255.255.0` | `192.168.10.1` | Đã ghi nhận cấu hình quản trị |
| SW-ToaA | VLAN 2 | `192.168.2.3` | `255.255.255.0` | `192.168.2.1` | Đã ghi nhận cấu hình quản trị |
| SW-A-T3 | VLAN 2 | `192.168.2.4` | `255.255.255.0` | `192.168.2.1` | Đã ghi nhận cấu hình quản trị |
| SW-A-T4 | VLAN 2 | `192.168.2.5` | `255.255.255.0` | `192.168.2.1` | Đã ghi nhận cấu hình quản trị |
| SW-A-T5 | VLAN 2 | `192.168.2.6` | `255.255.255.0` | `192.168.2.1` | Đã ghi nhận cấu hình quản trị |
| SW-Tang1B | VLAN 2 | `192.168.2.7` | `255.255.255.0` | `192.168.2.1` | Đã ghi nhận cấu hình quản trị |
| B2-SW | VLAN 2 | `192.168.2.8` | `255.255.255.0` | `192.168.2.1` | Đã ghi nhận cấu hình quản trị |
| B3-SW | VLAN 2 | `192.168.2.9` | `255.255.255.0` | `192.168.2.1` | Đã ghi nhận cấu hình quản trị |
| B4-SW | VLAN 2 | `192.168.2.10` | `255.255.255.0` | `192.168.2.1` | Đã ghi nhận cấu hình quản trị |
| DHCP Server | VLAN 10 | `192.168.10.10` | `255.255.255.0` | `192.168.10.1` | Địa chỉ máy chủ DHCP trong mô hình |
| Syslog Server | VLAN 10 | `192.168.10.50` | `255.255.255.0` | `192.168.10.1` | Máy chủ nhận log tập trung |

## I.3. Cấu hình Syslog

### I.3.1. Mục đích

Syslog tập trung nhật ký hoạt động từ switch/router về máy chủ. Người
quản trị có thể theo dõi trạng thái cổng, sự kiện bảo mật, thay đổi cấu
hình và lỗi phát sinh.

### I.3.2. Thông số triển khai

| Thông số | Giá trị |
|---|---|
| Syslog Server | `192.168.10.50` |
| Mạng/VLAN máy chủ | VLAN 10 – `192.168.10.0/24` |
| Gateway | `192.168.10.1` |
| Mức log đã trao đổi | `debugging` (mức 7) |
| Thiết bị đã ghi nhận cấu hình gửi log | SWitch; router, mô hình Firewall ASA cũng sử dụng Syslog Server `192.168.10.50` |

### I.3.3. Câu lệnh cấu hình tham khảo trên Cisco

```text
configure terminal
logging 192.168.10.50
logging trap debugging
end
write memory
```

### I.3.4. Kiểm tra

```text
show running-config | include logging
show logging
```

Trên máy chủ `192.168.10.50`, cần xác nhận bật dịch vụ Syslog đang chạy và
cho phép nhận log.
## I.4. Cấu hình Port Security

### I.4.1. Mục đích

Port Security giới hạn số địa chỉ MAC được phép xuất hiện trên một cổng
access. Sticky tự động học và lưu MAC vào cấu hình. Chế độ `restrict`
loại bỏ lưu lượng từ MAC vi phạm và ghi nhận vi phạm, không đưa cổng vào
trạng thái shutdown.

### I.4.2. Thông số cấu hình

| Tham số | Giá trị |
|---|---|
| Loại cổng | Access – kết nối thiết bị đầu cuối |
| Port Security | Bật |
| Maximum MAC | 2 MAC/cổng |
| Kiểu học MAC | Sticky |
| Violation mode | Restrict |
| PortFast | Bật trên cổng access phù hợp |
| Lưu cấu hình | `write memory` |

### I.4.3. Phạm vi cổng đã ghi nhận

| Thiết bị | Các cổng đã ghi nhận | Số cổng | Ghi chú |
|---|---|:---:|---|
| SW-Tang1 | Fa0/1–4, Fa0/6–9 | 8 | Cổng access; không áp dụng lên uplink trunk |
| SW-Tang2 | Fa0/2–10 | 9 | Fa0/1 là trunk |
| SW-A-T3 | Fa0/2–3, Fa0/6–8, Fa0/10–12, Fa0/14 | 9 | Cổng access; loại trừ uplink |
| SW-A-T4 | Chưa có danh sách chi tiết | 12 | Đã ghi nhận tổng số cổng triển khai |
| SW-A-T5 | Chưa có danh sách chi tiết | 6 | Đã ghi nhận tổng số cổng triển khai |
| SW-Tang1B | Chưa có danh sách chi tiết | 6 | Đã ghi nhận tổng số cổng triển khai |

### I.4.4. Câu lệnh cấu hình mẫu

Ví dụ dưới đây minh họa cấu hình trên một nhóm cổng access. Cần thay
đúng dải cổng theo từng switch.

```text
configure terminal
interface range fa0/2 - 4
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree portfast
end
write memory
```

**Không cấu hình Port Security trên cổng trunk/uplink.** Cần xác định
đúng vai trò cổng trước khi áp dụng.

## I.5. Kiểm tra và đánh giá kết quả

Các lệnh kiểm tra:

```text
show port-security
show port-security interface fa0/x
show port-security address
show running-config interface fa0/x
show logging
show running-config | include logging
show ip interface brief
show vlan brief
show interfaces trunk
```

Kết quả đã có dữ liệu xác nhận trên **SW-Tang2, cổng Fa0/2 (PC4, VLAN 21)**:

- Trạng thái: `Secure-up`.
- Violation mode: `Restrict`.
- Maximum: 2 MAC.
- Sticky MAC đã học: `00D0.FF75.A2D3`.
- Số lần vi phạm: 0.

Các cổng/thiết bị còn lại cần bổ sung lệnh `show` tương ứng để xác
nhận trạng thái thực tế trước khi nghiệm thu.

## I.6. Tiến độ thực hiện giai đoạn 3

| STT | Nội dung | Tiến độ |
|:---:|---|---|
| 1 | Lập kế hoạch IP quản trị và IP máy chủ | Đã tổng hợp; SW-Tang1B đã xác nhận IP |
| 2 | Cấu hình Syslog về `192.168.10.50` | Đã ghi nhận cấu hình trên SW-Tang2; cần xác nhận log đến máy chủ |
| 3 | Triển khai Port Security trên cổng access | Đã triển khai theo phạm vi cổng đã thống kê |
| 4 | Kiểm tra sticky MAC và violation | Đã xác nhận SW-Tang2 Fa0/2: Secure-up, Restrict, 0 violation |
| 5 | Hoàn thiện hồ sơ và nghiệm thu | Đang cập nhật minh chứng cho các switch còn lại |

----

# PHẦN II – CÁC GIẢI PHÁP PHÒNG CHỐNG TẤN CÔNG

## II.1. Tổng quan tiến độ

Giai đoạn 3 đã hoàn thành phần hạ tầng: IP quản trị, ACL trên ASA, Syslog và Port Security. Giai đoạn 3 tập trung vào bảo mật: xác định các kiểu tấn công theo từng lớp mạng, phân công người phụ trách và triển khai giải pháp phòng chống tương ứng.

| Hạng mục | Trạng thái |
|---|---|
| ACL OUTSIDE_IN trên ASA | Hoàn thành |
| Syslog tập trung | Hoàn thành |
| VLAN & trunk | Hoàn thành |
| IP quản trị switch | Hoàn thành |
| Port Security | Hoàn thành |
| DHCP Snooping | Hoàn thành |
| Chống tấn công lớp 2/3/4 + RBAC | Hoàn thành|

## II.2. Bảng ma trận phân chia nhiệm vụ

Phân công dựa trên danh sách tấn công đã thống nhất trong nhóm. Ký hiệu: ● = phụ trách chính (cấu hình, kiểm thử, viết báo cáo); ○ = hỗ trợ / phối hợp kiểm tra chéo;

### II.2.1. Ma trận tổng hợp

| STT | Kiểu tấn công | Lớp | Nguyễn Anh Xuân | Nguyễn Phúc Vượng | Nguyễn Minh Trí | Nhóm |
|:---:|---|---|:---:|:---:|:---:|---|
| 1 | Port Scan | Tầng 2 | ○ | ○ | ● | Bên ngoài |
| 2 | DHCP Spoofing / Starvation | Tầng 2 | ○ | ● | ○ | Bên ngoài |
| 3 | VLAN Hopping | Tầng 2 | ○ | ● | ○ | Bên ngoài |
| 4 | ARP Spoofing | Tầng 2 | ○ | ○ | ● | Bên ngoài |
| 5 | IP Spoofing, ICMP Flood | Tầng 3 | ● | ○ | ○ | Bên ngoài |
| 6 | TCP SYN Flood, UDP Flood, DNS | Tầng 4 | ○ | ○ | ● | Bên ngoài |
| 7 | DDoS / SYN | Tầng 3–4 | ○ | ○ | ● | Bên ngoài |
| 8 | Phân quyền RBAC | Quản trị | ● | ○ | ○ | Nội bộ |

### II.2.2. Chi tiết nhiệm vụ – Nguyễn Anh Xuân

| Nhiệm vụ | Giải pháp chính |
|---|---|
| IP Spoofing (tầng 3) | ACL chống giả mạo ở chiều vào outside; uRPF; IP Source Guard tại switch access |
| ICMP Flood / Smurf | Chỉ cho phép echo/echo-reply cần thiết; tắt directed-broadcast, redirects, unreachables |
| RBAC nội bộ | Tài khoản local theo cấp quyền (15/7/1), SSH, khóa đăng nhập sai, exec-timeout |
| Hỗ trợ | Kiểm tra chéo Port Scan, DHCP, VLAN Hopping, ARP, TCP/UDP flood – DNS, DDoS |

### II.2.3. Chi tiết nhiệm vụ – Nguyễn Phúc Vượng

| Nhiệm vụ | Giải pháp chính |
|---|---|
| DHCP Spoofing / Starvation (tầng 2) | DHCP Snooping (đã bật): rà soát cổng trust/untrusted trên toàn bộ switch, giới hạn tốc độ gói DHCP, kết hợp Port Security chống cạn kiệt địa chỉ |
| VLAN Hopping (tầng 2) | Tắt DTP, đổi native VLAN sang VLAN rỗng, giới hạn allowed VLAN, shutdown cổng thừa vào VLAN cách ly |
| Hỗ trợ | Kiểm tra chéo Port Scan, ARP, IP Spoofing – ICMP, TCP/UDP flood – DNS, DDoS, RBAC |

### II.2.4. Chi tiết nhiệm vụ – Nguyễn Minh Trí

| Nhiệm vụ | Giải pháp chính |
|---|---|
| ARP Spoofing (tầng 2) | DAI dựa trên bảng DHCP Snooping, ARP ACL cho thiết bị IP tĩnh, rate limit ARP, giám sát Syslog |
| TCP SYN Flood (tầng 4) | Giới hạn kết nối nửa mở trên ASA, chỉ mở cổng dịch vụ cần thiết, giám sát Syslog |
| UDP Flood (tầng 4) | ACL chỉ cho phép UDP cần thiết (53, 514), chặn UDP khác từ outside |
| DNS | DNS chỉ phục vụ nội bộ, chặn truy vấn DNS từ outside, máy trạm chỉ dùng DNS nội bộ |
| DDoS / SYN | Phối hợp nhiều lớp: anti-spoofing, giới hạn kết nối, chặn UDP/ICMP thừa, Syslog |
| Hỗ trợ | Kiểm tra chéo Port Scan, DHCP, VLAN Hopping, RBAC |

## II.3. Giải pháp bảo vệ dịch vụ ARP khi bị tấn công

### II.3.1. Bản chất tấn công

ARP không có cơ chế xác thực: máy nào cũng có thể gửi gratuitous ARP tuyên bố "IP 192.168.x.1 có MAC là của tôi". Kẻ tấn công lợi dụng việc này để chèn vào giữa máy trạm và gateway (Man-in-the-Middle), nghe lén, sửa dữ liệu, hoặc làm tê liệt mạng (ARP flooding / DoS).

- **Dấu hiệu:** bảng ARP của máy trạm có 2 IP trùng một MAC; MAC của gateway thay đổi bất thường; mạng chập chờn, chậm.
- **Mục tiêu bị nhắm:** gateway các VLAN (192.168.2.1, 192.168.10.1…), máy chủ Syslog 192.168.10.50, máy chủ web 192.168.10.12.

### II.3.2. Các giải pháp cần thiết (theo thứ tự ưu tiên)

| # | Giải pháp | Tác dụng | Điều kiện |
|:---:|---|---|---|
| 1 | Dynamic ARP Inspection (DAI) | Switch kiểm tra mọi gói ARP trên cổng untrusted, đối chiếu IP–MAC với bảng DHCP Snooping; gói giả bị hủy | Bắt buộc bật DHCP Snooping trước |
| 2 | ARP ACL cho thiết bị IP tĩnh | Khai báo cặp IP–MAC hợp lệ của gateway, máy chủ (không nhận IP từ DHCP) | Biết MAC thật của thiết bị |
| 3 | Trust đúng cổng uplink/trunk | Chỉ cổng nối router/switch phân phối là trusted, mọi cổng người dùng untrusted | Xác định đúng cổng uplink |
| 4 | Giới hạn tốc độ ARP (rate limit) | Cổng gửi quá số gói ARP/giây bị err-disable → chống ARP flood | Mặc định 15 pps trên cổng untrusted |
| 5 | ARP tĩnh cho gateway trên máy chủ quan trọng | Máy chủ không bị đầu độc ARP của gateway | Áp dụng cho số ít máy chủ |
| 6 | Port Security (đã có) | Giới hạn số MAC/cổng, hạn chế giả mạo MAC kèm ARP | Đã cấu hình 74 cổng |
| 7 | Giám sát qua Syslog | Gói ARP bị DAI hủy được ghi log về 192.168.10.50 | Syslog đã hoạt động |

### II.3.3. Cấu hình mẫu (switch tầng 1 tòa A, VLAN 2 và 10)

**Bước 1 – Bảo đảm DHCP Snooping đã bật**

```text
ip dhcp snooping
ip dhcp snooping vlan 2,10
no ip dhcp snooping information option
interface fastEthernet 0/1          ! cổng uplink
  ip dhcp snooping trust
```

**Bước 2 – Bật DAI và trust cổng uplink**

```text
ip arp inspection vlan 2,10
ip arp inspection validate src-mac dst-mac ip
interface fastEthernet 0/1
  ip arp inspection trust
interface range fastEthernet 0/2 - 24
  ip arp inspection limit rate 15
errdisable recovery cause arp-inspection
errdisable recovery interval 300
```

**Bước 3 – ARP ACL cho thiết bị IP tĩnh**

```text
arp access-list STATIC-HOSTS
  permit ip host 192.168.10.1  mac host <MAC-gateway>
  permit ip host 192.168.10.50 mac host <MAC-syslog>
  permit ip host 192.168.10.12 mac host <MAC-web>
ip arp inspection filter STATIC-HOSTS vlan 10
```

**Bước 4 – ARP tĩnh trên router/máy chủ (tùy chọn)**

```text
arp 192.168.10.50 <MAC-syslog> arpa
! Trên máy chủ Windows: arp -s 192.168.10.1 <MAC-gateway>
```

**Bước 5 – Kiểm tra**

```text
show ip arp inspection
show ip arp inspection vlan 10
show ip arp inspection interfaces
show ip dhcp snooping binding
show logging | include ARP
```

### II.3.4. Lưu ý khi triển khai

- Bật DAI khi chưa có DHCP Snooping hoặc chưa khai ARP ACL cho máy IP tĩnh sẽ làm mất kết nối của chính các máy đó (gateway, máy chủ, IP quản trị VLAN 2).
- Không đặt trust nhầm cho cổng người dùng; cổng trunk giữa các switch đều phải trust ở cả hai đầu.
- Packet Tracer không hỗ trợ đầy đủ mọi lệnh DAI ở mọi phiên bản/dòng switch. Kiểm tra bằng "ip arp inspection ?" trên thiết bị; nếu không hỗ trợ, dùng Port Security + ARP tĩnh + giám sát Syslog và ghi rõ hạn chế này trong báo cáo.

## II.4. Giải pháp thật sự cần thiết cho từng kiểu tấn công


### II.4.1. Port Scan (trinh sát)

- **Nguy cơ:** kẻ tấn công dò cổng mở để tìm dịch vụ khai thác.
- **Giải pháp:** giữ nguyên chính sách "chặn mặc định" của ASA (implicit deny), chỉ mở 80/443 tới máy chủ web; tắt dịch vụ không dùng trên router/switch; ghi log gói bị chặn.

```text
access-list OUTSIDE_IN extended deny ip any any log
no ip http server
no cdp run                 ! trên cổng hướng ra ngoài: no cdp enable
```

### II.4.2. DHCP Spoofing / Starvation (Vượng)

- **Giải pháp:** DHCP Snooping (đã bật trên mô hình), chỉ trust cổng nối DHCP server/uplink; giới hạn tốc độ gói DHCP; Port Security giới hạn số MAC để chống starvation.

```text
ip dhcp snooping
ip dhcp snooping vlan 2,10,21,31-33,41-44
interface fastEthernet 0/1
  ip dhcp snooping trust
interface range fastEthernet 0/2 - 24
  ip dhcp snooping limit rate 10
```

### II.4.3. VLAN Hopping (Vượng)

- **Nguy cơ:** Switch Spoofing (giả trunk qua DTP) và Double Tagging (gắn 2 tag, lợi dụng native VLAN).
- **Giải pháp cần thiết:**
  - Cổng người dùng: ép access, tắt DTP (switchport nonegotiate).
  - Cổng trunk: ép trunk, tắt DTP, đổi native VLAN sang VLAN không dùng (ví dụ 999), chỉ cho phép VLAN cần thiết.
  - Cổng thừa: shutdown và đưa vào VLAN cách ly (ví dụ 998 – BLACKHOLE).

```text
vlan 998
  name BLACKHOLE
vlan 999
  name NATIVE_UNUSED
interface range fastEthernet 0/2 - 20
  switchport mode access
  switchport nonegotiate
interface fastEthernet 0/1
  switchport mode trunk
  switchport nonegotiate
  switchport trunk native vlan 999
  switchport trunk allowed vlan 2,10,12,21,31-33,41-44,200
interface range fastEthernet 0/21 - 24
  switchport access vlan 998
  shutdown
```

- **Kiểm tra:** `show interfaces trunk`, `show interfaces fa0/2 switchport` (Negotiation of Trunking: Off).

### II.4.4. ARP Spoofing (Trí)

- **Nguy cơ:** kẻ tấn công gửi gói ARP giả, tuyên bố MAC của mình là MAC của gateway (192.168.2.1, 192.168.10.1…) hoặc máy chủ (Syslog 192.168.10.50, web 192.168.10.12) để chèn vào giữa luồng dữ liệu (Man-in-the-Middle), nghe lén/sửa dữ liệu hoặc gây gián đoạn mạng; gửi ARP ồ ạt để gây ARP flood.
- **Giải pháp cần thiết:**
  - Dynamic ARP Inspection (DAI): đối chiếu cặp IP–MAC trong gói ARP với bảng DHCP Snooping, hủy gói giả trên cổng untrusted.
  - DHCP Snooping phải bật trước (mục II.4.2) để có bảng binding hợp lệ.
  - Chỉ trust cổng uplink/trunk; mọi cổng người dùng để untrusted.
  - ARP ACL cho các thiết bị đặt IP tĩnh (gateway, máy chủ) vì không có trong bảng DHCP Snooping.
  - Giới hạn 15 gói ARP/giây trên cổng người dùng để chống ARP flood, tự phục hồi err-disable sau 300 giây.
  - Port Security (đã có) hạn chế giả mạo MAC; log gói ARP bị hủy được gửi về Syslog 192.168.10.50.

```text
ip dhcp snooping
ip dhcp snooping vlan 2,10
ip arp inspection vlan 2,10
ip arp inspection validate src-mac dst-mac ip
interface fastEthernet 0/1                ! cổng uplink
  ip dhcp snooping trust
  ip arp inspection trust
interface range fastEthernet 0/2 - 24
  ip arp inspection limit rate 15
errdisable recovery cause arp-inspection
errdisable recovery interval 300
arp access-list STATIC-HOSTS
  permit ip host 192.168.10.1  mac host <MAC-gateway>
  permit ip host 192.168.10.50 mac host <MAC-syslog>
  permit ip host 192.168.10.12 mac host <MAC-web>
ip arp inspection filter STATIC-HOSTS vlan 10
```

- **Kiểm tra:** `show ip arp inspection vlan 10`, `show ip arp inspection interfaces`, `show ip dhcp snooping binding`, `show logging | include ARP`; trên máy trạm dùng `arp -a` để xác nhận MAC của gateway không bị thay đổi.
- Chi tiết các bước và lưu ý triển khai xem thêm mục II.3.

### II.4.5. IP Spoofing (Xuân)

- **Nguy cơ:** gói từ bên ngoài giả IP nội bộ hoặc IP riêng để vượt ACL, phục vụ tấn công phản xạ/DDoS.
- **Giải pháp:** ACL chống giả mạo ở chiều vào outside (chặn dải nội bộ, dải riêng, loopback); uRPF trên router biên; IP Source Guard tại switch access (dựa trên bảng DHCP Snooping).

```text
! Router biên – cổng hướng ra ngoài
ip access-list extended ANTI-SPOOF
  deny ip 192.168.0.0 0.0.255.255 any log
  deny ip 10.0.0.0 0.255.255.255 any log      ! trừ link 10.255.0.0 nếu cần, đặt permit trước
  deny ip 172.16.0.0 0.15.255.255 any log
  deny ip 127.0.0.0 0.255.255.255 any log
  permit ip any any
interface <cổng-outside>
  ip access-group ANTI-SPOOF in
  ip verify unicast source reachable-via rx   ! uRPF (nếu PT hỗ trợ)
! Switch access
interface range fastEthernet 0/2 - 20
  ip verify source
```

> **Lưu ý:** link 10.255.0.1 – 10.255.0.2 và máy 192.168.45.2 đang được ACL OUTSIDE_IN cho phép; cần đặt dòng permit cho các địa chỉ hợp lệ này trước dòng deny để không chặn nhầm Syslog và truy cập web.

### II.4.6. ICMP Flood / Smurf (Xuân)

- **Giải pháp:** chỉ cho phép echo/echo-reply giữa các cặp địa chỉ đã quy định (ACL hiện tại đã làm); chặn ICMP khác từ outside; tắt directed-broadcast để chống Smurf; tắt redirects/unreachables trên cổng ngoài.

```text
interface <cổng-outside>
  no ip directed-broadcast
  no ip redirects
  no ip unreachables
! ASA: chỉ ICMP echo-reply quay về, chặn phần còn lại
access-list OUTSIDE_IN extended permit icmp any any echo-reply
access-list OUTSIDE_IN extended deny icmp any any log
```

### II.4.7. TCP SYN Flood (Trí)

- **Nguy cơ:** hàng loạt gói SYN không hoàn tất bắt tay làm cạn bảng kết nối của máy chủ web 192.168.10.12.
- **Giải pháp:** giới hạn kết nối nửa mở (embryonic) trên ASA bằng MPF; chỉ mở 80/443; Port Security đã cấu hình hạn chế nguồn từ bên trong; theo dõi log.

```text
access-list WEB_TRAFFIC extended permit tcp any host 192.168.10.12 eq www
access-list WEB_TRAFFIC extended permit tcp any host 192.168.10.12 eq 443
class-map WEB_CLASS
  match access-list WEB_TRAFFIC
policy-map OUTSIDE_POLICY
  class WEB_CLASS
    set connection embryonic-conn-max 100 per-client-embryonic-max 10
service-policy OUTSIDE_POLICY interface outside
```

ASA trong Packet Tracer hỗ trợ MPF hạn chế; nếu lệnh set connection không nhận, ghi nhận giải pháp ở mức thiết kế và minh chứng bằng ACL + Syslog.

### II.4.8. UDP Flood (Trí)

- **Giải pháp:** chính sách whitelist: chỉ cho phép UDP thật sự cần (514 Syslog từ 10.255.0.1, 53 tới máy chủ DNS nếu public), chặn toàn bộ UDP còn lại từ outside; giới hạn số kết nối UDP trên ASA.

```text
access-list OUTSIDE_IN extended permit udp host 10.255.0.1 host 192.168.10.50 eq 514
access-list OUTSIDE_IN extended deny udp any any log
```

### II.4.9. Tấn công DNS (Trí)

- **Nguy cơ:** DNS amplification (lợi dụng máy chủ DNS mở), DNS spoofing chuyển hướng người dùng.
- **Giải pháp:** máy chủ DNS chỉ trả lời truy vấn từ mạng nội bộ; ACL chỉ cho phép UDP/TCP 53 tới đúng máy chủ DNS; máy trạm chỉ được dùng DNS nội bộ (chặn DNS ra ngoài trực tiếp); bản ghi DNS quan trọng cấu hình tĩnh.

```text
! Router/ASA phía trong: chỉ máy chủ DNS được truy vấn ra ngoài
access-list INSIDE_OUT extended permit udp host <IP-DNS> any eq domain
access-list INSIDE_OUT extended deny udp any any eq domain log
access-list INSIDE_OUT extended permit ip any any
```

### II.4.10. DDoS / SYN (Trí)

- **Giải pháp:** kết hợp nhiều lớp – anti-spoofing (II.4.5), giới hạn kết nối (II.4.7), chặn UDP/ICMP thừa (II.4.6, II.4.8), Port Security và rate limit DHCP/ARP ở switch, giám sát tập trung Syslog để phát hiện sớm; bổ sung IDS/IPS và các giải pháp mở rộng ở mục II.4.11.

### II.4.11. Giải pháp chống DDoS bổ sung (Trí)

Đối chiếu với 7 giải pháp chống DDoS phổ biến, mô hình đã có Firewall/ACL và Anti-Spoofing; Rate Limiting mới có ở tầng 2 và tầng 4. Mục này bổ sung IDS/IPS (triển khai được trên Packet Tracer) và các giải pháp ở mức thiết kế cho hệ thống thực tế.

#### a) Các giải pháp đã có trên mô hình

- **Firewall và ACL:** ASA chặn mặc định, chỉ mở 80/443 tới máy chủ web; whitelist UDP 514/53; chặn ICMP thừa (mục II.4.1, II.4.6, II.4.8, II.4.9).
- **Anti-Spoofing:** ACL ANTI-SPOOF, uRPF, IP Source Guard, tắt directed-broadcast (mục II.4.5, II.4.6).
- **Rate Limiting:** giới hạn gói DHCP (10 pps), ARP (15 pps) trên switch; giới hạn kết nối nửa mở trên ASA (mục II.4.7). Phần giới hạn request HTTP theo IP được bổ sung ở mục c.

#### b) IDS/IPS – IOS IPS trên router biên 

- **IDS:** so khớp lưu lượng với chữ ký tấn công, sinh cảnh báo gửi về Syslog 192.168.10.50.
- **IPS:** đặt inline trên cổng hướng ra ngoài, tự động hủy gói khớp luật (deny-packet-inline).
- **Điều kiện:** router ISR 1941/2911 trong Packet Tracer, bật license securityk9.
- **Các bước triển khai:** bật license bảo mật cho router; tạo thư mục lưu chữ ký và đặt tên luật IPS; chỉ dùng nhóm chữ ký cơ bản để router không quá tải; áp luật IPS vào chiều vào của cổng hướng ra ngoài; bật chữ ký ICMP Echo (2004) với hành động cảnh báo và hủy gói; gửi cảnh báo về máy chủ Syslog.
- **Kiểm tra:** xem trạng thái IPS và các chữ ký đang bật trên router, đối chiếu log cảnh báo trên máy chủ Syslog.
- **Minh chứng:** từ máy bên ngoài ping liên tục vào mạng nội bộ → ping thất bại, Syslog ghi cảnh báo %IPS-4-SIGNATURE Sig:2004.

#### c) Rate Limiting tầng ứng dụng (mức thiết kế)

- Giới hạn số request HTTP trên mỗi IP trong một khoảng thời gian (ví dụ 10 request/giây) tại WAF hoặc máy chủ web (Nginx limit_req, Apache mod_evasive).
- IP vượt ngưỡng bị trả lỗi 429 hoặc tạm chặn, tránh cạn kiệt tài nguyên máy chủ web 192.168.10.12.

#### d) WAF – Web Application Firewall (mức thiết kế)

- Đặt trước máy chủ web 192.168.10.12 (ví dụ ModSecurity + OWASP CRS, hoặc WAF của Cloudflare).
- Lọc HTTP/HTTPS request bất thường, phát hiện bot, chặn tấn công tầng ứng dụng (HTTP flood, SQL Injection, XSS).
- Packet Tracer không có thiết bị WAF nên chỉ trình bày ở mức thiết kế.

#### e) Traffic Scrubbing Center và Anycast Routing (mức thiết kế)

- **Lý do cần:** khi lưu lượng tấn công vượt băng thông đường truyền Internet, ACL/IPS tại biên không còn tác dụng vì đường truyền đã nghẽn trước khi gói tin đến thiết bị.
- **Scrubbing Center:** khi phát hiện tấn công, lưu lượng được chuyển hướng qua trung tâm lọc của nhà mạng (dịch vụ Anti-DDoS của VNPT, Viettel…) hoặc nhà cung cấp cloud; gói độc hại bị loại bỏ, chỉ lưu lượng sạch được trả về hệ thống.
- **Anycast:** cùng một địa chỉ IP được quảng bá từ nhiều trung tâm dữ liệu; lưu lượng tấn công bị chia nhỏ về nhiều điểm, không một máy chủ nào bị quá tải.
- **Phạm vi:** đây là dịch vụ của nhà mạng/cloud (Cloudflare, AWS Shield, Akamai), không mô phỏng được trên Packet Tracer; đề xuất cho giai đoạn triển khai thực tế.

#### Kết luận

Mô hình phòng thủ DDoS gồm nhiều lớp: Scrubbing/Anycast (nhà mạng) → Firewall/ACL + Anti-Spoofing + IPS (biên mạng) → Rate Limiting + WAF (ứng dụng) → giám sát tập trung qua Syslog. Các lớp triển khai được trên Packet Tracer gồm ACL, Anti-Spoofing, Rate Limiting tầng 2/4 và IOS IPS.

## II.5. Giải pháp tấn công nội bộ – RBAC (Xuân)

Mục tiêu: mỗi người quản trị chỉ có đúng quyền cần thiết, mọi đăng nhập được xác thực, mã hóa và ghi log.

| Vai trò | Privilege | Quyền được phép |
|---|:---:|---|
| Quản trị viên (admin) | 15 | Toàn quyền cấu hình |
| Kỹ thuật viên (tech) | 7 | show, ping, traceroute, shutdown/no shutdown cổng, clear counters |
| Giám sát (monitor) | 1 | Chỉ xem trạng thái cơ bản |

```text
enable secret <mật-khẩu-mạnh>
service password-encryption
username admin   privilege 15 secret <mk1>
username tech    privilege 7  secret <mk2>
username monitor privilege 1  secret <mk3>
privilege exec level 7 show running-config
privilege exec level 7 configure terminal
privilege interface level 7 shutdown
privilege interface level 7 no shutdown
ip domain-name lab.local
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
login block-for 120 attempts 3 within 60
line vty 0 4
  login local
  transport input ssh
  exec-timeout 5 0
line console 0
  login local
  exec-timeout 5 0
```

- **Kiểm tra:** đăng nhập SSH bằng từng tài khoản, chạy `show privilege` và thử lệnh vượt quyền phải bị từ chối; xem log đăng nhập trên Syslog.

## II.6. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
|---|---|
| Packet Tracer không hỗ trợ đủ một số lệnh (DAI, uRPF, MPF trên ASA) | Kiểm tra bằng "?"; lệnh không hỗ trợ được trình bày ở mức thiết kế và ghi rõ hạn chế |
| Bật DAI/IP Source Guard dễ mất kết nối máy IP tĩnh | Khai ARP ACL / binding tĩnh trước, triển khai từng switch, bắt đầu từ tầng 1 tòa A |
| Native VLAN thay đổi gây lệch trunk | Đổi đồng thời ở cả hai đầu trunk, kiểm tra bằng show interfaces trunk |
