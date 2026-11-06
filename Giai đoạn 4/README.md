# BÁO CÁO TRIỂN KHAI
## GIAI ĐOẠN 4: TRIỂN KHAI HỆ THỐNG GIÁM SÁT AN TOÀN MẠNG VỚI WAZUH SIEM
### Xây dựng máy chủ Wazuh, tích hợp Agent, phát hiện sự kiện và gửi cảnh báo qua Telegram

| Thông tin | Chi tiết |
| :--- | :--- |
| **Đề tài** | Phân tích và thiết kế an toàn mạng |
| **Hạng mục phụ trách** | Wazuh SIEM Core và tích hợp cảnh báo Telegram |
| **Người thực hiện** | Nguyễn Anh Xuân |
| **Môi trường triển khai** | VMware Workstation – Ubuntu Server – Windows 11 |
| **Trạng thái báo cáo** | Tổng hợp phần đã triển khai và phần chờ tích hợp GNS3 |

---

## 1. Mục tiêu triển khai

Giai đoạn 4 tập trung xây dựng nền tảng giám sát an toàn mạng tập trung bằng Wazuh SIEM. Hệ thống có nhiệm vụ thu thập sự kiện từ các máy chủ/thiết bị đầu cuối, phân tích log, phát hiện dấu hiệu bất thường và hiển thị cảnh báo trên Dashboard. Đối với các cảnh báo đạt ngưỡng cấu hình, hệ thống gửi thông báo đến Telegram để nhóm có thể theo dõi kịp thời.

- Triển khai Wazuh All-in-one trên Ubuntu Server.
- Kết nối các máy Ubuntu và Windows vào Wazuh thông qua Wazuh Agent.
- Kiểm tra khả năng thu thập và phân tích sự kiện đăng nhập, sudo và SSH.
- Tích hợp Telegram Bot để chuyển cảnh báo từ Wazuh đến người quản trị.
- Chuẩn bị kiến trúc tiếp nhận Syslog từ Router/Switch trong mô hình GNS3 ở bước tích hợp tiếp theo.

---

## 2. Mô hình và thông số môi trường

Mô hình triển khai hiện tại sử dụng máy chủ Wazuh độc lập trong mạng VMware NAT. Các Agent kết nối về máy chủ qua mạng lab.

| Thành phần | Hostname / Agent | Địa chỉ IP | Vai trò |
| :--- | :--- | :--- | :--- |
| **Wazuh Server** | `wazuh-server` | `192.168.181.20` | Ubuntu Server 24.04.3; Wazuh Manager, Indexer, Dashboard, Filebeat |
| **Ubuntu Agent** | `ubuntu-server` / `002` | `192.168.181.129` | Thu thập log Linux; đang kết nối |
| **Ubuntu Agent** | `ubuntu-client` / `001` | `192.168.181.130` | Thu thập log Linux; đang kết nối |
| **Windows Agent** | `windows-local` / `003` | `192.168.181.1` | Thu thập log Windows; đang kết nối |

Các cổng dịch vụ chính trong kiến trúc:
- **TCP 443**: Cho Dashboard
- **TCP 1514**: Cho truyền sự kiện Agent
- **TCP 1515**: Cho đăng ký Agent
- **UDP/TCP 514**: Cho Syslog (sẽ được cấu hình khi tích hợp thiết bị mạng)
- Các cổng quản trị nội bộ khác cần được giới hạn truy cập theo phạm vi mạng quản trị.

---

## 3. Quy trình triển khai

### 3.1. Chuẩn bị máy chủ Wazuh
1. Tạo máy ảo Ubuntu Server 24.04.3 LTS trên VMware Workstation.
2. Đặt hostname là `wazuh-server` và cấu hình địa chỉ IP tĩnh `192.168.181.20/24` trong mạng VMware NAT.
3. Kiểm tra kết nối mạng, gateway và khả năng truy cập Internet để phục vụ cài đặt.
4. Cập nhật hệ điều hành và chuẩn bị tài nguyên cho máy chủ giám sát.

### 3.2. Cài đặt Wazuh All-in-one
Triển khai mô hình All-in-one để các thành phần cốt lõi chạy trên cùng một máy chủ, phù hợp với quy mô môi trường lab. Phiên bản đang sử dụng là **Wazuh 4.14.9**.

1. Cài đặt Wazuh Manager để tiếp nhận và phân tích sự kiện.
2. Cài đặt Wazuh Indexer để lưu trữ và lập chỉ mục dữ liệu cảnh báo.
3. Cài đặt Wazuh Dashboard để truy vấn, theo dõi và trực quan hóa dữ liệu.
4. Cài đặt Filebeat để chuyển dữ liệu cảnh báo từ Manager đến Indexer.
5. Kiểm tra trạng thái dịch vụ và truy cập Dashboard tại `https://192.168.181.20:443`.

### 3.3. Cài đặt và kết nối Wazuh Agent
Cài đặt Wazuh Agent trên các máy đầu cuối, cấu hình địa chỉ Wazuh Manager là `192.168.181.20`, thực hiện đăng ký Agent và kiểm tra trạng thái kết nối trên Dashboard.

- **Ubuntu Agent `ubuntu-server` (`192.168.181.129`):** đã kết nối và hoạt động.
- **Ubuntu Agent `ubuntu-client` (`192.168.181.130`):** đã kết nối và hoạt động.
- **Windows Agent `windows-local` (`192.168.181.1`):** đã kết nối và hoạt động.

### 3.4. Kiểm tra thu thập và phân tích sự kiện
Sau khi Agent kết nối, tiến hành tạo các sự kiện kiểm thử trên máy Linux và kiểm tra cảnh báo được Wazuh phân tích. Các sự kiện đã quan sát gồm đăng nhập phiên PAM, sử dụng sudo và các lần xác thực SSH thất bại.

| Sự kiện / Rule ID | Mức độ | Ý nghĩa kiểm thử |
| :--- | :--- | :--- |
| **PAM session open/close – 5501/5502** | Level 3 | Ghi nhận mở/đóng phiên đăng nhập. |
| **Successful sudo – 5402** | Level 3 | Ghi nhận thao tác nâng quyền bằng sudo. |
| **SSH brute-force – 5712** | Level 10 | Phát hiện nhiều lần xác thực SSH thất bại theo ngưỡng luật. |

Đối với Rule 5712, bài kiểm thử tạo 8 lần kết nối SSH thất bại trong khoảng thời gian ngắn. Rule có cơ chế tương quan theo cùng địa chỉ nguồn và có thuộc tính ignore để hạn chế cảnh báo lặp. Vì vậy, khi kiểm thử lại ngay sau lần phát hiện đầu tiên, hệ thống có thể không gửi thêm thông báo trong khoảng thời gian suppression. Đây là hành vi kiểm soát tần suất cảnh báo, không nhất thiết là lỗi.

### 3.5. Tích hợp Telegram Bot
Tạo Telegram Bot và sử dụng Bot Token cùng Chat ID để cấu hình kênh nhận cảnh báo. Một script Python tùy chỉnh được đặt tại `/var/ossec/integrations/custom-telegram`. Script đọc alert JSON do Wazuh truyền vào, trích xuất thông tin Rule, Agent, mức độ và thời gian, sau đó gọi Telegram Bot API `sendMessage`.

1. Tạo Bot bằng BotFather, khởi tạo cuộc trò chuyện với Bot và xác định Chat ID.
2. Tạo script `custom-telegram` để đọc tệp alert JSON và gửi nội dung cảnh báo.
3. Thiết lập quyền thực thi và quyền sở hữu phù hợp cho script.
4. Khai báo integration `custom-telegram` trong `/var/ossec/etc/ossec.conf`, sử dụng alert_format JSON và ngưỡng level.
5. Kiểm tra cấu hình Wazuh, khởi động lại `wazuh-manager` và xác nhận `wazuh-integratord` hoạt động.
6. Tạo sự kiện kiểm thử và xác minh thông báo xuất hiện trên Telegram; kiểm tra nhật ký `integrations.log`.

**Kết quả kiểm thử:** Telegram đã nhận được thông báo thử nghiệm và cảnh báo Wazuh. Nhật ký tích hợp ghi nhận trạng thái `SUCCESS` cùng Rule ID và tên Agent, xác nhận luồng gửi cảnh báo hoạt động.

---

## 4. Kiến trúc tích hợp Syslog từ GNS3 (công việc tiếp theo)

Tại thời điểm lập báo cáo, mô hình GNS3 của nhóm chưa hoàn tất nên phần tiếp nhận Syslog từ Router/Switch chưa được triển khai và chưa có kết quả kiểm thử thực tế. Kiến trúc dự kiến tận dụng chính Wazuh Server `192.168.181.20` làm máy chủ nhận Syslog, không cần tạo thêm một máy ảo riêng cho rsyslog.

```
Router/Switch GNS3  →  Syslog UDP/TCP 514  →  rsyslog  →  File log
                    →  Wazuh Manager (localfile/Decoder/Rules)  →  Dashboard / Telegram
```

1. Cài đặt/cấu hình rsyslog trên Wazuh Server để lắng nghe Syslog UDP hoặc TCP port 514.
2. Tạo thư mục và quy tắc lưu log mạng, ví dụ `/var/log/network/`; thiết lập quyền đọc phù hợp cho Wazuh.
3. Cấu hình Router/Switch trong GNS3 gửi Syslog đến `192.168.181.20`.
4. Kiểm tra gói tin Syslog đến máy chủ và xác minh nội dung log được ghi vào file.
5. Khai báo file log trong `ossec.conf` bằng `localfile` với định dạng phù hợp.
6. Kiểm tra Wazuh đọc được log; xây dựng Decoder/Rules cho các sự kiện cần giám sát.
7. Tạo sự kiện kiểm thử trên thiết bị mạng, xác minh cảnh báo trên Dashboard và Telegram.

> [!NOTE]
> **Lưu ý kỹ thuật:** rsyslog có nhiệm vụ tiếp nhận và ghi log; Wazuh mới là thành phần phân tích log thông qua localfile, Decoder và Rules. Việc nhận được Syslog không đồng nghĩa Wazuh đã nhận diện được mọi kiểu tấn công. Một số sự kiện cần bật logging trên thiết bị, tùy chỉnh Decoder/Rules hoặc kết hợp nhiều nguồn log.

---

## 5. Kết quả đạt được

- [x] Xây dựng được máy chủ Wazuh All-in-one phiên bản 4.14.9 trên Ubuntu Server 24.04.3 LTS.
- [x] Truy cập và sử dụng được Wazuh Dashboard qua HTTPS.
- [x] Kết nối được Ubuntu Agent và Windows Agent; kiểm tra được trạng thái Agent trên Dashboard.
- [x] Quan sát được các cảnh báo liên quan đến phiên đăng nhập PAM, sudo và SSH brute-force.
- [x] Tích hợp Telegram Bot thành công; cảnh báo được chuyển đến cuộc trò chuyện Telegram.
- [x] Có cơ sở hạ tầng sẵn sàng để tiếp nhận log mạng từ mô hình GNS3 trong bước tiếp theo.

---

## 6. Tồn tại và lưu ý

- Chưa tích hợp Syslog từ Router/Switch do mô hình GNS3 chưa hoàn thiện.
- Chưa xây dựng và kiểm thử đầy đủ Decoder/Rules chuyên biệt cho các sự kiện bảo mật mạng.
- Agent `ubuntu-client` đang Disconnected vì máy ảo tắt; cần bật lại nếu muốn đưa vào kiểm thử.
- Ngưỡng level của Telegram integration cần được rà soát để bảo đảm chỉ gửi các cảnh báo phù hợp, tránh quá nhiều thông báo.
- Bot Token từng xuất hiện trong ảnh chụp màn hình. Cần thu hồi token cũ, tạo token mới trước khi demo công khai hoặc chia sẻ repository; không đưa token vào Git.

---

## 7. Kế hoạch công việc tiếp theo

| STT | Công việc | Kết quả cần đạt |
| :---: | :--- | :--- |
| **1** | Hoàn thiện mô hình GNS3 và xác nhận kết nối đến Wazuh Server | Router/Switch liên lạc được với 192.168.181.20 |
| **2** | Cấu hình rsyslog nhận Syslog | Log thiết bị mạng được ghi vào file trên máy chủ |
| **3** | Khai báo localfile trong Wazuh | Wazuh đọc và phân tích được log mạng |
| **4** | Xây dựng Decoder/Rules | Nhận diện được các sự kiện mục tiêu |
| **5** | Kiểm thử sự kiện/tấn công trong lab | Có alert và bằng chứng kiểm thử |
| **6** | Xác minh Telegram và hoàn thiện tài liệu | Cảnh báo đến đúng kênh; có ảnh chụp minh chứng |

---

## 8. Kết luận

Giai đoạn triển khai Wazuh Core đã hoàn thành các chức năng nền tảng: cài đặt máy chủ SIEM, kết nối Agent, thu thập và phân tích một số sự kiện đầu cuối, đồng thời tích hợp gửi cảnh báo qua Telegram. Hệ thống đã được kiểm thử bằng các sự kiện đăng nhập, sudo và SSH brute-force.

Phần việc còn lại là tích hợp nguồn Syslog từ mô hình GNS3, cấu hình tiếp nhận log tập trung và phát triển Decoder/Rules phục vụ giám sát các sự kiện trên thiết bị mạng. Sau khi hoàn tất các bước này, nhóm sẽ có luồng giám sát xuyên suốt từ thiết bị mạng và máy đầu cuối đến Wazuh Dashboard và Telegram.

### Phụ lục – Vị trí cấu hình và nhật ký tham chiếu
- `/var/ossec/etc/ossec.conf` – Cấu hình Wazuh Manager, localfile và integration.
- `/var/ossec/integrations/custom-telegram` – Script Python gửi cảnh báo đến Telegram.
- `/var/ossec/logs/alerts/alerts.json` – Dữ liệu cảnh báo JSON do Wazuh tạo.
- `/var/ossec/logs/integrations.log` – Nhật ký kết quả thực thi integration.
- `/var/ossec/logs/ossec.log` – Nhật ký hoạt động của các thành phần Wazuh.

> **Ghi chú:** Báo cáo phản ánh trạng thái triển khai đã được xác nhận trong quá trình làm việc. Các hạng mục Syslog/GNS3 được mô tả là kế hoạch tích hợp, không phải kết quả đã hoàn thành.

---

## 9. Phụ lục kỹ thuật: câu lệnh triển khai và cấu hình

Phần này bổ sung các thao tác ở mức quản trị hệ thống để báo cáo có thể dùng làm tài liệu tái triển khai. Các lệnh dưới đây được viết theo môi trường đã ghi nhận: Wazuh Server `192.168.181.20`, Ubuntu Agent `192.168.181.129`, Ubuntu Agent `192.168.181.130` và Windows Agent `192.168.181.1`. Các giá trị bí mật như Bot Token không được ghi trực tiếp trong báo cáo.

### 9.1. Chuẩn bị và kiểm tra Ubuntu Server
Thực hiện trên máy Wazuh Server (hostname `wazuh-server`, IP `192.168.181.20`):

```bash
sudo apt update
sudo apt upgrade -y
hostnamectl
ip -br address
ip route
ping -c 4 192.168.181.2
getent hosts packages.wazuh.com
```

Địa chỉ IP tĩnh được cấu hình bằng Netplan. Tên file YAML và nội dung cấu hình phụ thuộc cấu hình card mạng của máy ảo tại thời điểm cài đặt; cần kiểm tra trước khi sửa:

```bash
ls -l /etc/netplan/
sudo nano /etc/netplan/<ten-file>.yaml
sudo netplan try
sudo netplan apply
```

> [!WARNING]
> **Lưu ý:** Không xóa file Netplan hoặc thay toàn bộ nội dung bằng một mẫu không khớp card mạng/gateway thực tế. Trong lab này máy chủ dùng IP `192.168.181.20/24`, gateway/DNS `192.168.181.2`. Nên sao lưu YAML trước khi sửa: `sudo cp /etc/netplan/<ten-file>.yaml /etc/netplan/<ten-file>.yaml.bak`.

### 9.2. Cài đặt Wazuh All-in-one
Môi trường ghi nhận Wazuh 4.14.9 trên Ubuntu Server 24.04.3 LTS. Lệnh cài đặt All-in-one theo nhánh Wazuh 4.14:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

Sau cài đặt, kiểm tra các dịch vụ chính:

```bash
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager
sudo systemctl status filebeat --no-pager
sudo ss -lntup | grep -E ':(443|1514|1515|55000|9200)\b'
```

Truy cập Dashboard tại `https://192.168.181.20`. Thông tin tài khoản ban đầu được sinh trong quá trình cài đặt; không đưa mật khẩu vào báo cáo hoặc Git.

> [!CAUTION]
> **Lưu ý:** Không xóa các thư mục `/var/ossec`, `/var/lib/wazuh-indexer` hoặc file cấu hình dịch vụ để xử lý lỗi cài đặt. Nếu cần gỡ cài đặt phải dùng quy trình uninstall chính thức tương ứng phiên bản và sao lưu dữ liệu trước.

### 9.3. Cài đặt và đăng ký Ubuntu Agent
Thực hiện trên từng máy Ubuntu Agent. Trước hết tải và chạy repository installer tương ứng phiên bản Wazuh đang dùng, sau đó cài `wazuh-agent` và khai báo địa chỉ Manager:

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo WAZUH_MANAGER='192.168.181.20' apt install wazuh-agent
```

Nếu Agent đã được cài đặt, không cài lại chỉ để đổi Manager. Kiểm tra file cấu hình và dịch vụ:

```bash
sudo nano /var/ossec/etc/ossec.conf
# Trong khối <client>, kiểm tra:
# <server>
#   <address>192.168.181.20</address>
#   <port>1514</port>
#   <protocol>tcp</protocol>
# </server>

sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent
sudo systemctl status wazuh-agent --no-pager
```

Đăng ký Agent bằng cơ chế enrollment của Wazuh (TCP 1515). Nếu dùng giao diện Deploy new agent, sử dụng đúng lệnh enrollment được Dashboard sinh ra cho hệ điều hành và phiên bản tương ứng. Không tự suy đoán mật khẩu enrollment hoặc khóa agent.

> **Lưu ý:** Ubuntu Agent `192.168.181.130` đã đăng ký nhưng hiện Disconnected vì VM đang tắt. Đây là trạng thái do máy ảo không chạy, không phải lỗi Manager.

### 9.4. Cài đặt Windows Agent
Windows Agent được cài trên Windows 11. Tệp MSI và lệnh cài đặt cần lấy từ trang Deploy new agent của Wazuh Dashboard để bảo đảm đúng phiên bản. Mẫu lệnh cài đặt từ PowerShell chạy quyền Administrator:

```powershell
msiexec.exe /i .\wazuh-agent-4.x.x-1.msi /q WAZUH_MANAGER="192.168.181.20"
NET START WazuhSvc
```

Kiểm tra dịch vụ trong PowerShell:

```powershell
Get-Service WazuhSvc
Get-NetTCPConnection -RemoteAddress 192.168.181.20 -ErrorAction SilentlyContinue
```

> **Lưu ý:** Thay tên MSI mẫu bằng đúng tên file đã tải. Không ghi cứng phiên bản giả định hoặc dùng lại MSI không rõ nguồn.

### 9.5. Kiểm tra Agent và nhật ký Wazuh
Trên Wazuh Server:

```bash
sudo /var/ossec/bin/agent_control -lc
sudo tail -n 50 /var/ossec/logs/ossec.log
sudo tail -n 20 /var/ossec/logs/alerts/alerts.json
```

Trên Ubuntu Agent, kiểm tra log Agent và kết nối đến Manager:

```bash
sudo tail -n 50 /var/ossec/logs/ossec.log
nc -vz 192.168.181.20 1514
nc -vz 192.168.181.20 1515
```

### 9.6. Cấu hình Telegram Integration
Script tích hợp được đặt tại `/var/ossec/integrations/custom-telegram`. Tạo hoặc chỉnh sửa bằng nano; nội dung script đọc alert JSON, trích xuất Rule/Agent/timestamp và gọi Telegram Bot API. Không đưa Bot Token thật vào tài liệu.

```bash
sudo nano /var/ossec/integrations/custom-telegram
sudo chown root:wazuh /var/ossec/integrations/custom-telegram
sudo chmod 750 /var/ossec/integrations/custom-telegram
```

Trong file `/var/ossec/etc/ossec.conf`, thêm hoặc kiểm tra khối integration bên trong một khối `<ossec_config>` hợp lệ. Thay `BOT_TOKEN` bằng token được lưu an toàn trên máy chủ; không chụp màn hình hoặc chia sẻ giá trị này:

```xml
<integration>
    <name>custom-telegram</name>
    <hook_url>TELEGRAM_CHAT_ID</hook_url>
    <api_key>BOT_TOKEN</api_key>
    <level>7</level>
    <alert_format>json</alert_format>
</integration>
```

> [!IMPORTANT]
> **Lưu ý cấu trúc:** `ossec.conf` của môi trường này có nhiều khối cấp cao `<ossec_config>`. Không dùng trình phân tích XML thông thường để parse toàn bộ file như một XML đơn-root; không xóa hoặc gộp các khối cấu hình hiện có. Chỉ thêm integration vào đúng cấu trúc Wazuh.

Kiểm tra cấu hình và khởi động lại Manager:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
sudo /var/ossec/bin/wazuh-integratord -t
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager --no-pager
pgrep -af wazuh-integratord
sudo tail -n 50 /var/ossec/logs/ossec.log
sudo tail -n 50 /var/ossec/logs/integrations.log
```

Khi gửi thành công, `integrations.log` có thể ghi nhận dòng `SUCCESS` kèm Rule ID và Agent. Nếu cảnh báo không đến Telegram, kiểm tra ngưỡng `<level>`, trạng thái `integratord`, quyền thực thi script, kết nối Internet và log `integrations.log`.

### 9.7. Kiểm thử phát hiện SSH brute-force
Trong lab, tạo nhiều lần xác thực SSH thất bại tới Ubuntu Agent `192.168.181.129`. Lệnh dưới đây minh họa kiểm thử từ một máy Linux có SSH client; dùng tài khoản không tồn tại và xác thực public-key-only để tránh nhập mật khẩu thật:

```bash
for i in $(seq 1 8); do
    ssh -o BatchMode=yes \
        -o ConnectTimeout=3 \
        -o StrictHostKeyChecking=no \
        -o UserKnownHostsFile=/dev/null \
        -o PreferredAuthentications=publickey \
        -o PasswordAuthentication=no \
        wazuh_demo_invalid@192.168.181.129 true \
        >/dev/null 2>&1
    echo "Attempt $i sent"
done
```

Kiểm tra Rule 5712 trên Wazuh Server:

```bash
sudo grep '"id":"5712"' /var/ossec/logs/alerts/alerts.json | tail -n 5
sudo grep '"id":"5710"' /var/ossec/logs/alerts/alerts.json | tail -n 15
```

> **Lưu ý:** Rule 5712 có điều kiện tương quan số lần thất bại, khoảng thời gian và cùng địa chỉ nguồn; cơ chế ignore có thể tạm ngăn cảnh báo lặp sau khi Rule đã kích hoạt. Khi kiểm thử lại, chờ hết thời gian suppression và giữ nguyên máy nguồn để bảo đảm cùng source IP. Đây là mô phỏng trong lab.

### 9.8. Các file/thư mục cần biết và nguyên tắc chỉnh sửa

| Đường dẫn | Mục đích | Thao tác |
| :--- | :--- | :--- |
| `/var/ossec/etc/ossec.conf` | Cấu hình Manager, Agent-side localfile và integration | Sao lưu rồi chỉnh sửa; không xóa toàn bộ file |
| `/var/ossec/integrations/custom-telegram` | Script Python gửi Telegram | Tạo/chỉnh sửa; chmod 750, owner root:wazuh |
| `/var/ossec/logs/alerts/alerts.json` | Cảnh báo JSON | Chỉ đọc/đối chiếu; không sửa thủ công |
| `/var/ossec/logs/ossec.log` | Log hoạt động Wazuh | Chỉ đọc/đối chiếu |
| `/var/ossec/logs/integrations.log` | Kết quả tích hợp ngoài | Chỉ đọc/đối chiếu |
| `/etc/netplan/*.yaml` | Cấu hình mạng Ubuntu | Sao lưu trước khi chỉnh; không xóa tùy tiện |
| `/etc/apt/sources.list.d/wazuh.list` | APT repository của Wazuh Agent | Chỉ sửa khi cần thay repository/phiên bản |

### 9.9. Sao lưu trước khi chỉnh sửa và các lệnh không nên dùng tùy tiện

```bash
# Sao lưu cấu hình trước khi sửa
sudo cp -a /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak.$(date +%F-%H%M%S)

# Xem cấu hình integration (cẩn thận: có thể lộ token)
sudo grep -n -A8 -B2 '<integration>' /var/ossec/etc/ossec.conf

# Khôi phục từ bản sao lưu nếu cấu hình lỗi (đổi tên file đúng với bản backup)
sudo cp -a /var/ossec/etc/ossec.conf.bak.<timestamp> /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-manager
```

> [!CAUTION]
> **Lưu ý:** Không chạy `rm -rf /var/ossec`, không xóa `ossec.conf`, không xóa thư mục indexer để xử lý lỗi thông thường. Không gửi ảnh chứa `api_key`/Bot Token. Token đã từng lộ trong ảnh chụp nên cần thu hồi token cũ và tạo token mới trước khi demo công khai hoặc đẩy repository lên GitHub.

---

## 10. Ghi chú về độ chính xác của câu lệnh

Một số lệnh ở phụ lục là mẫu triển khai chuẩn theo nhánh Wazuh 4.x hoặc lệnh kiểm tra trạng thái. Tên file Netplan, tên gói MSI Windows, lệnh enrollment đầy đủ và nội dung script phải đối chiếu với máy thực tế/Dashboard ở thời điểm triển khai. Báo cáo không ghi các giá trị bí mật hoặc giả định rằng những thao tác chưa thực hiện đã được hoàn tất. Trước khi dùng làm runbook tái triển khai, cần thay các placeholder như `<ten-file>.yaml`, `BOT_TOKEN`, `TELEGRAM_CHAT_ID` và tên MSI bằng giá trị phù hợp.

---

## 11. Phụ lục mã nguồn và tệp cấu hình

Ngoài câu lệnh triển khai, dự án có các tệp mã nguồn/cấu hình cần lưu cùng báo cáo. Các tệp được đóng gói riêng trong bộ mã nguồn đi kèm. Nội dung dưới đây mô tả vai trò, đường dẫn cài đặt và trạng thái của từng tệp.

- **`custom-telegram`** — Mã Python tích hợp Wazuh với Telegram. Đọc alert JSON do Wazuh truyền vào, lấy Rule/Agent/timestamp và gọi Telegram Bot API `sendMessage`. Đường dẫn triển khai: `/var/ossec/integrations/custom-telegram`.
- **`telegram-integration.xml`** — Đoạn cấu hình tích hợp Telegram để chèn vào cấu hình Wazuh hiện có. Thay `TELEGRAM_CHAT_ID` và `BOT_TOKEN` bằng giá trị thực tế tại máy chủ; không đưa token thật vào báo cáo hoặc Git.
- **`50-wazuh-network-syslog.conf.example`** — Mẫu cấu hình rsyslog nhận Syslog UDP/TCP 514 và ghi vào `/var/log/network/network.log`. Đây là mẫu chờ GNS3, chưa phải cấu hình đã triển khai.
- **`wazuh-localfile-syslog.xml`** — Mẫu localfile để Wazuh đọc file Syslog sau khi rsyslog được cấu hình. Chỉ áp dụng sau khi file log tồn tại và quyền đọc đã được thiết lập.
- **`README.txt`** — Hướng dẫn vị trí cài đặt, quyền file, trạng thái các mẫu cấu hình và lưu ý bảo mật.

### 11.1. Nội dung mã nguồn custom-telegram
Đây là mã nguồn Python được sử dụng trong môi trường lab. Script không chứa Bot Token; Wazuh truyền token và Chat ID thông qua tham số tích hợp.

```python
#!/usr/bin/env python3
"""
Wazuh custom integration: send selected Wazuh alerts to Telegram.

Expected Wazuh integration arguments:
  argv[1] = path to alert JSON file
  argv[2] = Telegram Bot Token (provided by Wazuh api_key)
  argv[3] = Telegram Chat ID (provided by Wazuh hook_url)

Do not hard-code the Bot Token in this file.
"""
import json
import sys
import urllib.parse
import urllib.request
import urllib.error

LOG_PATH = "/var/ossec/logs/integrations.log"


def write_log(message: str) -> None:
    with open(LOG_PATH, "a", encoding="utf-8") as log_file:
        log_file.write(message + "\n")


def main() -> None:
    if len(sys.argv) < 4:
        write_log("ERROR: Missing integration arguments")
        sys.exit(1)

    alert_file = sys.argv[1]
    bot_token = sys.argv[2]
    chat_id = sys.argv[3]

    try:
        with open(alert_file, "r", encoding="utf-8") as file:
            alert = json.load(file)

        rule = alert.get("rule", {})
        agent = alert.get("agent", {})

        message = (
            "🚨 WAZUH SECURITY ALERT\n\n"
            f"Severity: {rule.get('level', 'N/A')}\n"
            f"Rule ID: {rule.get('id', 'N/A')}\n"
            f"Description: {rule.get('description', 'N/A')}\n"
            f"Agent: {agent.get('name', 'N/A')}\n"
            f"Agent ID: {agent.get('id', 'N/A')}\n"
            f"Agent IP: {agent.get('ip', 'N/A')}\n"
            f"Timestamp: {alert.get('timestamp', 'N/A')}"
        )

        payload = urllib.parse.urlencode({
            "chat_id": chat_id,
            "text": message
        }).encode()

        url = f"https://api.telegram.org/bot{bot_token}/sendMessage"
        request = urllib.request.Request(url, data=payload, method="POST")

        with urllib.request.urlopen(request, timeout=15) as response:
            result = json.loads(response.read().decode())

        if result.get("ok"):
            write_log(
                f"SUCCESS: rule={rule.get('id')} "
                f"agent={agent.get('name')}"
            )
        else:
            write_log("ERROR: Telegram API returned failure")
            sys.exit(1)

    except urllib.error.HTTPError as error:
        write_log(f"HTTP ERROR: status={error.code}")
        sys.exit(1)
    except Exception as error:
        # Avoid logging tokens, request URLs, or other sensitive values.
        write_log(f"ERROR: {type(error).__name__}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

> [!NOTE]
> **Lưu ý:** Các tệp rsyslog và localfile trong thư mục `03_syslog_gns3_pending` là template cho giai đoạn tích hợp GNS3 sắp tới. Không mô tả chúng là cấu hình đã chạy thành công trong giai đoạn hiện tại.
