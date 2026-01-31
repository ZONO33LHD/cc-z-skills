---
name: bigquery
description: This skill should be used when the user asks about "BigQuery", "BQ", "Google BigQuery", writes or optimizes SQL queries for BigQuery, discusses query performance, cost optimization, or mentions BigQuery-specific features like partitioning, clustering, or materialized views.
version: 1.0.0
---

# BigQuery Skill

BigQueryのSQLクエリ作成・最適化とベストプラクティスを提供します。

## When This Skill Applies

- BigQueryのSQLクエリを作成・編集する場合
- クエリのパフォーマンス最適化
- コスト削減のアドバイス
- BigQuery固有の機能の使用

## Query Writing Guidelines

### Standard SQL を使用

```sql
-- 推奨: Standard SQL
SELECT column FROM `project.dataset.table`

-- 非推奨: Legacy SQL
SELECT column FROM [project:dataset.table]
```

### SELECT * を避ける

```sql
-- 非推奨: 全カラム取得はコスト増
SELECT * FROM `project.dataset.table`

-- 推奨: 必要なカラムのみ指定
SELECT id, name, created_at FROM `project.dataset.table`
```

### パーティションフィルタを活用

```sql
-- パーティションカラムでフィルタしてスキャン量削減
SELECT *
FROM `project.dataset.partitioned_table`
WHERE _PARTITIONDATE BETWEEN '2024-01-01' AND '2024-01-31'
```

## Performance Best Practices

### 1. JOINの最適化

```sql
-- 大きいテーブルを左に、小さいテーブルを右に配置
SELECT a.id, b.name
FROM large_table a
JOIN small_table b ON a.id = b.id
```

### 2. APPROX関数の活用

```sql
-- 厳密な値が不要な場合はAPPROX関数を使用
SELECT APPROX_COUNT_DISTINCT(user_id) as unique_users
FROM `project.dataset.events`
```

### 3. ウィンドウ関数の効率的な使用

```sql
-- QUALIFY句でウィンドウ関数の結果をフィルタ
SELECT *
FROM `project.dataset.table`
QUALIFY ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) = 1
```

### 4. CTEの活用

```sql
-- Common Table Expressions で可読性向上
WITH active_users AS (
  SELECT user_id
  FROM `project.dataset.events`
  WHERE event_date > DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
)
SELECT u.*, a.user_id IS NOT NULL as is_active
FROM `project.dataset.users` u
LEFT JOIN active_users a ON u.id = a.user_id
```

## Cost Optimization

### 1. プレビュークエリの活用

```sql
-- LIMITは課金に影響しないが、プレビューには有用
SELECT * FROM `project.dataset.table` LIMIT 100
```

### 2. パーティショニングとクラスタリング

```sql
-- テーブル作成時にパーティションとクラスタを設定
CREATE TABLE `project.dataset.new_table`
PARTITION BY DATE(created_at)
CLUSTER BY user_id, category
AS SELECT * FROM `project.dataset.source_table`
```

### 3. マテリアライズドビュー

```sql
-- 頻繁に実行する集計クエリはマテリアライズドビューに
CREATE MATERIALIZED VIEW `project.dataset.daily_stats`
AS SELECT
  DATE(created_at) as date,
  COUNT(*) as count,
  SUM(amount) as total_amount
FROM `project.dataset.transactions`
GROUP BY DATE(created_at)
```

### 4. BI Engine の活用

- 小規模データセット（< 10GB）はBI Engineでキャッシュ
- ダッシュボードクエリの高速化

## Common Patterns

### 日付範囲のクエリ

```sql
-- 過去N日間
WHERE created_at >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)

-- 特定月
WHERE DATE_TRUNC(created_at, MONTH) = '2024-01-01'
```

### 重複排除

```sql
-- 最新レコードのみ取得
SELECT * EXCEPT(row_num)
FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY id ORDER BY updated_at DESC) as row_num
  FROM `project.dataset.table`
)
WHERE row_num = 1
```

### 配列操作

```sql
-- 配列の展開
SELECT id, tag
FROM `project.dataset.table`, UNNEST(tags) as tag

-- 配列の集約
SELECT user_id, ARRAY_AGG(DISTINCT category) as categories
FROM `project.dataset.events`
GROUP BY user_id
```

### JSON操作

```sql
-- JSONフィールドの抽出
SELECT
  JSON_VALUE(data, '$.name') as name,
  JSON_QUERY(data, '$.items') as items
FROM `project.dataset.json_table`
```

## Anti-Patterns to Avoid

1. **ORDER BY without LIMIT**: 大量データのソートは高コスト
2. **CROSS JOIN**: 意図しない直積は避ける
3. **Correlated subqueries**: JOINに書き換える
4. **過度なNESTED/REPEATED**: フラット化を検討
5. **不要なDISTINCT**: GROUP BYで代替可能か検討

## Debugging Tips

### クエリプランの確認

```sql
-- EXPLAIN文でクエリプラン確認
EXPLAIN SELECT * FROM `project.dataset.table` WHERE id = 1
```

### スロット使用量の確認

```sql
-- INFORMATION_SCHEMAでジョブ情報を確認
SELECT
  job_id,
  total_bytes_processed,
  total_slot_ms
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
ORDER BY total_bytes_processed DESC
LIMIT 10
```
