# 7. Nginx 配置详解

## 基础配置（HTTP）

```bash
nano /etc/nginx/sites-available/ai-hardware
```

```nginx
server {
    listen 80;
    server_name yamiya-ai.com;     # 换成你的域名或服务器 IP

    root /var/www/ai-hardware;      # 前端文件目录
    index index.html;

    # Vue Router history 模式支持
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 后端 API 反向代理
    location /app-api/ {
        proxy_pass http://127.0.0.1:48080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # 用户上传文件
    location /uploads/ {
        alias /opt/ai-hardware/uploads/;
    }

    # 静态图片（带浏览器缓存）
    location /images/ {
        alias /var/www/ai-hardware/images/;
        expires 30d;
        add_header Cache-Control "public, immutable";
        try_files $uri =404;
    }
}
```

## 逐行解读

### `try_files $uri $uri/ /index.html`

- 先找 `$uri` 对应的真实文件（比如 `/assets/index.js`）
- 找不到就试试当目录找（`$uri/`）
- 还找不到就返回 `/index.html`，让 Vue Router 处理前端路由
- **这是 SPA 应用部署的关键配置**，没有它刷新页面会 404

### `proxy_pass http://127.0.0.1:48080`

- 把 `/app-api/` 开头的请求转发给本地 48080 端口的 Spring Boot
- `proxy_set_header` 把客户端真实 IP 传给后端，否则后端只能看到 `127.0.0.1`

### `alias` vs `root`

- `alias /opt/ai-hardware/uploads/`：URL `/uploads/a.jpg` 对应文件 `/opt/ai-hardware/uploads/a.jpg`
- 如果用 `root`：URL `/uploads/a.jpg` 对应 `/opt/ai-hardware/uploads/uploads/a.jpg`（路径会拼接）

> 当 location 路径和文件系统路径不同时，用 `alias`。

## 启用配置

```bash
# 创建符号链接启用站点配置
ln -s /etc/nginx/sites-available/ai-hardware /etc/nginx/sites-enabled/

# 删除默认站点（避免冲突）
rm -f /etc/nginx/sites-enabled/default

# 检查语法
nginx -t

# 重载配置（不中断服务）
systemctl reload nginx
```

## 开启 Gzip 压缩

编辑 `/etc/nginx/nginx.conf`，找到 gzip 相关配置：

```nginx
gzip on;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_types text/plain text/css application/json application/javascript
           text/xml application/xml application/xml+rss text/javascript
           image/svg+xml;
```

效果：JS/CSS 文件传输大小压缩到原来的 1/3 左右，页面加载速度明显提升。

```bash
nginx -t && systemctl reload nginx
```

## Nginx 配置文件结构

```
/etc/nginx/
├── nginx.conf                  # 全局配置（gzip、worker 等）
├── sites-available/            # 所有站点配置（存放处）
│   └── ai-hardware             # 我们的配置
└── sites-enabled/              # 已启用的站点（符号链接）
    └── ai-hardware → ../sites-available/ai-hardware
```

`sites-available` 存放配置，`sites-enabled` 通过软链接启用。这样可以方便地启用/禁用站点而不删除配置文件。
