# Changelog

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
