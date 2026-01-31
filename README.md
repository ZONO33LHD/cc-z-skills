# cc-z-skills

Claude Code用のカスタムスキル集。

## スキル一覧

| スキル | 説明 |
|--------|------|
| [bigquery](./bigquery/) | BigQueryのクエリ支援・ベストプラクティス |

## 使用方法

`~/.claude/settings.json`にプラグインパスを追加:

```json
{
  "plugins": [
    "/path/to/cc-z-skills/bigquery"
  ]
}
```