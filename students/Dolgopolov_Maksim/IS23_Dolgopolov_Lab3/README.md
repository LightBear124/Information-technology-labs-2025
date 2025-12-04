## Информационные технологии: лабораторная работа №3 (Docker)

### Выполнил: студент группы ИС-23 Долгополов Максим

---

### Задание 1.

Ссылка на репозиторий: https://hub.docker.com/repository/docker/max0194/custom-nginx/general .

### Задание 2.

Выполненные команды:

![Фото 1 задание 2](./screens/containercommands1.PNG)

![Фото 2 задание 2](./screens/containercommands2.PNG)

### Задание 3.

Выполненные команды:
Через docker attach подключаемся к потоку ввода/вывода/ошибок контейнера.

![Фото 1 задание 3](./screens/3task1.PNG)

Комбинация Ctrl+C отправляет сигнал потоку attach об прекращении отслеживания.

![Фото 2 задание 3](./screens/3task2.PNG)

![Фото 3 задание 3](./screens/3task3.PNG)

![Фото 4 задание 3](./screens/3task4.PNG)

![Фото 5 задание 3](./screens/3task5.PNG)

Поскольку мы изменили прослушиваемые порты (listen) с 80 на 81, по итогу nginx в контейнере прослушивает только 81 порт.

![Фото 6 задание 3](./screens/3task6.PNG)

Удалить запущенный контейнер одной командой можно при помощи команды docker rm -f custom-nginx-t2 (параметр -f - force).

### Задание 4.

![Фото 1 задание 4](./screens/4task1.PNG)

![Фото 2 задание 4](./screens/4task2.PNG)

### Задание 5.

Были созданы compose.yaml

![Compose.yaml задание 5](./screens/5taskcomposeyamlbefore.PNG)

И docker-compose.yaml

![docker-compose.yaml задание 5](./screens/5taskdockercompose.PNG)

Запуск docker compose up -d:

![Фото 1 задание 5](./screens/5task1.PNG)

При запуске команды docker compose up -d docker-compose-plugin изначально ищет файлы с именем compose формата .yaml или .yml. Если он не находит, то ищет docker-compose.yaml (или .yml).

Чтобы происходил запуск двух и более .yml и .yaml нужно указать через include название второго compose-файла:

![Compose.yaml задание 5](./screens/5taskcomposeyamlafter.PNG)

![Два compose задание 5](./screens/5taskmiltimpecompose.PNG)

Далее пушим изменения:

![Фото 3 задание 5](./screens/5task3.PNG)

После заходим на страницу portainer (localhost:9432) и по пути /home/local/stacks и add stack добавляем компоуз. Затем уже по пути /docker/containers/123-custom-nginx/inspect мы видим следующее:

![Фото 4 задание 5](./screens/5task4.PNG)

При удалении compose.yaml видим:

![Фото 5 задание 5](./screens/5task5.PNG)


Вторая WARN здесь указывает на то, что созданный в уже удаленном compose.yaml контейнер task-portainer-1 стал "сиротой", иначе - в текущем состоянии он может быть не нужен, так как не указан в имеющемся docker-compose. Далее предлагается при помощи флага --remove-orphans (удалить "сирот"), т.е. запустить команду docker compose up -d --remove-orphans и контейнер-сирота task-portainer-1 удалится.

