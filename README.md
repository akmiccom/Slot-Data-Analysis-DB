# Slot-Data-Analysis-DB

Slot-Data-Analysis システムで使用する Supabase / PostgreSQL のデータベース定義を管理するリポジトリです。

## 目的

このリポジトリを、以下のシステムで共有する **データベース構造変更の唯一の正本（Single Source of Truth）** とします。

- `Slot-Data-Analysis`
- `Slot-Data-Analysis-web`
- 今後、同じ Supabase プロジェクトを利用するアプリケーション

各アプリケーションのリポジトリは、許可された権限の範囲でデータの参照・更新を行いますが、テーブル構造や関数、RLS などのデータベース構造変更はこのリポジトリで管理します。

## このリポジトリで管理するもの

- スキーマ migration
- テーブル / 制約
- インデックス
- View
- Materialized View
- RPC / PostgreSQL Function
- RLS Policy
- GRANT / 権限設定
- メンテナンス用 SQL
- Supabase のデータベース設定

## リポジトリ構成

```text
.
├─ supabase/
│  └─ migrations/          # 正式な migration 履歴
├─ sql/
│  ├─ views/               # View 定義の参照用 SQL
│  ├─ materialized_views/  # Materialized View 定義の参照用 SQL
│  ├─ rpc/                 # RPC / Function 定義の参照用 SQL
│  ├─ indexes/             # Index 定義の参照用 SQL
│  ├─ policies/            # RLS / GRANT 関連の参照用 SQL
│  └─ maintenance/         # 手動運用・確認・診断用 SQL
└─ docs/
   └─ database-overview.md
```

## 変更時の基本フロー

1. feature / fix ブランチを作成する
2. `supabase/migrations/` に新しい migration を追加する
3. Pull Request で SQL をレビューする
4. Supabase Preview Branch または staging 環境でテストする
5. 問題がなければ merge する
6. Production に migration を適用する

すでに Production に適用済みの migration ファイルは原則として編集しません。変更が必要な場合は、新しい migration を追加します。

## 既存 Production DB について

現在の Production DB は、このリポジトリを作成する前から運用されています。

そのため、他のリポジトリに存在する既存 SQL をそのまま migration として再実行してはいけません。すでに同名のテーブル、View、RPC、Index、Policy などが存在する可能性があるためです。

まず、現在の Production DB の状態を基準として **baseline** を作成し、それ以降の変更をこのリポジトリの migration として管理します。

## セキュリティ

以下のような機密情報は、このリポジトリにコミットしません。

- `.env`
- Database Password
- `SUPABASE_SERVICE_ROLE_KEY`
- Supabase Secret Key
- パスワードを含む Database Connection String

各アプリケーションが使用する接続情報や API Key は、このリポジトリではなく、それぞれのアプリケーション側の環境変数や Secret 管理機能で管理します。

## Supabase CLI

ローカルで Supabase CLI の初期設定を作成する場合は、以下を実行します。

```bash
supabase init
```

生成された `supabase/config.toml` は、内容を確認したうえでこのリポジトリにコミットします。
