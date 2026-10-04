# 🧪 Тестирование Tarot Bot

Этот документ содержит информацию о тестировании приложения.

## 📌 Виды тестирования

### 1. Юнит-тестирование (Unit Tests)

Тестирование отдельных функций и классов.

#### Запуск тестов:

```bash
# Установите pytest
pip install pytest pytest-asyncio httpx

# Запустите тесты
pytest backend/tests/ -v
```

#### Пример теста:

```python
# backend/tests/test_tarot.py
import pytest
from backend.tarot import TarotManager

@pytest.fixture
def tarot_manager():
    return TarotManager()

def test_get_deck(tarot_manager):
    deck = tarot_manager.get_deck()
    assert deck is not None
    assert "cards" in deck
    assert len(deck["cards"]) == 78

def test_get_random_card(tarot_manager):
    card, is_reversed = tarot_manager.get_random_card()
    assert card is not None
    assert "id" in card
    assert "name_ru" in card
    assert isinstance(is_reversed, bool)

def test_get_spread(tarot_manager):
    spread = tarot_manager.get_spread("three_cards")
    assert spread is not None
    assert spread["id"] == "three_cards"
    assert len(spread["positions"]) == 3
```

### 2. Интеграционное тестирование (Integration Tests)

Тестирование взаимодействия между компонентами.

#### Пример теста:

```python
# backend/tests/test_api.py
import pytest
from fastapi.testclient import TestClient
from backend.main import app
from backend.database import get_db, init_db
from backend.models import Base
import tempfile
import os

@pytest.fixture
def test_db():
    # Создайте временную базу данных
    db_fd, db_path = tempfile.mkstemp()
    os.close(db_fd)
    
    # Настройте тестовую БД
    os.environ["DATABASE_URL"] = f"sqlite:///{db_path}"
    
    # Инициализируйте БД
    Base.metadata.create_all(bind=engine)
    
    yield db_path
    
    # Удалите временную БД
    os.unlink(db_path)

@pytest.fixture
def client(test_db):
    return TestClient(app)

def test_auth_vk(client):
    # Тест авторизации
    response = client.post("/api/auth/vk", json={
        "vk_user_id": "test_user",
        "access_token": "test_token"
    })
    
    assert response.status_code == 200
    assert "access_token" in response.json()

def test_get_decks(client):
    response = client.get("/api/tarot/decks")
    assert response.status_code == 200
    assert len(response.json()) > 0
```

### 3. E2E тестирование (End-to-End Tests)

Тестирование полного пользовательского сценария.

#### Инструменты:
- **Selenium**
- **Cypress**
- **Playwright**

#### Пример теста с Selenium:

```python
# tests/e2e/test_main_flow.py
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

def test_main_flow():
    # Настройка драйвера
    driver = webdriver.Chrome()
    
    try:
        # Откройте приложение
        driver.get("http://localhost:3000")
        
        # Дождитесь загрузки
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "welcome-section"))
        )
        
        # Проверьте, что заголовок отображается
        assert "Tarot Bot" in driver.title
        
        # Нажмите на кнопку "Сделать расклад"
        quick_spread_btn = driver.find_element(By.ID, "quick-three-cards")
        quick_spread_btn.click()
        
        # Дождитесь открытия модального окна
        WebDriverWait(driver, 10).until(
            EC.visibility_of_element_located((By.ID, "spread-modal"))
        )
        
        # Заполните вопрос
        question_input = driver.find_element(By.ID, "spread-question")
        question_input.send_keys("Что меня ожидает?")
        
        # Нажмите "Сделать расклад"
        start_btn = driver.find_element(By.ID, "start-spread")
        start_btn.click()
        
        # Дождитесь результата
        WebDriverWait(driver, 10).until(
            EC.visibility_of_element_located((By.ID, "spread-result"))
        )
        
        # Проверьте, что карты отображаются
        cards = driver.find_elements(By.CLASS_NAME, "tarot-card")
        assert len(cards) > 0
        
    finally:
        driver.quit()
```

### 4. Тестирование API

Используйте **Postman** или **curl** для тестирования API.

#### Примеры curl команд:

```bash
# Авторизация
curl -X POST http://localhost:8000/api/auth/vk \
  -H "Content-Type: application/json" \
  -d '{"vk_user_id": "test_user", "access_token": "test_token"}'

# Получение колод
curl -X GET http://localhost:8000/api/tarot/decks \
  -H "Authorization: Bearer YOUR_TOKEN"

# Создание расклада
curl -X POST http://localhost:8000/api/tarot/spread \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"spread_id": "three_cards", "question": "Что меня ожидает?"}'

# Получение истории
curl -X GET http://localhost:8000/api/tarot/spreads/history \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### 5. Тестирование в VK Max

#### Ручное тестирование:

1. Откройте VK Max
2. Найдите ваше приложение
3. Проверьте:
   - ✅ Авторизация через VK
   - ✅ Отображение бесплатных раскладов
   - ✅ Создание расклада "3 карты"
   - ✅ Проверка лимита бесплатных раскладов
   - ✅ Отображение Premium раскладов (если подписка активна)
   - ✅ Создание Premium раскладов
   - ✅ Сохранение в историю
   - ✅ Просмотр истории
   - ✅ Настройки приложения
   - ✅ Оформление подписки

#### Автоматизированное тестирование:

Используйте **VK Mini Apps Tester** для автоматизированного тестирования в VK среде.

---

## 📊 Покрытие кода тестами

### Запуск тестов с покрытием:

```bash
# Установите pytest-cov
pip install pytest-cov

# Запустите тесты с покрытием
pytest backend/tests/ --cov=backend --cov-report=html

# Посмотрите отчет
open htmlcov/index.html
```

### Целевые показатели:
- **Backend**: 80%+
- **Frontend**: 70%+
- **Общее**: 75%+

---

## 🐛 Отчеты об ошибках

Если вы находите ошибку, пожалуйста, создайте issue с:

1. **Описанием ошибки**
2. **Шагами для воспроизведения**
3. **Ожидаемым поведением**
4. **Фактическим поведением**
5. **Скриншотами** (если применимо)
6. **Логами** (если применимо)
7. **Версией приложения**
8. **Типом устройства и ОС**

---

## 🔧 Настройка тестового окружения

### Тестовая база данных

Создайте отдельную базу данных для тестирования:

```python
# backend/tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from backend.models import Base
from backend.database import get_db

@pytest.fixture
def test_db():
    # Создайте in-memory базу данных
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(bind=engine)
    
    # Создайте сессию
    TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
    
    # Переопределите get_db для тестов
    def override_get_db():
        try:
            db = TestingSessionLocal()
            yield db
        finally:
            db.close()
    
    return override_get_db
```

### Mock данных

Используйте **pytest-fixtures** для создания тестовых данных:

```python
# backend/tests/fixtures/users.py
import pytest
from backend.models import User
from datetime import datetime, timedelta

@pytest.fixture
def test_user(db):
    user = User(
        vk_id="test_user_1",
        first_name="Тест",
        last_name="Пользователь",
        is_premium=False
    )
    db.add(user)
    db.commit()
    db.refresh(user)
    return user

@pytest.fixture
def premium_user(db):
    user = User(
        vk_id="premium_user_1",
        first_name="Премиум",
        last_name="Пользователь",
        is_premium=True,
        subscription_start=datetime.utcnow(),
        subscription_end=datetime.utcnow() + timedelta(days=30)
    )
    db.add(user)
    db.commit()
    db.refresh(user)
    return user
```

---

## ⏱️ Производительность

### Тестирование нагрузки

Используйте **Locust** для тестирования нагрузки:

```python
# locustfile.py
from locust import HttpUser, task, between

class TarotBotUser(HttpUser):
    wait_time = between(1, 5)
    
    @task
    def get_decks(self):
        self.client.get("/api/tarot/decks")
    
    @task
    def create_spread(self):
        # Авторизация
        auth_response = self.client.post("/api/auth/vk", json={
            "vk_user_id": "load_test_user",
            "access_token": "test_token"
        })
        token = auth_response.json()["access_token"]
        
        # Создание расклада
        self.client.post(
            "/api/tarot/spread",
            json={"spread_id": "three_cards"},
            headers={"Authorization": f"Bearer {token}"}
        )
```

### Запуск теста нагрузки:

```bash
locust -f locustfile.py --host=http://localhost:8000
```

Откройте `http://localhost:8089` для просмотра статистики.

---

## 📈 Метрики качества

### Ключевые метрики:

| Метрика | Целевое значение | Текущее значение |
|---------|------------------|------------------|
| Покрытие тестами | 80% | ? |
| Процент успешных тестов | 100% | ? |
| Время ответа API (среднее) | < 200ms | ? |
| Время ответа API (95 перцентиль) | < 500ms | ? |
| Время загрузки страницы | < 2s | ? |
| Память (backend) | < 200MB | ? |
| CPU (backend) | < 50% | ? |

---

## 🎯 План тестирования

### Фаза 1: Разработка (текущая)
- [x] Юнит-тесты для основных функций
- [ ] Интеграционные тесты для API
- [ ] Тесты для таро логики
- [ ] Тесты для базы данных

### Фаза 2: Альфа-тестирование
- [ ] Ручное тестирование всех функций
- [ ] Тестирование на разных устройствах
- [ ] Тестирование в VK Max
- [ ] Исправление критических багов

### Фаза 3: Бета-тестирование
- [ ] Тестирование с реальными пользователями
- [ ] Сбор отзывов
- [ ] Оптимизация производительности
- [ ] Финальное исправление багов

### Фаза 4: Релиз
- [ ] Финальная проверка всех функций
- [ ] Нагрузочное тестирование
- [ ] Релиз в production

---

## 📚 Полезные ресурсы

- [Pytest Документация](https://docs.pytest.org/)
- [FastAPI Тестирование](https://fastapi.tiangolo.com/tutorial/testing/)
- [Selenium Документация](https://www.selenium.dev/documentation/)
- [Locust Документация](https://docs.locust.io/)
- [VK Mini Apps Тестирование](https://dev.vk.com/mini-apps/testing)

---

**Удачного тестирования! 🧪**
