# CLI日记工具AI服务集成方案

## 概述

本文档为CLI日记工具提供完整的AI服务集成方案，重点关注与Ollama本地AI服务的集成，确保工具具备智能问答、个性化问题生成和离线降级能力。

## 1. Ollama本地AI服务集成

### 1.1 Ollama Python客户端集成

**推荐使用官方Ollama Python库 (v0.6.0+)**

```python
# requirements.txt
ollama>=0.6.0
httpx>=0.27.0
pydantic>=2.0.0
```

**核心集成代码：**

```python
import ollama
from typing import Optional, List, Dict, Any
import asyncio
from dataclasses import dataclass
import logging

@dataclass
class AIConfig:
    host: str = "localhost"
    port: int = 11434
    model: str = "qwen:7b"
    timeout: int = 30
    max_retries: int = 3

class OllamaClient:
    def __init__(self, config: AIConfig):
        self.config = config
        self.client = ollama.Client(
            host=f"{config.host}:{config.port}",
            timeout=config.timeout
        )
        self.logger = logging.getLogger(__name__)

    async def generate_response(
        self,
        prompt: str,
        context: Optional[str] = None,
        memories: Optional[List[str]] = None
    ) -> str:
        """生成AI响应，支持上下文和记忆集成"""
        try:
            full_prompt = self._build_prompt(prompt, context, memories)

            response = await asyncio.to_thread(
                self.client.chat,
                model=self.config.model,
                messages=[{"role": "user", "content": full_prompt}]
            )

            return response['message']['content'].strip()

        except Exception as e:
            self.logger.error(f"AI服务调用失败: {e}")
            raise AIServiceError(f"无法连接到AI服务: {e}")

    def _build_prompt(self, prompt: str, context: Optional[str], memories: Optional[List[str]]) -> str:
        """构建完整的提示词，集成上下文和记忆"""
        prompt_parts = []

        # 系统角色定义
        prompt_parts.append("""
你是一个专业的日记助手，专门帮助程序员用户进行个人成长反思。
你的任务是基于用户的日记记录和个人记忆，生成有意义的个性化问题。
请用中文回答，问题应该简洁、有深度且与用户的具体情况相关。
""")

        # 添加长期记忆
        if memories:
            prompt_parts.append("\n用户的个人记忆:")
            for i, memory in enumerate(memories, 1):
                prompt_parts.append(f"{i}. {memory}")

        # 添加上下文（今日记录）
        if context:
            prompt_parts.append(f"\n用户的今日记录:\n{context}")

        # 主要任务
        prompt_parts.append(f"\n{prompt}")

        return "\n".join(prompt_parts)

    async def health_check(self) -> bool:
        """检查AI服务健康状态"""
        try:
            await asyncio.to_thread(self.client.list)
            return True
        except Exception:
            return False

class AIServiceError(Exception):
    """AI服务异常"""
    pass
```

### 1.2 错误处理和重试机制

```python
import time
from functools import wraps
from typing import Callable, Any

def retry_with_backoff(max_retries: int = 3, base_delay: float = 1.0):
    """指数退避重试装饰器"""
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        async def wrapper(*args, **kwargs) -> Any:
            for attempt in range(max_retries):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise

                    delay = base_delay * (2 ** attempt)
                    logging.warning(f"第{attempt + 1}次重试，等待{delay}秒: {e}")
                    await asyncio.sleep(delay)

            return None
        return wrapper
    return decorator

class RobustOllamaClient(OllamaClient):
    @retry_with_backoff(max_retries=3)
    async def generate_response_with_retry(self, prompt: str, context: Optional[str] = None) -> str:
        return await self.generate_response(prompt, context)
```

## 2. HTTP客户端库比较分析

### 2.1 库选择对比

| 特性 | requests | httpx | aiohttp |
|------|----------|-------|---------|
| **同步支持** | ✅ 优秀 | ✅ 良好 | ❌ 不支持 |
| **异步支持** | ❌ 不支持 | ✅ 优秀 | ✅ 优秀 |
| **HTTP/2支持** | ❌ 不支持 | ✅ 支持 | ✅ 支持 |
| **连接池** | ✅ 内置 | ✅ 更高级 | ✅ 高级 |
| **错误处理** | ✅ 简单 | ✅ 完善 | ✅ 完善 |
| **类型提示** | ❌ 基础 | ✅ 完整 | ✅ 完整 |
| **学习曲线** | ✅ 简单 | ✅ 中等 | ❌ 复杂 |

### 2.2 推荐方案

**主选择：httpx**
- 同时支持同步和异步调用
- 与requests API兼容
- 内置重试和超时机制
- 支持HTTP/2

**备用方案：requests（仅同步场景）**
- 最简单易用
- 社区支持最好
- 资源占用少

```python
# httpx示例实现
import httpx
from typing import Optional

class HTTPXClient:
    def __init__(self, base_url: str, timeout: int = 30):
        self.client = httpx.AsyncClient(
            base_url=base_url,
            timeout=timeout,
            limits=httpx.Limits(max_keepalive_connections=5)
        )

    async def post(self, endpoint: str, data: dict) -> dict:
        try:
            response = await self.client.post(endpoint, json=data)
            response.raise_for_status()
            return response.json()
        except httpx.HTTPError as e:
            raise AIServiceError(f"HTTP请求失败: {e}")

    async def close(self):
        await self.client.aclose()
```

## 3. AI提示工程最佳实践

### 3.1 个性化问题生成策略

```python
class PromptEngine:
    def __init__(self):
        self.question_templates = {
            "technical": [
                "根据你的代码记录，今天你遇到了什么技术挑战？",
                "从今天的编程工作中，你学到了什么新技能？",
                "你认为今天的技术决策有哪些可以改进的地方？"
            ],
            "personal": [
                "基于你的记录，今天你的情绪状态如何？",
                "今天的工作让你有什么感悟？",
                "你今天在个人成长方面有什么收获？"
            ],
            "goal": [
                "根据你的记忆，今天你为长期目标做了什么？",
                "今天的进展如何帮助你实现个人目标？",
                "你明天的计划如何与长期目标对齐？"
            ]
        }

    def generate_personalized_prompt(
        self,
        user_context: str,
        memories: List[str],
        question_count: int = 1
    ) -> str:
        """生成个性化提示词"""

        # 分析用户内容类型
        content_type = self._analyze_content_type(user_context)

        # 选择问题模板
        templates = self.question_templates.get(content_type, self.question_templates["personal"])

        # 构建完整提示
        prompt = f"""
基于以下信息，为用户生成{question_count}个有意义的反思问题：

用户今日记录：
{user_context}

用户个人记忆：
{chr(10).join(f"- {memory}" for memory in memories)}

要求：
1. 问题应该具体且与用户记录相关
2. 问题应该能促进深度思考
3. 问题应该简洁明了
4. 用中文提问
5. 基于用户的技术背景和个人情况

请生成{question_count}个问题，每行一个：
"""

        return prompt

    def _analyze_content_type(self, content: str) -> str:
        """分析内容类型"""
        technical_keywords = ["代码", "编程", "bug", "开发", "测试", "部署", "API"]
        personal_keywords = ["感受", "情绪", "想法", "思考", "生活"]
        goal_keywords = ["目标", "计划", "学习", "成长", "进步"]

        content_lower = content.lower()

        if any(keyword in content_lower for keyword in technical_keywords):
            return "technical"
        elif any(keyword in content_lower for keyword in goal_keywords):
            return "goal"
        else:
            return "personal"
```

### 3.2 上下文构建和记忆集成

```python
class ContextBuilder:
    def __init__(self, max_context_length: int = 2000):
        self.max_context_length = max_context_length

    def build_context(
        self,
        today_entries: List[str],
        recent_entries: List[str],
        memories: List[str]
    ) -> Dict[str, Any]:
        """构建完整的对话上下文"""

        # 处理今日记录
        today_context = self._process_entries(today_entries, limit=500)

        # 处理近期记录
        recent_context = self._process_entries(recent_entries, limit=800)

        # 处理长期记忆
        memory_context = self._select_relevant_memories(memories, today_context, limit=300)

        return {
            "today": today_context,
            "recent": recent_context,
            "memories": memory_context,
            "total_length": len(today_context) + len(recent_context) + len(memory_context)
        }

    def _process_entries(self, entries: List[str], limit: int) -> str:
        """处理和限制条目长度"""
        if not entries:
            return ""

        combined = "\n".join(entries)
        if len(combined) <= limit:
            return combined

        # 截断最相关的部分
        return combined[-limit:]

    def _select_relevant_memories(self, memories: List[str], context: str, limit: int) -> List[str]:
        """选择与上下文最相关的记忆"""
        if not memories:
            return []

        # 简单的关键词匹配（实际应用中可以用更复杂的语义匹配）
        context_words = set(context.lower().split())

        scored_memories = []
        for memory in memories:
            memory_words = set(memory.lower().split())
            relevance = len(context_words & memory_words)
            scored_memories.append((relevance, memory))

        # 按相关性排序并选择
        scored_memories.sort(key=lambda x: x[0], reverse=True)

        selected = []
        total_length = 0
        for _, memory in scored_memories:
            if total_length + len(memory) <= limit:
                selected.append(memory)
                total_length += len(memory)
            else:
                break

        return selected
```

## 4. 离线模式和服务降级策略

### 4.1 服务状态检测

```python
class ServiceHealthMonitor:
    def __init__(self, ai_client: OllamaClient):
        self.ai_client = ai_client
        self.last_check = None
        self.is_healthy = False
        self.check_interval = 60  # 秒

    async def check_service_health(self) -> bool:
        """检查服务健康状态"""
        try:
            self.is_healthy = await self.ai_client.health_check()
            self.last_check = time.time()
            return self.is_healthy
        except Exception as e:
            logging.warning(f"健康检查失败: {e}")
            self.is_healthy = False
            return False

    def should_check(self) -> bool:
        """判断是否需要重新检查"""
        if self.last_check is None:
            return True

        return time.time() - self.last_check > self.check_interval

    async def ensure_healthy(self) -> bool:
        """确保服务健康，必要时进行检查"""
        if self.should_check():
            await self.check_service_health()

        return self.is_healthy
```

### 4.2 降级策略实现

```python
class FallbackQuestionGenerator:
    def __init__(self):
        self.default_questions = [
            "今天你学到了什么新东西？",
            "今天的工作有什么值得记录的成就？",
            "今天你遇到了什么挑战，是如何解决的？",
            "今天有什么让你感到开心或满足的事情？",
            "明天你计划做什么来继续进步？"
        ]

    def generate_fallback_questions(self, count: int = 1) -> List[str]:
        """生成备用问题"""
        import random
        return random.sample(self.default_questions, min(count, len(self.default_questions)))

    def generate_context_based_questions(self, entries: List[str], count: int = 1) -> List[str]:
        """基于内容生成简单的备用问题"""
        questions = []

        if entries:
            # 基于关键词的简单问题生成
            combined_text = " ".join(entries).lower()

            if any(word in combined_text for word in ["bug", "错误", "问题"]):
                questions.append("今天你遇到了什么技术问题？")

            if any(word in combined_text for word in ["学习", "新", "学会"]):
                questions.append("今天你学到了什么新知识？")

            if any(word in combined_text for word in ["完成", "解决", "实现"]):
                questions.append("今天你完成了什么重要的工作？")

        # 如果没有生成足够的问题，使用默认问题
        while len(questions) < count:
            questions.extend(self.generate_fallback_questions(count - len(questions)))

        return questions[:count]

class AIQuestionService:
    def __init__(self, ai_client: OllamaClient):
        self.ai_client = ai_client
        self.health_monitor = ServiceHealthMonitor(ai_client)
        self.fallback_generator = FallbackQuestionGenerator()
        self.prompt_engine = PromptEngine()
        self.context_builder = ContextBuilder()

    async def generate_questions(
        self,
        today_entries: List[str],
        recent_entries: List[str],
        memories: List[str],
        question_count: int = 1
    ) -> List[str]:
        """生成问题，支持服务降级"""

        # 检查服务健康状态
        is_healthy = await self.health_monitor.ensure_healthy()

        if is_healthy:
            try:
                # 使用AI生成问题
                context = self.context_builder.build_context(
                    today_entries, recent_entries, memories
                )

                prompt = self.prompt_engine.generate_personalized_prompt(
                    context["today"] + "\n" + context["recent"],
                    memories,
                    question_count
                )

                response = await self.ai_client.generate_response_with_retry(prompt)

                # 解析AI响应中的问题
                questions = self._parse_questions(response)

                if len(questions) >= question_count:
                    return questions[:question_count]

            except Exception as e:
                logging.error(f"AI问题生成失败，使用备用方案: {e}")

        # 降级到本地生成
        return self.fallback_generator.generate_context_based_questions(
            today_entries, question_count
        )

    def _parse_questions(self, response: str) -> List[str]:
        """解析AI响应中的问题列表"""
        questions = []
        for line in response.strip().split('\n'):
            line = line.strip()
            if line and not line.startswith('#'):
                # 移除可能的编号前缀
                question = re.sub(r'^\d+[\.\)]\s*', '', line).strip()
                if question:
                    questions.append(question)

        return questions
```

## 5. AI模型适用性分析

### 5.1 模型推荐

| 模型 | 参数量 | 中文支持 | 速度 | 内存需求 | 适用场景 |
|------|--------|----------|------|----------|----------|
| **Qwen2.5:7b** | 7B | 优秀 | 中等 | 8GB+ | 主要推荐，平衡性能和质量 |
| **Qwen2.5:3b** | 3B | 优秀 | 快 | 4GB+ | 轻量级选择 |
| **Phi3:mini** | 3.8B | 良好 | 快 | 4GB+ | 备用选择 |
| **Llama3.1:8b** | 8B | 良好 | 中等 | 8GB+ | 英文为主的场景 |

### 5.2 模型配置建议

```python
# 推荐的模型配置
MODEL_CONFIGS = {
    "high_performance": {
        "model": "qwen2.5:7b",
        "temperature": 0.7,
        "top_p": 0.9,
        "max_tokens": 500
    },
    "balanced": {
        "model": "qwen2.5:3b",
        "temperature": 0.8,
        "top_p": 0.95,
        "max_tokens": 300
    },
    "lightweight": {
        "model": "phi3:mini",
        "temperature": 0.9,
        "top_p": 0.9,
        "max_tokens": 200
    }
}

def get_optimal_model_config(available_memory_gb: int) -> dict:
    """根据可用内存选择最优模型配置"""
    if available_memory_gb >= 8:
        return MODEL_CONFIGS["high_performance"]
    elif available_memory_gb >= 4:
        return MODEL_CONFIGS["balanced"]
    else:
        return MODEL_CONFIGS["lightweight"]
```

## 6. 集成架构建议

### 6.1 系统架构图

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   CLI Interface │◄──►│  Service Layer   │◄──►│  Storage Layer  │
│                 │    │                  │    │                 │
│ - Commands      │    │ - AI Service     │    │ - Config Files  │
│ - Validation    │    │ - Prompt Engine  │    │ - Journal Files │
│ - Formatting    │    │ - Health Monitor │    │ - Memory Store  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌──────────────────┐
                       │  Ollama Service  │
                       │                  │
                       │ - Model Loading  │
                       │ - Response Gen   │
                       │ - Health Check   │
                       └──────────────────┘
```

### 6.2 核心组件实现

```python
# 主要服务类
class DaybookAIService:
    def __init__(self, config_path: str):
        self.config = self._load_config(config_path)
        self.ai_client = OllamaClient(self.config.ai)
        self.question_service = AIQuestionService(self.ai_client)
        self.storage = StorageService(self.config.storage)

    async def initialize(self):
        """初始化服务"""
        await self.storage.ensure_directories()

        # 检查AI服务可用性
        if not await self.ai_client.health_check():
            logging.warning("AI服务不可用，将使用离线模式")

    async def generate_daily_questions(
        self,
        user_id: str,
        question_count: int = 1
    ) -> List[str]:
        """生成每日问题"""

        # 加载用户数据
        today_entries = await self.storage.get_today_entries(user_id)
        recent_entries = await self.storage.get_recent_entries(user_id, days=7)
        memories = await self.storage.get_user_memories(user_id)

        # 生成问题
        questions = await self.question_service.generate_questions(
            today_entries=today_entries,
            recent_entries=recent_entries,
            memories=memories,
            question_count=question_count
        )

        return questions

    async def save_interaction(
        self,
        user_id: str,
        question: str,
        answer: str
    ):
        """保存交互记录"""
        timestamp = datetime.now().strftime("%H:%M:%S")
        entry = f"{timestamp} Q: {question}\n{timestamp} A: {answer}"

        await self.storage.append_entry(user_id, entry)
```

## 7. 使用示例

### 7.1 基本使用

```python
import asyncio
from daybook_ai import DaybookAIService

async def main():
    # 初始化服务
    service = DaybookAIService("config.json")
    await service.initialize()

    # 生成每日问题
    questions = await service.generate_daily_questions("user123", question_count=1)

    for question in questions:
        print(f"AI问题: {question}")

        # 获取用户回答
        answer = input("你的回答: ")

        # 保存交互
        await service.save_interaction("user123", question, answer)

if __name__ == "__main__":
    asyncio.run(main())
```

### 7.2 配置文件示例

```json
{
  "ai": {
    "host": "localhost",
    "port": 11434,
    "model": "qwen2.5:7b",
    "timeout": 30,
    "max_retries": 3
  },
  "storage": {
    "base_path": "~/.daybook",
    "timezone": "Asia/Shanghai",
    "daily_reminder": "23:00"
  },
  "features": {
    "enable_ai": true,
    "fallback_on_error": true,
    "cache_responses": true
  }
}
```

## 8. 总结

本集成方案提供了完整的AI服务集成能力，具有以下特点：

1. **可靠的服务集成**：通过Ollama Python客户端实现稳定的AI服务连接
2. **优雅的降级策略**：当AI服务不可用时，自动切换到本地问题生成
3. **个性化体验**：基于用户记录和记忆生成针对性问题
4. **高性能架构**：支持异步操作和连接池管理
5. **灵活的模型选择**：根据硬件条件选择合适的AI模型
6. **完善的错误处理**：包含重试机制和异常恢复

该方案确保CLI日记工具在各种环境下都能提供良好的用户体验，既能利用AI的智能能力，又能在资源受限时保持基本功能。