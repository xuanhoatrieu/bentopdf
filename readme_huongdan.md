### PHẦN 1: TẠI MÁY TÍNH CÁ NHÂN (LOCAL)

*Mục tiêu: Sửa code, test và đẩy lên GitHub.*

**Bước 1: Lấy code về (Nếu chưa có hoặc muốn làm mới)**
Mở Terminal/CMD tại thư mục muốn lưu dự án:

```bash
# Clone dự án từ GitHub CỦA BẠN (nhánh custom-ui)
git clone -b custom-ui https://github.com/xuanhoatrieu/bentopdf.git

# Vào thư mục
cd bentopdf

# Cài đặt thư viện để test (chỉ làm 1 lần đầu)
npm install

```

**Bước 2: Sửa code và Test**

* Dùng VS Code sửa giao diện (file `src/index.html`, `src/ui.ts`...).
* Sửa `Dockerfile` (nếu cần thay đổi cấu hình build).
* Chạy thử xem ổn chưa: `npm run dev` (Truy cập `localhost:3000`).

**Bước 3: Lưu và Đẩy lên GitHub**
Sau khi sửa xong, chạy lần lượt 3 lệnh:

```bash
# 1. Thêm tất cả file đã sửa vào danh sách chờ
git add .

# 2. Lưu lại với ghi chú (thay nội dung trong ngoặc kép cho phù hợp)
git commit -m "Sửa giao diện ngày 01/01/2026"

# 3. Đẩy lên GitHub của bạn
git push origin custom-ui

```

---

### PHẦN 2: TẠI SERVER (SSH)

*Mục tiêu: Lấy code mới về server và đóng gói thành Image.*

**Bước 1: Kết nối SSH vào Server**

```bash
ssh user@ip-server-cua-ban

```

**Bước 2: Đồng bộ code mới nhất**
Chúng ta dùng lệnh `reset --hard` để đảm bảo code server giống hệt GitHub, tránh xung đột file.

```bash
# Vào thư mục dự án
cd ~/my-bentopdf (hoặc đường dẫn bạn đã lưu)

# Tải code mới về
git fetch origin

# Ép code hiện tại giống hệt nhánh custom-ui trên GitHub
git reset --hard origin/custom-ui

```

**Bước 3: Build Docker Image mới**

```bash
# Build image với tên 'bentopdf-custom'
docker build -t bentopdf-custom .

```

*Chờ đến khi chạy xong và báo thành công.*

---

### PHẦN 3: TRÊN PORTAINER (WEB)

*Mục tiêu: Cập nhật Website đang chạy sang Image vừa build.*

**Bước 1: Mở Stack cũ**

1. Đăng nhập Portainer -> Chọn **Stacks**.
2. Bấm vào Stack **bentopdf** của bạn.
3. Chọn tab **Editor**.

**Bước 2: Kiểm tra cấu hình (YAML)**
Đảm bảo file cấu hình đang dùng đúng Image custom của bạn (nếu đã sửa lần trước rồi thì không cần sửa lại):

```yaml
version: "3.8"
services:
  bentopdf:
    image: bentopdf-custom:latest   # <-- Quan trọng: Phải là tên này
    container_name: bentopdf
    ports:
      - "3000:8080"                  # Cổng Host : Cổng Container
    pull_policy: never               # <-- Quan trọng: Không tải từ mạng
    restart: unless-stopped

```

**Bước 3: Cập nhật (Update)**

1. Kéo xuống dưới cùng, bấm nút **Update the stack**.
2. 🔴 **LƯU Ý QUAN TRỌNG:** Khi popup hiện ra, hãy **GẠT TẮT** (Disable) tùy chọn **"Pull latest image"** (hoặc "Re-pull image").
* *Vì image nằm sẵn trên máy (Local build), nếu bật Pull nó sẽ tìm trên mạng và báo lỗi.*


3. Bấm **Update**.

**Bước 4: Tận hưởng**
Truy cập web của bạn (ví dụ `http://IP-Server:3000`) và kiểm tra thay đổi.

---

### Mẹo nhỏ (Troubleshooting)

* **Nếu quên mật khẩu GitHub khi push:** Hãy thiết lập SSH Key cho GitHub trên máy tính cá nhân để không phải nhập mật khẩu.
* **Nếu Build lỗi `npm ci`:** Nhớ kiểm tra trong `Dockerfile` xem đã đổi thành `RUN npm install` chưa (như chúng ta đã sửa hôm nay).
* **Nếu web không lên:** Kiểm tra lại Log trong Portainer xem nó báo lỗi gì (thường là do sai Port `80` hay `8080`).