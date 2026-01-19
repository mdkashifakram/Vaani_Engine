# Contributing to VAANI Engine

Thank you for your interest in contributing to VAANI Engine. This document provides guidelines and workflows for contributing to the project.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Contribution Workflow](#contribution-workflow)
- [Coding Standards](#coding-standards)
- [Testing Requirements](#testing-requirements)
- [Documentation Guidelines](#documentation-guidelines)
- [Pull Request Process](#pull-request-process)
- [Community](#community)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive environment for all contributors, regardless of:
- Experience level
- Gender identity and expression
- Sexual orientation
- Disability
- Personal appearance
- Body size
- Race or ethnicity
- Age
- Religion or lack thereof
- Nationality

### Expected Behavior

- Use welcoming and inclusive language
- Respect differing viewpoints and experiences
- Accept constructive criticism gracefully
- Focus on what is best for the community
- Show empathy towards other community members

### Unacceptable Behavior

- Harassment, trolling, or discriminatory comments
- Publishing others' private information without consent
- Spam or off-topic discussions
- Any conduct that would be inappropriate in a professional setting

---

## How Can I Contribute?

### Reporting Bugs

Before submitting a bug report:
1. Check the [existing issues](https://github.com/mdkashifakram/VAANI_Engine/issues) to avoid duplicates
2. Ensure you're using the latest version
3. Collect relevant system information

**Bug Report Template:**
```markdown
**Description**
Clear description of the bug

**Steps to Reproduce**
1. Go to '...'
2. Click on '...'
3. See error

**Expected Behavior**
What you expected to happen

**Actual Behavior**
What actually happened

**Environment**
- OS: [e.g., Ubuntu 22.04]
- Python version: [e.g., 3.11.5]
- VAANI version: [e.g., 2.0.0]

**Logs**
Relevant error messages or logs

**Screenshots**
If applicable, add screenshots
```

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion, include:

- **Clear title** describing the enhancement
- **Detailed description** of the proposed functionality
- **Use cases** explaining why this would be valuable
- **Implementation ideas** if you have any
- **Alternatives considered** other approaches you've thought about

### Priority Areas for Contribution

We especially welcome contributions in these areas:

#### 1. Language Support
- **Regional languages**: Tamil, Telugu, Bengali, Marathi, Gujarati
- **Dialect variations**: Regional accents and colloquialisms
- **Language model fine-tuning**: Improve accuracy for medical terminology

#### 2. Voice Quality
- **Custom voice training**: Clinic-specific voice personas
- **Emotion detection**: Identify patient stress or urgency
- **Background noise filtering**: Improve recognition in noisy environments

#### 3. Healthcare Integrations
- **EHR connectors**: Practo, Lybrate, HealthEngine
- **Appointment systems**: Integration with popular booking platforms
- **Payment gateways**: Support for Indian payment systems (UPI, Paytm)

#### 4. Testing & Quality
- **Unit test coverage**: Increase from current 75% to 90%+
- **Integration tests**: Test full voice pipeline end-to-end
- **Load testing**: Simulate high concurrent call volumes

#### 5. Documentation
- **Setup guides**: Improve onboarding for non-technical users
- **Video tutorials**: Screen recordings of installation and configuration
- **API examples**: More code samples for common use cases
- **Translations**: Translate documentation to Hindi and other languages

---

## Development Setup

### Prerequisites

Ensure you have the following installed:
```bash
# Required
- Python 3.11 or higher
- PostgreSQL 15+
- Redis 7+
- Git

# Optional (recommended)
- Docker & Docker Compose
- VS Code with Python extension
```

### Fork and Clone

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/VAANI_Engine.git
   cd VAANI_Engine
   ```

3. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/mdkashifakram/VAANI_Engine.git
   ```

### Local Development Environment

#### Option 1: Docker (Recommended for consistency)

```bash
# Build development containers
docker-compose up -d

# View logs
docker-compose logs -f backend

# Access backend shell
docker-compose exec backend bash
```

#### Option 2: Local Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Development dependencies

# Setup pre-commit hooks
pre-commit install

# Create .env file
cp .env.example .env
# Edit .env with your local configuration

# Start PostgreSQL and Redis
# (Use Docker or install locally)

# Run database migrations
alembic upgrade head

# Start development server
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

### Verify Setup

```bash
# Run health check
curl http://localhost:8000/health

# Run tests
pytest tests/ -v

# Check code quality
flake8 src/ tests/
mypy src/
```

---

## Contribution Workflow

### 1. Create a Feature Branch

```bash
# Sync with upstream
git fetch upstream
git checkout main
git merge upstream/main

# Create feature branch
git checkout -b feature/your-feature-name
```

**Branch naming conventions:**
- `feature/` - New features (e.g., `feature/hindi-tts`)
- `fix/` - Bug fixes (e.g., `fix/appointment-validation`)
- `docs/` - Documentation updates (e.g., `docs/api-examples`)
- `refactor/` - Code refactoring (e.g., `refactor/voice-pipeline`)
- `test/` - Test additions (e.g., `test/integration-suite`)

### 2. Make Your Changes

- Write clean, readable code following our [Coding Standards](#coding-standards)
- Add tests for new functionality
- Update documentation as needed
- Keep commits atomic and well-described

### 3. Commit Your Changes

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```bash
# Format: <type>(<scope>): <description>

# Examples:
git commit -m "feat(voice): add Hindi language support for TTS"
git commit -m "fix(appointments): validate date ranges correctly"
git commit -m "docs(api): add webhook examples"
git commit -m "test(voice): add integration tests for STT pipeline"
```

**Commit types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code formatting (no logic changes)
- `refactor`: Code restructuring (no behavior changes)
- `test`: Adding or updating tests
- `chore`: Maintenance tasks (dependencies, config)

### 4. Push and Create Pull Request

```bash
# Push to your fork
git push origin feature/your-feature-name
```

Then create a Pull Request on GitHub from your fork to `mdkashifakram/VAANI_Engine:main`.

---

## Coding Standards

### Python Style Guide

We follow [PEP 8](https://peps.python.org/pep-0008/) with some modifications:

```python
# Maximum line length: 100 characters (not 79)
# Use double quotes for strings (not single)
# Use type hints for all function signatures

# Good example:
async def create_appointment(
    patient_id: int,
    appointment_time: datetime,
    doctor_id: int
) -> Appointment:
    """Create a new appointment for a patient.

    Args:
        patient_id: Unique identifier for the patient
        appointment_time: Scheduled datetime for appointment
        doctor_id: Identifier for the assigned doctor

    Returns:
        Created Appointment object

    Raises:
        ValidationError: If appointment time is in the past
        DuplicateError: If appointment already exists
    """
    if appointment_time < datetime.now():
        raise ValidationError("Appointment time cannot be in the past")

    # Implementation...
    return appointment
```

### Code Organization

```python
# Import order (enforced by isort):
# 1. Standard library
import os
import sys
from datetime import datetime
from typing import Optional

# 2. Third-party packages
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession

# 3. Local modules
from src.models.appointment import Appointment
from src.utils.validation import validate_phone_number
```

### Async/Await Pattern

Always use async for I/O operations:

```python
# Good: Async database operations
async def get_patient(db: AsyncSession, patient_id: int) -> Optional[Patient]:
    result = await db.execute(
        select(Patient).where(Patient.id == patient_id)
    )
    return result.scalar_one_or_none()

# Bad: Blocking operations
def get_patient(db: Session, patient_id: int) -> Optional[Patient]:
    return db.query(Patient).filter(Patient.id == patient_id).first()
```

### Error Handling

```python
# Specific exceptions with context
try:
    appointment = await create_appointment(patient_id, time, doctor_id)
except ValidationError as e:
    logger.warning(f"Invalid appointment data: {e}")
    raise HTTPException(status_code=400, detail=str(e))
except DatabaseError as e:
    logger.error(f"Database error creating appointment: {e}")
    raise HTTPException(status_code=500, detail="Internal server error")
```

### Logging

```python
import logging

logger = logging.getLogger(__name__)

# Use appropriate log levels
logger.debug("Detailed diagnostic information")
logger.info("General informational messages")
logger.warning("Warning messages for recoverable issues")
logger.error("Error messages for failures")
logger.critical("Critical failures requiring immediate attention")

# Include context in logs
logger.info(f"Appointment created: patient_id={patient_id}, time={time}")
```

---

## Testing Requirements

### Test Coverage Standards

- **Minimum coverage**: 80% for all new code
- **Critical paths**: 100% coverage for voice pipeline, appointment booking
- **Integration tests**: Required for API endpoints
- **E2E tests**: Required for user-facing features

### Writing Tests

#### Unit Tests

```python
# tests/unit/test_appointment_manager.py
import pytest
from datetime import datetime, timedelta
from src.services.appointment_manager import AppointmentManager

@pytest.mark.asyncio
async def test_create_appointment_valid():
    """Test creating a valid appointment."""
    manager = AppointmentManager()
    future_time = datetime.now() + timedelta(days=1)

    appointment = await manager.create_appointment(
        patient_id=1,
        appointment_time=future_time,
        doctor_id=2
    )

    assert appointment.patient_id == 1
    assert appointment.doctor_id == 2
    assert appointment.status == "scheduled"

@pytest.mark.asyncio
async def test_create_appointment_past_time():
    """Test that creating appointment in past raises error."""
    manager = AppointmentManager()
    past_time = datetime.now() - timedelta(days=1)

    with pytest.raises(ValidationError) as exc:
        await manager.create_appointment(
            patient_id=1,
            appointment_time=past_time,
            doctor_id=2
        )

    assert "past" in str(exc.value).lower()
```

#### Integration Tests

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
                "CallSid": "CA1234567890",
                "From": "+919876543210",
                "To": "+911234567890"
            }
        )

    assert response.status_code == 200
    assert "TwiML" in response.text
```

### Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/unit/test_appointment_manager.py -v

# Run with coverage
pytest --cov=src --cov-report=html tests/

# Run only integration tests
pytest tests/integration/ -v

# Run tests matching pattern
pytest -k "appointment" -v
```

---

## Documentation Guidelines

### Code Documentation

#### Docstrings

Use Google-style docstrings for all public functions, classes, and modules:

```python
def calculate_call_duration(start_time: datetime, end_time: datetime) -> int:
    """Calculate call duration in seconds.

    Args:
        start_time: When the call started (UTC timezone)
        end_time: When the call ended (UTC timezone)

    Returns:
        Duration in seconds as integer

    Raises:
        ValueError: If end_time is before start_time

    Example:
        >>> start = datetime(2026, 1, 20, 10, 0, 0)
        >>> end = datetime(2026, 1, 20, 10, 5, 30)
        >>> calculate_call_duration(start, end)
        330
    """
    if end_time < start_time:
        raise ValueError("end_time must be after start_time")

    return int((end_time - start_time).total_seconds())
```

#### Inline Comments

```python
# Use inline comments sparingly, only for non-obvious logic
# Prefer self-documenting code over comments

# Good: Comment explains WHY, not WHAT
# Twilio requires μ-law encoding for audio streams
audio_encoded = audioop.lin2ulaw(audio_bytes, 2)

# Bad: Comment restates the code
# Convert audio to μ-law
audio_encoded = audioop.lin2ulaw(audio_bytes, 2)
```

### Markdown Documentation

- Use clear hierarchical headings (`#`, `##`, `###`)
- Include code examples with syntax highlighting
- Add tables for structured data
- Link to related documentation
- Keep line length under 100 characters for readability

### API Documentation

Document all API endpoints in `docs/API_REFERENCE.md`:

```markdown
### POST /api/v1/appointments

Create a new appointment.

**Request Body:**
```json
{
  "patient_id": 123,
  "appointment_time": "2026-01-25T14:30:00Z",
  "doctor_id": 45,
  "reason": "Annual checkup"
}
```

**Response (201 Created):**
```json
{
  "id": 789,
  "patient_id": 123,
  "appointment_time": "2026-01-25T14:30:00Z",
  "doctor_id": 45,
  "reason": "Annual checkup",
  "status": "scheduled",
  "created_at": "2026-01-20T10:00:00Z"
}
```

**Error Responses:**
- `400 Bad Request`: Invalid input data
- `404 Not Found`: Patient or doctor not found
- `409 Conflict`: Appointment time already booked
```

---

## Pull Request Process

### Before Submitting

Ensure your PR meets these requirements:

- [ ] Code follows style guidelines (passes `flake8`, `mypy`, `black`, `isort`)
- [ ] All tests pass locally (`pytest tests/`)
- [ ] Test coverage is at least 80%
- [ ] Documentation updated for new features
- [ ] Commit messages follow Conventional Commits format
- [ ] No merge conflicts with `main` branch
- [ ] PR description clearly explains changes

### PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix (non-breaking change fixing an issue)
- [ ] New feature (non-breaking change adding functionality)
- [ ] Breaking change (fix or feature causing existing functionality to not work as expected)
- [ ] Documentation update

## Motivation
Why is this change needed? What problem does it solve?

## Changes Made
- List key changes
- Include relevant file paths
- Mention any architectural decisions

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed

**Test coverage:** XX%

## Screenshots/Logs
If applicable, add screenshots or log outputs

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests pass locally
```

### Review Process

1. **Automated Checks**: GitHub Actions will run tests and linting
2. **Code Review**: At least one maintainer will review your code
3. **Feedback**: Address review comments by pushing new commits
4. **Approval**: Once approved, a maintainer will merge your PR

### After Merge

- Your contribution will be included in the next release
- You'll be added to the contributors list
- We may reach out for follow-up improvements or related features

---

## Community

### Communication Channels

- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: General questions and discussions
- **Discord**: Real-time chat with contributors (link in README)
- **Email**: support@vaani-engine.com for private inquiries

### Getting Help

If you're stuck or have questions:

1. Check existing [documentation](docs/)
2. Search [GitHub Issues](https://github.com/mdkashifakram/VAANI_Engine/issues)
3. Ask in [GitHub Discussions](https://github.com/mdkashifakram/VAANI_Engine/discussions)
4. Join our [Discord server](https://discord.gg/vaani-engine)

### Recognition

We value all contributions. Contributors are recognized through:

- Listing in `CONTRIBUTORS.md`
- Mentions in release notes
- "Contributor" badge on GitHub
- Invitation to join our community calls

---

## Development Resources

### Useful Links

- **FastAPI Documentation**: https://fastapi.tiangolo.com/
- **Twilio Voice API**: https://www.twilio.com/docs/voice
- **Deepgram API**: https://developers.deepgram.com/
- **SQLAlchemy Async**: https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html

### Recommended Tools

- **IDE**: VS Code with Python, Pylance extensions
- **API Testing**: Postman or httpie
- **Database Client**: pgAdmin or DBeaver
- **Git GUI**: GitKraken or GitHub Desktop (optional)

---

## License

By contributing to VAANI Engine, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing to VAANI Engine and helping improve healthcare accessibility in India!**
