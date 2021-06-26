# Пользователь - https://youtu.be/XDWWxA6g1eA

Добавляем пользователя

```
adduser user
```

Даём права sudo для user

```
adduser user sudo
```

Открываем настройку ssh

```
nano /etc/ssh/sshd_config
```

```
PermitRootLogin no
```

Рестартуем ssh

```
systemctl restart sshd
```

Выходим

```
exit
```

Подключаемся от user

```
ssh user@ip
```

Создаём папку для ключей

```
mkdir ~/.ssh
```

Локально: копируем публичный ключ

```
cat ~/.ssh/id_rsa.pub
```

Сервер: вставляет ключ

```
nano ~/.ssh/authorized_keys
```

Выставляем папке права

```
chmod -R 700 ~/.ssh/
```

Открывает настройку ssh

```
sudo nano /etc/ssh/sshd_config
```

Отключаем вход по паролю

```
PasswordAuthentication no
```

Рестартуем ssh

```
sudo systemctl restart sshd
```

Открываем настройку sudo

```
sudo visudo
```

Настраиваем sudo без пароля

```
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

Забираем права на папку для сайтов

```
sudo chown -R $USER:$USER /var/www/
```
