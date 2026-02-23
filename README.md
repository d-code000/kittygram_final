# Kittygram

Социальная сеть для любителей кошек. Пользователи могут регистрироваться, добавлять своих питомцев с фотографиями и описанием, а также просматривать котиков других пользователей.

## Функционал

- Регистрация и аутентификация пользователей
- Добавление, редактирование и удаление карточек котов
- Загрузка фотографий котиков
- Указание достижений и характеристик (цвет, год рождения)
- Просмотр списка всех котиков

## Стек технологий

- **Backend**: Django 3.2, Django REST Framework, Djoser
- **Frontend**: React
- **Database**: PostgreSQL
- **Web Server**: Nginx
- **Containerization**: Docker, Docker Compose
- **CI/CD**: GitHub Actions
- **Package Manager**: UV

## Установка и запуск

### Локальная разработка

1. Клонируйте репозиторий:
```bash
git clone git@github.com:d-code000/kittygram_final.git
cd kittygram_final
```

2. Создайте файл `.env` на основе `.env.example`:
```bash
cp .env.example .env
```

3. Заполните `.env` файл (пример):
```env
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=your_secure_password
DB_NAME=kittygram
SECRET_KEY=your_secret_key_here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
```

4. Запустите проект с помощью Docker Compose:
```bash
docker-compose up -d
```

5. Примените миграции:
```bash
docker-compose exec backend python manage.py migrate
```

6. Соберите статику:
```bash
docker-compose exec backend python manage.py collectstatic --no-input
```

7. Откройте приложение в браузере:
```
http://localhost:9000
```

### Production

1. Создайте файл `.env` с production настройками:
```env
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=your_secure_password
DB_NAME=kittygram
SECRET_KEY=your_production_secret_key
DEBUG=False
ALLOWED_HOSTS=your-domain.com,www.your-domain.com
DOCKER_USERNAME=your_dockerhub_username
```

2. Используйте production конфигурацию:
```bash
docker-compose -f docker-compose.production.yml up -d
```

3. Выполните миграции и соберите статику:
```bash
docker-compose -f docker-compose.production.yml exec backend python manage.py migrate
docker-compose -f docker-compose.production.yml exec backend python manage.py collectstatic --no-input
```

## CI/CD

Проект настроен на автоматическое тестирование и деплой с помощью GitHub Actions:

- **Тесты** запускаются при каждом пуше в любую ветку
- **Сборка и пуш образов** на Docker Hub выполняется только при пуше в `main` ветку
- Используются секреты GitHub для хранения учетных данных Docker Hub

### Требуемые секреты GitHub:

- `DOCKER_USERNAME` - имя пользователя на Docker Hub
- `DOCKER_PASSWORD` - пароль или токен Docker Hub

## Тестирование

Запуск тестов локально:

```bash
cd backend
uv sync --dev
uv run pytest ../tests/ --ignore=../tests/test_connection.py --ignore=../tests/test_dockerhub_images.py
```

## Структура проекта

```
kittygram_final/
├── backend/              # Django backend
│   ├── cats/            # Приложение для работы с котиками
│   ├── kittygram_backend/ # Настройки Django
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/            # React frontend
│   └── Dockerfile
├── nginx/               # Nginx конфигурация
│   ├── nginx.conf
│   └── Dockerfile
├── tests/               # Тесты
├── docker-compose.yml   # Конфигурация для разработки
├── docker-compose.production.yml  # Конфигурация для production
└── .github/workflows/   # GitHub Actions
```

## API

API доступен по адресу `/api/`:

- `POST /api/auth/token/login/` - получение токена
- `POST /api/auth/token/logout/` - выход
- `GET /api/cats/` - список всех котиков
- `POST /api/cats/` - создание нового котика
- `GET /api/cats/{id}/` - детальная информация о котике
- `PUT/PATCH /api/cats/{id}/` - обновление информации
- `DELETE /api/cats/{id}/` - удаление котика

## Автор

Дмитрий (d-code000)

## Лицензия

MIT License
