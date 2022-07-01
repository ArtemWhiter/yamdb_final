# yamdb_final
![workflow status](https://github.com/ArtemWhiter/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)

# REST API для сервиса **YamDb** 
Примененные технологии: DRF, Docker, CI/CD.


### Описание
Полное описание проекта лежит на https://github.com/ArtemWhiter/api_yamdb/blob/master/api_yamdb/static/redoc.yaml
    
    Проект **YaMDb** собирает отзывы пользователей на различные произведения.

    # Алгоритм регистрации пользователей
    1. Пользователь отправляет POST-запрос на добавление нового пользователя с параметрами `email` 
    и `username` на эндпоинт `/api/v1/auth/signup/`.
    
    2. **YaMDB** отправляет письмо с кодом подтверждения (`confirmation_code`) на адрес  `email`.
    
    3. Пользователь отправляет POST-запрос с параметрами `username` и `confirmation_code` на эндпоинт 
    `/api/v1/auth/token/`, в ответе на запрос ему приходит `token` (JWT-токен).
    
    4. При желании пользователь отправляет PATCH-запрос на эндпоинт `/api/v1/users/me/` и заполняет поля 
    в своём профайле (описание полей — в документации).

### Пользовательские роли
    - **Аноним** — может просматривать описания произведений, читать отзывы и комментарии.
    
    - **Аутентифицированный пользователь** (`user`) — может, как и **Аноним**, читать всё, 
    дополнительно он может публиковать отзывы и ставить оценку произведениям (фильмам/книгам/песенкам), 
    может комментировать чужие отзывы; может редактировать и удалять **свои** отзывы и комментарии. 
    Эта роль присваивается по умолчанию каждому новому пользователю.
    
    - **Модератор** (`moderator`) — те же права, что и у **Аутентифицированного пользователя** 
    плюс право удалять **любые** отзывы и комментарии.
    
    - **Администратор** (`admin`) — полные права на управление всем контентом проекта. 
    Может создавать и удалять произведения, категории и жанры. Может назначать роли пользователям.
    
    - **Суперюзер Django** — обладет правами администратора (`admin`)


## Развертывание проекта на локальном компьютере
### Установка Docker
Установите Docker, используя инструкции с официального сайта:
- для [Windows и MacOS](https://www.docker.com/products/docker-desktop)
- для [Linux](https://docs.docker.com/engine/install/ubuntu/). Отдельно потребуется установть [Docker Compose](https://docs.docker.com/compose/install/)

### Запуск проекта (на примере Linux)

- Склонируйте этот репозиторий в текущую папку `git clone git@github.com:ArtemWhiter/yamdb_final.git .`
- В каталоге infra cоздайте файл `touch .env` и добавьте в него переменные окружения для работы с базой данных:
```
DB_NAME=postgres # имя базы данных
POSTGRES_USER=postgres # логин для подключения к базе данных
POSTGRES_PASSWORD=postgres # пароль для подключения к БД (установите свой)
DB_HOST=db # название сервиса (контейнера)
DB_PORT=5432 # порт для подключения к БД 
```
- Запустите docker-compose командой `sudo docker-compose up -d`
- Последовательно выполните команды:
```
sudo docker-compose exec web python manage.py migrate
sudo docker-compose exec web python manage.py createsuperuser
sudo docker-compose exec web python manage.py collectstatic --no-input 
```
### Деплой проекта на удаленный сервер
- Скопируйте проект к себе в git
- Создайте переменные окружения  в разделе `secrets`:
```
DOCKER_PASSWORD # Пароль от Docker Hub
DOCKER_USERNAME # Логин от Docker Hub
HOST # Публичный ip адрес сервера
USER # Пользователь зарегистрированный на сервере
PASSPHRASE # Если ssh-ключ защищен фразой-паролем
SSH_KEY # Приватный ssh-ключ
TELEGRAM_TO # ID телеграм-аккаунта
TELEGRAM_TOKEN # Токен бота
```
- Создайте переменные окружения из файла .env


### После каждого обновления репозитория, согласно настройкам из workflows/yamdb_workflow.yml будет происходить, Actions:
- tests
  Проверка кода на соответствие стандарту PEP8 (с помощью пакета flake8) и запуск pytest из репозитория yamdb_final
- Push Docker image to DockerHub
  Сборка и доставка докер-образов на Docker Hub.
- delpoy
  Автоматический деплой на боевой сервер
- send message
  Отправка уведомления в Telegram.
