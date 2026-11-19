# MONITORING & CENTRALIZED LOG MANAGEMENT (WAZUH SIEM & TELEGRAM)

> **Phân hệ:** Quản lý nhật ký an ninh tập trung & Cảnh báo thời gian thực  
> **Nền tảng:** Wazuh SIEM v4.x All-in-One (Manager, Indexer, Dashboard) + Rsyslog Server + Telegram Bot Webhook

---

## 📂 Cấu trúc thư mục

```text
monitoring/
├── wazuh/
│   ├── ossec.conf            # File cấu hình trung tâm Wazuh Manager (Agents, Syslog UDP 514, Rules)
│   └── rules/                # Thư mục chứa các custom rules an ninh (/var/ossec/etc/rules/local_rules.xml)
└── alerts/
    └── custom-telegram.py    # Script Python nhận webhook từ Wazuh và gửi thông báo khẩn cấp tới Telegram
```

---

## ⚙️ Hướng dẫn cấu hình

### 1. Cấu hình Wazuh Manager (`wazuh/ossec.conf`)
- Đặt cổng tiếp nhận Syslog từ thiết bị mạng Cisco và Firewall:
  ```xml
  <remote>
    <connection>syslog</connection>
    <port>514</port>
    <protocol>udp</protocol>
    <allowed-ips>192.168.0.0/16</allowed-ips>
  </remote>
  ```
- Tích hợp script gửi cảnh báo Telegram:
  ```xml
  <integration>
    <name>custom-telegram</name>
    <hook_url>https://api.telegram.org/bot<TOKEN>/sendMessage</hook_url>
    <level>10</level>
    <alert_format>json</alert_format>
  </integration>
  ```

### 2. Triển khai Script cảnh báo Telegram (`alerts/custom-telegram.py`)
- Copy script vào `/var/ossec/integrations/custom-telegram` trên máy chủ Wazuh.
- Phân quyền thực thi:
  ```bash
  sudo chmod 750 /var/ossec/integrations/custom-telegram
  sudo chown root:wazuh /var/ossec/integrations/custom-telegram
  ```
- Khởi động lại dịch vụ Wazuh Manager:
  ```bash
  sudo systemctl restart wazuh-manager
  ```
