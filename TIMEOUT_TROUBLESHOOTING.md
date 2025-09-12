# Khắc phục lỗi SSH Timeout

## Lỗi hiện tại
```
drone-scp error: dial tcp ***:22: i/o timeout
```

## Nguyên nhân có thể

### 1. **Server DigitalOcean không hoạt động**
- Server bị tắt hoặc restart
- IP address thay đổi
- Firewall chặn kết nối SSH

### 2. **Cấu hình SSH không đúng**
- SSH key không được thêm vào server
- Username không đúng
- Port SSH không đúng

### 3. **Vấn đề mạng**
- Kết nối internet không ổn định
- DigitalOcean có vấn đề về mạng
- GitHub Actions runner có vấn đề

## Cách kiểm tra và khắc phục

### Bước 1: Kiểm tra server DigitalOcean

1. **Đăng nhập vào DigitalOcean Dashboard**
2. **Kiểm tra status server:**
   - Server có đang chạy không?
   - IP address có thay đổi không?
   - Có thông báo lỗi nào không?

3. **Kiểm tra từ máy local:**
   ```bash
   # Test ping
   ping YOUR_SERVER_IP
   
   # Test SSH connection
   ssh -i ~/.ssh/your-key root@YOUR_SERVER_IP
   ```

### Bước 2: Kiểm tra SSH configuration

1. **Kiểm tra SSH key trên server:**
   ```bash
   # SSH vào server
   ssh root@YOUR_SERVER_IP
   
   # Kiểm tra authorized_keys
   cat ~/.ssh/authorized_keys
   
   # Kiểm tra quyền file
   ls -la ~/.ssh/
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

2. **Kiểm tra SSH service:**
   ```bash
   # Kiểm tra SSH service
   systemctl status ssh
   
   # Restart SSH nếu cần
   systemctl restart ssh
   ```

### Bước 3: Kiểm tra firewall

```bash
# Kiểm tra firewall rules
ufw status

# Mở port 22 nếu cần
ufw allow 22
ufw allow ssh

# Hoặc tắt firewall tạm thời để test
ufw disable
```

### Bước 4: Cập nhật GitHub Secrets

1. **Kiểm tra lại các secrets:**
   - `DO_HOST`: IP server chính xác
   - `DO_USERNAME`: Username đúng (thường là `root`)
   - `DO_SSH_KEY`: Private key đầy đủ
   - `DO_PORT`: Port SSH (thường là `22`)

2. **Tạo SSH key mới nếu cần:**
   ```bash
   # Tạo key mới
   ssh-keygen -t ed25519 -C "deploy@github" -f ~/.ssh/digitalocean_deploy_new
   
   # Copy public key lên server
   ssh-copy-id -i ~/.ssh/digitalocean_deploy_new.pub root@YOUR_SERVER_IP
   
   # Test connection
   ssh -i ~/.ssh/digitalocean_deploy_new root@YOUR_SERVER_IP
   ```

### Bước 5: Test từ GitHub Actions

1. **Tạo workflow test đơn giản:**
   ```yaml
   - name: Test SSH Connection
     uses: appleboy/ssh-action@v1.0.3
     with:
       host: ${{ secrets.DO_HOST }}
       username: ${{ secrets.DO_USERNAME }}
       key: ${{ secrets.DO_SSH_KEY }}
       port: ${{ secrets.DO_PORT || 22 }}
       timeout: 60s
       script: |
         echo "SSH connection successful!"
         whoami
         pwd
   ```

2. **Commit và push để test:**
   ```bash
   git add .
   git commit -m "Test SSH connection"
   git push
   ```

## Giải pháp tạm thời

Nếu vẫn gặp timeout, có thể thử:

### 1. **Sử dụng password thay vì SSH key:**
```yaml
- name: Deploy with Password
  uses: appleboy/ssh-action@v1.0.3
  with:
    host: ${{ secrets.DO_HOST }}
    username: ${{ secrets.DO_USERNAME }}
    password: ${{ secrets.DO_PASSWORD }}
    port: ${{ secrets.DO_PORT || 22 }}
    timeout: 60s
    command_timeout: 30m
```

### 2. **Sử dụng DigitalOcean API:**
Thay vì SSH, có thể sử dụng DigitalOcean API để deploy.

### 3. **Sử dụng webhook:**
Tạo webhook trên server để GitHub Actions gọi API thay vì SSH.

## Kiểm tra logs chi tiết

1. **Vào GitHub Actions** → Click vào workflow run
2. **Xem logs chi tiết** của từng step
3. **Kiểm tra error messages** cụ thể

## Liên hệ hỗ trợ

Nếu vẫn không giải quyết được:
1. **Kiểm tra DigitalOcean Status Page**
2. **Liên hệ DigitalOcean Support**
3. **Kiểm tra GitHub Actions Status**

## Lưu ý quan trọng

- **Luôn backup** trước khi thay đổi cấu hình
- **Test trên môi trường dev** trước khi deploy production
- **Giữ SSH key an toàn** và không chia sẻ
