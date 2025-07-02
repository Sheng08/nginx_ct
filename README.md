# Nginx Docker 容器部署

## 快速啟動

### 1. 建立並啟動 nginx 容器
```bash
docker-compose up -d
```

### 2. 驗證容器運行狀態
```bash
docker ps
curl http://localhost
```

## 目錄結構

```
nginx/
├── config/
│   └── default.conf          # nginx 配置文件
├── nginx_html/
│   └── index.html            # 網站內容
├── docker-compose.yaml       # Docker Compose 配置
└── README.md
```

## 網路配置

此設置會創建一個名為 `nginx_network` 的 Docker 網路，其他容器可以加入此網路來與 nginx 通信。

### 讓其他容器加入 nginx 網路

#### 直接使用 docker run 命令連接網路

```bash
# 先查看網路名稱
docker network ls | grep nginx

# 將現有容器連接到 nginx 網路
docker network connect nginx_nginx_network your-container-name
```

## nginx 反向代理配置

在 `config/default.conf` 中配置反向代理到其他容器：

```nginx
upstream backend {
    server your-app-container:port;
}

server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 常用命令

```bash
# 查看容器日志
docker-compose logs nginx

# 重新加載 nginx 配置
docker-compose exec nginx nginx -s reload

# 停止容器
docker-compose down

# 查看網路詳情
docker network inspect nginx_nginx_network

# 查看哪些容器連接到此網路
docker network inspect nginx_nginx_network --format='{{range .Containers}}{{.Name}} {{end}}'
```

## 注意事項

1. **網路名稱**：實際的網路名稱會是 `nginx_nginx_network` (前綴是資料夾名稱)
2. **容器間通信**：同一網路內的容器可以通過容器名稱互相訪問
3. **配置修改**：修改 nginx 配置後需要重新加載或重啟容器
4. **權限問題**：確保 `/home/ubuntu/nginx/` 路徑存在且有適當權限

