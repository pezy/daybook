# Phase 0 Research: Daybook CLI - AI Journal for Programmers

**Created**: 2025-10-07
**Purpose**: Technical research and decision making for implementation planning

## Executive Summary

基于对CLI日记工具的全面技术研究，我们确定了**Python 3.11+ + Typer**作为核心技术栈，配合**Ollama本地AI服务**集成和**文件化存储**架构。该方案在开发效率、功能完整性和用户体验之间取得了最佳平衡，特别适合面向程序员用户的AI日记工具。

## Research Decisions

### 1. 编程语言和框架选择

**Decision**: Python 3.11+ + Typer CLI框架

**Rationale**:
- **开发效率优先**: Python生态成熟，开发速度快，适合快速迭代
- **AI集成便利**: 与AI服务集成的复杂度低，丰富的AI/ML库支持
- **足够性能**: 虽然启动时间比Rust慢，但对CLI工具仍在可接受范围
- **社区支持**: 丰富的CLI开发库和活跃的社区

**Alternatives Considered**:
- **Rust + clap**: 性能优秀但学习曲线陡峭，开发速度较慢
- **Go + cobra**: 性能好但AI集成复杂度较高
- **Node.js + commander**: 生态丰富但性能和内存占用一般

**Performance Mitigation**:
- 延迟加载AI相关模块
- 异步处理AI调用
- 缓存机制优化响应速度
- 使用PyInstaller优化打包

### 2. AI服务集成方案

**Decision**: Ollama + httpx + 自定义AI客户端

**Rationale**:
- **本地化隐私**: 用户数据完全本地处理，符合隐私要求
- **离线能力**: 不依赖外部网络，支持离线使用
- **模型灵活性**: 支持多种开源模型，用户可自由选择
- **成本控制**: 无API调用费用，适合个人工具

**Technical Implementation**:
```python
# 核心AI服务架构
class AIService:
    def __init__(self, model: str = "qwen2.5:3b"):
        self.client = httpx.AsyncClient()
        self.model = model
        self.health_checker = HealthChecker()

    async def generate_question(self, context: QuestionContext) -> str:
        """生成个性化问题"""
        pass

    async def health_check(self) -> bool:
        """服务健康检查"""
        pass
```

**Model Recommendations**:
- **Primary**: Qwen2.5:3b (平衡性能和资源消耗)
- **Upgrade**: Qwen2.5:7b (更好的问题质量)
- **Lightweight**: Phi3:mini (资源受限环境)

### 3. 数据存储架构

**Decision**: 文件化存储 + 混合格式

**Rationale**:
- **简单可靠**: 无需数据库服务，降低复杂度
- **用户友好**: Markdown格式便于用户直接查看和编辑
- **跨平台**: 文件操作在所有平台都得到良好支持
- **备份方便**: 用户可以直接复制文件夹进行备份

**File Structure**:
```
daybook/
├── config/
│   ├── settings.toml          # 用户配置
│   └── models.json            # AI模型配置
├── journals/
│   ├── 2025-10-07.md          # 每日日记文件
│   └── 2025-10-08.md
├── memory/
│   ├── long_term.json         # 长期记忆
│   └── tags.json              # 标签管理
├── cache/
│   ├── search_index.db        # 搜索索引
│   └── metadata.json          # 元数据缓存
├── backups/
│   └── auto/                  # 自动备份
└── logs/
    └── daybook.log            # 应用日志
```

**Data Formats**:
- **配置**: TOML (易读性和编辑性)
- **日记**: Markdown (人机友好)
- **记忆**: JSON (结构化数据)
- **索引**: SQLite (搜索性能)

### 4. 任务调度方案

**Decision**: APScheduler + 系统服务集成

**Rationale**:
- **功能完整**: APScheduler支持复杂的调度需求
- **跨平台**: 良好的跨平台兼容性
- **可靠性**: 持久化作业存储和错误恢复
- **集成性**: 易于与系统集成

**Implementation Strategy**:
```python
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.triggers.cron import CronTrigger

class ReminderService:
    def __init__(self, daybook_service: DaybookService):
        self.scheduler = BackgroundScheduler()
        self.daybook_service = daybook_service

    def schedule_daily_reminder(self, time: str, timezone: str):
        """安排每日提醒"""
        trigger = CronTrigger(
            hour=int(time.split(':')[0]),
            minute=int(time.split(':')[1]),
            timezone=timezone
        )
        self.scheduler.add_job(
            self.daybook_service.generate_daily_questions,
            trigger=trigger
        )
```

**System Integration**:
- **macOS**: launchd agent
- **Linux**: systemd service
- **Windows**: Windows service

### 5. 性能优化策略

**Decision**: 多层缓存 + 懒加载 + 异步处理

**Rationale**:
- **响应速度**: 减少用户等待时间
- **资源效率**: 合理使用内存和CPU
- **用户体验**: 保持界面响应性

**Optimization Techniques**:

1. **懒加载策略**:
```python
class LazyAIService:
    _instance = None

    def __getattr__(self, name):
        if self._instance is None:
            self._instance = AIService()
        return getattr(self._instance, name)
```

2. **缓存机制**:
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def get_memory_summary(date: str) -> str:
    """缓存记忆摘要"""
    pass
```

3. **异步处理**:
```python
async def log_with_ai_interaction(message: str):
    """异步日志记录和AI交互"""
    # 立即写入日志
    await write_log_async(message)

    # 异步生成AI响应
    asyncio.create_task(generate_ai_response(message))
```

### 6. 安全和隐私保护

**Decision**: 本地优先 + 选择性加密

**Rationale**:
- **隐私优先**: 用户数据完全本地处理
- **用户控制**: 用户决定是否启用加密
- **透明性**: 用户可以直接查看数据文件

**Security Measures**:

1. **数据加密** (可选):
```python
from cryptography.fernet import Fernet

class EncryptionService:
    def __init__(self, password: str):
        self.key = self._derive_key(password)
        self.cipher = Fernet(self.key)

    def encrypt_sensitive_data(self, data: str) -> str:
        return self.cipher.encrypt(data.encode()).decode()
```

2. **敏感信息检测**:
```python
def mask_sensitive_info(text: str) -> str:
    """遮蔽敏感信息"""
    import re
    # 遮蔽邮箱、电话、密码等
    text = re.sub(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
                  '[EMAIL]', text)
    return text
```

### 7. 测试策略

**Decision**: 测试驱动开发 + 多层测试

**Rationale**:
- **质量保证**: 确保功能正确性和稳定性
- **重构安全**: 支持代码重构和优化
- **用户信心**: 提供可靠的用户体验

**Testing Pyramid**:

1. **单元测试** (70%):
```python
import pytest
from daybook.core.journal import JournalService

class TestJournalService:
    def test_log_entry_creation(self):
        service = JournalService()
        entry = service.create_log("test message")
        assert entry.content == "test message"
        assert entry.timestamp is not None
```

2. **集成测试** (20%):
```python
class TestAIIntegration:
    async def test_question_generation(self):
        ai_service = AIService()
        context = QuestionContext(...)
        question = await ai_service.generate_question(context)
        assert len(question) > 0
        assert '?' in question
```

3. **端到端测试** (10%):
```python
class TestCLIWorkflow:
    def test_complete_journal_workflow(self):
        # 测试完整的用户工作流程
        pass
```

## Implementation Risks and Mitigations

### 1. AI服务依赖风险
**Risk**: Ollama服务不可用或性能问题
**Mitigation**:
- 实现多层降级机制
- 提供离线模式
- 本地缓存常见问题模板

### 2. 性能风险
**Risk**: 大量日记数据导致性能下降
**Mitigation**:
- 实现增量加载
- 建立搜索索引
- 定期数据归档

### 3. 跨平台兼容性风险
**Risk**: 不同操作系统行为差异
**Mitigation**:
- 使用跨平台库
- 充分的平台测试
- 平台特定的适配层

## Technology Stack Summary

### Core Dependencies
```toml
# pyproject.toml
[project]
dependencies = [
    # CLI Framework
    "typer>=0.9.0",        # 现代CLI框架，基于类型提示

    # Configuration
    "toml>=0.10.2",        # TOML配置文件解析
    "pydantic>=2.0.0",     # 数据验证和设置管理

    # AI Integration
    "httpx>=0.25.0",       # 异步HTTP客户端
    "ollama>=0.6.0",       # Ollama Python客户端

    # Task Scheduling
    "apscheduler>=3.10.0", # 任务调度框架

    # Data Processing
    "pytz>=2023.3",        # 时区处理
    "python-dateutil>=2.8.0", # 日期工具

    # Security (Optional)
    "cryptography>=41.0.0", # 数据加密
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",      # 测试框架
    "pytest-asyncio>=0.21.0", # 异步测试支持
    "pytest-cov>=4.1.0",  # 测试覆盖率
    "black>=23.0.0",       # 代码格式化
    "mypy>=1.5.0",         # 类型检查
    "pre-commit>=3.3.0",   # Git钩子
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project.scripts]
daybook = "daybook.cli.main:app"
```

### UV 项目管理
```bash
# 项目初始化
uv init daybook
cd daybook

# 添加依赖
uv add typer toml pydantic httpx ollama apscheduler pytz python-dateutil

# 添加开发依赖
uv add --dev pytest pytest-asyncio pytest-cov black mypy pre-commit

# 运行项目
uv run daybook --help

# 构建和发布
uv build
uv publish --token pypi-xxxx
```

### Project Structure
```
src/
├── daybook/
│   ├── __init__.py
│   ├── cli/                    # CLI命令处理
│   │   ├── __init__.py
│   │   ├── main.py            # 主命令入口
│   │   ├── init.py            # 初始化命令
│   │   ├── log.py             # 日志命令
│   │   ├── ask.py             # AI问答命令
│   │   └── mem.py             # 记忆管理命令
│   ├── core/                   # 核心业务逻辑
│   │   ├── __init__.py
│   │   ├── journal.py         # 日记管理
│   │   ├── memory.py          # 长期记忆
│   │   ├── ai_service.py      # AI服务
│   │   └── scheduler.py       # 任务调度
│   ├── models/                 # 数据模型
│   │   ├── __init__.py
│   │   ├── config.py          # 配置模型
│   │   ├── entry.py           # 日记条目
│   │   └── memory.py          # 记忆模型
│   ├── utils/                  # 工具函数
│   │   ├── __init__.py
│   │   ├── file_utils.py      # 文件操作
│   │   ├── time_utils.py      # 时间处理
│   │   └── security.py        # 安全工具
│   └── config/                 # 配置管理
│       ├── __init__.py
│       └── settings.py        # 设置管理
tests/
├── unit/                       # 单元测试
├── integration/                # 集成测试
└── e2e/                       # 端到端测试
docs/                          # 文档
├── api.md                      # API文档
├── user_guide.md              # 用户指南
└── development.md             # 开发指南
```

## Next Steps

1. **Phase 1**: 设计数据模型和API合约
2. **Phase 2**: 生成详细的任务分解
3. **Implementation**: 按优先级实现P1功能

## Conclusion

本研究为CLI日记工具提供了完整的技术方案，选择Python生态系统在满足功能需求的同时保持了开发的简洁性和效率。方案充分考虑了跨平台兼容性、性能优化和用户体验，为后续的开发实现奠定了坚实基础。

**推荐技术栈**: Python 3.11+ + Typer + Ollama + 文件存储
**项目管理**: uv 项目管理和依赖管理
**预期开发周期**: 4-6周 (MVP)
**团队规模**: 1-2名开发者