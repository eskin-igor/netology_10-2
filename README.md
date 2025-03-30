# netology_10-2
# Домашнее задание к занятию «Кластеризация и балансировка нагрузки».

## Задание 1
* Запустите два simple python сервера на своей виртуальной машине на разных портах
* Установите и настройте HAProxy, воспользуйтесь материалами к лекции по ссылке
* Настройте балансировку Round-robin на 4 уровне.
* На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

## Решение 1

Создаём две директории.
```
mkdir http1
mkdir http2
```
Создаём конфигурационный файл в каждой из директорий.
```
nano index.html
```
Содержание конфигурационного файла index.html в директории http1.
```
Server1 :8888
```
Содержание конфигурационного файла index.html в директории http2:
```
Server2 :9999
```
Запустим http сервера в среде python.
```
python3 -m http.server 8888 --bind 0.0.0.0
python3 -m http.server 9999 --bind 0.0.0.0
```
Проведём проверку.
![](https://github.com/eskin-igor/netology_10-2/blob/main/10-2/10-2-1-1.PNG)

![](https://github.com/eskin-igor/netology_10-2/blob/main/10-2/10-2-1-2.PNG)

Теперь установим HAProxy.
```
apt install haproxy
```
Отредактируем конфигурационный файл haproxy.cfg.  
Допишем раздел для сбора статистики:
```
listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics
```
Допишем раздел фронтэнд:
```
frontend example
        mode http
        bind :8088
        default_backend web_tcp
```
Допишем раздел бэкэнд:
```
listen web_tcp
       bind :1325
       server s1 127.0.0.1:8888 check inter 3s
       server s2 127.0.0.1:9999 check inter 3s
```
В данном случае балансировка серверов происходит по кругу (Round-robin) и на 4 уровне модели взаимодействия открытых систем (OSI).

Демонстрация переключения серверов по кругу (Round-robin).
![](https://github.com/eskin-igor/netology_10-2/blob/main/10-2/10-2-1-3.PNG)

Демонстрация переключения серверов на 4 уровне модели OSI.
![](https://github.com/eskin-igor/netology_10-2/blob/main/10-2/10-2-1-4.PNG)

Конфигурационный файл [haproxy.cfg.](https://github.com/eskin-igor/netology_10-2/blob/main/10-2/haproxy_10-2-1.cfg)


## Задание 2
* Запустите три simple python сервера на своей виртуальной машине на разных портах
* Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
* HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
* На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

## Решение 2

Запускаем 3-й сервер http, по аналогии с предыдущим заданием.  
Создаём директорию.
```
mkdir http3
```
Создаём конфигурационный файл в директорий.
```
nano index.html
```
Содержание конфигурационного файла index.html в директории http3.
```
Server3 :7777
```
Запустим http сервер в среде python.
```
python3 -m http.server 7777 --bind 0.0.0.0
```
Редактируем конфигурационный файл haproxy.cfg.  
Редактируем раздел фронтэнд:
```
frontend example
        mode http
        bind :8088
        acl ACL_example.com hdr(host) -i example.com
        use_backend web_servers if ACL_example.com
```
Редактируем раздел бэкэнд:
```
backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check weight 2
        server s2 127.0.0.1:9999 check weight 3
        server s3 127.0.0.1:7777 check weight 4
```
В данном случае балансировка серверов происходит по кругу (Round-robin) в соответствии с весом каждого сервера. И происходит балансировка на 7 уровне модели взаимодействия открытых систем (OSI).

Демонстрация переключения серверов по кругу (Round-robin) с учётом весов серверов. А также реакция на запрос без имени домена example.local и с именем домена example.local.
![](https://github.com/eskin-igor/netology_10-2/blob/main/10-2/10-2-2-1.PNG)

Демонстрация переключения серверов на 7 уровне модели OSI.
![](https://github.com/eskin-igor/netology_10-2/blob/main/10-2/10-2-2-2.PNG)

Конфигурационный файл [haproxy.cfg.](https://github.com/eskin-igor/netology_10-2/blob/main/10-2/haproxy_10-2-2.cfg)
