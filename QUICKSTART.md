# 🚀 Быстрый старт с Tarot Bot для VK Max

Этот guide поможет вам быстро запустить и протестировать Tarot Bot в VK Max.

---

## ⚡ Быстрая установка (5 минут)

### 1. Клонируйте репозиторий

```bash
# Клонируйте репозиторий
git clone https://github.com/your-username/tarot-bot-vkmax.git
cd tarot-bot-vkmax
```

*Или просто используйте текущую папку, если вы уже в ней.*

---

### 2. Установите зависимости

#### Backend (Python)

```bash
# Создайте виртуальное окружение
python -m venv venv

# Активируйте его
# Linux/Mac:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# Установите Python зависимости
pip install -r backend/requirements.txt
```

#### Frontend (опционально)

Для разработки frontend не требуется установка Node.js. Вы можете просто открыть HTML файлы в браузере.

---

### 3. Настройте конфигурацию

```bash
# Перейдите в папку backend
cd backend

# Скопируйте пример .env файла
cp .env.example .env

# Отредактируйте .env файл
nano .env  # или используйте любой редактор
```

**Минимальная конфигурация для разработки:**
```env
# VK App (можно оставить пустыми для локальной разработки)
VK_APP_ID=your_vk_app_id
VK_APP_SECRET=your_vk_app_secret

# База данных (SQLite по умолчанию)
DATABASE_URL=sqlite:///./tarot_bot.db

# Режим отладки
DEBUG=True

# Секретный ключ (сгенерируйте свой)
SECRET_KEY=your-secret-key-here

# Цена подписки (в рублях)
SUBSCRIPTION_PRICE=499
SUBSCRIPTION_DURATION_DAYS=30
```

> **Примечание:** Для генерации секретного ключа используйте:
> ```python
> import secrets
> print(secrets.token_hex(32))
> ```

---

### 4. Инициализируйте базу данных

```bash
# Вернитесь в корень проекта
cd ..

# Инициализируйте БД (таблицы будут созданы автоматически при первом запуске)
python -c "from backend.database import init_db; init_db()"
```

---

### 5. Запустите backend

```bash
# Перейдите в папку backend
cd backend

# Запустите сервер
uvicorn main:app --reload
```

✅ **Backend запущен!** Доступен по адресу: `http://localhost:8000`

---

### 6. Запустите frontend

Откройте новый терминал и выполните:

```bash
# Перейдите в папку frontend
cd frontend

# Запустите простой HTTP сервер
# Вариант 1: Python HTTP сервер
python -m http.server 3000

# Вариант 2: Установите live-server
# npm install -g live-server
# live-server --port=3000
```

✅ **Frontend запущен!** Доступен по адресу: `http://localhost:3000`

---

## 🎯 Тестирование работы

### 1. Проверьте API

Откройте в браузере: `http://localhost:8000/api/`

Вы должны увидеть:
```json
{
    "message": "Welcome to Tarot Bot VK Max API",
    "success": true,
    "version": "1.0.0",
    "docs": "/api/docs"
}
```

### 2. Проверьте документацию API

Откройте: `http://localhost:8000/api/docs`

Здесь вы найдете интерактивную документацию Swagger.

### 3. Проверьте frontend

Откройте: `http://localhost:3000`

Вы должны увидеть интерфейс Tarot Bot.

---

## 🔍 Основные функции для тестирования

### 1. Получение колод Таро

```bash
curl http://localhost:8000/api/tarot/decks
```

### 2. Получение бесплатных раскладов

```bash
curl http://localhost:8000/api/tarot/spreads/free
```

### 3. Создание расклада (требуется авторизация)

Сначала получите токен:
```bash
# Для тестирования можно использовать mock авторизацию
curl -X POST http://localhost:8000/api/auth/vk \
  -H "Content-Type: application/json" \
  -d '{"vk_user_id": "test_user", "access_token": "test_token"}'
```

Затем создайте расклад:
```bash
curl -X POST http://localhost:8000/api/tarot/spread \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"spread_id": "three_cards", "question": "Что меня ожидает?"}'
```

---

## 📱 Тестирование в VK Max

### 1. Создайте VK Mini App

1. Перейдите на [VK Разработчикам](https://dev.vk.com/)
2. Создайте новое приложение типа **Mini App**
3. Укажите:
   - Название: "Tarot Bot"
   - Base URL: `http://localhost:3000` (для разработки)
   - Start URL: `http://localhost:3000/frontend/index.html`

### 2. Настройте туннелирование (для локальной разработки)

Используйте **ngrok** для доступа к локальному серверу из VK:

```bash
# Установите ngrok
npm install -g ngrok

# Запустите туннель для frontend
ngrok http 3000

# Запустите туннель для backend
ngrok http 8000
```

Обновите настройки VK Mini App:
- Base URL: `https://your-ngrok-url.ngrok.io`
- Start URL: `https://your-ngrok-url.ngrok.io/frontend/index.html`

В настройках frontend (в `auth.js`):
```javascript
API_BASE_URL = 'https://your-backend-ngrok-url.ngrok.io';
```

### 3. Протестируйте в VK Max

1. Откройте VK Max
2. Найдите ваше приложение
3. Откройте его
4. Проверьте:
   - ✅ Отображение интерфейса
   - ✅ Авторизация через VK
   - ✅ Создание раскладов
   - ✅ Отображение истории

---

## 🎨 Доступные расклады

### Бесплатные (1 раз в неделю):
- **Три карты** (`three_cards`) - Прошлое, Настоящее, Будущее

### Premium (требуется подписка):
- **Кельтский крест** (`celtic_cross`) - 10 карт, глубокий анализ
- **Дерево жизни** (`tree_of_life`) - 11 карт, каббалистический расклад
- **Астрологический** (`astrological`) - 12 карт по знакам зодиака
- **Расклад на год** (`year_ahead`) - 12 карт на каждый месяц

---

## 🔧 Настройка Premium подписки

### Для тестирования:

Вы можете вручную активировать Premium для пользователя:

```python
# В Python shell
from backend.database import SessionLocal
from backend.models import User
from datetime import datetime, timedelta

db = SessionLocal()
user = db.query(User).filter(User.vk_id == "test_user").first()
if user:
    user.is_premium = True
    user.subscription_start = datetime.utcnow()
    user.subscription_end = datetime.utcnow() + timedelta(days=30)
    db.commit()
```

Или через API:
```bash
curl -X POST http://localhost:8000/api/payment/subscribe \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"payment_system": "demo"}'
```

---

## 🐛 Решение распространенных проблем

### Проблема: Backend не запускается
**Решение:**
```bash
# Проверьте установлены ли зависимости
pip list | grep -E 'fastapi|uvicorn'

# Проверьте версию Python
python --version

# Запустите с подробными логами
uvicorn backend.main:app --reload --log-level debug
```

### Проблема: Frontend не подгружает данные
**Решение:**
- Проверьте, что backend запущен
- Проверьте CORS настройки
- Проверьте, что API_BASE_URL правильно настроен в `auth.js`

### Проблема: Ошибка подключения к базе данных
**Решение:**
- Проверьте, что файл `.env` существует
- Проверьте, что путь к БД правильный
- Для SQLite убедитесь, что у вас есть права на запись в папку

### Проблема: VK Mini App не открывается
**Решение:**
- Проверьте Base URL и Start URL
- Проверьте, что приложение опубликовано
- Проверьте права доступа в настройках приложения

---

## 📚 Документация

- [Полная документация](README.md)
- [Развертывание](DEPLOYMENT.md)
- [Тестирование](TESTING.md)
- [API Документация](http://localhost:8000/api/docs)

---

## 💬 Поддержка

Если у вас возникли вопросы или проблемы:

1. Проверьте этот guide
2. Проверьте логи в консоли
3. Проверьте файл `.env`
4. Проверьте подключение к интернету
5. Перезапустите серверы

---

## ✅ Чеклист перед production

- [ ] Настроена база данных (не SQLite для production)
- [ ] Настроены переменные окружения
- [ ] Настроен HTTPS
- [ ] Настроен домен
- [ ] Настроена платежная система
- [ ] Проведено тестирование
- [ ] Настроен мониторинг
- [ ] Созданы резервные копии

---

**Готово! Ваш Tarot Bot работает! 🎉**

Теперь вы можете:
- Создавать расклады Таро
- Просматривать историю
- Управлять настройками
- Тестировать Premium функции

Удачи в разработке! 🚀
