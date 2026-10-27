# Kittygram

Практическая работа 4: запуск Kittygram в Docker и GitHub Actions.
Приложение позволяет регистрироваться, добавлять котов, фотографии и достижения.

## Запуск

Нужен Docker Desktop с запущенным Docker Engine.

```powershell
Copy-Item .env.example .env
```

В `.env` замените `POSTGRES_PASSWORD` и `SECRET_KEY` своими значениями.
`SECRET_KEY` можно сгенерировать командой:

```powershell
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

Запустите контейнеры из корня проекта:

```powershell
docker compose up -d --build
docker compose logs backend
```

Бэкенд выполнит миграции и соберёт статику перед запуском Gunicorn.
Сайт: http://localhost:9000/; админка: http://localhost:9000/admin/.

```powershell
docker compose exec backend python manage.py createsuperuser
```

Для остановки:

```powershell
docker compose down
```

Данные PostgreSQL, фотографии и статика хранятся в volumes.

## Проверка

```powershell
docker compose exec backend python manage.py test cats
docker compose run --rm frontend npm test -- --watchAll=false
```

GitHub Actions проверяет бэкенд с помощью Ruff и запускает готовые тесты
бэкенда и фронтенда. После успешных тестов при пуше в `main` собирает
и публикует три образа, затем отправляет уведомление в Telegram.

В настройках репозитория GitHub откройте Settings → Secrets and variables →
Actions и добавьте:

| Secret | Значение |
| --- | --- |
| DOCKER_USERNAME | Логин Docker Hub |
| DOCKER_PASSWORD | Токен Docker Hub с правом записи |
| TELEGRAM_TOKEN | Токен Telegram-бота |
| TELEGRAM_TO | ID чата для уведомления |

На Docker Hub нужны публичные репозитории `kittygram_backend`,
`kittygram_frontend`, `kittygram_gateway`.
Токен Docker Hub создаётся в настройках аккаунта, Telegram-бот — через BotFather.

Для запуска опубликованных образов укажите `DOCKERHUB_USERNAME` в `.env`:

```powershell
docker compose -f docker-compose.production.yml pull
docker compose -f docker-compose.production.yml up -d
```

`kittygram_workflow.yml` содержит копию `.github/workflows/main.yml`.

## Тесты исходного репозитория

Тесты из `tests/` проверяют также размещение Taski и Kittygram на сервере
и наличие образов Docker Hub. В учебном задании обязательны localhost,
GitHub Actions и Docker Hub; сервер для этих тестов нужен отдельно.

Для их запуска скопируйте `tests.yml.example` в `tests.yml` и укажите
реальные адреса развёрнутых проектов и логин Docker Hub. Затем установите
Python 3.10, зависимости из `backend/requirements.txt`, переменные окружения
из `.env` и выполните `pytest` из корня проекта.

В примере адрес Taski указан для заполнения; сам Taski в этот проект не входит.
