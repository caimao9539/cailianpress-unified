# Changelog

## 0.1.1 - 2026-03-28

Documentation and storage update release.

### Added
- SQLite ingest mode with `telegraph_main` and `telegraph_raw_log`
- SQLite query CLI and status CLI
- Subject relationship storage
- Documentation for stable NodeAPI-based ingestion
- Documentation for current 10-minute ingest cadence

### Changed
- Switched recommended persistence model from JSONL-first to SQLite-first
- Clarified current stable direct source as `nodeapi/telegraphList`
- Clarified that `/v1/roll/get_roll_list` is signature-protected and not the current stable direct ingest path

## 0.1.0 - 2026-03-27

Initial V1 release.

### Added
- Unified CLS telegraph CLI entrypoint
- Unified service layer for telegraph, red, hot, and article queries
- Canonical schema definitions
- NodeAPI adapter for `telegraphList`
- Page fallback adapter for `cls.cn/telegraph`
- Basic article share/detail extraction
- Text and Markdown formatters
- Initial tests for schema and filter logic
- GitHub-oriented documentation (`README.md`, `SKILL.md`, `docs/api_contract.md`)

### Notes
- Primary source is `https://www.cls.cn/nodeapi/telegraphList`
- Canonical red rule is `level in {A, B}`
- `pytest` execution still depends on the target environment having `pytest` installed
