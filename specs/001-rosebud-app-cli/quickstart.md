# Quick Start Guide: Daybook CLI

**Created**: 2025-10-07
**Purpose**: Quick start guide for developers implementing the AI journal CLI tool

## Overview

Daybook CLI是一个面向程序员的AI日记CLI工具，帮助用户通过智能问答和个人成长追踪来加速个人发展。本指南将帮助开发者快速搭建开发环境并开始实现功能。

## Prerequisites

### System Requirements

- **Operating System**: Linux, macOS, Windows 10+
- **Python**: 3.11 or higher
- **Memory**: 4GB RAM minimum (8GB recommended)
- **Storage**: 500MB free space
- **Network**: Internet connection for AI model download (initial setup only)

### Required Software

1. **Python 3.11+**
   ```bash
   # Verify Python version
   python --version

   # Install Python (if needed)
   # Ubuntu/Debian: sudo apt update && sudo apt install python3.11
   # macOS: brew install python@3.11
   # Windows: Download from python.org
   ```

2. **Git**
   ```bash
   git --version
   ```

3. **Ollama AI Service**
   ```bash
   # Install Ollama
   curl -fsSL https://ollama.ai/install.sh | sh

   # Start Ollama service
   ollama serve

   # Pull recommended model
   ollama pull qwen2.5:3b
   ```

## Development Setup

### 1. Clone Repository

```bash
git clone <repository-url>
cd daybook
```

### 2. Create Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# Linux/macOS:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# Verify activation
which python  # should show venv path
```

### 3. Install Dependencies

```bash
# Install development dependencies
pip install -r requirements-dev.txt

# Install package in development mode
pip install -e .
```

### 4. Verify Installation

```bash
# Test basic functionality
daybook --help

# Check AI service connection
daybook check-ai

# Verify all dependencies
python -m pytest tests/test_setup.py -v
```

## Project Structure

```
daybook/
├── src/
│   └── daybook/
│       ├── __init__.py
│       ├── cli/
│       │   ├── __init__.py
│       │   ├── main.py          # Main CLI entry point
│       │   ├── init.py          # Init command
│       │   ├── log.py           # Log command
│       │   ├── ask.py           # AI question command
│       │   └── mem.py           # Memory command
│       ├── core/
│       │   ├── __init__.py
│       │   ├── journal.py       # Journal management
│       │   ├── memory.py        # Long-term memory
│       │   ├── ai_service.py    # AI service integration
│       │   └── scheduler.py     # Task scheduling
│       ├── models/
│       │   ├── __init__.py
│       │   ├── config.py        # Configuration models
│       │   ├── entry.py         # Journal entry models
│       │   └── memory.py        # Memory models
│       ├── utils/
│       │   ├── __init__.py
│       │   ├── file_utils.py    # File operations
│       │   ├── time_utils.py    # Time handling
│       │   └── security.py      # Security utilities
│       └── config/
│           ├── __init__.py
│           └── settings.py      # Settings management
├── tests/
│   ├── unit/                   # Unit tests
│   ├── integration/            # Integration tests
│   └── e2e/                   # End-to-end tests
├── docs/                      # Documentation
├── requirements.txt           # Runtime dependencies
├── requirements-dev.txt       # Development dependencies
├── pyproject.toml            # Project configuration
├── README.md                 # Project README
└── .gitignore               # Git ignore file
```

## Configuration Files

### requirements.txt

```txt
# Core dependencies
typer>=0.9.0
pydantic>=2.0.0
toml>=0.10.2
httpx>=0.25.0
ollama>=0.6.0
apscheduler>=3.10.0
pytz>=2023.3
python-dateutil>=2.8.0

# Optional dependencies
cryptography>=41.0.0  # For encryption support
```

### requirements-dev.txt

```txt
# Include runtime dependencies
-r requirements.txt

# Development tools
pytest>=7.4.0
pytest-asyncio>=0.21.0
pytest-cov>=4.1.0
black>=23.0.0
mypy>=1.5.0
pre-commit>=3.3.0
isort>=5.12.0
flake8>=6.0.0

# Documentation
mkdocs>=1.5.0
mkdocs-material>=9.2.0
```

### pyproject.toml

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "daybook"
version = "0.1.0"
description = "AI journal CLI tool for programmers"
authors = [{name = "Your Name", email = "your.email@example.com"}]
license = {text = "MIT"}
readme = "README.md"
requires-python = ">=3.11"
classifiers = [
    "Development Status :: 3 - Alpha",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]

[project.scripts]
daybook = "daybook.cli.main:app"

[project.urls]
Homepage = "https://github.com/yourusername/daybook"
Repository = "https://github.com/yourusername/daybook"
Documentation = "https://daybook.readthedocs.io"

[tool.setuptools.packages.find]
where = ["src"]

[tool.black]
line-length = 88
target-version = ['py311']

[tool.isort]
profile = "black"
multi_line_output = 3

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = "--cov=src/daybook --cov-report=html --cov-report=term-missing"

[tool.coverage.run]
source = ["src/daybook"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
]
```

## Development Workflow

### 1. Feature Development

```bash
# Create feature branch
git checkout -b feature/<feature-name>

# Run tests before starting
python -m pytest

# Make changes
# ...

# Run tests frequently
python -m pytest tests/unit/ -v

# Run integration tests
python -m pytest tests/integration/ -v

# Check code quality
black src/ tests/
isort src/ tests/
mypy src/
flake8 src/

# Run full test suite
python -m pytest

# Commit changes
git add .
git commit -m "feat: add new feature"

# Push and create PR
git push origin feature/<feature-name>
```

### 2. Testing Strategy

```bash
# Run all tests
pytest

# Run unit tests only
pytest tests/unit/

# Run with coverage
pytest --cov=src/daybook --cov-report=html

# Run specific test file
pytest tests/unit/test_journal.py -v

# Run tests with specific pattern
pytest -k "test_log_entry" -v
```

### 3. Code Quality

```bash
# Format code
black src/ tests/

# Sort imports
isort src/ tests/

# Type checking
mypy src/

# Linting
flake8 src/

# Run all quality checks
pre-commit run --all-files
```

## Core Implementation Steps

### Step 1: Basic CLI Structure

Create the main CLI entry point:

```python
# src/daybook/cli/main.py
import typer
from typing import Optional

app = typer.Typer(
    name="daybook",
    help="AI journal CLI tool for programmers",
    add_completion=False,
)

@app.command()
def init(
    path: str = typer.Option(..., help="Path to daybook directory"),
    timezone: Optional[str] = typer.Option(None, help="Timezone"),
    model: Optional[str] = typer.Option(None, help="AI model"),
):
    """Initialize daybook configuration"""
    # Implementation
    pass

@app.command()
def log(message: str):
    """Add a journal entry"""
    # Implementation
    pass

@app.command()
def ask():
    """Start AI conversation"""
    # Implementation
    pass

@app.command()
def mem(action: str, content: Optional[str] = None):
    """Manage long-term memory"""
    # Implementation
    pass

if __name__ == "__main__":
    app()
```

### Step 2: Configuration Management

Implement configuration handling:

```python
# src/daybook/config/settings.py
from pathlib import Path
from typing import Optional
import toml
from pydantic import BaseModel

class UserSettings(BaseModel):
    daybook_path: str
    timezone: str = "UTC"
    ai_model: str = "qwen2.5:3b"
    reminder_enabled: bool = True
    reminder_time: str = "23:00"

class SettingsManager:
    def __init__(self, config_path: Path):
        self.config_path = config_path

    def load_settings(self) -> UserSettings:
        if not self.config_path.exists():
            raise FileNotFoundError("Configuration not found. Run 'daybook init' first.")

        with open(self.config_path, 'r') as f:
            data = toml.load(f)
            return UserSettings(**data)

    def save_settings(self, settings: UserSettings) -> None:
        self.config_path.parent.mkdir(parents=True, exist_ok=True)
        with open(self.config_path, 'w') as f:
            toml.dump(settings.dict(), f)
```

### Step 3: Journal Management

Implement basic journal functionality:

```python
# src/daybook/core/journal.py
from pathlib import Path
from datetime import datetime
from typing import List
from ..models.entry import JournalEntry, EntryType

class JournalService:
    def __init__(self, daybook_path: Path):
        self.daybook_path = daybook_path
        self.journals_dir = daybook_path / "journals"
        self.journals_dir.mkdir(parents=True, exist_ok=True)

    def add_entry(self, content: str, entry_type: EntryType = EntryType.LOG) -> JournalEntry:
        """Add a new journal entry"""
        now = datetime.now()
        entry = JournalEntry(
            timestamp=now,
            entry_type=entry_type,
            content=content
        )

        # Get today's journal file
        journal_file = self._get_journal_file(now)
        self._write_entry_to_file(journal_file, entry)

        return entry

    def _get_journal_file(self, date: datetime) -> Path:
        """Get the journal file for a specific date"""
        filename = date.strftime("%Y-%m-%d.md")
        return self.journals_dir / filename

    def _write_entry_to_file(self, file_path: Path, entry: JournalEntry):
        """Write entry to markdown file"""
        with open(file_path, 'a', encoding='utf-8') as f:
            timestamp_str = entry.timestamp.strftime("%H:%M:%S")
            f.write(f"\n## {timestamp_str}\n")
            f.write(f"{entry.content}\n")
```

### Step 4: AI Integration

Implement AI service integration:

```python
# src/daybook/core/ai_service.py
import httpx
from typing import List, Optional
from ..models.config import UserSettings
from ..models.entry import QuestionContext, GeneratedQuestion

class AIService:
    def __init__(self, settings: UserSettings):
        self.settings = settings
        self.client = httpx.AsyncClient(
            base_url=settings.ai_base_url,
            timeout=settings.timeout_seconds
        )

    async def health_check(self) -> bool:
        """Check if AI service is available"""
        try:
            response = await self.client.get("/api/tags")
            return response.status_code == 200
        except Exception:
            return False

    async def generate_question(self, context: QuestionContext) -> GeneratedQuestion:
        """Generate a personalized question"""
        prompt = self._build_prompt(context)

        response = await self.client.post(
            "/api/generate",
            json={
                "model": self.settings.ai_model,
                "prompt": prompt,
                "stream": False
            }
        )

        if response.status_code != 200:
            raise Exception(f"AI service error: {response.status_code}")

        result = response.json()
        question_text = result.get("response", "").strip()

        return GeneratedQuestion(
            question=question_text,
            question_type="reflection",
            relevance_score=0.8,
            model_used=self.settings.ai_model,
            generation_prompt=prompt
        )

    def _build_prompt(self, context: QuestionContext) -> str:
        """Build AI prompt from context"""
        prompt = f"""
Based on the following journal entries and memories, generate one thoughtful question for personal reflection:

Today's entries:
{chr(10).join([f"- {entry.content}" for entry in context.today_entries])}

Relevant memories:
{chr(10).join([f"- {memory.content}" for memory in context.recent_memories])}

Generate one specific, thought-provoking question that helps the user reflect on their day and personal growth.
"""
        return prompt.strip()
```

## Testing Examples

### Unit Tests

```python
# tests/unit/test_journal.py
import pytest
from datetime import datetime
from pathlib import Path
from daybook.core.journal import JournalService
from daybook.models.entry import EntryType

class TestJournalService:
    @pytest.fixture
    def temp_daybook_path(self, tmp_path):
        return tmp_path / "test_daybook"

    @pytest.fixture
    def journal_service(self, temp_daybook_path):
        return JournalService(temp_daybook_path)

    def test_add_entry(self, journal_service):
        content = "Test journal entry"
        entry = journal_service.add_entry(content)

        assert entry.content == content
        assert entry.entry_type == EntryType.LOG
        assert entry.timestamp is not None

    def test_journal_file_creation(self, journal_service, temp_daybook_path):
        content = "Test entry"
        journal_service.add_entry(content)

        journal_files = list((temp_daybook_path / "journals").glob("*.md"))
        assert len(journal_files) == 1

        with open(journal_files[0], 'r') as f:
            file_content = f.read()
            assert content in file_content
```

### Integration Tests

```python
# tests/integration/test_ai_service.py
import pytest
from daybook.core.ai_service import AIService
from daybook.models.config import UserSettings

class TestAIServiceIntegration:
    @pytest.fixture
    def ai_settings(self):
        return UserSettings(
            daybook_path="/tmp/test",
            ai_base_url="http://localhost:11434",
            ai_model="qwen2.5:3b"
        )

    @pytest.fixture
    def ai_service(self, ai_settings):
        return AIService(ai_settings)

    @pytest.mark.asyncio
    async def test_health_check(self, ai_service):
        # This test requires Ollama to be running
        is_healthy = await ai_service.health_check()
        assert isinstance(is_healthy, bool)

    @pytest.mark.asyncio
    async def test_question_generation(self, ai_service):
        from daybook.models.entry import QuestionContext, JournalEntry, Memory

        context = QuestionContext(
            current_date=datetime.now(),
            today_entries=[
                JournalEntry(
                    timestamp=datetime.now(),
                    entry_type=EntryType.LOG,
                    content="Learned about Python async programming"
                )
            ],
            recent_memories=[]
        )

        # This test requires Ollama to be running with the model
        try:
            question = await ai_service.generate_question(context)
            assert len(question.question) > 0
            assert question.model_used == "qwen2.5:3b"
        except Exception as e:
            pytest.skip(f"AI service not available: {e}")
```

## Debugging and Troubleshooting

### Common Issues

1. **AI Service Connection Failed**
   ```bash
   # Check Ollama status
   ollama list

   # Restart Ollama
   ollama serve

   # Check model availability
   ollama pull qwen2.5:3b
   ```

2. **Configuration File Not Found**
   ```bash
   # Initialize daybook
   daybook init --path ~/Documents/daybook

   # Check config file
   ls ~/Documents/daybook/config/
   ```

3. **Permission Errors**
   ```bash
   # Check directory permissions
   ls -la ~/Documents/daybook/

   # Fix permissions if needed
   chmod 755 ~/Documents/daybook/
   ```

### Debug Mode

Enable debug logging:

```bash
# Set debug environment variable
export DAYBOOK_DEBUG=1

# Run with verbose output
daybook --verbose log "test message"
```

### Test Coverage

Generate coverage report:

```bash
pytest --cov=src/daybook --cov-report=html

# View coverage report
open htmlcov/index.html
```

## Next Steps

1. **Implement Core Commands**: Complete the init, log, ask, and mem commands
2. **Add AI Integration**: Implement full AI service integration
3. **Add Scheduling**: Implement reminder scheduling functionality
4. **Add Search**: Implement journal search functionality
5. **Add Backup**: Implement backup and restore features
6. **Add Tests**: Comprehensive test coverage
7. **Add Documentation**: Complete user and developer documentation

## Resources

- [Typer Documentation](https://typer.tiangolo.com/)
- [Pydantic Documentation](https://docs.pydantic.dev/)
- [Ollama Documentation](https://github.com/ollama/ollama)
- [pytest Documentation](https://docs.pytest.org/)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass
6. Submit a pull request

For detailed contribution guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).