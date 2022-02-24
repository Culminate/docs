---
title: drone.io
description: 
published: true
date: 2022-02-24T17:18:28.295Z
tags: 
editor: markdown
dateCreated: 2022-02-24T13:52:30.882Z
---

# Установка и разворачивание drone.io
OS: Debian 10 buster
Drone.io: v1.9.1

Нам понадобится установить docker
P.S. Чтобы работать без sudo, нужно добавить пользователя в группу `docker`

1. Скачать образ drone.io `docker pull drone/drone:1`
2. Скачать SSH runner, чтобы drone.io в докере связывался с виртуалкой на astra linux. `docker pull drone/drone-runner-ssh`
3. В Gitea добавить OAuth2 приложение (URI: http://<ВАШ ХОСТ, КОТОРЫЙ ДОСТПУЕН GITEA>/login) https://docs.drone.io/server/provider/gitea/
4. После создания приложения в gitea нам понадобится Client_ID и Client_Secret 
5. Запустить drone.io
    - Здесь DRONE_GITEA_SERVER находится по адресу `http://192.168.1.1:3000`
    - DRONE_RPC_SECRET создаётся `openssl rand -hex 16`
    - DRONE_SERVER_HOST ваш хост
    - DRONE_USER_CREATE задаём администратора, чтобы задавать секреты на организацию

    ```bash
    docker run \
      --volume=/var/lib/drone:/data \
      --env=DRONE_GITEA_SERVER=http://192.168.1.1:3000 \
      --env=DRONE_GITEA_CLIENT_ID=728c7a03-98a3-4f5b-9609-7491bb1f04f8 \
      --env=DRONE_GITEA_CLIENT_SECRET=YsLGMYrFf92l100APf0DRF0rI6qaUfzN43GLO1vlpFA= \
      --env=DRONE_RPC_SECRET=4aa2399d51f74f7b7ef6fd495cfb4af2 \
      --env=DRONE_SERVER_HOST=192.168.1.99:2000 \
      --env=DRONE_SERVER_PROTO=http \
      --env=DRONE_GIT_ALWAYS_AUTH=true \
      --env=DRONE_USER_CREATE=username:grigoriev.ve,admin:true \
      --publish=2000:80 \
      --publish=2443:443 \
      --restart=always \
      --detach=true \
      --name=drone \
      drone/drone:2
    ```

6. Запустить ssh runner

    ```bash
    docker run -d \
      -e DRONE_RPC_PROTO=http \
      -e DRONE_RPC_HOST=192.168.1.99:2000 \
      -e DRONE_RPC_SECRET=4aa2399d51f74f7b7ef6fd495cfb4af2 \
      -p 3000:3000 \
      --restart always \
      --name runner \
      drone/drone-runner-ssh:1
    ```

7. Задаём секреты на организацию через CLI
    Секреты нужны для того чтобы не писать в общедоступных пайплайнах чувствительную информацию.

    7.1 Скачать CLI https://github.com/drone/drone-cli

    ```bash
    curl -L https://github.com/drone/drone-cli/releases/latest/download/drone_linux_amd64.tar.gz | tar zx
    sudo install -t /usr/local/bin drone
    ```

    7.2 Задать настройки подключения CLI, либо каждый раз запускать с ключами `drone -s http://drone.mycompany.com -t eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`
    ```bash
    export DRONE_SERVER=http://drone.mycompany.com
    export DRONE_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
    ```

    7.3 Добавить секреты
   ```bash
    drone orgsecret add --allow-pull-request repository host 192.168.1.75
    drone orgsecret add --allow-pull-request repository user test
    drone orgsecret add --allow-pull-request repository pass test
   ```

8. Написать pipeline и положить его в корень репозитория
    Этот пайплайн создан для репозитория repository. При Pull реквесте он загружает все репозитории, переключает на требуемую ветку все репозитории и если не нашёл, то переключается на develop

    ```yaml
    kind: pipeline
    type: ssh
    name: default

    server:
      host:
        from_secret: host
      user:
        from_secret: user
      password:
        from_secret: pass

    clone:
      disable: true

    steps:
    - name: clone
      commands:
        - git clone git@repo:repository.git .
        - git submodule init
        - git submodule update --remote
        - if git show-ref --quiet refs/remotes/origin/${DRONE_TARGET_BRANCH}; then git checkout ${DRONE_TARGET_BRANCH}; fi
        - git submodule foreach "if git show-ref --quiet refs/remotes/origin/${DRONE_TARGET_BRANCH}; then git checkout ${DRONE_TARGET_BRANCH}; fi"

    - name: build
      commands:
        - mkdir build
        - cd build
        - cmake ..
        - make $(if [ "$DRONE_REPO_NAME" != "repository" ]; then echo ${DRONE_REPO_NAME#ctl_}; fi;)

    trigger:
      event:
      - pull_request
    ```
