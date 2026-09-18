# Production Baseline 作成方針

作成日: 2026-09-18

## 結論

現在の Production DB はすでに運用中で、さらに Supabase 側には 21 件の migration history が存在する。

そのため、既存 SQL を `supabase/migrations/` に手作業で並べ直して再適用する方法は採用しない。

**Production の現行 schema を正として Supabase CLI で取得し、local / remote migration history を整合させる** 方針とする。

## 重要な前提

Production では以下がすでに存在する。

- 5 tables
- 約 206 万行の `results`
- 5 materialized views
- 13 views
- 4 public functions（うち RPC overload を含む）
- RLS / grants
- indexes
- pg_cron job
- 2026-09-15 以降の 21 migration records

このため、過去の SQL をそのまま再実行するのは危険。

## 実施手順

### 1. この Repo をローカルへ clone

```bash
git clone https://github.com/akmiccom/Slot-Data-Analysis-DB.git
cd Slot-Data-Analysis-DB
```

### 2. Supabase CLI 初期化

```bash
supabase init
```

すでに `supabase/` が存在するため、生成物を確認してから commit する。

### 3. Supabase CLI にログイン

```bash
supabase login
```

### 4. Production Project と link

```bash
supabase link --project-ref lpabzfnxlxljzcaofxkm
```

Database Password はローカル入力のみとし、GitHub へ保存しない。

### 5. まず migration history を確認

```bash
supabase migration list
```

**ここでは repair / push / reset をまだ実行しない。**

Production に既存 migration history があるため、local と remote の差を確認してから次へ進む。

### 6. Production schema を pull

migration history の状態を確認したうえで、

```bash
supabase db pull
```

を使用して remote schema を migration として取得する。

Supabase 公式では、既存 Project を local workflow へ移す際の baseline として `db pull` を利用する。

### 7. 生成 SQL をレビュー

特に確認する項目:

- Table / PK / FK / CHECK
- Sequence
- View / Materialized View
- Index
- Functions / RPC
- RLS / Policies
- Grants
- Extensions
- 不要な `DROP EXTENSION` 等が生成されていないか

### 8. pg_cron を別途確認

`db pull` だけで cron job の運用状態まで完全再現できる前提にはしない。

現在確認済みの Job ID 1:

```text
0 * * * *
```

5つの Dashboard MV を `REFRESH MATERIALIZED VIEW CONCURRENTLY` している。

cron 定義は baseline review 時に明示的に管理対象へ含める。

### 9. Local 再現テスト

Production へ push する前に local で:

```bash
supabase start
supabase db reset
```

を実行し、migration だけで schema を再現できることを確認する。

### 10. PR で baseline を merge

確認後:

- `supabase/config.toml`
- `supabase/migrations/*_remote_schema.sql`
- 必要な補助 migration / cron 定義
- baseline レビュー結果

を PR にして main へ merge。

## Production に対して実行しないこと

baseline 完了までは以下を避ける。

```bash
supabase db push
supabase db reset --linked
supabase migration repair ...
```

特に `migration repair` は remote migration history を変更するため、差分を確認して必要性が確定するまで実行しない。

## baseline 完了後の運用

以後の DB 変更は:

```text
feature/fix branch
    ↓
supabase/migrations/<timestamp>_<name>.sql
    ↓
local / Preview Branch で確認
    ↓
Pull Request
    ↓
main
    ↓
Production
```

とする。

## 既存 App Repo の SQL

`Slot-Data-Analysis` および `Slot-Data-Analysis-web` にある既存 SQL は、baseline 完了後にこの Repo と比較する。

扱いは次の3分類:

1. Production と同じ → 参照元を DB Repo へ寄せる
2. Production より古い → 廃止候補
3. Production にまだ無い → 新規 migration 候補

既存 SQL を無条件で `supabase/migrations/` にコピーしない。
