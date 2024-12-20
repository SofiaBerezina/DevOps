
# Лабораторная работа №2*

В этой лабораторной работе требуется написать хороший и плохой docker compose file.



## 🛠 Задание

1. Написать “плохой” Docker compose file, в котором есть не менее трех “bad practices” по написанию докерфайлов.
2. Написать “хороший” Docker compose file, в котором эти плохие практики исправлены.
3. В Readme описать каждую из плохих практик в плохом докерфайле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат.
4. После предыдущих пунктов в хорошем файле настроить сервисы так, чтобы контейнеры в рамках этого compose-проекта так же поднимались вместе, но не "видели" друг друга по сети. В отчете описать, как этого добились и кратко объяснить принцип такой изоляции

## 🚀 Шаги решения

***Все Dockerfile`ы в этой лабораторной работе мы писали в Rider от JetBrains, так как в нем уже был предустановлен плагин Docker, в отличие от того же PyCharm, где такого плагина изначально нет и установить его из нашего региона невозможно.***
- Поиски bad practises 
- Реализация bad practises в docker compose file
- Описание каждой из bad practises
- Исправление bad practises и создание "хорошего" docker compose file`а
- Настройка изоляции сервисов 


## 🔍 Поиск bad practises и создание "плохого" docker compose file`а

Для написания плохого docker compose file мы нашли 3 распространенных bad practices и постарались их воспроизвести в первом файле:
```yaml

version: '3.8'

services:
  app:
    image: my-app:latest
    ports:
      - "8080:80"
    environment:
      - DATABASE_URL=mongodb://db:27017/mydb
    volumes:
      - ./app:/usr/src/app
  db:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - ./data:/data/db
  redis:
    image: redis:latest
    ports:
      - "6379:6379"
```
Описание bad practises:

#### 1.	Bad practice 1: Использование latest тега
```yaml
image: mongo:latest
image: redis:latest
```
При использовании тега latest Docker будет всегда скачивать самую последнюю версию образа. Это опасно, потому что новые версии могут содержать несовместимые изменения, что приведет к сбоям или некорректной работе приложения. Это также затрудняет воспроизводимость сборок.

#### 2.	Bad practice 2: Открытые порты для всех сервисов
```yaml

ports:
  - "27017:27017"
ports:
  - "6379:6379"
``` 
Проброс портов для сервисов db и redis наружу (на хостовую машину) позволяет внешним пользователям обращаться напрямую к базе данных и кэш-серверу. Это потенциальная уязвимость, так как несанкционированный доступ к данным или манипуляции с данными может привести к сбоям в работе приложения.

#### 3. Bad practice 3: Запуск приложения с правами root
```yaml
volumes:
  - ./data:/data/db
```
Прямое подключение локальных директорий к контейнерам затрудняет переносимость. Если контейнер запускается на другой машине, локальные файлы могут не совпадать, что приведет к ошибкам или потерям данных.

## ✅ Исправление bad practises

Теперь исправим все плохие практики, использованные в первом файле, и создадим «хороший» Docker compose file:

```yaml
version: '3.8'

services:
  app:
    image: my-app:1.0.0
    ports:
      - "8080:80"
    environment:
      - DATABASE_URL=mongodb://db:27017/mydb
    volumes:
      - ./app:/usr/src/app
    depends_on:
      - db
      - redis
  db:
    image: mongo:5.0
    volumes:
      - db-data:/data/db
  redis:
    image: redis:6.2
    volumes:
      - redis-data:/data

volumes:
  db-data:
  redis-data:
```
Рассмотрим подробнее каждый шаг «хорошего» Docker compose file и объясним, как такие изменения влияют на стабильность, безопасность и производительность контейнера: 
#### 1. Указание фиксированных версий образов
```yaml
# Плохая практика
image: mongo:latest

# Исправленная практика
image: mongo:5.0
```
Вместо latest, в “хорошем” Docker Compose файле указаны конкретные версии (mongo:5.0, redis:6.2). Это позволяет контролировать используемые версии и избегать неожиданных проблем при обновлении образов. Приложение будет всегда развернуто в ожидаемом окружении.

#### 2. Изменение привязки портов
```yaml
# Плохая практика
db:
  ports:
    - "27017:27017"
```
Убрали проброс портов для db и redis. Теперь эти сервисы доступны только внутри Docker Compose сети. Это повышает безопасность, так как предотвращает доступ к этим сервисам снаружи хоста.

#### 3. Использование именованных volume’ов
```yaml
# Плохая практика
volumes:
  - ./data:/data/db

# Исправленная практика
volumes:
  - db-data:/data/db
```
В “хорошем” файле применены именованные volume’ы (db-data, redis-data). Это позволяет Docker управлять данными контейнеров. Именованные volume’ы проще в переноске, резервном копировании и восстановлении, что делает работу с данными более удобной и безопасной.

## 🗳️ Подведение итогов

#### 1. Удаление проброса портов для базы данных и Redis

До исправления:
- В “плохом” файле порты 27017 (MongoDB) и 6379 (Redis) проброшены на хостовую машину, что позволяло любому пользователю, имеющему доступ к хосту, напрямую подключиться к базе данных и кэшу.
- Это создавало серьёзный риск безопасности, так как любые сторонние подключения могли получить доступ к важным данным без аутентификации или ограничений.

После исправления:
- В “хорошем” файле порты для db и redis не проброшены наружу. Эти сервисы доступны только через внутренние сети Docker, и к ним может подключиться только приложение app.
- Это улучшило безопасность, поскольку теперь базы данных защищены от нежелательных внешних подключений.

#### 2. Использование фиксированных версий образов вместо latest

До исправления:
- В “плохом” файле использовались теги latest для всех образов. Это могло привести к тому, что Docker загружал самую свежую версию образа при каждом запуске, что делало окружение непредсказуемым.
- Если новая версия образа содержала изменения, несовместимые с приложением, это могло вызвать сбои в работе.

После исправления:
- В “хорошем” файле указаны конкретные версии образов (mongo:5.0, redis:6.2, my-app:1.0.0). Это гарантирует, что используются проверенные и стабильные версии образов.
- Исправление увеличило стабильность системы, так как теперь обновления Docker-образов не могут неожиданно нарушить работу приложения. Система будет работать одинаково на разных машинах и при каждом запуске.

#### 3. Замена локальных директорий на именованные volume’ы

До исправления:
- В “плохом” файле использовались локальные директории для хранения данных MongoDB (./data) и приложения (./app). Это создавало проблемы с переносимостью и зависимостью от конкретного пути на хосте.
- Данные, хранящиеся в локальной папке, могли случайно быть изменены или удалены, что привело бы к потерям.

После исправления:
- В “хорошем” файле используются именованные volume’ы (db-data, redis-data), которые автоматически создаются и управляются Docker. Они не зависят от конкретного местоположения на хостовой машине.
- Это улучшило переносимость, поскольку данные легко сохраняются и восстанавливаются между разными развёртываниями, а также упрощает управление данными.

## 🛠️ Настройка изоляции сервисов

```yaml
version: '3.8'

services:
  app:
    image: my-app:1.0.0
    ports:
      - "8080:80"
    environment:
      - DATABASE_URL=mongodb://db:27017/mydb
    volumes:
      - ./app:/usr/src/app
    depends_on:
      - db
      - redis
    networks:
      - app-net

  db:
    image: mongo:5.0
    volumes:
      - db-data:/data/db
    networks:
      - db-net

  redis:
    image: redis:6.2
    volumes:
      - redis-data:/data
    networks:
      - redis-net

volumes:
  db-data:
  redis-data:

networks:
  app-net:
  db-net:
  redis-net:
```
#### Создание отдельных сетей для каждого сервиса

В новом Docker Compose файле для каждого сервиса (app, db, redis) созданы отдельные сети:

- app подключён к сети app-net
- db подключён к сети db-net
- redis подключён к сети redis-net

#### Что делает этот подход?

Каждая из сетей (app-net, db-net, redis-net) представляет собой отдельное изолированное пространство в Docker. Контейнеры, подключённые к разным сетям, не могут общаться друг с другом напрямую, если они не добавлены в одну и ту же сеть.

#### Как это работает:

- Docker создает внутренние сети (виртуальные локальные сети), и каждый контейнер видит только те контейнеры, которые подключены к той же сети.
- Если контейнер app находится в сети app-net, он не сможет связаться с контейнерами db и redis, так как те находятся в db-net и redis-net соответственно.
- Даже если контейнеры запущены в одном проекте и на одной хостовой машине, сетевое взаимодействие между ними невозможно, пока они не будут подключены к общей сети.

## 🔐 Проверка изоляции сервисов

Чтобы проверить, что наши сервисы действительно изолированы, выведем ID их контейнеров и проверим, смогут ли они общаться:

```bash
docker ps
```
<img width="1208" alt="Снимок экрана 2024-10-21 в 00 30 42" src="https://github.com/user-attachments/assets/f769a861-e3c9-4be4-998b-1213da2fc5a1">

<img width="1512" alt="Снимок экрана 2024-10-21 в 00 22 56" src="https://github.com/user-attachments/assets/40b577e4-b92d-4400-96c3-91def45ef8bb">

Затем выведем все сети, которые есть в контейнере, чтобы убедиться, что они вообще существуют:

```bash
docker network ls
```
![image](https://github.com/user-attachments/assets/6e61750c-cb34-4248-93b0-e2b054f6567e)

Отлично, наши сети создались. Теперь проверим, могут ли контейнеры общаться между собой. Для этого заходим в любой контейнер и пробуем связаться с другим:

```bash
docker exec -it 684f68d8f00a /bin/sh
ping db-1
```
<img width="860" alt="Снимок экрана 2024-10-21 в 00 30 23" src="https://github.com/user-attachments/assets/49527c19-85a5-4f89-a68c-3a272213e346">

В ответ мы получаем "не найдено". Значит все работает и нам удалось изолировать сервисы!
#### Принцип изоляции контейнеров

В Docker изоляция сети обеспечивается с помощью сетевых пространств (network namespaces). Когда создаётся сеть, Docker создает виртуальный маршрутизатор для управления трафиком внутри этой сети. Контейнеры, подключённые к одной сети, могут общаться друг с другом через этот маршрутизатор, но другие сети являются изолированными и не могут обмениваться данными между собой.

Принцип работы:
1.	*Отдельные сети:* Каждый сервис получает свою собственную сеть, что означает, что они физически не могут отправлять пакеты друг другу, даже если они работают на одном хосте. Это эквивалентно тому, как если бы эти сервисы были на разных серверах.
2.	*Контроль доступа:* Docker по умолчанию запрещает пересечение сетевых пространств. Чтобы контейнеры могли “видеть” друг друга, они должны быть явно подключены к общей сети, что здесь не происходит.
3.	*Безопасность и конфиденциальность:* Поскольку контейнеры изолированы, злоумышленники не смогут использовать сеть для атаки на другие контейнеры в проекте.

Результат:
1.	*Улучшенная безопасность:*
Изоляция сетей помогает избежать несанкционированного доступа к сервисам и снижает риск атак. Даже если кто-то получит доступ к одному из контейнеров, он не сможет напрямую взаимодействовать с другими.
2.	*Контроль взаимодействий:*
Это позволяет более точно настроить взаимодействие контейнеров. Если нужно, чтобы два сервиса взаимодействовали, их можно подключить к общей сети, но без необходимости изменять основную структуру Compose файла.
3.	*Упрощенное управление:*
Поддержка и отладка становятся проще, так как можно быть уверенным, что сеть одного контейнера не повлияет на работу другого, и их можно тестировать и запускать независимо.

Таким образом, изоляция сетей улучшает безопасность и гибкость управления контейнерами, создавая более надёжную и стабильную инфраструктуру.
## 👩🏻‍🤝‍👩🏼 Авторы работы

- Березина Софья К3239
- Машковцева Марина К3240
