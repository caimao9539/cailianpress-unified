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

### 当前能力（V1.2 / 2026-03-29 文档同步）

- 普通电报查询
- 加红电报查询
- 热度电报查询
- article 详情基础补全
- 页面兜底抓取
- 统一 JSON / Text / Markdown 输出
- SQLite 本地存储模式（推荐）
- 原始抓取日志 + 原始层去重主表
- 主题关联入库与主题归类查询

### 统一规则

- `level in {"A", "B"}` → 加红
- `level == "C"` → 普通
- `reading_num` → 热度/阅读量
- `ctime` → 发布时间戳
- `shareurl` → 财联社文章分享链接

### 主数据源

- 主源：`https://www.cls.cn/nodeapi/telegraphList`
- 补充源：`https://www.cls.cn/nodeapi/updateTelegraphList`
- 详情：`https://api3.cls.cn/share/article/{id}`
- 兜底：`https://www.cls.cn/telegraph`

### 数据获取现状

当前已确认：
- `nodeapi/telegraphList` 可直接稳定返回电报数据
- `nodeapi/updateTelegraphList` 可访问，但默认返回与主接口同批数据
- `nodeapi/refreshTelegraphList` 仅返回轻量结构，不适合作为主抓取源
- `/v1/roll/get_roll_list` 为前端主接口，但直接请求会返回 `签名错误`，因此当前不作为稳定抓取主线

### SQLite 模式（推荐，当前默认每 10 分钟抓一次）

```bash
python3 skills/cailianpress-unified/scripts/cls_sqlite_ingest.py
python3 skills/cailianpress-unified/scripts/cls_sqlite_query.py telegraph --hours 1 --limit 20
python3 skills/cailianpress-unified/scripts/cls_sqlite_query.py red --hours 24 --limit 20
python3 skills/cailianpress-unified/scripts/cls_sqlite_query.py hot --hours 24 --min-reading 50000 --limit 20
python3 skills/cailianpress-unified/scripts/cls_sqlite_status.py
sqlite3 skills/cailianpress-unified/data/telegraph.db "SELECT COUNT(*) FROM telegraph_raw_main;"
sqlite3 skills/cailianpress-unified/data/telegraph.db "SELECT id, level, title FROM telegraph_raw_main WHERE level IN ('A','B') ORDER BY ctime DESC LIMIT 20;"
```

### 当前存储结构（已升级）

- **主真相源表**：`telegraph_raw_main`
  - 去重后的原始层主表
  - 尽量保留 NodeAPI 原始字段与原始 JSON
- **抓取日志表**：`telegraph_raw_log`
  - 每轮抓取痕迹
- **查询视图**：`v_telegraph_main`
  - 供日常查询使用
  - 在原始层基础上派生 `is_red` / `telegraph_type` / `published_at`
- **关联表**：
  - `telegraph_subjects`
  - `telegraph_stocks`
  - `telegraph_plates`

### 去重与更新规则

- 原始层主去重键：`id`
- 更新判断：优先 `modified_time`，其次 `content_hash`
- 抓取日志保留每次采集痕迹
- 查询默认走：`v_telegraph_main`

### 原始层现在会保留的关键扩展字段

除了常规电报正文和等级、阅读量外，主真相源还保留：
- `tags_json`
- `sub_titles_json`
- `timeline_json`
- `ad_json`
- `assoc_fast_fact_json`
- `assoc_article_url`
- `assoc_video_title`
- `assoc_video_url`
- `assoc_credit_rating_json`
- `stock_list_json`
- `subjects_json`
- `plate_list_json`
- `raw_json`

也就是说，当前已经从“精选主表”升级为“原始层去重主表”。

### 采集策略

- 当前默认频率：**每 10 分钟一次**
- 设计目标：
  - 尽量不漏数
  - 通过去重消化重复抓取

详见：`docs/sqlite_mode.md`

### 目录结构

```text
skills/cailianpress-unified/
├── README.md
├── SKILL.md
├── CHANGELOG.md
├── LICENSE
├── requirements.txt
├── docs/
│   ├── api_contract.md
│   ├── snapshot_mode.md
│   └── sqlite_mode.md
├── scripts/
│   ├── cls_query.py
│   ├── cls_service.py
│   ├── cls_snapshot.py
│   ├── cls_history_query.py
│   ├── cls_sqlite_ingest.py
│   ├── cls_sqlite_query.py
│   ├── cls_sqlite_status.py
│   ├── db.py
│   ├── storage.py
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

- 仅靠实时 `telegraphList` 不能稳定覆盖长历史，因此推荐启用 SQLite 持续抓取
- 当前稳定主抓取源仍是 `nodeapi` 链，不是签名保护的 `/v1/roll/get_roll_list`
- `article` 详情解析目前基于 HTML 提取，后续还可继续增强
- 某些财联社电报上游本身可能没有标题

---

## English Overview

This skill now uses a raw-first SQLite architecture:
- `telegraph_raw_main` as the deduplicated source-of-truth table
- `telegraph_raw_log` as the ingest audit log
- `v_telegraph_main` as the main query view for normal usage

The stable direct source is currently `nodeapi/telegraphList`, and the recommended ingest cadence is 10 minutes.
