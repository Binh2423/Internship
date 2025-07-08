# 🐳 HƯỚNG DẪN CÀI DOCKER CHI TIẾT

## 🎯 MỤC TIÊU: Cài Docker và hiểu cách sử dụng cơ bản

---

## 📋 BƯỚC 1: XÁC ĐỊNH HỆ ĐIỀU HÀNH

Trước tiên, cần biết bạn đang dùng hệ điều hành gì:

### Kiểm tra hệ điều hành:
```bash
# Trên Linux
cat /etc/os-release

# Trên macOS
sw_vers

# Trên Windows
systeminfo
```

---

## 🐧 CÁCH 1: CÀI DOCKER TRÊN LINUX (Ubuntu/Debian)

### Bước 1.1: Cập nhật hệ thống
```bash
# Cập nhật package list
sudo apt update

# Cập nhật các packages
sudo apt upgrade -y
```

### Bước 1.2: Cài đặt các dependencies cần thiết
```bash
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

**Giải thích:**
- `apt-transport-https`: Cho phép apt download qua HTTPS
- `ca-certificates`: Certificates để verify SSL
- `curl`: Tool để download
- `gnupg`: Để verify signatures
- `lsb-release`: Để detect Ubuntu version

### Bước 1.3: Thêm Docker's official GPG key
```bash
# Tạo thư mục cho keyrings
sudo mkdir -p /etc/apt/keyrings

# Download và add Docker's GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

### Bước 1.4: Thêm Docker repository
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Bước 1.5: Cài Docker Engine
```bash
# Cập nhật package list với repo mới
sudo apt update

# Cài Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Bước 1.6: Khởi động Docker service
```bash
# Start Docker service
sudo systemctl start docker

# Enable Docker để tự start khi boot
sudo systemctl enable docker

# Kiểm tra status
sudo systemctl status docker
```

### Bước 1.7: Test Docker installation
```bash
# Chạy hello-world container
sudo docker run hello-world
```

**Nếu thấy message "Hello from Docker!" thì đã cài thành công!**

---

## 🍎 CÁCH 2: CÀI DOCKER TRÊN macOS

### Bước 2.1: Download Docker Desktop
1. Vào https://www.docker.com/products/docker-desktop/
2. Click "Download for Mac"
3. Chọn chip phù hợp:
   - **Intel chip**: Docker Desktop for Mac with Intel chip
   - **Apple Silicon (M1/M2)**: Docker Desktop for Mac with Apple chip

### Bước 2.2: Cài đặt
1. Mở file `.dmg` đã download
2. Kéo Docker icon vào Applications folder
3. Mở Docker từ Applications
4. Follow setup wizard

### Bước 2.3: Verify installation
```bash
# Mở Terminal và chạy
docker --version
docker run hello-world
```

---

## 🪟 CÁCH 3: CÀI DOCKER TRÊN WINDOWS

### Bước 3.1: Kiểm tra requirements
- Windows 10 64-bit: Pro, Enterprise, hoặc Education (Build 16299 hoặc mới hơn)
- Hoặc Windows 11 64-bit
- WSL 2 feature enabled

### Bước 3.2: Enable WSL 2
```powershell
# Mở PowerShell as Administrator
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# Enable Virtual Machine Platform
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Restart máy tính
```

### Bước 3.3: Download và cài WSL 2 Linux kernel
1. Download từ: https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
2. Chạy file để cài

### Bước 3.4: Set WSL 2 as default
```powershell
wsl --set-default-version 2
```

### Bước 3.5: Download Docker Desktop
1. Vào https://www.docker.com/products/docker-desktop/
2. Click "Download for Windows"
3. Chạy installer
4. Follow setup instructions

### Bước 3.6: Test installation
```cmd
# Mở Command Prompt hoặc PowerShell
docker --version
docker run hello-world
```

---

## 🔧 BƯỚC 2: CẤU HÌNH DOCKER (QUAN TRỌNG!)

### Thêm user vào docker group (Linux only)
```bash
# Thêm user hiện tại vào docker group
sudo usermod -aG docker $USER

# Logout và login lại, hoặc chạy:
newgrp docker

# Test không cần sudo
docker run hello-world
```

**Tại sao cần làm này?**
- Mặc định chỉ root mới chạy được Docker
- Thêm vào group để không cần `sudo` mỗi lần

---

## 🧪 BƯỚC 3: TEST DOCKER VỚI CÁC LỆNH CƠ BẢN

### Test 1: Kiểm tra version
```bash
docker --version
# Output: Docker version 24.0.7, build afdd53b

docker info
# Hiển thị thông tin chi tiết về Docker
```

### Test 2: Chạy container đầu tiên
```bash
# Chạy nginx web server
docker run -d -p 8080:80 --name my-nginx nginx

# Giải thích:
# -d: chạy background (detached)
# -p 8080:80: map port 8080 của máy -> port 80 của container
# --name my-nginx: đặt tên container
# nginx: tên image
```

### Test 3: Kiểm tra container đang chạy
```bash
# Xem containers đang chạy
docker ps

# Xem tất cả containers (cả đã stop)
docker ps -a
```

### Test 4: Test web server
```bash
# Mở browser và vào: http://localhost:8080
# Hoặc dùng curl:
curl http://localhost:8080
```

**Nếu thấy trang nginx thì thành công!**

### Test 5: Dọn dẹp
```bash
# Stop container
docker stop my-nginx

# Remove container
docker rm my-nginx

# Remove image (optional)
docker rmi nginx
```

---

## 🔍 BƯỚC 4: HIỂU CÁC LỆNH DOCKER CƠ BẢN

### Quản lý Images
```bash
# Xem images có sẵn
docker images

# Download image (không chạy)
docker pull ubuntu

# Xóa image
docker rmi ubuntu
```

### Quản lý Containers
```bash
# Chạy container interactive
docker run -it ubuntu bash

# Chạy container background
docker run -d nginx

# Stop container
docker stop <container-id>

# Start lại container đã stop
docker start <container-id>

# Xem logs của container
docker logs <container-id>

# Vào bên trong container đang chạy
docker exec -it <container-id> bash
```

### Quản lý Networks
```bash
# Xem networks
docker network ls

# Tạo network mới
docker network create my-network

# Chạy container trong network cụ thể
docker run -d --network my-network --name web nginx
```

---

## 🚨 TROUBLESHOOTING - XỬ LÝ LỖI THƯỜNG GẶP

### Lỗi 1: "permission denied" (Linux)
```bash
# Lỗi: Got permission denied while trying to connect to the Docker daemon socket

# Fix: Thêm user vào docker group
sudo usermod -aG docker $USER
newgrp docker
```

### Lỗi 2: "Docker daemon is not running"
```bash
# Linux: Start Docker service
sudo systemctl start docker

# macOS/Windows: Mở Docker Desktop app
```

### Lỗi 3: "Port already in use"
```bash
# Lỗi: bind: address already in use

# Fix: Dùng port khác
docker run -p 8081:80 nginx  # thay vì 8080:80

# Hoặc tìm và kill process đang dùng port
sudo lsof -i :8080
sudo kill -9 <PID>
```

### Lỗi 4: "No space left on device"
```bash
# Docker chiếm nhiều disk space

# Dọn dẹp containers không dùng
docker container prune

# Dọn dẹp images không dùng
docker image prune

# Dọn dẹp tất cả (cẩn thận!)
docker system prune -a
```

---

## 🎯 BƯỚC 5: PROJECT ĐẦU TIÊN - WEB APP ĐỚN GIẢN

### Tạo file HTML đơn giản
```bash
# Tạo thư mục project
mkdir my-first-docker-app
cd my-first-docker-app

# Tạo file index.html
cat > index.html << EOF
<!DOCTYPE html>
<html>
<head>
    <title>My First Docker App</title>
</head>
<body>
    <h1>Hello from Docker!</h1>
    <p>This is my first containerized web app.</p>
</body>
</html>
EOF
```

### Tạo Dockerfile
```bash
cat > Dockerfile << EOF
# Sử dụng nginx làm base image
FROM nginx:alpine

# Copy file HTML vào container
COPY index.html /usr/share/nginx/html/

# Expose port 80
EXPOSE 80
EOF
```

### Build và chạy
```bash
# Build image
docker build -t my-web-app .

# Chạy container
docker run -d -p 8080:80 --name my-app my-web-app

# Test
curl http://localhost:8080
```

**Chúc mừng! Bạn đã tạo và chạy Docker app đầu tiên!**

---

## 📚 BƯỚC 6: NEXT STEPS

### Học tiếp:
1. **Docker Compose**: Chạy nhiều containers cùng lúc
2. **Docker Networks**: Containers communicate với nhau
3. **Docker Volumes**: Persistent data storage
4. **Multi-stage builds**: Optimize image size

### Chuẩn bị cho Container Networking project:
```bash
# Tạo network cho project
docker network create container-network

# Chạy database container
docker run -d --network container-network --name db postgres:13

# Chạy web app container
docker run -d --network container-network --name web -p 8080:80 nginx

# Test connectivity
docker exec web ping db
```

---

## ✅ CHECKLIST - ĐẢM BẢO ĐÃ CÀI THÀNH CÔNG

- [ ] Docker version hiển thị đúng
- [ ] `docker run hello-world` chạy thành công
- [ ] Có thể chạy nginx container và access qua browser
- [ ] Không cần `sudo` để chạy docker (Linux)
- [ ] Hiểu các lệnh cơ bản: run, ps, stop, rm
- [ ] Đã tạo và chạy được Docker app đầu tiên

---

## 🎯 TÓM TẮT

**Bạn đã học được:**
1. Cài Docker trên hệ điều hành của mình
2. Hiểu các lệnh Docker cơ bản
3. Chạy containers đầu tiên
4. Tạo Dockerfile và build image
5. Troubleshoot các lỗi thường gặp

**Bước tiếp theo:** Bắt đầu với Container Networking project!

**Có vấn đề gì không? Hãy chạy từng lệnh và báo lỗi cụ thể để tôi giúp fix!**
