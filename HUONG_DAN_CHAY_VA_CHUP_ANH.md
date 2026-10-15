# HƯỚNG DẪN CHI TIẾT TỪ A - Z: THỰC HÀNH DEVOPS & CHỤP ẢNH MINH CHỨNG (ĐỀ 001)

> **Cẩm nang thi thực hành DevOps**: Tài liệu hướng dẫn từng bước từ cấu hình, đẩy code, tạo SSH key, cấu hình Secrets, chạy CI/CD đến cách chụp đủ 4 ảnh minh chứng chuẩn 100% yêu cầu đề thi.

---

## MỤC LỤC
1. [Thông số đề thi cá nhân](#i-thông-số-đề-thi-cá-nhân)
2. [Bước 1: Khởi tạo Git & Đẩy code lên GitHub](#bước-1-khởi-tạo-git--đẩy-code-lên-github)
3. [Bước 2: Phân quyền GitHub Actions & GHCR](#bước-2-phân-quyền-github-actions--ghcr)
4. [Bước 3: Cấu hình SSH Key & GitHub Secrets](#bước-3-cấu-hình-ssh-key--github-secrets)
5. [Bước 4: Chạy CI/CD Pipeline & Kiểm tra](#bước-4-chạy-cicd-pipeline--kiểm-tra)
6. [Bước 5: Hướng dẫn chụp 4 ảnh minh chứng](#bước-5-hướng-dẫn-chụp-4-ảnh-minh-chứng)
7. [Bước 6: Đóng gói và Nộp bài (RAIA)](#bước-6-đóng-gói-và-nộp-bài-raia)
8. [Các lỗi thường gặp và cách xử lý (Troubleshooting)](#các-lỗi-thường-gặp-và-cách-xử-lý-troubleshooting)

---

## I. THÔNG SỐ ĐỀ THI CÁ NHÂN

| Mục | Thông tin chuẩn | Ghi chú |
| :--- | :--- | :--- |
| **Họ và tên** | Phan Thuận Thành | Điền trong `student.env` |
| **Mã sinh viên** | B24DTCN169 | Điền trong `student.env` |
| **Lớp** | K24CNTT2 | Điền trong `student.env` |
| **Email** | `thanh.pt@stu.ptit.edu.vn` | Điền trong `student.env` |
| **GitHub Account** | `Phanthuanthanh` | Tài khoản cá nhân |
| **GitHub Repository** | `devops-final-de001-phanthuanthanh` | Bắt buộc chế độ **Public**, nhánh `main` |
| **IP VPS (Server)** | `20.189.122.108` | Máy chủ triển khai |
| **SSH User trên VPS** | `thanhpt2706` | Tài khoản đăng nhập SSH |
| **Mật khẩu VPS & DB** | `270606` | Dùng cho SSH và MySQL |
| **Cổng cá nhân (APP_PORT)** | `8169` | Lấy theo đuôi MSSV 169 (tránh trùng lặp) |
| **COMPOSE_PROJECT_NAME** | `phanthuanthanh-de001` | Tên project container trên VPS |
| **Database Name** | `ptit_lab_db` | KHÔNG được sửa |
| **MySQL User / Password** | `app_user` / `270606` | Tài khoản ứng dụng kết nối DB |
| **MySQL Root Password** | `270606` | Mật khẩu quản trị DB `root` |

---

## BƯỚC 1: KHỞI TẠO GIT & ĐẨY CODE LÊN GITHUB

### 1. Tạo Repository trên web GitHub:
1. Đăng nhập vào GitHub `Phanthuanthanh`.
2. Bấm vào dấu `+` ở góc trên bên phải -> Chọn **New repository**.
3. Điền các mục:
   - **Repository name**: `devops-final-de001-phanthuanthanh`
   - **Visibility**: Chọn **Public** (Đề thi bắt buộc).
   - **Initialize this repository with**: Để trống tất cả (KHÔNG tick README, .gitignore, license).
4. Bấm **Create repository**.

### 2. Đẩy toàn bộ mã nguồn từ máy lên GitHub:
Mở cửa sổ **PowerShell** tại thư mục dự án trên máy (`C:\Users\pc\Downloads\devops-final-app-de001`), chạy lần lượt các lệnh sau:

```powershell
git init
git branch -M main
git add .
git commit -m "feat: complete devops project"
git remote add origin https://github.com/Phanthuanthanh/devops-final-de001-phanthuanthanh.git
git push -u origin main
```
*(Nếu đã có remote thì chạy lệnh `git push origin main`)*.

---

## BƯỚC 2: PHÂN QUYỀN GITHUB ACTIONS & GHCR

Để GitHub Actions có quyền tải (push) Docker image lên kho lưu trữ **GitHub Container Registry (GHCR)**:

1. Vào repository trên GitHub: [devops-final-de001-phanthuanthanh](https://github.com/Phanthuanthanh/devops-final-de001-phanthuanthanh)
2. Chọn tab **Settings** (ở thanh menu trên cùng).
3. Ở cột bên trái, tìm mục **Actions** -> bấm vào **General**.
4. Cuộn chuột xuống mục **Workflow permissions**:
   - Chọn vào nút radio: **Read and write permissions**.
   - Tick chọn ô: **Allow GitHub Actions to create and approve pull requests**.
5. Bấm nút **Save** màu xanh lá.

---

## BƯỚC 3: CẤU HÌNH SSH KEY & GITHUB SECRETS

### Cách nhanh nhất và chuẩn nhất (Thao tác trực tiếp trên VPS):

#### 1. Mở terminal VPS (`thanhpt2706@thanhpt2706:~$`):
Chạy lệnh tạo khóa SSH:
```bash
cd ~
ssh-keygen -t ed25519 -f deploy_key -N ""
```
*(Lệnh này sinh ra 2 file: `deploy_key` là khóa bí mật, `deploy_key.pub` là khóa công khai).*

#### 2. Nạp khóa công khai vào danh sách cho phép trên VPS:
```bash
mkdir -p ~/.ssh
cat deploy_key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

#### 3. In khóa bí mật ra màn hình để copy:
```bash
cat deploy_key
```
Màn hình sẽ hiển thị ra một đoạn mã có dạng:
```text
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
... (nội dung khóa) ...
-----END OPENSSH PRIVATE KEY-----
```
👉 Bạn hãy **dùng chuột bôi đen toàn bộ đoạn mã này** (bao gồm cả 2 dòng BEGIN và END) rồi bấm chuột phải / copy.

#### 4. Cấp quyền Docker cho tài khoản VPS:
*(Đảm bảo user có quyền chạy lệnh docker không cần sudo)*:
```bash
sudo usermod -aG docker $USER
```

---

### Cấu hình 3 Secrets trên GitHub:
1. Vào repository trên GitHub -> **Settings** -> **Secrets and variables** -> **Actions**.
2. Bấm nút **New repository secret** để thêm lần lượt 3 biến:

| Tên Secret (Name) | Giá trị (Secret Value) |
| :--- | :--- |
| **`SERVER_HOST`** | `20.189.122.108` |
| **`SERVER_USER`** | `thanhpt2706` |
| **`SERVER_SSH_KEY`** | Dán toàn bộ nội dung khóa bí mật bạn vừa copy từ lệnh `cat deploy_key` ở trên vào |

---

## BƯỚC 4: CHẠY CI/CD PIPELINE & KIỂM TRA

1. Vào tab **Actions** trên repository GitHub:
   👉 [https://github.com/Phanthuanthanh/devops-final-de001-phanthuanthanh/actions](https://github.com/Phanthuanthanh/devops-final-de001-phanthuanthanh/actions)
2. Nếu trước đó có lần chạy bị lỗi (đỏ), hãy click vào lần chạy đó -> bấm nút **Re-run all jobs**.
3. Nếu chưa chạy, bấm vào workflow **CI/CD** ở cột bên trái -> bấm **Run workflow** -> chọn nhánh `main` -> bấm nút xanh **Run workflow**.
4. Quá trình chạy gồm 2 Job:
   - **Job 1 (Build & Test)**: Kiểm tra biến môi trường `student.env`, build Docker Compose chạy thử, smoke test kiểm tra `/api/labs` trả về JSON, đăng nhập GHCR và push image với 2 tag commit SHA và `latest`.
   - **Job 2 (Deploy lên server)**: Tự động SSH vào VPS `20.189.122.108`, kéo image mới nhất từ GHCR về và khởi chạy Docker Compose trên server.
5. Khi cả 2 job hiện dấu tích xanh ✅ **Success** là hoàn thành triển khai!

---

## BƯỚC 5: HƯỚNG DẪN CHỤP 4 ẢNH MINH CHỨNG

Tất cả ảnh phải chụp rõ ràng và lưu vào thư mục `screenshots/`:

### 📸 ẢNH 1: `01-actions.png`
- **Địa chỉ**: [Tab Actions trên GitHub](https://github.com/Phanthuanthanh/devops-final-de001-phanthuanthanh/actions)
- **Thao tác**: Click vào lần chạy gần nhất thành công.
- **Yêu cầu hiển thị**:
  - Tên repo: `devops-final-de001-phanthuanthanh`.
  - Nhánh: `main`.
  - Cả 2 job: **Build & Test** và **Deploy lên server** đều màu xanh lá cây ✅.
- **Lưu tên file**: `screenshots/01-actions.png`.

---

### 📸 ẢNH 2: `02-ghcr.png`
- **Địa chỉ**: [GitHub Packages GHCR](https://github.com/Phanthuanthanh?tab=packages) hoặc mục **Packages** ở bên phải trang repo.
- **Thao tác**: Click vào package `app` (`devops-final-de001-phanthuanthanh/app`).
- **Yêu cầu hiển thị**:
  - Tên image: `ghcr.io/phanthuanthanh/devops-final-de001-phanthuanthanh/app`.
  - Bảng **OS/Arch & Tags** hiển thị rõ ràng 2 tags:
    - Tag commit SHA (dãy ký tự mã băm commit).
    - Tag `latest`.
- **Lưu tên file**: `screenshots/02-ghcr.png`.

---

### 📸 ẢNH 3: `03-compose-ps.png`
- **Nơi chụp**: Cửa sổ Terminal SSH trên VPS (`thanhpt2706@thanhpt2706:~$`).
- **Thao tác chạy lệnh**:
  ```bash
  cd ~/devops-final-de001-phanthuanthanh
  docker compose --env-file student.env ps
  ```
  *(Lưu ý: Bắt buộc phải có lệnh `cd ~/devops-final-de001-phanthuanthanh` trước để vào thư mục chứa file `student.env`)*.
- **Yêu cầu hiển thị**:
  - Dòng lệnh: `docker compose --env-file student.env ps`.
  - Danh sách 3 container đang chạy:
    - `phanthuanthanh-de001-app-1`: trạng thái `Up`.
    - `phanthuanthanh-de001-mysql-1`: trạng thái `Up (healthy)`.
    - `phanthuanthanh-de001-nginx-1`: trạng thái `Up`, cổng `0.0.0.0:8169->80/tcp`.
- **Lưu tên file**: `screenshots/03-compose-ps.png`.

---

### 📸 ẢNH 4: `04-website.png`
- **Nơi chụp**: Trình duyệt web (Google Chrome, Microsoft Edge, hoặc Firefox).
- **Thao tác**: Nhập địa chỉ:
  👉 **`http://20.189.122.108:8169`** *(hoặc `http://20.189.122.108:8169/api/labs`)*.
- **Yêu cầu hiển thị**:
  - Thanh địa chỉ URL của trình duyệt thấy rõ: `20.189.122.108:8169`.
  - Màn hình trang web hiển thị mảng dữ liệu JSON danh sách 8 phòng Lab (DevOps Lab, Cloud Lab, Network Lab, AI Lab, IoT Lab, Security Lab, Mobile Lab, Data Lab).
- **Lưu tên file**: `screenshots/04-website.png`.

---

## BƯỚC 6: ĐÓNG GÓI VÀ NỘP BÀI (RAIA)

1. **Sao chép ảnh minh chứng**:
   Copy toàn bộ 4 ảnh vừa chụp từ `screenshots/` sang thư mục `submission-content/`:
   ```powershell
   Copy-Item screenshots/* submission-content/
   ```

2. **Commit và đẩy ảnh lên GitHub**:
   ```powershell
   git add screenshots/ submission-content/
   git commit -m "docs: add all proof screenshots"
   git push origin main
   ```

3. **Nộp bài lên hệ thống RAIA**:
   - Nén thư mục bài làm `devops-final-de001-phanthuanthanh` thành file `.zip`.
   - Tải file `.zip` nộp lên hệ thống RAIA theo đúng thời hạn quy định của giáo viên.

---

## CÁC LỖI THƯỜNG GẶP VÀ CÁCH XỬ LÝ (TROUBLESHOOTING)

### 1. Lỗi `couldn't find env file: /home/.../student.env`:
- **Nguyên nhân**: Bạn đang đứng ở thư mục gốc `~` của VPS nên Docker không tìm thấy file `student.env`.
- **Cách khắc phục**: Phải gõ `cd ~/devops-final-de001-phanthuanthanh` trước rồi mới chạy lệnh `docker compose --env-file student.env ps`.

---

### 2. Lỗi `deploy_key: command not found`:
- **Nguyên nhân**: Gõ tên file trực tiếp như một câu lệnh. Trong Linux, `deploy_key` là file dữ liệu văn bản.
- **Cách khắc phục**: Dùng lệnh `cat` để đọc nội dung file:
  ```bash
  cat ~/deploy_key.pub   # Xem Public Key
  cat ~/deploy_key       # Xem Private Key
  ```

---

### 3. Lỗi `group 'docker' does not exist` khi chạy `usermod`:
- **Nguyên nhân**: VPS mới tinh chưa cài đặt dịch vụ Docker.
- **Cách khắc phục**: Cài đặt Docker trên VPS bằng lệnh:
  ```bash
  sudo apt-get update && sudo apt-get install -y docker.io docker-compose-v2
  sudo usermod -aG docker $USER
  ```

---

### 4. Lỗi Job Deploy trên GitHub Actions báo đỏ (SSH thất bại):
- **Nguyên nhân**:
  - Điền sai `SERVER_HOST` (phải là `20.189.122.108`).
  - Điền sai `SERVER_USER` (phải là `thanhpt2706`).
  - Điền thiếu hoặc thừa ký tự trong `SERVER_SSH_KEY` (phải copy trọn vẹn từ `-----BEGIN OPENSSH PRIVATE KEY-----` đến `-----END OPENSSH PRIVATE KEY-----`).
  - Chưa thêm `deploy_key.pub` vào `~/.ssh/authorized_keys` trên VPS.
- **Cách khắc phục**: Kiểm tra và cập nhật lại 3 biến Secrets trên GitHub.

---

### 5. Tại sao không được đặt `MYSQL_USER="root"` trong `student.env`?
- **Giải thích kỹ thuật**: Trong official Docker image `mysql:8.0`, biến `MYSQL_USER` dùng để tạo tài khoản thường. Nếu gán bằng `"root"`, MySQL container sẽ bị lỗi `[ERROR] [Entrypoint]: MYSQL_USER="root", MYSQL_USER and MYSQL_PASSWORD are for configuring a regular user and cannot be used for the root user` và dừng ngay lập tức (`exit 1`).
- Mật khẩu root đã được định nghĩa qua `MYSQL_ROOT_PASSWORD="270606"`.
- Tài khoản ứng dụng `MYSQL_USER="app_user"` với `MYSQL_PASSWORD="270606"` đảm bảo toàn bộ hệ thống chạy ổn định 100%.
