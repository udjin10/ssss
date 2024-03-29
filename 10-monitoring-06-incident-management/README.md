# Домашняя работа к занятию 10.6 «Инцидент-менеджмент»

## Задание 

Составьте постмортем, на основе реального сбоя системы Github в 2018 году.

Информация о сбое доступна [в виде краткой выжимки на русском языке](https://habr.com/ru/post/427301/) , а
также [развёрнуто на английском языке](https://github.blog/2018-10-30-oct21-post-incident-analysis/).

---
### Краткое описание инцидента
21 октября 2018 г. в 22:52 UTC после техобслуживания сетевого оборудования на 43 секунды прервалась связь между US East Coast network hub и US East Coast data center. Это вызвало цепь событий, которая привела к частичной недоступности сервисов GitHub в течение 24 часов и 11 минут.

### Предшествующие события
Техническое обслуживание оптического сетевого оборудования.

### Причина инцидента
Потеря согласованности между БД в основном датацентре на Восточном побережье США и ядром сети (network hub).

### Воздействие
В течение 24 часов и 11 минут у 100% пользователей не было возможности создать Issue, оставить комментарий, не работали Webhooks и сайты, хостящиеся на GitHub Pages.

### Обнаружение
В 22:54 UTC начали поступать алёрты о многочисленных ошибках в системе. В 23:02 UTC дежурные инженеры определили, что множество кластеров баз данных находились в неожиданном (unexpected) состоянии.

### Реакция
Дежурные инженеры вручную остановили деплой и перевели систему в статус "желтый".

Координатор по инцидентам изменил статус сервиса на "красный".

Были подключены инженеры из команды администрирования БД.

Была отключена часть сервисов для пользователей, чтобы снизить нагрузку на кластер и обезопасить систему от дальнейшей потери пользовательских данных.

### Восстановление
БД в датацентре на Восточном побережье была восстановлена из бэкапа, вручную сконфигурирована как primary для всех кластеров и топология репликации была перестроена. Пользовательские данные не пострадали.

### Таймлайн
- 21.10.2008 22:52 UTC: потеря связи между US East Coast network hub и US East Coast data center
- 21.10.2008 22:54 UTC: дежурные инженеры получают оповещения от системы мониторинга
- 21.10.2008 23:02 UTC: появляется понимание, что топология кластера БД в некорректном состоянии
- 21.10.2008 23:07 UTC: для предотвращения ухудшения ситуации внтуренние обработчики переведены в режим read-only
- 21.10.2008 23:09 UTC: статус работоспособности GitHub -- жёлтый, информация передана ответственному координатору инцидента
- 21.10.2008 23:11 UTC: координатор инцидента принял решение установить статус -- красный
- 21.10.2008 23:13 UTC: все сервисы переведены в режим read-only для восстановления работоспособности
- 21.10.2008 23:19 UTC: отключены webhook's и Github Pages
- 22.10.2008 00:05 UTC: принято решение восстановить базу из бэкапа и провести синхронизацию кластера
- 22.10.2008 00:41 UTC: запущено восстановление из бэкапа
- 22.10.2008 06:51 UTC: часть БД на Восточном побережье восстановлены, идёт репликация с Западным
- 22.10.2008 07:46 UTC: все БД на Восточном побережье восстановлены
- 22.10.2008 11:12 UTC: восстановился основной кластер БД на восточном побережье, базы продолжали репликацию
- 22.10.2008 13:15 UTC: зафиксирован пик нагрузки трафика на GitHub.com, разрыв между мастером БД и репликами стал увеличиваться, вместо того чтобы сокращаться. Инженеры приняли решения развернуть дополнительные реплики MySQL на чтение
- 22.10.2008 16:24 UTC: восстановление завершено, репликация завершена, топология восстановлена
- 22.10.2008 16:45 UTC: начинает обрабатываться очередь из скопившихся webhook'ов
- 22.10.2008 23:03 UTC: работа веб-хуков и GutHub Pages восстановлена, связность всех систем подтверждена

 
### Последующие действия
Сделаны выводы и запланированы изменения в системе:
- оркестратор не должен меня на primary БД в другом регионе, чтобы не нарушать топологию;
- улучшение системы мониторинга для сбора более точной и чёткой инормации о происходящих ошибках;
- улучшение архитектуры системы датацентров с целью обеспечения дополнительной избыточности хранимых данных для большей надёжности;
- проведение превентиных проверок стабильности системы для предотвращения проблем;
- появилось понимание, что существующие механизмы обеспечения бесперебойной работоспособности GitHub уже недостаточны для текущего уровня развития.
- компания будет инвестировать больше в chaos engineering, тестируя различные сценарии отказа;



