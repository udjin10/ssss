# ansible-8.3

### Что делает playbook

Плейбук развернёт на каждом из трёх хостов по одному приложению:
- clickhouse
- lighthouse
- vector

Дистрибутивы Clickhouse, Vector в пакетах `rpm`, статика Lighthouse c GitHub и Nginx из epel-репозитория скачиваются при запуске плейбука.

### Какие у него есть параметры 

- IP хостов нужно задать в файле инвентаризации [prod.yml](inventory/prod.yml)
- `Clickhouse` и `vector` будут установлены как службы, `lighthouse` будет установлен в `/home/dmitry/lighthouse` с правами `root`.
- Чтобы открыть `lighthouse` на хосте вместе с ним будет запущен веб-сервер `nginx`
- После установки и запуска `vector` он будет провалидирован

### Какие у него есть теги

- clickhouse
- lighhouse
- nginx
- vector
