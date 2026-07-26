Excellent. From this step onward, we're entering the part where you'll spend **80% of your daily work** as a Database Developer.

Forget APIs and Angular for a while. We are now inside **MariaDB**.

---

# Step 6 – Database Object Architecture

Think of MariaDB as a hospital.

Inside the hospital, every employee has a different responsibility.

Similarly, every database object has a different responsibility.

```text
                    MariaDB Database
                           │
 ┌──────────┬─────────┬──────────┬──────────┬──────────┬─────────┐
 │          │         │          │          │          │
 ▼          ▼         ▼          ▼          ▼          ▼
Tables    Views    Indexes   Triggers   Procedures  Functions
```

Each object works together.

Let's understand them one by one.

---

# 1. TABLE

## Purpose

A table stores data permanently.

Think of it as an Excel sheet.

Example

```text
patient
```

| patient_id | name  | dob        |
| ---------- | ----- | ---------- |
| 1001       | John  | 1990-01-01 |
| 1002       | David | 1985-06-08 |

---

Another table

```text
appointment
```

| appointment_id | patient_id | doctor   |
| -------------- | ---------- | -------- |
| 2001           | 1001       | Dr Smith |

---

Another table

```text
form_response
```

| id | answers |
| -- | ------- |
| 1  | JSON    |

---

Remember

**Tables only store data.**

They don't perform calculations.

---

# 2. VIEW

## Purpose

A View is a saved SQL query.

Instead of writing

```sql
SELECT
    p.name,
    a.date
FROM patient p
JOIN appointment a
ON p.id=a.patient_id;
```

every day,

create

```text
patient_appointment_view
```

Now users simply execute

```sql
SELECT *
FROM patient_appointment_view;
```

---

Think of a View as

```text
Shortcut

↓

Saved Query
```

Views do not store data.

They show data from tables.

---

# 3. INDEX

One of the most important DBA topics.

Suppose patient table has

```text
20 Million Rows
```

Searching

```text
MRN = MR12345
```

Without index

```text
Database

↓

Reads Row 1

↓

Row 2

↓

Row 3

...

↓

20 Million
```

Very slow.

---

With index

```text
Database

↓

Index

↓

Exact Location

↓

Patient Row
```

Very fast.

---

Think

Book

Without index

↓

Read every page

With index

↓

Go directly to page number

---

Common indexes

```sql
PRIMARY KEY

UNIQUE

INDEX

COMPOSITE INDEX
```

---

# 4. PRIMARY KEY

Every table needs a unique identity.

Example

```text
patient_id
```

Cannot repeat.

```text
1001

1002

1003
```

Never duplicate.

---

# 5. FOREIGN KEY

Connects tables.

Example

```text
patient

↓

appointment
```

Relationship

```text
patient.id

↓

appointment.patient_id
```

Now appointment must belong to a valid patient.

---

Example

```text
patient

1001

↓

appointment

patient_id=1001
```

---

# 6. STORED PROCEDURE (SP)

Very important.

Purpose

Perform business logic.

Example

```sql
CALL calculate_patient_score();
```

SP can

Read tables

↓

Update tables

↓

Insert data

↓

Delete data

↓

Call other procedures

↓

Return results

Think

```text
Mini Program
```

inside MariaDB.

---

# 7. FUNCTION

Looks similar to SP.

But different.

Function returns one value.

Example

```sql
calculate_age(DOB)
```

Returns

```text
36
```

Example

```sql
SELECT calculate_age(dob)
FROM patient;
```

Another

```sql
calculate_bmi(weight,height)
```

Returns

```text
24.6
```

---

Difference

Stored Procedure

```text
Can modify database
```

Function

```text
Usually returns value
```

---

# 8. TRIGGER

Runs automatically.

Nobody calls it.

Example

```sql
INSERT INTO form_response
```

↓

Trigger runs

↓

Stored Procedure runs

Trigger types

```text
BEFORE INSERT

AFTER INSERT

BEFORE UPDATE

AFTER UPDATE

BEFORE DELETE

AFTER DELETE
```

---

Think

Motion sensor light

```text
Person walks

↓

Light ON
```

Nobody presses switch.

Exactly like Trigger.

---

# 9. EVENT

Less common.

Runs on schedule.

Example

Every night

2 AM

↓

Delete logs older than 1 year

or

Generate reports

Example

```sql
CREATE EVENT
```

Runs automatically.

Think

Cron Job inside database.

---

# 10. CONSTRAINTS

Protect data quality.

Examples

NOT NULL

```text
Patient Name

Cannot be NULL
```

UNIQUE

```text
MRN

Cannot duplicate
```

CHECK

```text
Age > 0
```

FOREIGN KEY

```text
Appointment

Must belong to Patient
```

---

# 11. TRANSACTIONS

Suppose

Insert Patient

↓

Insert Appointment

↓

Insert Billing

If Appointment fails

Should Patient remain?

Usually

No.

Everything rolls back.

Example

```sql
START TRANSACTION;

INSERT...

INSERT...

UPDATE...

COMMIT;
```

If error

```sql
ROLLBACK;
```

---

Think

ATM Withdrawal

```text
Debit Account

↓

Dispense Cash

↓

Commit
```

If cash machine fails

↓

Rollback

Money should not disappear.

---

# 12. DATABASE FLOW

Let's see all objects together.

```text
Application

↓

INSERT

↓

TABLE

↓

TRIGGER

↓

Stored Procedure

↓

FUNCTION

↓

TABLE UPDATE

↓

VIEW

↓

Doctor Reads Data
```

---

# Example Using Patient Flow

```text
Create Patient

↓

patient
(TABLE)

↓

Create Appointment

↓

appointment
(TABLE)

↓

Assign Survey

↓

patient_survey
(TABLE)

↓

Patient submits survey

↓

form_response
(TABLE)

↓

Trigger

↓

Stored Procedure

↓

Function

↓

Score Calculation

↓

pro_score
(TABLE)

↓

View

↓

Doctor Dashboard
```

---

# How Everything Connects

```text
                    MariaDB
                        │
 ┌──────────────┬──────────────┬──────────────┐
 │              │              │
 ▼              ▼              ▼
Tables       Procedures      Views
 │              │              │
 │              ▼              │
 │          Functions          │
 │              │              │
 ▼              ▼              ▼
Triggers     Transactions   Indexes
 │
 ▼
Updated Tables
```

---

# Where a Database Developer Spends Time

| Object              | Typical DBA Work                                       |
| ------------------- | ------------------------------------------------------ |
| Tables              | Design schema, add/modify columns, enforce constraints |
| Indexes             | Improve query performance                              |
| Views               | Create reporting and reusable query layers             |
| Stored Procedures   | Implement and maintain business logic                  |
| Functions           | Reusable calculations and helper logic                 |
| Triggers            | Automate actions after data changes                    |
| Transactions        | Ensure data consistency                                |
| Constraints         | Maintain data integrity                                |
| Events              | Schedule recurring database tasks                      |
| Users & Permissions | Control who can access which objects                   |

---

# Daily Work Example

Imagine a developer says:

> "The Doctor Dashboard is loading slowly."

As a Database Developer, your investigation usually follows this path:

```text
Doctor Dashboard

↓

API Query

↓

View (if used)

↓

Stored Procedure (if used)

↓

Tables

↓

Indexes

↓

Execution Plan

↓

Optimize SQL
```

Or they report:

> "The patient's score wasn't updated."

You would typically check:

```text
Was the data inserted?

↓

Did the trigger execute?

↓

Did the stored procedure complete successfully?

↓

Was the target table updated?

↓

Were there any transaction errors?
```

Notice how the investigation follows the database object architecture you just learned.

---

## What you've learned in Step 6

You now understand the purpose of every major database object:

* **Tables** store data.
* **Views** present reusable query results.
* **Indexes** improve performance.
* **Primary and Foreign Keys** define relationships.
* **Stored Procedures** implement business logic.
* **Functions** return reusable computed values.
* **Triggers** react automatically to data changes.
* **Transactions** keep operations consistent.
* **Constraints** protect data quality.
* **Events** automate scheduled work.

This foundation is essential because almost every production issue, enhancement, or migration will involve one or more of these objects.

### Next: Step 7 – Database Schema & Relationships

We'll move from individual objects to the **complete database schema**. We'll learn how tables like `patient`, `appointment`, `patient_survey`, `form_response`, `pro_score`, and other core tables relate to each other using primary keys and foreign keys, so you can visualize the entire database as one connected system rather than isolated tables.
