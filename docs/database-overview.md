# Database Overview

最終確認日: 2026-09-18

## Supabase Project

- Project: `Slot-Data-Analysis-DB`
- Project Ref: `lpabzfnxlxljzcaofxkm`
- Region: `ap-northeast-1`
- PostgreSQL: 17
- Status: ACTIVE_HEALTHY

## public テーブル

| Table | Rows (概数) | RLS | Primary Key |
|---|---:|---|---|
| prefectures | 4 | ON | prefecture_id |
| halls | 74 | ON | hall_id |
| models | 9 | ON | model_id |
| results | 2,060,476 | ON | (hall_id, model_id, unit_no, date) |
| model_setting_benchmarks | 252 | ON | (model_id, metric_key, setting_no) |

### 主な外部キー

- `halls.prefecture_id -> prefectures.prefecture_id`
- `results.hall_id -> halls.hall_id`
- `results.model_id -> models.model_id`
- `model_setting_benchmarks.model_id -> models.model_id`

## Materialized Views

- `dashboard_active_units_mv`
- `dashboard_active_hall_models_mv`
- `dashboard_results_mv`
- `dashboard_pivot_daily_mv`
- `dashboard_unit_pattern_monthly_mv`

いずれも Dashboard 系の集計・高速化用。現在は直接公開を制限し、RPC 経由で利用する方針。

## Views

- `hall_model_medal_daily_v`
- `latest_models`
- `latest_units_per_hall`
- `latest_units_results`
- `medal_rate_by_hall`
- `medal_rate_by_hall_and_day_last`
- `medal_rate_by_hall_and_month_day`
- `medal_rate_by_hall_and_weekday`
- `medal_rate_by_hall_model_month_history`
- `medal_rate_by_model`
- `medal_rate_by_unit_no`
- `rb_medal_rate_long_3months`
- `result_joined`

## RPC / Functions

- `get_dashboard_pivot(...)`
- `get_dashboard_unit_pattern_summary(...)` 2 overloads
- `refresh_dashboard_materialized_views()`

`refresh_dashboard_materialized_views()` は `SECURITY DEFINER`、固定 `search_path`、5分の statement timeout、advisory lock を使用。

## 主な Index

### results

- PK: `(hall_id, model_id, unit_no, date)`
- `idx_results_date_hall_model_unit (date, hall_id, model_id, unit_no)`
- `idx_results_model_id (model_id)`

### dashboard materialized views

各 MV に `REFRESH MATERIALIZED VIEW CONCURRENTLY` を可能にする Unique Index が存在。

## RLS / Security

- `prefectures`, `halls`, `models`: public SELECT policy
- `results`: anon / authenticated SELECT policy
- `model_setting_benchmarks`: RLS ON、anon / authenticated の table privilege は削除済み
- Security Advisor は `model_setting_benchmarks` の「RLS ON だが policy なし」を INFO として報告している。現状の service-role-only 方針と整合するため、直ちに問題とは扱わない。

## Extensions

現在利用中として確認できた主なもの:

- `pg_cron`
- `pgcrypto`
- `pg_stat_statements`
- `uuid-ossp`
- `supabase_vault`
- `plpgsql`

## Cron

Job ID 1:

- schedule: `0 * * * *`（毎時）
- active: true
- 5つの Dashboard Materialized View を `CONCURRENTLY` refresh

## Migration History

Production の `supabase_migrations.schema_migrations` には 2026-09-15 以降の 21 件の migration 履歴が存在する。

この履歴は baseline 導入時に無視・破棄しない。既存 history と新しい Git 管理を整合させてから日常運用へ移行する。
