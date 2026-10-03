# 服务器部署

本目录适用于服务器已安装 Nginx 的情况。Docker Compose 只启动攻略站，Nginx 使用宿主机上的现有安装。

- `docker-compose.yml`：拉取 GHCR 镜像，配置自动重启和日志轮转。
- `nginx.conf`：站点反向代理配置，包含 WebSocket 转发。

访问链路：`浏览器 → Nginx 的 80/443 端口 → 宿主机 127.0.0.1:8501 → 容器 8501`。
攻略站的 8501 端口仅绑定到宿主机本机；公网访问由 Nginx 提供。

## 启动或更新网站

服务器需要安装 Docker 和支持 `docker compose` 命令的 Compose v2 或更高版本。
先在 GitHub 手动运行镜像发布工作流，确认镜像已成功发布，再将本目录复制到服务器并进入该目录。
无需把项目源码或攻略图片复制到服务器，镜像已包含运行所需的文件。

首次发布的 GHCR 镜像默认是私有的。可以在 GitHub Packages 中将镜像设为 Public；保留私有时，先在服务器执行：

```bash
docker login ghcr.io
```

用户名填写 GitHub 账号，密码填写具有 `read:packages` 权限的 classic PAT。
如果使用其他 fork 或镜像版本，修改 `docker-compose.yml` 中的 `image` 地址。

首次部署和以后更新镜像都执行：

```bash
docker compose config --quiet
docker compose pull
docker compose up -d
```

检查网站状态：

```bash
docker compose ps
curl -fsS http://127.0.0.1:8501/_stcore/health
docker compose logs --tail 100 sephiria-guide
```

健康接口应返回 `ok`。容器会继承镜像内的健康检查。
如果修改宿主机端口，需要同时修改 Compose 的端口映射和 Nginx 的 `proxy_pass` 地址。

## 配置现有 Nginx

先将 `nginx.conf` 中的 `wiki.example.com` 替换成实际域名，并将域名解析到服务器 IP。
此文件是站点配置，应该被主配置的 `http {}` 引入；不要用它覆盖 `/etc/nginx/nginx.conf`。

如果主配置已在 `http {}` 中包含 `/etc/nginx/conf.d/*.conf`，在当前目录执行：

```bash
sudo cp nginx.conf /etc/nginx/conf.d/sephiria.conf
sudo nginx -t
sudo nginx -s reload
```

然后访问 `http://你的域名`，并确保服务器安全组或防火墙允许访问 Nginx 的 80 端口。
Nginx 的反代目标是 `http://127.0.0.1:8501`，与 Compose 的宿主机端口一致。

如果已有 HTTPS 站点，保留现有 `listen 443` 和证书配置，将本文件中的 `location /` 合并到该站点的 `server` 中，并将 `map` 配置放在 `http` 层。
域名应挂在站点根路径；此配置没有设置子路径前缀。

修改站点配置后，重新复制文件、运行 `nginx -t` 并重载 Nginx 即可，不需要重新构建网站镜像。

## 停止网站

```bash
docker compose down
```

停止网站会移除本 Compose 项目的容器和网络，宿主机上的 Nginx 保持运行。
