# Сервер - https://youtu.be/XDWWxA6g1eA
Подключаемся к серверу

```
ssh root@ip
```

Обновляем систему

```
apt update -y
```

Меняем пароль root

```
passwd
```

Ставим зависимости для сборки nginx

```
apt install dpkg-dev build-essential gnupg2 git gcc cmake libpcre3 libpcre3-dev zlib1g zlib1g-dev openssl libssl-dev curl unzip -y
```

Добавляем GPG ключ nginx

```
curl -L https://nginx.org/keys/nginx_signing.key | apt-key add -
```

Добавляем репозитории nginx

```
nano /etc/apt/sources.list.d/nginx.list
```

```
deb http://nginx.org/packages/ubuntu/ focal nginx
deb-src http://nginx.org/packages/ubuntu/ focal nginx
```

Обновляем репозитории

```
apt update -y
```

Скачиваем исходники nginx

```
cd /usr/local/src
apt source nginx
```

Ставим зависимости для сборки

```
apt build-dep nginx -y
```

Скачиваем Brotli

```
git clone --recursive https://github.com/google/ngx_brotli.git
```

Обновляем правила сборки

```
cd /usr/local/src/nginx-*/
nano debian/rules
```

Найтём блоки

- `config.env.nginx`
- `config.env.nginx_debug`

Добавим новый ключ в каждом `./configure`

```
--add-module=/usr/local/src/ngx_brotli
```

Компилируем и собираем nginx

```
dpkg-buildpackage -b -uc -us
```

Проверяем deb-файлы

```
ls /usr/local/src/*.deb
```

Устанавливаем nginx из deb-файлов

```
dpkg -i /usr/local/src/*.deb
```

Настраиваем nginx

```
nano /etc/nginx/nginx.conf
```

```conf
user www-data;
worker_processes auto;
pid /var/run/nginx.pid;

events {
    worker_connections 768;
}

include /etc/nginx/sites-enabled/*.stream;

http {

    # Basic

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    types_hash_max_size 2048;
    server_tokens off;
    ignore_invalid_headers on;

    # Decrease default timeouts to drop slow clients

    keepalive_timeout 40s;
    send_timeout 20s;
    client_header_timeout 20s;
    client_body_timeout 20s;
    reset_timedout_connection on;

    # Hash sizes

    server_names_hash_bucket_size 64;

    # Mime types

    default_type  application/octet-stream;
    include /etc/nginx/mime.types;

    # Logs

    log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $bytes_sent "$http_referer" "$http_user_agent" "$gzip_ratio"';
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;

    # Limits

    limit_req_zone  $binary_remote_addr  zone=dos_attack:20m   rate=30r/m;

    # Gzip

    gzip on;
    gzip_disable "msie6";
    gzip_vary off;
    gzip_proxied any;
    gzip_comp_level 5;
    gzip_min_length 1000;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types
        application/atom+xml
        application/javascript
        application/json
        application/ld+json
        application/manifest+json
        application/rss+xml
        application/vnd.geo+json
        application/vnd.ms-fontobject
        application/x-font-ttf
        application/x-web-app-manifest+json
        application/xhtml+xml
        application/xml
        font/opentype
        image/bmp
        image/svg+xml
        image/x-icon
        text/cache-manifest
        text/css
        text/plain
        text/vcard
        text/vnd.rim.location.xloc
        text/vtt
        text/x-component
        text/x-cross-domain-policy;

    # Brotli

    brotli on;
    brotli_comp_level 6;
    brotli_types
        text/xml
        image/svg+xml
        application/x-font-ttf
        image/vnd.microsoft.icon
        application/x-font-opentype
        application/json
        font/eot
        application/vnd.ms-fontobject
        application/javascript
        font/otf
        application/xml
        application/xhtml+xml
        text/javascript
        application/x-javascript
        text/$;

    # Virtual Hosts

    include /etc/nginx/sites-enabled/*;

    # Configs

    include /etc/nginx/conf.d/*.conf;
    include /usr/share/nginx/modules/*.conf;

}
```

Проверяем конфиг nginx

```
nginx -t
```

Запускаем nginx

```
systemctl start nginx
systemctl status nginx
```

Фиксим ошибку с PID в Ubuntu 20

```
mkdir /etc/systemd/system/nginx.service.d
printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/systemd/system/nginx.service.d/override.conf
systemctl daemon-reload
systemctl restart nginx
systemctl status nginx
```

Проверяем Brotli

```
curl -H 'Accept-Encoding: br' -I http://localhost
```

```
Content-Encoding: br
```

Устанавливаем файрвол

```
apt install ufw
```

Добавляем nginx в доступ файрвол

```
nano /etc/ufw/applications.d/nginx.ini
```

```
[Nginx HTTP]
title=Web Server
description=Enable NGINX HTTP traffic
ports=80/tcp

[Nginx HTTPS] \
title=Web Server (HTTPS) \
description=Enable NGINX HTTPS traffic
ports=443/tcp

[Nginx Full]
title=Web Server (HTTP,HTTPS)
description=Enable NGINX HTTP and HTTPS traffic
ports=80,443/tcp
```

Проверяем список приложений доступных через файрвол

```
ufw app list
```

Включаем файрвол

```
ufw enable
```

Разрешаем сервисы доступные через файрволу

```
ufw allow 'Nginx Full'
ufw allow 'OpenSSH'
```

Проверяем статус

```
ufw status
```

```
Status: active

To                         Action      From
--                         ------      ----
Nginx Full                 ALLOW       Anywhere
OpenSSH                    ALLOW       Anywhere
Nginx Full (v6)            ALLOW       Anywhere (v6)
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

Мы установили сервер Ubuntu 20, nginx с модулем Brotli и настроили файрволу.
Теперь необходимо создать пользователя.