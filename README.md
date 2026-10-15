# DevOps Final – Đề 001: Quản lý phòng Lab

Bộ mã nguồn hoàn chỉnh cho bài thi **Hackathon DevOps Fundamentals – Cuối kỳ**.
Ứng dụng hiển thị **danh sách phòng lab** lấy từ MySQL thông qua Spring Boot, trả về **dạng JSON** qua Nginx (reverse proxy).

---

## 📖 HƯỚNG DẪN THỰC HÀNH & CHỤP ẢNH MINH CHỨNG
👉 **Xem hướng dẫn chi tiết từng bước từ A đến Z tại:** [HUONG_DAN_CHAY_VA_CHUP_ANH.md](HUONG_DAN_CHAY_VA_CHUP_ANH.md)

---

## Thông tin sinh viên & Triển khai

- **Sinh viên**: Phan Thuận Thành
- **Mã sinh viên**: B24DTCN169
- **Lớp**: K24CNTT2
- **GitHub**: [Phanthuanthanh/devops-final-de001-phanthuanthanh](https://github.com/Phanthuanthanh/devops-final-de001-phanthuanthanh)
- **Cổng cá nhân (APP_PORT)**: `8169`
- **Server VPS**: `20.189.122.108`
- **Địa chỉ ứng dụng**: http://20.189.122.108:8169

---

## Kiến trúc hệ thống

```
Trình duyệt ── http://20.189.122.108:8169
    ▼
[nginx]  nginx:1.27-alpine        GET /*  → app:8080 (chuyển hướng / -> /api/labs)
    ▼
[app]    Spring Boot 4 (JRE 21)   GET /api/labs → mysql:3306
    ▼
[mysql]  mysql:8.0                volume mysql_data, KHÔNG public cổng mysql
```

---

## Cấu trúc thư mục

```
devops-final-de001-phanthuanthanh/
├── src/main/java/com/ptit/devops/lab/   ← Controller, Entity, Repository
├── src/main/resources/application.properties
├── build.gradle                   ← Build ra file build/libs/app.jar
├── settings.gradle
├── Dockerfile                     ← Multi-stage build (Gradle 8.14 JDK 21 -> JRE 21 non-root)
├── .dockerignore
├── nginx/default.conf             ← Nginx reverse proxy
├── database/init.sql              ← Bảng labs + 8 bản ghi mẫu
├── docker-compose.yml             ← Cấu hình 3 services (mysql, app, nginx)
├── .github/workflows/ci-cd.yml    ← Pipeline CI/CD: Build & Test -> Push GHCR -> SSH Deploy
├── student.env                    ← Thông tin sinh viên và cấu hình môi trường
├── screenshots/                   ← Chứa 4 ảnh minh chứng theo yêu cầu
├── submission-content/            ← Chứa file minh chứng nộp bài RAIA
├── .gitignore
├── HUONG_DAN_CHAY_VA_CHUP_ANH.md  ← Cẩm nang chi tiết từng thao tác
└── README.md
```

---

## Các lệnh chạy thử nghiệm

```bash
# Khởi động toàn bộ hệ thống bằng Docker Compose
docker compose --env-file student.env up -d --build

# Kiểm tra trạng thái containers
docker compose --env-file student.env ps

# Kiểm tra API
curl http://localhost:8169/api/labs

# Dọn dẹp
docker compose --env-file student.env down
```
