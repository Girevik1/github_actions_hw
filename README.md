## PHP_2023  Timerkhanov A.D.
### hw21 Deploy приложения

- Использовал Github Actions так как до этого уже работал с Gitlab CI/CD
- Для работы взял свою домашку hw15
- Ключи для работы и доступа на сервер использовал Actions secrets

  ![secrets](img/secrets.png)

- мой workflow

```
# Имя флоу
name: Deploy on server hw15

# Когда действие запустится (триггеры)
on:
push:
# при push в main
branches: [ main ]

pull_request:
# при создании pull request на main
branches: [ main ]

# Что будем делать (экшены)
jobs:
# Имя действия, придумываем сами
integration-tests:
# На какой ОС будет работать виртуальная машина
# Можно выбрать Ubuntu, Windows Server или macOS
runs-on: ubuntu-latest
# Шаги действия
steps:
# Шаг 1: собираем сервисы в режиме тестирования
- uses: actions/checkout@v3
- name: Build the stack
run: docker-compose -f docker-compose.yaml up -d --build
# Шаг 2: запускаем тесты с небольшой детализацией
- name: Run tests
run: echo 'Здесь должны быть тесты!'

# Деплоим на сервер, переменные - секретные ключи заранее прописал в setting-secret-key
deploy:
needs: integration-tests
runs-on: ubuntu-latest
if: github.ref == 'refs/heads/main'
steps:
- name: Checkout code
uses: actions/checkout@v3
- name: Run command on remote server
uses: appleboy/ssh-action@master
with:
host: ${{ secrets.SERVER_HOST }}
username: ${{ secrets.SERVER_USERNAME }}
key: ${{ secrets.SERVER_KEY }}
port: ${{ secrets.SERVER_PORT }}
script: |
cd ${{ secrets.PROJECT_FOLDER }}
git checkout main
git pull
ls -la
docker-compose --file docker-compose.yaml down
docker-compose --file docker-compose.yaml up -d
docker system prune --all --force;
```
- Экшены все прошли удачно

![actions](img/actions.png)
![test](img/test.png)
![deploy](img/deploy.png)
![deploy](img/all_flow.png)

- И на сервер все выкатывается - контейнеры перезапускаются

![deploy](img/server.png)