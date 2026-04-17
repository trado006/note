### Re-install dependencies if change

Nội dung file bash script kiểm tra có sự thay đổi từ file quản lý dependencies

**Cách 1**: Kiểm tra bằng timestampt

```sh
#!/bin/sh
set -e

if [ ! -d "node_modules" ]; then
  echo "📦 node_modules not found → installing dependencies..."
  npm ci
elseif if [ package.json -nt node_modules ]; then
  echo "📦 package.json is newer than node_modules → reinstalling dependencies..."
  npm ci
else
  echo "✅ node_modules exists → skipping install"
fi

exec "$@"
```

**Cách 2**: Kiểm tra bằng hash content bị thay đổi

```sh
#!/bin/sh
set -e

HASH_FILE=".deps_hash"
CURRENT_HASH=$(cat package.json package-lock.json 2>/dev/null | md5sum | awk '{print $1}')

if [ ! -d "node_modules" ] || [ ! -f "$HASH_FILE" ] || [ "$CURRENT_HASH" != "$(cat $HASH_FILE)" ]; then
  echo "📦 Installing dependencies..."
  rm -rf node_modules
  npm ci
  echo "$CURRENT_HASH" > $HASH_FILE
fi

exec "$@"
```

Cách 3: Kiểm tra bằng lưu time install vào file thêm

```sh
#!/bin/sh
set -e

if [ ! -d "node_modules" ]; then
  echo "📦 node_modules not found → installing dependencies..."
  npm ci
elseif if [ package.json -nt node_modules/installed_time ]; then
  echo "📦 package.json is newer than node_modules → reinstalling dependencies..."
  npm ci
  echo "$(date '+%Y-%m-%d %H:%M:%S')" > node_modules/installed_time
else
  echo "✅ node_modules exists → skipping install"
fi

exec "$@"
```

### Setup supervisor

```sh
# Stage 1: build
FROM node:20-alpine

# Cài supervisor
RUN apk add --no-cache supervisor

# Copy config supervisor
COPY supervisord.conf /etc/supervisord.conf

# Copy app
WORKDIR /app

CMD ["supervisord", "-c", "/etc/supervisord.conf"]
```

Ý nghĩa của lệnh `apk add --no-cache`
+ tải package index về (/var/cache/apk)
+ setup rồi xóa file tải về (Bình thường file tải về sẽ được cache)

Nội dung file supervisor

```conf
[supervisord]
nodaemon=true

[program:keepalive]
command=tail -f /dev/null
autostart=true
autorestart=true

[program:node-app]
command=node index.js
autostart=true
autorestart=true
stderr_logfile=/dev/stderr
stdout_logfile=/dev/stdout

[program:worker]
command=node worker.js
autostart=true
autorestart=true
stderr_logfile=/dev/stderr
stdout_logfile=/dev/stdout
```

Note:
+ Program keep alive ở trên dùng để giữ cho container không stop sau khi thực thi xong
+ /dev/null là file mà ghi gì vào cũng mất, đọc ra thì không có gì

### Run docker with local user

Cách 1: Thiết lập trong dockerfile

```sh
ARG UID=1000
ARG GID=1000

RUN addgroup -g $GID appgroup \
 && adduser -D -u $UID -G appgroup appuser

USER appuser
```

Ví dụ

```sh
FROM php:8.2-fpm

# ARG để nhận UID/GID từ bên ngoài
ARG UID=1000
ARG GID=1000

# Cài extension cơ bản
RUN apt-get update && apt-get install -y \
    git curl zip unzip libpng-dev libonig-dev libxml2-dev \
    && docker-php-ext-install pdo pdo_mysql mbstring exif pcntl bcmath gd

# Tạo user giống host
RUN groupadd -g $GID laravel \
    && useradd -u $UID -g laravel -m laravel

# Set working dir
WORKDIR /var/www

# Copy code
COPY . .

# Set quyền (chỉ lần đầu build)
RUN chown -R laravel:laravel /var/www

# Chạy bằng user laravel
USER laravel

CMD ["php-fpm"]
```

Cách 2: Thiết lập trong docker-compose

```conf
services:
  app:
    user: "${UID}:${GID}"
```

Cách 3: export trong bash khi chạy

```sh
export UID=$(id -u)
export GID=$(id -g)
docker compose up
```

### Thiết lập file boot cho docker


