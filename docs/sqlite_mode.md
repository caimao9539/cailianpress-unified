# SQLite Mode

This mode upgrades local CLS persistence from JSONL snapshots to a SQLite-based storage model.

## Goals

- No duplicate records in the main query table
- No missing records caused by short live windows, as long as the ingest cadence is fast enough
- Preserve raw ingest logs for audit and replay
- Support fast queries for 1-hour / 24-hour / red / hot lookups

## Current recommended cadence

Default: every 10 minutes.

## Why SQLite mode exists

The stable direct source currently available is:

- `https://www.cls.cn/nodeapi/telegraphList`

Observed characteristics:
- single request usually returns about `20` telegraphs
- effective time coverage changes with news density
- direct use of the front-end endpoint `/v1/roll/get_roll_list` is blocked by signature validation (`签名错误`)

Because of that, stable history should rely on:
- repeated NodeAPI ingestion
- local SQLite deduplication
- raw log preservation

## Ingest command

```bash
python3 skills/cailianpress-unified/scripts/cls_sqlite_ingest.py
```

## Query commands

```bash
python3 skills/cailianpress-unified/scripts/cls_sqlite_query.py telegraph --hours 1 --limit 20
python3 skills/cailianpress-unified/scripts/cls_sqlite_query.py red --hours 24 --limit 20
python3 skills/cailianpress-unified/scripts/cls_sqlite_query.py hot --hours 24 --min-reading 50000 --limit 20
python3 skills/cailianpress-unified/scripts/cls_sqlite_status.py
```

## Storage layout

```text
skills/cailianpress-unified/data/
  telegraph.db
```

## Core tables

- `telegraph_main`: deduplicated current telegraphs
- `telegraph_raw_log`: every ingest event, kept for audit
- `telegraph_subjects`: subject relationships
- `telegraph_stocks`: stock relationships
- `telegraph_plates`: plate relationships

## Suggested cron

```cron
*/10 * * * * cd /home/tim/.openclaw/workspace && python3 skills/cailianpress-unified/scripts/cls_sqlite_ingest.py >> /tmp/cls_sqlite_ingest.log 2>&1
```

## Dedup strategy

- Main table unique key: `id`
- Update rule: prefer `modified_time`, fallback to `content_hash`
- Raw log table keeps every capture event for audit
- Main table keeps one current version per telegraph

## Operational notes

- `telegraph_main` is the main query surface
- `telegraph_raw_log` is not the main query surface; it exists for audit, replay, and debugging
- Subject statistics may exceed unique telegraph count because one telegraph can belong to multiple subjects
