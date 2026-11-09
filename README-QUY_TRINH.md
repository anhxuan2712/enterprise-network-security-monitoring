# README – Phân tích và thiết kế an toàn mạng

Tài liệu tổng hợp mô hình, cấu hình đã quan sát và kết quả kiểm thử của lab Cisco ASA + Cisco Switching + Wazuh SIEM + Ubuntu Web Server.

> Quy ước: IP/cấu hình/kết quả bên dưới lấy từ thông tin và ảnh kiểm tra đã cung cấp. Mục nào chưa có bằng chứng được ghi rõ “chưa xác nhận”. Không đưa credential hoặc Telegram Bot Token thật vào tài liệu.

## 1. Mục tiêu

- Phân đoạn mạng thành ADMIN, USERS và SERVERS.
- Dùng Cisco ASA làm gateway liên VLAN và áp dụng ACL.
- Thu thập Syslog từ ASA và Cisco Switch về Wazuh.
- Phát hiện thay đổi cấu hình, interface down, ASA ACL deny và Port Security violation.
- Gửi cảnh báo Wazuh sang Telegram.
- Giám sát Ubuntu Web Server và Windows bằng Wazuh Agent.

## 2. Sơ đồ logic

```text
                         External Connector (upstream)
                                   |
                                Router1
                                   |
                             ASA Gi0/1 (outside)
                                Cisco ASAv
                             ASA Gi0/0 (inside)
                                   |
                              CORE_SW E0/0
                              (802.1Q trunk)
                               /         \
                    CORE_SW E0/1         CORE_SW E0/2
                         |                    |
                    ACCESS_SW1           ACCESS_SW2
                      E0/0                  E0/0
                    /      \              /      \
          ADMIN / Connector1  USERS / C2  SERVERS / C3  PC Test
              Wazuh Server    endpoint    Ubuntu Web    Port Security
             10.10.10.10       (TBD)      10.10.30.10   (demo only)
```

Phân vùng theo thiết kế đã xác nhận:
- External Connector1: ADMIN, có Wazuh Server.
- External Connector2: USERS, hiện chưa xác nhận endpoint cố định.
- External Connector3: SERVERS, có Ubuntu Web Server.
- PC Test Port Security là node riêng, chỉ bật khi demo; không thuộc topology vận hành thường trực.

**Giới hạn node CML Free:** topology nền gồm Router1, ASA, CORE_SW, ACCESS_SW1 và ACCESS_SW2 (5 node thiết bị). Khi cần bật PC Test Port Security như node thứ sáu, có thể tắt Router1 nếu không cần kết nối upstream/Internet trong kịch bản đó. Tắt Router1 không thay thế gateway liên VLAN; ASA vẫn đảm nhiệm gateway cho VLAN 10/20/30. Không tắt Router1 nếu bài test đang cần đường upstream qua `10.255.255.1`.

Sơ đồ trên thể hiện vùng kết nối logic. Tên mô tả port trên switch (`WAZUH_SERVER_VLAN1`, `USERS_DEVICE`, `SERVER_DEVICE`, `ADMIN_DEVICE`, `test`) được ghi nhận từ `show interfaces status`; không đủ bằng chứng để khẳng định từng External Connector được nối trực tiếp vào đúng port access có tên mô tả đó ở mọi trạng thái lab.
## 3. Kế hoạch địa chỉ IP

### 3.1 VLAN và gateway ASA

| VLAN | Tên | Network | Gateway ASA | Security level |
|---:|---|---|---|---:|
| 10 | ADMIN | 10.10.10.0/24 | 10.10.10.1 | 100 |
| 20 | USERS | 10.10.20.0/24 | 10.10.20.1 | 80 |
| 30 | SERVERS | 10.10.30.0/24 | 10.10.30.1 | 70 |

ASA outside: `10.255.255.2/30`; next-hop `10.255.255.1` đã được xác nhận là Router1 Ethernet0/1. Router1 Ethernet0/1 dùng `10.255.255.1/30`; Router1 Ethernet0/0 nhận `192.168.255.118/24` qua DHCP từ upstream CML.

### 3.2 Thiết bị và IP

| Thiết bị | Interface/VLAN | IP |
|---|---|---|
| ASA | ADMIN / VLAN 10 | 10.10.10.1/24 |
| ASA | USERS / VLAN 20 | 10.10.20.1/24 |
| ASA | SERVERS / VLAN 30 | 10.10.30.1/24 |
| CORE_SW | VLAN 10 | 10.10.10.2/24 |
| ACCESS_SW1 | VLAN 10 | 10.10.10.3/24 |
| ACCESS_SW2 | VLAN 10 | 10.10.10.4/24 |
| Wazuh Server | ADMIN | 10.10.10.10/24 |
| Ubuntu Web Server | ens37 / VLAN 30 | 10.10.30.10/24 |
| Ubuntu Web Server | ens33 / management-NAT | 192.168.181.129/24 |
| Ubuntu Web Server | tailscale0 | 100.107.50.61/32 |
| Wazuh Server | management/NAT (đã quan sát) | 192.168.181.20/24 |

Ubuntu Web Server routing đã quan sát:

```text
default via 192.168.181.2 dev ens33
10.10.10.0/24 via 10.10.30.1 dev ens37
10.10.20.0/24 via 10.10.30.1 dev ens37
10.10.30.0/24 dev ens37 src 10.10.30.10
192.168.181.0/24 dev ens33 src 192.168.181.129
```

Wazuh Server có hai địa chỉ thuộc hai mạng khác nhau: `10.10.10.10` trong VLAN ADMIN và `192.168.181.20` trên mạng quản lý/NAT. Không nhầm hai địa chỉ này.

## 4. Kết nối switch

| Switch | Port | Kết nối | Mode/VLAN |
|---|---|---|---|
| CORE_SW | E0/0 | ASA Gi0/0 (inside trunk) | Trunk 10,20,30 |
| CORE_SW | E0/1 | ACCESS_SW1 E0/0 | Trunk 10,20,30 |
| CORE_SW | E0/2 | ACCESS_SW2 E0/0 | Trunk 10,20,30 |
| ACCESS_SW1 | E0/0 | CORE_SW E0/1 | Trunk 10,20,30 |
| ACCESS_SW1 | E0/1 | WAZUH_SERVER_VLAN1 (theo port description) | Access VLAN 10 |
| ACCESS_SW1 | E0/2 | USERS_DEVICE (theo port description) | Access VLAN 20 |
| ACCESS_SW1 | E0/3 | SERVER_DEVICE (theo port description) | Access VLAN 30 |
| ACCESS_SW2 | E0/0 | CORE_SW E0/2 | Trunk 10,20,30 |
| ACCESS_SW2 | E0/1 | ADMIN_DEVICE (theo port description) | Access VLAN 10 |
| ACCESS_SW2 | E0/2 | Port Security test | Access VLAN 20 |
| ACCESS_SW2 | E0/3 | SERVER_DEVICE (theo port description) | Access VLAN 30 |

CORE_SW E0/3 được thấy ở VLAN 1 trong output `show vlan brief`. `show interfaces status` xác nhận E0/0 là trunk `testlog`, E0/1 và E0/2 là trunk tới hai access switch. Các cổng trunk có native VLAN 1 và allowed VLAN 10,20,30.

## 5. Router1 – Upstream Router

### 5.1 Interface và định tuyến

Thông tin đã xác nhận từ ảnh `show ip interface brief`, `show running-config` và `show ip route`:

| Interface | Vai trò | IP | Trạng thái |
|---|---|---|---|
| Ethernet0/0 | Upstream / CML NAT | `192.168.255.118/24` (DHCP) | up/up |
| Ethernet0/1 | Transit đến ASA outside | `10.255.255.1/30` | up/up |
| Ethernet0/2 | Không sử dụng | — | administratively down |
| Ethernet0/3 | Không sử dụng | — | administratively down |

Các route đã quan sát:

```cisco
ip route 0.0.0.0 0.0.0.0 192.168.255.1
ip route 10.10.10.0 255.255.255.0 10.255.255.2
ip route 10.10.20.0 255.255.255.0 10.255.255.2
ip route 10.10.30.0 255.255.255.0 10.255.255.2
```

Bảng định tuyến hiển thị default route qua `192.168.255.1`, ba route tĩnh đến các VLAN qua ASA `10.255.255.2`, và mạng transit `10.255.255.0/30` kết nối trực tiếp.

### 5.2 NAT và Syslog

Cấu hình đã quan sát:

```cisco
interface Ethernet0/0
 description TO-VMWARE-EXTERNAL
 ip address dhcp
 ip nat outside

interface Ethernet0/1
 description LINK_TO_ASAV_OUTSIDE
 ip address 10.255.255.1 255.255.255.252
 ip nat inside

ip access-list standard 10
 10 permit 10.10.10.0 0.0.0.255
 20 permit 10.10.20.0 0.0.0.255
 30 permit 10.10.30.0 0.0.0.255
 40 permit 10.255.255.0 0.0.0.3

ip nat inside source list 10 interface Ethernet0/0 overload
logging host 192.168.181.20
```

Ảnh running-config xác nhận NAT overload được thực hiện trên Router1, dùng ACL standard 10 và địa chỉ interface Ethernet0/0. Router1 gửi Syslog tới Wazuh Server management IP `192.168.181.20`, không phải IP ADMIN `10.10.10.10`.

### 5.3 SSH và giới hạn bằng chứng

Running-config hiển thị `ip ssh bulk-mode 131072`, `line vty 0 4`, `login` và `transport input ssh`. Ảnh chưa thể hiện đầy đủ cấu hình local username/secret. Chưa có bằng chứng kiểm thử đăng nhập SSH thành công/thất bại trên Router1; Rule Wazuh `100102` vì vậy vẫn chưa nghiệm thu.

### 5.4 Lệnh tham chiếu

```cisco
show ip interface brief
show ip route
show running-config
show logging
```

## 6. Cisco ASA Firewall

### 5.1 Interface

Cấu hình interface đã quan sát:

```cisco
interface GigabitEthernet0/0
 nameif inside
 security-level 100
 no ip address

interface GigabitEthernet0/0.10
 vlan 10
 nameif ADMIN
 security-level 100
 ip address 10.10.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 vlan 20
 nameif USERS
 security-level 80
 ip address 10.10.20.1 255.255.255.0

interface GigabitEthernet0/0.30
 vlan 30
 nameif SERVERS
 security-level 70
 ip address 10.10.30.1 255.255.255.0

interface GigabitEthernet0/1
 nameif outside
 security-level 0
 ip address 10.255.255.2 255.255.255.252
```

Default route đã quan sát:

```cisco
route outside 0.0.0.0 0.0.0.0 10.255.255.1
```

ASA logging đã được cấu hình/quan sát:

```cisco
logging enable
logging timestamp
logging trap informational
logging host ADMIN 10.10.10.10
```

`show nat` đã được chạy nhưng ảnh cung cấp không hiển thị nội dung NAT. Vì vậy README không kết luận NAT đã được cấu hình hoặc nghiệm thu.

### 5.2 ACL gắn trên interface

```cisco
access-group ADMIN_IN in interface ADMIN
access-group USERS_IN in interface USERS
access-group SERVERS_IN in interface SERVERS
access-group OUTSIDE_IN in interface outside
```

Các ACE quan trọng nhìn thấy trong `show access-list`:

```cisco
! ADMIN_IN
access-list ADMIN_IN extended permit tcp 10.10.10.0 255.255.255.0 host 10.10.30.10 eq ssh
access-list ADMIN_IN extended permit tcp 10.10.10.0 255.255.255.0 host 10.10.30.10 eq www
access-list ADMIN_IN extended permit tcp 10.10.10.0 255.255.255.0 host 10.10.30.10 eq https
access-list ADMIN_IN extended permit icmp 10.10.10.0 255.255.255.0 host 10.10.30.10
access-list ADMIN_IN extended deny ip 10.10.10.0 255.255.255.0 10.10.30.0 255.255.255.0 log informational interval 300
access-list ADMIN_IN extended permit ip 10.10.10.0 255.255.255.0 any

! USERS_IN
access-list USERS_IN extended permit tcp 10.10.20.0 255.255.255.0 host 10.10.30.10 eq www
access-list USERS_IN extended permit tcp 10.10.20.0 255.255.255.0 host 10.10.30.10 eq https
access-list USERS_IN extended deny ip 10.10.20.0 255.255.255.0 10.10.30.0 255.255.255.0 log informational interval 300
access-list USERS_IN extended permit icmp 10.10.20.0 255.255.255.0 host 10.10.10.10
access-list USERS_IN extended deny ip 10.10.20.0 255.255.255.0 10.10.10.0 255.255.255.0 log informational interval 300
access-list USERS_IN extended permit ip 10.10.20.0 255.255.255.0 any

! SERVERS_IN
access-list SERVERS_IN extended permit tcp host 10.10.30.10 host 10.10.10.10 eq 1514
access-list SERVERS_IN extended permit tcp host 10.10.30.10 host 10.10.10.10 eq 1515
access-list SERVERS_IN extended deny ip 10.10.30.0 255.255.255.0 10.10.10.0 255.255.255.0 log informational interval 300
access-list SERVERS_IN extended permit ip 10.10.30.0 255.255.255.0 any

! OUTSIDE_IN
access-list OUTSIDE_IN extended permit icmp host 10.255.255.1 host 10.10.10.1
access-list OUTSIDE_IN extended permit icmp host 10.255.255.1 host 10.10.20.1
access-list OUTSIDE_IN extended permit icmp host 10.255.255.1 host 10.10.30.1
```

Lưu ý: `show access-list` còn hiển thị ACL tên `SERVER_IN` với các ACE tương tự. Đây là ACL tạo nhầm, người triển khai quyết định giữ lại. Output `show running-config access-group` chỉ xác nhận `SERVERS_IN` được gắn vào interface SERVERS; không ghi `SERVER_IN` là ACL đang áp dụng.

### 5.3 Kiểm thử ASA packet-tracer

Ví dụ lệnh:

```cisco
packet-tracer input USERS tcp 10.10.20.100 49152 10.10.30.10 80
packet-tracer input USERS tcp 10.10.20.100 49152 10.10.30.10 443
packet-tracer input USERS tcp 10.10.20.100 49152 10.10.30.10 22
packet-tracer input ADMIN tcp 10.10.10.100 49152 10.10.30.10 22
packet-tracer input ADMIN tcp 10.10.10.100 49152 10.10.30.10 80
packet-tracer input ADMIN tcp 10.10.10.100 49152 10.10.30.10 443
packet-tracer input SERVERS tcp 10.10.30.10 49152 10.10.10.10 1514
packet-tracer input SERVERS tcp 10.10.30.10 49152 10.10.10.10 1515
```

ACL được xử lý theo thứ tự từ trên xuống; các permit cụ thể phải đứng trước deny subnet tương ứng. Ví dụ, USERS được permit TCP/80 và TCP/443 tới Web Server trước khi rule deny toàn bộ USERS tới subnet SERVERS được xét. ICMP từ USERS tới Wazuh cũng được permit trước rule deny USERS tới ADMIN.

Kết quả đã quan sát:

| Nguồn | Đích | Port | Kết quả |
|---|---|---:|---|
| USERS 10.10.20.100 | Web 10.10.30.10 | TCP/80 | ALLOW |
| USERS 10.10.20.100 | Web 10.10.30.10 | TCP/443 | ALLOW |
| USERS 10.10.20.100 | Web 10.10.30.10 | TCP/22 | DROP |
| ADMIN 10.10.10.100 | Web 10.10.30.10 | TCP/22 | ALLOW |
| ADMIN 10.10.10.100 | Web 10.10.30.10 | TCP/80,443 | ALLOW |
| SERVERS 10.10.30.10 | Wazuh 10.10.10.10 | TCP/1514,1515 | ALLOW |

## 7. Cisco Switch

### 6.1 VLAN và trunk

Cấu hình VLAN trên switch:

```cisco
vlan 10
 name ADMIN
vlan 20
 name USERS
vlan 30
 name SERVERS
```

Cấu hình trunk đại diện (đối chiếu với output `show interfaces trunk`; đây là mẫu cấu hình, không khẳng định từng lệnh đã được nhập giống hệt trên cả ba switch):

```cisco
interface Ethernet0/0
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

Các trunk đã xác nhận:
- CORE_SW E0/0, E0/1, E0/2.
- ACCESS_SW1 E0/0.
- ACCESS_SW2 E0/0.

Native VLAN trên các trunk là VLAN 1. Nếu image IOS không hỗ trợ `switchport trunk encapsulation dot1q`, chỉ cấu hình `switchport mode trunk` và kiểm tra lại bằng `show interfaces trunk`.

### 6.2 Access port và SVI quản trị

Các dòng sau là cấu hình access/SVI tương ứng với phân hoạch đã quan sát; tên interface và địa chỉ cần giữ đúng theo từng switch:

```cisco
! Access port mẫu
interface Ethernet0/1
 switchport mode access
 switchport access vlan 10

interface Ethernet0/2
 switchport mode access
 switchport access vlan 20

interface Ethernet0/3
 switchport mode access
 switchport access vlan 30

! CORE_SW
interface Vlan10
 ip address 10.10.10.2 255.255.255.0
 no shutdown

! ACCESS_SW1
interface Vlan10
 ip address 10.10.10.3 255.255.255.0
 no shutdown

! ACCESS_SW2
interface Vlan10
 ip address 10.10.10.4 255.255.255.0
 no shutdown
```

### 6.3 STP

CORE_SW báo `rapid-pvst mode`. Đã kiểm tra:

```cisco
show spanning-tree summary
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
```

Output cho thấy các trunk ở trạng thái forwarding; root/designated port khác nhau tùy VLAN.

### 6.4 Port Security trên ACCESS_SW2 E0/2

Cấu hình thực tế:

```cisco
interface Ethernet0/2
 description test
 switchport access vlan 20
 switchport mode access
 switchport port-security mac-address sticky
 switchport port-security
 spanning-tree portfast
```

Thông số kiểm tra đã ghi nhận:
- Port Security: Enabled.
- Violation mode: Shutdown.
- Maximum MAC addresses: 1.
- Sticky MAC: 1.
- Security Violation Count: 2.
- Port Status: Secure-shutdown.
- `show interfaces status`: E0/2 `err-disabled`.

Lệnh kiểm tra:

```cisco
show port-security
show port-security interface Ethernet0/2
show running-config interface Ethernet0/2
show interfaces status
show logging
```

Khi MAC thiết bị test thay đổi, switch ghi `%PORT_SECURITY-2-PSECURE_VIOLATION` và `%PM-4-ERR_DISABLE`, tăng violation counter và khóa E0/2. Wazuh nhận sự kiện và kích hoạt Rule `100103`. Ảnh kiểm tra cuối cho thấy E0/2 vẫn ở `Secure-shutdown`/`err-disabled`, violation count là 2; vì vậy đây là trạng thái sau test, chưa phải trạng thái port đã phục hồi để dùng bình thường.

ACCESS_SW1 E0/2 đã được gỡ cấu hình Port Security; trạng thái cuối không bật Port Security trên ACCESS_SW1. Trước lần demo tiếp theo, cần kiểm tra và phục hồi trạng thái err-disabled trên ACCESS_SW2 E0/2 theo đúng quy trình của IOS image (xử lý nguyên nhân/MAC sticky trước, sau đó shutdown/no shutdown hoặc clear port-security theo cú pháp image hỗ trợ).

### 6.5 Syslog trên Cisco Switch

Các lệnh cấu hình Syslog IOS tương ứng với đích đã kiểm tra:

```cisco
service timestamps log datetime msec
logging host 10.10.10.10
logging trap informational
```

Một số output `show logging` hiển thị Syslog đang enabled, trap level informational, gửi tới `10.10.10.10` qua UDP/514; output cũng hiển thị `Logging Source-Interface: Vlan10` trên thiết bị được chụp. Do chưa có `show running-config` đầy đủ của từng switch, không khẳng định mọi switch đều có cùng toàn bộ dòng cấu hình hoặc cùng source-interface.

Kiểm tra:

```cisco
show logging
show running-config | include logging
```

Không dùng `logging enable` trong phần cấu hình Cisco IOS switch; đó là lệnh đã xuất hiện trong cấu hình ASA, không phải bằng chứng cấu hình switch.
## 8. Ubuntu Web Server

### 7.1 Network

```text
ens33       192.168.181.129/24
ens37       10.10.30.10/24
tailscale0  100.107.50.61/32
```

Gateway VLAN SERVERS: `10.10.30.1`.

### 7.2 Nginx

Lệnh:

```bash
ip -br addr
ip route
systemctl is-active nginx
curl -I http://127.0.0.1
curl -I http://10.10.30.10
```

Kết quả:
- Nginx `active`.
- HTTP `200 OK`.
- Server header `nginx/1.28.3 (Ubuntu)`.

### 7.3 Wazuh Agent và kết nối

```bash
systemctl is-active wazuh-agent
ping -c 4 10.10.30.1
nc -zvw3 10.10.10.10 1514
nc -zvw3 10.10.10.10 1515
```

Kết quả:
- Agent service `active`.
- Ping gateway 4/4.
- TCP 1514 và 1515 đều `succeeded`.

Trên Manager, `sudo /var/ossec/bin/agent_control -lc` hiển thị:

```text
ID: 000, Name: wazuh-server (server), IP: 127.0.0.1, Active/Local
ID: 002, Name: ubuntu-server, IP: any, Active
ID: 003, Name: windows-local, IP: any, Active
```

## 9. Wazuh Server và Syslog

### 8.1 Luồng xử lý log – phân biệt các port

```text
Cisco ASA / Cisco Switch
        |
        | Syslog UDP/514
        v
Wazuh Server – rsyslog
        |
        | ghi file
        v
/var/log/network/network.log
        |
        | Wazuh localfile logcollector
        v
Wazuh analysisd – decoder/rules
        |
        v
/var/ossec/logs/alerts/alerts.json
        |
        v
custom-telegram integration
        |
        v
Telegram Bot
```

Đây là hai luồng khác nhau, không được nhầm port:

- **Thiết bị Cisco gửi Syslog:** UDP/514 tới rsyslog trên Wazuh Server.
- **Wazuh Agent kết nối Manager:** TCP/1514; TCP/1515 được dùng cho đăng ký/enrollment agent.

Cisco ASA và switch trong mô hình này không được chứng minh là đã cài Wazuh Agent trực tiếp. Vì vậy alert do log Cisco tạo ra có `agent.name` là `wazuh-server`, `agent.id` là `000`, `agent.ip` có thể là `N/A`. Đây là hành vi phù hợp với cách thu thập log qua rsyslog rồi Wazuh đọc file cục bộ, không phải dấu hiệu Cisco thiết bị đã trở thành Wazuh Agent.

### 8.2 Dịch vụ

Wazuh version đã triển khai: 4.14.9. OS ban đầu: Ubuntu Server 24.04.3 LTS.

Địa chỉ:
- ADMIN: `10.10.10.10`.
- Management/NAT: `192.168.181.20/24`.
- Dashboard đã truy cập: `https://192.168.181.20:443`.

Kiểm tra:

```bash
sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard rsyslog
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard rsyslog --no-pager
sudo /var/ossec/bin/agent_control -lc
```

Bốn service đều trả về `active` ở lần kiểm tra cuối.

### 8.3 rsyslog UDP/514

Wazuh Server routing đã quan sát trên `ip route`:

```text
default via 192.168.181.2 dev ens33
10.10.10.0/24 dev ens37 proto kernel scope link src 10.10.10.10
10.10.20.0/24 via 10.10.10.1 dev ens37 proto static
10.10.30.0/24 via 10.10.10.1 dev ens37 proto static
192.168.181.0/24 dev ens33 proto kernel scope link src 192.168.181.20
```

Trong `/etc/rsyslog.conf`:

```rsyslog
module(load="imudp")
input(type="imudp" port="514")
```

File `/etc/rsyslog.d/10-network.conf`:

```rsyslog
if ($inputname == "imudp") then {
    action(
        type="omfile"
        file="/var/log/network/network.log"
    )
    stop
}
```

Tạo thư mục và quyền:

```bash
sudo mkdir -p /var/log/network
sudo chown syslog:adm /var/log/network
sudo chmod 750 /var/log/network
```

Kiểm tra/restart:

```bash
sudo rsyslogd -N1
sudo systemctl restart rsyslog
sudo systemctl is-active rsyslog
sudo ss -lunp | grep ':514'
sudo tcpdump -ni any udp port 514
sudo tail -f /var/log/network/network.log
```

Syslog từ ASA và switch đã tới Wazuh UDP/514. Đã thấy ASA `%ASA-6-106100` và switch `%PORT_SECURITY-2-PSECURE_VIOLATION`.

### 8.4 Wazuh đọc network log

Trong `/var/ossec/etc/ossec.conf`:

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/network/network.log</location>
</localfile>
```

Sau khi thêm cấu hình, Wazuh Manager đã được restart.

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
sudo systemctl restart wazuh-manager
sudo systemctl is-active wazuh-manager
```

## 10. Wazuh Telegram Integration

Trong `/var/ossec/etc/ossec.conf`, integration có dạng:

```xml
<integration>
  <name>custom-telegram</name>
  <hook_url>TELEGRAM_CHAT_ID</hook_url>
  <api_key>TELEGRAM_BOT_TOKEN</api_key>
  <level>7</level>
  <alert_format>json</alert_format>
</integration>
```

- `hook_url` được script sử dụng làm Chat ID.
- `api_key` là Bot Token; không ghi token thật vào README.
- Ngưỡng Level 7: gửi alert Level 7 trở lên.
- Telegram đã nhận các Rule `100100`, `100101`, `100103`, `100500`.

Script: `/var/ossec/integrations/custom-telegram`.

Quyền đã thiết lập:

```bash
sudo chown root:wazuh /var/ossec/integrations/custom-telegram
sudo chmod 750 /var/ossec/integrations/custom-telegram
```

Script Python 3 đọc alert JSON, trích rule/agent/timestamp, tạo message và gửi tới Telegram Bot API `sendMessage`. Log:

```text
/var/ossec/logs/integrations.log
```

Kiểm tra:

```bash
sudo tail -f /var/ossec/logs/integrations.log
```

**Bảo mật:** Bot Token thật đã xuất hiện trong ảnh terminal được gửi. Hãy thu hồi token cũ bằng BotFather, tạo token mới, cập nhật `api_key`, rồi kiểm thử lại. Không commit token hoặc credential vào Git.

## 11. Wazuh Custom Rules

File: `/var/ossec/etc/rules/local_rules.xml`.

Các rule đã quan sát:

```xml
<group name="local,syslog,sshd,">

  <rule id="100001" level="5">
    <if_sid>5716</if_sid>
    <srcip>1.1.1.1</srcip>
    <description>sshd: authentication failed from IP 1.1.1.1</description>
    <group>authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5,</group>
  </rule>

  <rule id="100100" level="10">
    <if_sid>4721</if_sid>
    <description>SECURITY: Cisco Network Device configuration changed</description>
    <group>cisco_config_changed,network_security,</group>
  </rule>

  <rule id="100101" level="8">
    <if_sid>4715</if_sid>
    <match>LINEPROTO-5-UPDOWN</match>
    <regex>changed state to down</regex>
    <description>SECURITY: Cisco Network Device interface DOWN</description>
    <group>cisco_interface_down,network_security,</group>
  </rule>

  <rule id="100102" level="10">
    <if_sid>4724</if_sid>
    <description>SECURITY: Cisco Router failed login attempt</description>
    <group>cisco_authentication_failed,network_security,</group>
  </rule>

  <rule id="100200" level="7">
    <match>WazuhTest</match>
    <description>SECURITY: Windows Event Log test detected</description>
    <group>windows_test,windows_event,</group>
  </rule>

  <rule id="100500" level="10">
    <if_sid>64005</if_sid>
    <match>6-106100</match>
    <description>SECURITY: Cisco ASA denied network traffic by ACL</description>
    <group>firewall_denied,network_security,</group>
  </rule>

  <rule id="100103" level="10">
    <if_sid>4712</if_sid>
    <match>PORT_SECURITY-2-PSECURE_VIOLATION</match>
    <description>SECURITY: Unauthorized MAC address detected - Cisco Switch Port Security Violation</description>
    <group>port_security_violation,network_security,</group>
  </rule>

</group>
```

Kiểm tra:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
sudo /var/ossec/bin/wazuh-logtest
sudo grep -E '"id":"100100"|"id":"100101"|"id":"100103"|"id":"100500"' \
  /var/ossec/logs/alerts/alerts.json | tail -n 20
```

Rule 100001, 100102 và 100200 có trong file cấu hình; dữ liệu hiện có chưa chứng minh các kịch bản tương ứng đã được nghiệm thu. Không mô tả chúng là test PASS. Rule `100001` đang giới hạn `<srcip>1.1.1.1</srcip>`, nên đây là rule thử nghiệm cho một IP cụ thể, không phải rule tổng quát phát hiện mọi SSH brute-force. Ngoài ra, các rule hiện được đặt chung trong group cha `local,syslog,sshd`; điều này làm các rule Cisco/Windows kế thừa nhãn group `sshd`. Nên tách group theo nguồn sự kiện trong lần refactor riêng, sau khi backup và kiểm thử lại, không sửa vội cấu hình đang chạy.

## 12. Kịch bản kiểm thử

| ID | Kịch bản | Lệnh/cách kiểm thử | Kết quả |
|---|---|---|---|
| T01 | Nginx hoạt động | `curl -I http://10.10.30.10` | PASS – HTTP 200 |
| T02 | Ubuntu đến ASA | `ping -c 4 10.10.30.1` | PASS – 4/4 |
| T03 | Ubuntu đến Wazuh 1514 | `nc -zvw3 10.10.10.10 1514` | PASS |
| T04 | Ubuntu đến Wazuh 1515 | `nc -zvw3 10.10.10.10 1515` | PASS |
| T05 | Ubuntu Agent kết nối | `agent_control -lc` | PASS – Active |
| T06 | USERS đến Web HTTP | packet-tracer TCP/80 | PASS – ALLOW |
| T07 | USERS đến Web HTTPS | packet-tracer TCP/443 | PASS – ALLOW |
| T08 | USERS SSH đến Web | packet-tracer TCP/22 | PASS – DROP theo policy |
| T09 | ADMIN SSH đến Web | packet-tracer TCP/22 | PASS – ALLOW |
| T10 | SERVERS đến Wazuh | packet-tracer TCP/1514,1515 | PASS – ALLOW |
| T11 | ASA ACL deny | Tạo traffic bị ACL deny | PASS – Rule 100500 + Telegram |
| T12 | Cisco config change | Thay đổi cấu hình thiết bị | PASS – Rule 100100 + Telegram |
| T13 | Interface down | Shutdown interface để tạo log | PASS – Rule 100101 + Telegram |
| T14 | Port Security violation | Thay MAC thiết bị test ACCESS_SW2 E0/2 | PASS – violation count 2, Rule 100103 + Telegram |
| T15 | Windows Agent | `agent_control -lc` | PASS – Active |
| T16 | Router failed login | Rule 100102 | Chưa có bằng chứng PASS |
| T17 | Windows custom event | Rule 100200 | Chưa có bằng chứng PASS |

### Bằng chứng alert

Telegram đã nhận:
- `100100`, Level 10: Cisco Network Device configuration changed.
- `100101`, Level 8: Cisco Network Device interface DOWN.
- `100103`, Level 10: Cisco Switch Port Security Violation.
- `100500`, Level 10: Cisco ASA denied network traffic by ACL.

ASA deny được kiểm tra trong `/var/ossec/logs/alerts/alerts.json`, decoder `cisco-asa`, log ID `6-106100`. Nội dung log mẫu đã quan sát có dạng `ASA-6-106100: access-list USERS_IN denied tcp ...`, tức ASA ghi nhận gói tin bị ACL `USERS_IN` từ chối. Port Security violation được ghi trong `/var/log/network/network.log`, decoder `cisco-ios`; `wazuh-logtest` trả Rule ID `100103`, Level 10.

Do log Cisco được Wazuh đọc từ file cục bộ trên Manager, Telegram hiển thị Agent `wazuh-server` / ID `000` / IP `N/A` cho các sự kiện này. Không nên ghi trong báo cáo rằng cảnh báo được gửi trực tiếp từ một Wazuh Agent cài trên ASA hoặc switch.

## 13. Lệnh nghiệm thu tổng hợp

### Wazuh Server

```bash
ip -br addr
ip route
sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard rsyslog
sudo /var/ossec/bin/agent_control -lc
sudo ss -lunp | grep ':514'
sudo tcpdump -ni any udp port 514
sudo tail -f /var/log/network/network.log
sudo tail -f /var/ossec/logs/integrations.log
sudo /var/ossec/bin/wazuh-analysisd -t
```

### ASA

```cisco
show interface ip brief
show route
show nameif
show access-list
show running-config access-group
show logging
show nat
```

### Switch

```cisco
show vlan brief
show interfaces trunk
show interfaces status
show ip interface brief
show spanning-tree summary
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
show logging
show cdp neighbors
show port-security
show port-security interface Ethernet0/2
```

### Ubuntu Web Server

```bash
ip -br addr
ip route
systemctl is-active nginx
curl -I http://127.0.0.1
curl -I http://10.10.30.10
systemctl is-active wazuh-agent
ping -c 4 10.10.30.1
nc -zvw3 10.10.10.10 1514
nc -zvw3 10.10.10.10 1515
```

## 14. Backup và lưu cấu hình

Thư mục backup Wazuh đã được tạo:

```text
~/network-security-backup/
├── wazuh/
│   ├── wazuh.conf
│   ├── 10-network.conf
│   ├── ossec.conf
│   └── local_rules.xml
└── integrations/
    └── custom-telegram
```

Kiểm tra:

```bash
find ~/network-security-backup -type f -printf '%p\n'
```

Trên mỗi thiết bị Cisco, lưu cấu hình:

```cisco
write memory
```

Kiểm tra:

```cisco
show startup-config
show running-config
```

Không commit Bot Token, mật khẩu, private key, credential Wazuh hoặc thông tin truy cập Tailscale vào repository.

## 15. Ghi chú giới hạn và điểm cần hoàn thiện

1. CML Free giới hạn số node thiết bị chạy đồng thời; PC Test Port Security chỉ bật khi demo.
2. External Connector1 thuộc ADMIN, Connector2 thuộc USERS, Connector3 thuộc SERVERS theo mô tả topology.
3. Chưa xác nhận thiết bị đầu cuối USERS cố định trong mô hình.
4. Port Security được kiểm thử trên ACCESS_SW2 E0/2; ACCESS_SW1 không bật Port Security ở trạng thái cuối.
5. ACL `SERVER_IN` là ACL tạo nhầm được giữ lại; ACL đang gắn vào interface SERVERS là `SERVERS_IN`.
6. Rule 100102 (router failed login) và 100200 (Windows custom event) đã cấu hình nhưng chưa có bằng chứng kiểm thử PASS.
7. Thay Bot Token đã lộ trước khi demo/chia sẻ tài liệu hoặc repository.

## 16. Những thông tin còn thiếu/chưa thể xác nhận

Các hạng mục dưới đây là những khoảng trống còn lại trong bằng chứng hiện có; các phần Router1, ASA routing/ACL, switch VLAN/trunk/STP, Wazuh, Syslog và các rule đã nghiệm thu không yêu cầu cung cấp lại.

1. **Switch management gateway:** đã xác nhận SVI VLAN10 (`10.10.10.2`, `.3`, `.4`) nhưng chưa có output chứng minh `ip default-gateway` hoặc route quản trị trên từng switch.
2. **USERS endpoint:** chưa xác nhận thiết bị người dùng cuối thường trực; không gán IP cho endpoint USERS.
3. **Windows Agent:** Manager xác nhận `windows-local` Active, nhưng chưa có IP và chưa có bằng chứng Rule `100200` được kích hoạt thành công.
4. **Router failed login:** Rule `100102` đã khai báo, nhưng chưa có bằng chứng log đăng nhập thất bại được decoder/rule bắt và gửi Telegram.
5. **Cisco running-config backup dạng file:** đã xác nhận người triển khai dùng `write memory`; chưa có file text cấu hình Cisco được lưu ra ngoài lab trong bằng chứng hiện có.
6. **Wazuh backup:** thư mục backup đã tồn tại với các file `wazuh.conf`, `10-network.conf`, `ossec.conf`, `local_rules.xml`, `custom-telegram`; cần rà soát token trong bản backup trước khi chia sẻ.
7. **ASA NAT riêng:** chưa có output `show nat` đọc được để xác nhận ASA có NAT rule riêng. NAT overload trên Router1 đã được xác nhận; không nhầm NAT Router1 với NAT ASA.

### Chỉ cần bổ sung nếu muốn đóng các khoảng trống tùy chọn

Không cần chạy lại các lệnh nghiệm thu đã có bằng chứng. Hai output dưới đây chỉ cần khi muốn ghi rõ thêm default gateway quản trị của switch hoặc NAT riêng trên ASA:

```cisco
! Trên switch, nếu muốn xác nhận default gateway quản trị
show running-config | include ip default-gateway

! Trên ASA, nếu muốn xác nhận ASA có NAT rule riêng
show nat
```

**Không cần gửi lại** output Router1, ASA ACL/route, VLAN/trunk/STP, Port Security, Ubuntu Web Server, Wazuh service/agent, rsyslog hay các alert Telegram đã cung cấp.

## 17. Trạng thái lab tại thời điểm chốt tài liệu

- ACCESS_SW2 E0/2 đã ghi nhận Port Security violation 2 lần và đang ở `Secure-shutdown` / `err-disabled` trong ảnh kiểm tra cuối. PC Test Port Security đang tắt; đây là trạng thái lab sau demo, không phải port đã phục hồi để cắm thiết bị người dùng.
- ACCESS_SW1 E0/2 không còn cấu hình Port Security trong output cuối.
- Router1 có thể tắt khi cần giải phóng node để bật PC test, nhưng khi tắt sẽ không còn đường upstream qua Router1. Các bài test nội bộ giữa VLAN qua ASA vẫn có thể chạy nếu các node và đường kết nối nội bộ còn hoạt động.
- Trước buổi demo, kiểm tra lại trạng thái interface, ACL, dịch vụ Wazuh và token Telegram; không mặc định trạng thái lab hiện tại giống trạng thái lúc kiểm thử trước.

## 18. Kết luận

Mô hình đã triển khai và kiểm chứng các chức năng cốt lõi: phân đoạn VLAN, kiểm soát liên VLAN bằng ASA ACL, trunk và STP trên switch, Port Security trên ACCESS_SW2, thu thập Syslog tập trung bằng rsyslog, phân tích sự kiện bằng Wazuh và gửi cảnh báo Telegram.

Ubuntu Web Server chạy Nginx trả HTTP 200, Wazuh Agent hoạt động và được Manager hiển thị Active. Windows Agent cũng được Manager hiển thị Active.

Các kịch bản chưa có bằng chứng được đánh dấu rõ ràng là chưa nghiệm thu, tránh nhầm lẫn giữa cấu hình đã khai báo và chức năng đã kiểm thử thực tế.
