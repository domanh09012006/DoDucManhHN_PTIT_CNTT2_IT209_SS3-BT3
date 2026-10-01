# Bài 3: Bảo mật tài nguyên bằng HTTP Basic Authentication

## 1. Mục tiêu

Cấu hình Nginx sử dụng HTTP Basic Authentication để bảo vệ tài nguyên `/admin`.

**Yêu cầu thực hiện:**
- Cài đặt `apache2-utils`.
- Tạo tài khoản `admin_user`.
- Lưu mật khẩu đã băm tại `/etc/nginx/.htpasswd`.
- Cấu hình Nginx bảo vệ `/admin`.
- Kiểm tra truy cập không có tài khoản và có tài khoản hợp lệ.

---

## 2. Cài đặt công cụ

Cài đặt `apache2-utils`:

```bash
apt update
apt install apache2-utils -y
```

> **Ghi chú:** Công cụ `htpasswd` được sử dụng để tạo và lưu mật khẩu dưới dạng đã băm.

---

## 3. Tạo tài khoản

Tạo file mật khẩu tại `/etc/nginx/.htpasswd`:

```bash
htpasswd -c /etc/nginx/.htpasswd admin_user
```

> **Lưu ý:** File mật khẩu được đặt ngoài thư mục `/var/www/` để tránh bị truy cập trực tiếp thông qua Web Server.

---

## 4. Cấu hình Nginx

File Server Block:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/ptit-web/html;
    index index.html;

    location /admin {
        auth_basic "Restricted Admin Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
        try_files $uri $uri/ =404;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Sau khi chỉnh sửa cấu hình, kiểm tra:

```bash
nginx -t
```

**Kết quả:**
```text
syntax is ok
test is successful
```

Sau đó reload Nginx:

```bash
systemctl reload nginx
```

---

## 5. Tạo tài nguyên Admin

Tạo thư mục:

```bash
mkdir -p /var/www/ptit-web/html/admin
```

Tạo file `/var/www/ptit-web/html/admin/index.html` với nội dung:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Admin Area</title>
</head>
<body>
    <h1>PTIT Admin Area</h1>
    <p>Restricted Admin Area</p>
</body>
</html>
```

---

## 6. Kiểm tra khi chưa xác thực

Sử dụng lệnh:

```bash
curl -I http://221.121.3.177/admin/
```

**Kết quả:**
```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm="Restricted Admin Area"
```

> *Điều này chứng minh tài nguyên `/admin/` yêu cầu xác thực.*

---

## 7. Kiểm tra với tài khoản hợp lệ

Sử dụng lệnh:

```bash
curl -u admin_user:<PASSWORD> -I http://221.121.3.177/admin/
```

**Kết quả:**
```http
HTTP/1.1 200 OK
```

> *Điều này chứng minh tài khoản hợp lệ có thể truy cập tài nguyên `/admin/`.*

---

## 8. Kết quả

Đã hoàn thành cấu hình HTTP Basic Authentication cho Nginx:
- **Tài khoản:** `admin_user`
- **File mật khẩu:** `/etc/nginx/.htpasswd`
- **Tài nguyên được bảo vệ:** `/admin/`
- **Chưa xác thực:** `401 Unauthorized`
- **Xác thực đúng:** `200 OK`

---

## 9. Hình ảnh minh chứng

### Ảnh 1: Cấu hình Nginx
Chụp kết quả:
```bash
nginx -t
```
*(Chèn ảnh tại đây)*

### Ảnh 2: Truy cập không có tài khoản
Chụp kết quả:
```bash
curl -I http://221.121.3.177/admin/
```
Hiển thị:
```http
HTTP/1.1 401 Unauthorized
```
*(Chèn ảnh tại đây)*

### Ảnh 3: Truy cập với tài khoản hợp lệ
Chụp kết quả:
```bash
curl -u admin_user:<PASSWORD> -I http://221.121.3.177/admin/
```
Hiển thị:
```http
HTTP/1.1 200 OK
```
*(Chèn ảnh tại đây)*

---

## 10. Cấu trúc thư mục nộp bài

```text
session_03/
└── ex3/
    ├── README.md
    ├── ptit-web.conf
    └── .htpasswd
```