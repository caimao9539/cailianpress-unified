# cailianpress-unified

A unified Cailian Press (CLS / 财联社) data skill for OpenClaw.

This skill provides a single, canonical entrypoint for CLS telegraph data so other skills and scripts do not need to call CLS directly. It standardizes query behavior, output schema, red/highlight logic, and fallback behavior.

## Features

- Unified access to CLS telegraph data
- Canonical red/highlight detection based on `level`
- Heat filtering based on `reading_num`
- Stable normalized output schema for downstream consumers
- CLI interface for terminal and skill-to-skill calls
- Page fallback when the primary NodeAPI path fails
- GitHub-friendly documentation and structure

## Why this exists

In many workspaces, CLS-related logic grows organically and becomes fragmented across:
- direct `nodeapi/telegraphList` calls
- page scraping against `cls.cn/telegraph`
- parsing front-end state blobs such as `__NEXT_DATA__`
- local red-message JSON snapshots
- local formatted push files
- hot-ranking caches from other systems

That creates consistency problems:
- same user request, different answer depending on which path was used
- “red” vs “important” vs “hot” mixed together
- no single source of truth
- difficult downstream reuse by other skills

This skill fixes that by defining one canonical contract.

## V1 canonical design

### Primary source
- `https://www.cls.cn/nodeapi/telegraphList`

### Secondary source
- `https://api3.cls.cn/share/article/{id}`

### Fallback source
- `https://www.cls.cn/telegraph`

### Canonical rules
- `level in {"A", "B"}` → red/highlighted
- `level == "C"` → normal
- `reading_num` → heat / reading count
- `ctime` → publish timestamp
- `shareurl` → article share URL

## Project structure

```text
skills/cailianpress-unified/
├── .gitignore
├── CHANGELOG.md
├── README.md
├── SKILL.md
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
    ├── test_schemas.py
    └── test_service_filters.py
```

## Requirements

- Python 3.10+
- `requests`
- Optional for tests: `pytest`

## Usage

### Raw telegraph items

```bash
python3 skills/cailianpress-unified/scripts/cls_query.py telegraph --hours 1 --limit 10
```

### Red/highlighted items

```bash
python3 skills/cailianpress-unified/scripts/cls_query.py red --hours 24 --format text
```

### Hot items

```bash
python3 skills/cailianpress-unified/scripts/cls_query.py hot --hours 1 --min-reading 10000 --format markdown
```

### Article lookup

```bash
python3 skills/cailianpress-unified/scripts/cls_query.py article --id 2326490 --format text
```

## Output formats

Supported output formats:
- `json` (default): for programmatic use by other skills/scripts
- `text`: for terminal / Telegram / plain text summaries
- `markdown`: for documents and richer reports

## Normalized schema

### `ClsItem`

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

## Integration rules

Once adopted, other skills should not call CLS directly. They should use this skill through:
- `scripts/cls_query.py`
- or `scripts/cls_service.py`

Examples of migration targets:
- hourly pulse reports
- red telegraph summaries
- 1-hour / 24-hour CLS snapshots
- stock-news aggregators that depend on CLS telegraphs

## Known limitations

- V1 focuses on telegraph data, not full article systems
- Article detail extraction is HTML-based and may need future hardening
- Page fallback depends on current page data structure and should be treated as backup only
- Some telegraph items may have an empty `title` field from upstream CLS data

## Testing

If `pytest` is installed:

```bash
cd skills/cailianpress-unified
PYTHONPATH=. python3 -m pytest tests -q
```

## Roadmap

### V1
- unified telegraph query
- unified red query
- unified hot query
- article share URL and basic detail extraction
- page fallback support

### V1.1
- stronger article detail parsing
- better handling for title-less telegraphs
- richer markdown output
- migration guide for dependent skills

### V2
- caching layer
- richer service API
- observability and health checks
- explicit adapter registry

## Publishing checklist

Before pushing to GitHub:
- verify examples still work
- run tests in an environment with `pytest`
- confirm no local-only paths remain in docs
- add a repository-level license if needed
