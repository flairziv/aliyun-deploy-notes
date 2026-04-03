# 8. HTTPS 与域名配置

## 前置条件

1. **拥有域名**：如 `hello-ai.com`
2. **域名备案**：大陆服务器必须完成 ICP 备案
3. **DNS 解析**：在域名管理后台添加 A 记录，指向服务器 IP

## 关于 ICP 备案

中国大陆服务器必须备案，否则阿里云会拦截 80/443 端口，用户看到的是「该网站未备案」提示页。

**备案流程**：
1. 阿里云控制台 → 「备案」入口
2. 填写主体信息（公司/个人信息）
3. 填写网站信息（域名、服务器 IP）
4. 提交审核，等待工信部审批（约 7-20 个工作日）
5. 通过后，在网站底部展示备案号

> 建议在开发阶段就开始备案申请，因为等待时间较长。

## 申请 SSL 证书（Let's Encrypt 免费证书）

```bash
# 安装 certbot
apt install certbot python3-certbot-nginx -y

# 自动申请证书并配置 Nginx
certbot --nginx -d hello-ai.com
```

certbot 会自动：
1. 验证你拥有该域名
2. 申请 SSL 证书
3. 修改 Nginx 配置，添加 HTTPS 支持

## 配置自动续期

Let's Encrypt 证书有效期只有 **90 天**，需要自动续期：

```bash
systemctl enable certbot.timer
```

这会创建一个定时任务，定期检查并自动续期即将过期的证书。

## 完整的 HTTPS Nginx 配置

certbot 自动配置后，建议检查并整理为如下格式：

```nginx
# HTTP → HTTPS 跳转
server {
    listen 80;
    listen [::]:80;
    server_name hello-ai.com;
    return 301 https://$host$request_uri;
}

# HTTPS 主配置
server {
    listen 443 ssl;
    listen [::]:443 ssl ipv6only=on;
    server_name hello-ai.com;

    ssl_certificate /etc/letsencrypt/live/hello-ai.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/hello-ai.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    root /var/www/ai-hardware;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /app-api/ {
        proxy_pass http://127.0.0.1:48080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /uploads/ {
        alias /opt/ai-hardware/uploads/;
    }

    location /images/ {
        alias /var/www/ai-hardware/images/;
        expires 30d;
        add_header Cache-Control "public, immutable";
        try_files $uri =404;
    }
}
```

## 阿里云安全组端口放行

在阿里云控制台 → 安全组 → 入方向，添加规则：

| 端口 | 用途 | 是否必须 |
|------|------|---------|
| 80 | HTTP（会被重定向到 HTTPS） | 是 |
| 443 | HTTPS | 是 |
| 22 | SSH 登录管理 | 是 |
| 48080 | Spring Boot | **不需要开放** |

> 48080 只在服务器内部由 Nginx 转发访问，不对外暴露。
