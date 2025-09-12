# Hướng dẫn cấu hình SSH Key cho GitHub Actions

## Lỗi hiện tại
```
Error: can't connect without a private SSH key or password
```

## Nguyên nhân
- SSH key không được cấu hình đúng trong GitHub Secrets
- Format SSH key không đúng
- SSH key chưa được thêm vào server DigitalOcean

## Giải pháp từng bước

### Bước 1: Tạo SSH Key Pair

Trên máy local của bạn, chạy lệnh:

```bash
# Tạo SSH key mới (thay your-email@example.com bằng email của bạn)
ssh-keygen -t rsa -b 4096 -C "your-email@example.com" -f ~/.ssh/digitalocean_deploy

# Hoặc sử dụng ed25519 (khuyến nghị)
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/digitalocean_deploy
```

### Bước 2: Copy Public Key lên DigitalOcean Server

```bash
# Copy public key lên server
ssh-copy-id -i ~/.ssh/digitalocean_deploy.pub root@YOUR_SERVER_IP

# Hoặc copy thủ công
cat ~/.ssh/digitalocean_deploy.pub
# Copy nội dung và thêm vào file ~/.ssh/authorized_keys trên server
```

### Bước 3: Test SSH Connection

```bash
# Test kết nối SSH
ssh -i ~/.ssh/digitalocean_deploy root@YOUR_SERVER_IP

# Nếu thành công, bạn sẽ thấy terminal của server
```

### Bước 4: Cấu hình GitHub Secrets

1. **Vào GitHub Repository** → Settings → Secrets and variables → Actions

2. **Thêm các secrets sau:**

#### DO_HOST
- Name: `DO_HOST`
- Value: `123.456.789.0` (IP của server DigitalOcean)

#### DO_USERNAME  
- Name: `DO_USERNAME`
- Value: `root` (hoặc username khác)

#### DO_SSH_KEY
- Name: `DO_SSH_KEY`
- Value: **Toàn bộ nội dung file private key**

```bash
# Lấy nội dung private key
cat ~/.ssh/digitalocean_deploy
```

**Copy TOÀN BỘ nội dung, bao gồm:**
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEA1234567890abcdef...
-----END OPENSSH PRIVATE KEY-----
```

#### DO_PORT (tùy chọn)
- Name: `DO_PORT`
- Value: `22` (hoặc port SSH khác)

### Bước 5: Cấu hình Server DigitalOcean

```bash
# SSH vào server
ssh root@YOUR_SERVER_IP

# Tạo thư mục cho project
mkdir -p /opt/my-laravel
chmod 755 /opt/my-laravel

# Cài đặt Docker (nếu chưa có)
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Cài đặt Docker Compose
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Tạo file .env.production
nano /opt/my-laravel/.env.production
```

### Bước 6: Test GitHub Actions

1. **Commit và push code:**
```bash
git add .
git commit -m "Fix SSH configuration"
git push
```

2. **Tạo tag để trigger deployment:**
```bash
git tag v1.0.6
git push origin v1.0.6
```

3. **Kiểm tra GitHub Actions:**
   - Vào tab "Actions" trong GitHub
   - Xem log của workflow
   - Nếu SSH test thành công, deployment sẽ tiếp tục

## Troubleshooting

### Lỗi "Permission denied (publickey)"

```bash
# Kiểm tra quyền file SSH key
chmod 600 ~/.ssh/digitalocean_deploy
chmod 644 ~/.ssh/digitalocean_deploy.pub

# Kiểm tra SSH agent
ssh-add ~/.ssh/digitalocean_deploy
```

### Lỗi "Host key verification failed"

```bash
# Thêm server vào known_hosts
ssh-keyscan -H YOUR_SERVER_IP >> ~/.ssh/known_hosts
```

### Lỗi "No such file or directory"

```bash
# Tạo thư mục .ssh nếu chưa có
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

### Kiểm tra SSH key format

SSH key phải có format đúng:

**RSA Key:**
```
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

**Ed25519 Key:**
```
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

## Lưu ý quan trọng

1. **KHÔNG** commit private key vào git
2. **KHÔNG** chia sẻ private key với ai
3. **LUÔN** sử dụng SSH key riêng cho deployment
4. **KIỂM TRA** quyền file SSH key (600 cho private, 644 cho public)

## Test cuối cùng

Sau khi cấu hình xong, test bằng cách:

```bash
# Test SSH từ máy local
ssh -i ~/.ssh/digitalocean_deploy root@YOUR_SERVER_IP "echo 'SSH working!'"

# Nếu thành công, tạo tag mới
git tag v1.0.7
git push origin v1.0.7
```
