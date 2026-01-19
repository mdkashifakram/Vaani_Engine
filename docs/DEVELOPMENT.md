# VAANI Engine - Development Guide

**Complete development workflows, best practices, and tooling for VAANI Engine contributors**

---

## Table of Contents

1. [Development Environment Setup](#development-environment-setup)
2. [Project Structure](#project-structure)
3. [Development Workflow](#development-workflow)
4. [Coding Standards](#coding-standards)
5. [Testing Strategy](#testing-strategy)
6. [Debugging](#debugging)
7. [Database Development](#database-development)
8. [Voice Pipeline Development](#voice-pipeline-development)
9. [Performance Profiling](#performance-profiling)
10. [Release Process](#release-process)

---

## 1. Development Environment Setup

### 1.1 IDE Configuration

**Recommended IDE:** Visual Studio Code

**Required Extensions:**
```
Python (ms-python.python)
Pylance (ms-python.vscode-pylance)
Python Docstring Generator (njpwerner.autodocstring)
GitLens (eamodio.gitlens)
Docker (ms-azuretools.vscode-docker)
YAML (redhat.vscode-yaml)
```

**VS Code Settings (.vscode/settings.json):**
```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/venv/bin/python",
  "python.linting.enabled": true,
  "python.linting.flake8Enabled": true,
  "python.linting.mypyEnabled": true,
  "python.formatting.provider": "black",
  "python.testing.pytestEnabled": true,
  "python.testing.pytestArgs": ["tests"],
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  "files.exclude": {
    "**/__pycache__": true,
    "**/*.pyc": true,
    "**/.pytest_cache": true,
    "**/.mypy_cache": true
  }
}
```

**Launch Configuration (.vscode/launch.json):**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "FastAPI: Debug",
      "type": "python",
      "request": "launch",
      "module": "uvicorn",
      "args": [
        "src.main:app",
        "--reload",
        "--host", "0.0.0.0",
        "--port", "8000"
      ],
      "jinja": true,
      "justMyCode": false,
      "env": {
        "PYTHONPATH": "${workspaceFolder}"
      }
    },
    {
      "name": "Pytest: Current File",
      "type": "python",
      "request": "launch",
      "module": "pytest",
      "args": [
        "${file}",
        "-v",
        "-s"
      ],
      "console": "integratedTerminal",
      "justMyCode": false
    }
  ]
}
```

### 1.2 Git Configuration

**Install Pre-commit Hooks:**
```bash
# Install pre-commit
pip install pre-commit

# Install git hooks
pre-commit install
```

**.pre-commit-config.yaml:**
```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ['--maxkb=500']
      - id: check-json
      - id: check-toml

  - repo: https://github.com/psf/black
    rev: 23.11.0
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/PyCQA/isort
    rev: 5.12.0
    hooks:
      - id: isort
        args: ["--profile", "black"]

  - repo: https://github.com/PyCQA/flake8
    rev: 6.1.0
    hooks:
      - id: flake8
        args: ['--max-line-length=100', '--extend-ignore=E203']

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.1
    hooks:
      - id: mypy
        additional_dependencies: [types-all]
```

**Run Pre-commit Manually:**
```bash
# Run on all files
pre-commit run --all-files

# Run on staged files only
pre-commit run
```

### 1.3 Environment Variables Management

**Use direnv for Automatic Environment Loading:**

```bash
# Install direnv
brew install direnv  # macOS
sudo apt install direnv  # Ubuntu

# Add to shell (.bashrc or .zshrc)
eval "$(direnv hook bash)"  # For bash
eval "$(direnv hook zsh)"   # For zsh
```

**Create .envrc:**
```bash
# .envrc
export PYTHONPATH="${PWD}"
export ENVIRONMENT=development
export DEBUG=true
dotenv .env.development
```

```bash
# Allow direnv in this directory
direnv allow
```

---

## 2. Project Structure

### 2.1 Directory Layout

```
VAANI_Engine/
├── src/                          # Application source code
│   ├── __init__.py
│   ├── main.py                   # FastAPI app entry point
│   ├── config.py                 # Configuration management
│   │
│   ├── api/                      # API routes
│   │   ├── __init__.py
│   │   ├── voice.py              # Voice webhook handlers
│   │   ├── appointments.py       # Appointment CRUD
│   │   ├── patients.py           # Patient management
│   │   ├── doctors.py            # Doctor management
│   │   ├── call_logs.py          # Call history
│   │   └── admin.py              # Admin endpoints
│   │
│   ├── core/                     # Core business logic
│   │   ├── __init__.py
│   │   ├── voice_engine.py       # Voice pipeline orchestration
│   │   ├── stt_service.py        # Speech-to-text service
│   │   ├── tts_service.py        # Text-to-speech service
│   │   ├── llm_service.py        # LLM conversation logic
│   │   ├── vad_service.py        # Voice activity detection
│   │   └── audio_converter.py   # Audio format conversion
│   │
│   ├── models/                   # SQLAlchemy ORM models
│   │   ├── __init__.py
│   │   ├── base.py               # Base model class
│   │   ├── patient.py
│   │   ├── appointment.py
│   │   ├── doctor.py
│   │   ├── clinic.py
│   │   └── call_log.py
│   │
│   ├── schemas/                  # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── patient.py
│   │   ├── appointment.py
│   │   ├── doctor.py
│   │   └── call_log.py
│   │
│   ├── services/                 # Business services
│   │   ├── __init__.py
│   │   ├── appointment_manager.py
│   │   ├── patient_records.py
│   │   ├── notification_service.py
│   │   └── analytics_service.py
│   │
│   ├── utils/                    # Utility functions
│   │   ├── __init__.py
│   │   ├── security.py           # Encryption, hashing
│   │   ├── validation.py         # Input validation
│   │   ├── logging.py            # Logging configuration
│   │   └── datetime_helpers.py   # Date/time utilities
│   │
│   └── dependencies/             # FastAPI dependencies
│       ├── __init__.py
│       ├── database.py           # DB session dependency
│       ├── auth.py               # Auth dependencies
│       └── redis.py              # Redis client
│
├── tests/                        # Test suite
│   ├── __init__.py
│   ├── conftest.py               # Pytest fixtures
│   │
│   ├── unit/                     # Unit tests
│   │   ├── test_appointment_manager.py
│   │   ├── test_patient_records.py
│   │   └── test_audio_converter.py
│   │
│   ├── integration/              # Integration tests
│   │   ├── test_voice_api.py
│   │   ├── test_appointment_api.py
│   │   └── test_database.py
│   │
│   └── e2e/                      # End-to-end tests
│       ├── test_call_flow.py
│       └── test_appointment_booking.py
│
├── alembic/                      # Database migrations
│   ├── versions/
│   ├── env.py
│   └── script.py.mako
│
├── scripts/                      # Utility scripts
│   ├── run_dev.sh
│   ├── backup_db.sh
│   ├── seed_database.py
│   └── test_call.py
│
├── docs/                         # Documentation
├── knowledge_base/               # Design decisions
├── reference/                    # Project reference
├── .github/                      # GitHub workflows
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
│
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
├── requirements-dev.txt
├── pyproject.toml
├── setup.py
├── .env.example
├── .gitignore
└── README.md
```

### 2.2 Module Responsibilities

| Module | Responsibility | Key Files |
|--------|----------------|-----------|
| `api/` | HTTP endpoint handlers | `voice.py`, `appointments.py` |
| `core/` | Core business logic, voice processing | `voice_engine.py`, `stt_service.py` |
| `models/` | Database ORM models | `patient.py`, `appointment.py` |
| `schemas/` | Request/response validation | Pydantic models |
| `services/` | Business service layer | `appointment_manager.py` |
| `utils/` | Shared utilities | `security.py`, `validation.py` |
| `tests/` | Test suite | Unit, integration, e2e tests |

---

## 3. Development Workflow

### 3.1 Feature Development Flow

**1. Create Feature Branch**
```bash
# Sync with main
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/hindi-voice-support
```

**2. Implement Feature**
```bash
# Make changes
# Write tests as you go

# Run tests
pytest tests/ -v

# Check code quality
black src/ tests/
isort src/ tests/
flake8 src/ tests/
mypy src/
```

**3. Commit Changes**
```bash
# Stage changes
git add .

# Commit with conventional commit message
git commit -m "feat(voice): add Hindi language support for TTS

- Integrate Hindi voice models
- Add language detection for Hindi/English
- Update configuration for multi-language support

Closes #42"
```

**4. Push and Create PR**
```bash
# Push to remote
git push origin feature/hindi-voice-support

# Create pull request on GitHub
# Request code review
```

### 3.2 Code Review Process

**Reviewer Checklist:**
- [ ] Code follows style guidelines (black, flake8, mypy pass)
- [ ] Tests added for new functionality (coverage >= 80%)
- [ ] Documentation updated (docstrings, README if needed)
- [ ] No security vulnerabilities introduced
- [ ] Performance impact considered
- [ ] Database migrations included if schema changed
- [ ] Breaking changes clearly documented

**Example Review Comments:**
```python
# Bad: Magic number without explanation
if duration > 300:
    timeout()

# Good: Named constant with clear intent
MAX_CALL_DURATION_SECONDS = 300
if duration > MAX_CALL_DURATION_SECONDS:
    timeout()
```

### 3.3 Continuous Integration

**GitHub Actions Workflow (.github/workflows/ci.yml):**
```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          pip install -r requirements-dev.txt
      - name: Run linters
        run: |
          black --check src/ tests/
          isort --check src/ tests/
          flake8 src/ tests/
          mypy src/

  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
      - name: Run tests
        env:
          DATABASE_URL: postgresql+asyncpg://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379
        run: |
          pytest tests/ -v --cov=src --cov-report=xml
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t vaani-engine:test .
```

---

## 4. Coding Standards

### 4.1 Python Style Guide

**Follow PEP 8 with Black Formatting:**

```python
# Maximum line length: 100 characters
# Use double quotes for strings
# Type hints required for all functions

# Good Example
async def create_appointment(
    patient_id: int,
    doctor_id: int,
    scheduled_time: datetime,
    db: AsyncSession
) -> Appointment:
    """Create a new appointment for a patient.

    Args:
        patient_id: Unique identifier for the patient
        doctor_id: Unique identifier for the doctor
        scheduled_time: When the appointment is scheduled
        db: Database session

    Returns:
        Created Appointment object

    Raises:
        ValidationError: If scheduled_time is in the past
        ConflictError: If time slot is already booked
    """
    if scheduled_time < datetime.now(timezone.utc):
        raise ValidationError("Appointment time must be in the future")

    # Check for conflicts
    existing = await db.execute(
        select(Appointment).where(
            Appointment.doctor_id == doctor_id,
            Appointment.scheduled_time == scheduled_time
        )
    )

    if existing.scalar_one_or_none():
        raise ConflictError("Time slot already booked")

    # Create appointment
    appointment = Appointment(
        patient_id=patient_id,
        doctor_id=doctor_id,
        scheduled_time=scheduled_time,
        status=AppointmentStatus.SCHEDULED
    )

    db.add(appointment)
    await db.commit()
    await db.refresh(appointment)

    return appointment
```

### 4.2 Async/Await Best Practices

```python
# Good: Concurrent I/O operations
async def fetch_patient_data(patient_id: int, db: AsyncSession):
    """Fetch patient and appointments concurrently."""
    patient_task = db.execute(
        select(Patient).where(Patient.id == patient_id)
    )
    appointments_task = db.execute(
        select(Appointment).where(Appointment.patient_id == patient_id)
    )

    patient_result, appointments_result = await asyncio.gather(
        patient_task,
        appointments_task
    )

    return patient_result.scalar_one(), appointments_result.scalars().all()


# Bad: Sequential awaits for independent operations
async def fetch_patient_data_slow(patient_id: int, db: AsyncSession):
    patient_result = await db.execute(
        select(Patient).where(Patient.id == patient_id)
    )
    appointments_result = await db.execute(
        select(Appointment).where(Appointment.patient_id == patient_id)
    )
    return patient_result.scalar_one(), appointments_result.scalars().all()
```

### 4.3 Error Handling Patterns

```python
# Define custom exceptions
class VANIIException(Exception):
    """Base exception for VAANI Engine."""
    pass

class ValidationError(VANIIException):
    """Raised when input validation fails."""
    pass

class ResourceNotFoundError(VANIIException):
    """Raised when requested resource doesn't exist."""
    pass

class ConflictError(VANIIException):
    """Raised when resource conflict occurs."""
    pass


# Use specific exception handling
async def get_appointment(appointment_id: int, db: AsyncSession) -> Appointment:
    """Get appointment by ID."""
    try:
        result = await db.execute(
            select(Appointment).where(Appointment.id == appointment_id)
        )
        appointment = result.scalar_one_or_none()

        if not appointment:
            raise ResourceNotFoundError(f"Appointment {appointment_id} not found")

        return appointment

    except ResourceNotFoundError:
        # Re-raise custom exceptions
        raise

    except SQLAlchemyError as e:
        # Log database errors
        logger.error(f"Database error fetching appointment: {e}")
        raise VANIIException("Failed to retrieve appointment") from e
```

### 4.4 Logging Standards

```python
import logging
from src.utils.logging import get_logger

logger = get_logger(__name__)

async def process_voice_call(call_sid: str):
    """Process incoming voice call."""
    logger.info(f"Processing call: {call_sid}")

    try:
        # Process call
        result = await handle_call(call_sid)
        logger.info(
            f"Call processed successfully",
            extra={
                "call_sid": call_sid,
                "duration": result.duration,
                "status": result.status
            }
        )

    except Exception as e:
        logger.error(
            f"Call processing failed",
            extra={"call_sid": call_sid, "error": str(e)},
            exc_info=True
        )
        raise


# Log Levels:
# DEBUG: Detailed diagnostic information
# INFO: General informational messages
# WARNING: Warning messages for recoverable issues
# ERROR: Error messages for failures
# CRITICAL: Critical failures requiring immediate attention
```

---

## 5. Testing Strategy

### 5.1 Test Structure

**Test Pyramid:**
```
        /\
       /  \         E2E Tests (10%)
      /____\        - Full user flows
     /      \       - Call → Appointment booking
    /________\
   /          \     Integration Tests (30%)
  /____________\    - API endpoints
 /              \   - Database operations
/________________\  Unit Tests (60%)
                    - Business logic
                    - Utilities
```

### 5.2 Unit Testing

**Example: Testing Appointment Manager**

```python
# tests/unit/test_appointment_manager.py
import pytest
from datetime import datetime, timedelta, timezone
from unittest.mock import AsyncMock, MagicMock

from src.services.appointment_manager import AppointmentManager
from src.models.appointment import Appointment, AppointmentStatus
from src.exceptions import ValidationError, ConflictError


@pytest.fixture
def appointment_manager():
    """Create appointment manager instance."""
    return AppointmentManager()


@pytest.fixture
def mock_db_session():
    """Create mock database session."""
    return AsyncMock()


@pytest.mark.asyncio
async def test_create_appointment_success(appointment_manager, mock_db_session):
    """Test successful appointment creation."""
    future_time = datetime.now(timezone.utc) + timedelta(days=1)

    # Mock database query to return no conflicts
    mock_db_session.execute.return_value.scalar_one_or_none.return_value = None

    appointment = await appointment_manager.create_appointment(
        patient_id=1,
        doctor_id=2,
        scheduled_time=future_time,
        reason="Checkup",
        db=mock_db_session
    )

    assert appointment.patient_id == 1
    assert appointment.doctor_id == 2
    assert appointment.status == AppointmentStatus.SCHEDULED
    mock_db_session.add.assert_called_once()
    mock_db_session.commit.assert_called_once()


@pytest.mark.asyncio
async def test_create_appointment_past_time(appointment_manager, mock_db_session):
    """Test that creating appointment in past raises error."""
    past_time = datetime.now(timezone.utc) - timedelta(days=1)

    with pytest.raises(ValidationError) as exc:
        await appointment_manager.create_appointment(
            patient_id=1,
            doctor_id=2,
            scheduled_time=past_time,
            reason="Checkup",
            db=mock_db_session
        )

    assert "future" in str(exc.value).lower()


@pytest.mark.asyncio
async def test_create_appointment_conflict(appointment_manager, mock_db_session):
    """Test conflict when time slot is already booked."""
    future_time = datetime.now(timezone.utc) + timedelta(days=1)

    # Mock existing appointment at same time
    existing_appointment = Appointment(
        id=100,
        doctor_id=2,
        scheduled_time=future_time
    )
    mock_db_session.execute.return_value.scalar_one_or_none.return_value = existing_appointment

    with pytest.raises(ConflictError) as exc:
        await appointment_manager.create_appointment(
            patient_id=1,
            doctor_id=2,
            scheduled_time=future_time,
            reason="Checkup",
            db=mock_db_session
        )

    assert "already booked" in str(exc.value).lower()
```

### 5.3 Integration Testing

**Example: Testing Voice API**

```python
# tests/integration/test_voice_api.py
import pytest
from httpx import AsyncClient
from src.main import app


@pytest.mark.asyncio
async def test_incoming_call_webhook():
    """Test Twilio incoming call webhook."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post(
            "/api/v1/voice/incoming",
            data={
                "CallSid": "CA1234567890abcdef",
                "From": "+919876543210",
                "To": "+911234567890",
                "CallStatus": "ringing"
            }
        )

    assert response.status_code == 200
    assert "application/xml" in response.headers["content-type"]
    assert "<Response>" in response.text
    assert "<Stream" in response.text


@pytest.mark.asyncio
async def test_create_appointment_endpoint(test_db_session):
    """Test appointment creation via API."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post(
            "/api/v1/appointments",
            json={
                "patient_id": 1,
                "doctor_id": 2,
                "scheduled_time": "2026-01-25T14:30:00Z",
                "reason": "Checkup"
            },
            headers={"Authorization": "Bearer test_token"}
        )

    assert response.status_code == 201
    data = response.json()
    assert data["success"] is True
    assert "id" in data["data"]
    assert data["data"]["status"] == "scheduled"
```

### 5.4 Test Fixtures

**conftest.py:**
```python
# tests/conftest.py
import pytest
import asyncio
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker

from src.models.base import Base
from src.config import settings


@pytest.fixture(scope="session")
def event_loop():
    """Create event loop for async tests."""
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()


@pytest.fixture(scope="session")
async def test_engine():
    """Create test database engine."""
    engine = create_async_engine(
        "postgresql+asyncpg://postgres:postgres@localhost:5432/test_db",
        echo=False
    )

    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
        await conn.run_sync(Base.metadata.create_all)

    yield engine

    await engine.dispose()


@pytest.fixture
async def test_db_session(test_engine):
    """Create test database session."""
    async_session = sessionmaker(
        test_engine,
        class_=AsyncSession,
        expire_on_commit=False
    )

    async with async_session() as session:
        yield session
        await session.rollback()
```

### 5.5 Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/unit/test_appointment_manager.py -v

# Run tests matching pattern
pytest tests/ -k "appointment" -v

# Run with coverage
pytest tests/ --cov=src --cov-report=html --cov-report=term

# Run only unit tests
pytest tests/unit/ -v

# Run only integration tests
pytest tests/integration/ -v

# Run with verbose output and show print statements
pytest tests/ -vv -s

# Run failed tests from last run
pytest --lf

# Run tests in parallel (requires pytest-xdist)
pytest tests/ -n auto
```

---

## 6. Debugging

### 6.1 Using Python Debugger (pdb)

```python
# Insert breakpoint in code
import pdb; pdb.set_trace()

# Or use built-in breakpoint() (Python 3.7+)
breakpoint()

# Common pdb commands:
# n - next line
# s - step into function
# c - continue execution
# l - list source code
# p <var> - print variable
# q - quit debugger
```

### 6.2 VS Code Debugging

**Set Breakpoints:**
1. Click left margin in code editor (red dot appears)
2. Press F5 to start debugging
3. Use debug toolbar to step through code

**Debug Configuration:**
```json
{
  "name": "FastAPI: Debug",
  "type": "python",
  "request": "launch",
  "module": "uvicorn",
  "args": [
    "src.main:app",
    "--reload",
    "--host", "0.0.0.0",
    "--port", "8000"
  ],
  "jinja": true,
  "justMyCode": false
}
```

### 6.3 Logging for Debugging

```python
import logging
logging.basicConfig(level=logging.DEBUG)

logger = logging.getLogger(__name__)

async def process_audio(audio_data: bytes):
    logger.debug(f"Processing audio: {len(audio_data)} bytes")

    # Log variable values
    logger.debug(f"Audio format: sample_rate={sample_rate}, channels={channels}")

    # Log with structured data
    logger.debug(
        "Audio processing complete",
        extra={
            "duration_ms": duration,
            "format": audio_format,
            "size_bytes": len(audio_data)
        }
    )
```

### 6.4 HTTP Request Debugging

**Use httpie for Testing:**
```bash
# Install httpie
pip install httpie

# Test API endpoints
http POST http://localhost:8000/api/v1/appointments \
  patient_id:=1 \
  doctor_id:=2 \
  scheduled_time="2026-01-25T14:30:00Z" \
  Authorization:"Bearer test_token"

# Test with verbose output
http --verbose POST http://localhost:8000/api/v1/appointments
```

---

## 7. Database Development

### 7.1 Creating Models

```python
# src/models/appointment.py
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey, Enum
from sqlalchemy.orm import relationship
from datetime import datetime, timezone
import enum

from src.models.base import Base


class AppointmentStatus(str, enum.Enum):
    SCHEDULED = "scheduled"
    COMPLETED = "completed"
    CANCELLED = "cancelled"
    NO_SHOW = "no_show"


class Appointment(Base):
    __tablename__ = "appointments"

    id = Column(Integer, primary_key=True, index=True)
    patient_id = Column(Integer, ForeignKey("patients.id"), nullable=False, index=True)
    doctor_id = Column(Integer, ForeignKey("doctors.id"), nullable=False, index=True)
    scheduled_time = Column(DateTime(timezone=True), nullable=False, index=True)
    status = Column(Enum(AppointmentStatus), default=AppointmentStatus.SCHEDULED)
    reason = Column(String(500))
    notes = Column(String(1000))
    created_at = Column(DateTime(timezone=True), default=lambda: datetime.now(timezone.utc))
    updated_at = Column(
        DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc),
        onupdate=lambda: datetime.now(timezone.utc)
    )

    # Relationships
    patient = relationship("Patient", back_populates="appointments")
    doctor = relationship("Doctor", back_populates="appointments")

    def __repr__(self):
        return f"<Appointment(id={self.id}, patient_id={self.patient_id}, time={self.scheduled_time})>"
```

### 7.2 Creating Migrations

```bash
# Create new migration
alembic revision --autogenerate -m "Add appointments table"

# Review generated migration
cat alembic/versions/xxxxx_add_appointments_table.py

# Apply migration
alembic upgrade head

# Rollback migration
alembic downgrade -1

# Show migration history
alembic history

# Show current version
alembic current
```

**Example Migration:**
```python
# alembic/versions/xxxxx_add_appointments_table.py
from alembic import op
import sqlalchemy as sa


def upgrade():
    op.create_table(
        'appointments',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('patient_id', sa.Integer(), nullable=False),
        sa.Column('doctor_id', sa.Integer(), nullable=False),
        sa.Column('scheduled_time', sa.DateTime(timezone=True), nullable=False),
        sa.Column('status', sa.String(20), nullable=False),
        sa.Column('reason', sa.String(500)),
        sa.Column('created_at', sa.DateTime(timezone=True), nullable=False),
        sa.PrimaryKeyConstraint('id')
    )
    op.create_index('ix_appointments_patient_id', 'appointments', ['patient_id'])
    op.create_index('ix_appointments_doctor_id', 'appointments', ['doctor_id'])
    op.create_foreign_key(None, 'appointments', 'patients', ['patient_id'], ['id'])
    op.create_foreign_key(None, 'appointments', 'doctors', ['doctor_id'], ['id'])


def downgrade():
    op.drop_table('appointments')
```

### 7.3 Database Queries

```python
from sqlalchemy import select, update, delete
from sqlalchemy.orm import joinedload

# Select with eager loading
async def get_appointment_with_relations(appointment_id: int, db: AsyncSession):
    result = await db.execute(
        select(Appointment)
        .options(
            joinedload(Appointment.patient),
            joinedload(Appointment.doctor)
        )
        .where(Appointment.id == appointment_id)
    )
    return result.scalar_one_or_none()


# Complex query with filters
async def get_upcoming_appointments(doctor_id: int, limit: int, db: AsyncSession):
    result = await db.execute(
        select(Appointment)
        .where(
            Appointment.doctor_id == doctor_id,
            Appointment.scheduled_time > datetime.now(timezone.utc),
            Appointment.status == AppointmentStatus.SCHEDULED
        )
        .order_by(Appointment.scheduled_time.asc())
        .limit(limit)
    )
    return result.scalars().all()


# Update query
async def cancel_appointment(appointment_id: int, db: AsyncSession):
    await db.execute(
        update(Appointment)
        .where(Appointment.id == appointment_id)
        .values(status=AppointmentStatus.CANCELLED)
    )
    await db.commit()
```

---

## 8. Voice Pipeline Development

### 8.1 Testing Voice Components Locally

**Mock Audio Input:**
```python
# scripts/test_voice_pipeline.py
import asyncio
import numpy as np
from src.core.voice_engine import VoiceEngine

async def test_voice_pipeline():
    """Test voice pipeline with mock audio."""
    engine = VoiceEngine()

    # Generate mock audio (1 second of silence at 16kHz)
    mock_audio = np.zeros(16000, dtype=np.int16)

    # Process audio
    result = await engine.process_audio(mock_audio)

    print(f"Transcript: {result.transcript}")
    print(f"Response: {result.response_text}")

asyncio.run(test_voice_pipeline())
```

### 8.2 Voice Provider Development

**Create Custom STT Provider:**
```python
# src/core/stt_providers/custom_stt.py
from abc import ABC, abstractmethod
from dataclasses import dataclass


@dataclass
class TranscriptionResult:
    text: str
    language: str
    confidence: float
    latency_ms: int


class STTProvider(ABC):
    """Abstract base class for STT providers."""

    @abstractmethod
    async def transcribe(self, audio: bytes) -> TranscriptionResult:
        """Transcribe audio to text."""
        pass


class CustomSTT(STTProvider):
    """Custom STT implementation."""

    async def transcribe(self, audio: bytes) -> TranscriptionResult:
        # Implement transcription logic
        pass
```

---

## 9. Performance Profiling

### 9.1 Profiling Python Code

**Using py-spy:**
```bash
# Install py-spy
pip install py-spy

# Profile running process
py-spy top --pid <process_id>

# Generate flame graph
py-spy record -o profile.svg --pid <process_id>

# Profile specific function
py-spy record -o profile.svg -- python -m pytest tests/test_voice_pipeline.py
```

**Using cProfile:**
```python
import cProfile
import pstats

def profile_function():
    profiler = cProfile.Profile()
    profiler.enable()

    # Code to profile
    result = some_expensive_function()

    profiler.disable()
    stats = pstats.Stats(profiler)
    stats.sort_stats('cumulative')
    stats.print_stats(10)
```

### 9.2 Database Query Profiling

```python
# Enable SQL query logging
import logging
logging.getLogger('sqlalchemy.engine').setLevel(logging.INFO)

# Use EXPLAIN ANALYZE for query optimization
async def analyze_query(db: AsyncSession):
    result = await db.execute(
        text("EXPLAIN ANALYZE SELECT * FROM appointments WHERE doctor_id = 5")
    )
    print(result.all())
```

---

## 10. Release Process

### 10.1 Version Numbering

Follow [Semantic Versioning](https://semver.org/):
- **MAJOR**: Incompatible API changes (2.0.0)
- **MINOR**: New functionality, backwards compatible (2.1.0)
- **PATCH**: Bug fixes, backwards compatible (2.1.1)

### 10.2 Release Checklist

**Pre-Release:**
- [ ] All tests passing (unit, integration, e2e)
- [ ] Code coverage >= 80%
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Database migrations tested
- [ ] Performance benchmarks validated
- [ ] Security audit completed

**Release:**
```bash
# Update version
bump2version minor  # or major/patch

# Create release tag
git tag -a v2.1.0 -m "Release version 2.1.0"

# Push tags
git push origin v2.1.0

# Build and push Docker image
docker build -t vaani-engine:2.1.0 .
docker tag vaani-engine:2.1.0 registry.example.com/vaani-engine:2.1.0
docker push registry.example.com/vaani-engine:2.1.0
```

**Post-Release:**
- [ ] Deploy to staging environment
- [ ] Run smoke tests
- [ ] Deploy to production
- [ ] Monitor error rates and performance
- [ ] Update GitHub release notes

---

**Document Version**: 1.0
**Last Updated**: 2026-01-20
**Maintained By**: VAANI Development Team
