# Лабораторная работа №3: CI/CD с использованием GitHub Actions для деплоя на GitHub Pages
## ✍🏻 Описание

В этой лабораторной работе мы создадим и настроим процесс автоматического развертывания статической веб-страницы с использованием GitHub Actions. Для этого мы будем использовать CI/CD пайплайн, который автоматически развертывает страницу на **GitHub Pages**.

### 🗓️ Задачи:
1. Создать GitHub репозиторий. (https://github.com/SofiaBerezina/Practice.git)
2. Настроить CI/CD пайплайн с использованием GitHub Actions.
3. Автоматически деплоить сайт на GitHub Pages и сделать проверку работы CI/CD файлов.

## 📝 Задание лабораторной:

1. Написать “плохой” CI/CD файл, который работает, но в нем есть не менее пяти “bad practices” по написанию CI/CD
2. Написать “хороший” CI/CD, в котором эти плохие практики исправлены
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат

## 💻 Шаг 1: Создание репозитория

1. Создадим новый репозиторий на GitHub, например, с именем `Practice`.
2. Клонируем репозиторий на локальную машину.

```bash
git clone https://github.com/SofiaBerezina/Practice.git
cd Practice
```

## ⚙️ Шаг 2: Создание и настройка файлов workflow
1. Создаем каталог .github/workflows в нашем репозитории.
   ```yaml
   mkdir -p .github/workflows
   ```
2. Внутри этого каталога создаем файлы deploy_bad.yml и deploy_good.yml с содержимым:
   Этот файл настроен для того, чтобы автоматически деплоить статический сайт на GitHub Pages каждый раз, когда изменения отправляются в ветку main.
    - actions/checkout@v4: этот шаг проверяет наш репозиторий.
    - Создание статического сайта: мы создаем папку public и генерируем простую HTML-страницу.
    - peaceiris/actions-gh-pages@v4: это действие используется для деплоя содержимого папки public на GitHub Pages.
    
    В этом примере мы создаем простой HTML файл в процессе CI/CD:
    
    ```yaml
    <html>
      <body>
        <h1>Bad Practices Deployed!</h1>
      </body>
    </html>
    ```
    
    Этот файл будет сгенерирован в папке public и развернут на GitHub Pages.
    
    После того как мы создали файл workflow и добавили код, закоммитьте изменения:
    
    ```yaml
    git add .
    git commit -m "Add GitHub Actions workflow for deployment"
    git push origin main
    ```
    После успешного выполнения CI/CD pipeline, сайт будет доступен по следующему URL:
     ```yaml
     https://your-username.github.io/Practice/
     ```
    
   "Плохой" CI/CD файл. В первой версии CI/CD файла мы допустим несколько ошибок и плохих практик, которые могут создать проблемы в будущем.

   ```yaml
    name: Deploy to GitHub Pages

    on:
      push:
        branches:
          - main
    
    jobs:
      deploy:
        runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Install dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y wget curl

    - name: Build static site
      run: |
        mkdir -p public
        echo "<html><body><h1>Bad Practices Deployed!</h1></body></html>" > public/index2.html

    - name: Deploy using git
      run: |
        git config user.name "bot"
        git config user.email "bot@example.com"
        git add .
        git commit -m "Deploy Bad Practices"
        git push origin main
   ```


### ❌ Bad practices
#### 1. Неэффективная установка зависимостей
В первой версии есть шаг установки зависимостей (wget и curl) с использованием sudo apt-get install, но для простого деплоя статического сайта эти зависимости не нужны.
- Решение: Удаление ненужных зависимостей помогает ускорить процесс и снизить нагрузку на систему. В улучшенной версии шаг по установке зависимостей был удален.
#### 2. Коммит и пуш в процессе CI/CD
В первой версии CI/CD пайплайн делает коммит и пуш после того, как генерирует файл index2.html. Это создает лишний коммит в репозитории и может вызвать рекурсивные циклы, если в процессе пуша сработает новый workflow. 
- Решение: Вместо этого следует использовать специализированные действия для деплоя (например, peaceiris/actions-gh-pages) без модификации основной ветки.
#### 3. Использование git для деплоя
В первой версии используется стандартная команда git push для деплоя, что не является лучшей практикой. 
- Решение: GitHub Actions предлагает специализированные действия для деплоя на GitHub Pages.
#### 4. Отсутствие безопасных переменных (токенов) для деплоя
В первой версии не используется GitHub токен, что может привести к проблемам с безопасностью и доступом. Без использования секретов GitHub может возникнуть ошибка доступа или безопасность данных.
- Решение: Использовать secrets.GITHUB_TOKEN в комбинации с действиями, такими как peaceiris/actions-gh-pages, для автоматического управления доступом и деплоем.
#### 5.  Генерация файла с некорректным именем
В шаге Build static site создается файл public/index2.html, который не распознается GitHub Pages в качестве главной страницы. GitHub Pages ожидает файл index.html в корне или папке деплоя. В противном случае сайт может не отобразиться.
- Решение: Генерировать файл с именем index.html для обеспечения ожидаемого поведения GitHub Pages.

### ✅ Good practices
***Исправим bad practices и напишем новый CI/CD файл***

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main

  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Build static site
      run: |
        mkdir -p public
        echo "<html><body><h1>Good Practices Deployed!</h1></body></html>" > public/index.html

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v4
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./public
```

## 💻 Шаг 3: Проверка работы файлов
 Выполним коммиты и сделаем пуш в наш репозиторий. После этого сразу запустится наш CI/CD файл и мы должны успешно пройти проверку.
 <img width="1512" alt="Снимок экрана 2024-11-20 в 01 10 25" src="https://github.com/user-attachments/assets/05a7ca68-54ee-4059-b4df-b97b92e66e6f">
 <img width="1512" alt="Снимок экрана 2024-11-20 в 01 10 36" src="https://github.com/user-attachments/assets/7c99844a-d306-4ab2-bd60-cc448a14d5c2">
 Отлично, наши файлы работают. Теперь проверим, получилось ли деплоить на pages.
 <img width="1512" alt="Снимок экрана 2024-11-20 в 01 12 22" src="https://github.com/user-attachments/assets/a7c6fe82-729c-49bf-ae8a-c1d100a6b94b">
<img width="1512" alt="Снимок экрана 2024-11-20 в 01 12 47" src="https://github.com/user-attachments/assets/a43ad984-940c-4640-9c4a-57c8a7fd6673">
<img width="1512" alt="Снимок экрана 2024-11-20 в 01 12 59" src="https://github.com/user-attachments/assets/69c5a8f9-74ac-44d0-9f42-d6aadfe22b5c">
<img width="1512" alt="Снимок экрана 2024-11-20 в 01 12 53" src="https://github.com/user-attachments/assets/f645e6fc-5076-4257-80a5-b725f880a705">
Перейдем на страницу сайта:
<img width="1512" alt="Снимок экрана 2024-11-19 в 16 26 08" src="https://github.com/user-attachments/assets/1a6a4fac-5601-4e76-85c4-9fe3a4b3b617">

Успех!

## 📊 Вывод
В ходе лабораторной работы мы создали и улучшили CI/CD файл для деплоя статического сайта на GitHub Pages. В первой версии файла мы допустили несколько ошибок и плохих практик, которые могут привести к проблемам с производительностью, безопасностью и избыточными деплоями. Во второй версии мы исправили эти ошибки, улучшив процесс и повысив эффективность работы системы.

## 🔗 Ресурсы
Этот CI/CD процесс использует GitHub Actions, peaceiris/actions-gh-pages, и стандартные практики для деплоя на GitHub Pages.

## 👩🏻‍🤝‍👩🏼 Авторы
- Машковцева Марина Алексеевна
- Березина Софья Константиновна
