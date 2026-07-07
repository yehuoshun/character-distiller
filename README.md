# 🎭 Character Distiller

> 从长文本中蒸馏角色 — LLM 两阶段流水线，输出 SillyTavern 角色卡或结构化角色技能文件夹。

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Methodology](https://img.shields.io/badge/methodology-LLM%20Pipeline-green)]()

把一大段文本（Galgame 脚本 / 小说 / 聊天记录 / 任意长篇文本）丢进来，指定一个角色名，LLM 自动蒸馏出结构化角色产物。

## ✨ 核心能力

- **文本无关**：Galgame 脚本、小说、QQ 聊天记录、任何长文本都能用
- **两阶段流水线**：切片分析 → LLM 去重压缩 → 整合生成
- **双模式输出**：SillyTavern v3 角色卡 或 结构化技能文件夹（7 文件）
- **VNDB 可选**：提供 VNDB 数据可自动匹配角色立绘和基础信息
- **纯 Agent + LLM**：不依赖 Python 环境，Agent 直接编排 LLM 调用

## 📦 输出格式

### 模式一：SillyTavern v3 角色卡

生成 9 个字段的完整角色卡 JSON，可直接导入 SillyTavern / Keystone 酒馆等前端：

| 字段 | 说明 |
|------|------|
| `name` | 角色名 |
| `description` | 结构化描述（人格/外貌/服装/喜好） |
| `personality` | 性格简述 |
| `first_mes` | 开场白 |
| `mes_example` | `<START>` 分隔的示例对话 |
| `scenario` | 默认场景/关系设定 |
| `system_prompt` | 角色扮演系统指令 |
| `post_history_instructions` | 上下文强化指令 |
| `depth_prompt` | 深层心理层 |
| `character_book_entries` | Lorebook/Worldbook 条目 |

### 模式二：结构化技能文件夹

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

## 🔄 蒸馏流程

```
输入文本
  │
  ├─→ tiktoken 计 Token → 按 ~50K token/片 切片
  │
  ├─→ 阶段一：每片独立 LLM 角色分析
  │     ├─ Skills 模式：输出 Markdown 摘要
  │     └─ Chara Card 模式：输出 JSON（character_analysis + lorebook_entries）
  │
  ├─→ Token 超预算？→ LLM 去重压缩（识别并删除跨片重复内容）
  │
  └─→ 阶段二：整合生成最终产物
        ├─ Skills 模式：LLM 逐文件生成 7 个 Markdown 文件
        └─ Chara Card 模式：LLM 逐字段生成 9 个字段 + 填充 JSON 模板
```

## 🚀 使用方式

### 通过 AI Agent（推荐）

将本仓库的 Prompt 模板交给支持工具调用的 AI Agent，直接对话即可：

> "帮我从这个 Galgame 脚本里提取「逢坂明日葉」的角色卡"

Agent 会自动执行切片、分析、去重、整合的全流程。

### 通过原项目 Web UI

原项目 [GalgameCharacterSkills](https://github.com/JodieRuth/GalgameCharacterSkills) 提供 Flask Web 界面，支持文件上传、VNDB 查询、断点续传。

```bash
git clone https://github.com/JodieRuth/GalgameCharacterSkills.git
cd GalgameCharacterSkills
pip install -r requirements.txt
python main.py
# 打开 http://127.0.0.1:5000
```

## 📂 仓库结构

```
character-distiller/
├── README.md                           # 本文件
├── SKILL.md                            # Agent Skill 入口（OpenClaw 格式）
└── references/
    ├── prompt-templates.md             # 5 套完整 Prompt 模板
    └── chara-card-template.md          # SillyTavern v3 JSON 模板
```

## ⚙️ 模型推荐

- **复杂角色**（多面性格、丰富背景）：Claude 3.5 Sonnet / DeepSeek V4 Pro / GLM-5
- **简单角色**（标签化性格）：GPT-4o-mini / DeepSeek V4 Flash
- **需要推理深度**：启用 reasoning/thinking 模式的模型

## ⚠️ 注意事项

1. **角色名必须指定** — 蒸馏是角色粒度的，不是全文提取所有角色
2. **Galgame 脚本效果最佳** — 对话密度高，提取信号更纯净
3. **小说也能用** — 但叙述+对话混合，提取精度略低于纯对话文本
4. **聊天记录可用** — 多人混聊噪声大，建议先过滤到目标角色发言
5. **生成结果建议人工复核** — LLM 可能遗漏细节或引入偏差

## 🙏 鸣谢

本项目的方法论源自 **[JodieRuth/GalgameCharacterSkills](https://github.com/JodieRuth/GalgameCharacterSkills)**，感谢原作者的卓越工作。

原项目提供完整的 Flask Web 应用，包含 VNDB 集成、断点续传、流式响应等工程实现。本仓库将其核心蒸馏方法论抽象为 Prompt 模板和 Agent 工作流，使其可以在纯 Agent + LLM 环境下运行。

## 📄 License

MIT © 2026
