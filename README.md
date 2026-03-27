# 财联社统一技能 / cailianpress-unified

统一的财联社（CLS / 财联社）数据技能，为 OpenClaw 与其他可复用脚本提供单一、稳定、可审计的数据访问入口。

A unified Cailian Press (CLS / 财联社) data skill for OpenClaw. It provides a single canonical entrypoint for CLS telegraph data so other skills and scripts do not need to call CLS directly.

## 中文说明

### 这是什么

这是一个面向 OpenClaw / AgentSkill 场景设计的财联社统一技能，目标是把分散的财联社接口调用统一收口成一个可复用入口，避免不同脚本、任务、报表各自调用不同财联社接口而导致：

- 电报口径不一致
- 加红、热度、普通电报混用
- 同一请求多次返回结果不同
- 下游技能难以稳定复用

### 当前能力（V1）

- 普通电报查询
- 加红电报查询
- 热度电报查询
- article 详情基础补全
- 页面兜底抓取
- 统一 JSON / Text / Markdown 输出

### 统一规则

- `level in {"A", "B"}` → 加红
- `level == "C"` → 普通
- `reading_num` → 热度/阅读量
- `ctime` → 发布时间戳
- `shareurl` → 财联社文章分享链接

### 主数据源

- 主源：`https://www.cls.cn/nodeapi/telegraphList`
- 详情：`https://api3.cls.cn/share/article/{id}`
- 兜底：`https://www.cls.cn/telegraph`

### 适用场景

适合以下请求或下游技能调用：
- 财联社过去 1 小时电报
- 财联社过去 24 小时加红电报
- 财联社热度前 10
- 财联社文章详情
- 任何依赖财联社电报流的晨报 / 盘中简报 / 快讯聚合

### 快速使用

```bash
python3 skills/cailianpress-unified/scripts/cls_query.py telegraph --hours 1
python3 skills/cailianpress-unified/scripts/cls_query.py red --hours 24 --format text
python3 skills/cailianpress-unified/scripts/cls_query.py hot --hours 1 --min-reading 10000 --format markdown
python3 skills/cailianpress-unified/scripts/cls_query.py article --id 2326490 --format text
```

### 目录结构

```text
skills/cailianpress-unified/
├── README.md
├── SKILL.md
├── CHANGELOG.md
├── LICENSE
├── requirements.txt
├── docs/
│   └── api_contract.md
├── scripts/
│   ├── cls_query.py
│   ├── cls_service.py
│   ├── adapters/
│   │   ├── article_share.py
│   │   ├── telegraph_nodeapi.py
│   │   └── telegraph_page_fallback.py
│   ├── formatters/
│   │   ├── json_formatter.py
│   │   ├── markdown_formatter.py
│   │   └── text_formatter.py
│   └── models/
│       └── schemas.py
└── tests/
```

### 已知限制

- V1 重点解决电报流统一，不是完整财联社内容平台 SDK
- `article` 详情解析目前基于 HTML 提取，后续还可继续增强
- `telegraphList` 当前实现仍以接口返回窗口为主，24 小时真正全量历史能力还可继续增强
- 某些财联社电报上游本身可能没有标题

### 测试

如果环境里安装了 `pytest`：

```bash
cd skills/cailianpress-unified
PYTHONPATH=. python3 -m pytest tests -q
```

---

## English Overview

### What this is

This is a unified CLS telegraph skill for OpenClaw. It centralizes CLS access into one canonical interface and prevents downstream scripts from mixing raw telegraphs, highlighted/red items, hot items, page scraping, and local caches inconsistently.

### Features

- Unified access to CLS telegraph data
- Canonical red/highlight detection based on `level`
- Heat filtering based on `reading_num`
- Stable normalized output schema for downstream consumers
- CLI interface for terminal and skill-to-skill calls
- Page fallback when the primary NodeAPI path fails

### Canonical design

#### Primary source
- `https://www.cls.cn/nodeapi/telegraphList`

#### Secondary source
- `https://api3.cls.cn/share/article/{id}`

#### Fallback source
- `https://www.cls.cn/telegraph`

#### Canonical rules
- `level in {"A", "B"}` → red/highlighted
- `level == "C"` → normal
- `reading_num` → heat / reading count
- `ctime` → publish timestamp
- `shareurl` → article share URL

### Usage

```bash
python3 skills/cailianpress-unified/scripts/cls_query.py telegraph --hours 1 --limit 10
python3 skills/cailianpress-unified/scripts/cls_query.py red --hours 24 --format text
python3 skills/cailianpress-unified/scripts/cls_query.py hot --hours 1 --min-reading 10000 --format markdown
python3 skills/cailianpress-unified/scripts/cls_query.py article --id 2326490 --format text
```

### Normalized schema example

```json
{
  "id": 2326490,
  "title": "沪指翻红 上涨个股近3800只",
  "brief": "【沪指翻红 上涨个股近3800只】财联社3月27日电...",
  "content": "完整正文",
  "level": "B",
  "is_red": true,
  "reading_num": 190163,
  "ctime": 1774577349,
  "published_at": "2026-03-27 10:09:09",
  "shareurl": "https://api3.cls.cn/share/article/2326490?...",
  "stock_list": [],
  "subjects": [],
  "plate_list": [],
  "raw_source": "nodeapi"
}
```

### Publishing notes

Before pushing or packaging, verify:
- examples still match the real CLI behavior
- no local-only paths leak into docs
- tests are run in an environment with `pytest`
- the repository-level `LICENSE` matches your intended publication policy
