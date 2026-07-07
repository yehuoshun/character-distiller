# 🎭 Character Distiller

> Distill characters from long-form text — LLM two-stage pipeline, outputting SillyTavern character cards or structured skill folders.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Methodology](https://img.shields.io/badge/methodology-LLM%20Pipeline-green)]()
[![中文文档](https://img.shields.io/badge/文档-中文-red)](README_CN.md)

Throw in a long chunk of text (Galgame scripts / novels / chat logs / any long-form text), specify a character name, and the LLM automatically distills structured character output.

## ✨ Features

- **Text-agnostic**: Works with Galgame scripts, novels, QQ chat logs, any long-form text
- **Two-stage pipeline**: Slice analysis → LLM deduplication → integration & generation
- **Dual output modes**: SillyTavern v3 character cards or structured skill folders (7 files)
- **Optional VNDB**: Provide VNDB data for automatic character portrait matching and basic info enrichment
- **Pure Agent + LLM**: No Python environment required — Agent orchestrates LLM calls directly

## 📦 Output Formats

### Mode 1: SillyTavern v3 Character Card

Generates a complete 10-field character card JSON, importable into SillyTavern, Keystone Tavern, and other frontends:

| Field | Description |
|------|------|
| `name` | Character name |
| `description` | Structured description (personality/appearance/outfit/likes) |
| `personality` | Personality summary |
| `first_mes` | Opening greeting |
| `mes_example` | Example dialogues separated by `<START>` |
| `scenario` | Default scenario/relationship setting |
| `system_prompt` | Roleplay system instruction |
| `post_history_instructions` | Context reinforcement instruction |
| `depth_prompt` | Deep psychological layer |
| `character_book_entries` | Lorebook/Worldbook entries |

### Mode 2: Structured Skill Folder

```
{character}-skill-main/
├── SKILL.md          # Entry point with YAML frontmatter + roleplay rules
├── soul.md           # Inner drives, values, emotional core
├── limit.md          # Guardrails, unsupported topics
└── resource/
    ├── behavior_guide.md       # Behavior patterns, habits, situational reactions
    ├── speech_patterns.md      # Speech style, verbal tics, address patterns
    ├── relationship_dynamics.md # Relationship dynamics with other characters
    └── key_life_events.md      # Key experiences, turning points, memory anchors
```

## 🔄 Pipeline

```
Input text
  │
  ├─→ Token counting (tiktoken) → Slice into ~50K token chunks
  │
  ├─→ Stage 1: Per-slice LLM character analysis
  │     ├─ Skills mode: Output Markdown summaries
  │     └─ Chara Card mode: Output JSON (character_analysis + lorebook_entries)
  │
  ├─→ Token budget exceeded? → LLM deduplication (identify and remove cross-slice duplicates)
  │
  └─→ Stage 2: Integration & final generation
        ├─ Skills mode: LLM generates 7 Markdown files
        └─ Chara Card mode: LLM generates 10 fields + fills JSON template
```

## 🚀 Usage

### Via AI Agent (Recommended)

Hand the prompt templates from this repository to a tool-calling AI Agent and chat directly:

> "Extract a character card for 'Asaba Akane' from this Galgame script"

The Agent automatically handles slicing, analysis, deduplication, and integration.

### Via Original Project Web UI

The original project [GalgameCharacterSkills](https://github.com/JodieRuth/GalgameCharacterSkills) provides a Flask web interface with file upload, VNDB lookup, and checkpoint resume support.

```bash
git clone https://github.com/JodieRuth/GalgameCharacterSkills.git
cd GalgameCharacterSkills
pip install -r requirements.txt
python main.py
# Open http://127.0.0.1:5000
```

## 📂 Repository Structure

```
character-distiller/
├── README.md                           # This file (English)
├── README_CN.md                        # 中文文档
├── SKILL.md                            # Agent Skill entry (OpenClaw format)
└── references/
    ├── prompt-templates.md             # 5 complete prompt templates
    └── chara-card-template.md          # SillyTavern v3 JSON template
```

## ⚙️ Model Recommendations

- **Complex characters** (multi-faceted personality, rich background): Claude 3.5 Sonnet / DeepSeek V4 Pro / GLM-5
- **Simple characters** (tagged personality): GPT-4o-mini / DeepSeek V4 Flash
- **Deep reasoning needed**: Models with reasoning/thinking mode enabled

## ⚠️ Notes

1. **Character name must be specified** — distillation is character-granular, not full-text extraction of all characters
2. **Galgame scripts work best** — high dialogue density yields cleaner extraction signals
3. **Novels also work** — but narrative+dialogue mix reduces precision compared to pure dialogue text
4. **Chat logs are usable** — multi-person chat has high noise; filter to target character's messages first
5. **Manual review recommended** — LLMs may miss details or introduce bias

## 🙏 Acknowledgments

This project's methodology is derived from **[JodieRuth/GalgameCharacterSkills](https://github.com/JodieRuth/GalgameCharacterSkills)**. Huge thanks to the original author for their excellent work.

The original project provides a complete Flask web application with VNDB integration, checkpoint resume, streaming responses, and more. This repository abstracts the core distillation methodology into prompt templates and Agent workflows, enabling pure Agent + LLM operation.

## 📄 License

MIT © 2026
