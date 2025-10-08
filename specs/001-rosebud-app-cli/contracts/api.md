# API Contracts: Daybook CLI

**Created**: 2025-10-07
**Purpose**: Define internal API contracts and interfaces for the AI journal CLI tool

## Overview

本文档定义了Daybook CLI工具的内部API合约，包括命令行接口、服务接口和数据访问接口。采用面向对象设计，确保模块间的松耦合和高内聚。

## CLI Command Contracts

### Command Interface Definition

```python
from abc import ABC, abstractmethod
from typing import List, Optional, Dict, Any
from pydantic import BaseModel

class CommandResult(BaseModel):
    """命令执行结果"""
    success: bool
    message: str
    data: Optional[Dict[str, Any]] = None
    errors: List[str] = []

class BaseCommand(ABC):
    """命令基类"""

    @abstractmethod
    def execute(self, **kwargs) -> CommandResult:
        """执行命令"""
        pass

    @abstractmethod
    def validate_args(self, **kwargs) -> bool:
        """验证参数"""
        pass
```

### 1. Init Command Contract

```python
from pathlib import Path
from typing import Optional

class InitArgs(BaseModel):
    """初始化命令参数"""
    path: str = Field(..., description="日记存储路径")
    timezone: Optional[str] = Field(None, description="时区")
    model: Optional[str] = Field(None, description="AI模型")
    reminder_time: Optional[str] = Field(None, description="提醒时间")

class InitCommand(BaseCommand):
    """初始化命令合约"""

    def validate_args(self, **kwargs) -> bool:
        """验证初始化参数"""
        # 验证路径有效性
        # 验证时区格式
        # 验证模型名称
        # 验证时间格式
        pass

    def execute(self, **kwargs) -> CommandResult:
        """执行初始化"""
        # 1. 验证参数
        # 2. 创建目录结构
        # 3. 生成配置文件
        # 4. 初始化索引
        # 5. 验证AI服务连接
        pass

# Contract Specification
class InitCommandSpec:
    """初始化命令规范"""

    # 输入规范
    INPUT_SCHEMA = {
        "path": {
            "type": "string",
            "required": True,
            "description": "日记存储路径，必须是有效且可写的目录"
        },
        "timezone": {
            "type": "string",
            "required": False,
            "default": "UTC",
            "pattern": r"^[A-Za-z_]+/[A-Za-z_]+$",
            "description": "时区标识符，如 Asia/Shanghai"
        },
        "model": {
            "type": "string",
            "required": False,
            "default": "gemma3:270m",
            "enum": ["gemma3:270m", "qwen2.5:3b", "qwen2.5:7b", "phi3:mini"],
            "description": "默认AI模型"
        },
        "reminder_time": {
            "type": "string",
            "required": False,
            "default": "23:00",
            "pattern": r"^[0-9]{2}:[0-9]{2}$",
            "description": "每日提醒时间，格式HH:MM"
        }
    }

    # 输出规范
    OUTPUT_SCHEMA = {
        "success": {
            "type": "boolean",
            "description": "初始化是否成功"
        },
        "message": {
            "type": "string",
            "description": "执行结果描述"
        },
        "data": {
            "type": "object",
            "properties": {
                "config_path": {"type": "string"},
                "daybook_path": {"type": "string"},
                "created_files": {"type": "array", "items": {"type": "string"}}
            }
        }
    }

    # 错误情况
    ERROR_CASES = [
        ("INVALID_PATH", "路径无效或无写入权限"),
        ("TIMEZONE_INVALID", "时区格式错误"),
        ("MODEL_UNAVAILABLE", "指定模型不可用"),
        ("DIRECTORY_EXISTS", "目标目录已存在"),
        ("AI_SERVICE_UNAVAILABLE", "AI服务连接失败")
    ]
```

### 2. Log Command Contract

```python
class LogArgs(BaseModel):
    """日志命令参数"""
    message: str = Field(..., description="日志内容")
    tags: Optional[List[str]] = Field(None, description="标签列表")
    mood: Optional[str] = Field(None, description="情绪标签")

class LogCommand(BaseCommand):
    """日志命令合约"""

    def validate_args(self, **kwargs) -> bool:
        """验证日志参数"""
        # 验证消息内容不为空
        # 验证标签格式
        # 验证情绪标签
        pass

    def execute(self, **kwargs) -> CommandResult:
        """执行日志记录"""
        # 1. 获取当前日期
        # 2. 创建/打开当日日记文件
        # 3. 添加时间戳和条目
        # 4. 更新搜索索引
        # 5. 返回执行结果
        pass

class LogCommandSpec:
    """日志命令规范"""

    INPUT_SCHEMA = {
        "message": {
            "type": "string",
            "required": True,
            "min_length": 1,
            "max_length": 5000,
            "description": "日志内容，支持多行文本"
        },
        "tags": {
            "type": "array",
            "required": False,
            "items": {"type": "string", "pattern": r"^[a-zA-Z0-9_\u4e00-\u9fff]+$"},
            "max_items": 10,
            "description": "标签列表，支持中文和英文"
        },
        "mood": {
            "type": "string",
            "required": False,
            "enum": ["happy", "sad", "angry", "neutral", "excited", "anxious", "calm"],
            "description": "情绪标签"
        }
    }

    OUTPUT_SCHEMA = {
        "success": {"type": "boolean"},
        "message": {"type": "string"},
        "data": {
            "type": "object",
            "properties": {
                "entry_id": {"type": "string"},
                "timestamp": {"type": "string"},
                "file_path": {"type": "string"},
                "word_count": {"type": "integer"}
            }
        }
    }
```

### 3. Ask Command Contract

```python
class AskArgs(BaseModel):
    """AI问答命令参数"""
    context_type: Optional[str] = Field(None, description="上下文类型")
    follow_up: bool = Field(False, description="是否为追问")

class AskCommand(BaseCommand):
    """AI问答命令合约"""

    def validate_args(self, **kwargs) -> bool:
        """验证问答参数"""
        pass

    def execute(self, **kwargs) -> CommandResult:
        """执行AI问答"""
        # 1. 加载今日日记内容
        # 2. 获取相关长期记忆
        # 3. 构建问题上下文
        # 4. 调用AI服务生成问题
        # 5. 保存问题和回答
        # 6. 更新记忆库
        pass

class AskCommandSpec:
    """AI问答命令规范"""

    INPUT_SCHEMA = {
        "context_type": {
            "type": "string",
            "required": False,
            "enum": ["learning", "health", "relationships", "goals", "reflection"],
            "description": "指定问题类型上下文"
        },
        "follow_up": {
            "type": "boolean",
            "required": False,
            "default": False,
            "description": "是否为追问模式"
        }
    }

    OUTPUT_SCHEMA = {
        "success": {"type": "boolean"},
        "message": {"type": "string"},
        "data": {
            "type": "object",
            "properties": {
                "question": {"type": "string"},
                "question_type": {"type": "string"},
                "context_summary": {"type": "string"},
                "ai_model": {"type": "string"},
                "generation_time": {"type": "number"}
            }
        }
    }
```

### 4. Memory Command Contract

```python
class MemoryArgs(BaseModel):
    """记忆命令参数"""
    action: str = Field(..., description="操作类型")
    content: Optional[str] = Field(None, description="记忆内容")
    category: Optional[str] = Field(None, description="记忆类别")
    tags: Optional[List[str]] = Field(None, description="标签")

class MemoryCommand(BaseCommand):
    """记忆命令合约"""

    def validate_args(self, **kwargs) -> bool:
        """验证记忆参数"""
        pass

    def execute(self, **kwargs) -> CommandResult:
        """执行记忆操作"""
        # 1. 解析操作类型
        # 2. 执行相应操作 (add/list/search/update/delete)
        # 3. 更新记忆索引
        # 4. 返回结果
        pass

class MemoryCommandSpec:
    """记忆命令规范"""

    INPUT_SCHEMA = {
        "action": {
            "type": "string",
            "required": True,
            "enum": ["add", "list", "search", "update", "delete"],
            "description": "记忆操作类型"
        },
        "content": {
            "type": "string",
            "required": False,
            "min_length": 1,
            "max_length": 1000,
            "description": "记忆内容，add和update操作必需"
        },
        "category": {
            "type": "string",
            "required": False,
            "enum": ["personal", "goals", "preferences", "relationships", "skills", "habits", "values", "other"],
            "description": "记忆类别"
        },
        "tags": {
            "type": "array",
            "required": False,
            "items": {"type": "string"},
            "max_items": 5,
            "description": "记忆标签"
        }
    }
```

## Service Layer Contracts

### AI Service Contract

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any

class AIServiceInterface(ABC):
    """AI服务接口合约 - 专注于问题生成"""

    @abstractmethod
    async def health_check(self) -> bool:
        """检查AI服务健康状态"""
        pass

    @abstractmethod
    async def generate_question(
        self,
        context: QuestionContext,
        question_count: int = 1
    ) -> List[GeneratedQuestion]:
        """生成个性化问题（AI Agent核心功能）"""
        pass

    # 注意：AI Agent只问问题，不提供回答
    # 用户回答由用户自己提供，用于后续问题生成的上下文

    @abstractmethod
    async def list_available_models(self) -> List[str]:
        """获取可用模型列表"""
        pass

class AIServiceContract:
    """AI服务合约规范"""

    # 性能要求
    PERFORMANCE_REQUIREMENTS = {
        "health_check_timeout": 5.0,  # 秒
        "question_generation_timeout": 30.0,  # 秒
        "response_generation_timeout": 30.0,  # 秒
        "max_retry_attempts": 3
    }

    # 质量要求 - 专注于问题生成
    QUALITY_REQUIREMENTS = {
        "question_relevance_threshold": 0.8,  # 相关性阈值（更高要求）
        "min_question_length": 15,  # 最短问题长度
        "max_question_length": 200,  # 最长问题长度
        "question_depth_requirement": "引导性",  # 问题必须具有引导思考的特性
        "context_usage_weight": 0.7,  # 上下文权重（昨日回答重要）
        "forbidden_content": ["敏感话题", "违法内容", "提供直接答案"]  # 禁止内容
    }

    # 错误处理
    ERROR_HANDLING = {
        "service_unavailable": "AI服务暂时不可用，请稍后重试",
        "model_not_found": "指定模型不可用，请检查配置",
        "generation_failed": "问题生成失败，请重试",
        "timeout": "请求超时，请检查网络连接"
    }
```

### Storage Service Contract

```python
class StorageServiceInterface(ABC):
    """存储服务接口合约"""

    @abstractmethod
    def save_journal_entry(self, entry: JournalEntry) -> bool:
        """保存日记条目"""
        pass

    @abstractmethod
    def load_daily_journal(self, date: datetime) -> Optional[DailyJournal]:
        """加载每日日记"""
        pass

    @abstractmethod
    def save_memory(self, memory: Memory) -> bool:
        """保存长期记忆"""
        pass

    @abstractmethod
    def load_memories(self) -> List[Memory]:
        """加载所有记忆"""
        pass

    @abstractmethod
    def search_entries(self, query: str, limit: int) -> List[Dict[str, Any]]:
        """搜索条目"""
        pass

class StorageServiceContract:
    """存储服务合约规范"""

    # 文件操作约束
    FILE_CONSTRAINTS = {
        "max_file_size": 10 * 1024 * 1024,  # 10MB
        "max_entry_length": 5000,  # 单条目最大长度
        "allowed_extensions": [".md", ".json", ".toml"],
        "backup_retention_days": 30
    }

    # 数据完整性要求
    INTEGRITY_REQUIREMENTS = {
        "atomic_write": True,  # 原子写入
        "backup_before_write": True,  # 写入前备份
        "checksum_verification": True,  # 校验和验证
        "file_locking": True  # 文件锁定
    }
```

### Scheduler Service Contract

```python
class SchedulerServiceInterface(ABC):
    """调度服务接口合约"""

    @abstractmethod
    def schedule_daily_reminder(self, time: str, timezone: str) -> bool:
        """安排每日提醒"""
        pass

    @abstractmethod
    def cancel_reminder(self) -> bool:
        """取消提醒"""
        pass

    @abstractmethod
    def is_reminder_active(self) -> bool:
        """检查提醒是否激活"""
        pass

    @abstractmethod
    def get_next_reminder_time(self) -> Optional[datetime]:
        """获取下次提醒时间"""
        pass

class SchedulerServiceContract:
    """调度服务合约规范"""

    # 调度精度要求
    SCHEDULING_REQUIREMENTS = {
        "precision_minutes": 1,  # 调度精度(分钟)
        "max_missed_jobs": 3,  # 最大错过任务数
        "retry_interval_minutes": 5,  # 重试间隔
        "persistence": True  # 持久化
    }

    # 系统集成要求
    SYSTEM_INTEGRATION = {
        "background_service": True,  # 后台服务
        "auto_start": True,  # 自动启动
        "log_rotation": True,  # 日志轮转
        "graceful_shutdown": True  # 优雅关闭
    }
```

## Data Access Contracts

### Repository Pattern

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar, List, Optional

T = TypeVar('T')

class RepositoryInterface(ABC, Generic[T]):
    """仓储接口合约"""

    @abstractmethod
    def save(self, entity: T) -> bool:
        """保存实体"""
        pass

    @abstractmethod
    def find_by_id(self, entity_id: str) -> Optional[T]:
        """根据ID查找实体"""
        pass

    @abstractmethod
    def find_all(self) -> List[T]:
        """查找所有实体"""
        pass

    @abstractmethod
    def delete(self, entity_id: str) -> bool:
        """删除实体"""
        pass

class JournalRepositoryInterface(RepositoryInterface[JournalEntry]):
    """日记仓储接口"""

    @abstractmethod
    def find_by_date(self, date: datetime) -> List[JournalEntry]:
        """根据日期查找条目"""
        pass

    @abstractmethod
    def find_by_type(self, entry_type: EntryType) -> List[JournalEntry]:
        """根据类型查找条目"""
        pass

    @abstractmethod
    def find_by_date_range(
        self,
        start_date: datetime,
        end_date: datetime
    ) -> List[JournalEntry]:
        """根据日期范围查找条目"""
        pass

class MemoryRepositoryInterface(RepositoryInterface[Memory]):
    """记忆仓储接口"""

    @abstractmethod
    def find_by_category(self, category: MemoryCategory) -> List[Memory]:
        """根据类别查找记忆"""
        pass

    @abstractmethod
    def find_by_tags(self, tags: List[str]) -> List[Memory]:
        """根据标签查找记忆"""
        pass

    @abstractmethod
    def search_by_content(self, query: str) -> List[Memory]:
        """根据内容搜索记忆"""
        pass

    @abstractmethod
    def get_recent_memories(self, days: int = 30) -> List[Memory]:
        """获取最近的记忆"""
        pass
```

## Event System Contracts

### Event Interface

```python
from abc import ABC, abstractmethod
from typing import Any, Callable, List
from datetime import datetime

class Event(ABC):
    """事件基类"""

    def __init__(self, timestamp: datetime = None):
        self.timestamp = timestamp or datetime.now()
        self.event_id = self._generate_id()

    @abstractmethod
    def _generate_id(self) -> str:
        """生成事件ID"""
        pass

class EventHandler(ABC):
    """事件处理器接口"""

    @abstractmethod
    def handle(self, event: Event) -> None:
        """处理事件"""
        pass

class EventManagerInterface(ABC):
    """事件管理器接口"""

    @abstractmethod
    def publish(self, event: Event) -> None:
        """发布事件"""
        pass

    @abstractmethod
    def subscribe(self, event_type: type, handler: EventHandler) -> None:
        """订阅事件"""
        pass

    @abstractmethod
    def unsubscribe(self, event_type: type, handler: EventHandler) -> None:
        """取消订阅"""
        pass

# 具体事件类型
class JournalEntryCreatedEvent(Event):
    """日记条目创建事件"""

    def __init__(self, entry: JournalEntry):
        super().__init__()
        self.entry = entry

    def _generate_id(self) -> str:
        return f"journal_created_{self.entry.timestamp.isoformat()}"

class MemoryAddedEvent(Event):
    """记忆添加事件"""

    def __init__(self, memory: Memory):
        super().__init__()
        self.memory = memory

    def _generate_id(self) -> str:
        return f"memory_added_{self.memory.id}"

class QuestionGeneratedEvent(Event):
    """问题生成事件"""

    def __init__(self, question: GeneratedQuestion):
        super().__init__()
        self.question = question

    def _generate_id(self) -> str:
        return f"question_generated_{self.question.generated_at.isoformat()}"

class EventContract:
    """事件系统合约"""

    # 事件处理要求
    EVENT_REQUIREMENTS = {
        "async_handling": True,  # 异步处理
        "error_isolation": True,  # 错误隔离
        "retry_mechanism": True,  # 重试机制
        "event_persistence": False  # 事件持久化(可选)
    }

    # 性能要求
    PERFORMANCE_REQUIREMENTS = {
        "max_event_handlers": 10,  # 最大事件处理器数
        "processing_timeout": 5.0,  # 处理超时(秒)
        "queue_size": 1000  # 事件队列大小
    }
```

## Integration Contracts

### External Service Integration

```python
class OllamaClientInterface(ABC):
    """Ollama客户端接口合约"""

    @abstractmethod
    async def generate(
        self,
        prompt: str,
        model: str,
        **kwargs
    ) -> str:
        """生成文本"""
        pass

    @abstractmethod
    async def list_models(self) -> List[Dict[str, Any]]:
        """列出可用模型"""
        pass

    @abstractmethod
    async def pull_model(self, model: str) -> bool:
        """拉取模型"""
        pass

class SystemServiceInterface(ABC):
    """系统服务接口合约"""

    @abstractmethod
    def install_service(self) -> bool:
        """安装系统服务"""
        pass

    @abstractmethod
    def start_service(self) -> bool:
        """启动服务"""
        pass

    @abstractmethod
    def stop_service(self) -> bool:
        """停止服务"""
        pass

    @abstractmethod
    def get_service_status(self) -> Dict[str, Any]:
        """获取服务状态"""
        pass

class IntegrationContract:
    """集成合约"""

    # 网络要求
    NETWORK_REQUIREMENTS = {
        "timeout_seconds": 30,
        "max_retries": 3,
        "backoff_factor": 2.0,
        "connection_pool_size": 5
    }

    # 安全要求
    SECURITY_REQUIREMENTS = {
        "ssl_verification": True,
        "api_key_protection": True,
        "data_encryption": False,  # 本地服务，可选
        "audit_logging": True
    }
```

## Testing Contracts

### Test Interface

```python
class TestableInterface(ABC):
    """可测试接口"""

    @abstractmethod
    def get_test_data(self) -> Dict[str, Any]:
        """获取测试数据"""
        pass

    @abstractmethod
    def set_test_mode(self, enabled: bool) -> None:
        """设置测试模式"""
        pass

class MockAIService(AIServiceInterface, TestableInterface):
    """AI服务模拟实现"""

    def __init__(self):
        self.test_mode = False
        self.mock_responses = {}

    def get_test_data(self) -> Dict[str, Any]:
        return {
            "available_models": ["gemma3:270m", "qwen2.5:3b", "phi3:mini"],
            "mock_questions": [
                "你今天学到了什么新知识？",
                "今天的经历给你什么启发？"
            ]
        }

    def set_test_mode(self, enabled: bool) -> None:
        self.test_mode = enabled

class TestingContract:
    """测试合约"""

    # 测试覆盖率要求
    COVERAGE_REQUIREMENTS = {
        "unit_test_coverage": 80.0,  # 单元测试覆盖率(%)
        "integration_test_coverage": 60.0,  # 集成测试覆盖率(%)
        "api_test_coverage": 90.0  # API测试覆盖率(%)
    }

    # 性能测试要求
    PERFORMANCE_TEST_REQUIREMENTS = {
        "command_response_time": 1.0,  # 命令响应时间(秒)
        "ai_generation_time": 30.0,  # AI生成时间(秒)
        "memory_usage_limit": 100 * 1024 * 1024  # 内存使用限制(字节)
    }
```

## Error Handling Contracts

### Error Interface

```python
class DaybookException(Exception):
    """基础异常类"""

    def __init__(self, message: str, error_code: str = None, details: Dict[str, Any] = None):
        super().__init__(message)
        self.error_code = error_code
        self.details = details or {}
        self.timestamp = datetime.now()

class ConfigurationException(DaybookException):
    """配置异常"""
    pass

class AIServiceException(DaybookException):
    """AI服务异常"""
    pass

class StorageException(DaybookException):
    """存储异常"""
    pass

class ValidationException(DaybookException):
    """验证异常"""
    pass

class ErrorHandlingContract:
    """错误处理合约"""

    # 错误分类
    ERROR_CATEGORIES = {
        "user_input": "用户输入错误",
        "configuration": "配置错误",
        "ai_service": "AI服务错误",
        "storage": "存储错误",
        "system": "系统错误"
    }

    # 错误处理策略
    ERROR_HANDLING_STRATEGIES = {
        "retry": ["ai_service", "system"],
        "user_intervention": ["user_input", "configuration"],
        "graceful_degradation": ["ai_service"],
        "fail_fast": ["storage", "system"]
    }

    # 日志记录要求
    LOGGING_REQUIREMENTS = {
        "log_level": "INFO",
        "error_logging": True,
        "audit_logging": True,
        "sensitive_data_masking": True
    }
```

## Conclusion

本API合约文档定义了Daybook CLI工具的完整接口规范，确保：

1. **模块化设计**: 各组件通过明确的接口交互
2. **可测试性**: 所有接口都有对应的测试合约
3. **可扩展性**: 接口设计支持未来功能扩展
4. **错误处理**: 完善的错误分类和处理机制
5. **性能要求**: 明确的性能指标和约束

这些合约为开发团队提供了清晰的实现指导，确保代码质量和系统稳定性。