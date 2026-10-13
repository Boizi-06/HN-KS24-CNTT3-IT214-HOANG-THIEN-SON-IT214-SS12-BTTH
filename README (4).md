# TASK MANAGER – Triển khai ứng dụng bằng Docker và VPS

## 1. Thông tin nhóm

| STT | Họ và tên | Phân công công việc |
|---:|---|---|
| 1 | **Hoàng Thiên Sơn** | Phụ trách Backend Spring Boot và MySQL bằng Docker Compose; cấu hình biến môi trường; kiểm tra API và phối hợp triển khai toàn hệ thống. |
| 2 | **Nguyễn Minh Duy** | Phụ trách Frontend React + Vite; cài đặt thư viện, chạy thử giao diện, kiểm tra gọi API và build bản production. |
| 3 | **Hà Quang Huy** | Phụ trách chuẩn bị VPS và Docker; quản lý container, kiểm tra trạng thái dịch vụ và log khi có lỗi. |
| 4 | **Lê Thành Long** | Phụ trách Nginx, tên miền và kiểm thử sau triển khai; cấu hình reverse proxy, kiểm tra truy cập qua tên miền và tổng hợp báo cáo. |

> Lưu ý: Đây là phương án phân công để trình bày trong báo cáo. Nhóm nên điều chỉnh nếu công việc thực tế được phân chia khác.

## 2. Liên kết mã nguồn

- **Frontend (FE):** https://github.com/minh-duy2608/btthss12_devops
- **Backend (BE):** https://github.com/Boizi-06/HN-KS24-CNTT3-IT214-HOANG-THIEN-SON-IT209-SS12-Demo

## 3. Mục tiêu

Xây dựng và triển khai ứng dụng Task Manager gồm giao diện Frontend, Backend REST API và cơ sở dữ liệu MySQL. Ứng dụng được đóng gói và vận hành bằng Docker trên VPS, sử dụng Nginx làm reverse proxy và tên miền để truy cập từ Internet.

## 4. Công nghệ sử dụng

- **Frontend:** React, Vite, JavaScript, HTML, CSS.
- **Backend:** Java 21, Spring Boot, REST API.
- **Database:** MySQL 8.
- **Container:** Docker, Docker Compose.
- **Web server / Reverse proxy:** Nginx.
- **Môi trường triển khai:** Ubuntu VPS.
- **Tên miền:** `app.boizi06.cloud` cho giao diện và `api.boizi06.cloud` cho API.

## 5. Kiến trúc triển khai

```text
Người dùng
    |
    | HTTP/HTTPS
    v
Nginx trên VPS
    |
    +---- app.boizi06.cloud
    |          |
    |          v
    |     Frontend React/Vite
    |     (file build trong /var/www/task-manager)
    |
    +---- api.boizi06.cloud
               |
               v
        Backend Spring Boot
        (127.0.0.1:8091 trên VPS)
               |
               v
          MySQL 8
        (Docker network nội bộ)
```

Nginx tiếp nhận yêu cầu từ bên ngoài và chuyển yêu cầu API đến Backend. Frontend hiển thị giao diện quản lý công việc và gửi yêu cầu HTTP đến API. Backend xử lý nghiệp vụ và đọc/ghi dữ liệu trong MySQL. Cơ sở dữ liệu không cần công khai cổng ra Internet.

## 6. Các API đã kiểm tra

| Phương thức | Endpoint | Chức năng | Kết quả mong đợi |
|---|---|---|---|
| `GET` | `/api/health` | Kiểm tra trạng thái Backend | HTTP 200, nội dung `OK` |
| `GET` | `/api/tasks` | Lấy danh sách công việc | HTTP 200, trả về mảng JSON |
| `POST` | `/api/tasks` | Tạo công việc mới | HTTP 201, trả về công việc vừa tạo |
| `DELETE` | `/api/tasks/{id}` | Xóa công việc theo ID | HTTP 204 khi xóa thành công |

Ví dụ dữ liệu tạo công việc:

```json
{
  "title": "Task Manager deployed on VPS"
}
```

## 7. Quy trình triển khai (Workflow)

1. **Chuẩn bị mã nguồn:** Hoàn thiện mã nguồn Frontend và Backend, kiểm tra cấu trúc thư mục và các tệp cấu hình.
2. **Đóng gói Backend:** Tạo Docker image cho ứng dụng Spring Boot bằng Dockerfile.
3. **Khởi chạy Backend và Database:** Dùng Docker Compose chạy Backend và MySQL; cấu hình thông tin kết nối bằng biến môi trường trong tệp `.env`.
4. **Kiểm tra Backend:** Kiểm tra container, log ứng dụng và các API như `/api/health`, `/api/tasks`.
5. **Build Frontend:** Cài đặt thư viện bằng npm và chạy lệnh build để tạo thư mục `dist` chứa các tài nguyên tĩnh.
6. **Đưa Frontend lên VPS:** Chuyển các tệp trong `dist` lên thư mục `/var/www/task-manager`.
7. **Cấu hình Nginx:** Tạo cấu hình riêng cho `api.boizi06.cloud` và `app.boizi06.cloud`; chuyển tiếp yêu cầu API tới Backend và phục vụ các tệp giao diện.
8. **Cấu hình DNS và HTTPS:** Trỏ tên miền về IP VPS và kiểm tra chứng chỉ TLS/HTTPS nếu đã cấu hình.
9. **Kiểm thử tổng thể:** Mở giao diện qua tên miền, kiểm tra danh sách công việc và thử các thao tác gọi API.
10. **Hoàn thiện báo cáo:** Ghi lại kiến trúc, công nghệ, phân công, quy trình triển khai và kết quả kiểm thử.

### Luồng xử lý khi người dùng sử dụng ứng dụng

```text
Người dùng mở app.boizi06.cloud
        |
        v
Nginx trả về giao diện Frontend
        |
        v
Frontend gửi request tới API
        |
        v
Nginx chuyển request tới Spring Boot
        |
        v
Spring Boot xử lý và truy vấn MySQL
        |
        v
Backend trả JSON về Frontend
        |
        v
Giao diện cập nhật kết quả cho người dùng
```

## 8. Các lệnh kiểm tra tham khảo

Chạy các lệnh sau **trên VPS** khi cần kiểm tra:

```bash
# Kiểm tra các container
docker compose ps

# Xem log dịch vụ
docker compose logs --tail=100 app
docker compose logs --tail=100 mysql

# Kiểm tra cấu hình Nginx
sudo nginx -t

# Nạp lại Nginx sau khi cấu hình hợp lệ
sudo systemctl reload nginx
```

Kiểm tra API từ máy cá nhân:

```bash
curl -i https://api.boizi06.cloud/api/health
curl -i https://api.boizi06.cloud/api/tasks
```

> Nếu HTTPS chưa được cấu hình hoặc kiểm tra thành công cho tên miền tương ứng, hãy dùng `http://` để kiểm tra tạm thời và chỉ báo cáo HTTPS là hoàn thành sau khi xác minh thực tế.

## 9. Kết quả đạt được

- Frontend React/Vite được build thành các tệp tĩnh để triển khai.
- Backend Spring Boot và MySQL được vận hành bằng Docker Compose trên VPS.
- API kiểm tra sức khỏe và API quản lý công việc đã được kiểm tra.
- Nginx được cấu hình để định tuyến yêu cầu tới Frontend và Backend qua các tên miền.
- Người dùng có thể truy cập giao diện và sử dụng API từ môi trường triển khai.

## 10. Kết luận

Qua bài thực hành, nhóm đã triển khai ứng dụng web gồm Frontend, Backend và Database lên VPS. Nhóm sử dụng Docker và Docker Compose để quản lý dịch vụ, Nginx làm web server/reverse proxy, đồng thời kiểm tra API và khả năng truy cập ứng dụng qua tên miền. Quy trình giúp nhóm hiểu rõ hơn cách đóng gói, triển khai, kết nối và kiểm thử các thành phần trong một hệ thống web.

---

**Tên bài:** Task Manager – Dockerized Web Application Deployment  
**Thành viên:** Hoàng Thiên Sơn, Nguyễn Minh Duy, Hà Quang Huy, Lê Thành Long
