# Database Folder Structure — SPRI Platform

Focused on the DB-relevant folders across all repos: Prisma schema/migrations, raw SQL scripts, ERD diagrams, and the legacy data migration pipeline. This is what you'll touch to fix Jira DB tickets (e.g. `SOPMP-1669` — LOST migration data gap).

## Where things live

| Location | What it is |
|---|---|
| `common-library-dev/prisma/` | **Source of truth schema** — Prisma models, all migrations, seed data. Shared by patient-service & survey-service. |
| `patient-service-dev/database/` | ERD `.drawio` diagrams + one-off staging SQL script + `init.sql` |
| `survey-service-dev/Database/` | ERD `.drawio` diagrams only (survey/manual & automated) |
| `temp-legacy-data-import-dev/migration/` | Python-orchestrated legacy → SPRI ETL: 35 "parts", each with stage-table-create → CSV-load → stage-to-destination SQL |

---

## 1) common-library-dev/prisma  (Prisma ORM — canonical schema)

```
prisma/
  schema.prisma              <- the canonical DB schema (all models)
  seed.ts
  ts-node-resolver.js
  SEED_AUDIT.md
  migrations/                 <- 93 migrations, chronological, e.g.:
    20260129145239_initial_database_27_01/
    20260507130000_lost_info_lost_date_nullable/        <- relevant to LOST/lost_info tickets
    20260714191219_expand_lost_info_reason_to_1000/     <- relevant to LOST/lost_info tickets
    ... (93 total, each folder = migration.sql + metadata)
    migration_lock.toml
    README.md
  dumps/
    staging-dump.sql.gz        <- staging DB dump (4MB)
  seeds/
    index.ts, README.md, list-tables.js
    <table-name>/data.json + seed.ts    (~60 lookup/reference-table seed folders, e.g. lk-sex, lk-country, facility, patient, clinician, form-template, automated-survey, lost-info, lost-replacement-detail, pro-score ...)
    backfill-geo/               <- one-off backfill scripts + country-map.json / state-map.json
    tenants/<tenant-uuid>/      <- per-tenant seed overrides (2 tenants)
    utils/                      <- seed-loader, tenant-resolver, id-generator, qa-protection, etc.
```

**Migrations directly touching `lost_info`** (relevant to SOPMP-1669):
- `20260507130000_lost_info_lost_date_nullable`
- `20260714191219_expand_lost_info_reason_to_1000`
- seed folder: `prisma/seeds/lost-info/seed.ts`, `prisma/seeds/lost-replacement-detail/seed.ts`

---

## 2) patient-service-dev/database

```
database/
  init.sql                                  <- initial local DB setup
  diagrams/
    ERD-Resource-Database.drawio
    ERD-clinical-data.drawio
    ERD-patient-design.drawio
    ERD-patient-forms-design.drawio
    ERD-reception-design.drawio
    Resource_Database_ERD.svg
    appointment-detail.drawio
  scripts/
    Part_1_Stage_Data_Loading_to_Destination_Tables.sql
```

---

## 3) survey-service-dev/Database

```
Database/
  diagrams/
    patient-survey-manual.drawio
    patient_survey_automated.drawio
```
(plus, at the project root: `scripts/seed-appointment-types.sql`, `scripts/migrate-tenant.js`)


## 4) temp-legacy-data-import-dev/migration  (Python ETL — legacy → SPRI migration)

```
migration/
  DBA_INSTRUCTIONS.md          <- read this first, written for DBAs
  DBA_LLM_PROMPT.md
  Dockerfile
  manifest.yaml                 <- staging config
  manifest-prod.yaml             <- prod config
  status.yaml                    <- migration run status/progress tracking
  migrate.py                     <- main orchestrator script
  requirements.txt
  test_local.sh
  sql/
    part1_core/part_1_drop_temp_objects.sql
    part1_core/part_1_loading_csv_to_stage.sql
    part1_core/part_1_loading_from_stage_to_destination.sql
    part1_core/part_1_stage_table_creation.sql
    part1_core/part_1_test_loading_csv_to_stage.sql
    part2_core/part_2_create_stage_table.sql
    part2_core/part_2_csv_to_stage_load.sql
    part2_core/part_2_drop_temp_objects.sql
    part2_core/part_2_from_stage_to_destination.sql
    part3_core/part_3_create_stage_tables.sql
    part3_core/part_3_csv_to_stage_table.sql
    part3_core/part_3_drop_temp_objects.sql
    part3_core/part_3_stage_to_destination.sql
    part4_core/.gitkeep
    part4_core/part_4_create_stage_tables.sql
    part4_core/part_4_csv_to_stage.sql
    part4_core/part_4_drop_temp_tables.sql
    part4_core/part_4_stage_to_destination.sql
    part5_core/part_5_create_stage_table.sql
    part5_core/part_5_create_stage_table_fixed_rds_no_seed.sql
    part5_core/part_5_csv_to_stage_load.sql
    part5_core/part_5_csv_to_stage_load_fixed_rds_no_seed.sql
    part5_core/part_5_drop_temp_objects.sql
    part5_core/part_5_drop_temp_objects_fixed_rds_no_seed.sql
    part5_core/part_5_from_stage_to_destination.sql
    part5_core/part_5_from_stage_to_destination_fixed_rds_no_seed.sql
    part5_core/part_16_drop_temp_objects.sql
    part6_core/part_6_create_stage_tables.sql
    part6_core/part_6_csv_to_stage.sql
    part6_core/part_6_drop_temp_objects.sql
    part6_core/part_6_stage_to_destination.sql
    part7_core/part_7_create_stage_table.sql
    part7_core/part_7_csv_to_stage_load.sql
    part7_core/part_7_drop_temp_objects.sql
    part7_core/part_7_from_stage_to_destination.sql
    part8_core/part_8_create_stage_table.sql
    part8_core/part_8_csv_to_stage_load.sql
    part8_core/part_8_drop_temp_objects.sql
    part8_core/part_8_from_stage_to_destination.sql
    part9_core/part_9_create_stage_table.sql
    part9_core/part_9_csv_to_stage_load.sql
    part9_core/part_9_drop_temp_objects.sql
    part9_core/part_09_from_stage_to_destination.sql
    part9_core/part_9_from_stage_to_destination.sql
    part10_core/part_10_create_stage_table.sql
    part10_core/part_10_csv_to_stage_load.sql
    part10_core/part_10_drop_temp_objects.sql
    part10_core/part_10_from_stage_to_destination.sql
    part11_core/part_11_create_stage_table.sql
    part11_core/part_11_csv_to_stage_load.sql
    part11_core/part_11_drop_temp_objects.sql
    part11_core/part_11_from_stage_to_destination.sql
    part12_core/part_12_create_stage_table.sql
    part12_core/part_12_csv_to_stage_load.sql
    part12_core/part_12_drop_temp_objects.sql
    part12_core/part_12_from_stage_to_destination.sql
    part13_core/part_13_create_stage_table.sql
    part13_core/part_13_csv_to_stage_load.sql
    part13_core/part_13_drop_temp_objects.sql
    part13_core/part_13_from_stage_to_destination.sql
    part14_core/part_14_create_stage_table.sql
    part14_core/part_14_csv_to_stage_load.sql
    part14_core/part_14_drop_temp_objects.sql
    part14_core/part_14_from_stage_to_destination.sql
    part15_core/part_12_drop_temp_objects.sql
    part15_core/part_15_create_stage_table.sql
    part15_core/part_15_csv_to_stage_load.sql
    part15_core/part_15_drop_temp_objects.sql
    part15_core/part_15_from_stage_to_destination.sql
    part16_core/part_16_create_stage_table.sql
    part16_core/part_16_csv_to_stage_load.sql
    part16_core/part_16_drop_temp_objects.sql
    part16_core/part_16_from_stage_to_destination.sql
    part17_core/part_17_create_stage_table.sql
    part17_core/part_17_csv_to_stage_load.sql
    part17_core/part_17_drop_temp_objects.sql
    part17_core/part_17_from_stage_to_destination.sql
    part18_core/part_18_create_stage_table.sql
    part18_core/part_18_csv_to_stage_load.sql
    part18_core/part_18_drop_temp_objects.sql
    part18_core/part_18_from_stage_to_destination.sql
    part19_core/part_16_from_stage_to_destination.sql
    part19_core/part_19_create_stage_table.sql
    part19_core/part_19_csv_to_stage_load.sql
    part19_core/part_19_drop_temp_objects.sql
    part19_core/part_19_from_stage_to_destination.sql
    part20_core/part_20_create_stage_table.sql
    part20_core/part_20_csv_to_stage_load.sql
    part20_core/part_20_drop_temp_objects.sql
    part20_core/part_20_from_stage_to_destination.sql
    part21_core/part_21_create_stage_table.sql
    part21_core/part_21_csv_to_stage_load.sql
    part21_core/part_21_drop_temp_objects.sql
    part21_core/part_21_from_stage_to_destination.sql
    part22_core/part_22_create_stage_table.sql
    part22_core/part_22_csv_to_stage_load.sql
    part22_core/part_22_drop_temp_objects.sql
    part22_core/part_22_from_stage_to_destination.sql
    part23_core/part_23_create_stage_table.sql
    part23_core/part_23_csv_to_stage_load.sql
    part23_core/part_23_drop_temp_objects.sql
    part23_core/part_23_from_stage_to_destination.sql
    part23_core/part_23_from_stage_to_destination_b4fixed_audit.sql
    part24_core/part_24_create_stage_table.sql
    part24_core/part_24_csv_to_stage_load.sql
    part24_core/part_24_drop_temp_objects.sql
    part24_core/part_24_from_stage_to_destination.sql
    part25_core/Part_1_25_Apply_Grant_MJP_Appointment_Type_Changes_1.sql
    part25_core/Part_1_25_Apply_tbl_pat_demo_audit_user_delta_1.sql
    part25_core/part_1_25_Apply_1281_Facility_Defect_Delta_All.sql
    part25_core/part_1_25_unknown_lk_joint_type_to_false.sql
    part25_core/part_2_25_Apply_tbl_web_form_email_user_audit_source_mode_delta.sql
    part25_core/part_25_drop_all_temp_objects.sql
    part25_core/part_25_drop_temp_objects.sql
    part25_core/part_25_fix_Invalid_Form_ID_values.sql
    part26_core/part26_update_form_template.sql
    part26_core/part26_update_form_template_set_inactive_with_urls.sql
    part26_core/part26_update_form_template_urls_and_names.sql
    part26_core/part_1_26_appointment_status_notes_update.sql
    part26_core/part_2_26_Apply_patient_survey_appointment_form_dedupe_delta.sql
    part27_core/part27_form_template_inactive_patch.sql
    part28_core/part_2_28_create_stage_table.sql
    part28_core/part_2_28_csv_to_stage_load.sql
    part28_core/part_2_28_restore_appointments_backfill_patient_survey_and_dedupe.sql
    part28_core/part_28_01_create_stage_table.sql
    part28_core/part_28_02_csv_to_stage_load.sql
    part28_core/part_28_03_QA_precheck_before_restore_and_dedupe.sql
    part28_core/part_28_04_restore_appointments_backfill_patient_survey.sql
    part28_core/part_28_05_QA_precheck_remaining_null_appointment_seed.sql
    part28_core/part_28_06_seed_remaining_null_patient_survey_appointments.sql
    part28_core/part_28_07_fix_exact_patient_survey_duplicates.sql
    part28_core/part_28_08_deactivate_all_patient_survey.sql
    part28_core/part_28_09_sopmp_1281_set_is_active_0.sql
    part29_core/part29_fix_hip_arthroplasty_templates.sql
    part29_core/part29_update_8_records_imaging_mri.sql
    part30_core/part30_1_create_rpt_hip_table_form_response.sql
    part30_core/part30_2_clinician_staff_surgeon.sql
    part30_core/part30_3_prod_archive_physician_inactive_fix_final.sql
    part30_core/part30_4_self_assoc_surgeon_role_fix.sql
    part30_core/part30_5_Deactivate_is_active_flag_Imaging_Emergency.sql
    part30_core/part30_6_Ultra_Sound_Archive.sql
    part30_core/part30_clinician_staff_surgeon.sql
    part30_core/part30_create_rpt_hip_table_form_response.sql
    part30_core/part_30_7_production_uat_appointment_status_mapping_fix.sql
    part31_core/part31_fix_all_doctor_roles_to_surgeon.sql
    part32_core/part32_populate_lost_info_from_tbl_pat_demo.sql
    part32_populate_lost_info_from_tbl_pat_demo.sql
    part33_core/part33_populate_patient_procedure_from_surg_csvs.sql
    part33_core/part33_pro_score_load_from_form_response.sql
    part34_core/part34_patient_ssn_backfill_from_tbl_pat_demo.sql
    part35_core/part35_mjp_baseline_232records_with_apptype_team.sql
```

**testdata/** (sample CSVs matching the legacy source tables): tbl_pat_demo.csv, tbl_patient_appointment.csv, tbl_physician.csv
