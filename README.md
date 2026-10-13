# YaMDb

YaMDb — REST API для сбора отзывов на произведения (книги, фильмы, музыка). Сами произведения в проекте не хранятся, их нельзя посмотреть или почитать. Здесь можно только оставить отзыв, поставить оценку и прокомментировать чужие отзывы.

Проект упакован в Docker: PostgreSQL + Django (gunicorn) + nginx поднимаются одной командой.

## Что умеет проект

- Произведения делятся на **категории** (книги, фильмы, музыка) и **жанры**. Одно произведение может иметь несколько жанров.
- Пользователи оставляют **отзывы** и ставят оценку от 1 до 10. На одно произведение пользователь может оставить только один отзыв.
- Из оценок автоматически считается **рейтинг** произведения.
- К отзывам можно писать **комментарии**.
- Авторизация по **JWT-токену**.
- Роли пользователей: аноним (только чтение), пользователь, модератор, администратор.
- Автор может редактировать и удалять только свои отзывы и комментарии. Модератор и администратор могут редактировать и удалять любые.
- Управлять категориями, жанрами и произведениями может только администратор.

## Технологии

- Python 3
- Django, Django REST Framework
- JWT (simplejwt / djoser)
- PostgreSQL
- gunicorn, nginx
- Docker, docker-compose
- pytest


## Как запустить проект

### Вариант 1. В Docker (рекомендуется)

Нужны установленные Docker и docker-compose.

**1. Клонировать репозиторий и перейти в папку infra**

```
git clone https://github.com/<ваш_логин>/yamdb_final.git
cd yamdb_final/infra
```

**2. Создать файл `.env` в папке `infra`**

```
DB_ENGINE=django.db.backends.postgresql
DB_NAME=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
DB_HOST=db
DB_PORT=5432
```

**3. Собрать и запустить контейнеры**

```
docker-compose up -d --build
```

**4. Выполнить миграции, собрать статику, создать суперпользователя**

```
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py collectstatic --no-input
docker-compose exec web python manage.py createsuperuser
```

После этого проект доступен по адресу `http://localhost/`, админка — `http://localhost/admin/`.

Остановить контейнеры:

```
docker-compose down
```

> После любого изменения кода нужно заново выполнять `docker-compose up -d --build`, иначе контейнер продолжит работать со старой версией.

### Вариант 2. Локально, без Docker

```
git clone https://github.com/<ваш_логин>/yamdb_final.git
cd yamdb_final/api_yamdb

python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Проект будет доступен по адресу `http://127.0.0.1:8000/`.

## Как получить токен и пользоваться API
 
Регистрация и авторизация устроены в два шага: сначала регистрируемся и получаем код подтверждения, потом обмениваем его на JWT-токен.
 
**1. Зарегистрироваться**
 
`POST /api/v1/auth/signup/`
 
```json
{
    "email": "user@example.com",
    "username": "user"
}
```
 
На указанную почту придёт код подтверждения (`confirmation_code`).
 
**2. Получить токен**
 
`POST /api/v1/auth/token/`
 
```json
{
    "username": "user",
    "confirmation_code": "<код из письма>"
}
```
 
В ответ придёт JWT-токен.
 
**3. Передавать токен в заголовке запросов**
 
```
Authorization: Bearer <токен>
```
 
**4. Посмотреть и изменить свой профиль**
 
`GET /api/v1/users/me/` и `PATCH /api/v1/users/me/`
 
## Эндпоинты
 
Все адреса начинаются с `/api/v1/`. Полная документация доступна на странице `/redoc/`.
 
| Метод | Адрес | Описание |
|-------|-------|----------|
| POST | `auth/signup/` | регистрация, отправка кода подтверждения |
| POST | `auth/token/` | получение JWT-токена |
| GET, PATCH | `users/me/` | свой профиль |
| GET, POST, PATCH, DELETE | `users/` | управление пользователями (администратор) |
| GET, POST, DELETE | `categories/` | категории |
| GET, POST, DELETE | `genres/` | жанры |
| GET, POST, PATCH, DELETE | `titles/` | произведения |
| GET, POST, PATCH, DELETE | `titles/{title_id}/reviews/` | отзывы на произведение |
| GET, POST, PATCH, DELETE | `titles/{title_id}/reviews/{review_id}/comments/` | комментарии к отзыву |
 
## Тесты
 
Из корня проекта:
 
```
pytest
```