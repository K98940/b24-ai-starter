# 🐍 Python Backend: Общие знания для разработки с Битрикс24

## 📋 Обзор

Этот файл содержит **общую информацию по разработке Python-приложений** для Битрикс24, не зависящую от конкретных задач. Для специфических инструкций обратитесь к соответствующим файлам в этой папке.

---

## 🚀 Python экосистема для Битрикс24

### Основные инструменты

#### Bitrix24 Python SDK
- **Библиотека**: `b24pysdk`
- **Версия**: Стабильная релизная
- **Требования**: Python 3.8+, requests, pydantic
- **Лицензия**: MIT

#### Типичные зависимости (requirements.txt)
```txt
b24pysdk>=1.0.0
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
pydantic>=2.0.0
python-multipart>=0.0.6
aiofiles>=23.0.0
python-dotenv>=1.0.0

# Для разработки
pytest>=7.4.0
pytest-asyncio>=0.21.0
black>=23.0.0
isort>=5.12.0
mypy>=1.5.0
```

### Типичная архитектура Python-проекта

```
project/
├── app/
│   ├── __init__.py
│   ├── main.py                # FastAPI приложение
│   ├── config.py              # Конфигурация
│   ├── models/                # Pydantic модели
│   │   ├── __init__.py
│   │   ├── deal.py
│   │   └── contact.py
│   ├── services/              # Бизнес-логика
│   │   ├── __init__.py
│   │   ├── bitrix24_service.py
│   │   └── deal_service.py
│   ├── routers/               # API маршруты
│   │   ├── __init__.py
│   │   ├── deals.py
│   │   └── contacts.py
│   └── utils/                 # Утилиты
├── tests/                     # Тесты
├── requirements.txt
├── .env
└── Dockerfile
```

---

## 🔧 Основные паттерны разработки

### 1. Инициализация SDK

#### Простая инициализация
```python
from b24pysdk import Bitrix24

# Webhook
b24 = Bitrix24(
    webhook_url="https://your-portal.bitrix24.com/rest/1/webhook_key/"
)

# OAuth
b24 = Bitrix24(
    domain="your-portal.bitrix24.com",
    client_id="your_client_id",
    client_secret="your_client_secret",
    access_token="access_token",
    refresh_token="refresh_token"
)
```

#### С конфигурацией (рекомендуется)
```python
# config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    bitrix24_webhook_url: str
    bitrix24_domain: str = ""
    bitrix24_client_id: str = ""
    bitrix24_client_secret: str = ""
    
    class Config:
        env_file = ".env"

settings = Settings()

# services/bitrix24_service.py
from b24pysdk import Bitrix24
from ..config import settings

class Bitrix24Service:
    def __init__(self):
        if settings.bitrix24_webhook_url:
            self.b24 = Bitrix24(webhook_url=settings.bitrix24_webhook_url)
        else:
            self.b24 = Bitrix24(
                domain=settings.bitrix24_domain,
                client_id=settings.bitrix24_client_id,
                client_secret=settings.bitrix24_client_secret
            )
```

### 2. Работа с данными CRM (асинхронно)

```python
import asyncio
from typing import List, Optional
from b24pysdk import Bitrix24

class DealService:
    def __init__(self, b24: Bitrix24):
        self.b24 = b24
    
    async def get_deals_list(
        self, 
        stage_id: Optional[str] = None,
        limit: int = 50
    ) -> List[dict]:
        """Получение списка сделок"""
        filter_params = {}
        if stage_id:
            filter_params['STAGE_ID'] = stage_id
            
        deals = await self.b24.crm.deals.list(
            filter=filter_params,
            select=['ID', 'TITLE', 'OPPORTUNITY', 'STAGE_ID'],
            limit=limit
        )
        return deals
    
    async def get_deal_by_id(self, deal_id: int) -> Optional[dict]:
        """Получение сделки по ID"""
        try:
            deal = await self.b24.crm.deals.get(deal_id)
            return deal
        except Exception:
            return None
    
    async def create_deal(self, deal_data: dict) -> int:
        """Создание новой сделки"""
        deal_id = await self.b24.crm.deals.add(deal_data)
        return deal_id
    
    async def update_deal(self, deal_id: int, update_data: dict) -> bool:
        """Обновление сделки"""
        result = await self.b24.crm.deals.update(deal_id, update_data)
        return result
```

### 3. Pydantic модели для типизации

```python
# models/deal.py
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime

class DealBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=255)
    opportunity: Optional[float] = Field(None, ge=0)
    currency_id: str = Field(default="RUB")
    stage_id: Optional[str] = None

class DealCreate(DealBase):
    pass

class DealUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=255)
    opportunity: Optional[float] = Field(None, ge=0)
    stage_id: Optional[str] = None

class DealResponse(DealBase):
    id: int
    date_create: Optional[datetime] = None
    date_modify: Optional[datetime] = None
    
    class Config:
        from_attributes = True

# Конвертация из Bitrix24 API response
class DealConverter:
    @staticmethod
    def from_bitrix24(data: dict) -> DealResponse:
        """Конвертация ответа API Битрикс24 в Pydantic модель"""
        return DealResponse(
            id=int(data['ID']),
            title=data.get('TITLE', ''),
            opportunity=float(data.get('OPPORTUNITY', 0)) if data.get('OPPORTUNITY') else None,
            currency_id=data.get('CURRENCY_ID', 'RUB'),
            stage_id=data.get('STAGE_ID'),
            date_create=data.get('DATE_CREATE'),
            date_modify=data.get('DATE_MODIFY')
        )
```

### 4. FastAPI маршруты

```python
# routers/deals.py
from fastapi import APIRouter, HTTPException, Depends
from typing import List, Optional
from ..models.deal import DealCreate, DealUpdate, DealResponse
from ..services.deal_service import DealService
from ..dependencies import get_deal_service

router = APIRouter(prefix="/deals", tags=["deals"])

@router.get("/", response_model=List[DealResponse])
async def get_deals(
    stage_id: Optional[str] = None,
    limit: int = 50,
    deal_service: DealService = Depends(get_deal_service)
):
    """Получить список сделок"""
    deals = await deal_service.get_deals_list(stage_id=stage_id, limit=limit)
    return [DealConverter.from_bitrix24(deal) for deal in deals]

@router.get("/{deal_id}", response_model=DealResponse)
async def get_deal(
    deal_id: int,
    deal_service: DealService = Depends(get_deal_service)
):
    """Получить сделку по ID"""
    deal = await deal_service.get_deal_by_id(deal_id)
    if not deal:
        raise HTTPException(status_code=404, detail="Deal not found")
    return DealConverter.from_bitrix24(deal)

@router.post("/", response_model=dict)
async def create_deal(
    deal_data: DealCreate,
    deal_service: DealService = Depends(get_deal_service)
):
    """Создать новую сделку"""
    deal_id = await deal_service.create_deal(deal_data.model_dump())
    return {"id": deal_id, "message": "Deal created successfully"}

@router.patch("/{deal_id}")
async def update_deal(
    deal_id: int,
    update_data: DealUpdate,
    deal_service: DealService = Depends(get_deal_service)
):
    """Обновить сделку"""
    # Исключаем None значения
    update_dict = update_data.model_dump(exclude_none=True)
    
    result = await deal_service.update_deal(deal_id, update_dict)
    if not result:
        raise HTTPException(status_code=404, detail="Deal not found")
    
    return {"message": "Deal updated successfully"}
```

---

## 🏗️ Архитектурные подходы

### 1. Dependency Injection с FastAPI

```python
# dependencies.py
from functools import lru_cache
from .services.bitrix24_service import Bitrix24Service
from .services.deal_service import DealService

@lru_cache()
def get_bitrix24_service() -> Bitrix24Service:
    """Singleton Bitrix24 сервиса"""
    return Bitrix24Service()

def get_deal_service(
    b24_service: Bitrix24Service = Depends(get_bitrix24_service)
) -> DealService:
    """Получение сервиса для работы с сделками"""
    return DealService(b24_service.b24)
```

### 2. Repository паттерн

```python
# repositories/deal_repository.py
from abc import ABC, abstractmethod
from typing import List, Optional
from ..models.deal import DealResponse

class DealRepositoryInterface(ABC):
    @abstractmethod
    async def find_by_id(self, deal_id: int) -> Optional[DealResponse]:
        pass
    
    @abstractmethod
    async def find_by_stage(self, stage_id: str) -> List[DealResponse]:
        pass
    
    @abstractmethod
    async def create(self, deal_data: dict) -> int:
        pass

class Bitrix24DealRepository(DealRepositoryInterface):
    def __init__(self, b24: Bitrix24):
        self.b24 = b24
    
    async def find_by_id(self, deal_id: int) -> Optional[DealResponse]:
        try:
            deal_data = await self.b24.crm.deals.get(deal_id)
            return DealConverter.from_bitrix24(deal_data)
        except Exception:
            return None
    
    async def find_by_stage(self, stage_id: str) -> List[DealResponse]:
        deals_data = await self.b24.crm.deals.list(
            filter={'STAGE_ID': stage_id}
        )
        return [DealConverter.from_bitrix24(deal) for deal in deals_data]
    
    async def create(self, deal_data: dict) -> int:
        return await self.b24.crm.deals.add(deal_data)
```

### 3. Service Layer

```python
# services/deal_service.py
from typing import List, Optional
from ..repositories.deal_repository import DealRepositoryInterface
from ..models.deal import DealResponse, DealCreate

class DealService:
    def __init__(self, deal_repository: DealRepositoryInterface):
        self.repository = deal_repository
    
    async def get_active_deals(self) -> List[DealResponse]:
        """Получить активные сделки"""
        active_stages = ['NEW', 'PREPARATION', 'PROPOSAL']
        deals = []
        
        for stage in active_stages:
            stage_deals = await self.repository.find_by_stage(stage)
            deals.extend(stage_deals)
            
        return sorted(deals, key=lambda x: x.date_create, reverse=True)
    
    async def create_deal_with_validation(self, deal_data: DealCreate) -> int:
        """Создать сделку с дополнительной валидацией"""
        
        # Бизнес-логика валидации
        if deal_data.opportunity and deal_data.opportunity < 1000:
            raise ValueError("Минимальная сумма сделки: 1000")
        
        # Создание сделки
        deal_dict = deal_data.model_dump()
        deal_id = await self.repository.create(deal_dict)
        
        # Логирование или дополнительные действия
        await self._log_deal_creation(deal_id, deal_data)
        
        return deal_id
    
    async def _log_deal_creation(self, deal_id: int, deal_data: DealCreate):
        """Логирование создания сделки"""
        # Здесь может быть логика логирования, уведомлений и т.д.
        pass
```

---

## 🔐 Безопасность и best practices

### 1. Валидация и обработка ошибок

```python
from fastapi import HTTPException
from pydantic import ValidationError
import logging

logger = logging.getLogger(__name__)

class DealService:
    async def safe_create_deal(self, deal_data: dict) -> dict:
        """Безопасное создание сделки с обработкой ошибок"""
        try:
            # Валидация через Pydantic
            validated_data = DealCreate(**deal_data)
            
            # Создание сделки
            deal_id = await self.repository.create(validated_data.model_dump())
            
            logger.info(f"Deal created successfully: {deal_id}")
            return {"success": True, "deal_id": deal_id}
            
        except ValidationError as e:
            logger.error(f"Validation error: {e}")
            raise HTTPException(status_code=422, detail=str(e))
            
        except Exception as e:
            logger.error(f"Unexpected error creating deal: {e}")
            raise HTTPException(status_code=500, detail="Internal server error")
```

### 2. Кэширование с Redis

```python
import redis.asyncio as redis
import json
from typing import Optional

class CachedDealService:
    def __init__(self, deal_service: DealService, redis_client: redis.Redis):
        self.deal_service = deal_service
        self.redis = redis_client
        self.cache_ttl = 300  # 5 минут
    
    async def get_deal_by_id(self, deal_id: int) -> Optional[DealResponse]:
        """Получение сделки с кэшированием"""
        cache_key = f"deal:{deal_id}"
        
        # Проверяем кэш
        cached = await self.redis.get(cache_key)
        if cached:
            deal_data = json.loads(cached)
            return DealResponse(**deal_data)
        
        # Получаем из API
        deal = await self.deal_service.get_deal_by_id(deal_id)
        if deal:
            # Сохраняем в кэш
            await self.redis.setex(
                cache_key, 
                self.cache_ttl, 
                deal.model_dump_json()
            )
        
        return deal
```

### 3. Асинхронные batch-запросы

```python
import asyncio
from typing import List

class BatchDealService:
    def __init__(self, b24: Bitrix24):
        self.b24 = b24
    
    async def get_deals_with_contacts(self, deal_ids: List[int]) -> dict:
        """Получение сделок и их контактов параллельно"""
        
        # Создаем задачи для параллельного выполнения
        deal_tasks = [self.b24.crm.deals.get(deal_id) for deal_id in deal_ids]
        contact_tasks = [
            self.b24.crm.deals.contacts.get(deal_id) 
            for deal_id in deal_ids
        ]
        
        # Выполняем все запросы параллельно
        deals_results = await asyncio.gather(*deal_tasks, return_exceptions=True)
        contacts_results = await asyncio.gather(*contact_tasks, return_exceptions=True)
        
        # Обрабатываем результаты
        result = {}
        for i, deal_id in enumerate(deal_ids):
            if not isinstance(deals_results[i], Exception):
                result[deal_id] = {
                    'deal': deals_results[i],
                    'contacts': contacts_results[i] if not isinstance(contacts_results[i], Exception) else []
                }
        
        return result
```

---

## 🧪 Тестирование

### Unit тесты с pytest

```python
# tests/test_deal_service.py
import pytest
from unittest.mock import AsyncMock, Mock
from app.services.deal_service import DealService
from app.models.deal import DealCreate, DealResponse

@pytest.fixture
def mock_repository():
    """Mock репозитория для тестов"""
    repository = Mock()
    repository.create = AsyncMock(return_value=123)
    repository.find_by_id = AsyncMock(return_value=None)
    return repository

@pytest.fixture
def deal_service(mock_repository):
    """Сервис сделок с mock репозиторием"""
    return DealService(mock_repository)

@pytest.mark.asyncio
async def test_create_deal_with_validation_success(deal_service, mock_repository):
    """Тест успешного создания сделки"""
    deal_data = DealCreate(
        title="Test Deal",
        opportunity=50000.0,
        currency_id="RUB"
    )
    
    result = await deal_service.create_deal_with_validation(deal_data)
    
    assert result == 123
    mock_repository.create.assert_called_once()

@pytest.mark.asyncio
async def test_create_deal_validation_error(deal_service):
    """Тест ошибки валидации"""
    deal_data = DealCreate(
        title="Small Deal",
        opportunity=500.0  # Меньше минимальной суммы
    )
    
    with pytest.raises(ValueError, match="Минимальная сумма сделки: 1000"):
        await deal_service.create_deal_with_validation(deal_data)
```

### Integration тесты

```python
# tests/test_integration.py
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_create_and_get_deal():
    """Интеграционный тест создания и получения сделки"""
    async with AsyncClient(app=app, base_url="http://test") as client:
        # Создаем сделку
        create_response = await client.post(
            "/deals/",
            json={
                "title": "Integration Test Deal",
                "opportunity": 75000.0,
                "currency_id": "RUB"
            }
        )
        
        assert create_response.status_code == 200
        deal_id = create_response.json()["id"]
        
        # Получаем созданную сделку
        get_response = await client.get(f"/deals/{deal_id}")
        
        assert get_response.status_code == 200
        deal_data = get_response.json()
        assert deal_data["title"] == "Integration Test Deal"
        assert deal_data["opportunity"] == 75000.0
```

---

## 📊 Мониторинг и производительность

### 1. Логирование с структурированными данными

```python
import logging
import structlog
from typing import Any, Dict

# Конфигурация structlog
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.UnicodeDecoder(),
        structlog.processors.JSONRenderer()
    ],
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    cache_logger_on_first_use=True,
)

logger = structlog.get_logger()

class DealService:
    async def create_deal(self, deal_data: dict) -> int:
        """Создание сделки с логированием"""
        logger.info(
            "Creating new deal",
            deal_title=deal_data.get('title'),
            deal_amount=deal_data.get('opportunity')
        )
        
        try:
            deal_id = await self.repository.create(deal_data)
            
            logger.info(
                "Deal created successfully",
                deal_id=deal_id,
                deal_title=deal_data.get('title')
            )
            
            return deal_id
            
        except Exception as e:
            logger.error(
                "Failed to create deal",
                error=str(e),
                deal_data=deal_data,
                exc_info=True
            )
            raise
```

### 2. Метрики производительности

```python
import time
from functools import wraps
from typing import Callable

def measure_time(func_name: str = None):
    """Декоратор для измерения времени выполнения"""
    def decorator(func: Callable):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            name = func_name or f"{func.__module__}.{func.__name__}"
            start_time = time.perf_counter()
            
            try:
                result = await func(*args, **kwargs)
                duration = time.perf_counter() - start_time
                
                logger.info(
                    "Function executed successfully",
                    function_name=name,
                    duration_seconds=duration
                )
                
                return result
                
            except Exception as e:
                duration = time.perf_counter() - start_time
                
                logger.error(
                    "Function execution failed",
                    function_name=name,
                    duration_seconds=duration,
                    error=str(e)
                )
                raise
                
        return wrapper
    return decorator

# Использование
class DealService:
    @measure_time("deal_service.get_deals_list")
    async def get_deals_list(self, stage_id: str = None) -> List[dict]:
        # Реализация метода
        pass
```

### 3. Health check endpoints

```python
# routers/health.py
from fastapi import APIRouter, HTTPException
from ..services.bitrix24_service import Bitrix24Service

router = APIRouter(prefix="/health", tags=["health"])

@router.get("/")
async def health_check():
    """Базовая проверка здоровья приложения"""
    return {"status": "healthy", "timestamp": time.time()}

@router.get("/bitrix24")
async def bitrix24_health_check(
    b24_service: Bitrix24Service = Depends(get_bitrix24_service)
):
    """Проверка подключения к Битрикс24"""
    try:
        # Простой запрос для проверки связи
        result = await b24_service.b24.crm.deals.list(limit=1)
        
        return {
            "status": "healthy",
            "bitrix24_connection": "ok",
            "timestamp": time.time()
        }
        
    except Exception as e:
        raise HTTPException(
            status_code=503,
            detail=f"Bitrix24 connection failed: {str(e)}"
        )
```

---

## 🔧 DevOps и развертывание

### Docker

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Установка зависимостей
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копирование кода
COPY ./app ./app

# Переменные окружения
ENV PYTHONPATH=/app
ENV PYTHONUNBUFFERED=1

# Запуск приложения
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### docker-compose для разработки

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - BITRIX24_WEBHOOK_URL=${BITRIX24_WEBHOOK_URL}
    volumes:
      - ./app:/app/app
    depends_on:
      - redis
    
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

---

## 📚 Специфические инструкции

### Детальные руководства в этой папке:

**➡️ SDK и API интеграция:** [`bitrix24-python-sdk.md`](bitrix24-python-sdk.md)

**➡️ Code Review стандарты:** [`code-review.md`](code-review.md)

---

## ⚠️ Часто встречающиеся проблемы

### 1. Асинхронность и блокирующие операции

**Проблема:** Использование синхронных библиотек в асинхронном коде
**Решение:** Использовать `asyncio.to_thread()` для блокирующих операций

### 2. Управление соединениями

**Проблема:** Слишком много открытых соединений к API
**Решение:** Использовать connection pooling и правильный lifecycle

### 3. Memory leaks в long-running приложениях

**Проблема:** Утечки памяти при длительной работе
**Решение:** Правильное управление объектами, профилирование памяти

---

*Обновлено: 25 ноября 2025*
*Версия: 2.0 - Модульная архитектура знаний*