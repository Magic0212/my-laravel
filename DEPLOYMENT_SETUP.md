# Hướng dẫn cấu hình Deployment lên DigitalOcean

## Vấn đề đã được sửa

1. ✅ **Lỗi syntax trong workflow**: Đã sửa lỗi ở dòng 54-55 trong file `.github/workflows/release.yml`
2. ✅ **Thêm bước deployment**: Đã thêm các bước để deploy lên DigitalOcean
3. ✅ **Tạo file docker-compose.prod.yml**: File cấu hình cho production

## Cấu hình Secrets trong GitHub

Bạn cần thêm các secrets sau vào GitHub repository:

### 1. Truy cập GitHub Secrets
- Vào repository trên GitHub
- Settings → Secrets and variables → Actions
- Click "New repository secret"

### 2. Thêm các secrets sau:

| Secret Name | Mô tả | Ví dụ |
|-------------|-------|-------|
| `DO_HOST` | IP address của DigitalOcean server | `123.456.789.0` |
| `DO_USERNAME` | Username để SSH vào server | `root` hoặc `ubuntu` |
| `DO_SSH_KEY` | Private SSH key để kết nối server | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `DO_PORT` | SSH port (tùy chọn, mặc định 22) | `22` |

### 3. Cấu hình trên DigitalOcean Server

#### Tạo thư mục và cấu hình:
```bash
# Tạo thư mục cho project
sudo mkdir -p /opt/my-laravel
sudo chown $USER:$USER /opt/my-laravel

# Tạo file .env.production
sudo nano /opt/my-laravel/.env.production
```

#### Nội dung file .env.production:
```env
APP_NAME="My Laravel"
APP_ENV=production
APP_KEY=base64:your-app-key-here
APP_DEBUG=false
APP_URL=https://yourdomain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=your-db-password

# Thêm các cấu hình khác cần thiết
```

#### Cài đặt Docker và Docker Compose:
```bash
# Cài đặt Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Cài đặt Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Thêm user vào docker group
sudo usermod -aG docker $USER
```

## Cách sử dụng

1. **Tạo tag mới**:
   ```bash
   git tag v1.0.5
   git push origin v1.0.5
   ```

2. **GitHub Actions sẽ tự động**:
   - Build Docker images
   - Push lên GitHub Container Registry
   - Deploy lên DigitalOcean server

## Kiểm tra deployment

Sau khi deployment thành công, kiểm tra:

```bash
# SSH vào server
ssh user@your-server-ip

# Kiểm tra containers đang chạy
docker ps

# Xem logs
docker-compose -f /opt/my-laravel/docker-compose.prod.yml logs

# Kiểm tra website
curl http://localhost
```

## Troubleshooting

### Nếu deployment thất bại:

1. **Kiểm tra logs GitHub Actions**:
   - Vào tab "Actions" trong GitHub
   - Click vào workflow run
   - Xem chi tiết lỗi

2. **Kiểm tra server**:
   ```bash
   # Kiểm tra Docker
   docker --version
   docker-compose --version
   
   # Kiểm tra thư mục
   ls -la /opt/my-laravel/
   
   # Kiểm tra quyền
   ls -la /opt/my-laravel/docker-compose.prod.yml
   ```

3. **Kiểm tra kết nối SSH**:
   ```bash
   # Test SSH connection
   ssh -i your-private-key user@server-ip
   ```

### Lỗi thường gặp:

- **Permission denied**: Kiểm tra quyền SSH key và user
- **Docker not found**: Cài đặt Docker trên server
- **File not found**: Kiểm tra đường dẫn file docker-compose.prod.yml
- **Port already in use**: Dừng service đang sử dụng port 80
