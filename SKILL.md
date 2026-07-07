---
name: character-distiller
description: |
  从长文本（Galgame脚本、小说、聊天记录等）中蒸馏角色，输出结构化角色分析或 SillyTavern 角色卡。
  触发词：蒸馏角色、提取角色、角色卡生成、角色分析、character card、Galgame 角色、VNDB 角色。
  适用场景：用户给一段文本+角色名，要提取角色设定、生成角色卡、或做角色对话模拟。
---

# Character Distiller — 长文本角色蒸馏

把一大段文本（Galgame 脚本 / 小说 / 聊天记录 / 任意长篇文本）丢进来，指定一个角色名，LLM 把角色蒸馏成结构化产物。

## 输出模式

### 模式一：角色分析（Skills 模式）
输出一个 `{角色名}-skill-main/` 文件夹，包含 7 个文件：

```
{角色名}-skill-main/
├── SKILL.md          # 入口，含 YAML frontmatter + roleplay 规则
├── soul.md           # 内在驱动力、价值观、情感核心
├── limit.md          # 边界约束、不支持的领域
└── resource/
    ├── behavior_guide.md       # 行为模式、习惯、情境反应
    ├── speech_patterns.md      # 语言风格、口癖、称呼习惯
    ├── relationship_dynamics.md # 与其他人物的关系动态
    └── key_life_events.md      # 关键经历、转折点、记忆锚点
```

### 模式二：角色卡（Chara Card 模式）
输出一个 SillyTavern v3 兼容的 `.json` 角色卡，包含 9 个字段：

| 字段 | 说明 |
|------|------|
| name | 角色名 |
| description | `[{{char}}'s Personality= ...]` 结构化描述 |
| personality | 性格简述 |
| first_mes | 开场白 |
| mes_example | `<START>` 分隔的示例对话 |
| scenario | 默认场景/关系设定 |
| system_prompt | 角色扮演指令 |
| post_history_instructions | 上下文强化指令 |
| depth_prompt | 深层心理层 |
| character_book_entries | Lorebook/Worldbook 条目 |

可选：对接 VNDB 自动匹配立绘，合成带图的 `.png` 角色卡。

## 蒸馏流程（两阶段流水线）

### 阶段一：文本切片 → 逐片分析

```
输入文本 → tiktoken 计 Token → 按 50K token/片 切片 → 每片独立 LLM 分析
```

每片分析调用 LLM，输出到 `{角色名}_summaries/` 目录。

- Skills 模式：每片输出一个 Markdown 摘要文件
- Chara Card 模式：每片输出一个 JSON 文件（含 `character_analysis` + `lorebook_entries`）

支持并发处理多片（ThreadPoolExecutor）。

### 阶段二：整合 → 生成最终产物

1. **LLM 去重压缩**（仅在 token 超预算时触发）：将多片分析结果分组，LLM 识别并删除跨片重复内容
2. **优先级排序**（head-tail weighted）：头尾交替选取，确保重要内容不丢失
3. **整合生成**：
   - Skills 模式：LLM 逐文件生成 7 个 Markdown 文件
   - Chara Card 模式：LLM 逐字段生成 9 个字段 + 集成角色分析 + 填充 JSON 模板

## 关键参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| slice_size_k | 50 | 切片大小（千 token） |
| target_total_chars | 200000 | 整合阶段最大字符数 |
| context_limit | 模型相关 | 自动检测（litellm.get_model_info），回退 115000 |
| max_retries | 3 | LLM 调用失败重试次数 |
| reasoning_effort | 空 | 传给 litellm 的推理强度 |

## LLM 配置

通过 litellm 抽象层，兼容任何 OpenAI API 格式的端点：

- **Base URL**：API 地址（如 `https://api.openai.com/v1`）
- **Model Name**：模型名（自动补全 provider 前缀，如 `deepseek/`、`openai/`、`anthropic/`）
- **API Key**：密钥
- **Stream**：支持流式响应

VNDB 数据（可选）会作为高优先级参考数据注入 system prompt。

## 使用方式（Agent 执行）

### 纯 LLM 蒸馏（不需要本地部署 Flask）

用户提供文本和角色名后，直接按以下步骤执行：

**Step 1: 文本预处理**
```
1. 将用户提供的文本保存到临时文件
2. 用 tiktoken (cl100k_base) 计算 token 数
3. 按 ~50K token/片 估算切片数
```

**Step 2: 角色分析 Prompt 模板**

以下 prompt 是核心蒸馏逻辑，直接复用原项目的分析框架：

<details>
<summary>Skills 模式 System Prompt（点击展开）</summary>

```
You are a professional character analysis assistant.
Your task is to analyze text content and extract a comprehensive character profile for "{角色名}".

ANALYSIS APPROACH - Blend of third-person observation and first-person perspective:

## PART A: Character Memory (客观事实与经历)
Extract factual information about the character:
- Basic identity (name, age, appearance, background)
- Key life events and timeline
- Important relationships and their dynamics
- Core values and beliefs (as demonstrated through actions)
- Significant memories or turning points
- Habits and routines

Present this section as OBSERVED FACTS from the text, using third-person perspective.

## PART B: Character Persona (行为模式与表达风格)
Extract actionable behavioral patterns that can drive dialogue:

### Layer 1: Identity Anchors
- Who they are at their core
- Self-perception vs. how others see them

### Layer 2: Expression Style (CRITICAL - be specific)
- Speech patterns: exact phrases, sentence structures, verbal tics
- Tone variations by context (formal/casual/emotional)
- Punctuation and rhythm habits
- Vocabulary preferences
- How they address different people

### Layer 3: Emotional & Decision Patterns
- How they express different emotions
- Decision-making style
- Conflict response patterns
- Stress coping mechanisms

### Layer 4: Behavioral Rules
- Physical habits and mannerisms
- Social interaction patterns
- Default responses to common situations
- "If-then" behavioral rules

## CRITICAL REQUIREMENTS:
1. Focus EXCLUSIVELY on "{角色名}"
2. Include SPECIFIC examples from text - actual quotes, exact phrases
3. Distinguish between: what's explicitly shown vs what's reasonably inferred
4. Capture NUANCE: contradictions, growth, context-dependent behaviors

## OUTPUT FORMAT:
Use markdown with clear hierarchy. Save to the specified file path.
```

</details>

**Step 3: 整合与输出**
```
1. 汇总所有切片分析结果
2. 如果总 token 超过目标预算 → LLM 去重压缩
3. Skills 模式：生成 7 个 Markdown 文件
4. Chara Card 模式：生成 9 个字段 + 填充模板 JSON
```

### 简化版（单角色、短文本）

如果文本较短（< 50K token），跳过切片步骤，直接一次性分析：

```
1. 读取文本 → 保存
2. 用上述 prompt 调用 LLM 分析
3. 输出角色分析文件
```

## 源码参考

原始项目：`GalgameCharacterSkills/main.py`（约 2200 行）
- 切片逻辑：`FileProcessor.slice_multiple_files()`
- LLM 交互：`LLMInteraction` 类（litellm 封装，含流式解析、重试、工具调用）
- 压缩去重：`_compress_with_llm()` + `compress_content_with_llm()`
- 角色卡生成：`generate_character_card_with_tools()`（逐字段 Tool Call 模式）
- 技能文件夹：`generate_skills_folder_init()`（7 文件 + 可选额外文件）
- 断点续传：`CheckpointManager`（支持中途恢复）
- JSON 模板：`utils/chara_card_template.json`（SillyTavern v3 格式）

## 注意事项

1. **角色名必须指定**：蒸馏是角色粒度的，不是全文提取所有角色
2. **Galgame 脚本最佳**：对话密度高，提取信号更纯净
3. **小说也能用**：但叙述+对话混合，提取精度略低于纯对话文本
4. **聊天记录可用**：多人混聊噪声大，建议先过滤到目标角色发言
5. **VNDB 是可选项**：有 VNDB 数据时注入 system prompt 提高准确度，没有也能跑
6. **模型选择**：推荐 reasoning 能力强的模型（如 deepseek-v4-pro、claude），复杂角色需要深层推理
