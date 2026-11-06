# BÁO CÁO TIẾN ĐỘ GIAI ĐOẠN 3

## Cấu hình địa chỉ IP, Syslog và Port Security

-   **Ngày lập:** 06/11/2026
-   **Phạm vi:** Hệ thống mạng mô phỏng gồm các switch tầng/khu vực,
    switch trung tâm và máy chủ Syslog.

> **Lưu ý:** Các địa chỉ IP chưa được xác nhận bằng cấu hình thực tế
> được ghi rõ là kế hoạch/đề xuất. Cần đối chiếu `running-config` trước
> khi chốt nghiệm thu.

## 1. Mục tiêu giai đoạn 3

-   Hoàn thiện kế hoạch địa chỉ IP quản trị cho các thiết bị mạng.
-   Cấu hình Syslog để thiết bị gửi nhật ký về máy chủ tập trung
    `192.168.10.50`.
-   Triển khai Port Security trên các cổng access kết nối thiết bị đầu
    cuối.
-   Kiểm tra trạng thái cấu hình, MAC sticky và số lần vi phạm Port
    Security.

## 2. Kế hoạch chia địa chỉ IP

  ----------------------------------------------------------------------------------------
  Thiết       VLAN/Mạng          Địa chỉ IP Subnet mask       Gateway          Trạng
  bị/Thành                                                                     thái/Ghi
  phần                                                                         chú
  ----------- ----------- ----------------- ----------------- ---------------- -----------
  Gateway     VLAN 2          `192.168.2.1` `255.255.255.0`   ---              Gateway
  quản trị                                                                     trong kế
  VLAN 2                                                                       hoạch quản
                                                                               trị

  SW-Tang1    VLAN 10        `192.168.10.2` `255.255.255.0`   `192.168.10.1`   Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  SW-Tang2    VLAN 10        `192.168.10.3` `255.255.255.0`   `192.168.10.1`   Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  SW-ToaA     VLAN 2          `192.168.2.3` `255.255.255.0`   `192.168.2.1`    Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  SW-A-T3     VLAN 2          `192.168.2.4` `255.255.255.0`   `192.168.2.1`    Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  SW-A-T4     VLAN 2          `192.168.2.5` `255.255.255.0`   `192.168.2.1`    Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  SW-A-T5     VLAN 2          `192.168.2.6` `255.255.255.0`   `192.168.2.1`    Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  SW-Tang1B   VLAN 2          `192.168.2.7` `255.255.255.0`   `192.168.2.1`    Đã xác nhận
                                                                               qua `show ip
                                                                               interface brief`

  B2-SW       VLAN 2          `192.168.2.8` `255.255.255.0`   `192.168.2.1`    Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  B3-SW       VLAN 2          `192.168.2.9` `255.255.255.0`   `192.168.2.1`    Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  B4-SW       VLAN 2         `192.168.2.10` `255.255.255.0`   `192.168.2.1`    Đã ghi nhận
                                                                               cấ hình
                                                                               quản trị

  DHCP Server VLAN 10       `192.168.10.10` `255.255.255.0`   `192.168.10.1`   Địa chỉ máy
                                                                               chủ DHCP
                                                                               trong mô
                                                                               hình

  Syslog      VLAN 10       `192.168.10.50` `255.255.255.0`   `192.168.10.1`   Máy chủ
  Server                                                                       nhận log
                                                                               tập trung
  ----------------------------------------------------------------------------------------

---

## 3. Cấu hình Syslog

### 3.1. Mục đích

Syslog tập trung nhật ký hoạt động từ switch/router về máy chủ. Người
quản trị có thể theo dõi trạng thái cổng, sự kiện bảo mật, thay đổi cấu
hình và lỗi phát sinh.

### 3.2. Thông số triển khai

  -----------------------------------------------------------------------
  Thông số                            Giá trị
  ----------------------------------- -----------------------------------
  Syslog Server                       `192.168.10.50`

  Mạng/VLAN máy chủ                   VLAN 10 -- `192.168.10.0/24`

  Gateway                             `192.168.10.1`

  Mức log đã trao đổi                 `debugging` (mức 7)

  Thiết bị đã ghi nhận cấu hình gửi   SWitch; router, mô hình Firewall ASA cũng sử dụng
  log                                 Syslog Server `192.168.10.50`
  -----------------------------------------------------------------------

### 3.3. Câu lệnh cấu hình tham khảo trên Cisco

``` text
configure terminal
logging 192.168.10.50
logging trap debugging
end
write memory
```

### 3.4. Kiểm tra

``` text
show running-config | include logging
show logging
```

Trên máy chủ `192.168.10.50`, cần xác nhận dịch vụ Syslog đang chạy và
cho phép nhận log. Nếu chưa nhận được log, kiểm tra kết nối IP, default
gateway, ACL/firewall và dịch vụ Syslog.

## 4. Cấu hình Port Security

### 4.1. Mục đích

Port Security giới hạn số địa chỉ MAC được phép xuất hiện trên một cổng
access. Sticky tự động học và lưu MAC vào cấu hình. Chế độ `restrict`
loại bỏ lưu lượng từ MAC vi phạm và ghi nhận vi phạm, không đưa cổng vào
trạng thái shutdown.

### 4.2. Thông số cấu hình

  Tham số          Giá trị
  ---------------- -------------------------------------
  Loại cổng        Access -- kết nối thiết bị đầu cuối
  Port Security    Bật
  Maximum MAC      2 MAC/cổng
  Kiểu học MAC     Sticky
  Violation mode   Restrict
  PortFast         Bật trên cổng access phù hợp
  Lưu cấu hình     `write memory`

### 4.3. Phạm vi cổng đã ghi nhận

  ------------------------------------------------------------------------
  Thiết bị         Các cổng đã ghi                Số cổng Ghi chú
                   nhận                                   
  ---------------- ---------------- --------------------- ----------------
  SW-Tang1         Fa0/1--4,                            8 Cổng access;
                   Fa0/6--9                               không áp dụng
                                                          lên uplink trunk

  SW-Tang2         Fa0/2--10                            9 Fa0/1 là trunk

  SW-A-T3          Fa0/2--3,                            9 Cổng access;
                   Fa0/6--8,                              loại trừ uplink
                   Fa0/10--12,                            
                   Fa0/14                                 

  SW-A-T4          Chưa có danh                        12 Đã ghi nhận tổng
                   sách chi tiết                          số cổng triển
                                                          khai

  SW-A-T5          Chưa có danh                         6 Đã ghi nhận tổng
                   sách chi tiết                          số cổng triển
                                                          khai

  SW-Tang1B        Chưa có danh                         6 Đã ghi nhận tổng
                   sách chi tiết                          số cổng triển
                                                          khai
  ------------------------------------------------------------------------

### 4.4. Câu lệnh cấu hình mẫu

Ví dụ dưới đây minh họa cấu hình trên một nhóm cổng access. Cần thay
đúng dải cổng theo từng switch.

``` text
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

## 5. Kiểm tra và đánh giá kết quả

Các lệnh kiểm tra và lưu ảnh minh chứng:

``` text
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

Kết quả đã có dữ liệu xác nhận trên **SW-Tang2, cổng Fa0/2 (PC4, VLAN
21)**:

-   Trạng thái: `Secure-up`.
-   Violation mode: `Restrict`.
-   Maximum: 2 MAC.
-   Sticky MAC đã học: `00D0.FF75.A2D3`.
-   Số lần vi phạm: 0.

Các cổng/thiết bị còn lại cần bổ sung ảnh lệnh `show` tương ứng để xác
nhận trạng thái thực tế trước khi nghiệm thu.

## 6. Tiến độ thực hiện giai đoạn 3

  -----------------------------------------------------------------------------------------------
                    STT Nội dung          Tiến độ          Minh chứng cần lưu
  --------------------- ----------------- ---------------- --------------------------------------
                      1 Lập kế hoạch IP   Đã tổng hợp;      `show ip interface brief`, cấu hình
                        quản trị và IP    SW-Tang1B đã     SVI, bảng IP; kiểm tra
                        máy chủ           xác nhận IP       default gateway thực tế    

                      2 Cấu hình Syslog   Đã ghi nhận cấu  `show logging`, ảnh log trên Syslog
                        về                hình trên        Server
                        `192.168.10.50`   SW-Tang2; cần    
                                          xác nhận log đến 
                                          máy chủ          

                      3 Triển khai Port   Đã triển khai    `show port-security`,
                        Security trên     theo phạm vi     `show port-security address`
                        cổng access       cổng đã thống kê 

                      4 Kiểm tra sticky   Đã xác nhận      Ảnh kết quả
                        MAC và violation  SW-Tang2 Fa0/2:  `show port-security interface fa0/2`
                                          Secure-up,       
                                          Restrict, 0      
                                          violation        

                      5 Hoàn thiện hồ sơ  Đang cập nhật    Ảnh cấu hình từng switch và bảng tổng
                        và nghiệm thu     minh chứng cho   hợp
                                          các switch còn   
                                          lại              
  -----------------------------------------------------------------------------------------------

## 7. Kết luận và công việc tiếp theo

Giai đoạn 3 tập trung vào ba nội dung: quy hoạch địa chỉ IP quản trị/máy
chủ, cấu hình Syslog tập trung và tăng cường bảo mật cổng access bằng
Port Security. Các thông số Port Security chính đã được xác định
(maximum 2, sticky, restrict); máy chủ Syslog được thống nhất tại
`192.168.10.50`. Kết quả kiểm tra cụ thể đã xác nhận trên SW-Tang2
Fa0/2.

Công việc cần tiếp tục:

1.  Đối chiếu các IP quản trị được ghi là kế hoạch/đề xuất với cấu hình
    thực tế trên từng switch.
2.  Kiểm tra default gateway thực tế của SW-Tang1B; bổ sung danh sách
    cổng Port Security chi tiết của SW-A-T4, SW-A-T5 và SW-Tang1B.
3.  Kiểm tra máy chủ Syslog thực sự nhận log từ từng thiết bị; lưu ảnh
    minh chứng.
4.  Chụp kết quả `show port-security` và `show port-security address`
    trên từng switch để xác nhận MAC học được và violation count.

------------------------------------------------------------------------
