
# Отчёт по лабораторной работе: Работа с Docker и Docker Compose
**Выполнил:** Сныткин Илья  
**ОС:** Debian

---

## Задача 1: Сборка и публикация кастомного образа `custom-nginx`

Я создал локальный Docker-образ на основе официального `nginx:1.21.1`. Для этого я подготовил два файла:

### `index.html`
```html
<html>
<head>
Hey, ZGU!
</head>
<body>
<p>I will be IT Engineer!</p>
</body>
</html>
```

### `Dockerfile`
```dockerfile
FROM nginx:1.21.1
COPY index.html /usr/share/nginx/html/index.html
```

**Теория:**
- `FROM` — указывает базовый образ, на основе которого строится новый.
- `COPY` — копирует файл(ы) из контекста сборки (в данном случае — текущей директории) внутрь образа.

Затем я выполнил следующие команды:
```bash
docker build -t custom-nginx:1.0.0 .
docker tag custom-nginx:1.0.0 [USERNAME]/custom-nginx:1.0.0
docker login
docker push [USERNAME]/custom-nginx:1.0.0
```
![Образ](assets/1/Установка.png)

**Пояснение команд:**
- `docker build -t имя:тег .` — собирает образ из `Dockerfile` в текущей директории (`.` — контекст сборки) и присваивает тег.
- `docker tag` — создаёт алиас образа в формате `репозиторий/имя:тег`, необходимый для публикации в Docker Hub.
- `docker login` — аутентифицирует пользователя в Docker Hub.
- `docker push` — отправляет образ в удалённый репозиторий.

Таким образом, я собрал образ, привязал к нему имя моего Docker Hub и загрузил его в публичный репозиторий.

### Собранный образ:![Образ](assets/1/Docker.png)
  
### Ссылка на репозиторий: https://hub.docker.com/repository/docker/xflame0x/custom-nginx

---

##Задача 2: Запуск и диагностика контейнера

Я запустил контейнер с именем `SnytkinIlya-custom-nginx-t2` в фоновом режиме и привязал его к порту `8080` только на `127.0.0.1`:
```bash
docker run -d --name SnytkinIlya-custom-nginx-t2 -p 127.0.0.1:8080:80 custom-nginx:1.0.0
```

**Пояснение флагов:**
- `-d` (`--detach`) — запускает контейнер в фоне (без привязки к терминалу).
- `--name` — задаёт человекочитаемое имя контейнера.
- `-p IP:host_port:container_port` — пробрасывает порт контейнера на указанный IP и порт хоста. Здесь: только на loopback (`127.0.0.1`), порт `8080`.

После этого я переименовал контейнер:
```bash
docker rename Ilya-custom-nginx-t2 custom-nginx-t2
```

Затем выполнил единую диагностическую команду:
```bash
date +"%d-%m-%Y %T.%N %Z" ; sleep 0.150 ; \
docker ps ; \
ss -tlpn | grep 127.0.0.1:8080 ; \
docker logs custom-nginx-t2 -n1 ; \
docker exec -it custom-nginx-t2 base64 /usr/share/nginx/html/index.html
```
**Объяснение:**
- `date` — фиксирует точное время начала проверки.
- `sleep 0.150` — небольшая пауза для стабильности вывода.
- `docker ps` — показывает только **запущенные** контейнеры.
- `ss -tlpn | grep 127.0.0.1:8080`:
    - `ss` — современная замена `netstat` для просмотра сокетов,
    - `-t` — TCP,
    - `-l` — listening (прослушивающие порты),
    - `-p` — показать PID процесса,
    - `-n` — не разрешать имена портов (показывать числа).
- `docker logs -n1` — выводит последнюю строку лога контейнера.
- `docker exec -it ... base64 ...` — выполняет команду внутри контейнера и кодирует вывод в base64, чтобы избежать проблем с экранированием и кодировкой в терминале.

И проверил доступность через `curl`:
```bash
curl http://127.0.0.1:8080
```
![Запуски](assets/2/Диагностика.png)

Всё работало корректно — веб-страница отдавалась, а `base64`-вывод подтверждал, что файл именно мой.


## Задача 3: Взаимодействие с контейнером и изменение конфигурации

Я подключился к контейнеру командой:
```bash
docker attach custom-nginx-t2
```
и сразу нажал **Ctrl-C**. Контейнер остановился — я понял, что это произошло потому, что основной процесс (NGINX) завершился, а в Docker контейнер зависит от PID 1.

![Контейнеры](assets/3/Подключение.png)

**Теория:** В Docker контейнер живёт ровно столько, сколько живёт его главный процесс (PID 1). `docker attach` подключает терминал напрямую к потоку ввода/вывода этого процесса. Нажатие **Ctrl-C** посылает сигнал `SIGINT`, что приводит к штатному завершению NGINX → контейнер останавливается.

Я проверил статус:
```bash
docker ps -a  # контейнер в состоянии Exited
```

Затем перезапустил его:
```bash
docker start custom-nginx-t2
```

И вошёл в интерактивную оболочку:
```bash
docker exec -it custom-nginx-t2 /bin/bash
```

**Флаги `-it`:**
- `-i` (`--interactive`) — сохраняет STDIN открытым, позволяет вводить команды.
- `-t` (`--tty`) — выделяет псевдотерминал, чтобы оболочка работала как обычная.

Внутри контейнера я установил `nano`:
```bash
apt-get update && apt-get install -y nano
```

Отредактировал `/etc/nginx/conf.d/default.conf`, заменив:
```nginx
listen 80;
```
на:
```nginx
listen 81;
```
![Контейнеры](assets/3/Изменения.png)

Перезагрузил NGINX:
```bash
nginx -s reload
```

**Важно:** NGINX поддерживает "плавную" перезагрузку конфигурации без остановки самого сервиса.

И проверил:
```bash
curl http://127.0.0.1:81  # работает
curl http://127.0.0.1:80  # не работает
```

После выхода на хост я выполнил:
```bash
curl http://127.0.0.1:8080  # ошибка!
```

**Объяснение:**  
Docker пробрасывает порт **8080 хоста → порт 80 контейнера** (`-p 127.0.0.1:8080:80`). Но внутри контейнера NGINX теперь слушает **порт 81**, а не 80. Поэтому трафик попадает в "пустой" порт → соединение отклоняется.

В конце я удалил контейнер:
```bash
docker rm -f custom-nginx-t2
```
![Контейнеры](assets/3/ПРИНУДИТЕЛЬНО.png)

![Контейнеры](assets/3/Пустинфо.png)

**Флаг `-f`:** принудительно останавливает и удаляет контейнер, даже если он запущен.

---

## Задача 4: Работа с bind mounts между контейнерами

Я создал директорию и запустил два контейнера, смонтировав текущую папку хоста в `/data`:

```bash
docker run -d --name centos-t4 -v "$(pwd)":/data centos:latest tail -f /dev/null
docker run -d --name debian-t4 -v "$(pwd)":/data debian:stable-slim tail -f /dev/null
```

**Теория:**
- `-v "$(pwd)":/data` — создаёт **bind mount**: директория хоста напрямую монтируется в контейнер. Любые изменения синхронизируются в реальном времени.
- `tail -f /dev/null` — бесконечная команда, удерживающая контейнер в активном состоянии (иначе CentOS/Debian завершились бы сразу).

![Работа](assets/4/Директории.png)

Из контейнера CentOS я создал файл:
```bash
docker exec -it centos-t4 bash -c 'echo "Hello from CentOS!" > /data/centos.txt'
```

На хосте создал второй файл:
```bash
echo "Hello from Host!" > host.txt
```

Затем из контейнера Debian я вывел список и содержимое:
```bash
docker exec -it debian-t4 ls -l /data
docker exec -it debian-t4 cat /data/centos.txt
docker exec -it debian-t4 cat /data/host.txt
```
![Контейнеры](assets/4/Список.png)
Оба файла были видны — это показало, что bind mount даёт **реальное совместное использование файлов** между хостом и контейнерами.

---

## Задача 5: Docker Compose, Registry и Portainer

Согласно официальной документации ([Docker Compose Application Model](https://docs.docker.com/compose/compose-application-model/#the-compose-file)):

> _“Стандартный путь для файла Compose — это  `compose.yaml` (предпочтительный вариант).
Compose также поддерживает  `docker-compose.yaml` для обратной совместимости.
Если оба файла присутствуют, Compose отдаёт предпочтение каноническому файлу compose.yaml.`.”_

Именно это и произошло у меня при первом запуске — запустился только Portainer.

Чтобы запустить оба сервиса, я воспользовался директивой [`include`](https://docs.docker.com/compose/compose-file/14-include/), как рекомендовано в [Compose Specification](https://distribution.github.io/distribution/about/deploying/).

### `docker-compose.yaml` (registry)
```yaml
version: "3"
services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
```

### `compose.yaml` (основной файл с include)
```yaml
version: "3.8"
include:
  - path: ./docker-compose.yaml

services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

**Пояснение:**
- `network_mode: host` — контейнер использует сетевой стек хоста напрямую (Portainer по умолчанию слушает `:9000`).
- `volumes: - /var/run/docker.sock:/var/run/docker.sock` — даёт Portainer доступ к Docker API.

После `docker compose up -d` запустились оба сервиса.

Затем я загрузил свой образ в локальный registry:
```bash
docker tag custom-nginx:1.0.0 127.0.0.1:5000/custom-nginx:latest
docker push 127.0.0.1:5000/custom-nginx:latest
```
![Порт](assets/5/Контейнеры.png)
**Важно:** По умолчанию Docker считает реестры на `localhost:5000` **небезопасными**, поэтому пуш разрешён без TLS.

Через веб-интерфейс Portainer (`https://127.0.0.1:9000`) я:
- создал учётную запись администратора,
- выбрал Local-окружение,
- в разделе **Stacks → Web Editor** задеплоил стек:
![Порт](assets/5/Портайнер.png)

```yaml
version: '3'
services:
  nginx:
    image: 127.0.0.1:5000/custom-nginx
    ports:
      - "9090:80"
```
![Порт](assets/5/Код.png)
Страница стала доступна по `http://127.0.0.1:9090`.

Я открыл Inspect для этого контейнера в Portainer, перешёл во вкладку **Tree → Config** и сделал скриншот от `AppArmorProfile` до `Driver`.
![Порт](assets/5/Inspect.png)
Потом я удалил `compose.yaml` и снова выполнил `docker compose up -d`. Появилось предупреждение:

> **WARNING**: The “compose.yaml” file is missing. Project state may be inconsistent.

**Объяснение:** Docker Compose сохраняет метаданные проекта (включая имя и пути к файлам) в метках контейнеров. При отсутствии файла он использует кэшированную конфигурацию, чтобы не нарушить работу стека — это защитный механизм.

Чтобы аккуратно всё остановить и удалить, я выполнил **одну команду**:
```bash
docker compose down
```

Она остановила и удалила все контейнеры, сети и ресурсы проекта.

---

##  Вывод

Я успешно выполнил все пять задач:
- научился собирать и публиковать образы,
- освоил управление контейнерами (запуск, диагностика, редактирование, удаление),
- понял, как работает проброс портов и почему возникают ошибки при изменении конфигурации внутри контейнера,
- продемонстрировал общий доступ к файлам через bind mounts,
- освоил работу с Docker Compose, включая приоритет файлов и директиву `include`,
- развернул локальный registry и интегрировал его с Portainer.

Все действия подтверждены скриншотами и соответствуют требованиям задания.

