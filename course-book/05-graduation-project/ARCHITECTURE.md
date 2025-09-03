# PyForge Architecture Design Document

## System Architecture Overview

```mermaid
graph TB
    subgraph "Client Layer"
        CLI[CLI/TUI Client]
        WEB[Web Dashboard]
    end
    
    subgraph "API Layer"
        GW[API Gateway<br/>FastAPI]
        WS[WebSocket Server]
    end
    
    subgraph "Service Layer"
        AS[Analysis Service]
        MS[ML Service]
        PS[Project Service]
        NS[Notification Service]
    end
    
    subgraph "Data Layer"
        PG[(PostgreSQL)]
        RD[(Redis Cache)]
        S3[(Object Storage)]
    end
    
    subgraph "Infrastructure"
        MQ[Message Queue<br/>RabbitMQ]
        MON[Monitoring<br/>Prometheus]
    end
    
    CLI --> GW
    WEB --> GW
    WEB --> WS
    
    GW --> AS
    GW --> PS
    GW --> MS
    
    AS --> MQ
    MS --> MQ
    PS --> MQ
    MQ --> NS
    
    AS --> PG
    PS --> PG
    MS --> PG
    
    AS --> RD
    PS --> RD
    MS --> RD
    
    PS --> S3
    MS --> S3
    
    AS --> MON
    MS --> MON
    PS --> MON
    NS --> MON
```

---

## 🏗️ Service Architecture

### API Gateway (`pyforge-gateway`)

**Purpose**: Single entry point for all client requests

**Responsibilities**:
- Request routing and load balancing
- Authentication and authorization
- Rate limiting and throttling
- Request/Response transformation
- API versioning
- CORS handling

**Technology Stack**:
```python
# Core dependencies
FastAPI = "^0.104.0"
Pydantic = "^2.5.0"
python-jose = "^3.3.0"  # JWT
redis = "^5.0.0"        # Rate limiting
httpx = "^0.25.0"       # Service communication
```

**Key Endpoints**:
```python
# Authentication
POST   /auth/login
POST   /auth/refresh
POST   /auth/logout

# Projects
GET    /api/v1/projects
POST   /api/v1/projects
GET    /api/v1/projects/{id}
PUT    /api/v1/projects/{id}
DELETE /api/v1/projects/{id}

# Analysis
POST   /api/v1/analyze/code
POST   /api/v1/analyze/project
GET    /api/v1/analyze/metrics/{project_id}

# ML Features
POST   /api/v1/ml/predict/quality
POST   /api/v1/ml/suggest/refactor
POST   /api/v1/ml/detect/bugs
```

---

### Analysis Service (`pyforge-analyzer`)

**Purpose**: Static code analysis and metrics collection

**Core Components**:

```python
class AnalysisEngine:
    """Main analysis orchestrator"""
    
    def __init__(self):
        self.parsers: dict[str, Parser] = {}
        self.analyzers: list[Analyzer] = []
        self.reporters: list[Reporter] = []
    
    async def analyze(self, code: str, language: str) -> AnalysisResult:
        # Parse AST
        # Run analyzers
        # Aggregate results
        # Generate report
```

**Analysis Modules**:
1. **Complexity Analyzer**: Cyclomatic complexity, cognitive complexity
2. **Type Checker**: MyPy integration for type safety
3. **Security Scanner**: Bandit integration, SAST analysis
4. **Performance Profiler**: Memory usage, execution time
5. **Style Checker**: Ruff/Black compliance

**Message Queue Integration**:
```python
async def handle_analysis_request(message: AnalysisRequest):
    result = await engine.analyze(message.code, message.language)
    await publish_event("analysis.completed", result)
    await cache.set(f"analysis:{message.id}", result, ttl=3600)
```

---

### ML Service (`pyforge-ml`)

**Purpose**: Machine learning predictions and suggestions

**Model Architecture**:

```python
class CodeQualityModel:
    """Predicts code quality score"""
    
    def __init__(self):
        self.tokenizer = CodeTokenizer()
        self.encoder = TransformerEncoder(
            vocab_size=50000,
            d_model=768,
            n_heads=12,
            n_layers=6
        )
        self.classifier = nn.Linear(768, 5)  # 5 quality levels
    
    def forward(self, code: str) -> QualityScore:
        tokens = self.tokenizer(code)
        embeddings = self.encoder(tokens)
        score = self.classifier(embeddings.mean(dim=1))
        return QualityScore(score)
```

**ML Features**:
1. **Code Quality Prediction**: 0-100 score based on various metrics
2. **Refactoring Suggestions**: Pattern-based improvements
3. **Bug Detection**: Common bug patterns using NLP
4. **Performance Hints**: Optimization opportunities
5. **Security Vulnerability Detection**: CVE pattern matching

**Model Deployment**:
```python
# Using TorchServe or ONNX Runtime
model_server = ModelServer(
    models={
        "quality": CodeQualityModel(),
        "refactor": RefactoringSuggester(),
        "bugs": BugDetector(),
    },
    batch_size=32,
    max_latency_ms=100
)
```

---

### Project Service (`pyforge-projects`)

**Purpose**: Project management and collaboration

**Domain Model**:

```python
from dataclasses import dataclass
from typing import Optional
from datetime import datetime

@dataclass
class Project:
    id: str
    name: str
    description: str
    owner_id: str
    team_ids: list[str]
    created_at: datetime
    updated_at: datetime
    settings: ProjectSettings
    
@dataclass
class ProjectFile:
    id: str
    project_id: str
    path: str
    content: str
    version: int
    analysis_results: Optional[AnalysisResult]
    
@dataclass
class ProjectSettings:
    linting_enabled: bool
    auto_format: bool
    ml_suggestions: bool
    complexity_threshold: int
```

**Repository Pattern**:
```python
class ProjectRepository:
    async def create(self, project: Project) -> Project:
        # Persist to PostgreSQL
        # Publish creation event
        # Update cache
        
    async def get(self, project_id: str) -> Optional[Project]:
        # Check cache first
        # Fallback to database
        
    async def update(self, project: Project) -> Project:
        # Update database
        # Invalidate cache
        # Publish update event
```

---

### Notification Service (`pyforge-notify`)

**Purpose**: Multi-channel notifications

**Notification Channels**:
```python
class NotificationService:
    def __init__(self):
        self.channels = {
            "email": EmailChannel(),
            "slack": SlackChannel(),
            "webhook": WebhookChannel(),
            "websocket": WebSocketChannel(),
        }
    
    async def notify(self, event: Event, user_prefs: UserPreferences):
        for channel in user_prefs.enabled_channels:
            await self.channels[channel].send(event)
```

---

## 📊 Data Architecture

### PostgreSQL Schema

```sql
-- Users and Authentication
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Projects
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    owner_id UUID REFERENCES users(id),
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Project Files
CREATE TABLE project_files (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    path VARCHAR(500) NOT NULL,
    content TEXT,
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(project_id, path)
);

-- Analysis Results
CREATE TABLE analysis_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    file_id UUID REFERENCES project_files(id) ON DELETE CASCADE,
    metrics JSONB NOT NULL,
    issues JSONB DEFAULT '[]',
    suggestions JSONB DEFAULT '[]',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ML Predictions
CREATE TABLE ml_predictions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    file_id UUID REFERENCES project_files(id) ON DELETE CASCADE,
    model_name VARCHAR(100) NOT NULL,
    prediction JSONB NOT NULL,
    confidence FLOAT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Redis Cache Strategy

```python
# Cache keys structure
cache_keys = {
    "user:session:{user_id}": "User session data",
    "project:{project_id}": "Project metadata",
    "analysis:{file_id}": "Analysis results",
    "metrics:{project_id}": "Aggregated metrics",
    "ml:prediction:{file_id}:{model}": "ML predictions",
    "rate_limit:{user_id}:{endpoint}": "Rate limiting counter",
}

# Cache TTLs
cache_ttls = {
    "session": 3600,      # 1 hour
    "project": 300,       # 5 minutes
    "analysis": 1800,     # 30 minutes
    "metrics": 600,       # 10 minutes
    "ml_prediction": 900, # 15 minutes
}
```

---

## 🔄 Event-Driven Architecture

### Event Types

```python
from enum import Enum

class EventType(Enum):
    # Project Events
    PROJECT_CREATED = "project.created"
    PROJECT_UPDATED = "project.updated"
    PROJECT_DELETED = "project.deleted"
    
    # Analysis Events
    ANALYSIS_REQUESTED = "analysis.requested"
    ANALYSIS_COMPLETED = "analysis.completed"
    ANALYSIS_FAILED = "analysis.failed"
    
    # ML Events
    ML_PREDICTION_REQUESTED = "ml.prediction.requested"
    ML_PREDICTION_COMPLETED = "ml.prediction.completed"
    ML_MODEL_UPDATED = "ml.model.updated"
    
    # System Events
    SYSTEM_HEALTH_CHECK = "system.health_check"
    SYSTEM_ALERT = "system.alert"
```

### Message Queue Configuration

```python
# RabbitMQ Setup
import aio_pika

async def setup_rabbitmq():
    connection = await aio_pika.connect_robust(
        "amqp://guest:guest@rabbitmq/"
    )
    channel = await connection.channel()
    
    # Declare exchanges
    await channel.declare_exchange(
        "pyforge.events",
        aio_pika.ExchangeType.TOPIC,
        durable=True
    )
    
    # Declare queues
    queues = {
        "analysis": ["analysis.*"],
        "ml": ["ml.*"],
        "notifications": ["project.*", "analysis.completed"],
        "audit": ["*"],  # Receives all events
    }
    
    for queue_name, routing_keys in queues.items():
        queue = await channel.declare_queue(
            f"pyforge.{queue_name}",
            durable=True
        )
        for key in routing_keys:
            await queue.bind("pyforge.events", routing_key=key)
```

---

## 🔒 Security Architecture

### Authentication Flow

```python
# JWT-based authentication
from datetime import datetime, timedelta
import jwt

class AuthService:
    def __init__(self, secret_key: str):
        self.secret_key = secret_key
        self.algorithm = "HS256"
    
    def create_tokens(self, user_id: str) -> tuple[str, str]:
        # Access token (15 minutes)
        access_payload = {
            "user_id": user_id,
            "type": "access",
            "exp": datetime.utcnow() + timedelta(minutes=15)
        }
        access_token = jwt.encode(
            access_payload, 
            self.secret_key, 
            algorithm=self.algorithm
        )
        
        # Refresh token (7 days)
        refresh_payload = {
            "user_id": user_id,
            "type": "refresh",
            "exp": datetime.utcnow() + timedelta(days=7)
        }
        refresh_token = jwt.encode(
            refresh_payload,
            self.secret_key,
            algorithm=self.algorithm
        )
        
        return access_token, refresh_token
```

### Authorization Model

```python
from enum import Enum

class Permission(Enum):
    PROJECT_READ = "project:read"
    PROJECT_WRITE = "project:write"
    PROJECT_DELETE = "project:delete"
    ANALYSIS_RUN = "analysis:run"
    ML_PREDICT = "ml:predict"
    ADMIN_ALL = "admin:*"

class Role(Enum):
    VIEWER = [Permission.PROJECT_READ]
    DEVELOPER = [
        Permission.PROJECT_READ,
        Permission.PROJECT_WRITE,
        Permission.ANALYSIS_RUN,
        Permission.ML_PREDICT
    ]
    OWNER = [
        Permission.PROJECT_READ,
        Permission.PROJECT_WRITE,
        Permission.PROJECT_DELETE,
        Permission.ANALYSIS_RUN,
        Permission.ML_PREDICT
    ]
    ADMIN = [Permission.ADMIN_ALL]
```

---

## 📈 Monitoring & Observability

### Metrics Collection

```python
from prometheus_client import Counter, Histogram, Gauge

# Define metrics
request_count = Counter(
    'pyforge_requests_total',
    'Total number of requests',
    ['method', 'endpoint', 'status']
)

request_duration = Histogram(
    'pyforge_request_duration_seconds',
    'Request duration in seconds',
    ['method', 'endpoint']
)

active_connections = Gauge(
    'pyforge_active_connections',
    'Number of active connections'
)

model_inference_time = Histogram(
    'pyforge_ml_inference_seconds',
    'ML model inference time',
    ['model_name']
)
```

### Distributed Tracing

```python
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

tracer = trace.get_tracer(__name__)

@tracer.start_as_current_span("analyze_code")
async def analyze_code(code: str) -> AnalysisResult:
    with tracer.start_as_current_span("parse_ast"):
        ast = parse_code(code)
    
    with tracer.start_as_current_span("run_analyzers"):
        results = await run_analyzers(ast)
    
    with tracer.start_as_current_span("generate_report"):
        report = generate_report(results)
    
    return report
```

---

## 🚀 Deployment Architecture

### Container Orchestration

```yaml
# docker-compose.yml for development
version: '3.8'

services:
  gateway:
    build: ./services/gateway
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - redis
      - rabbitmq
    environment:
      DATABASE_URL: postgresql://user:pass@postgres/pyforge
      REDIS_URL: redis://redis:6379
      RABBITMQ_URL: amqp://guest:guest@rabbitmq/
  
  analyzer:
    build: ./services/analyzer
    depends_on:
      - postgres
      - redis
      - rabbitmq
    deploy:
      replicas: 3
  
  ml:
    build: ./services/ml
    depends_on:
      - postgres
      - redis
      - rabbitmq
    deploy:
      replicas: 2
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
  
  postgres:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: pyforge
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
  
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
  
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:
```

### Kubernetes Production Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pyforge-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pyforge-gateway
  template:
    metadata:
      labels:
        app: pyforge-gateway
    spec:
      containers:
      - name: gateway
        image: pyforge/gateway:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: pyforge-secrets
              key: database-url
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: pyforge-gateway-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: pyforge-gateway
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

---

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

```yaml
name: PyForge CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [gateway, analyzer, ml, projects, notify]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.12'
    
    - name: Install Poetry
      run: |
        curl -sSL https://install.python-poetry.org | python3 -
    
    - name: Install dependencies
      working-directory: ./services/${{ matrix.service }}
      run: |
        poetry install
    
    - name: Run tests
      working-directory: ./services/${{ matrix.service }}
      run: |
        poetry run pytest --cov --cov-report=xml
    
    - name: Type checking
      working-directory: ./services/${{ matrix.service }}
      run: |
        poetry run mypy . --strict
    
    - name: Linting
      working-directory: ./services/${{ matrix.service }}
      run: |
        poetry run ruff check .
    
    - name: Security scan
      working-directory: ./services/${{ matrix.service }}
      run: |
        poetry run bandit -r . -f json -o bandit-report.json

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: pyforge/platform:latest,pyforge/platform:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to Kubernetes
      uses: azure/k8s-deploy@v4
      with:
        manifests: |
          k8s/deployment.yaml
          k8s/service.yaml
        images: |
          pyforge/platform:${{ github.sha }}
```

---

## 📊 Performance Considerations

### Optimization Strategies

1. **Database Optimization**
   - Proper indexing on frequently queried columns
   - Connection pooling with asyncpg
   - Read replicas for analytics queries
   - Materialized views for complex aggregations

2. **Caching Strategy**
   - Redis for hot data
   - CDN for static assets
   - Application-level caching with TTL
   - Cache invalidation patterns

3. **Async Processing**
   - FastAPI async endpoints
   - Async database queries
   - Background job processing
   - Event-driven architecture

4. **ML Model Optimization**
   - Model quantization
   - Batch inference
   - Edge caching for predictions
   - ONNX runtime for inference

---

## 🎯 Scalability Targets

- **Concurrent Users**: 1,000+
- **Requests per Second**: 10,000
- **Analysis Throughput**: 100 files/second
- **ML Inference**: < 100ms p95
- **Database Connections**: 100 pooled connections
- **Message Queue**: 50,000 messages/second

---

This architecture provides a solid foundation for building a production-ready, scalable development platform that showcases mastery of modern Python engineering practices.