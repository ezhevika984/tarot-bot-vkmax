# 🔮 Tarot Bot для VK Max

Чат-бот для раскладов таро с премиум-подпиской, работающий в мессенджере **VK Max** (VK WorkSpace).

## 📌 О проекте

Бот предоставляет:
- **Бесплатные расклады:** 3 карты (1 раз в неделю)
- **Премиум-функции:** Кельтский крест, Дерево жизни, Астрологический, Расклад на год, персональные интерпретации
- **Регистрация пользователей** через VK ID
- **Хранение истории раскладов**
- **Выбор колоды таро** (Rider-Waite, Marseille)
- **Экспорт в PDF**

## 🛠 Технический стек

- **Frontend:** HTML/CSS/JavaScript (VK Mini App)
- **Backend:** Python + FastAPI
- **База данных:** SQLite (для прототипа)
- **API:** VK Mini Apps API, VK ID для авторизации

## 📁 Структура проекта

```
tarot-bot-vkmax/
├── backend/
│   ├── main.py          # FastAPI сервер
│   ├── config.py        # Конфигурация
│   ├── models.py        # Модели БД
│   ├── database.py      # Работа с БД
│   ├── tarot.py         # Логика карт и раскладов
│   ├── auth.py          # Авторизация через VK
│   ├── payment.py       # Система подписок
│   └── schemas.py       # Pydantic схемы
│
├── frontend/
│   ├── index.html       # Главная страница
│   ├── styles.css       # Стили
│   ├── app.js           # Логика фронтенда
│   ├── auth.js          # Авторизация
│   ├── spreads.js       # Логика раскладов
│   └── assets/
│       ├── images/      # Изображения карт
│       └── icons/
│
├── data/
│   ├── tarot_deck.json  # База карт таро
│   └── spreads.json     # Описания раскладов
│
├── static/
│   └── (скомпилированные файлы для хостинга)
│
└── README.md
```

## 🚀 Быстрый старт

### 1. Настройка VK Mini App

1. Перейдите в [VK Разработчикам](https://dev.vk.com/)
2. Создайте новое Mini App приложение
3. Укажите:
   - Название: "Tarot Bot"
   - Тип: Mini App
   - Доступ к VK Max: включите
4. Запомните **App ID** и **Secret Key**

### 2. Настройка окружения

```bash
cd tarot-bot-vkmax
python -m venv venv
source venv/bin/activate  # Linux/Mac
# или
venv\Scripts\activate     # Windows

pip install -r requirements.txt
```

### 3. Создайте файл `.env`

```env
VK_APP_ID=your_app_id
VK_APP_SECRET=your_app_secret
VK_SERVICE_TOKEN=your_service_token
DATABASE_URL=sqlite:///./tarot_bot.db
DEBUG=True
```

### 4. Запуск backend

```bash
cd backend
uvicorn main:app --reload
```

### 5. Запуск frontend

Откройте `frontend/index.html` в браузере или разверните на хостинге.

## 📦 Зависимости

- fastapi
- uvicorn
- sqlalchemy
- python-dotenv
- pydantic
- aiohttp
- reportlab (для генерации PDF)

## 🎴 Данные карт таро

Файл `data/tarot_deck.json` содержит:
- 78 карт (22 Старшие Арканы + 56 Младшие Арканы)
- Названия на русском и английском
- Описания в прямом и перевернутом положении
- Категории (Аркан, масть, номер)
- Изображения (ссылки или локальные пути)

## 🔐 Авторизация

Используется VK ID для:
- Регистрации пользователей
- Идентификации в системе
- Связывания с подписками

## 💳 Система подписок

Реализовано:
- Проверка статуса подписки
- Ограничение бесплатных раскладов (1 раз в неделю)
- Доступ к премиум-раскладам

Для интеграции с платежными системами:
- YooMoney
- Сбербанк Онлайн
- VK Pay

## 📊 API Эндпоинты

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST  | /api/auth | Авторизация через VK |
| GET   | /api/user | Получение данных пользователя |
| POST  | /api/spread | Создание расклада |
| GET   | /api/spreads | История раскладов |
| POST  | /api/subscribe | Оформление подписки |
| GET   | /api/tarot/cards | Получение случайных карт |
| GET   | /api/tarot/deck | Получение всей колоды |

## 🎨 Дизайн

- Адаптивный интерфейс для VK Mini App
- Темная/светлая тема
- Анимации при переворачивании карт
- Красивая визуализация раскладов

## 📝 Лицензия

MIT License

---

Создано для VK Max мессенджера.
