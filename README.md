# Домашнее задание "Фильтрация трафика"

## 1. Knocking port

centralRouter может попасть на ssh inetrRouter через knock скрипт.
На centralRouter выполняем следующие команды:

```
./knock.sh 192.168.255.1 8881 7777 9991
ssh vagrant@192.168.255.1
```

Для подключения вводим пароль "vagrant"

## 2. Доступ к nginx

Nginx запущен на centralServer. При обращении к inetRouter2 на порт 8080 откроется страница доступная на порту 80 centralServer.
Страница доступна по ссылке http://192.168.11.151:8080/