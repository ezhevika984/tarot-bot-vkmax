# 🚀 Развертывание Tarot Bot для VK Max

Этот документ содержит инструкции по развертыванию бота на различных платформах.

## 📋 Предварительные требования

### Необходимые инструменты:
- **Python 3.9+** (для backend)
- **Node.js 16+** (опционально, для сборки frontend)
- **Git** (для клонирования репозитория)
- **VK Разработчикам аккаунт** (для создания Mini App)

### Рекомендуемые инструменты:
- **PostgreSQL** или **SQLite** (для базы данных)
- **Redis** (для кэширования, опционально)
- **Nginx** или **Apache** (для production)

---

## 🌐 Развертывание на локальном компьютере (для разработки)

### 1. Клонируйте репозиторий

```bash
git clone <url-репозитория>
cd tarot-bot-vkmax
```

### 2. Настройка backend

```bash
# Создайте виртуальное окружение
python -m venv venv

# Активируйте виртуальное окружение
# Linux/Mac
source venv/bin/activate
# Windows
venv\Scripts\activate

# Установите зависимости
pip install -r backend/requirements.txt

# Создайте файл .env в папке backend
cp backend/.env.example backend/.env

# Отредактируйте .env файл
nano backend/.env  # или используйте другой редактор
```

**Пример содержимого .env файла:**
```env
VK_APP_ID=ваш_app_id
VK_APP_SECRET=ваш_app_secret
VK_SERVICE_TOKEN=ваш_service_token
DATABASE_URL=sqlite:///./tarot_bot.db
DEBUG=True
SECRET_KEY=ваш_секретный_ключ
SUBSCRIPTION_PRICE=499
SUBSCRIPTION_DURATION_DAYS=30
```

### 3. Запуск backend

```bash
cd backend
uvicorn main:app --reload
```

Backend будет доступен по адресу: `http://localhost:8000`

### 4. Запуск frontend

Для разработки можно просто открыть файл `frontend/index.html` в браузере.

Для более профессиональной разработки:

```bash
# Установите live-server
npm install -g live-server

# Запустите сервер
live-server frontend --port=3000
```

Frontend будет доступен по адресу: `http://localhost:3000`

---

## ☁️ Развертывание на VK Cloud

### 1. Создайте VK Mini App

1. Перейдите на [VK Разработчикам](https://dev.vk.com/)
2. Создайте новое приложение типа **Mini App**
3. Укажите:
   - Название: "Tarot Bot"
   - Описание: "Чат-бот для раскладов Таро"
   - Тип: Mini App
   - Доступ к VK Max: включите
4. Запомните **App ID** и **Secret Key**

### 2. Настройка VK Mini App

В настройках приложения:
- Укажите **Start URL**: `https://ваш-домен.com`
- Укажите **Base URL**: `https://ваш-домен.com`
- Настройте **Права доступа**:
  - `friends`
  - `photos`
  - `video`
  - `stories`
  - `wall`
  - `groups`
  - `stats`

### 3. Развертывание backend

#### Вариант 1: VK Cloud Functions

1. Установите VK Cloud CLI:
```bash
pip install vk-cloud-cli
```

2. Создайте функцию:
```bash
vk-cloud function create tarot-bot-backend --runtime python3.9
```

3. Разверните backend:
```bash
cd backend
vk-cloud function deploy tarot-bot-backend --handler main:app
```

#### Вариант 2: VK Cloud VM

1. Создайте виртуальную машину в VK Cloud
2. Подключитесь по SSH:
```bash
ssh user@ваш-ip-адрес
```

3. Установите зависимости:
```bash
sudo apt update
sudo apt install -y python3.9 python3-pip python3-venv nginx
```

4. Создайте и активируйте виртуальное окружение:
```bash
python3 -m venv /opt/tarot-bot/venv
source /opt/tarot-bot/venv/bin/activate
```

5. Установите зависимости:
```bash
pip install -r /path/to/tarot-bot-vkmax/backend/requirements.txt
```

6. Настройте systemd сервис:

Создайте файл `/etc/systemd/system/tarot-bot.service`:
```ini
[Unit]
Description=Tarot Bot Backend
After=network.target

[Service]
User=user
Group=user
WorkingDirectory=/opt/tarot-bot
Environment="PATH=/opt/tarot-bot/venv/bin"
ExecStart=/opt/tarot-bot/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

7. Запустите сервис:
```bash
sudo systemctl daemon-reload
sudo systemctl start tarot-bot
sudo systemctl enable tarot-bot
```

8. Настройте Nginx:

Создайте файл `/etc/nginx/sites-available/tarot-bot`:
```nginx
server {
    listen 80;
    server_name ваш-домен.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /static/ {
        alias /opt/tarot-bot/static/;
        expires 30d;
    }
}
```

9. Включите сайт:
```bash
sudo ln -s /etc/nginx/sites-available/tarot-bot /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 4. Развертывание frontend

Скопируйте файлы из `frontend/` в директорию, доступную через веб-сервер:

```bash
cp -r frontend/* /opt/tarot-bot/static/
```

### 5. Настройка HTTPS (рекомендуется)

Используйте Let's Encrypt для получения SSL сертификата:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d ваш-домен.com
```

---

## 🎯 Развертывание на других платформах

### Heroku

1. Создайте файл `Procfile` в корне проекта:
```
web: uvicorn backend.main:app --host 0.0.0.0 --port $PORT
```

2. Создайте файл `requirements.txt` в корне:
```
-r backend/requirements.txt
```

3. Разверните:
```bash
heroku create tarot-bot-vkmax
heroku config:set VK_APP_ID=ваш_app_id VK_APP_SECRET=ваш_secret
git push heroku main
```

### PythonAnywhere

1. Создайте новый веб-приложение
2. Выберите Python 3.9+
3. Загрузите код проекта
4. Настройте виртуальное окружение
5. Установите зависимости
6. Настройте WSGI файл

---

## 📡 Настройка VK Mini App

После развертывания backend:

1. Перейдите в настройки вашего VK Mini App
2. Укажите **Base URL**: `https://ваш-домен.com`
3. Укажите **Start URL**: `https://ваш-домен.com/frontend/index.html`
4. Настройте **Права доступа**
5. Сохраните изменения

### Проверка работы

1. Откройте VK Max
2. Найдите ваше приложение в каталоге
3. Откройте его
4. Проверьте, что все функции работают:
   - Авторизация
   - Бесплатные расклады
   - Premium функции (если подписка активна)
   - История раскладов

---

## 🔧 Настройка платежной системы

### YooMoney

1. Зарегистрируйтесь на [YooMoney](https://yoomoney.ru/)
2. Создайте магазин
3. Получите `client_id` и `client_secret`
4. Добавьте в .env:
```env
YOOMONEY_CLIENT_ID=ваш_client_id
YOOMONEY_CLIENT_SECRET=ваш_client_secret
YOOMONEY_REDIRECT_URI=https://ваш-домен.com/payment/callback
```

5. Настройте callback URL в YooMoney

### Сбербанк Онлайн

1. Зарегистрируйтесь в [Сбербанк Бизнес Онлайн](https://business.sberbank.ru/)
2. Создайте точку продаж
3. Получите API ключи
4. Добавьте в .env:
```env
SBERBANK_MERCHANT_ID=ваш_merchant_id
SBERBANK_API_KEY=ваш_api_ключ
```

---

## 📊 Мониторинг и логирование

### Логи

Логи backend можно просматривать:
- В консоли при запуске с `--reload`
- В файлах, если настроено логирование в файл
- В systemd journal (для production):
```bash
journalctl -u tarot-bot -f
```

### Мониторинг

Используйте:
- **Prometheus** + **Grafana** для метрик
- **Sentry** для отслеживания ошибок
- **Loggly** или **ELK Stack** для централизованного логирования

---

## 🔄 Обновление приложения

### Обновление кода

```bash
# Остановите сервис
sudo systemctl stop tarot-bot

# Обновите код
git pull

# Установите новые зависимости
source venv/bin/activate
pip install -r backend/requirements.txt

# Запустите сервис
sudo systemctl start tarot-bot
```

### Обновление базы данных

Если вы добавили новые миграции:

```bash
# Примените миграции
alembic upgrade head
```

---

## 🚨 Решение проблем

### Общие проблемы

**Проблема:** Backend не запускается
**Решение:**
```bash
# Проверьте логи
journalctl -u tarot-bot -n 50

# Проверьте зависимости
pip list | grep -E 'fastapi|uvicorn|sqlalchemy'

# Запустите в debug режиме
uvicorn backend.main:app --reload --log-level debug
```

**Проблема:** Frontend не подгружается
**Решение:**
- Проверьте, что файлы доступны по правильному пути
- Проверьте права доступа к файлам
- Проверьте настройки Nginx/Apache

**Проблема:** VK Mini App не открывается
**Решение:**
- Проверьте, что Base URL и Start URL правильно настроены
- Проверьте, что приложение опубликовано
- Проверьте права доступа в настройках приложения

### Проблемы с базой данных

**Проблема:** SQLite не работает в production
**Решение:** Используйте PostgreSQL:
```env
DATABASE_URL=postgresql://user:password@localhost/tarot_bot
```

**Проблема:** Миграции не применяются
**Решение:**
```bash
# Установите alembic
pip install alembic

# Инициализируйте alembic (если не сделано)
alembic init migrations

# Сгенерируйте миграцию
alembic revision --autogenerate -m "Initial migration"

# Примените миграцию
alembic upgrade head
```

---

## 📞 Поддержка

Если у вас возникли проблемы:
1. Проверьте логи
2. Проверьте настройки .env
3. Убедитесь, что все зависимости установлены
4. Проверьте настройки VK Mini App

Для дополнительной помощи обратитесь в техническую поддержку VK или изучите [документацию VK](https://dev.vk.com/).

---

## 📚 Дополнительные ресурсы

- [Документация VK Mini Apps](https://dev.vk.com/mini-apps)
- [Документация VK Bridge](https://dev.vk.com/mini-apps/bridge)
- [FastAPI Документация](https://fastapi.tiangolo.com/)
- [SQLAlchemy Документация](https://www.sqlalchemy.org/)
- [YooMoney API](https://yoomoney.ru/docs/)

---

**Удачного развертывания! 🚀**
