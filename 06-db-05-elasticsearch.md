# Домашнее задание к занятию 5. «Elasticsearch»

## Задача 1

### В этом задании вы потренируетесь в:

* установке Elasticsearch,
* первоначальном конфигурировании Elasticsearch,
* запуске Elasticsearch в Docker.
* Используя Docker-образ centos:7 как базовый и документацию по установке и запуску Elastcisearch:

* составьте Dockerfile-манифест для Elasticsearch,
* соберите Docker-образ и сделайте push в ваш docker.io-репозиторий,
* запустите контейнер из получившегося образа и выполните запрос пути / c хост-машины.
* Требования к elasticsearch.yml:

* данные path должны сохраняться в /var/lib,
* имя ноды должно быть netology_test.
* В ответе приведите:

*  текст Dockerfile-манифеста,
```
FROM debian:bullseye

RUN apt-get update && apt-get -y install wget apt-utils gnupg dirmngr; apt-get clean all && \
        groupadd --gid 1000 elasticsearch && \
        useradd -c "elasticsearch" -g elasticsearch elasticsearch && \
        mkdir /var/lib/elasticsearch/ && \
        chown -R elasticsearch:elasticsearch /var/lib/elasticsearch/ && \
        wget -qO - https://mirror.g-soft.info/elasticsearch/GPG-KEY-elasticsearch | gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg && \
        wget https://mirror.g-soft.info/elasticsearch/elasticsearch-8.6.2-amd64.deb && \
        dpkg -i elasticsearch-8.6.2-amd64.deb


USER elasticsearch

WORKDIR /usr/share/elasticsearch

ENV EL_VER=8.6.2

EXPOSE 9200

CMD ["bin/elasticsearch"]
```

Удобнее и дешевле использовать готовый образ по этому его и использовал.
```
version: "3"
services:

  elasticsearch:
    image: elasticsearch:8.6.0
    container_name: elasticsearch
    ports:
      - 9200:9200
      - 9300:9300
    volumes:
      - elast_data:/var/lib/elasticsearch
      - elast_logs:/var/log/elasticsearch/log
    environment:
      - cluster.name=netology_test
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms256m -Xmx256m
      - xpack.security.enabled=false
      - xpack.license.self_generated.type=trial
    networks:
      - subnet
    restart: always

volumes:
  elast_data:
  elast_logs:

networks:
  subnet:
    driver: bridge
```
*  ссылку на образ в репозитории dockerhub,

https://hub.docker.com/repository/docker/romera14/elastic-deb11/general

*  ответ Elasticsearch на запрос пути / в json-виде.
![](https://github.com/Romera14/11-01_BD/blob/main/Снимок%20экрана%202023-07-12%20в%2005.23.22.png)

Подсказки:

* возможно, вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum,
* при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml,
* при некоторых проблемах вам поможет Docker-директива ulimit,
* Elasticsearch в логах обычно описывает проблему и пути её решения.
* Далее мы будем работать с этим экземпляром Elasticsearch.

## Задача 2

### В этом задании вы научитесь:

* создавать и удалять индексы,
* изучать состояние кластера,
* обосновывать причину деградации доступности данных.

Ознакомьтесь с документацией и добавьте в Elasticsearch 3 индекса в соответствии с таблицей:

Имя	Количество реплик	Количество шард
ind-1	0	1
ind-2	1	2
ind-3	2	4

Получите список индексов и их статусов, используя API, и приведите в ответе на задание.
![](https://github.com/Romera14/11-01_BD/blob/main/состояние%20кластера.png)

Получите состояние кластера Elasticsearch, используя API.
![](https://github.com/Romera14/11-01_BD/blob/main/состояние%20кластера2.png)

Как вы думаете, почему часть индексов и кластер находятся в состоянии yellow?

* Потому что мы указали количество реплик больше 1, а реплицировать нем некуда, потому что одна нода.
 
Удалите все индексы.

Важно

При проектировании кластера Elasticsearch нужно корректно рассчитывать количество реплик и шард, иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

### В этом задании вы научитесь:

* создавать бэкапы данных,
* восстанавливать индексы из бэкапов.
* Создайте директорию {путь до корневой директории с Elasticsearch в образе}/snapshots.

Используя API, зарегистрируйте эту директорию как snapshot repository c именем netology_backup.

* добавил в docker-compose манифест строке в env: ```- path.repo=/usr/share/elasticsearch/snapshots``` и пересоздал контейнер.

Приведите в ответе запрос API и результат вызова API для создания репозитория.
```
curl -X PUT -u elastic:changeme "localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/snapshots",
    "compress": true
  }
}'
{"acknowledged":true}
```

Создайте индекс test с 0 реплик и 1 шардом и приведите в ответе список индексов.
```
paromov@debian11:~/docker_elasticsearch$ curl -X PUT "http://localhost:9200/_snapshot/netology_backup/%3Cmy_snapshot_%7Bnow%2Fd%7D%3E?pretty"
{
  "accepted" : true
}
```

Создайте snapshot состояния кластера Elasticsearch.
```
curl -X PUT "http://localhost:9200/_snapshot/netology_backup/%3Cmy_snapshot_%7Bnow%2Fd%7D%3E?pretty"
```

Приведите в ответе список файлов в директории со snapshot.
```
paromov@debian11:~/docker_elasticsearch$ docker exec -u root -it c85d47942aa1 ls -lah /usr/share/elasticsearch/snapshots
total 48K
drwxrwxr-x 3 elasticsearch root 4.0K Jul 22 03:05 .
drwxrwxr-x 1 root          root 4.0K Jul 22 02:42 ..
-rw-rw-r-- 1 elasticsearch root  855 Jul 22 03:05 index-0
-rw-rw-r-- 1 elasticsearch root    8 Jul 22 03:05 index.latest
drwxrwxr-x 4 elasticsearch root 4.0K Jul 22 03:05 indices
-rw-rw-r-- 1 elasticsearch root  19K Jul 22 03:05 meta-Z1u08NjnTGi_3zvu7fgiTg.dat
-rw-rw-r-- 1 elasticsearch root  360 Jul 22 03:05 snap-Z1u08NjnTGi_3zvu7fgiTg.dat
```

Удалите индекс test и создайте индекс test-2. Приведите в ответе список индексов.
```
paromov@debian11:~/docker_elasticsearch$ curl -X DELETE -u elastic:changeme 'http://localhost:9200/test?pretty'
{
  "acknowledged" : true
}
paromov@debian11:~/docker_elasticsearch$ curl -X PUT -u elastic:changeme "http://localhost:9200/test2?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index": {
      "number_of_shards": 1,  
      "number_of_replicas": 0 
    }
  }
}
'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test2"
}
paromov@debian11:~/docker_elasticsearch$ curl -X GET  "http://localhost:9200/_cat/indices?v=true"
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test2 64mQ38qJTuiHOvV9o4KB2w   1   0          0            0       225b           225b
```

Восстановите состояние кластера Elasticsearch из snapshot, созданного ранее.

Приведите в ответе запрос к API восстановления и итоговый список индексов.
```
paromov@debian11:~/docker_elasticsearch$ curl -X POST http://localhost:9200/_snapshot/netology_backup/my_snapshot_2023.07.22/_restore?pretty
{
  "accepted" : true
}
paromov@debian11:~/docker_elasticsearch$ curl -X GET  "http://localhost:9200/_cat/indices?v=true"
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test2 hbkPSAX4Q6Sy8bAXOFrMng   1   0          0            0       225b           225b
green  open   test  LlxbTp1cQ2mmoemRdvq_FQ   1   0          0            0       225b           225b
```

Подсказки:

возможно, вам понадобится доработать elasticsearch.yml в части директивы path.repo и перезапустить Elasticsearch.
