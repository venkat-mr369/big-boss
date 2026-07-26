Good question — let's zoom out from all the SQL files and look at the big picture.

## The old system vs. the new system

There are basically **two databases** in play here:

1. **The old/legacy database** — called `resource_spri` (you saw this in `bigr-app.txt`: `select count(*) from patient;` etc., and tables like `tbl_pat_demo`, `tbl_hip_surg`). This is the research institute's old system — probably built years ago, with old-style table names (`tbl_` prefixes, all lowercase legacy naming).

2. **The new SPRI platform** — the modern system being built now: `patient-service`, `survey-service`, `spri-client` (the Angular website), all talking to a new, cleaner database (tables like `patient`, `lost_info`, `pro_score`, `form_response`).

The whole `temp-legacy-data-import-dev` project's job is to **move data from the old database into the new one**. That's what all those `part1_core` → `part35_core` SQL scripts are doing — reading old messy tables and loading cleaned-up versions into the new schema.

## What "staging" means here

You'll see two names floating around: `resource_spri` and `resource_spri_staging`. Think of it like this:

- **Production (prod)** = the real, live database that doctors/patients actually use.
- **Staging** = a copy/practice environment that looks just like production, but nobody's real workflow depends on it. It's where you test "did my migration script work correctly?" *before* running it against production.

So `resource_spri_staging` is just "the staging copy of the SPRI database" — a safe sandbox to run and re-run the migration scripts, check the row counts, fix bugs, and only once it's verified, run the same script against the real production database (`resource_spri`).

That's also why you saw both `manifest.yaml` (staging config) and `manifest-prod.yaml` (production config) in the migration folder — same scripts, pointed at two different targets.

## What Prisma is

**Prisma** is a tool that sits between your application code and the database. Instead of a developer writing raw SQL like `CREATE TABLE patient (...)` by hand, they write a simple description of the tables in one file called `schema.prisma`, and Prisma:

1. **Generates the actual database tables** for you (via "migrations" — that's what all those `20260129145239_initial_database...` folders are: each one is a small, dated change to the schema, like "add a column" or "create a table").
2. **Generates code** the NestJS services (`patient-service`, `survey-service`) use to read/write data safely, without writing raw SQL everywhere.
3. Keeps a **history** of every schema change, in order, so anyone can see exactly how the database evolved over time — like a version control system, but for your database structure instead of your code.

So in `common-library-dev/prisma/`:
- `schema.prisma` = the current, final blueprint of every table (patient, lost_info, form_response, etc.)
- `migrations/` = the step-by-step history of how that blueprint was built, one small change at a time
- `seeds/` = scripts to fill tables with starter/reference data (like a list of countries, sexes, joint types) so a fresh database isn't empty

## Putting it all together

Here's the flow, in order:

```
OLD SYSTEM (resource_spri)              NEW SYSTEM (SPRI Platform)
tbl_pat_demo, tbl_hip_surg, etc.         patient, lost_info, form_response, etc.
        |                                        ^
        |   migration/sql/part1..35_core          |
        |   (staging → transform → load)          |
        └──────────────────────────────────────────┘
                       |
              tested first on:
        resource_spri_staging (safe copy)
                       |
              then run against:
              resource_spri (real prod data)

Meanwhile, Prisma (in common-library-dev)
defines what the NEW tables should look like,
and the NestJS services (patient-service,
survey-service) use Prisma to actually read/write
that data once the migration is done.
```

So when you're fixing a Jira ticket like SOPMP-1669 (the `lost_info` gap), you're really debugging **one step of this move**: the `part32_core` script that's supposed to take old `tbl_pat_demo` data and correctly fill in the new `lost_info` table — and something in that transformation is dropping rows or leaving fields empty.

Want me to walk through one full example end-to-end — say, trace exactly how one patient's LOST data moves from the old `tbl_pat_demo` row all the way to a `lost_info` row in Prisma's schema — so you can see the whole pipeline in action on real column names?