# Feature Specification: Daybook CLI - AI Journal for Programmers

**Feature Branch**: `001-rosebud-app-cli`
**Created**: 2025-10-07
**Status**: Draft
**Input**: User description: "构建一个 CLI 工具，目标是融合 journaling, habit-building, 以及 emotional support。采用针对 programmer 的方式。保持简单，专注于呈现 companion agent 的能力。"

## Clarifications

### Session 2025-10-07

- Q: 日志记录时应该在日记文件中使用什么时间戳格式？ → A: 使用 24 小时格式: `23:43:00 用户输入内容`
- Q: 智能问答互动功能中，AI应该问几个问题？ → A: 用户主动交互时问1个问题，定时任务时问3个问题
- Q: 产品的官方名称应该是什么？ → A: Daybook CLI (避免与任何其他产品关联)

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - 初始化配置 (Priority: P1)

作为程序员用户，我希望能够通过简单的初始化命令设置我的个人日记环境，包括存储位置、时区、AI模型和每日提醒时间，这样我就能够开始使用这个工具来记录我的成长历程。

**Why this priority**: 初始化是所有其他功能的基础，没有正确的配置，用户无法使用任何核心功能。这是用户首次接触产品的关键体验。

**Independent Test**: 可以通过运行初始化命令并验证配置文件是否正确创建来独立测试。用户能够成功设置 daybook 文件夹路径、时区、模型选择和提醒时间，即表示功能完成。

**Acceptance Scenarios**:

1. **Given** 用户首次安装工具，**When** 运行初始化命令，**Then** 系统应该引导用户完成配置流程并创建配置文件
2. **Given** 用户指定日记存储文件夹路径，**When** 初始化完成，**Then** 系统应该在该路径下创建配置文件并验证文件夹权限
3. **Given** 用户选择时区和AI模型，**When** 配置保存，**Then** 系统应该验证设置的有效性并提供确认反馈

---

### User Story 2 - 实时日志记录 (Priority: P1)

作为程序员用户，我希望能够随时随地通过命令行快速记录想法、代码片段或工作进展，这样我就能够捕获重要的灵感和工作成果。

**Why this priority**: 日志记录是工具的核心功能，为用户提供便捷的记录方式是提升使用频率的关键。

**Independent Test**: 可以通过运行日志命令并验证内容是否正确写入当天的日记文件来独立测试。用户能够输入内容并看到内容被正确保存，即表示功能完成。

**Acceptance Scenarios**:

1. **Given** 用户已初始化工具，**When** 运行日志记录命令并输入内容，**Then** 系统应该以 `HH:MM:SS 用户内容` 格式将此内容添加到当天的日记文件中
2. **Given** 用户输入多行内容，**When** 运行日志记录命令，**Then** 系统应该为每条记录添加时间戳并保持格式正确写入日记文件
3. **Given** 当天的日记文件不存在，**When** 用户首次使用日志记录命令，**Then** 系统应该自动创建当天的日记文件并按时间戳格式记录内容

---

### User Story 3 - 智能问答互动 (Priority: P1)

作为程序员用户，我希望能够基于我今天的记录和长期记忆与AI助手进行有意义的对话，获得关于学习、健康和人际关系的洞察和建议。

**Why this priority**: 智能问答是产品的核心差异化功能，通过个性化的AI互动帮助用户实现个人成长。

**Independent Test**: 可以通过运行智能问答命令并验证AI是否基于用户的历史记录提出相关问题来独立测试。AI能够根据今日内容和记忆生成1个针对性问题，即表示功能完成。

**Acceptance Scenarios**:

1. **Given** 用户今天有记录内容，**When** 运行智能问答命令，**Then** AI应该基于今日内容和用户记忆生成1个相关问题
2. **Given** 用户设置了长期记忆，**When** AI提问，**Then** 问题应该体现出对用户个人情况的理解
3. **Given** 用户回答问题，**When** 对话结束，**Then** 系统应该将对话内容保存到当天的日记文件中

---

### User Story 4 - 长期记忆管理 (Priority: P2)

作为程序员用户，我希望能够存储关于我的个人信息、目标和偏好的长期记忆，这样AI助手就能够更好地了解我并提供个性化的建议。

**Why this priority**: 长期记忆是AI个性化服务的基础，虽然不如记录功能紧急，但对提升用户体验至关重要。

**Independent Test**: 可以通过运行 mem 命令并验证记忆是否正确存储来独立测试。用户能够添加记忆并在后续的AI对话中看到AI利用这些信息，即表示功能完成。

**Acceptance Scenarios**:

1. **Given** 用户想要添加个人记忆，**When** 运行记忆管理命令并输入内容，**Then** 系统应该将此信息存储为长期记忆
2. **Given** 用户已有多个记忆，**When** 运行智能问答命令，**Then** AI应该能够利用这些记忆提供更个性化的建议
3. **Given** 用户想要查看现有记忆，**When** 运行记忆查看命令，**Then** 系统应该显示所有已存储的长期记忆

---

### User Story 5 - 定时智能提醒 (Priority: P2)

作为程序员用户，我希望在每天设定的时间自动收到AI的个性化问题，这样我就能够保持持续的反思和成长习惯。

**Why this priority**: 定时提醒帮助用户养成使用习惯，是提升用户留存和产品价值的重要功能。

**Independent Test**: 可以通过设置提醒时间并验证在指定时间是否自动触发问题来独立测试。系统能够在设定时间自动生成并记录问题，即表示功能完成。

**Acceptance Scenarios**:

1. **Given** 用户设置了每日23:00的提醒，**When** 时间到达23:00，**Then** 系统应该自动生成3个问题并写入当天日记文件
2. **Given** 用户当天有记录内容，**When** 定时提醒触发，**Then** 生成的问题应该基于当天的记录内容
3. **Given** 用户更改了提醒时间，**When** 新的时间到达，**Then** 系统应该按照新时间执行定时提醒

---

### Edge Cases

- 当用户指定的 daybook 文件夹路径不存在或没有写入权限时，系统应该提供清晰的错误信息并引导用户解决
- 当 Ollama 服务不可用或指定模型不存在时，系统应该优雅地降级并提供替代方案
- 当用户在同一分钟内多次运行 log 命令时，系统应该按时间顺序记录所有内容
- 当定时提醒触发时用户正在使用系统，系统应该避免冲突并提供适当的通知
- 当日记文件损坏或格式异常时，系统应该能够恢复或重建文件结构

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: 系统必须允许用户通过初始化命令设置个人配置，包括存储路径、时区、AI模型和提醒时间
- **FR-002**: 系统必须支持通过日志记录命令将内容以 `HH:MM:SS 用户内容` 格式实时写入当天的日记文件（格式：YYYY-MM-DD.md）
- **FR-003**: 系统必须能够存储和管理用户的长期记忆信息，通过记忆管理命令添加
- **FR-004**: 系统必须能够基于今日记录和长期记忆生成1个个性化问题，通过智能问答命令触发
- **FR-005**: 系统必须支持定时提醒功能，在用户设定的时间自动生成问题并写入日记文件
- **FR-006**: 系统必须与本地AI服务集成，支持AI模型的调用
- **FR-007**: 系统必须正确处理时区转换，确保所有时间记录都使用用户配置的时区
- **FR-008**: 系统必须验证所有用户输入的有效性，包括文件路径、时间和模型名称
- **FR-009**: 系统必须提供清晰的错误信息和使用指导
- **FR-010**: 系统必须确保数据的持久化存储，包括配置、记忆和日记内容

### Key Entities *(include if feature involves data)*

- **用户配置**: 包含 daybook 文件夹路径、时区、默认AI模型、每日提醒时间
- **日记条目**: 按日期组织的文本文件，包含带时间戳的用户记录（HH:MM:SS 格式）、AI问题和回答
- **长期记忆**: 用户提供的个人信息、目标和偏好，用于AI个性化
- **AI问题**: 基于当日内容和长期记忆生成的个性化反思问题（用户主动交互时1个，定时任务时3个）
- **时间戳**: 所有操作都包含准确的时间信息，使用用户配置的时区

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: 用户能够在极短时间内完成初始化配置并开始使用工具
- **SC-002**: 用户能够快速完成一次日志记录操作（从输入到内容保存完成）
- **SC-003**: AI生成的问题中至少80%能够与用户的当日内容或长期记忆相关联（适用于主动交互和定时提醒）
- **SC-004**: 系统的定时提醒功能可靠性达到95%以上（在设定时间正确触发）
- **SC-005**: 用户对AI问题相关性和有用性的满意度评分达到4.0/5.0以上
- **SC-006**: 系统响应时间：命令执行响应即时，AI问题生成时间在用户可接受范围内
- **SC-007**: 用户留存率：使用工具一周后仍有70%的用户继续使用
- **SC-008**: 错误恢复能力：系统能够从90%的常见错误状态中自动恢复并提供用户指导
