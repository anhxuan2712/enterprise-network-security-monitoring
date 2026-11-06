# BÁO CÁO TIẾN ĐỘ – GIAI ĐOẠN 3

**Triển khai giải pháp phòng chống tấn công cho hệ thống mạng Cisco (Packet Tracer)**

| Hạng mục | Nội dung |
|---|---|
| Đề tài | Phân tích và thiết kế hệ thống giám sát và quản lý nhật ký an toàn cho mạng doanh nghiệp |
| Giai đoạn | Giai đoạn 3 – Phòng chống tấn công bên ngoài và bên trong nội bộ |

## MỤC LỤC

- [1. Tổng quan tiến độ](#1-tổng-quan-tiến-độ)
- [2. Bảng ma trận phân chia nhiệm vụ](#2-bảng-ma-trận-phân-chia-nhiệm-vụ)
- [3. Giải pháp bảo vệ dịch vụ ARP khi bị tấn công](#3-giải-pháp-bảo-vệ-dịch-vụ-arp-khi-bị-tấn-công)
- [4. Giải pháp thật sự cần thiết cho từng kiểu tấn công](#4-giải-pháp-thật-sự-cần-thiết-cho-từng-kiểu-tấn-công)
- [5. Giải pháp tấn công nội bộ – RBAC (Xuân)](#5-giải-pháp-tấn-công-nội-bộ--rbac-xuân)
- [6. Khó khăn và hướng xử lý](#6-khó-khăn-và-hướng-xử-lý)

## 1. Tổng quan tiến độ

Giai đoạn 3 đã hoàn thành phần hạ tầng: IP quản trị, ACL trên ASA, Syslog và Port Security. Giai đoạn 3 tập trung vào bảo mật: xác định các kiểu tấn công theo từng lớp mạng, phân công người phụ trách và triển khai giải pháp phòng chống tương ứng.

| Hạng mục | Trạng thái |
|---|---|
| ACL OUTSIDE_IN trên ASA | Hoàn thành |
| Syslog tập trung | Hoàn thành |
| VLAN & trunk | Hoàn thành |
| IP quản trị switch | Hoàn thành |
| Port Security | Hoàn thành |
| DHCP Snooping | Hoàn thành |
| Chống tấn công lớp 2/3/4 + RBAC | Giai đoạn 3 |

## 2. Bảng ma trận phân chia nhiệm vụ

Phân công dựa trên danh sách tấn công đã thống nhất trong nhóm. Ký hiệu: ● = phụ trách chính (cấu hình, kiểm thử, viết báo cáo); ○ = hỗ trợ / phối hợp kiểm tra chéo;

### 2.1. Ma trận tổng hợp

| STT | Kiểu tấn công | Lớp | Nguyễn Anh Xuân | Nguyễn Phúc Vượng | Trí | Nhóm |
|:---:|---|---|:---:|:---:|:---:|---|
| 1 | Port Scan | Tầng 2 | ○ | ○ | ○ | Bên ngoài |
| 2 | DHCP Spoofing / Starvation | Tầng 2 | ○ | ● | ○ | Bên ngoài |
| 3 | VLAN Hopping | Tầng 2 | ○ | ● | ○ | Bên ngoài |
| 4 | ARP Spoofing | Tầng 2 | ○ | ○ | ● | Bên ngoài |
| 5 | IP Spoofing, ICMP Flood | Tầng 3 | ● | ○ | ○ | Bên ngoài |
| 6 | TCP SYN Flood, UDP Flood, DNS | Tầng 4 | ○ | ○ | ● | Bên ngoài |
| 7 | DDoS / SYN | Tầng 3–4 | ○ | ○ | ● | Bên ngoài |
| 8 | Phân quyền RBAC | Quản trị | ● | ○ | ○ | Nội bộ |

### 2.2. Chi tiết nhiệm vụ – Nguyễn Anh Xuân

| Nhiệm vụ | Giải pháp chính |
|---|---|
| IP Spoofing (tầng 3) | ACL chống giả mạo ở chiều vào outside; uRPF; IP Source Guard tại switch access |
| ICMP Flood / Smurf | Chỉ cho phép echo/echo-reply cần thiết; tắt directed-broadcast, redirects, unreachables |
| RBAC nội bộ | Tài khoản local theo cấp quyền (15/7/1), SSH, khóa đăng nhập sai, exec-timeout |
| Hỗ trợ | Kiểm tra chéo Port Scan, DHCP, VLAN Hopping, ARP, TCP/UDP flood – DNS, DDoS |

### 2.3. Chi tiết nhiệm vụ – Nguyễn Phúc Vượng

| Nhiệm vụ | Giải pháp chính |
|---|---|
| DHCP Spoofing / Starvation (tầng 2) | DHCP Snooping (đã bật): rà soát cổng trust/untrusted trên toàn bộ switch, giới hạn tốc độ gói DHCP, kết hợp Port Security chống cạn kiệt địa chỉ |
| VLAN Hopping (tầng 2) | Tắt DTP, đổi native VLAN sang VLAN rỗng, giới hạn allowed VLAN, shutdown cổng thừa vào VLAN cách ly |
| Hỗ trợ | Kiểm tra chéo Port Scan, ARP, IP Spoofing – ICMP, TCP/UDP flood – DNS, DDoS, RBAC |

### 2.4. Chi tiết nhiệm vụ – Trí

| Nhiệm vụ | Giải pháp chính |
|---|---|
| ARP Spoofing (tầng 2) | DAI dựa trên bảng DHCP Snooping, ARP ACL cho thiết bị IP tĩnh, rate limit ARP, giám sát Syslog |
| TCP SYN Flood (tầng 4) | Giới hạn kết nối nửa mở trên ASA, chỉ mở cổng dịch vụ cần thiết, giám sát Syslog |
| UDP Flood (tầng 4) | ACL chỉ cho phép UDP cần thiết (53, 514), chặn UDP khác từ outside |
| DNS | DNS chỉ phục vụ nội bộ, chặn truy vấn DNS từ outside, máy trạm chỉ dùng DNS nội bộ |
| DDoS / SYN | Phối hợp nhiều lớp: anti-spoofing, giới hạn kết nối, chặn UDP/ICMP thừa, Syslog |
| Hỗ trợ | Kiểm tra chéo Port Scan, DHCP, VLAN Hopping, RBAC |

## 3. Giải pháp bảo vệ dịch vụ ARP khi bị tấn công

### 3.1. Bản chất tấn công

ARP không có cơ chế xác thực: máy nào cũng có thể gửi gratuitous ARP tuyên bố "IP 192.168.x.1 có MAC là của tôi". Kẻ tấn công lợi dụng việc này để chèn vào giữa máy trạm và gateway (Man-in-the-Middle), nghe lén, sửa dữ liệu, hoặc làm tê liệt mạng (ARP flooding / DoS).

- **Dấu hiệu:** bảng ARP của máy trạm có 2 IP trùng một MAC; MAC của gateway thay đổi bất thường; mạng chập chờn, chậm.
- **Mục tiêu bị nhắm:** gateway các VLAN (192.168.2.1, 192.168.10.1…), máy chủ Syslog 192.168.10.50, máy chủ web 192.168.10.12.

### 3.2. Các giải pháp cần thiết (theo thứ tự ưu tiên)

| # | Giải pháp | Tác dụng | Điều kiện |
|:---:|---|---|---|
| 1 | Dynamic ARP Inspection (DAI) | Switch kiểm tra mọi gói ARP trên cổng untrusted, đối chiếu IP–MAC với bảng DHCP Snooping; gói giả bị hủy | Bắt buộc bật DHCP Snooping trước |
| 2 | ARP ACL cho thiết bị IP tĩnh | Khai báo cặp IP–MAC hợp lệ của gateway, máy chủ (không nhận IP từ DHCP) | Biết MAC thật của thiết bị |
| 3 | Trust đúng cổng uplink/trunk | Chỉ cổng nối router/switch phân phối là trusted, mọi cổng người dùng untrusted | Xác định đúng cổng uplink |
| 4 | Giới hạn tốc độ ARP (rate limit) | Cổng gửi quá số gói ARP/giây bị err-disable → chống ARP flood | Mặc định 15 pps trên cổng untrusted |
| 5 | ARP tĩnh cho gateway trên máy chủ quan trọng | Máy chủ không bị đầu độc ARP của gateway | Áp dụng cho số ít máy chủ |
| 6 | Port Security (đã có) | Giới hạn số MAC/cổng, hạn chế giả mạo MAC kèm ARP | Đã cấu hình 74 cổng |
| 7 | Giám sát qua Syslog | Gói ARP bị DAI hủy được ghi log về 192.168.10.50 | Syslog đã hoạt động |

### 3.3. Cấu hình mẫu (switch tầng 1 tòa A, VLAN 2 và 10)

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

### 3.4. Lưu ý khi triển khai

- Bật DAI khi chưa có DHCP Snooping hoặc chưa khai ARP ACL cho máy IP tĩnh sẽ làm mất kết nối của chính các máy đó (gateway, máy chủ, IP quản trị VLAN 2).
- Không đặt trust nhầm cho cổng người dùng; cổng trunk giữa các switch đều phải trust ở cả hai đầu.
- Packet Tracer không hỗ trợ đầy đủ mọi lệnh DAI ở mọi phiên bản/dòng switch. Kiểm tra bằng "ip arp inspection ?" trên thiết bị; nếu không hỗ trợ, dùng Port Security + ARP tĩnh + giám sát Syslog và ghi rõ hạn chế này trong báo cáo.

## 4. Giải pháp thật sự cần thiết cho từng kiểu tấn công

Chỉ chọn các biện pháp có tác dụng trực tiếp, triển khai được trên mô hình hiện tại và phù hợp với ACL/VLAN đã cấu hình.

### 4.1. Port Scan (trinh sát)

- **Nguy cơ:** kẻ tấn công dò cổng mở để tìm dịch vụ khai thác.
- **Giải pháp:** giữ nguyên chính sách "chặn mặc định" của ASA (implicit deny), chỉ mở 80/443 tới máy chủ web; tắt dịch vụ không dùng trên router/switch; ghi log gói bị chặn.

```text
access-list OUTSIDE_IN extended deny ip any any log
no ip http server
no cdp run                 ! trên cổng hướng ra ngoài: no cdp enable
```

### 4.2. DHCP Spoofing / Starvation (Vượng)

- **Giải pháp:** DHCP Snooping (đã bật trên mô hình), chỉ trust cổng nối DHCP server/uplink; giới hạn tốc độ gói DHCP; Port Security giới hạn số MAC để chống starvation.

```text
ip dhcp snooping
ip dhcp snooping vlan 2,10,21,31-33,41-44
interface fastEthernet 0/1
  ip dhcp snooping trust
interface range fastEthernet 0/2 - 24
  ip dhcp snooping limit rate 10
```

### 4.3. VLAN Hopping (Vượng)

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

### 4.4. ARP Spoofing (Trí)

Xem chi tiết mục 3: DAI + DHCP Snooping + ARP ACL cho IP tĩnh + rate limit + Syslog.

### 4.5. IP Spoofing (Xuân)

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

### 4.6. ICMP Flood / Smurf (Xuân)

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

### 4.7. TCP SYN Flood (Trí)

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

### 4.8. UDP Flood (Trí)

- **Giải pháp:** chính sách whitelist: chỉ cho phép UDP thật sự cần (514 Syslog từ 10.255.0.1, 53 tới máy chủ DNS nếu public), chặn toàn bộ UDP còn lại từ outside; giới hạn số kết nối UDP trên ASA.

```text
access-list OUTSIDE_IN extended permit udp host 10.255.0.1 host 192.168.10.50 eq 514
access-list OUTSIDE_IN extended deny udp any any log
```

### 4.9. Tấn công DNS (Trí)

- **Nguy cơ:** DNS amplification (lợi dụng máy chủ DNS mở), DNS spoofing chuyển hướng người dùng.
- **Giải pháp:** máy chủ DNS chỉ trả lời truy vấn từ mạng nội bộ; ACL chỉ cho phép UDP/TCP 53 tới đúng máy chủ DNS; máy trạm chỉ được dùng DNS nội bộ (chặn DNS ra ngoài trực tiếp); bản ghi DNS quan trọng cấu hình tĩnh.

```text
! Router/ASA phía trong: chỉ máy chủ DNS được truy vấn ra ngoài
access-list INSIDE_OUT extended permit udp host <IP-DNS> any eq domain
access-list INSIDE_OUT extended deny udp any any eq domain log
access-list INSIDE_OUT extended permit ip any any
```

### 4.10. DDoS / SYN (Trí)

- **Giải pháp:** kết hợp nhiều lớp – anti-spoofing (4.5), giới hạn kết nối (4.7), chặn UDP/ICMP thừa (4.6, 4.8), Port Security và rate limit DHCP/ARP ở switch, giám sát tập trung Syslog để phát hiện sớm; bổ sung IDS/IPS và các giải pháp mở rộng ở mục 4.11.

### 4.11. Giải pháp chống DDoS bổ sung (Trí)

Đối chiếu với 7 giải pháp chống DDoS phổ biến, mô hình đã có Firewall/ACL và Anti-Spoofing; Rate Limiting mới có ở tầng 2 và tầng 4. Mục này bổ sung IDS/IPS (triển khai được trên Packet Tracer) và các giải pháp ở mức thiết kế cho hệ thống thực tế.

#### a) Các giải pháp đã có trên mô hình

- **Firewall và ACL:** ASA chặn mặc định, chỉ mở 80/443 tới máy chủ web; whitelist UDP 514/53; chặn ICMP thừa (mục 4.1, 4.6, 4.8, 4.9).
- **Anti-Spoofing:** ACL ANTI-SPOOF, uRPF, IP Source Guard, tắt directed-broadcast (mục 4.5, 4.6).
- **Rate Limiting:** giới hạn gói DHCP (10 pps), ARP (15 pps) trên switch; giới hạn kết nối nửa mở trên ASA (mục 4.7). Phần giới hạn request HTTP theo IP được bổ sung ở mục c.

#### b) IDS/IPS – IOS IPS trên router biên (triển khai được)

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

## 5. Giải pháp tấn công nội bộ – RBAC (Xuân)

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

## 6. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
|---|---|
| Packet Tracer không hỗ trợ đủ một số lệnh (DAI, uRPF, MPF trên ASA) | Kiểm tra bằng "?"; lệnh không hỗ trợ được trình bày ở mức thiết kế và ghi rõ hạn chế |
| Bật DAI/IP Source Guard dễ mất kết nối máy IP tĩnh | Khai ARP ACL / binding tĩnh trước, triển khai từng switch, bắt đầu từ tầng 1 tòa A |
| Native VLAN thay đổi gây lệch trunk | Đổi đồng thời ở cả hai đầu trunk, kiểm tra bằng show interfaces trunk |
