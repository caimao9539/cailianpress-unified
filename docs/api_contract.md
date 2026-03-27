# API Contract

## Query types

### `telegraph`
Returns raw telegraph items from the canonical CLS telegraph source.

Parameters:
- `--hours <int>` optional
- `--limit <int>` optional
- `--format json|text|markdown`

### `red`
Returns telegraph items filtered by `level in {A, B}`.

Parameters:
- `--hours <int>` optional
- `--limit <int>` optional
- `--format json|text|markdown`

### `hot`
Returns telegraph items filtered by reading count.

Parameters:
- `--hours <int>` optional
- `--limit <int>` optional
- `--min-reading <int>` optional, default `10000`
- `--format json|text|markdown`

### `article`
Returns or resolves article detail metadata from a known CLS article id.

Parameters:
- `--id <int>` required
- `--format json|text|markdown`

## Canonical field meanings

- `id`: CLS article/telegraph identifier
- `title`: headline
- `brief`: summary
- `content`: full or extended text when available
- `level`: raw CLS level field
- `is_red`: derived boolean, true when `level in {A, B}`
- `reading_num`: CLS reading count
- `ctime`: raw publish timestamp
- `published_at`: normalized Asia/Shanghai datetime string
- `shareurl`: article share URL
- `raw_source`: adapter name used to produce the item
