Excellent. From now on we'll think like a **Database Architect** rather than just a SQL developer.

Up to now you've learned:

* Step 1 → Business
* Step 2 → Database Overview
* Step 3 → Stored Procedure Flow
* Step 4 → End-to-End Application Flow
* Step 5 → Repository Architecture
* Step 6 → Database Objects

Now we'll connect everything.

---

# Step 7 – Database Schema & Relationships

A database is not a collection of independent tables.

It is a **network of related tables**.

Think of it as a family tree.

```text
Father
   │
   ├── Son
   ├── Daughter
   └── Grandchildren
```

Similarly,

```text
patient
   │
   ├── appointment
   ├── patient_survey
   ├── form_response
   └── pro_score
```

Everything revolves around the patient.

---

# Database ER Diagram (High Level)

```text
                           patient
                    (Master Table)
                          │
        ┌─────────────────┼──────────────────┐
        │                 │                  │
        ▼                 ▼                  ▼
 appointment      patient_survey        patient_note
        │                 │
        │                 ▼
        │          survey_definition
        │
        ▼
 form_response
        │
        ▼
 pro_score
```

Think of this as the backbone of the application.

---

# 1. patient (Master Table)

This is the center of the database.

```text
patient
```

Example

| patient_id | Name | MRN   |
| ---------- | ---- | ----- |
| 1001       | John | MR001 |

Every other table ultimately refers to this patient.

---

## Relationships

```text
patient

↓

appointment

↓

patient_survey

↓

form_response

↓

pro_score
```

---

# Cardinality

One patient

↓

Many appointments

```text
Patient

↓

Appointment 1

Appointment 2

Appointment 3
```

Database notation

```text
1 ---- N
```

---

# Patient → Appointment

```text
patient
---------------------
patient_id (PK)

appointment
---------------------
appointment_id (PK)

patient_id (FK)
```

Relationship

```text
One Patient

↓

Many Appointments
```

---

# Patient → Surveys

One patient may receive multiple surveys.

Example

```text
John

↓

Pre Surgery Survey

↓

6 Weeks Survey

↓

3 Months Survey

↓

1 Year Survey
```

Database

```text
patient

↓

patient_survey
```

---

# Patient → Survey Response

Every assigned survey may produce one response.

```text
patient

↓

patient_survey

↓

form_response
```

Example

```text
Survey Assigned

↓

Patient Submitted

↓

Response Saved
```

---

# Patient → PRO Score

Each completed survey generates one or more scores.

```text
patient

↓

form_response

↓

pro_score
```

Example

```text
Patient

↓

Survey

↓

Harris Score

↓

IKDC Score

↓

WOMAC Score
```

---

# One-to-Many Relationships

The most common relationship.

Example

```text
Patient

↓

Appointment

Appointment

Appointment

Appointment
```

Database

```text
patient

1

↓

N

appointment
```

---

# One-to-One Relationship

Sometimes

One record

↓

One record

Example

```text
Patient

↓

Patient Profile
```

```text
1

↓

1
```

---

# Many-to-Many

Imagine

Doctors

↓

Patients

One doctor sees many patients.

One patient visits many doctors.

```text
Doctor

⇄

Patient
```

Database cannot store directly.

Need bridge table.

```text
doctor

↓

doctor_patient

↓

patient
```

---

# Why Foreign Keys Exist

Suppose appointment table contains

```text
patient_id = 5000
```

But patient

5000

doesn't exist.

Database becomes inconsistent.

Foreign Key prevents this.

```text
patient

1001

↓

appointment

patient_id=1001

✓ Valid
```

---

# Normalized Design

Instead of storing

```text
John

MR001

John

MR001

John

MR001
```

inside every table,

store only

```text
patient_id
```

Example

Patient Table

| patient_id | Name |
| ---------- | ---- |
| 1001       | John |

Appointment

| appointment | patient_id |
| ----------- | ---------- |
| 2001        | 1001       |

Survey

| survey | patient_id |
| ------ | ---------- |
| 3001   | 1001       |

No duplicate data.

---

# Complete Patient Journey

Let's follow John.

---

### Step 1

Patient Created

```text
patient
```

| patient_id | Name |
| ---------- | ---- |
| 1001       | John |

---

### Step 2

Appointment

```text
appointment
```

| appointment | patient_id |
| ----------- | ---------- |
| 2001        | 1001       |

---

### Step 3

Survey Assigned

```text
patient_survey
```

| survey | patient_id |
| ------ | ---------- |
| 3001   | 1001       |

---

### Step 4

Patient Submits

```text
form_response
```

| response | patient_id |
| -------- | ---------- |
| 4001     | 1001       |

---

### Step 5

Score Created

```text
pro_score
```

| score_id | patient_id |
| -------- | ---------- |
| 5001     | 1001       |

---

Everything is connected through

```text
patient_id
```

---

# Real Database Flow

```text
patient
   │
   ▼
appointment
   │
   ▼
patient_survey
   │
   ▼
form_response
   │
   ▼
pro_score
```

Notice something.

Each table stores **different information**.

No table stores everything.

---

# Why Split into Multiple Tables?

Suppose everything was stored in one table.

```text
Patient

Appointment

Survey

Doctor

Hospital

Score

Address

Phone

Pain

Walking

Sports

...
```

Imagine

100 columns

Millions of rows.

Problems

❌ Duplicate data

❌ Slow updates

❌ Large storage

❌ Difficult maintenance

Instead

Split logically.

```text
Patient

Appointment

Survey

Score
```

Each table has one responsibility.

---

# Primary Keys

Every table has its own identity.

```text
patient

patient_id
```

```text
appointment

appointment_id
```

```text
form_response

response_id
```

```text
pro_score

score_id
```

Primary Keys are unique within their own table.

---

# Foreign Keys

Tables connect through foreign keys.

```text
appointment.patient_id

↓

patient.patient_id
```

```text
form_response.patient_id

↓

patient.patient_id
```

```text
pro_score.patient_id

↓

patient.patient_id
```

---

# Complete Relationship Diagram

```text
                      patient
                  PK: patient_id
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
appointment       patient_survey      patient_note
PK appointment_id PK survey_id        PK note_id
FK patient_id     FK patient_id       FK patient_id
      │                  │
      │                  ▼
      │          survey_definition
      │
      ▼
form_response
PK response_id
FK patient_id
FK survey_id
      │
      ▼
pro_score
PK score_id
FK patient_id
FK response_id
```

---

# Think Like an Architect

When someone says:

> "Patient score is missing."

A Database Architect doesn't immediately open the `pro_score` table.

They mentally trace the data flow:

```text
Was Patient Created?

↓

Was Appointment Created?

↓

Was Survey Assigned?

↓

Was Survey Submitted?

↓

Was Response Stored?

↓

Was Score Generated?

↓

Can UI Read It?
```

This mindset helps you identify where the process stopped instead of jumping straight to the last table.

---

# What You've Learned in Step 7

You now understand:

* How the database is organized around the **patient**.
* Why tables are split by responsibility.
* One-to-One, One-to-Many, and Many-to-Many relationships.
* The purpose of Primary Keys and Foreign Keys.
* How data flows from one table to another.
* How to trace a patient's journey through the database.

---

## Next: Step 8 – Data Lifecycle

In the next step, we'll follow **one patient record** from the moment it is created until years later when follow-up surveys, additional appointments, updated scores, reports, and archival come into play. This will help you understand how data evolves over time in the SPRI system, not just how it's stored.
