# Auth Module API with YandexMail on Python (FastAPI)

## 🧐 Что это и зачем?

Данный модуль был написан как часть некоторой системы, однако я решил, что его можно вынести отдельно для пользования еще где-либо.

> да, легче использовать готовые открытые библиотеки с авторизацией типа oAuth, но был интерес проработать вручную, с яндекс почтой и своим алгоритмом работы.

---
## 🎨 Стек

- FastAPI, pydantic

- SQLAlchemy

- Alembic

- PostgreSQL

- Docker, docker-compose

---

## 📡 Настройка переменных окружения

Проект использует **два** файла конфигурации. Реальные `.env`-файлы не попадают в
репозиторий — вместо них в git лежат шаблоны `.env.example`. Скопируйте их и заполните
своими значениями.

| Файл | Назначение |
|------|------------|
| `.env.docker` | поднимает PostgreSQL (логин/пароль/имя БД для `docker-compose`) |
| `src/.env` | читает приложение и Alembic: DSN, JWT-секреты, SMTP для рассылки |

1. Скопируйте шаблоны:
   ```bash
   cp .env.docker.example .env.docker
   cp src/.env.example src/.env
   ```

2. Отредактируйте оба файла: задайте логин/пароль/имя БД, сгенерируйте JWT-секреты
   (`openssl rand -hex 32`) и укажите почту с паролем приложения для SMTP.

   > ⚠️ Пароль БД продублирован в двух файлах (в `.env.docker` и внутри DSN в
   > `src/.env`). При смене пароля обновляйте **оба**, иначе приложение или база
   > поднимутся с разными значениями.

   > В `src/.env` два DSN: `APP_CONFIG__DB__URL` использует хост `db` и порт `5432`
   > (для запуска внутри Docker), а `APP_CONFIG__DB__URL_MIGRATIONS` — `localhost:5433`
   > (для запуска миграций с хоста через venv). Не путайте их.

## 🛠 Быстрый старт

1. Склонируйте репозиторий и перейдите в папку проекта:

```bash 
   git clone https://github.com/coddemn/auth_module_with_YandexMail.git

   cd asao_main_api

```

2. Создайте файл `.env.docker` на основе шаблона:

```bash

   cp .env.docker.template .env.docker

```

   При необходимости отредактируйте значения `DB_USER`, `DB_PASS`, `DB_NAME`.

3. Запустите проект:

```bash

   docker-compose --env-file .env.docker up --build

```

   Дождитесь сообщения `Application startup complete.`

4. Примените миграции базы данных (в отдельном терминале):

```bash

   docker exec -it asao_api bash -c "cd src && alembic upgrade head"

```

5. Откройте приложение в браузере:

   - API: http://localhost:8000

   - Swagger: http://localhost:8000/docs

### ❗ Если вы изменили `.env.docker` после первой сборки контейнера

Docker Compose не перечитывает env-файл у уже запущенного контейнера. После изменения `.env.docker` пересоздайте сервис `api`:

```bash

docker-compose --env-file .env.docker up -d --force-recreate api

```

Данные PostgreSQL при этом сохраняются в named volume `pgdata`.

---
# 📝 Дополнительно

#### Подробная документация с пошаговым алгоритмом работы для тестирования в Swagger описана в файле `AuthAPI_Doc.pdf`
