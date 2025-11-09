# Docker 部署指南

## 快速开始

### 1. 构建镜像

```bash
cd ruoyi-fastapi-backend
docker build -t ruoyi-fastapi-backend:latest .
```

### 2. 准备配置文件

复制 `.env.prod.example` 为 `.env.prod` 并修改配置：

```bash
cp .env.prod.example .env.prod
# 编辑 .env.prod 修改数据库、Redis 等配置
```

### 3. 运行容器

```bash
docker run -d \
  --name ruoyi-fastapi \
  -p 9099:9099 \
  --env-file .env.prod \
  ruoyi-fastapi-backend:latest
```

### 4. 查看日志

```bash
docker logs -f ruoyi-fastapi
```

### 5. 健康检查

```bash
curl http://localhost:9099/dev-api/health
```

## 推送到 Docker Hub

### 1. 登录 Docker Hub

```bash
docker login
```

### 2. 标记镜像

```bash
docker tag ruoyi-fastapi-backend:latest your-dockerhub-username/ruoyi-fastapi-backend:latest
docker tag ruoyi-fastapi-backend:latest your-dockerhub-username/ruoyi-fastapi-backend:1.0.0
```

### 3. 推送镜像

```bash
docker push your-dockerhub-username/ruoyi-fastapi-backend:latest
docker push your-dockerhub-username/ruoyi-fastapi-backend:1.0.0
```

## 环境变量说明

| 变量名         | 说明                          | 默认值        |
| -------------- | ----------------------------- | ------------- |
| APP_ENV        | 运行环境                      | prod          |
| APP_HOST       | 监听地址                      | 0.0.0.0       |
| APP_PORT       | 监听端口                      | 9099          |
| DB_TYPE        | 数据库类型 (mysql/postgresql) | mysql         |
| DB_HOST        | 数据库地址                    | -             |
| DB_PORT        | 数据库端口                    | 3306          |
| DB_USERNAME    | 数据库用户名                  | -             |
| DB_PASSWORD    | 数据库密码                    | -             |
| DB_DATABASE    | 数据库名称                    | ruoyi-fastapi |
| REDIS_HOST     | Redis 地址                    | -             |
| REDIS_PORT     | Redis 端口                    | 6379          |
| REDIS_PASSWORD | Redis 密码                    | -             |

## 注意事项

1. **数据库和 Redis**: 此镜像只包含 FastAPI 应用，需要单独部署 MySQL/PostgreSQL 和 Redis
2. **持久化数据**: 建议将 `/app/vf_admin/upload_path` 目录挂载到宿主机
3. **日志**: 日志位于 `/app/logs` 目录
4. **安全性**: 生产环境请修改 JWT_SECRET_KEY 和数据库密码

## Docker Compose 示例（带外部数据库和 Redis）

```yaml
version: '3.8'

services:
  ruoyi-fastapi:
    image: your-dockerhub-username/ruoyi-fastapi-backend:latest
    container_name: ruoyi-fastapi
    restart: unless-stopped
    ports:
      - '9099:9099'
    environment:
      - APP_ENV=prod
      - DB_HOST=your-db-host
      - DB_PORT=3306
      - DB_USERNAME=your-username
      - DB_PASSWORD=your-password
      - REDIS_HOST=your-redis-host
      - REDIS_PORT=6379
      - REDIS_PASSWORD=your-redis-password
    volumes:
      - ./uploads:/app/vf_admin/upload_path
      - ./logs:/app/logs
    healthcheck:
      test:
        [
          'CMD',
          'python',
          '-c',
          "import urllib.request; urllib.request.urlopen('http://localhost:9099/dev-api/health').read()",
        ]
      interval: 30s
      timeout: 10s
      retries: 3
```
