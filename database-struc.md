Now let me pull the core table list from the canonical Prisma schema, plus check `tenant-service-dev`'s separate schema and any triggers/procedures defined there.This calls for both a reference document (full table/procedure/trigger list) and a visual flow diagram, since you specifically asked for the "flow structure." Let me build both.Now the full reference document with every table, procedure, and trigger listed by name and grouped logically.Now let me build the final reference document.The diagram above shows the exact runtime call chain from `form_response` to `pro_score`, and the file lists every table, procedure, and trigger by name, grouped logically.

<img width="1364" height="1512" alt="image" src="https://github.com/user-attachments/assets/30183531-9137-4ff9-941d-810e04c37735" />

One thing worth flagging from building this: `auth-service-dev` has its own `schema.prisma` defining a mostly-duplicate table set (lowercase naming — `patient`, `physician`, `form_response`, `pro_score`, `lost_info`, etc.) separate from `common-library-dev`'s canonical schema, but with zero migrations applied. That's either an unused scaffold or a real risk if someone runs a migration against it not realizing it diverges from the schema everything else actually uses — worth a quick check with your lead.

---
## SPRI Platform — Database Objects Reference (Tables, Procedures, Triggers, Flow)

Source: `common-library-dev/prisma/schema.prisma` (canonical schema) and
`common-library-dev/prisma/migrations/20260623120000_pro_score_db_implementation_v11/migration.sql`
(the deployed stored procedures and triggers). `tenant-service-dev` and `auth-service-dev`
have their own separate schemas, called out at the end.

---

## 1. Core tables (common-library-dev / resource_spri schema)

Grouped by area. Table name shown as it appears in the actual database (snake_case),
Prisma model name in parentheses.

### Patient & clinical
- `patient` (Patient)
- `clinician` (Clinician)
- `clinician_joint` (ClinicianJoint)
- `clinical_exam` (ClinicalExam)
- `patient_note` (PatientNote)
- `patient_procedure` (PatientProcedure)
- `op_note` (OpNote)

### Facility & appointment
- `facility` (Facility)
- `appointment` (Appointment)
- `appointment_status_history` (AppointmentStatusHistory)
- `appointment_notes` (AppointmentNotes)
- `appointment_form_status` (AppointmentFormStatus)
- `appointment_form_option` (AppointmentFormOption)
- `appointment_form_template_selection` (AppointmentFormTemplateSelection)

### Surgery / procedure detail
- `ankle_surgery` (AnkleSurgery)
- `foot_surgery` (FootSurgery)
- `procedure_type` (ProcedureType)
- `surgical_visit` (SurgicalVisit)
- `surgical_visit_cpt_code` (SurgicalVisitCptCode)
- `surgical_visit_diagnosis` (SurgicalVisitDiagnosis)
- `surgical_visit_implant` (SurgicalVisitImplant)
- `surgical_visit_complication` (SurgicalVisitComplication)

### Surveys & forms
- `form_template` (FormTemplate)
- `form_template_questions` (FormTemplateQuestions)
- `form_response` (FormResponse) — **holds raw survey answers as JSON**
- `form_pdf_upload_log` (FormPdfUploadLog)
- `patient_survey` (PatientSurvey)
- `patient_survey_notification` (PatientSurveyNotification)
- `clinician_survey` (ClinicianSurvey)
- `automated_survey` (AutomatedSurvey)
- `automated_survey_appointment_type` (AutomatedSurveyAppointmentType)
- `automated_survey_criterion` (AutomatedSurveyCriterion)
- `automated_survey_run_log` (AutomatedSurveyRunLog)

### PRO scoring & LOST tracking
- `pro_score` (ProScore) — **destination table for calculated PRO scores**
- `lost_info` (LostInfo) — SOPMP-1669
- `lost_replacement_detail` (LostReplacementDetail)

### Notification & correspondence
- `notification_template` (NotificationTemplate)
- `correspondence` (Correspondence)
- `correspondence_custom_tag` (CorrespondenceCustomTag)

### System / audit
- `system_user` (SystemUser)
- `system_user_session` (SystemUserSession)
- `token_blacklist` (TokenBlacklist)
- `mfa_verification_code` (MfaVerificationCode)
- `hl7_audit` (Hl7Audit)
- `hst_audit` (HstAudit)
- `hst_error_log` (HstErrorLog)
- `qr_code_log` (QrCodeLog)
- `triggered_by_person` (TriggeredByPerson)
- `dicom_study_metadata` (DicomStudyMetadata)
- `otel_ingest_dedupe` (OtelIngestDedupe)

### Analytics
- `analytics_access_share`, `analytics_system_metrics`, `analytics_user_activity_monthly`,
  `analytics_user_joint_activity_monthly`

### Lookup tables (lk_*)
`lk_clinician_role`, `lk_clinician_category`, `lk_clinician_survey_status`,
`lk_patient_survey_status`, `lk_joint_type`, `lk_appointment_type`,
`lk_appointment_status_type`, `lk_laterality`, `lk_note_category`,
`lk_notification_type`, `lk_notification_delivery_status`, `lk_system_user_role`,
`lk_correspondence_group`, `lk_survey_type`, `lk_automated_delivery_method`,
`lk_appointment_review_status`, `lk_joint_form_status`, `lk_appt_form_selection_option`,
`lk_automated_schedule_mode`, `lk_appointment_category`, `lk_form_template_type`,
`lk_country`, `lk_state_province`, `lk_sex`
Plus joint-specific injury lookups: `foot_injury_type`, `toe_injury_type`

---

## 2. Stored procedures (PRO score engine)

All defined in one migration file: `20260623120000_pro_score_db_implementation_v11`.

### Helper / utility procedures (called by everything below)
| Procedure | Purpose |
|---|---|
| `spri_pro_norm` | Normalizes a raw text value (trim, case) before comparison |
| `spri_pro_json_text` | Core key lookup — searches a pipe-delimited list of candidate key names against a JSON object, returns first match |
| `spri_pro_num` | Same as above, but casts result to a number |
| `spri_pro_date` | Same as above, but parses result as a date (tries multiple date formats) |
| `spri_pro_sum_items` | Sums a group of related JSON keys (e.g. all WOMAC pain sub-questions) into one subtotal |
| `spri_pro_payload_json` | Normalizes `form_response.answers` into a flat, scoreable JSON object — handles legacy flat payload, FormsOrt `{"answers":{...}}` wrapper, and escaped JSON string shapes. **Does not currently handle FormSite's native `items`-array shape — see SOPMP-1609** |

### Score calculator procedures (one per outcome instrument)
`spri_pro_score_sf12`, `spri_pro_score_womac`, `spri_pro_score_lysholm`,
`spri_pro_score_faam`, `spri_pro_score_aofas`, `spri_pro_score_dash`,
`spri_pro_score_ases`, `spri_pro_score_ikdc`, `spri_pro_score_hos`,
`spri_pro_score_harris_vhs`, `spri_pro_score_sane`, `spri_pro_score_fadi`,
`spri_pro_score_spine_misc` — **13 procedures**, each takes the normalized JSON
and outputs one or more score columns via a fixed list of candidate legacy key names.

### Orchestration procedures
| Procedure | Purpose |
|---|---|
| `spri_validate_pro_score_calc_prereqs` | Checks the schema itself is ready (tables/columns exist) — returns rows only if something is missing |
| `spri_assert_pro_score_calc_prereqs` | Same check, but raises a SQL error instead of returning rows — called at the start of every calculation |
| `spri_pro_calculate_for_response` | The main dispatcher — resolves patient_id, exam_date, joint_type_id, laterality_id, calls `spri_pro_payload_json`, then calls all 13 score calculator procedures in sequence, clamps out-of-range values |
| `spri_select_pro_score_calculated` | Read-only wrapper — calls the dispatcher and returns the result as a row, without writing anything (used for manual testing) |
| `spri_apply_pro_score_for_form_response_silent` | Upserts into `pro_score`, no result set returned — **this is the version called by the two triggers**, since triggers cannot return result sets |
| `spri_apply_pro_score_for_form_response` | Same upsert, but does return a result set — used for manual/application-level calls, not by triggers |

### Deprecated / backward-compatible wrappers (kept for older callers)
`spri_select_pro_score_calculated_v8`, `spri_apply_pro_score_for_form_response_v9`,
`spri_apply_pro_score_for_form_response_v8`, `spri_validate_pro_score_calc_prereqs_v8`

### One-off legacy migration procedure (unrelated to the runtime flow above)
`migrate_pro_score_laterality_id` — from an earlier migration
(`20260207210000_pro_score_laterality`), self-drops after running once.

---

## 3. Triggers

| Trigger | Fires on | Calls |
|---|---|---|
| `trg_form_response_ai_pro_score` | `AFTER INSERT` on `form_response` | `spri_apply_pro_score_for_form_response_silent` |
| `trg_form_response_au_pro_score` | `AFTER UPDATE` on `form_response` | `spri_apply_pro_score_for_form_response_silent` |

No `DELETE` trigger exists on `form_response` by design (per code comment in the migration).

---

## 4. Runtime flow (see diagram above)

```
form_response (INSERT or UPDATE)
        |
        v
trg_form_response_ai_pro_score / trg_form_response_au_pro_score
        |
        v
spri_apply_pro_score_for_form_response_silent   (upsert wrapper, no result set)
        |
        v
spri_pro_calculate_for_response                  (main dispatcher)
        |-- spri_assert_pro_score_calc_prereqs    (schema sanity check)
        |-- spri_pro_payload_json                 (normalize answers JSON)
        |-- spri_pro_score_sf12 / womac / lysholm / faam / aofas / dash /
        |   ases / ikdc / hos / harris_vhs / sane / fadi / spine_misc
        v
pro_score table  (INSERT ... ON DUPLICATE KEY UPDATE, keyed on
                  patient_id + joint_type_id + exam_date + laterality_id)
```

**Known gap (SOPMP-1609):** `spri_pro_payload_json` does not recognize FormSite's native
answer shape (`{"items":[{"id":"92","value":"..."}]}`), so every `spri_pro_score_*`
procedure returns NULL for FormSite submissions specifically. See the SOPMP-1609 fix file
(`sopmp1609_formsite_items_mapping.sql`) for the proposed remediation.

---

## 5. Separate schemas (not part of the flow above)

### tenant-service-dev — own independent Prisma schema + 7 migrations
Tables: `time_zone`, `tenant`, `allowed_issuer`, `tenant_domain`, `tenant_config`,
`tenant_change_window`, `tenant_branding`, `integration_endpoint`, `tenant_note`,
`role`, `role_permission`.
No stored procedures or triggers found in this schema's migrations — pure Prisma-managed schema.

### auth-service-dev — own schema.prisma, but zero migrations applied
Defines a large, mostly duplicate table set (lowercase/snake_case naming: `patient`,
`physician`, `appointment`, `form_response`, `pro_score`, `lost_info`, etc.) — appears to be
either an older or a scaffold copy of the common-library schema. Worth confirming with the
team whether this is actively used or a leftover template, since having two divergent
schemas pointing at overlapping table names is a risk if both are ever migrated independently.
