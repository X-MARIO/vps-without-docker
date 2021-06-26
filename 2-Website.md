# Сайт - https://youtu.be/XDWWxA6g1eA

Создаём папку для сайта

```
mkdir -p /var/www/apps/html
```

Добавляем индексный файл

```
nano /var/www/apps/html/index.html
```

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Например</title>
    <meta charset="utf-8">
</head>
<body>
    <h1>Например</h1>
</body>
</html>
```

Создаём папки для конфига

```
mkdir -p /etc/nginx/sites-available/
mkdir -p /etc/nginx/sites-enabled/
```

Правим конфиг

```
nano /etc/nginx/sites-available/apps.conf
```

```
server {
    listen 80;
    listen [::]:80;

    server_name example.com www.example.com;
    root /var/www/apps/html;
    index index.html index.xml;
}
```

Включаем конфиг
Делам симлинку из доступных сайтов во включённые

```
ln -s /etc/nginx/sites-available/apps.conf /etc/nginx/sites-enabled/
```

Проверяем конфиг

```
nginx -t
```

Рестартуем nginx

```
systemctl restart nginx
```

Проверяем домен

```
curl heal-candy.ru
```
