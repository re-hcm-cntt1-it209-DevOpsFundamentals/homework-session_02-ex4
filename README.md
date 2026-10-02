# Bài 4: Cấu hình tường lửa bảo vệ máy chủ

## 1. Mục tiêu

Cấu hình tường lửa cho Droplet bằng hai lớp:

* UFW được cấu hình trực tiếp trên Ubuntu Server.
* DigitalOcean Cloud Firewall được cấu hình ở phía hạ tầng.

Mục đích là chỉ cho phép những dịch vụ cần thiết từ bên ngoài truy cập vào máy chủ, trong bài này gồm SSH và HTTP.

---

## 2. Kiểm tra UFW hiện tại

Trước khi cấu hình, tôi kiểm tra trạng thái của UFW:

```bash
sudo ufw status verbose
```

Nếu UFW chưa được kích hoạt, hệ thống có thể hiển thị:

```text
Status: inactive
```

---

## 3. Cấu hình chính sách mặc định

Tôi thiết lập chính sách mặc định:

* Chặn tất cả kết nối đi vào.
* Cho phép kết nối đi ra.

Sử dụng các lệnh:

```bash
sudo ufw default deny incoming
```

```bash
sudo ufw default allow outgoing
```

Sau khi cấu hình, các kết nối từ bên ngoài sẽ bị chặn mặc định nếu không có rule cho phép.

---

## 4. Mở cổng SSH

Để vẫn có thể đăng nhập và quản trị server thông qua SSH, tôi mở cổng 22:

```bash
sudo ufw allow 22/tcp
```

Kiểm tra rule:

```bash
sudo ufw status
```

Kết quả sẽ có rule tương tự:

```text
22/tcp                    ALLOW
```

Việc mở port 22 được thực hiện trước khi bật UFW để tránh trường hợp tự chặn kết nối SSH tới server.

---

## 5. Mở cổng HTTP

Do bài trước đã cấu hình Nginx phục vụ website trên HTTP port 80, tôi cho phép kết nối tới port 80:

```bash
sudo ufw allow 80/tcp
```

Kiểm tra lại:

```bash
sudo ufw status
```

Kết quả sẽ có hai port được cho phép:

```text
22/tcp                    ALLOW
80/tcp                    ALLOW
```

---

## 6. Kích hoạt UFW

Sau khi đã mở các cổng cần thiết, tôi bật UFW:

```bash
sudo ufw enable
```

Hệ thống có thể cảnh báo rằng thao tác này có thể làm gián đoạn kết nối SSH.

Tôi xác nhận để tiếp tục vì port 22 đã được cho phép.

---

## 7. Kiểm tra trạng thái UFW

Sau khi bật, kiểm tra lại:

```bash
sudo ufw status verbose
```

Kết quả mong đợi:

```text
Status: active

Default: deny (incoming), allow (outgoing)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
```

Nếu sử dụng IPv6, có thể xuất hiện thêm các rule tương ứng cho `(v6)`.

Ảnh minh họa:

> Chèn ảnh Terminal hiển thị kết quả `sudo ufw status verbose` tại đây.

---

## 8. Cấu hình DigitalOcean Cloud Firewall

Ngoài UFW trên Ubuntu, tôi cấu hình thêm Cloud Firewall trên DigitalOcean.

Trong DigitalOcean Console, tôi tạo một Firewall mới và cấu hình các Inbound Rules.

Các rule cần thiết:

| Type | Protocol | Port | Sources                       |
| ---- | -------- | ---- | ----------------------------- |
| SSH  | TCP      | 22   | All IPv4/IPv6 hoặc IP cá nhân |
| HTTP | TCP      | 80   | All IPv4/IPv6                 |

SSH được sử dụng để quản trị Droplet.

HTTP được sử dụng để truy cập website Nginx.

Các cổng dịch vụ khác không được thêm vào Inbound Rules.

Sau khi tạo Cloud Firewall, tôi gán Droplet đang sử dụng vào Firewall.

Ảnh minh họa:

> Chèn ảnh DigitalOcean Console hiển thị Cloud Firewall và các Inbound Rules tại đây.

---

## 9. Kiểm tra kết nối SSH

Từ máy tính cá nhân, tôi kiểm tra lại khả năng kết nối SSH:

```bash
ssh -i ~/.ssh/id_ed25519 devops@IP_ADDRESS_DROPLET
```

Nếu đang sử dụng tài khoản `root` theo yêu cầu bài trước:

```bash
ssh -i ~/.ssh/id_ed25519 root@IP_ADDRESS_DROPLET
```

Nếu kết nối thành công thì chứng minh port 22 vẫn được cho phép.

---

## 10. Kiểm tra HTTP

Từ trình duyệt trên máy cá nhân, truy cập:

```text
http://IP_ADDRESS_DROPLET
```

Nếu Nginx đang hoạt động và port 80 được mở, website PTIT từ bài trước sẽ được hiển thị.

Có thể kiểm tra thêm bằng:

```bash
curl http://IP_ADDRESS_DROPLET
```

Kết quả trả về nội dung HTML của website.

---

## 11. Kiểm tra port không được mở

Để kiểm tra nguyên tắc chỉ mở những dịch vụ cần thiết, tôi thử kết nối tới một port khác không được cấu hình trong UFW.

Ví dụ có thể sử dụng:

```bash
nc -zv IP_ADDRESS_DROPLET 8080
```

Nếu port 8080 không có dịch vụ và không được firewall cho phép, kết nối sẽ không thành công.

Có thể kiểm tra thêm một port khác:

```bash
nc -zv IP_ADDRESS_DROPLET 3000
```

Kết quả có thể hiển thị timeout hoặc connection refused tùy thuộc vào cách firewall và dịch vụ xử lý kết nối.

---

## 12. Kết quả

Sau khi thực hiện bài tập, tôi đã cấu hình tường lửa theo hai lớp.

### UFW

Chính sách mặc định:

```text
Incoming: DENY
Outgoing: ALLOW
```

Các port được phép:

```text
22/tcp
80/tcp
```

UFW hoạt động với trạng thái:

```text
Status: active
```

### DigitalOcean Cloud Firewall

Inbound Rules chỉ cho phép:

```text
SSH  - TCP 22
HTTP - TCP 80
```

Sau khi cấu hình, tôi vẫn có thể:

* SSH vào Droplet thông qua port 22.
* Truy cập website Nginx thông qua port 80.
* Không truy cập được các port khác không được mở.

---

## 13. Các lệnh chính đã sử dụng

```bash
sudo ufw status verbose

sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp
sudo ufw allow 80/tcp

sudo ufw enable

sudo ufw status verbose
```

Kiểm tra HTTP:

```bash
curl http://IP_ADDRESS_DROPLET
```

Kiểm tra port khác:

```bash
nc -zv IP_ADDRESS_DROPLET 8080
```

---

## 14. Cấu trúc thư mục bài tập

```text
homework/
└── session_02/
    └── ex4/
        └── README.md
```

README chứa các bước cấu hình UFW, cấu hình DigitalOcean Cloud Firewall và kết quả kiểm tra.
