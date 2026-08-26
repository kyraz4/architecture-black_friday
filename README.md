# pymongo-api

[Финальная архитектура](./final_diagram.png)

## Как запустить

0. Перейти в папку с финальным решением

```shell
cd sharding-repl-cache
```

1. Удалить старые контейнеры и тома (при необходимости)

```shell
docker compose down -v
```

2. Запуск проекта

```shell
docker compose up -d
```

3. Проверить состояние

```shell
docker compose ps
```

Должны быть запущены:

configSrv
shard1-1
shard1-2
shard1-3
shard2-1
shard2-2
shard2-3
mongos_router
redis
pymongo_api

4. Проверить Redis

```shell
docker compose exec redis redis-cli ping
```

Ожидаем ответ "PONG"


5. Инициализируем MongoDB (сделать скрипт исполняемым, при необходимости "chmod +x scripts/init-replication.sh")

```shell
../scripts/init-replication.sh
```

6. Заполняем mongodb данными

```shell
../scripts/mongo-init.sh
```

Готово! Можно проверять

## Как проверить

### Если вы запускаете проект на локальной машине

Проверка общего количества документов:

```shell
docker compose exec -T mongos_router mongosh --port 27024 --quiet <<EOF
use somedb
db.helloDoc.countDocuments()
EOF
```

Проверка распределения документов

```shell
docker compose exec -T mongos_router mongosh --port 27024 --quiet <<EOF
use somedb
db.helloDoc.getShardDistribution()
EOF
```

Проверка количества реплик

```shell
docker compose exec -T shard1-1 mongosh --port 27018 --quiet \
  --eval 'rs.status().members.length'
```

```shell
docker compose exec -T shard2-1 mongosh --port 27021 --quiet \
  --eval 'rs.status().members.length'
```

Проверка Redis-кеширования и время выполнения

Несколько раз сделать: 

```shell
time curl -s http://localhost:8080/helloDoc/users > /dev/null
```

P.S. или - можно просто зайти на http://localhost:8080/helloDoc/users, открыть инструменты разраюотчика (раздел Networks) и перезагрузить страницу 

Проверка наличия данных в Redis

```shell
docker compose exec redis redis-cli DBSIZE
```

Откройте в браузере http://localhost:8080

### Если вы запускаете проект на предоставленной виртуальной машине

Узнать белый ip виртуальной машины

```shell
curl --silent http://ifconfig.me
```

Откройте в браузере http://<ip виртуальной машины>:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs
