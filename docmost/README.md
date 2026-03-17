## 如何搭建一个云文档服务

### 背景
ubuntu 22.04
云文档服务选用开源的项目：docmost

### docmost
现代团队的企级 Wiki

官网介绍：
欢迎来到 Docmost，一个开源的企业级协作维基和文档软件。它专为无缝实时协作而设计，允许多个用户同时实时编辑同一页面，而不会互相覆盖。
Docmost 是 Notion 和 Confluence 等产品的开源替代方案。无论是管理维基、知识库还是大量的项目文档，Docmost 都能提供您所需的工具，轻松创建、协作和共享知识。Docmost 完全支持自托管，可以在没有外部依赖的离线环境中运行。
Docmost 支持 Spaces 功能。您可以根据需求为不同的团队、项目或部门创建 Spaces。每个 Space 都有其独立的权限。通过可选的企业版，Docmost 可扩展以满足大型组织的需求，包括 SSO、AI、API 访问等。

### 安装

#### 系统要求:
Ubuntu 20.04+，最低 1 核 CPU、1GB RAM

#### 第一步：安装 Docker
~~~bash
sudo apt update && sudo apt install -y docker.io docker-compose
~~~

#### 第二步：创建目录并生成密钥
~~~bash
mkdir -p ~/docmost && cd ~/docmost

# 生成 APP_SECRET（必须替换，不能留默认值）
# 复制输出的字符串备用
openssl rand -hex 32
~~~

#### 第三步：创建 docker-compose.yml
~~~bash
version: "3"
services:
  docmost:
    image: docmost/docmost:latest
    container_name: docmost
    depends_on:
      - db
      - redis
    environment:
      APP_URL: "http://你的IP或域名:3000"
      APP_SECRET: "替换为openssl生成的密钥"
      DATABASE_URL: "postgresql://docmost:STRONG_DB_PASSWORD@db:5432/docmost?schema=public"
      REDIS_URL: "redis://redis:6379"
    ports:
      - "3000:3000"
    volumes:
      - docmost_data:/app/data/storage
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    container_name: docmost-db
    environment:
      POSTGRES_DB: docmost
      POSTGRES_USER: docmost
      POSTGRES_PASSWORD: STRONG_DB_PASSWORD
    volumes:
      - db_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7.2-alpine
    container_name: docmost-redis
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  docmost_data:
  db_data:
  redis_data:
~~~

#### 第四步：启动服务
~~~bash
# 启动
docker compose up -d

# 查看启动日志
docker compose logs -f docmost
~~~

#### 第五步：初始化设置
浏览器打开后，按提示：
1. 填写工作区名称
2. 填写管理员姓名、邮箱、密码
3. 点击「Create workspace」完成初始化

第六步（可选）：配置 Nginx 反向代理 + SSL
如果需要通过域名 HTTPS 访问：
~~~bash
sudo apt install -y nginx certbot python3-certbot-nginx
sudo certbot --nginx -d docs.yourdomain.com
~~~
编辑 /etc/nginx/sites-available/docmost：
~~~nginx
server {
    server_name docs.yourdomain.com;
    listen 443 ssl;

    ssl_certificate /etc/letsencrypt/live/docs.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/docs.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name docs.yourdomain.com;
    return 301 https://$host$request_uri;
}
~~~
~~~bash
sudo ln -s /etc/nginx/sites-available/docmost /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
~~~
配置好后将 docker-compose.yml 中的 APP_URL 改为 https://docs.yourdomain.com，然后重启：
~~~bash
docker compose restart docmost
~~~