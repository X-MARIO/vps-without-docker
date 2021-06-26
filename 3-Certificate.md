# Сертификат выпускать после настройки nginx, чтобы выпустить для всех поддоменов  - https://youtu.be/XDWWxA6g1eA

Генерируем ключ

```
openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Создаёт папку для снипетов. Туда положим настройки, которые будем использовать потом в config

```
mkdir -p /etc/nginx/snippets/
```

Создаём снипет для SSL, который будем подключать во все конфиги сайтов.
Генерировал тут: https://ssl-config.mozilla.org/

```
nano /etc/nginx/snippets/ssl-params.conf
```

```
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;

ssl_dhparam /etc/ssl/certs/dhparam.pem;

ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;

add_header Strict-Transport-Security "max-age=63072000" always;
```

Обновляем snapd

```
sudo apt update
sudo apt install snapd
snap install core
```

Устанавливаем certbot

```
snap install --classic certbot
```

Проверка certbot

```
ln -s /snap/bin/certbot /usr/bin/certbot
```

Выпускаем сертификат: email, Y, N, 1 2

```
certbot certonly --nginx
```

Меняет конфиг сайта

```
nano /etc/nginx/sites-available/apps.conf
```

```
server {
    listen 80;
    listen [::]:80;

    server_name heal-candy.ru www.heal-candy.ru;
    return 301 https://heal-candy.ru$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name www.heal-candy.ru;
    return 301 https://heal-candy.ru$request_uri;

    ssl_certificate /etc/letsencrypt/live/heal-candy.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/heal-candy.ru/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/heal-candy.ru/chain.pem;

    include snippets/ssl-params.conf;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name heal-candy.ru;
    root /var/www/apps/html;
    index index.html index.xml;

    ssl_certificate /etc/letsencrypt/live/heal-candy.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/heal-candy.ru/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/heal-candy.ru/chain.pem;

    include snippets/ssl-params.conf;
}
```

Проверяем конфиг

```
nginx -t
```

Рестартуем nginx

```
systemctl restart nginx
```

Проверяет домен

- `curl http://heal-candy.ru` — редирект
- `curl https://heal-candy.ru` — сайт
