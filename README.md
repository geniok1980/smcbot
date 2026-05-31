# SMCBOT — Crypto Trading Bot Platform

[![Docker](https://img.shields.io/badge/Docker-Compose-2496ed?logo=docker)](https://www.docker.com)
[![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688?logo=fastapi)](https://fastapi.tiangolo.com)
[![Bybit](https://img.shields.io/badge/Bybit-API-F7B731)](https://bybit.com)
[![Next.js](https://img.shields.io/badge/Next.js-15-000000?logo=next.js)](https://nextjs.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql)](https://postgresql.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

**SMCBOT** — это платформа для автоматизированной криптоторговли, построенная по микросервисной архитектуре на Docker Compose. Включает торгового робота для Bybit (спот/фьючерсы), ML-сервис для предиктивной аналитики, WebSocket-сборщик свечей и позиций, Telegram-бота для управления и веб-дашборд.

![Architecture](https://via.placeholder.com/800x400/1a1a28/7c5cfc?text=SMCBOT+Architecture)

## 📋 Содержание

- [Архитектура](#архитектура)
- [Возможности](#возможности)
- [Технологический стек](#технологический-стек)
- [Быстрый старт](#быстрый-старт)
- [Сервисы](#сервисы)
- [API](#api)
- [Дашборд](#дашборд)
- [ML-сервис](#ml-сервис)
- [Telegram-бот](#telegram-бот)
- [Безопасность](#безопасность)
- [Лицензия](#лицензия)

## Архитектура

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Docker Compose                              │
│                                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
│  │  Bybit   │  │  Bybit │  │ Telegram │  │   Web Dashboard   │  │
│  │  (WS)    │  │  (WS)    │  │  (Bot)   │  │  (Next.js :3000)  │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────────┬──────────┘  │
│       │              │             │                  │             │
│  ┌────▼──────────────▼─────────────▼──────────────────▼──────────┐ │
│  │                     FastAPI (:8000)                            │ │
│  │         REST API — метрики, управление, события, клиенты        │ │
│  └────────────┬────────────────────────────────────────┬─────────┘ │
│               │                                        │           │
│  ┌────────────▼──────────┐           ┌─────────────────▼─────────┐ │
│  │      PostgreSQL 16    │           │     ML Service (:8010)    │ │
│  │  (свечи, события,     │           │  Обучение + предикт       │ │
│  │   снапшоты позиций)   │           │  (LightGBM/XGBoost)       │ │
│  └───────────────────────┘           └───────────────────────────┘ │
│                                                                     │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────────────┐  │
│  │  Robot         │  │  Kline Scanner │  │  Positions Service   │  │
│  │  (торговая     │  │  (свечи с      │  │  (снапшот позиций    │  │
│  │   логика)      │  │   Bybit)     │  │   с Bybit WS)        │  │
│  └────────────────┘  └────────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

## Возможности

### 🤖 Торговый робот (Bybit)
- **Поддержка спота и фьючерсов** (linear) — Bybit API (REST + WebSocket)
- **Smart Money Strategy** — поиск сетапов на основе концепций Price Action: зоны интереса OB (Order Blocks), дисбаланс FVG (Fair Value Gap) на старшем таймфрейме (HTF), вход на младшем таймфрейме (LTF)
- **Управление рисками** — Stop Loss, Take Profit, трейлинг-стоп, reduce-only
- **Ручной режим** — отправка рыночных/лимитных ордеров через API
- **Sell All** — экстренное закрытие всех позиций одним запросом
- **Конфигурация через .env** — смена режима (demo/testnet/mainnet) без пересборки
- **Логирование всех событий и сделок** в PostgreSQL

## Технологический стек

| Компонент | Технология |
|-----------|-----------|
| **Торговый робот** | Python 3.11, asyncio, aiohttp, websockets |
| **API** | FastAPI 0.115, Pydantic, SQLAlchemy, Alembic |
| **БД** | PostgreSQL 16 |
| **ML** | LightGBM, XGBoost, scikit-learn, pandas, numpy |
| **Дашборд** | Next.js 15, React 18, Tailwind CSS, shadcn/ui |
| **Telegram-бот** | python-telegram-bot 21.x |
| **Сбор данных** | websockets, aiohttp (Bybit/Bybit WS API) |
| **Инфраструктура** | Docker Compose |

## Быстрый старт

### Требования

- Docker Engine 24+ и Docker Compose v2
- Аккаунт на [Bybit](https://bybit.com) (API ключи)

### Установка

```bash
# 1. Клонировать репозиторий
git clone https://github.com/geniok1980/cryptobots.git
cd cryptobots

# 2. Настроить переменные окружения
cp SMCBOT/robot/.env.example SMCBOT/robot/.env
# Отредактировать .env — указать API ключи Bybit, режим работы

# 3. Запустить
docker compose up -d --build

# 4. Открыть дашборд
# http://localhost:3000/dashboard/overview
```

### Переменные окружения

Минимально необходимые:

| Переменная | Описание | Пример |
|-----------|----------|--------|
| `EE_BYBIT_ENV` | Режим Bybit | `testnet` / `mainnet` |
| `EE_BYBIT_CATEGORY` | Тип торговли | `linear` (фьючерсы) |
| `EE_BYBIT_API_KEY` | API ключ Bybit | `your_api_key` |
| `EE_BYBIT_SECRET_KEY` | Секретный ключ Bybit | `your_secret_key` |
| `EE_DATABASE_URL` | Подключение к БД | `postgresql://cryptobot:cryptobot@db:5432/cryptobot` |
| `EE_FASTAPI_URL` | URL FastAPI | `http://fastapi:8000` |

## Сервисы

| Сервис | Контейнер | Назначение | Порт |
|--------|-----------|-----------|------|
| `db` | `cryptobot-db` | PostgreSQL 16 | 5432 |
| `fastapi` | `cryptobot-fastapi` | REST API + метрики | 8000 |
| `robot` | `cryptobot-robot` | Торговая логика Bybit | — |
| `ml` | `cryptobot-ml` | ML-сервис (предикты) | 8010 |
| `telegram` | `cryptobot-telegram` | Telegram-бот управления | — |
| `dashboard` | `cryptobot-dashboard` | Next.js веб-панель | 3000 |
| `kline_scann` | `cryptobot-kline-scann` | Сбор свечей Bybit WS | — |
| `positions` | `cryptobot-positions` | Снапшот позиций Bybit WS | — |

Все сервисы используют общий `.env` файл из `SMCBOT/robot/.env`.

## API

Основные эндпойнты FastAPI (`http://localhost:8000`):

### Метрики
- `GET /metrics/summary` — агрегированные показатели для панели (profit/balance/open_positions/winrate)
- `GET /settings/env` — просмотр `EE_*` переменных (секреты замаскированы)

### Управление роботом
- `POST /robot/start` — включить торговлю
- `POST /robot/stop` — остановить торговлю
- `POST /robot/sell_all` — закрыть все текущие позиции market + reduceOnly

### Данные
- `PUT /positions/` — принять снапшот позиций
- `PUT /klines/` — записать свечи
- `PUT /klines/history` — записать исторические свечи

### Аналитика
- `POST /analysis/event` — записать событие анализа
- `GET /analysis/latest` — последние события

## Дашборд

Веб-панель на Next.js:

| Страница | Маршрут | Описание |
|----------|---------|----------|
| **Обзор** | `/dashboard/overview` | 4 KPI: прибыль, баланс, открытые позиции, винрейт |
| **Управление** | `/dashboard/control` | Кнопки старт/стоп/SELL ALL |
| **Настройки** | `/dashboard/settings` | Переменные `EE_*` (без секретов) |

Дашборд использует Tailwind CSS и shadcn/ui, адаптирован под мобильные устройства.

## ML-сервис

Включает:
- **Обучение моделей** на исторических данных (LightGBM)
- **Фильтрация входов и выходов** — ML-модель подтверждает или отклоняет Smart Money сетапы, отсеивая ложные сигналы
- **API для робота** — робот запрашивает сигнал перед открытием позиции
- **Автоматическое переобучение** — по расписанию или при накоплении новых данных

Отдельный контейнер, не блокирует работу робота.

## Telegram-бот

Позволяет управлять ботом через Telegram:

- `/start` — запустить торговлю
- `/stop` — остановить
- `/status` — текущее состояние
- `/balance` — баланс на бирже
- `/positions` — открытые позиции и PnL

## Безопасность

- **API ключи хранятся только в `.env`**, исключённом из репозитория
- **Режим testnet** для отладки без риска
- **Reduce-only ордера** для защиты при экстренном закрытии
- **Маскировка секретов** в веб-дашборде
- **Изоляция контейнеров** — Docker Compose, внутренняя сеть

## Лицензия

MIT License. Используйте на свой страх и риск. Криптотрейдинг сопряжён с высоким риском.
