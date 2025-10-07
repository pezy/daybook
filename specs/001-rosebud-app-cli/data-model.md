# Data Model: Daybook CLI

**Created**: 2025-10-07
**Purpose**: Define data structures and relationships for the AI journal CLI tool

## Overview

本文档定义了Daybook CLI工具的核心数据模型，包括用户配置、日记条目、长期记忆和AI问题等实体。设计遵循简洁性、可扩展性和用户隐私原则。

## Core Entities

### 1. User Configuration (用户配置)

**File**: `config/settings.toml`

```python
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import time
from enum import Enum

class AIModel(str, Enum):
    QWEN2_3B = "qwen2.5:3b"
    QWEN2_7B = "qwen2.5:7b"
    PHI3_MINI = "phi3:mini"
    GEMMA2_2B = "gemma2:2b"

class UserSettings(BaseModel):
    """用户配置模型"""
    # 基本配置
    daybook_path: str = Field(..., description="日记存储路径")
    timezone: str = Field(default="UTC", description="时区设置")

    # AI配置
    ai_model: AIModel = Field(default=AIModel.QWEN2_3B, description="默认AI模型")
    ai_temperature: float = Field(default=0.7, ge=0.0, le=1.0, description="AI创造性参数")
    ai_max_tokens: int = Field(default=500, ge=100, le=2000, description="AI最大生成token数")

    # 提醒配置
    reminder_enabled: bool = Field(default=True, description="是否启用定时提醒")
    reminder_time: time = Field(default=time(23, 0), description="每日提醒时间")

    # 界面配置
    date_format: str = Field(default="%Y-%m-%d", description="日期格式")
    time_format: str = Field(default="%H:%M:%S", description="时间格式")

    # 高级配置
    auto_backup: bool = Field(default=True, description="自动备份")
    backup_retention_days: int = Field(default=30, ge=1, le=365, description="备份保留天数")
    encryption_enabled: bool = Field(default=False, description="启用数据加密")

class AIServiceConfig(BaseModel):
    """AI服务配置"""
    ollama_base_url: str = Field(default="http://localhost:11434", description="Ollama服务地址")
    timeout_seconds: int = Field(default=30, ge=5, le=300, description="请求超时时间")
    retry_attempts: int = Field(default=3, ge=1, le=10, description="重试次数")
    health_check_interval: int = Field(default=60, ge=10, le=600, description="健康检查间隔(秒)")
```

### 2. Journal Entry (日记条目)

**File**: `journals/YYYY-MM-DD.md`

```python
from datetime import datetime
from typing import Optional, List
from enum import Enum

class EntryType(str, Enum):
    LOG = "log"           # 用户日志
    AI_QUESTION = "ai_q"  # AI问题
    AI_ANSWER = "ai_a"    # AI回答
    MEMORY = "memory"     # 记忆添加
    SYSTEM = "system"     # 系统信息

class JournalEntry(BaseModel):
    """日记条目模型"""
    timestamp: datetime = Field(..., description="时间戳")
    entry_type: EntryType = Field(..., description="条目类型")
    content: str = Field(..., description="内容")

    # 可选字段
    tags: List[str] = Field(default_factory=list, description="标签列表")
    mood: Optional[str] = Field(None, description="情绪标签")
    metadata: dict = Field(default_factory=dict, description="扩展元数据")

class DailyJournal(BaseModel):
    """每日日记模型"""
    date: datetime = Field(..., description="日期")
    entries: List[JournalEntry] = Field(default_factory=list, description="条目列表")
    word_count: int = Field(default=0, description="当日字数统计")

    def add_entry(self, entry: JournalEntry):
        """添加条目"""
        self.entries.append(entry)
        self.word_count += len(entry.content.split())

    def get_entries_by_type(self, entry_type: EntryType) -> List[JournalEntry]:
        """按类型获取条目"""
        return [e for e in self.entries if e.entry_type == entry_type]

    def get_log_entries(self) -> List[JournalEntry]:
        """获取日志条目"""
        return self.get_entries_by_type(EntryType.LOG)

    def get_ai_conversations(self) -> List[tuple]:
        """获取AI对话"""
        conversations = []
        questions = self.get_entries_by_type(EntryType.AI_QUESTION)

        for question in questions:
            # 查找对应的回答
            answers = [
                e for e in self.entries
                if e.entry_type == EntryType.AI_ANSWER and
                e.timestamp > question.timestamp
            ]
            if answers:
                conversations.append((question, answers[0]))

        return conversations
```

### 3. Long-term Memory (长期记忆)

**File**: `memory/long_term.json`

```python
from datetime import datetime
from typing import List, Optional, Dict, Any

class MemoryCategory(str, Enum):
    PERSONAL = "personal"      # 个人信息
    GOALS = "goals"           # 目标
    PREFERENCES = "preferences" # 偏好
    RELATIONSHIPS = "relationships" # 人际关系
    SKILLS = "skills"         # 技能
    HABITS = "habits"         # 习惯
    VALUES = "values"         # 价值观
    OTHER = "other"           # 其他

class Memory(BaseModel):
    """长期记忆模型"""
    id: str = Field(..., description="唯一标识符")
    content: str = Field(..., description="记忆内容")
    category: MemoryCategory = Field(..., description="记忆类别")

    # 时间信息
    created_at: datetime = Field(default_factory=datetime.now, description="创建时间")
    updated_at: datetime = Field(default_factory=datetime.now, description="更新时间")
    last_reviewed: Optional[datetime] = Field(None, description="最后回顾时间")

    # 重要性标签
    importance: int = Field(default=3, ge=1, le=5, description="重要性(1-5)")
    tags: List[str] = Field(default_factory=list, description="标签")

    # 关联信息
    related_memories: List[str] = Field(default_factory=list, description="关联记忆ID")
    context: Dict[str, Any] = Field(default_factory=dict, description="上下文信息")

class MemoryBank(BaseModel):
    """记忆库模型"""
    memories: List[Memory] = Field(default_factory=list, description="记忆列表")
    categories: Dict[MemoryCategory, List[str]] = Field(default_factory=dict, description="分类索引")
    tags_index: Dict[str, List[str]] = Field(default_factory=dict, description="标签索引")

    def add_memory(self, memory: Memory):
        """添加记忆"""
        self.memories.append(memory)
        self._update_indexes(memory)

    def search_memories(self, query: str, category: Optional[MemoryCategory] = None) -> List[Memory]:
        """搜索记忆"""
        results = []
        query_lower = query.lower()

        for memory in self.memories:
            if category and memory.category != category:
                continue

            # 简单的文本匹配搜索
            if (query_lower in memory.content.lower() or
                any(query_lower in tag.lower() for tag in memory.tags)):
                results.append(memory)

        # 按重要性和更新时间排序
        results.sort(key=lambda m: (m.importance, m.updated_at), reverse=True)
        return results

    def get_memories_for_context(self, limit: int = 10) -> List[Memory]:
        """获取用于AI上下文的记忆"""
        # 优先获取重要性和最近更新的记忆
        relevant_memories = sorted(
            self.memories,
            key=lambda m: (m.importance, m.updated_at),
            reverse=True
        )
        return relevant_memories[:limit]

    def _update_indexes(self, memory: Memory):
        """更新索引"""
        # 更新分类索引
        if memory.category not in self.categories:
            self.categories[memory.category] = []
        self.categories[memory.category].append(memory.id)

        # 更新标签索引
        for tag in memory.tags:
            if tag not in self.tags_index:
                self.tags_index[tag] = []
            self.tags_index[tag].append(memory.id)
```

### 4. AI Question Generation (AI问题生成)

```python
from typing import List, Dict, Any
from datetime import datetime

class QuestionType(str, Enum):
    REFLECTION = "reflection"    # 反思类
    LEARNING = "learning"       # 学习类
    HEALTH = "health"          # 健康类
    RELATIONSHIP = "relationship" # 人际关系类
    GOAL = "goal"              # 目标类
    HABIT = "habit"            # 习惯类

class QuestionContext(BaseModel):
    """问题生成上下文"""
    current_date: datetime = Field(..., description="当前日期")
    today_entries: List[JournalEntry] = Field(..., description="今日条目")
    recent_memories: List[Memory] = Field(..., description="相关记忆")
    user_preferences: Dict[str, Any] = Field(default_factory=dict, description="用户偏好")

    # 统计信息
    today_word_count: int = Field(default=0, description="今日字数")
    streak_days: int = Field(default=0, description="连续记录天数")
    last_ai_interaction: Optional[datetime] = Field(None, description="上次AI交互")

class GeneratedQuestion(BaseModel):
    """生成的问题"""
    question: str = Field(..., description="问题内容")
    question_type: QuestionType = Field(..., description="问题类型")
    relevance_score: float = Field(..., ge=0.0, le=1.0, description="相关性评分")
    context_used: List[str] = Field(default_factory=list, description="使用的上下文")

    # 生成信息
    generated_at: datetime = Field(default_factory=datetime.now, description="生成时间")
    model_used: str = Field(..., description="使用的AI模型")
    generation_prompt: str = Field(..., description="生成提示词")

class QuestionTemplate(BaseModel):
    """问题模板"""
    template: str = Field(..., description="问题模板")
    question_type: QuestionType = Field(..., description="问题类型")
    required_context: List[str] = Field(default_factory=list, description="需要的上下文类型")
    variables: List[str] = Field(default_factory=list, description="模板变量")

# 预定义问题模板
QUESTION_TEMPLATES = [
    QuestionTemplate(
        template="基于你今天记录的{content_type}, 你有什么新的感悟吗?",
        question_type=QuestionType.REFLECTION,
        required_context=["today_entries"],
        variables=["content_type"]
    ),
    QuestionTemplate(
        template="你提到的{skill}学习进展如何？遇到了什么挑战?",
        question_type=QuestionType.LEARNING,
        required_context=["memories"],
        variables=["skill"]
    ),
    QuestionTemplate(
        template="今天的{activity}让你感觉如何？对身心健康有什么影响?",
        question_type=QuestionType.HEALTH,
        required_context=["today_entries", "memories"],
        variables=["activity"]
    ),
    QuestionTemplate(
        template="你与{person}的互动中，有什么值得思考的地方?",
        question_type=QuestionType.RELATIONSHIP,
        required_context=["today_entries", "memories"],
        variables=["person"]
    ),
    QuestionTemplate(
        template="关于{goal}, 你今天取得了什么进展?",
        question_type=QuestionType.GOAL,
        required_context=["memories", "today_entries"],
        variables=["goal"]
    )
]
```

### 5. Search Index (搜索索引)

```python
import sqlite3
from typing import List, Dict, Any, Optional
from datetime import datetime

class SearchIndex(BaseModel):
    """搜索索引模型"""
    db_path: str = Field(..., description="索引数据库路径")

    def __init__(self, **data):
        super().__init__(**data)
        self._init_database()

    def _init_database(self):
        """初始化数据库"""
        with sqlite3.connect(self.db_path) as conn:
            conn.execute("""
                CREATE TABLE IF NOT EXISTS entries (
                    id TEXT PRIMARY KEY,
                    date TEXT NOT NULL,
                    content TEXT NOT NULL,
                    entry_type TEXT NOT NULL,
                    tags TEXT,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """)

            conn.execute("""
                CREATE VIRTUAL TABLE IF NOT EXISTS entries_fts
                USING fts5(content, tags, content='entries', content_rowid='rowid')
            """)

            conn.execute("""
                CREATE INDEX IF NOT EXISTS idx_entries_date ON entries(date)
            """)

    def index_entry(self, entry_id: str, date: str, content: str,
                   entry_type: str, tags: List[str]):
        """索引条目"""
        with sqlite3.connect(self.db_path) as conn:
            # 插入主表
            conn.execute(
                "INSERT OR REPLACE INTO entries (id, date, content, entry_type, tags) VALUES (?, ?, ?, ?, ?)",
                (entry_id, date, content, entry_type, ",".join(tags))
            )

            # 更新全文搜索表
            conn.execute(
                "INSERT OR REPLACE INTO entries_fts (rowid, content, tags) VALUES ((SELECT rowid FROM entries WHERE id = ?), ?, ?)",
                (entry_id, content, ",".join(tags))
            )

    def search(self, query: str, limit: int = 50) -> List[Dict[str, Any]]:
        """搜索条目"""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("""
                SELECT e.id, e.date, e.content, e.entry_type, e.tags,
                       rank
                FROM entries_fts fts
                JOIN entries e ON e.id = fts.rowid
                WHERE entries_fts MATCH ?
                ORDER BY rank
                LIMIT ?
            """, (query, limit))

            return [
                {
                    "id": row[0],
                    "date": row[1],
                    "content": row[2],
                    "entry_type": row[3],
                    "tags": row[4].split(",") if row[4] else [],
                    "rank": row[5]
                }
                for row in cursor.fetchall()
            ]
```

## Data Flow and Relationships

### Data Flow Diagram

```
用户输入
    ↓
CLI命令处理
    ↓
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  配置管理    │    │  日记管理    │    │  记忆管理    │
│ (settings)  │    │ (journals)  │    │ (memory)    │
└─────────────┘    └─────────────┘    └─────────────┘
    ↓                    ↓                    ↓
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  AI服务     │    │  搜索索引    │    │  定时任务    │
│ (ollama)    │    │ (sqlite)    │    │ (scheduler) │
└─────────────┘    └─────────────┘    └─────────────┘
```

### Entity Relationships

1. **UserSettings** → 配置所有其他组件的行为
2. **DailyJournal** → 包含多个JournalEntry
3. **JournalEntry** → 可以关联到Memory
4. **Memory** → 可以被多个JournalEntry引用
5. **QuestionContext** → 整合DailyJournal和Memory用于AI生成
6. **SearchIndex** → 索引所有JournalEntry内容

## File Format Specifications

### Configuration File (settings.toml)

```toml
[daybook]
path = "/Users/username/Documents/daybook"
timezone = "Asia/Shanghai"
date_format = "%Y-%m-%d"
time_format = "%H:%M:%S"

[ai]
model = "qwen2.5:3b"
temperature = 0.7
max_tokens = 500
base_url = "http://localhost:11434"
timeout_seconds = 30
retry_attempts = 3

[reminder]
enabled = true
time = "23:00:00"

[backup]
enabled = true
retention_days = 30
auto_backup_interval = 24  # hours

[security]
encryption_enabled = false
```

### Journal File (2025-10-07.md)

```markdown
# 2025-10-07

## 09:15:30
今天开始学习Rust编程语言，感觉很有意思。

## 14:30:22
完成了第一个Rust项目，虽然遇到了很多编译错误，但最终还是解决了。

## 20:15:45
### AI Question
基于你今天记录的学习经历，你觉得Rust与其他编程语言相比有什么优势？

## 20:16:12
### AI Answer
根据你今天的学习体验，Rust的主要优势包括：1. 内存安全保证，2. 高性能，3. 强大的类型系统。这些特点虽然增加了学习曲线，但长期来看会提高代码质量。

## 23:00:00
### Daily Reflection
今天的学习收获很大，虽然有些挑战，但克服困难的感觉很好。
```

### Memory File (long_term.json)

```json
{
  "version": "1.0",
  "created_at": "2025-10-07T00:00:00Z",
  "updated_at": "2025-10-07T15:30:00Z",
  "memories": [
    {
      "id": "mem_001",
      "content": "我是一名软件工程师，主要使用Python和JavaScript，正在学习Rust",
      "category": "personal",
      "importance": 4,
      "tags": ["编程", "职业", "学习"],
      "created_at": "2025-10-01T10:00:00Z",
      "updated_at": "2025-10-07T15:30:00Z"
    },
    {
      "id": "mem_002",
      "content": "目标是掌握系统编程，能够开发高性能的服务器应用",
      "category": "goals",
      "importance": 5,
      "tags": ["目标", "系统编程", "服务器"],
      "created_at": "2025-10-02T09:00:00Z",
      "updated_at": "2025-10-02T09:00:00Z"
    }
  ]
}
```

## Validation Rules

### Data Validation

1. **Configuration Validation**:
   - daybook_path 必须是有效且可写的路径
   - timezone 必须是有效的时区标识符
   - ai_model 必须是支持的模型名称
   - reminder_time 必须是有效的时间格式

2. **Journal Entry Validation**:
   - timestamp 必须是有效的datetime
   - content 不能为空
   - entry_type 必须是预定义的类型
   - 标签必须符合命名规则

3. **Memory Validation**:
   - id 必须唯一
   - content 长度限制 (1-1000字符)
   - category 必须是预定义类别
   - importance 必须在1-5范围内

### Business Logic Validation

1. **File Operations**:
   - 日记文件按日期创建，不能跨日期写入
   - 配置文件必须先于其他操作创建
   - 备份文件保留期限检查

2. **AI Integration**:
   - AI服务健康检查
   - 问题生成数量限制 (1个主动交互，3个定时提醒)
   - 内容长度和复杂度限制

3. **Privacy and Security**:
   - 敏感信息检测和遮蔽
   - 本地数据访问权限验证
   - 加密数据完整性检查

## Performance Considerations

### Indexing Strategy

1. **Memory Index**:
   - 按重要性和时间排序的记忆索引
   - 标签和分类的快速查找索引
   - AI上下文相关的记忆缓存

2. **Search Index**:
   - SQLite FTS5全文搜索
   - 按日期和类型的复合索引
   - 增量索引更新

### Caching Strategy

1. **Configuration Cache**:
   - 启动时加载配置到内存
   - 配置变更时热重载

2. **Recent Entries Cache**:
   - 最近7天的日记条目缓存
   - AI交互结果缓存

3. **AI Response Cache**:
   - 相似上下文的AI响应缓存
   - 问题模板结果缓存

### Data Size Management

1. **Journal Files**:
   - 每日文件自动分割
   - 大文件自动归档机制
   - 历史数据压缩存储

2. **Memory Storage**:
   - 记忆数量限制 (建议<1000条)
   - 自动重要性评估和清理
   - 定期记忆回顾和更新

## Migration and Versioning

### Schema Versioning

```python
class DataVersion(BaseModel):
    """数据版本管理"""
    version: str = Field(..., description="版本号")
    migration_date: datetime = Field(default_factory=datetime.now)
    description: str = Field(..., description="版本描述")

# 版本历史
DATA_VERSIONS = [
    DataVersion(version="1.0", description="初始版本"),
    # 未来版本...
]
```

### Migration Strategy

1. **Backward Compatibility**:
   - 支持旧版本配置文件读取
   - 数据格式自动转换
   - 用户友好的迁移提示

2. **Forward Compatibility**:
   - 预留扩展字段
   - 可选的配置项设计
   - 渐进式功能升级

## Conclusion

本数据模型设计遵循以下原则：

1. **简洁性**: 最小化复杂度，易于理解和维护
2. **可扩展性**: 支持未来功能扩展
3. **性能**: 考虑大量数据时的性能表现
4. **隐私**: 用户数据本地存储，保护隐私
5. **用户友好**: 支持用户直接查看和编辑数据文件

该模型为Daybook CLI工具提供了坚实的数据基础，支持所有核心功能的实现，同时为未来的功能扩展保留了灵活性。