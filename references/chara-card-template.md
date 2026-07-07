# SillyTavern v3 角色卡 JSON 模板

```json
{
    "spec": "chara_card_v3",
    "spec_version": "3.0",
    "data": {
        "name": "{{name}}",
        "description": "{{description}}",
        "personality": "{{personality}}",
        "scenario": "{{scenario}}",
        "first_mes": "{{first_mes}}",
        "mes_example": "{{mes_example}}",
        "system_prompt": "{{system_prompt}}",
        "post_history_instructions": "{{post_history_instructions}}",
        "depth_prompt": "{{depth_prompt}}",
        "creator_notes": "{{creatorcomment}}",
        "character_version": "1.0",
        "creator": "{{creator}}",
        "tags": {{tags}},
        "alternate_greetings": [],
        "group_only_greetings": [],
        "character_book": {
            "name": "{{world_name}}",
            "description": "World book for {{world_name}}",
            "scan_depth": 50,
            "token_budget": 2048,
            "recursive_scanning": false,
            "extensions": {},
            "entries": {{character_book_entries}}
        },
        "extensions": {
            "talkativeness": "1.0",
            "fav": false,
            "world": "",
            "depth_prompt": {
                "prompt": "",
                "depth": 4,
                "role": "system"
            },
            "create_date": "{{create_date}}"
        },
        "create_date": "{{create_date}}"
    }
}
```

## 字段填充说明

| 占位符 | 类型 | 来源 |
|--------|------|------|
| `{{name}}` | string | LLM 生成 / VNDB |
| `{{description}}` | string | LLM 生成（SillyTavern 格式） |
| `{{personality}}` | string | LLM 生成 |
| `{{scenario}}` | string | LLM 生成 |
| `{{first_mes}}` | string | LLM 生成 |
| `{{mes_example}}` | string | LLM 生成（<START> 分隔） |
| `{{system_prompt}}` | string | LLM 生成 |
| `{{post_history_instructions}}` | string | LLM 生成 |
| `{{depth_prompt}}` | string | LLM 生成 |
| `{{creatorcomment}}` | string | 自动生成（含 VNDB ID 如适用） |
| `{{creator}}` | string | 用户输入或默认 "AI Character Generator" |
| `{{tags}}` | array | 自动生成 |
| `{{world_name}}` | string | 同 name |
| `{{character_book_entries}}` | array | LLM 生成的 lorebook 条目 |
| `{{create_date}}` | string | ISO 时间戳 |

## Lorebook Entry 格式

```json
{
    "id": 0,
    "keys": ["关键词1", "关键词2"],
    "secondary_keys": [],
    "comment": "条目名称",
    "content": "{{user}}: 问题\n{{char}}: 回答",
    "constant": false,
    "selective": true,
    "insertion_order": 100,
    "enabled": true,
    "position": "before_char",
    "use_regex": true,
    "extensions": {
        "position": 0,
        "exclude_recursion": false,
        "display_index": 0,
        "probability": 100,
        "useProbability": true,
        "depth": 4,
        "selectiveLogic": 0,
        "group": "",
        "group_override": false,
        "group_weight": 100,
        "prevent_recursion": false,
        "delay_until_recursion": false,
        "scan_depth": null,
        "match_whole_words": null,
        "use_group_scoring": false,
        "case_sensitive": null,
        "automation_id": "",
        "role": 0,
        "vectorized": false,
        "sticky": 0,
        "cooldown": 0,
        "delay": 0
    }
}
```
