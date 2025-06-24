---
title: 'Podman 部署 immich (rootless + quadlet)'
published: 2025-06-24T08:02:45.975Z
description: ''
updated: ''
tags:
  - Note
  - Podman
  - Immich
draft: false
pin: 0
toc: true
lang: ''
abbrlink: ''
---

创建secret，存储database密码

```shell
echo "postgres" | podman secret create immich_db_password -
```

`shared-external.network`：

```toml
[Network]
NetworkName=shared-external
```

`immich.pod`：

```toml
[Unit]
Description=Immich pod
Requires=shared-external-network.service
After=shared-external-network.service

[Pod]
PodName=immich

Network=shared-external
PublishPort=2283:2283

[Install]
WantedBy=default.target
```

`immich-server.container`：

```toml
[Unit]
Description=Immich server container
Requires=immich-pod.service immich-redis.service immich-postgres.service immich-machine-learning.service
After=immich-pod.service immich-redis.service immich-postgres.service immich-machine-learning.service

[Container]
ContainerName=immich-server
Image=ghcr.io/immich-app/immich-server:release

Environment=TZ=Asia/Shanghai
Environment=DB_DATABASE_NAME=immich
Environment=DB_USERNAME=postgres
Environment=DB_HOSTNAME=immich-postgres
Environment=REDIS_HOSTNAME=immich-redis

Secret=immich_db_password,type=env,target=DB_PASSWORD

Volume=/containers/immich/library:/usr/src/app/upload
Volume=/media/gallery:/gallery
Volume=/etc/localtime:/etc/localtime:ro

GroupAdd=keep-groups
AddDevice=/dev/dri/card0
AddDevice=/dev/dri/renderD128

Pod=immich.pod

[Service]
Restart=always

[Install]
WantedBy=immich-pod.service
```

`immich-machine-learning.container`：

```toml
[Unit]
Description=Immich machine learning container
Requires=immich-pod.service
After=immich-pod.service

[Container]
ContainerName=immich-machine-learning
Image=ghcr.io/immich-app/immich-machine-learning:release

Environment=TZ=Asia/Shanghai

Volume=/containers/immich/model_cache:/cache

Pod=immich.pod

[Service]
Restart=always

[Install]
WantedBy=immich-pod.service
```

`immich-redis.container`：

```toml
[Unit]
Description=Immich redis container
Requires=immich-pod.service
After=immich-pod.service

[Container]
ContainerName=redis
Image=docker.io/valkey/valkey@sha256:ff21bc0f8194dc9c105b769aeabf9585fea6a8ed649c0781caeac5cb3c247884

Environment=TZ=Asia/Shanghai

HealthCmd=redis-cli ping || exit 1

Pod=immich.pod

[Service]
Restart=always

[Install]
WantedBy=immich-pod.service
```

`immich-database.container`：

```toml
[Unit]
Description=Immich database container
Requires=immich-pod.service
After=immich-pod.service

[Container]
ContainerName=database
Image=ghcr.io/immich-app/postgres@sha256:fa4f6e0971f454cd95fec5a9aaed2ed93d8f46725cc6bc61e0698e97dba96da1

Environment=TZ=Asia/Shanghai

Environment=DB_STORAGE_TYPE=HDD
Environment=POSTGRES_INITDB_ARGS=--data-checksums
Environment=POSTGRES_DB=immich
Environment=POSTGRES_USER=postgres

Secret=immich_db_password,type=env,target=POSTGRES_PASSWORD

Volume=/containers/immich/postgres:/var/lib/postgresql/data

Pod=immich.pod

[Service]
Restart=always

[Install]
WantedBy=immich-pod.service
```
