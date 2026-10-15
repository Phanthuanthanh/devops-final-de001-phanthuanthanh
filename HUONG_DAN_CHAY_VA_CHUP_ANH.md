# HƯỚNG DẪN CHI TIẾT TRIỂN KHAI VÀ CHỤP ẢNH MINH CHỨNG (ĐỀ 001)

---

## I. THÔNG TIN SINH VIÊN & THÔNG SỐ ĐÃ CẤU HÌNH

- **Họ và tên**: Phan Thuận Thành
- **Mã sinh viên**: B24DTCN169
- **Lớp**: K24CNTT2
- **Email**: thanhpt.b24dtcn169@stu.ptit.edu.vn / phanthuanthanh270606@gmail.com
- **GitHub Username**: Phanthuanthanh
- **Tên Repository GitHub**: `devops-final-de001-phanthuanthanh` (nhánh `main`, Public)
- **IP VPS (Server chung)**: `20.189.122.108`
- **Cổng cá nhân (APP_PORT)**: `8169` (tránh trùng lặp với các sinh viên khác)
- **Tên Compose Project**: `phanthuanthanh-de001`
- **MySQL Database**: `ptit_lab_db`
- **MySQL User / Password**: `app_user` / `270606`
- **MySQL Root Password**: `270606`

> **Lưu ý quan trọng**: Toàn bộ các file mã nguồn (`Dockerfile`, `docker-compose.yml`, `default.conf`, `ci-cd.yml`, `student.env`, `build.gradle`, `application.properties`) đã được cấu hình chuẩn chỉnh 100% và **hoàn toàn không chứa comment (chú thích)** đúng theo yêu cầu.

---

## II. QUY TRÌNH TRIỂN KHAI TỪNG BƯỚC

### BƯỚC 1: TẠO REPOSITORY TRÊN GITHUB & PUSH MÃ NGUỒN

1. Đăng nhập vào GitHub với tài khoản `Phanthuanthanh`.
2. Tạo một repository mới:
   - **Repository name**: `devops-final-de001-phanthuanthanh`
   - **Visibility**: Chọn **Public** (Bắt buộc theo yêu cầu đề thi)
   - **Không** tick chọn tạo README, .gitignore hay license (vì đã có sẵn trong source).
3. Mở terminal PowerShell tại thư mục dự án `C:\Users\pc\Downloads\devops-final-app-de001` và chạy các lệnh sau:

```powershell
git init
git branch -M main
git add .
git commit -m "feat: complete devops project without comments"
git remote add origin https://github.com/Phanthuanthanh/devops-final-de001-phanthuanthanh.git
git push -u origin main
```

---

### BƯỚC 2: CẤU HÌNH QUYỀN GITHUB ACTIONS & GHCR

Để GitHub Actions có thể push Docker image lên GitHub Container Registry (GHCR):

1. Trên GitHub, vào repository `devops-final-de001-phanthuanthanh` -> **Settings** -> **Actions** -> **General**.
2. Cuộn xuống mục **Workflow permissions**:
   - Chọn **Read and write permissions**.
   - Tick chọn **Allow GitHub Actions to create and approve pull requests**.
   - Bấm **Save**.

---

### BƯỚC 3: CẤU HÌNH SSH VÀ GITHUB SECRETS

#### 1. Tạo cặp khóa SSH cho pipeline:
Mở PowerShell trên máy tính cá nhân hoặc trong terminal VPS:

```powershell
ssh-keygen -t ed25519 -f deploy_key -N ""
```

Lệnh trên tạo ra 2 file:
- `deploy_key`: Khóa bí mật (Private Key)
- `deploy_key.pub`: Khóa công khai (Public Key)

#### 2. Thêm Public Key vào VPS (`20.189.122.108`):
Đăng nhập vào VPS với tài khoản của bạn (ví dụ: `ubuntu` hoặc `thanh` hoặc `root`):

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat deploy_key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

*(Đảm bảo tài khoản SSH đã thuộc group `docker` bằng lệnh: `sudo usermod -aG docker $USER`)*

#### 3. Cấu hình GitHub Secrets:
Trên GitHub, vào repository `devops-final-de001-phanthuanthanh` -> **Settings** -> **Secrets and variables** -> **Actions** -> Bấm **New repository secret** và thêm 3 secrets sau:

1. **`SERVER_HOST`**:
   - Value: `20.189.122.108`
2. **`SERVER_USER`**:
   - Value: Tên tài khoản SSH của bạn trên VPS (ví dụ: `ubuntu` hoặc `thanh` hoặc `root`)
3. **`SERVER_SSH_KEY`**:
   - Value: Toàn bộ nội dung của file `deploy_key` (bắt đầu từ `-----BEGIN OPENSSH PRIVATE KEY-----` cho đến `-----END OPENSSH PRIVATE KEY-----`).

---

### BƯỚC 4: KÍCH HOẠT CI/CD PIPELINE

Sau khi cấu hình Secrets xong, bạn kích hoạt pipeline bằng 1 trong 2 cách:
- **Cách 1 (Khuyên dùng)**: Vào tab **Actions** trên GitHub -> Chọn workflow **CI/CD** ở cột bên trái -> Bấm **Run workflow** -> Chọn branch `main` -> Bấm nút xanh **Run workflow**.
- **Cách 2**: Thực hiện một thay đổi nhỏ rồi `git push origin main`.

Pipeline sẽ chạy 2 jobs liên tiếp:
1. **Build & Test**: Kiểm tra `student.env`, chạy Docker Compose test hệ thống, test endpoint `/api/labs`, login GHCR và push image với 2 tag (commit SHA và `latest`).
2. **Deploy**: SSH vào VPS `20.189.122.108`, pull image mới từ GHCR và khởi động hệ thống qua Docker Compose không cần build lại.

---

## III. HƯỚNG DẪN CHI TIẾT CHỤP 4 ẢNH MINH CHỨNG (PHẦN 5)

Tất cả 4 ảnh minh chứng phải được lưu vào thư mục `screenshots/` với định dạng tên chuẩn xác:

### 1. Ảnh `01-actions.png`:
- **Nơi chụp**: Trình duyệt, vào repository GitHub -> tab **Actions**.
- **Nội dung cần thấy rõ**:
  - Tên repository: `devops-final-de001-phanthuanthanh`
  - Nhánh: `main`
  - Workflow run gần nhất có icon tích xanh ✅ **Success**.
  - Bấm vào chi tiết run: hiển thị rõ cả 2 job **Build & Test** và **Deploy** đều màu xanh lá cây thành công.
- **Lưu file**: `screenshots/01-actions.png`.

---

### 2. Ảnh `02-ghcr.png`:
- **Nơi chụp**: Trình duyệt trên GitHub.
  - Vào trang chủ tài khoản GitHub -> tab **Packages** (hoặc ở cột bên phải trang repo mục **Packages** -> click vào package `devops-final-de001-phanthuanthanh/app`).
- **Nội dung cần thấy rõ**:
  - Tên image package: `ghcr.io/phanthuanthanh/devops-final-de001-phanthuanthanh/app`
  - Mục **OS/Arch & Tags** hiển thị rõ ràng cả 2 tags:
    - Tag commit SHA (ví dụ: `a1b2c3d...`)
    - Tag `latest`
- **Lưu file**: `screenshots/02-ghcr.png`.

---

### 3. Ảnh `03-compose-ps.png`:
- **Nơi chụp**: Cửa sổ Terminal (SSH vào VPS `20.189.122.108`).
- **Thực hiện lệnh**:
  ```bash
  cd ~/devops-final-de001-phanthuanthanh
  docker compose --env-file student.env ps
  ```
- **Nội dung cần thấy rõ**:
  - Dòng lệnh: `docker compose --env-file student.env ps`
  - Danh sách 3 containers: `mysql`, `app`, `nginx` với project name `phanthuanthanh-de001` đều có trạng thái **Up / running** (hoặc `healthy`).
  - Cột Ports hiển thị rõ: `0.0.0.0:8169->80/tcp` của container `nginx`.
- **Lưu file**: `screenshots/03-compose-ps.png`.

---

### 4. Ảnh `04-website.png`:
- **Nơi chụp**: Trình duyệt web (Chrome, Edge hoặc Firefox).
- **Thực hiện truy cập**:
  - Thanh địa chỉ: `http://20.189.122.108:8169` (hoặc `http://20.189.122.108:8169/api/labs`)
- **Nội dung cần thấy rõ**:
  - Thanh URL của trình duyệt hiển thị rõ: `20.189.122.108:8169` (hoặc `20.189.122.108:8169/api/labs`).
  - Màn hình hiển thị dữ liệu JSON danh sách các phòng Lab gồm 8 bản ghi (DevOps Lab, Cloud Lab, Network Lab, AI Lab, IoT Lab, Security Lab, Mobile Lab, Data Lab).
- **Lưu file**: `screenshots/04-website.png`.

---

## IV. BƯỚC CUỐI CÙNG: ĐÓNG GÓI VÀ NỘP BÀI (RAIA)

Sau khi đã lưu đủ 4 file ảnh vào `screenshots/`:

1. Copy toàn bộ 4 ảnh sang thư mục `submission-content/`:
   ```powershell
   Copy-Item screenshots/* submission-content/
   ```
2. Đổi tên thư mục gốc của bài thi (nếu cần theo quy chế thi): `devops-final-de001-phanthuanthanh`.
3. Commit và push các ảnh minh chứng lên GitHub:
   ```powershell
   git add screenshots/ submission-content/
   git commit -m "docs: add verification screenshots"
   git push origin main
   ```
4. Dùng SFTP (WinSCP, FileZilla, hoặc zip) tải toàn bộ thư mục bài làm và nộp lên hệ thống RAIA theo đúng thời hạn quy định.
