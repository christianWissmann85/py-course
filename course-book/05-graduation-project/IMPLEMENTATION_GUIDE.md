# PyForge Implementation Guide

## Week-by-Week Development Plan

This guide provides detailed, day-by-day instructions for building PyForge from scratch.

---

## 📅 Week 1: Foundation & Infrastructure

### Day 1: Project Setup & Architecture

#### Morning (4 hours)
**Goal**: Initialize project structure and development environment

```bash
# Create project structure
mkdir pyforge
cd pyforge

# Initialize git repository
git init
git remote add origin <your-repo-url>

# Create service directories
mkdir -p services/{gateway,analyzer,ml,projects,notify}
mkdir -p infrastructure/{docker,k8s,terraform}
mkdir -p cli
mkdir -p docs
mkdir -p tests/integration
```

**Project Structure**:
```
pyforge/
├── services/
│   ├── gateway/          # API Gateway
│   ├── analyzer/         # Code Analysis Service
│   ├── ml/              # Machine Learning Service
│   ├── projects/        # Project Management Service
│   └── notify/          # Notification Service
├── cli/                 # CLI/TUI Application
├── infrastructure/
│   ├── docker/          # Docker configurations
│   ├── k8s/            # Kubernetes manifests
│   └── terraform/      # IaC definitions
├── shared/             # Shared libraries
├── tests/
│   └── integration/    # Integration tests
└── docs/               # Documentation
```

**Initialize Poetry in each service**:
```bash
# For each service directory
cd services/gateway
poetry init
poetry add fastapi uvicorn pydantic python-jose httpx redis asyncpg
poetry add --group dev mypy ruff pytest pytest-cov pytest-asyncio

# Create standard files
touch main.py config.py __init__.py
mkdir -p {api,core,models,services,utils}
```

#### Afternoon (4 hours)
**Goal**: Set up infrastructure and shared configurations

**Docker Compose Setup** (`docker-compose.yml`):
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: pyforge
      POSTGRES_USER: pyforge
      POSTGRES_PASSWORD: ${DB_PASSWORD:-changeme}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./infrastructure/docker/init.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  rabbitmq:
    image: rabbitmq:3-management
    environment:
      RABBITMQ_DEFAULT_USER: pyforge
      RABBITMQ_DEFAULT_PASS: ${MQ_PASSWORD:-changeme}
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: pyforge
      MINIO_ROOT_PASSWORD: ${S3_PASSWORD:-changeme}
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data

volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:
  minio_data:
```

**Shared Configuration** (`shared/config.py`):
```python
from pydantic_settings import BaseSettings
from typing import Optional

class Settings(BaseSettings):
    # Database
    database_url: str = "postgresql+asyncpg://pyforge:changeme@localhost/pyforge"
    
    # Redis
    redis_url: str = "redis://localhost:6379"
    
    # RabbitMQ
    rabbitmq_url: str = "amqp://pyforge:changeme@localhost/"
    
    # S3/MinIO
    s3_endpoint: str = "http://localhost:9000"
    s3_access_key: str = "pyforge"
    s3_secret_key: str = "changeme"
    s3_bucket: str = "pyforge-files"
    
    # Security
    secret_key: str = "your-secret-key-here"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 15
    refresh_token_expire_days: int = 7
    
    # API
    api_v1_prefix: str = "/api/v1"
    
    # Environment
    environment: str = "development"
    debug: bool = True
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

settings = Settings()
```

**Daily Checklist**:
- [ ] Project structure created
- [ ] Git repository initialized
- [ ] Poetry dependencies installed for all services
- [ ] Docker Compose running
- [ ] Shared configuration created
- [ ] Environment variables configured

---

### Day 2: Database Design & ORM Setup

#### Morning (4 hours)
**Goal**: Design and implement database schema

**Database Schema** (`infrastructure/docker/init.sql`):
```sql
-- Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    is_superuser BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Teams table
CREATE TABLE teams (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Team members
CREATE TABLE team_members (
    team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    role VARCHAR(50) NOT NULL DEFAULT 'member',
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (team_id, user_id)
);

-- Projects table
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    owner_id UUID REFERENCES users(id),
    team_id UUID REFERENCES teams(id),
    settings JSONB DEFAULT '{}',
    is_public BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Project files
CREATE TABLE project_files (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    path VARCHAR(500) NOT NULL,
    content TEXT,
    size_bytes INTEGER,
    mime_type VARCHAR(100),
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(project_id, path)
);

-- Analysis results
CREATE TABLE analysis_results (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    file_id UUID REFERENCES project_files(id) ON DELETE CASCADE,
    analyzer_name VARCHAR(100) NOT NULL,
    metrics JSONB NOT NULL,
    issues JSONB DEFAULT '[]',
    suggestions JSONB DEFAULT '[]',
    execution_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ML predictions
CREATE TABLE ml_predictions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    file_id UUID REFERENCES project_files(id) ON DELETE CASCADE,
    model_name VARCHAR(100) NOT NULL,
    model_version VARCHAR(50),
    prediction JSONB NOT NULL,
    confidence FLOAT,
    execution_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Audit logs
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(100),
    resource_id UUID,
    details JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_projects_owner ON projects(owner_id);
CREATE INDEX idx_project_files_project ON project_files(project_id);
CREATE INDEX idx_analysis_results_file ON analysis_results(file_id);
CREATE INDEX idx_ml_predictions_file ON ml_predictions(file_id);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_created ON audit_logs(created_at DESC);

-- Full text search
CREATE INDEX idx_projects_search ON projects USING GIN(
    to_tsvector('english', name || ' ' || COALESCE(description, ''))
);
```

#### Afternoon (4 hours)
**Goal**: Implement SQLAlchemy models and repositories

**SQLAlchemy Models** (`shared/models/base.py`):
```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import declarative_base, sessionmaker
from sqlalchemy import Column, String, Boolean, DateTime, UUID, JSON, ForeignKey, Integer, Float
from datetime import datetime
import uuid

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    email = Column(String(255), unique=True, nullable=False)
    username = Column(String(100), unique=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    is_superuser = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

class Project(Base):
    __tablename__ = "projects"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    name = Column(String(255), nullable=False)
    description = Column(String)
    owner_id = Column(UUID(as_uuid=True), ForeignKey("users.id"))
    team_id = Column(UUID(as_uuid=True), ForeignKey("teams.id"))
    settings = Column(JSON, default=dict)
    is_public = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

class ProjectFile(Base):
    __tablename__ = "project_files"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    project_id = Column(UUID(as_uuid=True), ForeignKey("projects.id"))
    path = Column(String(500), nullable=False)
    content = Column(String)
    size_bytes = Column(Integer)
    mime_type = Column(String(100))
    version = Column(Integer, default=1)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

**Repository Pattern** (`shared/repositories/base.py`):
```python
from typing import Generic, TypeVar, Optional, List
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, update, delete
from shared.models.base import Base

T = TypeVar("T", bound=Base)

class BaseRepository(Generic[T]):
    def __init__(self, model: type[T], session: AsyncSession):
        self.model = model
        self.session = session
    
    async def create(self, **kwargs) -> T:
        instance = self.model(**kwargs)
        self.session.add(instance)
        await self.session.commit()
        await self.session.refresh(instance)
        return instance
    
    async def get(self, id: uuid.UUID) -> Optional[T]:
        result = await self.session.execute(
            select(self.model).where(self.model.id == id)
        )
        return result.scalar_one_or_none()
    
    async def list(self, skip: int = 0, limit: int = 100) -> List[T]:
        result = await self.session.execute(
            select(self.model).offset(skip).limit(limit)
        )
        return result.scalars().all()
    
    async def update(self, id: uuid.UUID, **kwargs) -> Optional[T]:
        await self.session.execute(
            update(self.model).where(self.model.id == id).values(**kwargs)
        )
        await self.session.commit()
        return await self.get(id)
    
    async def delete(self, id: uuid.UUID) -> bool:
        result = await self.session.execute(
            delete(self.model).where(self.model.id == id)
        )
        await self.session.commit()
        return result.rowcount > 0
```

**Daily Checklist**:
- [ ] Database schema created and tested
- [ ] SQLAlchemy models implemented
- [ ] Repository pattern established
- [ ] Database migrations set up
- [ ] Connection pooling configured
- [ ] Basic CRUD operations tested

---

### Day 3: Message Queue & Event System

#### Morning (4 hours)
**Goal**: Set up RabbitMQ and event-driven architecture

**Event System** (`shared/events/base.py`):
```python
from dataclasses import dataclass
from enum import Enum
from typing import Any, Optional
from datetime import datetime
import uuid
import json

class EventType(Enum):
    # Project events
    PROJECT_CREATED = "project.created"
    PROJECT_UPDATED = "project.updated"
    PROJECT_DELETED = "project.deleted"
    
    # Analysis events
    ANALYSIS_REQUESTED = "analysis.requested"
    ANALYSIS_COMPLETED = "analysis.completed"
    ANALYSIS_FAILED = "analysis.failed"
    
    # ML events
    ML_PREDICTION_REQUESTED = "ml.prediction.requested"
    ML_PREDICTION_COMPLETED = "ml.prediction.completed"
    
    # System events
    USER_REGISTERED = "user.registered"
    USER_LOGIN = "user.login"

@dataclass
class Event:
    id: str = None
    type: EventType = None
    payload: dict = None
    metadata: dict = None
    timestamp: datetime = None
    
    def __post_init__(self):
        if self.id is None:
            self.id = str(uuid.uuid4())
        if self.timestamp is None:
            self.timestamp = datetime.utcnow()
        if self.metadata is None:
            self.metadata = {}
    
    def to_json(self) -> str:
        return json.dumps({
            "id": self.id,
            "type": self.type.value,
            "payload": self.payload,
            "metadata": self.metadata,
            "timestamp": self.timestamp.isoformat()
        })
    
    @classmethod
    def from_json(cls, data: str) -> "Event":
        parsed = json.loads(data)
        return cls(
            id=parsed["id"],
            type=EventType(parsed["type"]),
            payload=parsed["payload"],
            metadata=parsed["metadata"],
            timestamp=datetime.fromisoformat(parsed["timestamp"])
        )
```

**RabbitMQ Publisher** (`shared/events/publisher.py`):
```python
import aio_pika
from aio_pika import ExchangeType
from shared.events.base import Event
from shared.config import settings

class EventPublisher:
    def __init__(self):
        self.connection = None
        self.channel = None
        self.exchange = None
    
    async def connect(self):
        self.connection = await aio_pika.connect_robust(
            settings.rabbitmq_url
        )
        self.channel = await self.connection.channel()
        self.exchange = await self.channel.declare_exchange(
            "pyforge.events",
            ExchangeType.TOPIC,
            durable=True
        )
    
    async def publish(self, event: Event):
        if not self.exchange:
            await self.connect()
        
        message = aio_pika.Message(
            body=event.to_json().encode(),
            delivery_mode=aio_pika.DeliveryMode.PERSISTENT,
            content_type="application/json"
        )
        
        routing_key = event.type.value
        await self.exchange.publish(message, routing_key=routing_key)
    
    async def close(self):
        if self.connection:
            await self.connection.close()
```

#### Afternoon (4 hours)
**Goal**: Implement event consumers and handlers

**Event Consumer** (`shared/events/consumer.py`):
```python
import aio_pika
from typing import Callable, Dict, List
from shared.events.base import Event, EventType
import asyncio
import logging

logger = logging.getLogger(__name__)

class EventConsumer:
    def __init__(self, queue_name: str):
        self.queue_name = queue_name
        self.handlers: Dict[EventType, List[Callable]] = {}
        self.connection = None
        self.channel = None
        self.queue = None
    
    def register_handler(self, event_type: EventType, handler: Callable):
        if event_type not in self.handlers:
            self.handlers[event_type] = []
        self.handlers[event_type].append(handler)
    
    async def connect(self):
        self.connection = await aio_pika.connect_robust(
            settings.rabbitmq_url
        )
        self.channel = await self.connection.channel()
        await self.channel.set_qos(prefetch_count=10)
        
        exchange = await self.channel.declare_exchange(
            "pyforge.events",
            ExchangeType.TOPIC,
            durable=True
        )
        
        self.queue = await self.channel.declare_queue(
            f"pyforge.{self.queue_name}",
            durable=True
        )
        
        # Bind to relevant routing keys
        for event_type in self.handlers.keys():
            await self.queue.bind(exchange, routing_key=event_type.value)
    
    async def process_message(self, message: aio_pika.IncomingMessage):
        async with message.process():
            try:
                event = Event.from_json(message.body.decode())
                
                if event.type in self.handlers:
                    for handler in self.handlers[event.type]:
                        await handler(event)
                
            except Exception as e:
                logger.error(f"Error processing message: {e}")
                # Optionally, send to dead letter queue
    
    async def start_consuming(self):
        if not self.queue:
            await self.connect()
        
        await self.queue.consume(self.process_message)
        
        try:
            await asyncio.Future()  # Run forever
        except KeyboardInterrupt:
            await self.stop_consuming()
    
    async def stop_consuming(self):
        if self.connection:
            await self.connection.close()
```

**Example Event Handler** (`services/notify/handlers.py`):
```python
from shared.events.base import Event, EventType
from shared.events.consumer import EventConsumer
import logging

logger = logging.getLogger(__name__)

class NotificationHandlers:
    def __init__(self):
        self.consumer = EventConsumer("notifications")
        self.register_handlers()
    
    def register_handlers(self):
        self.consumer.register_handler(
            EventType.PROJECT_CREATED,
            self.handle_project_created
        )
        self.consumer.register_handler(
            EventType.ANALYSIS_COMPLETED,
            self.handle_analysis_completed
        )
    
    async def handle_project_created(self, event: Event):
        logger.info(f"Project created: {event.payload}")
        # Send email notification
        # Post to Slack
        # Update dashboard
    
    async def handle_analysis_completed(self, event: Event):
        logger.info(f"Analysis completed: {event.payload}")
        # Notify user
        # Update metrics
        # Trigger follow-up actions
    
    async def start(self):
        await self.consumer.start_consuming()
```

**Daily Checklist**:
- [ ] RabbitMQ configured and running
- [ ] Event system implemented
- [ ] Publisher class tested
- [ ] Consumer class tested
- [ ] Event handlers registered
- [ ] Dead letter queue configured

---

### Day 4: Cache Layer & Session Management

#### Morning (4 hours)
**Goal**: Implement Redis caching and session management

**Cache Manager** (`shared/cache/manager.py`):
```python
import redis.asyncio as redis
import json
from typing import Optional, Any
from datetime import timedelta
import pickle

class CacheManager:
    def __init__(self, redis_url: str):
        self.redis_url = redis_url
        self.client = None
    
    async def connect(self):
        self.client = redis.from_url(self.redis_url, decode_responses=False)
    
    async def get(self, key: str) -> Optional[Any]:
        if not self.client:
            await self.connect()
        
        value = await self.client.get(key)
        if value:
            try:
                return json.loads(value)
            except json.JSONDecodeError:
                return pickle.loads(value)
        return None
    
    async def set(
        self, 
        key: str, 
        value: Any, 
        ttl: Optional[int] = None
    ):
        if not self.client:
            await self.connect()
        
        try:
            serialized = json.dumps(value)
        except (TypeError, ValueError):
            serialized = pickle.dumps(value)
        
        if ttl:
            await self.client.setex(key, ttl, serialized)
        else:
            await self.client.set(key, serialized)
    
    async def delete(self, key: str):
        if not self.client:
            await self.connect()
        await self.client.delete(key)
    
    async def exists(self, key: str) -> bool:
        if not self.client:
            await self.connect()
        return await self.client.exists(key) > 0
    
    async def expire(self, key: str, ttl: int):
        if not self.client:
            await self.connect()
        await self.client.expire(key, ttl)
    
    async def increment(self, key: str) -> int:
        if not self.client:
            await self.connect()
        return await self.client.incr(key)
    
    async def close(self):
        if self.client:
            await self.client.close()
```

**Session Manager** (`shared/auth/session.py`):
```python
from typing import Optional
from datetime import datetime, timedelta
import uuid
from shared.cache.manager import CacheManager

class SessionManager:
    def __init__(self, cache: CacheManager):
        self.cache = cache
        self.session_ttl = 3600  # 1 hour
    
    async def create_session(self, user_id: str, data: dict = None) -> str:
        session_id = str(uuid.uuid4())
        session_data = {
            "user_id": user_id,
            "created_at": datetime.utcnow().isoformat(),
            "data": data or {}
        }
        
        key = f"session:{session_id}"
        await self.cache.set(key, session_data, ttl=self.session_ttl)
        
        return session_id
    
    async def get_session(self, session_id: str) -> Optional[dict]:
        key = f"session:{session_id}"
        session_data = await self.cache.get(key)
        
        if session_data:
            # Refresh TTL on access
            await self.cache.expire(key, self.session_ttl)
        
        return session_data
    
    async def update_session(self, session_id: str, data: dict):
        session_data = await self.get_session(session_id)
        if session_data:
            session_data["data"].update(data)
            key = f"session:{session_id}"
            await self.cache.set(key, session_data, ttl=self.session_ttl)
    
    async def delete_session(self, session_id: str):
        key = f"session:{session_id}"
        await self.cache.delete(key)
```

#### Afternoon (4 hours)
**Goal**: Implement rate limiting and caching decorators

**Rate Limiter** (`shared/middleware/rate_limit.py`):
```python
from fastapi import HTTPException, Request
from shared.cache.manager import CacheManager
from typing import Optional

class RateLimiter:
    def __init__(
        self, 
        cache: CacheManager,
        requests_per_minute: int = 60,
        requests_per_hour: int = 1000
    ):
        self.cache = cache
        self.rpm_limit = requests_per_minute
        self.rph_limit = requests_per_hour
    
    async def check_rate_limit(
        self, 
        identifier: str,
        endpoint: Optional[str] = None
    ) -> bool:
        # Check per-minute limit
        minute_key = f"rate_limit:minute:{identifier}"
        if endpoint:
            minute_key += f":{endpoint}"
        
        minute_count = await self.cache.increment(minute_key)
        if minute_count == 1:
            await self.cache.expire(minute_key, 60)
        
        if minute_count > self.rpm_limit:
            return False
        
        # Check per-hour limit
        hour_key = f"rate_limit:hour:{identifier}"
        if endpoint:
            hour_key += f":{endpoint}"
        
        hour_count = await self.cache.increment(hour_key)
        if hour_count == 1:
            await self.cache.expire(hour_key, 3600)
        
        if hour_count > self.rph_limit:
            return False
        
        return True
    
    async def __call__(self, request: Request):
        # Extract identifier (user_id or IP)
        user_id = getattr(request.state, "user_id", None)
        identifier = user_id or request.client.host
        
        endpoint = request.url.path
        
        if not await self.check_rate_limit(identifier, endpoint):
            raise HTTPException(
                status_code=429,
                detail="Rate limit exceeded"
            )
```

**Cache Decorator** (`shared/decorators/cache.py`):
```python
from functools import wraps
import hashlib
import json
from typing import Optional

def cached(ttl: Optional[int] = 300):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Generate cache key from function name and arguments
            cache_key = generate_cache_key(func.__name__, args, kwargs)
            
            # Try to get from cache
            cached_result = await cache_manager.get(cache_key)
            if cached_result is not None:
                return cached_result
            
            # Execute function
            result = await func(*args, **kwargs)
            
            # Store in cache
            await cache_manager.set(cache_key, result, ttl=ttl)
            
            return result
        
        return wrapper
    return decorator

def generate_cache_key(func_name: str, args: tuple, kwargs: dict) -> str:
    key_parts = [func_name]
    
    # Add args
    for arg in args:
        if hasattr(arg, "__dict__"):
            # Skip object arguments
            continue
        key_parts.append(str(arg))
    
    # Add kwargs
    for k, v in sorted(kwargs.items()):
        if hasattr(v, "__dict__"):
            continue
        key_parts.append(f"{k}={v}")
    
    key_string = ":".join(key_parts)
    
    # Hash if too long
    if len(key_string) > 200:
        key_string = hashlib.md5(key_string.encode()).hexdigest()
    
    return f"cache:{key_string}"
```

**Daily Checklist**:
- [ ] Redis cache manager implemented
- [ ] Session management working
- [ ] Rate limiting tested
- [ ] Cache decorators functional
- [ ] Cache invalidation strategies defined
- [ ] Performance benchmarks established

---

### Day 5: Object Storage & File Management

#### Morning (4 hours)
**Goal**: Set up MinIO and implement file storage

**S3 Storage Manager** (`shared/storage/s3.py`):
```python
import aioboto3
from typing import Optional, BinaryIO
import uuid
from pathlib import Path

class S3Storage:
    def __init__(
        self,
        endpoint: str,
        access_key: str,
        secret_key: str,
        bucket: str
    ):
        self.endpoint = endpoint
        self.access_key = access_key
        self.secret_key = secret_key
        self.bucket = bucket
        self.session = aioboto3.Session()
    
    async def upload_file(
        self,
        file_content: bytes,
        key: str,
        content_type: Optional[str] = None
    ) -> str:
        async with self.session.client(
            's3',
            endpoint_url=self.endpoint,
            aws_access_key_id=self.access_key,
            aws_secret_access_key=self.secret_key
        ) as s3:
            extra_args = {}
            if content_type:
                extra_args['ContentType'] = content_type
            
            await s3.put_object(
                Bucket=self.bucket,
                Key=key,
                Body=file_content,
                **extra_args
            )
            
            return f"{self.endpoint}/{self.bucket}/{key}"
    
    async def download_file(self, key: str) -> bytes:
        async with self.session.client(
            's3',
            endpoint_url=self.endpoint,
            aws_access_key_id=self.access_key,
            aws_secret_access_key=self.secret_key
        ) as s3:
            response = await s3.get_object(
                Bucket=self.bucket,
                Key=key
            )
            return await response['Body'].read()
    
    async def delete_file(self, key: str):
        async with self.session.client(
            's3',
            endpoint_url=self.endpoint,
            aws_access_key_id=self.access_key,
            aws_secret_access_key=self.secret_key
        ) as s3:
            await s3.delete_object(
                Bucket=self.bucket,
                Key=key
            )
    
    async def list_files(self, prefix: str = "") -> list:
        async with self.session.client(
            's3',
            endpoint_url=self.endpoint,
            aws_access_key_id=self.access_key,
            aws_secret_access_key=self.secret_key
        ) as s3:
            paginator = s3.get_paginator('list_objects_v2')
            pages = paginator.paginate(
                Bucket=self.bucket,
                Prefix=prefix
            )
            
            files = []
            async for page in pages:
                if 'Contents' in page:
                    for obj in page['Contents']:
                        files.append({
                            'key': obj['Key'],
                            'size': obj['Size'],
                            'last_modified': obj['LastModified']
                        })
            
            return files
    
    async def generate_presigned_url(
        self,
        key: str,
        expires_in: int = 3600
    ) -> str:
        async with self.session.client(
            's3',
            endpoint_url=self.endpoint,
            aws_access_key_id=self.access_key,
            aws_secret_access_key=self.secret_key
        ) as s3:
            return await s3.generate_presigned_url(
                'get_object',
                Params={'Bucket': self.bucket, 'Key': key},
                ExpiresIn=expires_in
            )
```

#### Afternoon (4 hours)
**Goal**: Implement file processing pipeline

**File Processor** (`shared/storage/processor.py`):
```python
import hashlib
import mimetypes
from typing import Optional, Tuple
from PIL import Image
import io

class FileProcessor:
    def __init__(self, storage: S3Storage):
        self.storage = storage
    
    async def process_upload(
        self,
        file_content: bytes,
        filename: str,
        project_id: str
    ) -> dict:
        # Calculate file hash
        file_hash = hashlib.sha256(file_content).hexdigest()
        
        # Detect MIME type
        mime_type, _ = mimetypes.guess_type(filename)
        
        # Generate storage key
        key = f"projects/{project_id}/{file_hash[:2]}/{file_hash}/{filename}"
        
        # Process based on file type
        if mime_type and mime_type.startswith('image/'):
            file_content, metadata = await self.process_image(file_content)
        else:
            metadata = {}
        
        # Upload to S3
        url = await self.storage.upload_file(
            file_content,
            key,
            content_type=mime_type
        )
        
        return {
            'url': url,
            'key': key,
            'filename': filename,
            'size': len(file_content),
            'mime_type': mime_type,
            'hash': file_hash,
            'metadata': metadata
        }
    
    async def process_image(
        self,
        image_content: bytes
    ) -> Tuple[bytes, dict]:
        # Open image
        img = Image.open(io.BytesIO(image_content))
        
        # Get metadata
        metadata = {
            'width': img.width,
            'height': img.height,
            'format': img.format,
            'mode': img.mode
        }
        
        # Optimize if needed
        if img.width > 1920 or img.height > 1080:
            img.thumbnail((1920, 1080), Image.Resampling.LANCZOS)
            
            # Convert back to bytes
            buffer = io.BytesIO()
            img.save(buffer, format=img.format or 'PNG', optimize=True)
            image_content = buffer.getvalue()
        
        return image_content, metadata
```

**Daily Checklist**:
- [ ] MinIO configured and running
- [ ] S3 storage manager implemented
- [ ] File upload/download working
- [ ] File processing pipeline created
- [ ] Presigned URLs functional
- [ ] Storage tests written

**Week 1 Summary**:
- Infrastructure fully configured
- Database design implemented
- Message queue system operational
- Caching layer functional
- Object storage ready
- All foundational components tested

---

## 📅 Week 2: Service Implementation

### Day 1: API Gateway Service

#### Morning (4 hours)
**Goal**: Implement FastAPI gateway with authentication

**Gateway Main** (`services/gateway/main.py`):
```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager
import uvicorn

from shared.config import settings
from shared.cache.manager import CacheManager
from shared.events.publisher import EventPublisher
from .routers import auth, projects, analysis, ml
from .middleware import RateLimitMiddleware, LoggingMiddleware
from .dependencies import get_current_user

# Lifespan context manager
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    app.state.cache = CacheManager(settings.redis_url)
    await app.state.cache.connect()
    
    app.state.publisher = EventPublisher()
    await app.state.publisher.connect()
    
    yield
    
    # Shutdown
    await app.state.cache.close()
    await app.state.publisher.close()

# Create app
app = FastAPI(
    title="PyForge API Gateway",
    version="1.0.0",
    lifespan=lifespan
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Custom middleware
app.add_middleware(RateLimitMiddleware)
app.add_middleware(LoggingMiddleware)

# Include routers
app.include_router(
    auth.router,
    prefix="/auth",
    tags=["Authentication"]
)
app.include_router(
    projects.router,
    prefix=f"{settings.api_v1_prefix}/projects",
    tags=["Projects"],
    dependencies=[Depends(get_current_user)]
)
app.include_router(
    analysis.router,
    prefix=f"{settings.api_v1_prefix}/analysis",
    tags=["Analysis"],
    dependencies=[Depends(get_current_user)]
)
app.include_router(
    ml.router,
    prefix=f"{settings.api_v1_prefix}/ml",
    tags=["Machine Learning"],
    dependencies=[Depends(get_current_user)]
)

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

@app.get("/ready")
async def readiness_check():
    # Check all dependencies
    try:
        # Check cache
        await app.state.cache.get("test")
        # Add more checks as needed
        return {"status": "ready"}
    except Exception as e:
        raise HTTPException(status_code=503, detail=str(e))

if __name__ == "__main__":
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=settings.debug
    )
```

**Authentication Router** (`services/gateway/routers/auth.py`):
```python
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel, EmailStr
from typing import Optional
from datetime import datetime, timedelta
import bcrypt
import jwt

from shared.config import settings
from shared.models.base import User
from shared.repositories.base import BaseRepository
from ..dependencies import get_db

router = APIRouter()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

class UserCreate(BaseModel):
    email: EmailStr
    username: str
    password: str

class Token(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"

class TokenData(BaseModel):
    user_id: Optional[str] = None

def hash_password(password: str) -> str:
    salt = bcrypt.gensalt()
    return bcrypt.hashpw(password.encode('utf-8'), salt).decode('utf-8')

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return bcrypt.checkpw(
        plain_password.encode('utf-8'),
        hashed_password.encode('utf-8')
    )

def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(
        minutes=settings.access_token_expire_minutes
    )
    to_encode.update({"exp": expire, "type": "access"})
    return jwt.encode(
        to_encode,
        settings.secret_key,
        algorithm=settings.algorithm
    )

def create_refresh_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(
        days=settings.refresh_token_expire_days
    )
    to_encode.update({"exp": expire, "type": "refresh"})
    return jwt.encode(
        to_encode,
        settings.secret_key,
        algorithm=settings.algorithm
    )

@router.post("/register", response_model=Token)
async def register(
    user_data: UserCreate,
    db = Depends(get_db)
):
    # Check if user exists
    user_repo = BaseRepository(User, db)
    
    # Hash password
    hashed_password = hash_password(user_data.password)
    
    # Create user
    user = await user_repo.create(
        email=user_data.email,
        username=user_data.username,
        password_hash=hashed_password
    )
    
    # Generate tokens
    access_token = create_access_token({"sub": str(user.id)})
    refresh_token = create_refresh_token({"sub": str(user.id)})
    
    return Token(
        access_token=access_token,
        refresh_token=refresh_token
    )

@router.post("/token", response_model=Token)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db = Depends(get_db)
):
    # Authenticate user
    user_repo = BaseRepository(User, db)
    
    # Find user by username/email
    # ... (implement user lookup)
    
    # Verify password
    if not verify_password(form_data.password, user.password_hash):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password"
        )
    
    # Generate tokens
    access_token = create_access_token({"sub": str(user.id)})
    refresh_token = create_refresh_token({"sub": str(user.id)})
    
    return Token(
        access_token=access_token,
        refresh_token=refresh_token
    )

@router.post("/refresh", response_model=Token)
async def refresh_token(
    token: str,
    db = Depends(get_db)
):
    try:
        payload = jwt.decode(
            token,
            settings.secret_key,
            algorithms=[settings.algorithm]
        )
        
        if payload.get("type") != "refresh":
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Invalid token type"
            )
        
        user_id = payload.get("sub")
        
        # Generate new tokens
        access_token = create_access_token({"sub": user_id})
        refresh_token = create_refresh_token({"sub": user_id})
        
        return Token(
            access_token=access_token,
            refresh_token=refresh_token
        )
        
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token has expired"
        )
    except jwt.JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Could not validate credentials"
        )
```

#### Afternoon (4 hours)
**Goal**: Implement project and analysis routers

**Projects Router** (`services/gateway/routers/projects.py`):
```python
from fastapi import APIRouter, Depends, HTTPException, UploadFile, File
from pydantic import BaseModel
from typing import List, Optional
import uuid

from shared.models.base import Project, ProjectFile
from shared.events.base import Event, EventType
from ..dependencies import get_current_user, get_db, get_publisher

router = APIRouter()

class ProjectCreate(BaseModel):
    name: str
    description: Optional[str] = None
    settings: dict = {}

class ProjectResponse(BaseModel):
    id: str
    name: str
    description: Optional[str]
    owner_id: str
    settings: dict
    created_at: datetime
    updated_at: datetime

@router.get("/", response_model=List[ProjectResponse])
async def list_projects(
    skip: int = 0,
    limit: int = 100,
    current_user = Depends(get_current_user),
    db = Depends(get_db)
):
    # List user's projects
    projects = await db.execute(
        select(Project).where(
            Project.owner_id == current_user.id
        ).offset(skip).limit(limit)
    )
    return projects.scalars().all()

@router.post("/", response_model=ProjectResponse)
async def create_project(
    project_data: ProjectCreate,
    current_user = Depends(get_current_user),
    db = Depends(get_db),
    publisher = Depends(get_publisher)
):
    # Create project
    project = Project(
        name=project_data.name,
        description=project_data.description,
        owner_id=current_user.id,
        settings=project_data.settings
    )
    
    db.add(project)
    await db.commit()
    await db.refresh(project)
    
    # Publish event
    await publisher.publish(
        Event(
            type=EventType.PROJECT_CREATED,
            payload={
                "project_id": str(project.id),
                "name": project.name,
                "owner_id": str(current_user.id)
            }
        )
    )
    
    return project

@router.get("/{project_id}", response_model=ProjectResponse)
async def get_project(
    project_id: uuid.UUID,
    current_user = Depends(get_current_user),
    db = Depends(get_db)
):
    # Get project
    project = await db.get(Project, project_id)
    
    if not project:
        raise HTTPException(status_code=404, detail="Project not found")
    
    # Check permissions
    if project.owner_id != current_user.id:
        raise HTTPException(status_code=403, detail="Access denied")
    
    return project

@router.post("/{project_id}/files")
async def upload_file(
    project_id: uuid.UUID,
    file: UploadFile = File(...),
    current_user = Depends(get_current_user),
    db = Depends(get_db),
    storage = Depends(get_storage)
):
    # Check project ownership
    project = await db.get(Project, project_id)
    if not project or project.owner_id != current_user.id:
        raise HTTPException(status_code=403, detail="Access denied")
    
    # Read file content
    content = await file.read()
    
    # Process and store file
    file_info = await storage.process_upload(
        content,
        file.filename,
        str(project_id)
    )
    
    # Save file record
    project_file = ProjectFile(
        project_id=project_id,
        path=file.filename,
        size_bytes=file_info['size'],
        mime_type=file_info['mime_type']
    )
    
    db.add(project_file)
    await db.commit()
    
    return {
        "file_id": str(project_file.id),
        "url": file_info['url'],
        "size": file_info['size']
    }
```

**Daily Checklist**:
- [ ] API Gateway service running
- [ ] Authentication endpoints working
- [ ] JWT tokens implemented
- [ ] Project CRUD operations functional
- [ ] File upload working
- [ ] Rate limiting active

---

### Day 2: Analysis Service Implementation

#### Morning (4 hours)
**Goal**: Build code analysis service

**Analysis Service** (`services/analyzer/main.py`):
```python
import asyncio
from typing import Dict, Any
import ast
import sys
from pathlib import Path

from shared.events.consumer import EventConsumer
from shared.events.base import Event, EventType
from .analyzers import (
    ComplexityAnalyzer,
    TypeAnalyzer,
    SecurityAnalyzer,
    StyleAnalyzer
)

class AnalysisService:
    def __init__(self):
        self.consumer = EventConsumer("analysis")
        self.analyzers = {
            'complexity': ComplexityAnalyzer(),
            'types': TypeAnalyzer(),
            'security': SecurityAnalyzer(),
            'style': StyleAnalyzer()
        }
        self.register_handlers()
    
    def register_handlers(self):
        self.consumer.register_handler(
            EventType.ANALYSIS_REQUESTED,
            self.handle_analysis_request
        )
    
    async def handle_analysis_request(self, event: Event):
        payload = event.payload
        code = payload['code']
        language = payload.get('language', 'python')
        file_id = payload['file_id']
        
        # Parse code
        try:
            tree = ast.parse(code)
        except SyntaxError as e:
            # Handle syntax error
            await self.publish_error(file_id, str(e))
            return
        
        # Run analyzers
        results = {}
        for name, analyzer in self.analyzers.items():
            try:
                results[name] = await analyzer.analyze(tree, code)
            except Exception as e:
                results[name] = {'error': str(e)}
        
        # Aggregate results
        analysis_result = self.aggregate_results(results)
        
        # Store in database
        await self.store_results(file_id, analysis_result)
        
        # Publish completion event
        await self.publish_completion(file_id, analysis_result)
    
    def aggregate_results(self, results: Dict[str, Any]) -> dict:
        return {
            'metrics': {
                'complexity': results.get('complexity', {}).get('score', 0),
                'type_coverage': results.get('types', {}).get('coverage', 0),
                'security_score': results.get('security', {}).get('score', 100),
                'style_score': results.get('style', {}).get('score', 0)
            },
            'issues': self.collect_issues(results),
            'suggestions': self.collect_suggestions(results)
        }
    
    def collect_issues(self, results):
        issues = []
        for analyzer_results in results.values():
            if 'issues' in analyzer_results:
                issues.extend(analyzer_results['issues'])
        return issues
    
    def collect_suggestions(self, results):
        suggestions = []
        for analyzer_results in results.values():
            if 'suggestions' in analyzer_results:
                suggestions.extend(analyzer_results['suggestions'])
        return suggestions
    
    async def start(self):
        await self.consumer.start_consuming()

if __name__ == "__main__":
    service = AnalysisService()
    asyncio.run(service.start())
```

**Complexity Analyzer** (`services/analyzer/analyzers/complexity.py`):
```python
import ast
from typing import Dict, Any

class ComplexityAnalyzer:
    def __init__(self):
        self.complexity_threshold = 10
    
    async def analyze(self, tree: ast.AST, code: str) -> Dict[str, Any]:
        visitor = ComplexityVisitor()
        visitor.visit(tree)
        
        return {
            'score': visitor.total_complexity,
            'functions': visitor.function_complexities,
            'issues': self.identify_issues(visitor),
            'suggestions': self.generate_suggestions(visitor)
        }
    
    def identify_issues(self, visitor):
        issues = []
        for func_name, complexity in visitor.function_complexities.items():
            if complexity > self.complexity_threshold:
                issues.append({
                    'type': 'high_complexity',
                    'severity': 'warning',
                    'message': f"Function '{func_name}' has high complexity ({complexity})",
                    'line': visitor.function_lines.get(func_name, 0)
                })
        return issues
    
    def generate_suggestions(self, visitor):
        suggestions = []
        for func_name, complexity in visitor.function_complexities.items():
            if complexity > self.complexity_threshold:
                suggestions.append({
                    'type': 'refactor',
                    'message': f"Consider breaking down '{func_name}' into smaller functions"
                })
        return suggestions

class ComplexityVisitor(ast.NodeVisitor):
    def __init__(self):
        self.total_complexity = 0
        self.function_complexities = {}
        self.function_lines = {}
        self.current_function = None
        self.current_complexity = 0
    
    def visit_FunctionDef(self, node):
        # Save previous context
        prev_function = self.current_function
        prev_complexity = self.current_complexity
        
        # Set new context
        self.current_function = node.name
        self.current_complexity = 1  # Base complexity
        self.function_lines[node.name] = node.lineno
        
        # Visit function body
        self.generic_visit(node)
        
        # Save results
        self.function_complexities[node.name] = self.current_complexity
        self.total_complexity += self.current_complexity
        
        # Restore context
        self.current_function = prev_function
        self.current_complexity = prev_complexity
    
    def visit_If(self, node):
        self.current_complexity += 1
        self.generic_visit(node)
    
    def visit_While(self, node):
        self.current_complexity += 1
        self.generic_visit(node)
    
    def visit_For(self, node):
        self.current_complexity += 1
        self.generic_visit(node)
    
    def visit_ExceptHandler(self, node):
        self.current_complexity += 1
        self.generic_visit(node)
```

#### Afternoon (4 hours)
**Goal**: Implement remaining analyzers

**Type Analyzer** (`services/analyzer/analyzers/types.py`):
```python
import ast
from typing import Dict, Any

class TypeAnalyzer:
    async def analyze(self, tree: ast.AST, code: str) -> Dict[str, Any]:
        visitor = TypeHintVisitor()
        visitor.visit(tree)
        
        total_items = visitor.total_functions + visitor.total_arguments
        typed_items = visitor.typed_functions + visitor.typed_arguments
        
        coverage = (typed_items / total_items * 100) if total_items > 0 else 100
        
        return {
            'coverage': coverage,
            'total_functions': visitor.total_functions,
            'typed_functions': visitor.typed_functions,
            'total_arguments': visitor.total_arguments,
            'typed_arguments': visitor.typed_arguments,
            'issues': self.identify_issues(visitor),
            'suggestions': self.generate_suggestions(visitor)
        }
    
    def identify_issues(self, visitor):
        issues = []
        for func_name in visitor.untyped_functions:
            issues.append({
                'type': 'missing_type_hint',
                'severity': 'info',
                'message': f"Function '{func_name}' lacks return type annotation"
            })
        return issues
    
    def generate_suggestions(self, visitor):
        if visitor.typed_functions < visitor.total_functions:
            return [{
                'type': 'type_hints',
                'message': 'Add type hints to improve code clarity and catch bugs early'
            }]
        return []

class TypeHintVisitor(ast.NodeVisitor):
    def __init__(self):
        self.total_functions = 0
        self.typed_functions = 0
        self.total_arguments = 0
        self.typed_arguments = 0
        self.untyped_functions = []
    
    def visit_FunctionDef(self, node):
        self.total_functions += 1
        
        # Check return type
        if node.returns:
            self.typed_functions += 1
        else:
            self.untyped_functions.append(node.name)
        
        # Check arguments
        for arg in node.args.args:
            self.total_arguments += 1
            if arg.annotation:
                self.typed_arguments += 1
        
        self.generic_visit(node)
```

**Daily Checklist**:
- [ ] Analysis service running
- [ ] Event consumer working
- [ ] All analyzers implemented
- [ ] Results aggregation tested
- [ ] Database storage working
- [ ] Analysis events published

---

### Day 3: ML Service Implementation

#### Morning (4 hours)
**Goal**: Set up ML models and training pipeline

**ML Models** (`services/ml/models/quality.py`):
```python
import torch
import torch.nn as nn
from transformers import AutoTokenizer, AutoModel
from typing import List, Tuple

class CodeQualityModel(nn.Module):
    def __init__(self, pretrained_model: str = "microsoft/codebert-base"):
        super().__init__()
        self.encoder = AutoModel.from_pretrained(pretrained_model)
        self.tokenizer = AutoTokenizer.from_pretrained(pretrained_model)
        
        # Freeze encoder layers initially
        for param in self.encoder.parameters():
            param.requires_grad = False
        
        # Classification head
        self.dropout = nn.Dropout(0.1)
        self.classifier = nn.Sequential(
            nn.Linear(768, 256),
            nn.ReLU(),
            nn.Dropout(0.1),
            nn.Linear(256, 64),
            nn.ReLU(),
            nn.Linear(64, 5)  # 5 quality levels
        )
    
    def forward(self, input_ids, attention_mask):
        outputs = self.encoder(
            input_ids=input_ids,
            attention_mask=attention_mask
        )
        
        # Use [CLS] token representation
        pooled = outputs.last_hidden_state[:, 0, :]
        pooled = self.dropout(pooled)
        
        logits = self.classifier(pooled)
        return logits
    
    def predict(self, code: str) -> Tuple[int, float]:
        # Tokenize
        inputs = self.tokenizer(
            code,
            return_tensors="pt",
            max_length=512,
            truncation=True,
            padding=True
        )
        
        # Forward pass
        with torch.no_grad():
            logits = self.forward(
                inputs['input_ids'],
                inputs['attention_mask']
            )
            probs = torch.softmax(logits, dim=-1)
            
            # Get prediction and confidence
            confidence, prediction = torch.max(probs, dim=-1)
            
        return prediction.item(), confidence.item()

class RefactoringSuggester(nn.Module):
    def __init__(self):
        super().__init__()
        # Implement sequence-to-sequence model
        # for code refactoring suggestions
        pass
    
    def suggest(self, code: str) -> List[dict]:
        # Generate refactoring suggestions
        return [
            {
                'type': 'extract_method',
                'description': 'Extract complex logic into separate method',
                'line_start': 10,
                'line_end': 25
            }
        ]
```

#### Afternoon (4 hours)
**Goal**: Implement ML service API

**ML Service** (`services/ml/main.py`):
```python
from fastapi import FastAPI
import torch
from contextlib import asynccontextmanager

from .models.quality import CodeQualityModel, RefactoringSuggester
from .models.bug_detector import BugDetector
from shared.events.consumer import EventConsumer
from shared.events.base import Event, EventType

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Load models
    app.state.quality_model = CodeQualityModel()
    app.state.quality_model.load_state_dict(
        torch.load("models/quality_model.pt", map_location="cpu")
    )
    app.state.quality_model.eval()
    
    app.state.refactoring_model = RefactoringSuggester()
    app.state.bug_detector = BugDetector()
    
    # Start event consumer
    app.state.consumer = EventConsumer("ml")
    app.state.consumer.register_handler(
        EventType.ML_PREDICTION_REQUESTED,
        handle_prediction_request
    )
    
    # Start consuming in background
    asyncio.create_task(app.state.consumer.start_consuming())
    
    yield
    
    # Cleanup
    await app.state.consumer.stop_consuming()

app = FastAPI(
    title="PyForge ML Service",
    version="1.0.0",
    lifespan=lifespan
)

@app.post("/predict/quality")
async def predict_quality(code: str):
    prediction, confidence = app.state.quality_model.predict(code)
    
    quality_levels = ["Very Poor", "Poor", "Average", "Good", "Excellent"]
    
    return {
        "quality_score": prediction * 20,  # Convert to 0-100
        "quality_level": quality_levels[prediction],
        "confidence": confidence
    }

@app.post("/suggest/refactoring")
async def suggest_refactoring(code: str):
    suggestions = app.state.refactoring_model.suggest(code)
    
    return {
        "suggestions": suggestions,
        "total": len(suggestions)
    }

@app.post("/detect/bugs")
async def detect_bugs(code: str):
    bugs = app.state.bug_detector.detect(code)
    
    return {
        "bugs": bugs,
        "total": len(bugs),
        "severity_counts": count_severities(bugs)
    }

async def handle_prediction_request(event: Event):
    payload = event.payload
    model_type = payload['model_type']
    code = payload['code']
    file_id = payload['file_id']
    
    if model_type == 'quality':
        result = await predict_quality(code)
    elif model_type == 'refactoring':
        result = await suggest_refactoring(code)
    elif model_type == 'bugs':
        result = await detect_bugs(code)
    else:
        result = {'error': f'Unknown model type: {model_type}'}
    
    # Store result in database
    await store_prediction(file_id, model_type, result)
    
    # Publish completion event
    await publish_prediction_complete(file_id, model_type, result)
```

**Daily Checklist**:
- [ ] ML models implemented
- [ ] Model loading functional
- [ ] Prediction endpoints working
- [ ] Event handling integrated
- [ ] Model performance acceptable
- [ ] GPU support configured (if available)

---

### Day 4: Notification Service & Integration

#### Morning (4 hours)
**Goal**: Build notification service

**Notification Service** (`services/notify/main.py`):
```python
import asyncio
from typing import Dict, List
import aiosmtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import httpx
from jinja2 import Environment, FileSystemLoader

from shared.events.consumer import EventConsumer
from shared.events.base import Event, EventType
from shared.config import settings

class NotificationService:
    def __init__(self):
        self.consumer = EventConsumer("notifications")
        self.channels = {
            'email': EmailChannel(),
            'slack': SlackChannel(),
            'webhook': WebhookChannel()
        }
        self.template_env = Environment(
            loader=FileSystemLoader('templates')
        )
        self.register_handlers()
    
    def register_handlers(self):
        # Register for all relevant events
        for event_type in [
            EventType.PROJECT_CREATED,
            EventType.ANALYSIS_COMPLETED,
            EventType.ML_PREDICTION_COMPLETED
        ]:
            self.consumer.register_handler(
                event_type,
                self.handle_event
            )
    
    async def handle_event(self, event: Event):
        # Get user preferences
        user_id = event.metadata.get('user_id')
        preferences = await self.get_user_preferences(user_id)
        
        # Prepare notification
        notification = self.prepare_notification(event)
        
        # Send through enabled channels
        tasks = []
        for channel_name in preferences.enabled_channels:
            if channel_name in self.channels:
                channel = self.channels[channel_name]
                tasks.append(channel.send(notification, preferences))
        
        await asyncio.gather(*tasks)
    
    def prepare_notification(self, event: Event) -> dict:
        template_map = {
            EventType.PROJECT_CREATED: 'project_created.html',
            EventType.ANALYSIS_COMPLETED: 'analysis_completed.html',
            EventType.ML_PREDICTION_COMPLETED: 'ml_completed.html'
        }
        
        template_name = template_map.get(event.type)
        if template_name:
            template = self.template_env.get_template(template_name)
            html_content = template.render(**event.payload)
        else:
            html_content = str(event.payload)
        
        return {
            'subject': f"PyForge: {event.type.value}",
            'html': html_content,
            'text': html_content,  # TODO: Generate plain text version
            'metadata': event.metadata
        }
    
    async def start(self):
        await self.consumer.start_consuming()

class EmailChannel:
    def __init__(self):
        self.smtp_host = settings.smtp_host
        self.smtp_port = settings.smtp_port
        self.smtp_user = settings.smtp_user
        self.smtp_password = settings.smtp_password
    
    async def send(self, notification: dict, preferences: dict):
        message = MIMEMultipart('alternative')
        message['Subject'] = notification['subject']
        message['From'] = self.smtp_user
        message['To'] = preferences['email']
        
        # Add text and HTML parts
        text_part = MIMEText(notification['text'], 'plain')
        html_part = MIMEText(notification['html'], 'html')
        
        message.attach(text_part)
        message.attach(html_part)
        
        # Send email
        await aiosmtplib.send(
            message,
            hostname=self.smtp_host,
            port=self.smtp_port,
            username=self.smtp_user,
            password=self.smtp_password
        )

class SlackChannel:
    def __init__(self):
        self.webhook_url = settings.slack_webhook_url
    
    async def send(self, notification: dict, preferences: dict):
        async with httpx.AsyncClient() as client:
            await client.post(
                self.webhook_url,
                json={
                    'text': notification['subject'],
                    'attachments': [{
                        'color': 'good',
                        'text': notification['text']
                    }]
                }
            )

class WebhookChannel:
    async def send(self, notification: dict, preferences: dict):
        webhook_url = preferences.get('webhook_url')
        if webhook_url:
            async with httpx.AsyncClient() as client:
                await client.post(
                    webhook_url,
                    json=notification
                )

if __name__ == "__main__":
    service = NotificationService()
    asyncio.run(service.start())
```

#### Afternoon (4 hours)
**Goal**: Service integration and testing

**Integration Tests** (`tests/integration/test_services.py`):
```python
import pytest
import asyncio
from httpx import AsyncClient
from shared.events.publisher import EventPublisher
from shared.events.base import Event, EventType

@pytest.mark.asyncio
async def test_full_analysis_flow():
    # 1. Upload file through gateway
    async with AsyncClient(base_url="http://localhost:8000") as client:
        # Login
        login_response = await client.post(
            "/auth/token",
            data={"username": "test", "password": "test"}
        )
        token = login_response.json()["access_token"]
        headers = {"Authorization": f"Bearer {token}"}
        
        # Create project
        project_response = await client.post(
            "/api/v1/projects",
            json={"name": "Test Project"},
            headers=headers
        )
        project_id = project_response.json()["id"]
        
        # Upload file
        files = {"file": ("test.py", b"def test():\n    return 42")}
        upload_response = await client.post(
            f"/api/v1/projects/{project_id}/files",
            files=files,
            headers=headers
        )
        file_id = upload_response.json()["file_id"]
        
        # Request analysis
        analysis_response = await client.post(
            f"/api/v1/analyze/{file_id}",
            headers=headers
        )
        
        # Wait for analysis to complete
        await asyncio.sleep(2)
        
        # Check results
        results_response = await client.get(
            f"/api/v1/analyze/{file_id}/results",
            headers=headers
        )
        
        assert results_response.status_code == 200
        results = results_response.json()
        assert "metrics" in results
        assert results["metrics"]["complexity"] > 0

@pytest.mark.asyncio
async def test_ml_predictions():
    async with AsyncClient(base_url="http://localhost:8001") as client:
        # Test quality prediction
        quality_response = await client.post(
            "/predict/quality",
            json={"code": "def hello():\n    return 'world'"}
        )
        
        assert quality_response.status_code == 200
        quality = quality_response.json()
        assert 0 <= quality["quality_score"] <= 100
        assert quality["confidence"] > 0

@pytest.mark.asyncio
async def test_event_flow():
    publisher = EventPublisher()
    await publisher.connect()
    
    # Publish test event
    event = Event(
        type=EventType.ANALYSIS_REQUESTED,
        payload={
            "file_id": "test-123",
            "code": "print('hello')"
        }
    )
    
    await publisher.publish(event)
    
    # Wait for processing
    await asyncio.sleep(1)
    
    # Check if event was processed
    # (would check database or cache in real test)
    
    await publisher.close()
```

**Daily Checklist**:
- [ ] Notification service running
- [ ] Email sending working
- [ ] Slack integration tested
- [ ] Webhook delivery functional
- [ ] All services integrated
- [ ] End-to-end flow tested

---

### Day 5: Performance Optimization

#### Morning (4 hours)
**Goal**: Optimize service performance

**Performance Improvements**:

1. **Database Query Optimization**
2. **Caching Strategy Implementation**
3. **Async Processing Optimization**
4. **Connection Pooling**

**Benchmark Tests** (`tests/performance/benchmark.py`):
```python
import asyncio
import time
from locust import HttpUser, task, between

class PyForgeUser(HttpUser):
    wait_time = between(1, 3)
    
    def on_start(self):
        # Login
        response = self.client.post(
            "/auth/token",
            data={"username": "test", "password": "test"}
        )
        self.token = response.json()["access_token"]
        self.headers = {"Authorization": f"Bearer {self.token}"}
    
    @task(3)
    def list_projects(self):
        self.client.get("/api/v1/projects", headers=self.headers)
    
    @task(2)
    def analyze_code(self):
        self.client.post(
            "/api/v1/analyze/code",
            json={"code": "def test(): pass"},
            headers=self.headers
        )
    
    @task(1)
    def ml_prediction(self):
        self.client.post(
            "/api/v1/ml/predict/quality",
            json={"code": "def test(): pass"},
            headers=self.headers
        )
```

#### Afternoon (4 hours)
**Goal**: Monitoring and observability setup

**Prometheus Metrics** (`shared/monitoring/metrics.py`):
```python
from prometheus_client import Counter, Histogram, Gauge, generate_latest
from fastapi import Response

# Define metrics
request_count = Counter(
    'pyforge_requests_total',
    'Total requests',
    ['method', 'endpoint', 'status']
)

request_duration = Histogram(
    'pyforge_request_duration_seconds',
    'Request duration',
    ['method', 'endpoint']
)

active_users = Gauge(
    'pyforge_active_users',
    'Currently active users'
)

analysis_queue_size = Gauge(
    'pyforge_analysis_queue_size',
    'Analysis queue size'
)

ml_inference_duration = Histogram(
    'pyforge_ml_inference_seconds',
    'ML inference duration',
    ['model']
)

# Metrics endpoint
async def metrics_endpoint():
    return Response(
        content=generate_latest(),
        media_type="text/plain"
    )
```

**Daily Checklist**:
- [ ] Performance benchmarks completed
- [ ] Bottlenecks identified
- [ ] Optimizations implemented
- [ ] Monitoring metrics exposed
- [ ] Grafana dashboards created
- [ ] Alerts configured

**Week 2 Summary**:
- All services implemented
- Authentication/authorization working
- Analysis pipeline functional
- ML predictions operational
- Notifications delivering
- Services integrated and optimized

---

## 📅 Week 3: CLI/TUI & ML Training

### Day 1-2: CLI/TUI Implementation

**CLI Application** (`cli/main.py`):
```python
import typer
from rich.console import Console
from rich.table import Table
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, DataTable

# Implement full CLI/TUI interface
# See implementation details in separate guide
```

### Day 3-4: ML Model Training

**Training Pipeline**:
```python
# Implement model training
# Dataset preparation
# Training loop
# Model evaluation
# Model deployment
```

### Day 5: Integration Testing

**Full system integration tests**

---

## 📅 Week 4: Production Deployment

### Day 1-2: Docker & Kubernetes

**Production Docker images**
**Kubernetes manifests**
**Helm charts**

### Day 3: CI/CD Pipeline

**GitHub Actions workflows**
**Automated testing**
**Deployment automation**

### Day 4: Documentation

**API documentation**
**User guides**
**Architecture documentation**

### Day 5: Final Testing & Demo

**Load testing**
**Security scanning**
**Demo preparation**

---

## 🎓 Assessment Checklist

### Functional Requirements ✅
- [ ] User registration and authentication
- [ ] Project creation and management
- [ ] File upload and processing
- [ ] Code analysis with multiple analyzers
- [ ] ML predictions (quality, bugs, refactoring)
- [ ] Real-time notifications
- [ ] CLI/TUI interface
- [ ] API documentation

### Technical Requirements ✅
- [ ] MyPy strict mode passing
- [ ] 90%+ test coverage
- [ ] < 100ms p95 API latency
- [ ] Handles 1000+ concurrent users
- [ ] Docker containerization
- [ ] Kubernetes deployment ready
- [ ] CI/CD pipeline functional
- [ ] Monitoring and alerting

### Code Quality ✅
- [ ] Clean architecture principles
- [ ] SOLID principles followed
- [ ] Design patterns properly used
- [ ] Comprehensive error handling
- [ ] Proper logging throughout
- [ ] Security best practices
- [ ] Performance optimized

---

## 🚀 Submission Requirements

1. **GitHub Repository** with:
   - Complete source code
   - Comprehensive tests
   - CI/CD configuration
   - Documentation

2. **Live Demo** showing:
   - All features working
   - Performance metrics
   - Monitoring dashboards
   - Load testing results

3. **Documentation Package**:
   - Architecture document
   - API documentation
   - User guide
   - Deployment guide

4. **Presentation** covering:
   - Architecture decisions
   - Challenges overcome
   - Performance achievements
   - Future improvements

---

Congratulations! By completing this project, you've demonstrated mastery of modern Python development, from foundations through ML engineering, ready for L3/L4 engineering roles!