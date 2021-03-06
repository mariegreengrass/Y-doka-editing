---
title: Что такое Docker?
name: docker
author: igsekor
co-authors:
designers:
contributors:
tags: 
summary:
  - докер
  - контейнер
  - container
cover: 
---

## Кратко

Docker — это технология, с которой создают и используют приложения в изолированном окружении. В основе Docker лежит амбициозная идея: «Если работает у вас, будет работать где угодно».

Docker чаще всего применяется для разработки, тестирования и развертывания серверных приложений на базе операционной системы Linux, но может использоваться и в мире фронтенда:
* для сборки [бандлов](/js/tools/bundlers);
* для [статического анализа кода](/js/tools/static-analysis);
* для модульного и интеграционного тестирования приложений;
* для тестирования пользовательского интерфейса;
* для подготовки ресурсов приложения (картинок, шрифтов, иконок и пр.);
* для воспроизводимых экспериментов с новыми технологиями и демок;
* для создания изолированного окружения разработчика.

## Как начать

[Установите Docker на свой компьютер](https://docs.docker.com/engine/install/) (хост). Можно поиграть с Docker в [песочнице](https://labs.play-with-docker.com). После запуска Docker в системе можно проверить его работоспособность с помощью графического интерфейса [Dashboard](https://docs.docker.com/desktop/dashboard/) или команды в терминале:

```bash
docker --version
```

У разработчиков Doсker есть готовая демка, запускается так:

```bash
docker run -d -p 80:80 docker/getting-started
```

После загрузки образа, сборки и запуска контейнера на его основе можно открыть в браузере стартовую документацию Docker по адресу [http://localhost](localhost). Готовое веб-приложение — наиболее популярный кейс использования.

## Как понять

### Службы

Docker Engine (движок) — клиент-серверное приложение, которое нужно для управления объектами Docker. Состоит из трех компонентов — Docker Daemon (сервер), Docker API (интерфейс) и Docker CLI (консольный клиент).

![Архитектура Docker](https://docs.docker.com/engine/images/architecture.svg)

Все операции с объектами Docker (образы, контейнеры, тома, внутренняя виртуальная локальная сеть) реализуются с помощью демона dockerd (Docker Daemon), запущенного на вашем компьютере (Docker Host). Работа этого демона основана на технологии контейнеризации [LXC](https://linuxcontainers.org/lxc/introduction/).

[Docker API](https://docs.docker.com/engine/api/) помогает реализовать программный интерфейс, благодаря которому консольный клиент Docker CLI или другие приложения могут получить доступ к системе мониторинга и управления Docker-объектами. Система команд консольного клиента Docker CLI полностью покрывает всю функциональность Docker.

Хранение образов может быть локальным или удаленным. За локальное хранение отвечает демон, за хранение образов на сервере — служба [Docker Registry](https://docs.docker.com/registry/). Список некоторых публичных реестров образов:

* [Docker Hub](https://hub.docker.com/)
* [Google Cloud Container Registry](https://cloud.google.com/container-registry)
* [Azure Container Registry](https://azure.microsoft.com/services/container-registry)
* [IBM Cloud Container Registry](https://www.ibm.com/cloud/container-registry)
* [Oracle Cloud Infrastructure Container Registry](https://www.oracle.com/cloud-native/container-registry/)
* [Yandex Container Registry](https://cloud.yandex.com/services/container-registry)

Docker Desktop — пакет приложений, включающий виртуальную машину для работы с движком (Docker Engine), визуальный интерфейс (Dashboard), консольный клиент (Docker CLI), инструменты для работы с реестром Docker Hub и пр.

Для платформ Mac и Windows невозможно использовать механизм контейнеризации Linux напрямую — понадобится виртуальная машина (гипервизор) с установленным на нее ядром Linux. Контейнеры [LXC](https://linuxcontainers.org/lxc/introduction/) реализуются на уровне ядра. Если нужна дополнительная специфичность, устанавливаются  надстройки контейнера с конкретной операционной системой Linux на борту. Такая архитектура экономит ресурсы компьютера на Mac или Windows.

Функции Docker Desktop:

* контейнеризация и совместное использование любого приложения на любой облачной платформе, на нескольких языках и фреймворках;
* простая установка и настройка полной среды разработки Docker;
* управление кластером [Kubernetes](https://kubernetes.io);
* в Windows — переключение между средами Linux и Windows Server для создания приложений;
* работа изначально на Linux через WSL 2 на машинах Windows;
* установка тома для кода и данных, включая уведомления об изменении файлов и легкий доступ к запущенным контейнерам в локальной сети;
* разработка и отладка в контейнере с поддерживаемыми IDE.

### Объекты

Docker Image (образ) — прототип будущего котейнера, содержащий его конфигурацию и набор файлов приложения.

Конфигурация может быть описана в файлах конфигурации. Как правило, это Dockerfile или docker-compose.yml.

Конфигурационный файл всегда начинается с объявления родительского образа, имя которого прописывается после директивы `FROM`. Для указания варианта сборки базового образа к имени может быть добавлен тег через двоеточие. Дальше идут настройки.

Например, вы можете упаковать свое Node.js приложение в контейнер, добавив в него файл Dockerfile:

```bash
FROM node:12

# Создание директории приложения
WORKDIR /app

# Установка зависимостей, учитывая package.json и package-lock.json
COPY package*.json ./

RUN npm i

# Сборка проекта, если есть необходимость
# RUN npm ci --only=production

# Копирование исходного кода приложения
COPY . .

EXPOSE 8080
CMD [ "node", "app.js" ]
```

и выполнив команду:

```bash
docker build -t app .
```

После ключа `-t` задается имя образа (здесь `app` выбрано в качестве имени образа). Точка в конце означает, что образ собирается из текущей папки. Имя конфигурационного файла — по умолчанию Dockerfile. После сборки образа можно запустить контейнер на его основе с приложением внутри него. Таким образом сохраняется изолированное и воспроизводимое окружение.

Docker Container (контейнер) — собранное приложение в изолированном окружении, в соответствии с конфигурацией образа.

Контейнер — приложение в изолированном окружении. Окружение формируется послойно. Каждый новый слой расширяет функционал предыдущего.

Файловая система контейнера — стековая ([Union File Systems](https://www.terriblecode.com/blog/how-docker-images-work-union-file-systems-for-dummies/)). Каждая ветвь файловой системы — каталоги и файлы отдельного слоя. Впоследствии они накладываются друг на друга, образуя единое целое, чтобы не было отдельных копий слоев.

Docker Volume (том) — папка, которую можно примонтировать к контейнерам в качестве внешней. Папка может быть связана с конкретной папкой на компьютере.

Тома необходимы для хранения файлов конфигурации, критических с точки зрения безопасности, файлов баз данных, файлов, которые нельзя удалять после окончания работы приложения.

В качестве тома можно примонтировать локальную папку. Но это не совсем правильное использование — оно снижает безопасность системы и приложения.

Docker Network (сеть) — виртуальная локальная сеть для совместного использования нескольких запущенных контейнеров и/или сопряжения запущенного контейнера с компьютером.

В основном используются три основных режима работы сетевой инфраструктуры Docker. По умолчанию виртуальная сеть (сетевая подсистема) использует режим `bridge`. Это означает, что все контейнеры могут взаимодействовать между собой подобно тому, как это реализуется в связке веб-сервера и СУБД. Режим `host` используется для доступа к локальному сетевому окружению на компьютере. Есть еще режим `none`, в котором сеть для контейнеров полностью отключена.

### Инструменты

Docker Hub — официальный реестр образов Docker.

Образы хранит служба [Docker Registry](https://docs.docker.com/registry/). Каждый может бесплатно создавать публичные репозитории образов и один приватный репозиторий.

Существуют официальные образы Docker для многих платформ и языков, они часто обновляются и являются относительно безопасными. Для фронтендера будут интересны:

* [Node](https://hub.docker.com/_/node)
* [Express with compatible of MongoDB](https://hub.docker.com/_/mongo-express)
* [Drupal](https://hub.docker.com/_/drupal)
* [Python](https://hub.docker.com/_/python)
* [PHP](https://hub.docker.com/_/php)
* [Ruby](https://hub.docker.com/_/ruby)

Docker CLI — консольный клиент для управления Docker через интерфейс командной строки.

В нем есть команды для управления объектами Docker. Описания основных команд в документации:
* [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)
* [docker run](https://docs.docker.com/engine/reference/commandline/run/)
* [docker image](https://docs.docker.com/engine/reference/commandline/image/)
* [docker container](https://docs.docker.com/engine/reference/commandline/container/)
* [docker volume](https://docs.docker.com/storage/volumes/)

## Как пользоваться

Ключи командного интерфейса Docker CLI хорошо проработаны, имеют подобное поведение с точностью до функции. Например, дополнительный ключ `prune` удаляет неиспользуемые объекты. Ключ `rm` служит для удаления, а ключ `ls` — для просмотра объектов. У каждого объекта должно быть уникальное имя — или придуманное пользователем, или сформированное с помощью хэш-функции.

### Главные функции Docker CLI

**Мониторинг запущенных контейнеров**

Для просмотра запущенных контейнеров используется команда:

```bash
docker ps
```

Чтобы увидеть не только запущенные в настоящий момент контейнеры, но и уже остановленные, необходимо использовать ключ `-a`:

```bash
docker ps -a
```

Ключ `-s` служит для вывода количества дискового пространства, которое использует каждый запущенный контейнер:

```bash
docker ps -s
```

Если список контейнеров слишком большой, можно использовать ключ `-f` для фильтрации. Например, так выглядит команда для фильтрации запущенных контейнеров по именам, содержащим hello:

```bash
docker ps -f name=hello
```

Полный список ключей для команды `docker ps` доступен в [документации](https://docs.docker.com/engine/reference/commandline/ps/).

**Запуск контейнеров**

Команда для запуска контейнера, который доступен локально или на Docker Hub:

```bash
docker run --name test -i -t hello
```

Ключ `--name` используется для установки имени запущенного контейнера. Ключи -i и -t указывают, что для запуска контейнера будет использоваться [стандартный поток ввода](https://ru.wikipedia.org/wiki/Стандартные_потоки) и [терминала TTY](https://ru.wikipedia.org/wiki/TTY-абстракция) соответственно. Чтобы при запуске контейнера примонтировать том, который будет связан с папкой на вашем компьютере, а потом получить доступ к контейнеру через терминал, необходимо выполнить команду:

```bash
docker run -t -i --mount type=bind,src=/data,dst=/data hello bash
```

Полный список ключей для команды `docker run` доступен в [документации](https://docs.docker.com/engine/reference/commandline/run/).

**Управление образами**

Команда, чтобы посмотреть все образы, доступные локально:

```bash
docker image ls
```

Ключи `prune`, `rm` действуют как обычно — удаляют неиспользуемые или конкретные образы.
Команды для работы с реестром:

```bash
# Для загрузки образа с именем hello из реестра (Docker Hub)
docker image pull hello

# Для загрузки образа с именем hello в реестр (Docker Hub)
docker image push hello
```

Команда для получения полной информации о контейнере hello:

```bash
docker image inspect hello
```

Чтобы собрать контейнер из текущей папки с учетом конфигурации в файле Dockerfile, расположенном в этой же папке, понадобится команда:

```bash
docker image build .
```

Полный список ключей для команды `docker image` — в [документации](https://docs.docker.com/engine/reference/commandline/image/).

**Управление контейнерами**

Команды запуска и остановки контейнеров:

```bash
# Команда для запуска контейнера
docker container start

# Команда для перезапуска контейнера
docker container restart

# Команда для остановки контейнера
docker container stop

# Команда для постановки контейнера на паузу
docker container pause
```

Полный список ключей для команды `docker container` — в [документации](https://docs.docker.com/engine/reference/commandline/container/).

**Управление томами**

Новый том создается командой `docker volume create`. К примеру, создадим том с именем `hello`:

```bash
docker volume create hello
```

Получить исчерпывающую информацию о томе:

```bash
docker volume inspect hello
```

Ключ `ls` используется для просмотра всех томов, которые можно отфильтровать с помощью дополнительного ключа `-f`.

```bash
# Вывод всего списка томов
docker volume ls

# Вывод списка томов, отфильтрованного по имени
docker volume ls -f name=hello
```

Полный список ключей для команды `docker volume` — в [документации](https://docs.docker.com/engine/reference/commandline/volume/).
