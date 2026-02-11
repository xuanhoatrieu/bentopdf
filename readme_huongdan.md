# Hướng dẫn Triển khai BentoPDF (Custom UI)

> Dự án sử dụng **CI/CD tự động**: Khi push code lên GitHub, GitHub Actions sẽ tự build Docker image và đẩy lên GitHub Container Registry (GHCR). Server chỉ cần pull image mới về.

---

## PHẦN 1: SỬA CODE & PUSH (Máy tính cá nhân)

### Bước 1: Lấy code về (Lần đầu hoặc máy mới)

```bash
git clone -b custom-ui https://github.com/xuanhoatrieu/bentopdf.git
cd bentopdf
npm install
```

### Bước 2: Sửa code & Test

- Sửa giao diện: `index.html`, `src/ui.ts`...
- Chạy thử: `npm run dev` → truy cập `localhost:3000`

### Bước 3: Push lên GitHub

```bash
git add .
git commit -m "Mô tả thay đổi"
git push origin custom-ui
```

Sau khi push, vào **GitHub → Actions** kiểm tra workflow **"Build & Deploy Custom UI"** chạy thành công ✅ (khoảng 3-5 phút).

---

## PHẦN 2: CẬP NHẬT TRÊN SERVER (Đã cài sẵn)

### Cách 1: Dùng Portainer (Giao diện web)

1. Đăng nhập **Portainer** → **Stacks** → chọn stack **bentopdf**
2. Bấm **Update the stack**
3. ✅ **BẬT** tùy chọn **"Pull latest image"**
4. Bấm **Update** → Xong!

### Cách 2: Dùng Terminal (SSH)

```bash
cd ~/bentopdf    # hoặc thư mục chứa docker-compose.yml

# Pull image mới từ GHCR
docker compose pull

# Khởi động lại container với image mới
docker compose up -d
```

---

## PHẦN 3: CÀI ĐẶT TRÊN SERVER MỚI

### Bước 1: Cài Docker & Docker Compose

```bash
# Cài Docker
curl -fsSL https://get.docker.com | sh

# Thêm user hiện tại vào group docker (khỏi cần sudo)
sudo usermod -aG docker $USER

# Đăng xuất rồi đăng nhập lại để có hiệu lực
exit
```

### Bước 2: Đăng nhập GHCR (Nếu repo private)

Tạo **Personal Access Token** trên GitHub: **Settings → Developer settings → Personal access tokens → Fine-grained tokens** (quyền **Packages: Read**).

```bash
echo "YOUR_TOKEN" | docker login ghcr.io -u xuanhoatrieu --password-stdin
```

> Nếu repo **public** thì bỏ qua bước này.

### Bước 3: Tạo thư mục & file docker-compose

```bash
mkdir -p ~/bentopdf && cd ~/bentopdf

cat > docker-compose.yml << 'EOF'
version: "3.8"
services:
  bentopdf:
    image: ghcr.io/xuanhoatrieu/bentopdf-custom:latest
    container_name: bentopdf
    ports:
      - "3300:8080"
    restart: unless-stopped
EOF
```

### Bước 4: Chạy

```bash
docker compose up -d
```

Truy cập `http://IP-Server:3300` → Xong! 🎉

---

## Tóm tắt Quy trình hàng ngày

```
Sửa code → git push origin custom-ui
                    ↓
        GitHub Actions tự build (3-5 phút)
                    ↓
        Server: docker compose pull && docker compose up -d
        hoặc Portainer: Update stack (bật Pull)
                    ↓
              Website cập nhật ✅
```

---

## Troubleshooting

| Vấn đề | Giải pháp |
|---------|-----------|
| GitHub Actions lỗi | Vào **Actions** tab → xem log chi tiết |
| `docker pull` bị denied | Kiểm tra đã `docker login ghcr.io` chưa |
| Web không lên | `docker logs bentopdf` để xem lỗi |
| Port bị chiếm | Đổi `3300` thành port khác trong docker-compose.yml |