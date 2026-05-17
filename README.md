<div align="center">

# AuctionLive

**Высоконагруженная real-time платформа онлайн-аукционов —
живые ставки, авто-биддинг, WebSocket-стриминг и эскроу-платежи в одной экосистеме.**

![Python](https://img.shields.io/badge/Python-3.12%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-5.x-092E20?style=for-the-badge&logo=django&logoColor=white)
![DRF](https://img.shields.io/badge/DRF-3.15-A30000?style=for-the-badge&logo=django&logoColor=white)
![Channels](https://img.shields.io/badge/Channels-4.2-44B78B?style=for-the-badge&logo=django&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Redux](https://img.shields.io/badge/Redux_Toolkit-2.5-764ABC?style=for-the-badge&logo=redux&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Celery](https://img.shields.io/badge/Celery-5.4-37814A?style=for-the-badge&logo=celery&logoColor=white)
![Daphne](https://img.shields.io/badge/Daphne-ASGI-009688?style=for-the-badge)
![Stripe](https://img.shields.io/badge/Stripe-Payments-635BFF?style=for-the-badge&logo=stripe&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-Reverse_Proxy-009639?style=for-the-badge&logo=nginx&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-FFCB05?style=for-the-badge)

</div>

---

## Содержание

1. [О проекте](#1-о-проекте)
2. [Ключевые возможности](#2-ключевые-возможности)
3. [Технологический стек](#3-технологический-стек)
4. [Структура репозитория](#4-структура-репозитория)
5. [Архитектура и как это работает](#5-архитектура-и-как-это-работает)
6. [Доменная модель (крупными блоками)](#6-доменная-модель-крупными-блоками)
7. [Сервисы в Docker Compose](#7-сервисы-в-docker-compose)
8. [Быстрый старт (локально, Docker)](#8-быстрый-старт-локально-docker)
9. [Основные команды Makefile](#9-основные-команды-makefile)
10. [Ручной запуск frontend и backend](#10-ручной-запуск-frontend-и-backend)
11. [Конфигурация и переменные окружения](#11-конфигурация-и-переменные-окружения)
12. [API, WebSocket и интеграции](#12-api-websocket-и-интеграции)
13. [Мониторинг и эксплуатация](#13-мониторинг-и-эксплуатация)
14. [CI/CD](#14-cicd)
15. [Безопасность и платёжный контур](#15-безопасность-и-платёжный-контур)
16. [Роли компонентов в продакшене](#16-роли-компонентов-в-продакшене)
17. [Лицензия](#17-лицензия)
18. [Поддержка](#18-поддержка)

---

## 1. О проекте

**AuctionLive** — это **продуктовая SaaS-платформа онлайн-аукционов** в реальном времени: лоты, живые ставки по WebSocket, синхронизированные таймеры обратного отсчёта, авто-биддинг, watchlist, эскроу и платежи через Stripe — всё в одном окне. Система рассчитана на конечных пользователей (веб-интерфейс), продавцов (личный кабинет и аналитика) и интеграторов (REST API + WebSocket API), с возможностью эксплуатации в Docker — от локальной разработки до production.

### Что это за тип системы

По архитектуре AuctionLive — **многосервисная распределённая платформа** (не монолит «в одном процессе»):

| Аспект         | Описание                                                                              |
| -------------- | ------------------------------------------------------------------------------------- |
| **Продукт**    | B2C/C2C-сервис онлайн-аукционов со ставками, авто-биддингом, эскроу и аналитикой     |
| **Архитектура**| Многосервисная: Django REST API, Django Channels (ASGI), React SPA, Celery воркеры    |
| **Хранилище**  | PostgreSQL (метаданные) + Media-volume (изображения лотов) + Redis (кэш, брокер, pub/sub) |
| **Real-time**  | Двунаправленный WebSocket поверх Daphne + Redis Channel Layer                         |
| **Платежи**    | Stripe (карты, эскроу-удержание, webhook-подтверждения)                              |
| **Деплой**     | Docker Compose, Nginx как reverse proxy, ASGI-сервер Daphne                          |

### Кому подойдёт

* **Маркетплейсам** — готовый движок аукционов поверх существующего каталога
* **Аукционным домам** — цифровизация торгов с прозрачной историей ставок
* **B2B-площадкам** — закупочные аукционы и тендеры с авто-биддингом
* **Стартапам** — стартовая база (boilerplate) для запуска MVP за дни, а не месяцы

---

## 2. Ключевые возможности

### Торги и ставки
* **Живые ставки в реальном времени** — WebSocket-канал на лот, мгновенное обновление цены у всех подключённых клиентов
* **Авто-биддинг (Proxy Bidding)** — система автоматически повышает ставку пользователя до его максимального лимита
* **Синхронизированный таймер** — серверный отсчёт времени единый для всех клиентов, защита от рассинхронизации
* **Anti-snipe (продление аукциона)** — автоматическое продление при ставке в последние секунды
* **Полная история ставок** — аудит-трейл с timestamp, IP, user-agent для каждой ставки

### Лоты и каталог
* **Иерархические категории** — древовидная структура с фильтрацией и поиском
* **Мультизагрузка изображений** — несколько фото на лот с превью и lightbox-просмотром
* **Состояния лота** — `draft`, `scheduled`, `live`, `ended`, `cancelled`, `paid`
* **Резервная цена и шаг ставки** — кастомизация по каждому лоту

### Пользователи и роли
* **JWT-аутентификация** — access + refresh токены, безопасный re-login
* **Ролевая модель** — покупатель (bidder), продавец (seller), администратор
* **Личный кабинет продавца** — управление лотами, аналитика, выручка, выполненные сделки
* **Watchlist** — избранные аукционы с push-уведомлениями о действиях

### Платежи и доставка
* **Stripe-интеграция** — оплата картой, 3D-Secure, webhook-обработка
* **Эскроу-удержание** — деньги резервируются до подтверждения получения товара
* **Возвраты и диспуты** — workflow для спорных сделок

### Уведомления
* **Multi-channel** — email (Celery + SMTP), push в браузер, in-app уведомления
* **События** — outbid, auction-ending-soon (5/1 мин), auction-won, payment-received

### Эксплуатация
* **Health-checks** для PostgreSQL и Redis
* **Background tasks** — Celery worker + Celery Beat для расписаний (закрытие аукционов, рассылки)
* **Graceful migrations** при старте контейнера

---

## 3. Технологический стек

| Слой              | Технология                                                       |
| ----------------- | ---------------------------------------------------------------- |
| **Backend**       | Python 3.12+, Django 5.1, Django REST Framework 3.15             |
| **Real-time**     | Django Channels 4.2, channels-redis, Daphne ASGI                 |
| **Async tasks**   | Celery 5.4, django-celery-beat, django-celery-results            |
| **Frontend**      | React 18, Redux Toolkit 2.5, React Router 7, Axios               |
| **UI/UX**         | react-icons, react-toastify (нотификации)                        |
| **База данных**   | PostgreSQL 16                                                    |
| **Кэш / Broker**  | Redis 7 (кэш, Celery broker, Channels layer — три разных DB)     |
| **Auth**          | djangorestframework-simplejwt (Access + Refresh)                 |
| **Изображения**   | Pillow                                                           |
| **Платежи**       | Stripe SDK 11.x, webhook-подпись                                 |
| **Reverse proxy** | Nginx 1.25                                                       |
| **WSGI/ASGI**     | Daphne (production ASGI server) + gunicorn (fallback WSGI)       |
| **CORS & Filter** | django-cors-headers, django-filter                               |
| **Контейнеры**    | Docker, Docker Compose v2                                        |
| **Утилиты**       | django-environ, python-dateutil, whitenoise                      |

---

## 4. Структура репозитория

```
AuctionLive/
├── backend/                        # Django + DRF + Channels
│   ├── apps/                       # Бизнес-приложения (bounded contexts)
│   │   ├── accounts/               # Пользователи, профили, JWT
│   │   ├── auctions/               # Лоты, категории, seller dashboard
│   │   ├── bids/                   # Ставки, авто-биддинг, история
│   │   ├── watchlist/              # Избранное пользователя
│   │   ├── payments/               # Stripe, эскроу, webhooks
│   │   └── notifications/          # Email / push / in-app оповещения
│   ├── config/                     # Конфигурация проекта
│   │   ├── settings/               # base / development / production
│   │   ├── asgi.py                 # ASGI entrypoint (Daphne)
│   │   ├── wsgi.py                 # WSGI entrypoint (gunicorn fallback)
│   │   ├── routing.py              # WebSocket URL routing (Channels)
│   │   ├── celery.py               # Celery app конфигурация
│   │   └── urls.py                 # HTTP-роутинг REST API
│   ├── utils/                      # Общие утилиты (пагинация, исключения)
│   ├── manage.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── frontend/                       # React SPA
│   ├── public/                     # Статика, favicon, index.html
│   ├── src/
│   │   ├── api/                    # Axios-клиент, базовые запросы
│   │   ├── components/             # Переиспользуемые компоненты
│   │   ├── pages/                  # Страницы (Home, Auction, Profile, ...)
│   │   ├── hooks/                  # useWebSocket, useAuth, useCountdown
│   │   ├── services/               # WS-клиент, Stripe-обвязка
│   │   ├── store/                  # Redux Toolkit slices
│   │   ├── styles/                 # SCSS / CSS-модули
│   │   ├── App.jsx
│   │   └── index.js
│   ├── package.json
│   └── Dockerfile
│
├── nginx/
│   └── nginx.conf                  # Reverse proxy: HTTP → Daphne, WS → Daphne
│
├── docker-compose.yml              # Оркестрация всех сервисов
├── .env.example                    # Шаблон переменных окружения
├── .gitignore
└── README.md                       # Этот файл
```

---

## 5. Архитектура и как это работает

AuctionLive построен по принципу **separation of concerns**: HTTP-трафик и WebSocket-трафик обрабатываются единым ASGI-приложением (Daphne), но идут по разным путям. Долгие задачи вынесены в Celery, real-time-стриминг — в Channels с Redis-бэкендом.

### Высокоуровневая диаграмма

```
                    ┌───────────────────────────────────────────┐
                    │          Браузер (React SPA)              │
                    │   ┌─────────────┐   ┌─────────────────┐   │
                    │   │  Axios HTTP │   │  WebSocket API  │   │
                    │   └──────┬──────┘   └────────┬────────┘   │
                    └──────────┼───────────────────┼────────────┘
                               │                   │
                          HTTP/HTTPS          WS / WSS
                               │                   │
                    ┌──────────▼───────────────────▼────────────┐
                    │           Nginx (reverse proxy)           │
                    │     /api → HTTP    /ws → WebSocket        │
                    └──────────┬───────────────────┬────────────┘
                               │                   │
                    ┌──────────▼───────────────────▼────────────┐
                    │         Daphne (ASGI, port 8000)          │
                    │   ┌──────────────┐   ┌────────────────┐   │
                    │   │ Django + DRF │   │ Django Channels│   │
                    │   │  (HTTP API)  │   │  (consumers)   │   │
                    │   └──────┬───────┘   └────────┬───────┘   │
                    └──────────┼───────────────────┬┘            
                               │                   │            
              ┌────────────────┼───────────────────┼─────────────┐
              │                │                   │             │
        ┌─────▼──────┐   ┌─────▼──────┐    ┌──────▼──────┐  ┌───▼────┐
        │ PostgreSQL │   │   Redis    │    │   Redis     │  │ Stripe │
        │ (метадата) │   │ (cache+    │    │ (Channels   │  │  API   │
        │            │   │  broker)   │    │  Layer)     │  │        │
        └─────▲──────┘   └─────▲──────┘    └─────────────┘  └────────┘
              │                │
              │                │
        ┌─────┴────────────────┴─────┐
        │      Celery Worker         │
        │  • закрытие аукционов      │
        │  • email/push рассылки     │
        │  • Stripe webhooks         │
        │  • генерация отчётов       │
        └────────────────────────────┘
                     ▲
                     │ расписания
                     │
        ┌────────────┴───────────────┐
        │       Celery Beat          │
        │  • cron-задачи             │
        │  • периодические события   │
        └────────────────────────────┘
```

### Сценарий «живая ставка» end-to-end

1. **Покупатель** открывает страницу лота → React SPA по HTTP получает текущее состояние через `/api/auctions/{id}/`
2. SPA открывает **WebSocket** `wss://.../ws/auction/{id}/` → Nginx → Daphne → Channels Consumer
3. Consumer **подписывает** клиента на группу `auction_{id}` через Redis Channel Layer
4. Пользователь нажимает «Bid» → SPA шлёт `{"type": "place_bid", "amount": "150.00"}`
5. Consumer **валидирует** (ставка выше текущей + шаг, аукцион активен, у юзера есть баланс)
6. Запись `Bid` сохраняется в PostgreSQL атомарно (`SELECT ... FOR UPDATE`)
7. Consumer **broadcast-ит** новую цену всем участникам группы `auction_{id}` → у всех экранов мгновенно меняется значение
8. Параллельно Celery-таска **рассылает** email-уведомления `outbid` всем, кого только что перебили
9. По истечении таймера **Celery Beat** триггерит закрытие лота, определяет победителя, инициирует Stripe-эскроу

---

## 6. Доменная модель (крупными блоками)

| Контекст           | Ключевые сущности                                | Зона ответственности                                    |
| ------------------ | ------------------------------------------------ | ------------------------------------------------------- |
| **accounts**       | `User`, `Profile`, `SellerInfo`                  | Регистрация, JWT, профили, верификация продавцов        |
| **auctions**       | `Category`, `Auction`, `AuctionImage`            | Каталог лотов, состояния, медиа, поиск, фильтры         |
| **bids**           | `Bid`, `AutoBid`                                 | Ручные и авто-ставки, история, валидация шага           |
| **watchlist**      | `WatchlistItem`                                  | Избранные аукционы пользователя                         |
| **payments**       | `Payment`, `EscrowTransaction`, `StripeWebhook`  | Платежи, удержания, возвраты, webhook-обработка         |
| **notifications**  | `Notification`, `NotificationPreference`         | Email/push/in-app каналы, шаблоны, юзер-настройки       |

### Состояния аукциона (state machine)

```
   draft ─────► scheduled ─────► live ─────► ended ─────► paid
     │              │              │           │
     │              └──► cancelled │           └──► refunded
     └──► deleted                  └──► extended (anti-snipe)
```

---

## 7. Сервисы в Docker Compose

| Сервис         | Образ / Build                | Порт   | Назначение                                        |
| -------------- | ---------------------------- | ------ | ------------------------------------------------- |
| `db`           | `postgres:16-alpine`         | `5432` | Реляционное хранилище метаданных                  |
| `redis`        | `redis:7-alpine`             | `6379` | Кэш + Celery broker + Channels Layer (3 БД)       |
| `backend`      | `./backend` (Daphne ASGI)    | `8000` | Django + DRF + Channels, миграции при старте      |
| `celery`       | `./backend` (worker)         | —      | Фоновые задачи, рассылки, Stripe webhooks         |
| `celery-beat`  | `./backend` (beat scheduler) | —      | Cron-расписания, закрытие аукционов               |
| `frontend`     | `./frontend` (React dev)     | `3000` | React SPA (в dev — hot reload, в prod — статика)  |
| `nginx`        | `nginx:1.25-alpine`          | `80`   | Reverse proxy: HTTP API, WebSocket, статика       |

Volumes: `postgres_data`, `redis_data`, `static_volume`, `media_volume`.

---

## 8. Быстрый старт (локально, Docker)

### Предварительные требования
* **Docker** 24+ и **Docker Compose v2**
* **Git**
* 4 ГБ свободной RAM, 2 ГБ места на диске

### Шаг 1. Клонировать репозиторий

```bash
git clone https://github.com/NodirOdilov/AuctionLive.git
cd AuctionLive
```

### Шаг 2. Создать `.env` из шаблона

```bash
cp .env.example .env
# Откройте .env и заполните DJANGO_SECRET_KEY, STRIPE_*, EMAIL_* при необходимости
```

### Шаг 3. Собрать и запустить весь стек

```bash
docker compose up --build -d
```

### Шаг 4. Применить миграции (выполняется автоматически при старте, но можно повторить)

```bash
docker compose exec backend python manage.py migrate
```

### Шаг 5. Создать суперпользователя

```bash
docker compose exec backend python manage.py createsuperuser
```

### Шаг 6. Загрузить демо-данные (опционально)

```bash
docker compose exec backend python manage.py loaddata fixtures/demo.json
```

### Шаг 7. Открыть в браузере

| Точка входа      | URL                                            |
| ---------------- | ---------------------------------------------- |
| Frontend (SPA)   | <http://localhost>                             |
| REST API         | <http://localhost/api/>                        |
| Django Admin     | <http://localhost/api/admin/>                  |
| WebSocket        | `ws://localhost/ws/auction/{auction_id}/`      |
| Backend (прямой) | <http://localhost:8000>                        |
| Frontend (dev)   | <http://localhost:3000>                        |

---

## 9. Основные команды Makefile

> Если в проекте присутствует `Makefile`, используйте короткие алиасы. Если нет — все команды эквивалентны вызовам `docker compose`.

| Команда            | Эквивалент                                                | Что делает                            |
| ------------------ | --------------------------------------------------------- | ------------------------------------- |
| `make up`          | `docker compose up -d`                                    | Поднять весь стек в фоне              |
| `make down`        | `docker compose down`                                     | Остановить и удалить контейнеры       |
| `make build`       | `docker compose build`                                    | Пересобрать образы                    |
| `make logs`        | `docker compose logs -f`                                  | Стрим логов всех сервисов             |
| `make migrate`     | `docker compose exec backend python manage.py migrate`    | Применить миграции БД                 |
| `make superuser`   | `docker compose exec backend python manage.py createsuperuser` | Создать админа                  |
| `make shell`       | `docker compose exec backend python manage.py shell`      | Django shell                          |
| `make test`        | `docker compose exec backend python manage.py test`       | Прогнать тесты                        |
| `make celery-logs` | `docker compose logs -f celery`                           | Логи Celery worker                    |
| `make psql`        | `docker compose exec db psql -U auctionlive`              | Зайти в psql                          |
| `make redis-cli`   | `docker compose exec redis redis-cli`                     | Зайти в redis-cli                     |
| `make clean`       | `docker compose down -v`                                  | Удалить **и тома** (sic!) — снести БД |

---

## 10. Ручной запуск frontend и backend

Если нужно запускать сервисы **без Docker** (например, для отладки или работы из IDE):

### Backend (Django + Daphne)

```bash
cd backend
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Запустить PostgreSQL и Redis отдельно (нативно или в Docker)

python manage.py migrate
python manage.py createsuperuser

# HTTP + WebSocket через ASGI
daphne -b 0.0.0.0 -p 8000 config.asgi:application

# В отдельном терминале — Celery
celery -A config worker -l info --concurrency=4
celery -A config beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```

### Frontend (React)

```bash
cd frontend
npm install
npm start
# SPA на http://localhost:3000, проксирует API на http://localhost:8000
```

---

## 11. Конфигурация и переменные окружения

Все переменные хранятся в `.env` (см. `.env.example`). Ключевые группы:

### Django
| Переменная                | Назначение                                      |
| ------------------------- | ----------------------------------------------- |
| `DJANGO_SECRET_KEY`       | Криптографический ключ (обязательно сменить!)   |
| `DJANGO_DEBUG`            | `True` — dev, `False` — production              |
| `DJANGO_ALLOWED_HOSTS`    | Список разрешённых хостов через запятую         |
| `DJANGO_SETTINGS_MODULE`  | `config.settings.development` / `production`    |

### PostgreSQL
| Переменная           | Назначение                                |
| -------------------- | ----------------------------------------- |
| `POSTGRES_DB`        | Имя базы                                  |
| `POSTGRES_USER`      | Пользователь                              |
| `POSTGRES_PASSWORD`  | Пароль (в проде — из секрет-стора)        |
| `POSTGRES_HOST`      | Хост (в Docker — имя сервиса `db`)        |
| `POSTGRES_PORT`      | Порт (`5432`)                             |

### Redis (три логические БД)
| Переменная                | Назначение                            |
| ------------------------- | ------------------------------------- |
| `REDIS_URL`               | Кэш Django (`/0`)                     |
| `CELERY_BROKER_URL`       | Брокер Celery (`/1`)                  |
| `CHANNEL_LAYERS_REDIS`    | Channel Layer для WS (`/2`)           |

### JWT
| Переменная                            | Назначение                |
| ------------------------------------- | ------------------------- |
| `JWT_ACCESS_TOKEN_LIFETIME_MINUTES`   | Срок access-токена (60)   |
| `JWT_REFRESH_TOKEN_LIFETIME_DAYS`     | Срок refresh-токена (7)   |

### Email (Celery-нотификации)
`EMAIL_BACKEND`, `EMAIL_HOST`, `EMAIL_PORT`, `EMAIL_USE_TLS`, `EMAIL_HOST_USER`, `EMAIL_HOST_PASSWORD`.

### Stripe
| Переменная               | Назначение                                       |
| ------------------------ | ------------------------------------------------ |
| `STRIPE_PUBLIC_KEY`      | Публичный ключ для фронта                        |
| `STRIPE_SECRET_KEY`      | Секретный ключ для server-side API               |
| `STRIPE_WEBHOOK_SECRET`  | Подпись webhook для верификации событий          |

### Frontend
`REACT_APP_API_URL`, `REACT_APP_WS_URL` — точки подключения SPA к API и WS.

---

## 12. API, WebSocket и интеграции

### REST API

| Группа          | Метод   | Endpoint                            | Назначение                  |
| --------------- | ------- | ----------------------------------- | --------------------------- |
| **Auth**        | POST    | `/api/auth/register/`               | Регистрация                 |
|                 | POST    | `/api/auth/login/`                  | Получить JWT пару           |
|                 | POST    | `/api/auth/refresh/`                | Обновить access-токен       |
|                 | GET/PUT | `/api/auth/profile/`                | Профиль пользователя        |
| **Auctions**    | GET     | `/api/auctions/`                    | Список активных лотов       |
|                 | POST    | `/api/auctions/`                    | Создать лот                 |
|                 | GET     | `/api/auctions/{id}/`               | Детали лота                 |
|                 | PUT     | `/api/auctions/{id}/`               | Редактировать лот           |
|                 | DELETE  | `/api/auctions/{id}/`               | Удалить лот                 |
|                 | GET     | `/api/auctions/categories/`         | Дерево категорий            |
|                 | GET     | `/api/auctions/seller-dashboard/`   | Аналитика продавца          |
| **Bids**        | POST    | `/api/bids/`                        | Сделать ставку              |
|                 | GET     | `/api/bids/auction/{id}/`           | История ставок по лоту      |
|                 | POST    | `/api/bids/auto-bid/`               | Включить авто-биддинг       |
|                 | GET     | `/api/bids/auto-bid/`               | Список своих авто-ставок    |
|                 | DELETE  | `/api/bids/auto-bid/{id}/`          | Отменить авто-биддинг       |
| **Watchlist**   | GET     | `/api/watchlist/`                   | Избранные аукционы          |
|                 | POST    | `/api/watchlist/`                   | Добавить в избранное        |
|                 | DELETE  | `/api/watchlist/{id}/`              | Убрать из избранного        |
| **Payments**    | POST    | `/api/payments/`                    | Создать платёж              |
|                 | GET     | `/api/payments/{id}/`               | Статус платежа              |
|                 | POST    | `/api/payments/{id}/confirm/`       | Подтвердить получение       |
|                 | POST    | `/api/payments/stripe-webhook/`     | Stripe webhook endpoint     |

### WebSocket API

**Endpoint:**
```
ws://localhost/ws/auction/{auction_id}/
```

**Клиент → сервер** (ставка):
```json
{ "type": "place_bid", "amount": "150.00" }
```

**Сервер → клиент** (новая ставка):
```json
{
  "type": "new_bid",
  "bid": {
    "id": 1,
    "bidder": "alice",
    "amount": "150.00",
    "timestamp": "2026-05-17T10:30:00Z"
  },
  "auction": { "current_price": "150.00", "total_bids": 15 }
}
```

**Сервер → клиент** (закрытие лота):
```json
{
  "type": "auction_update",
  "status": "ended",
  "winner": "alice",
  "final_price": "250.00"
}
```

### Очереди Celery

| Очередь / задача                  | Назначение                                       |
| --------------------------------- | ------------------------------------------------ |
| `auctions.close_expired_auctions` | Beat-периодическая, проверяет истёкшие лоты      |
| `bids.process_auto_bid`           | Эмулирует автоматическую ставку proxy-биддера    |
| `notifications.send_email`        | Асинхронная отправка email                       |
| `notifications.send_outbid_batch` | Массовая рассылка «вас перебили»                 |
| `payments.handle_stripe_webhook`  | Обработка webhook-событий Stripe                 |

---

## 13. Мониторинг и эксплуатация

* **Health-checks** в Docker Compose для `db` и `redis` (`pg_isready`, `redis-cli ping`)
* **Логи** структурированные, поток в `stdout/stderr` — собираются драйвером Docker (`json-file` или внешний)
* **Метрики Celery** — рекомендуется добавить `flower`:
  ```bash
  docker compose exec celery celery -A config flower --port=5555
  ```
* **Sentry** (опционально) — подключается через `SENTRY_DSN` в `settings/production.py`
* **Запросы и медленные SQL** — `django-silk` или APM-агент (DataDog, New Relic) на ваш выбор
* **WebSocket-нагрузка** — масштабируется горизонтально: несколько инстансов Daphne за Nginx upstream, общий Redis Channel Layer

---

## 14. CI/CD

Рекомендуемая схема (GitHub Actions / GitLab CI):

1. **Lint** — `ruff`, `black --check`, `eslint`, `prettier --check`
2. **Tests** — `python manage.py test` + `npm test -- --watchAll=false`
3. **Security audit** — `pip-audit`, `npm audit`, `bandit`
4. **Build images** — multi-stage `Dockerfile`, тег `:sha` + `:latest`
5. **Push** в registry (GHCR / Docker Hub / private)
6. **Deploy** — `docker compose pull && docker compose up -d` через SSH или Kubernetes-манифесты

Шаблон workflow можно положить в `.github/workflows/ci.yml`.

---

## 15. Безопасность и платёжный контур

### Аутентификация и авторизация
* **JWT** с коротким временем жизни access-токена (60 мин) и refresh (7 дней)
* **Refresh rotation** — каждый refresh инвалидирует предыдущий
* **Permissions DRF** — `IsAuthenticated`, `IsSellerOrReadOnly`, `IsOwnerOrAdmin`
* **Throttling** — лимиты на запросы (login, registration, place_bid)

### Защита торгов
* **Race conditions при ставках** — `SELECT ... FOR UPDATE` в транзакции, плюс уникальные индексы
* **Anti-snipe** — последние N секунд автоматически продлевают аукцион
* **Идемпотентность** — клиентский `request_id` для защиты от двойных кликов

### Платежи
* **Stripe webhook signature** — каждый webhook верифицируется по `STRIPE_WEBHOOK_SECRET`
* **Эскроу-удержание** — `payment_intent.capture_method=manual`, средства захватываются после подтверждения покупателем
* **PCI DSS** — карточные данные **никогда** не проходят через ваш сервер (Stripe Elements / Checkout)

### Хранение и передача
* **Секреты** только в `.env` / секрет-сторе (никогда не в git)
* **HTTPS** обязательно в проде — Nginx + Let's Encrypt / ваш CA
* **CORS** ограничен `CORS_ALLOWED_ORIGINS`
* **Пароли** — `PBKDF2` (Django default) или `Argon2` для повышенной стойкости
* **Изображения лотов** — отдельный volume / S3, проверка MIME и размера

---

## 16. Роли компонентов в продакшене

| Компонент          | Роль в продакшене                                                                  |
| ------------------ | ---------------------------------------------------------------------------------- |
| **Nginx**          | TLS termination, статика, reverse proxy на Daphne, rate limiting                   |
| **Daphne**         | ASGI-сервер — HTTP + WebSocket в одном процессе, масштабируется горизонтально      |
| **Django + DRF**   | Бизнес-логика, ORM, валидация, сериализация, админка                               |
| **Channels**       | WebSocket consumers, маршрутизация realtime-событий                                |
| **Redis**          | Кэш Django, брокер Celery, Channel Layer для WS-broadcast                          |
| **PostgreSQL**     | Источник правды по всем сущностям (users, auctions, bids, payments)                |
| **Celery worker**  | Воркер фоновых задач: рассылки, обработка webhook, тяжёлые отчёты                  |
| **Celery beat**    | Планировщик — закрытие лотов, периодические напоминания                            |
| **Stripe**         | Внешний платёжный шлюз, эскроу-удержание, диспуты                                  |
| **React SPA**      | Клиент: рендер UI, WS-подписки, оптимистичные обновления, Stripe Elements          |

---

## 17. Лицензия

Проект распространяется под **MIT License** — см. файл [`LICENSE`](LICENSE).
Вы можете свободно использовать код в личных и коммерческих проектах с сохранением copyright-уведомления.

---

## 18. Поддержка

* **Issues** — [GitHub Issues](https://github.com/NodirOdilov/AuctionLive/issues) для багов и feature requests
* **Discussions** — обсуждение архитектурных решений и идей
* **Pull Requests** — приветствуются! Перед PR сделайте `make test` и `make lint`
* **Контакты автора** — [@NodirOdilov](https://github.com/NodirOdilov)

---

<div align="center">

**AuctionLive** — открытая платформа онлайн-аукционов нового поколения.
Сделано с ❤ на Python, Django и React.

⭐ Поставьте звезду, если проект оказался полезен!

</div>
