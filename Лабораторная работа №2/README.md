
# Лабораторная работа №2

В этой лабораторной работе требуется написать хороший и плохой dockerfile.



## 🛠 Задание

1. Написать “плохой” Dockerfile, в котором есть не менее трех “bad practices” по написанию докерфайлов.
2. Написать “хороший” Dockerfile, в котором эти плохие практики исправлены.
3. В Readme описать каждую из плохих практик в плохом докерфайле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат.
4. В Readme описать 2 плохих практики по работе с контейнерами. ! Не по написанию докерфайлов, а о том, как даже используя хороший докерфайл можно накосячить именно в работе с контейнерами.


## 🚀 Шаги решения

***Все Dockerfile`ы в этой лабораторной работе мы писали в Rider от JetBrains, так как в нем уже был предустановлен плагин Docker, в отличие от того же PyCharm, где такого плагина изначально нет и установить его из нашего региона невозможно.***
- Поиски bad practises 
- Реализация bad practises в dockerfile
- Описание каждой из bad practises
- Исправление bad practises и создание "хорошего" dockerfile`а
- Описание bad practises по работе с контейнерами 


## 🔍 Поиск bad practises и создание "плохого" dockerfile`а

Для написания плохого dockerfile мы нашли 6 распространенных bad practices и постарались их воспроизвести в первом файле:
```yaml

FROM node:latest 

RUN apt-get update && apt-get install -y \
    vim \
    curl \
    git

USER root

COPY . /app

CMD ["sh", "-c", "service nginx start && node /app/app.js"]

ENV DB_PASSWORD=mysecretpassword
```
Каждая строка данного кода содержит по одной ошибке и сейчас мы разберем подробнее каждую:
#### 1.	`FROM node:latest`
#### Bad practice 1: Использование latest тега
Тег latest динамический и постоянно изменяется, что может привести к нестабильности или изменениям в контейнере при пересборке образа. В данном случае не контролируется, какая именно версия Node.js будет установлена, что в дальнейшем может привести к несоответствиям в зависимости от изменений в последнем образе.

#### 2.	`RUN apt-get update && apt-get install -y \ vim \ curl \ git`
#### Bad practice 2: Установка ненужных пакетов
Установка лишних пакетов увеличит размер образа и количество потенциальных уязвимостей контейнера. Каждый установленный пакет может содержать уязвимости, и чем меньше компонентов, тем более безопасен контейнер. В целом в контейнер обычно устанавливаются только те зависимости, которые требуются для запуска, а в данном случае vim, git и даже curl не нужны для работы, их установка – это лишняя операция.

#### 3.	`USER root`
#### Bad practice 3: Запуск приложения с правами root
Запуск контейнера с правами суперпользователя (т.е. root) может привести к проблемам безопасности данных. Если какой-то злоумышленник получит доступ к данному контейнеру, то он получит максимальные права к нему, что может привести к взлому всего хост-сервера.

#### 4.	`COPY . /app`
#### Bad practice 4: Игнорирование build контекста
Команда `COPY . /app` копирует весь контекст проекта в контейнер, в том числе ненужные файлы (временные файлы, конфигурационные файлы для IDE, папку с зависимостями), которые могут быть переустановлены в контейнере. Это увеличивает размер образа и может добавить лишние данные в контейнер. Также копирование всех файлов может случайно включить «sensitive» данные (например, .env файлы), что может стать угрозой безопасности.

#### 5.	`CMD ["sh", "-c", "service nginx start && node /app/app.js"]`
#### Bad practice 5: Запуск нескольких процессов в одном контейнере
Docker-контейнеры должны быть однозадачными – один процесс на один контейнер. Каждый контейнер должен отвечать только за один процесс, чтобы упростить управление и масштабирование.

#### 6.	`ENV DB_PASSWORD=mysecretpassword`
#### Bad practice 6: Hardcoding данных (например, паролей)
Хранение паролей, ключей API или других «sensitive» данных непосредственно в Dockerfile — это очень опасно. Dockerfile может быть опубликован в репозиторий, или его могут увидеть другие разработчики. Хранение данных внутри образа делает их доступными для всех, кто имеет доступ к контейнеру или образу. В лучшем случае пароль будет зашифрован, но в данном примере он хранится в открытом виде.


## ✅ Исправление bad practises

Теперь исправим все плохие практики, использованные в первом файле, и создадим «хороший» Dockerfile:

```yaml


FROM node:14

RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

RUN useradd -m appuser
USER appuser

WORKDIR /app

COPY package.json package-lock.json /app/

RUN npm install --production

COPY . /app

CMD ["node", "app.js"]

ENV DB_PASSWORD=${DB_PASSWORD}
```
Рассмотрим подробнее каждый шаг «хорошего» Dockerfile и объясним, как такие изменения влияют на стабильность, безопасность и производительность контейнера: 

#### 1.	`FROM node:14`
#### Best practice 1: Указание конкретной версии образа
Указание конкретной версии (в данном случае 14-ой) гарантирует, что образ всегда будет собираться с одной и той же версией Node.js. Это предотвращает проблемы с несовместимостью зависимостей и неожиданными изменениями, которые могут возникнуть при использовании динамического тега latest. Также это упростит процесс отладки и диагностики ошибок, так как если что-то пойдет не так, всегда можно будет отследить точную версию, с которой создавался образ.

#### 2.	`RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*`
#### Best practice 2: Минимизация установленных пакетов
В отличие от исходного варианта, тут мы устанавливаем только необходимый пакет curl, что значительно сокращает размер конечного образа. Также этой командой мы удаляем кэш установок, что еще больше сокращает размер образа. 

#### 3.	`RUN groupadd -r appgroup && useradd -r -g appgroup appuser`
####       `USER appuser`
#### Best practice 3: Создание и использование пользователя без привилегий
Создание нового пользователя appuser и назначение ему прав для работы приложения обеспечит ограниченный доступ к системным ресурсам, что значительно снижает потенциальные последствия взлома контейнера. А еще запуск под непривилегированным пользователем позволяет изолировать процессы, что делает контейнер более устойчивым и безопасным.

#### 4.	`COPY ./src /app`
#### Best practice 4: Копирование только нужных файлов
Вместо копирования всего проекта (COPY . /app), в хорошем Dockerfile копируем только необходимые файлы (например, файлы из директории src, которые содержат исходный код приложения). Это исключит попадание в контейнер временных файлов, конфигураций IDE и других ненужных данных и уменьшит размер образа и упростит его хранение и передачу.

#### 5.	`CMD ["node", "/app/app.js"]`
#### Best practice 5: Запуск только одного процесса
Каждый контейнер должен выполнять только одну задачу. В «хорошем» файле контейнер запускает только Node.js приложение, что упрощает управление. При запуске только одного процесса Docker легко может отслеживать его состояние и перезапускать контейнер, если процесс неожиданно завершится.

#### 6. ***Удаляем из Dockerfile:*** `ENV DB_PASSWORD=mysecretpassword`
#### Best practice 6: Удаление хардкодинга конфиденциальных данных
Вместо хардкодинга пароля в Dockerfile, лучше использовать переменные окружения на этапе запуска контейнера:

`docker run -e DB_PASSWORD=mysecretpassword myapp`

Для изоляции конфиденциальных данных лучше использовать файлы конфигурации .env и не хранить их в контексте Docker-сборки (например, исключить их через .dockerignore), а затем передавать через команду запуска или ,например, с помощью Kubernetes. 

В итоге все изменения сделали Dockerfile более безопасным, предсказуемым и удобным для дальнейшего использования в разработке и эксплуатации контейнера.
## 📦 Container's bad practises

#### 1.	Запуск контейнеров без ограничения ресурсов (CPU и память)
По умолчанию контейнеры запускаются без ограничений на использование ресурсов хост-системы. Это значит, что контейнер может потреблять неограниченное количество памяти и процессорных ресурсов, что может привести к снижению производительности всей системы. Если контейнер начнет потреблять слишком много ресурсов, другие контейнеры на той же машине могут замедлиться или перестать работать. 
Чтобы исправить эту проблему, нужно ограничивать ресурсы при запуске контейнеров. Например, чтобы ограничить использование процессора и памяти, можно указать следующее:
docker run -m 512m --cpus="1.0" myapp
где: 
- `-m 512m` ограничит контейнер до 512 MB памяти.
-  `--cpus="1.0"` указывает, что контейнер может использовать только один процессор.

Это изолирует контейнеры друг от друга и поможет избежать проблем с ресурсами.
#### 2.	Отсутствие мониторинга и логирования контейнеров
При запуске контейнеров важно следить за их состоянием и собирать логи. Это нужно, чтобы знать, как контейнеры работают, замечать проблемы до того, как они начнут влиять на приложение. 
Для мониторинга можно использовать такие инструменты, как Prometheus или Grafana, чтобы отслеживать работу контейнеров. Они помогают увидеть, как контейнеры используют ресурсы (CPU, память) и сразу узнавать, если что-то с контейнером идет не так.
Сбор логов можно делать с помощью ELK Stack или Fluentd. Логи позволяют хранить всю информацию о работе приложения и быстро находить ошибки. Это особенно важно, если контейнеров много.

## 📝 Запуск контейнеров и проверка

Соберем Docker-образы в терминале командами:
```bash
docker build -t myapp:bad -f DockerfileBadPractice .
docker build -t myapp:good -f DockerfileBestPractice .
```
<img width="468" alt="image" src="https://github.com/user-attachments/assets/cdaabcd3-72fb-4b37-be29-8a67cd2df01b">

Для Dockerfile с плохими практиками система даже выдаст предупреждение:

<img width="468" alt="image" src="https://github.com/user-attachments/assets/012b6013-dc2b-4845-9ef8-29c1d63fefdc">

У файла с хорошими практиками никаких предупреждений быть не должно:

<img width="468" alt="image" src="https://github.com/user-attachments/assets/886a44e2-8b61-43d2-9748-4ca5b8552268">

И статистика по докерам:

<img width="468" alt="image" src="https://github.com/user-attachments/assets/9a78d474-1030-457c-916b-ab32c369f86d">

<img width="413" alt="image" src="https://github.com/user-attachments/assets/ae92b970-1a8c-4788-bfe5-59adb8a7b1d1">

Получается, что докер с хорошими практиками весит меньше и NET I/O (объем данных, отправленных и полученных по сети) у него также вдвое меньше.

## 👩🏻‍🤝‍👩🏼 Авторы работы

- Березина Софья К3239
- Машковцева Марина К3240
