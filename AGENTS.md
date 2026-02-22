# AGENTS.md - Coding Guidelines for Kittygram

## Project Overview
Full-stack Django + React application with Docker containerization. Uses UV package manager, PostgreSQL, and GitHub Actions CI/CD.

## Build/Lint/Test Commands

### Backend (UV Package Manager)
```bash
cd backend

# Install dependencies
uv sync --dev

# Linting
uv run flake8 .

# Run all tests
uv run pytest ../tests/

# Run single test
uv run pytest ../tests/test_files.py::test_backend_dockerfile_exists -v

# Run Django tests
uv run python manage.py test

# Update lock file
uv lock
```

### Docker Commands
```bash
# Local development
docker-compose up -d
docker-compose down

# Production (after pushing to Docker Hub)
docker-compose -f docker-compose.production.yml up -d
docker-compose -f docker-compose.production.yml exec backend python manage.py migrate
docker-compose -f docker-compose.production.yml exec backend python manage.py collectstatic --no-input
```

### Testing Specific Components
```bash
# Test specific file
cd backend && uv run pytest ../tests/test_files.py -v

# Test with specific marker
cd backend && uv run pytest ../tests/ -k "test_infra" -v

# Django unit tests
cd backend && uv run python manage.py test cats.tests.CatsAPITestCase.test_list_exists
```

## Code Style Guidelines

### General
- **Line length:** 79 characters maximum (PEP8)
- **Quotes:** Use double quotes for strings (`"string"`)
- **Indentation:** 4 spaces (no tabs)
- **Trailing whitespace:** Remove before committing
- **End of file:** Ensure newline at end of file

### Imports (isort style)
Order: Standard library → Third party → Django → Local
```python
# Standard library
import base64
import datetime as dt

# Third party
import webcolors
from rest_framework import serializers

# Django
from django.contrib.auth import get_user_model
from django.db import models

# Local
from .models import Achievement, Cat
```

### Naming Conventions
- **Classes:** PascalCase (e.g., `CatViewSet`, `AchievementSerializer`)
- **Functions/Methods:** snake_case (e.g., `perform_create`, `get_age`)
- **Variables:** snake_case (e.g., `birth_year`, `achievements_data`)
- **Constants:** UPPER_CASE (e.g., `MAX_LENGTH`)
- **Private:** Leading underscore (e.g., `_private_method`)

### Django Patterns

**Models:**
- Always define `__str__` method
- Use related_name for ForeignKey/ManyToMany
- Keep field definitions on single lines
```python
class Cat(models.Model):
    name = models.CharField(max_length=16)
    owner = models.ForeignKey(
        User, related_name='cats', on_delete=models.CASCADE
    )
    
    def __str__(self):
        return self.name
```

**Serializers:**
- Use `serializers.SerializerMethodField` for computed fields
- Override `create`/`update` for complex logic
- Use `Meta` class with explicit fields tuple

**Views:**
- Use Django REST Framework ViewSets
- Set `pagination_class` explicitly
- Override `perform_create` for automatic owner assignment
```python
class CatViewSet(viewsets.ModelViewSet):
    queryset = Cat.objects.all()
    serializer_class = CatSerializer
    pagination_class = PageNumberPagination

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

### Error Handling
- Use Django REST Framework's `ValidationError` for API errors
- Catch specific exceptions, not bare `except:`
- Raise meaningful error messages in Russian if needed
```python
try:
    data = webcolors.hex_to_name(data)
except ValueError:
    raise serializers.ValidationError('Для этого цвета нет имени')
```

### Type Hints
Not required but welcome for complex functions:
```python
def get_age(self, obj: Cat) -> int:
    return dt.datetime.now().year - obj.birth_year
```

### Flake8 Configuration
Exclusions configured in `backend/.flake8`:
- `.venv`, `.git`, `__pycache__`
- `build`, `dist`, `*.egg-info`
- `.tox`, `.pytest_cache`, `.mypy_cache`
- `migrations` (auto-generated)

### Docker Best Practices
- Use slim images (e.g., `python:3.9-slim`)
- Install dependencies before copying source
- Keep containers stateless
- Use `CMD` with exec form: `["gunicorn", "--bind", "0.0.0.0:8000", "..."]`

### Environment Variables
Never hardcode secrets. Use `os.getenv()` with defaults:
```python
SECRET_KEY = os.getenv("SECRET_KEY", "fallback-for-dev")
DEBUG = os.getenv("DEBUG", "True") == "True"
```

### Testing
- Use pytest for integration tests
- Use Django TestCase for unit tests
- Fixtures defined in `tests/conftest.py`
- Integration tests validate deployed URLs (see `tests.yml`)

## CI/CD Notes
- GitHub Actions runs on every push to `main`
- Requires secrets: DOCKER_USERNAME, DOCKER_PASSWORD, HOST, USER, SSH_KEY
- Workflow: Tests → Build & Push Images → Deploy to Server
- Production uses `docker-compose.production.yml` (pre-built images)
