# Сложности локальной разработки в Docker

Предположим у нас есть приложение в каталоге `front`.
Локальная разработка в Docker подразумевает,
что вы запускаете ваше приложение внутри контейнера, например так:

```bash
cd front
docker run --rm -it -v $(pwd):/app -w /app -e TOKEN=12345 -p 3000:3000 node:14 sh -c "yarn install && yarn start"
```

Команда запуска, сокращается при использовании `docker-compose`.
Тут ничего нового.
Создадим каталог `start` на одном уровне с `front`,
положим туда `docker-compose.yml`:

```yaml
version: "3.9"
services:
  front:
    image: node:14
    working_dir: "/app"
    command: sh -c "yarn install && yarn start"
    volumes:
      - ../front:/app
    ports:
      - "3000:3000"
    environment:
      - TOKEN=12345
```

Теперь команда для запуска выглядит так:

```bash
docker-compose up
```

Запустились, открываем `localhost:3000`. И вот какие проблемы вы встретите.

## Слушать все входящие соединения

Многие приложения по умолчанию слушают только `localhost`,
а `localhost` внутри контейнера это не тот же самый, который на вашей локальной машине.

Пример: не забывайте указывать `-b 0.0.0.0` при запуске Rails.

## Сохранение данных между контейнерами

TLDR: сделать volume. Дальше разжёвываем почему.

Когда вы запустили команду, то всё что она сделает произойдёт внутри контейнера
и результат её работы останется там же.

В случае с `yarn` или `npm` результатом работы команды будет каталог `node_modules`,
который находится, как правило, в каталоге с проектом (`front` в нашем примере).
Поэтому когда в следующий раз вы запустите:

```
docker-compose run --rm front yarn install
```

у вашего front приложения уже будут установлены модули
только потому, что в `docker-compose.yml` написано:
```
volumes:
  - ../front:/app
```
Это означает что вы смонтировали ваш локальный каталог внутрь проекта,
и `node_moduldes` сохранилось в вашей файловой системе, а не внутри контейнера.

Но не все команды так работают.
Например `bundle install` по умолчанию сохранит `gem`-ы **вне** директории проекта.
Куда именно &mdash; зависит от настроек, но в любом случае,
если удалится контейнер, с ним удалятся и установленные библиотеки.

Поэтому чтобы эти данные сохранялись между запусками разных контейнеров,
в них нужно монтировать `volume` (либо каталог с локальной машины, либо `docker-volume`)

```yaml
version: "3.9"
services:
  back:
    build:
      context: ../back
      dockerfile: Dockerfile.dev
    command: sh -c "bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - ../back:/app
      - bundle-data:/usr/local/bundle
volumes:
  bundle-data:
```

То же самое относится ко всему что имеет состояние, например к базам данных:

```yaml
version: "3.9"
services:
  db:
    image: postgres:13.4
    volumes:
      - pg-data:/var/lib/postgresql/data
    environment:
      PGDATA: /var/lib/postgresql/data
      POSTGRES_PASSWORD: "postgres"
volumes:
  pg-data:
```

## Запуск команд внутри контейнеров

Теперь ваш `yarn` или любые другие утилиты не утсановлены локально,
а только внутри контейнера.
Чтобы выполнить любую команду, например `yarn`,
вам нужно сказать что-то в духе:

```bash
docker-compose run --rm front yarn add и-тут-что-хочу
```

Понятно, что это крайне неудобно. Как выйти из положения?

Есть [gem `dip`](https://github.com/bibendi/dip) который адресует эти вопросы.
Но опять же, это `gem` который подразумевает дополнительную установку.

Я предпочитаю написать простой bash script `cli` следующего вида:

```bash
#!/bin/bash

command=$1
arguments=${@:2}
container=$2
rest=${@:3}

case $command in

up | start | stop | down | build)
docker-compose $command $arguments
;;

rails | rake | bundle)
docker-compose run --rm back $command $arguments
;;

yarn | npm)
docker-compose run --rm front $command $arguments
;;

help)

cat <<EOF
This is help:

$0 up              # Launch everything
...
EOF

;;

*)
$0 help
;;

esac

```

## Права доступа

в процессе написания...

