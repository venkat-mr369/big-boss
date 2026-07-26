### Step 2 – Database Architecture

Think of the database as a hospital filing system.

Every table stores a specific type of information.

```text
                    patient
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
 appointment    patient_survey   patient_note
        │
        ▼
 form_response
        │
        ▼
 pro_score
```

Let's understand each table.

---

# 1. patient (TABLE)

This is the master table.

Everything starts here.

Example:

| patient_id | name | MRN     | DOB        |
| ---------- | ---- | ------- | ---------- |
| P1001      | John | MR12345 | 1990-05-20 |

Purpose:

* Patient demographics
* Medical Record Number (MRN)
* Contact information

Relationship:

```text
patient
   │
   ├── appointments
   ├── surveys
   ├── notes
   └── scores
```

---

# 2. appointment (TABLE)

One patient can have many appointments.

Example:

| appointment_id | patient_id | doctor   | date       |
| -------------- | ---------- | -------- | ---------- |
| A101           | P1001      | Dr Smith | 2026-07-20 |

Purpose:

* Visit details
* Doctor
* Clinic
* Joint type

---

# 3. patient_survey (TABLE)

This table tells us **which survey is assigned**.

Example:

| survey_id | patient_id | survey_type    | status |
| --------- | ---------- | -------------- | ------ |
| S201      | P1001      | Knee Follow-up | Sent   |

This does **not** store answers.

It only stores assignment information.

---

# 4. form_response (TABLE)

This is one of the most important tables.

When the patient submits the survey,

the answers are saved here.

Example:

| id    | patient_id | answers |
| ----- | ---------- | ------- |
| FR101 | P1001      | JSON    |

Example JSON:

```json
{
  "Pain":8,
  "Walking":"Yes",
  "Sports":"No"
}
```

Or (FormSite format):

```json
{
  "items":[
      {
          "id":"92",
          "value":"8"
      }
  ]
}
```

This table stores **raw data only**.

No calculations happen here.

---

# 5. Trigger

Database Object

```
TRIGGER
```

Example:

```
trg_form_response_ai_pro_score
```

Meaning:

```
After Insert
```

Flow:

```text
INSERT into form_response

↓

Trigger executes automatically

↓

Stored Procedure starts
```

Nobody manually calls the trigger.

---

# 6. Stored Procedures (SP)

These contain the business logic.

Example:

```
spri_apply_pro_score_for_form_response
```

↓

calls

```
spri_pro_calculate_for_response
```

↓

calls

```
spri_pro_payload_json
```

↓

calculates scores

↓

stores result.

---

# 7. pro_score (TABLE)

This stores calculated scores.

Example

| patient_id | Harris | IKDC | WOMAC |
| ---------- | ------ | ---- | ----- |
| P1001      | 92     | 87   | 81    |

Doctors read this table.

They do **not** read `form_response`.

---

# Data Flow

```text
Patient
    │
    ▼
patient
(TABLE)

    │
    ▼
appointment
(TABLE)

    │
    ▼
patient_survey
(TABLE)

    │
    ▼
Patient submits survey

    │
    ▼
form_response
(TABLE)

    │
    ▼
AFTER INSERT Trigger
(TRIGGER)

    │
    ▼
Stored Procedures
(SP)

    │
    ▼
pro_score
(TABLE)

    │
    ▼
Doctor Dashboard
```

---

# Which objects are important for a DBA?

| Object                                   | Type             | Responsibility                            |
| ---------------------------------------- | ---------------- | ----------------------------------------- |
| `patient`                                | Table            | Patient master data                       |
| `appointment`                            | Table            | Visit details                             |
| `patient_survey`                         | Table            | Survey assignment                         |
| `form_response`                          | Table            | Raw survey answers                        |
| `pro_score`                              | Table            | Calculated medical scores                 |
| `trg_form_response_ai_pro_score`         | Trigger          | Starts score calculation automatically    |
| `spri_apply_pro_score_for_form_response` | Stored Procedure | Creates/updates `pro_score`               |
| `spri_pro_calculate_for_response`        | Stored Procedure | Computes the scores                       |
| `spri_pro_payload_json`                  | Stored Procedure | Normalizes the survey JSON before scoring |

---

# Real Example: SOPMP-1609

Now you can understand where the issue happened.

```text
Patient
      │
      ▼
form_response
(TABLE)
      │
      ▼
Trigger
      │
      ▼
Stored Procedure
      │
      ▼
❌ JSON format mismatch
      │
      ▼
Scores became NULL
      │
      ▼
pro_score
(TABLE)
```

The survey reached the database successfully, but the stored procedure could not understand the new FormSite JSON structure. As a result, the score calculation did not populate the score columns correctly.

---

## Before moving to Step 3

As a database developer, remember this simple rule:

* **Tables** store data.
* **Triggers** automatically react to data changes.
* **Stored Procedures** implement business logic.
* **APIs** call the database.
* **UI** displays the results.

In **Step 3**, we'll go deeper into the **Stored Procedure call flow**. We'll trace exactly which stored procedure calls which other procedure, what parameters are passed, and how a single survey submission travels through the database until a row is inserted into `pro_score`. This is the key to understanding tickets like SOPMP-1609.
