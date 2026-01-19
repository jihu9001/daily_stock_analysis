# AGENTS.md - Code Guidelines for A股智能分析系统

## Build / Lint / Test Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run main application
python main.py

# Run with specific options
python main.py --market-review          # Market review only
python main.py --no-market-review       # Stocks only
python main.py --debug                  # Debug mode with verbose logging
python main.py --schedule               # Scheduled task mode
python main.py --webui                  # WebUI management interface

# No dedicated test framework - test via main.py with --debug
python main.py --debug --dry-run        # Dry run without AI analysis
```

## Code Style Guidelines

### File Headers
```python
# -*- coding: utf-8 -*-
"""
===================================
模块名称 - 功能描述
===================================

职责：
1. 功能点一
2. 功能点二
"""
```

### Imports
- Standard library first, then third-party, then local
- Group imports by type with blank lines between groups
- Use `from typing import ...` for type hints
- Avoid `import *`

```python
import os
from pathlib import Path
from typing import List, Optional, Dict, Any
from dataclasses import dataclass, field

import requests
from dotenv import load_dotenv

from config import get_config
```

### Type Annotations
- Use type hints for all function signatures
- Use `Optional[T]` instead of `Union[T, None]`
- Use `List[T]`, `Dict[K, V]` for collections
- Complex types: `Dict[str, Any]`, `List[Dict[str, Any]]`

```python
def analyze_stock(self, code: str) -> Optional[AnalysisResult]:
    ...
```

### Naming Conventions
- **Classes**: PascalCase (`Config`, `AnalysisResult`, `NotificationService`)
- **Functions/Variables**: snake_case (`get_config`, `stock_list`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_RETRIES`, `DEFAULT_TIMEOUT`)
- **Private methods/attributes**: prefix with `_` (`_load_from_env`, `_instance`)
- **Module-level constants**: UPPER_SNAKE_CASE in module scope

### Data Classes (Config, Results)
- Use `@dataclass` decorator
- Define types explicitly for all fields
- Provide default values where appropriate
- Use `field(default_factory=list)` for mutable defaults

```python
@dataclass
class Config:
    stock_list: List[str] = field(default_factory=list)
    gemini_api_key: Optional[str] = None
    max_workers: int = 3
```

### Error Handling
- Use `try/except` with specific exception types
- Log errors with `logger.error()` before re-raising or returning False
- Use `tenacity` library for retry logic (exponential backoff)
- Return `bool` for operations that may fail

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential())
def fetch_data(self, url: str) -> Optional[Dict]:
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        logger.error(f"Request failed: {e}")
        return None
```

### Logging
- Use module-level logger: `logger = logging.getLogger(__name__)`
- Log levels: `DEBUG` (details), `INFO` (progress), `WARNING` (issues), `ERROR` (failures)
- Include context in log messages: `logger.info(f"Processing {stock_code}...")`

### Configuration
- Use singleton pattern for Config (`Config.get_instance()`)
- Load from `.env` file using `python-dotenv`
- Validate configuration with `config.validate()` method
- Provide sensible defaults for all settings

### Notifications
- Support multiple channels (wechat, feishu, telegram, email, custom webhook)
- Use `NotificationService` for all notification logic
- Auto-detect available channels based on configuration
- Implement chunking for messages exceeding platform limits

### Strings and Encoding
- Use f-strings for string interpolation
- All strings: UTF-8 encoding
- Chinese comments/descriptions are acceptable
- Use `ensure_ascii=False` when serializing JSON with Chinese

### Code Structure
- Single responsibility: each module has one clear purpose
- Separation of concerns: config, analysis, notification, storage layers
- No hardcoded values: use Config or constants
- No secrets in code: use environment variables

### Comments
- Avoid unnecessary comments; let code explain itself
- Use comments for non-obvious logic or workarounds
- Document public APIs with docstrings
- Include `Args` and `Returns` for function docstrings

```python
def process_data(self, data: Dict[str, Any]) -> bool:
    """
    Process incoming data and persist to database.

    Args:
        data: Raw data dictionary from API response

    Returns:
        True if processing succeeded, False otherwise
    """
```
