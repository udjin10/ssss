# Домашняя работа к занятию 10.2 «Системы мониторинга»

## Обязательные задания

### 1. Опишите основные плюсы и минусы pull и push систем мониторинга.

#### Push

Плюсы:
- Можно слать данные в несколько таргетов. Это может быть полезно не только для репликации, но и в принципе чтобы слать в разные контуры. Например, если на сервере несколько сервисов, не связанных друг с другом
- Работает за NAT
- Можно мониторить ноды без лишних `alerts`, если им не всегда нужно подключение к сети, или у них оно не всегда есть, например какие-то мобильные ноды, или при инвентаризации рабочих станций, которые включают/выключают постоянно
- Можно получить данные с хостов,  с которых мы их изначально не ждали. Иными словам, при вводе ноды в эксплуатацию, нужно настроить только ноду, сервер настраивать не нужно

Минусы:
- Агенты могут зафлудить сервера запросами и устроить ему DDoS
- Требует открытия порта сервера во вне, что может создать проблемы со службой безопасности и безопасности в принципе
- Могут приходить данные, которые нам не нужны, т.е. сервер не контролирует ничего: частоту отправки данных, объём и тд. 

#### Pull

Плюсы:
- Не требует открытия порта сервера во вне. При этом, порт должен быть открыт на клиенте, но с точки зрения безопасности это предпочтительней
- Подойдёт в ситуации, когда с ноды могут запрашивать данные разные сервисы, каждому из которых нужны свои данные
- Сервер тянет данные с агентов когда может, и если сейчас нет свободных ресурсов - заберёт данные позже
- Сервер сам определяет, в каком объёме нужны данные
- Проще защитить трафик, т.к. часто используется HTTP/S

Минусы:
- Не работает за NAT, либо надо ставить какой-нибудь прокси
- Менее производительный, более ресурсоёмкий, т.к. данные забираются по HTTP/S в основном


### 2. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

- **Prometheus:** Pull (Push с Pushgateway)
- **TICK:** Push
- **Zabbix:** Push (Pull с Zabbix Proxy)
- **VictoriaMetrics:** Push/Pull, зависит от источника
- **Nagios:** Pull

### 3. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк, используя технологии docker и docker-compose.

#### В виде решения на это упражнение приведите выводы команд с вашего компьютера (виртуальной машины):

>     - curl http://localhost:8086/ping
>     - curl http://localhost:8888
>     - curl http://localhost:9092/kapacitor/v1/ping
> 
>P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например
`./data:/var/lib:Z`
```
dmitry@Lenovo-B50:~/netology/sandbox$ curl http://localhost:8086/ping -v
*   Trying 127.0.0.1:8086...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8086 (#0)
> GET /ping HTTP/1.1
> Host: localhost:8086
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Request-Id: d022772c-f38e-11ec-807a-0242ac150003
< X-Influxdb-Build: OSS
< X-Influxdb-Version: 1.8.10
< X-Request-Id: d022772c-f38e-11ec-807a-0242ac150003
< Date: Fri, 24 Jun 2022 07:25:24 GMT
<
* Connection #0 to host localhost left intact
```

```
dmitry@Lenovo-B50:~/netology/sandbox$ curl http://localhost:8888 -v
*   Trying 127.0.0.1:8888...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8888 (#0)
> GET / HTTP/1.1
> Host: localhost:8888
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: public, max-age=3600
< Content-Length: 336
< Content-Security-Policy: script-src 'self'; object-src 'self'
< Content-Type: text/html; charset=utf-8
< Etag: "3362220244"
< Last-Modified: Tue, 22 Mar 2022 20:02:44 GMT
< Vary: Accept-Encoding
< X-Chronograf-Version: 1.9.4
< X-Content-Type-Options: nosniff
< X-Frame-Options: SAMEORIGIN
< X-Xss-Protection: 1; mode=block
< Date: Fri, 24 Jun 2022 07:26:17 GMT
<
* Connection #0 to host localhost left intact
<!DOCTYPE html><html><head><meta http-equiv="Content-type" content="text/html; charset=utf-8"><title>Chronograf</title><link rel="icon shortcut" href="/favicon.fa749080.ico"><link rel="stylesheet" href="/src.9cea3e4e.css"></head><body> <div id="react-root" data-basepath=""></div> <script src="/src.a969287c.js"></script> </body></html>
```

```
dmitry@Lenovo-B50:~/netology/sandbox$ curl http://localhost:9092/kapacitor/v1/ping -v
*   Trying 127.0.0.1:9092...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 9092 (#0)
> GET /kapacitor/v1/ping HTTP/1.1
> Host: localhost:9092
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 204 No Content
< Content-Type: application/json; charset=utf-8
< Request-Id: 1680e954-f38f-11ec-8080-000000000000
< X-Kapacitor-Version: 1.6.4
< Date: Fri, 24 Jun 2022 07:27:22 GMT
<
* Connection #0 to host localhost left intact
```


#### А также скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`):
![](media/chronograf.png)

### 4. Перейдите в веб-интерфейс Chronograf (`http://localhost:8888`) и откройте вкладку `Data explorer`:

>    - Нажмите на кнопку `Add a query`
>    - Изучите вывод интерфейса и выберите БД `telegraf.autogen`
>    - В `measurments` выберите mem->host->telegraf_container_id , а в `fields` выберите used_percent. 
>    Внизу появится график утилизации оперативной памяти в контейнере telegraf.
>    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. 
>    Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

#### Для выполнения задания приведите скриншот с отображением метрик утилизации места на диске (disk->host->telegraf_container_id) из веб-интерфейса:
![](media/chronograf_disk.png)

### 5. Изучите список [telegraf inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs). Добавьте в конфигурацию telegraf следующий плагин - [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker):
>```
>[[inputs.docker]]
>  endpoint = "unix:///var/run/docker.sock"
>```
>
>Дополнительно вам может потребоваться донастройка контейнера telegraf в `docker-compose.yml` дополнительного volume и 
>режима privileged:
>```
>  telegraf:
>    image: telegraf:1.4.0
>    privileged: true
>    volumes:
>      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
>      - /var/run/docker.sock:/var/run/docker.sock:Z
>    links:
>      - influxdb
>    ports:
>      - "8092:8092/udp"
>      - "8094:8094"
>      - "8125:8125/udp"
>```
>
>После настройки перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список `measurments` в 
>веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker.
>
>Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.

#### Приведите скриншотом список measurments в веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker:
![](media/chronograf_docker.png)

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

> В веб-интерфейсе откройте вкладку `Dashboards`. Попробуйте создать свой dashboard с отображением:
>
>    - утилизации ЦПУ
>    - количества использованного RAM
>    - утилизации пространства на дисках
>    - количество поднятых контейнеров
>    - аптайм
>    - ...
>    - фантазируйте)
>    
![](media/chronograf_dashboard.png)



