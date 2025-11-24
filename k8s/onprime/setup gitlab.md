# curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
# apt install gitlab-ee=15.7.7-ee.0

# sudo certbot certonly --standalone -d gitlab.devopseduvn.live --preferred-challenges http --agree-tos -m $EMAIL --keep-until-expiring

nano /etc/gitlab/gitlab.rb

sửa external_url = https://gitlab.ducanh.io.vn
letsencrypt['enable'] = true
letsencrypt['contact_emails'] = ['youremail@domain.com']
sudo gitlab-ctl reconfigure

check status sudo gitlab-ctl status

## Tạo Cloudflare Origin Certificate

### Trên Cloudflare:
SSL/TLS → Origin Server → Create Certificate
Cấu hình:
Private key type: RSA
Hostnames: gitlab.ducanh.io.vn
Validity: 15 years
Cloudflare sẽ trả bạn 2 phần:
Origin Certificate
Private Key
Giữ nguyên format, không chỉnh sửa.

### Add cert phân quyền
sudo mkdir -p /etc/gitlab/ssl
sudo chmod 700 /etc/gitlab/ssl
sudo nano /etc/gitlab/ssl/gitlab.ducanh.io.vn.crt
sudo nano /etc/gitlab/ssl/gitlab.ducanh.io.vn.key
sudo chmod 600 /etc/gitlab/ssl/gitlab.ducanh.io.vn.key
sudo chmod 644 /etc/gitlab/ssl/gitlab.ducanh.io.vn.crt

### sửa cấu hình
sudo nano /etc/gitlab/gitlab.rb
external_url "https://gitlab.ducanh.io.vn"
letsencrypt['enable'] = false
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.ducanh.io.vn.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.ducanh.io.vn.key"

### run
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
