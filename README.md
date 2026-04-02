# 阿里云部署 Spring Boot + Vue 前后端项目学习笔记

> 记录将一个 Spring Boot + Vue3 前后端分离项目部署到阿里云 ECS 的完整过程，包括环境搭建、踩坑记录和排错经验。

## 项目背景

- **后端**：Spring Boot 3.2 + MyBatis + MySQL 8.0 + Java 21
- **前端**：Vue 3 + Vite + Element Plus
- **服务器**：阿里云 ECS / 轻量应用服务器，Ubuntu 22.04
- **Web 服务器**：Nginx（托管前端静态文件 + 反向代理后端 API）

## 目录

- [1. 整体架构理解](docs/01-architecture.md)
- [2. 本地打包](docs/02-local-build.md)
- [3. 服务器环境搭建](docs/03-server-setup.md)
- [4. 数据库初始化](docs/04-database.md)
- [5. 后端部署](docs/05-backend-deploy.md)
- [6. 前端部署](docs/06-frontend-deploy.md)
- [7. Nginx 配置详解](docs/07-nginx.md)
- [8. HTTPS 与域名配置](docs/08-https-domain.md)
- [9. 日常更新与维护](docs/09-maintenance.md)
- [10. 踩坑记录与排错](docs/10-troubleshooting.md)

## 适合谁看

第一次将项目部署到云服务器的同学，特别是：
- 本地开发用 Windows，服务器是 Linux
- 不熟悉 Nginx、systemd、MySQL 命令行操作
- 想了解前后端分离项目上线的完整流程
