# Slot-Data-Analysis-DB

Supabase / PostgreSQL database definitions for the Slot-Data-Analysis system.

## Purpose

This repository is the **single source of truth** for database schema changes shared by:

- `Slot-Data-Analysis`
- `Slot-Data-Analysis-web`
- future applications that use the same Supabase project

Application repositories may read/write data through their permitted roles, but database structure changes should be managed here.

## Managed here

- schema migrations
- tables / constraints
- indexes
- views
- materialized views
- RPC / PostgreSQL functions
- RLS policies
- grants / privileges
- maintenance SQL
- Supabase database configuration

## Repository structure

```text
.
├─ supabase/
│  └─ migrations/          # authoritative migration history
├─ sql/
│  ├─ views/               # reference SQL for views
│  ├─ materialized_views/  # reference SQL for materialized views
│  ├─ rpc/                 # reference SQL for RPC/functions
│  ├─ indexes/             # reference SQL for indexes
│  ├─ policies/            # RLS / grants reference SQL
│  └─ maintenance/         # manual operational / diagnostic SQL
└─ docs/
   └─ database-overview.md
```

## Change workflow

1. Create a feature/fix branch.
2. Add a new migration under `supabase/migrations/`.
3. Review the SQL in a pull request.
4. Test against a Supabase Preview Branch or staging environment.
5. Merge after verification.
6. Apply/deploy the migration to Production.

Do not edit already-applied migration files. Prefer a new migration.

## Existing production database

The current Production database predates this repository.

Before importing existing SQL from other repositories, establish a **baseline** from the current Production schema. Existing SQL must not be blindly re-run as migrations, because objects may already exist.

## Security

Never commit secrets, including:

- `.env`
- database passwords
- `SUPABASE_SERVICE_ROLE_KEY`
- Supabase secret keys
- full database connection strings containing credentials

Application credentials belong in each application's environment/secret store, not in this repository.

## Supabase CLI

Generate the local Supabase CLI configuration with:

```bash
supabase init
```

The generated `supabase/config.toml` can then be reviewed and committed separately.
