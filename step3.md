Excellent. This is the most important step. Once you understand this, you'll be able to debug most database tickets in this project.

# Step 3 – Stored Procedure Flow (SP Call Chain)

Let's trace what happens **after the patient clicks Submit**.

Imagine the patient completes the survey.

```text
Patient

↓

Clicks Submit
```

The database receives the survey.

Now everything happens automatically.

---

# Complete Flow

```text
Patient
   │
   ▼
survey-service
(API)
   │
   ▼
INSERT INTO form_response
(TABLE)
   │
   ▼
AFTER INSERT Trigger
(TRIGGER)
   │
   ▼
spri_apply_pro_score_for_form_response()
(SP)
   │
   ▼
spri_pro_calculate_for_response()
(SP)
   │
   ▼
spri_pro_payload_json()
(SP)
   │
   ▼
Score Calculation Procedures
(SP)
   │
   ▼
INSERT / UPDATE pro_score
(TABLE)
```

Now let's understand every step.

---

# Step 1

## survey-service

Type

```
API
```

Responsibility

Receives survey data from FormSite.

Example JSON

```json
{
    "patient_id":"P1001",
    "answers":{
        ...
    }
}
```

The API saves it.

SQL

```sql
INSERT INTO form_response(...)
VALUES(...);
```

Nothing else.

---

# Step 2

## form_response

Type

```
TABLE
```

Stores

```text
Response ID

Patient ID

External Form ID

Answers JSON
```

Example

| id    | patient_id | answers |
| ----- | ---------- | ------- |
| FR100 | P1001      | JSON    |

---

# Step 3

## Trigger fires

Type

```
TRIGGER
```

Name

```
trg_form_response_ai_pro_score
```

Meaning

```
AFTER INSERT
```

Whenever a row is inserted

↓

Database automatically runs

```sql
CALL spri_apply_pro_score_for_form_response(...)
```

The Jira deployment checklist verifies this trigger exists. 

---

# Step 4

## spri_apply_pro_score_for_form_response()

Type

```
Stored Procedure
```

Think of it as the **Manager**.

It doesn't calculate scores.

It coordinates everything.

Responsibilities

✅ Read form_response ID

↓

Call calculator

↓

Receive scores

↓

Insert or Update pro_score

Think

```text
Manager

↓

Employee 1

↓

Employee 2

↓

Store result
```

---

# Step 5

## spri_pro_calculate_for_response()

Type

```
Stored Procedure
```

This is the main calculator.

Input

```text
form_response_id
```

It performs

### Read survey

```sql
SELECT answers
FROM form_response;
```

↓

Reads

```text
answers JSON
```

↓

Needs

```text
Pain

Walking

Sports

etc.
```

↓

Calls JSON parser.

---

# Step 6

## spri_pro_payload_json()

Type

```
Stored Procedure
```

This is a **Translator**.

Imagine three people speaking different languages.

Person A speaks English.

Person B speaks Telugu.

Person C speaks Hindi.

Before they communicate,

someone translates.

That's exactly what

```text
spri_pro_payload_json
```

does.

It converts every survey format into one standard format.

---

Old format

```json
{
"HARRIS":80
}
```

Works.

---

FormsOrt

```json
{
"answers":{
"HARRIS":80
}
}
```

Works.

---

Escaped JSON

```json
"{\"HARRIS\":80}"
```

Works.

---

FormSite

```json
{
"items":[]
}
```

Originally

❌ Didn't work.

The SQL comments in the fix describe these four supported JSON shapes and explain that FormSite's `items` array needed special handling. 

---

# Step 7

## New Procedure (SOPMP-1609)

```
spri_pro_flatten_formsite_items()
```

Purpose

Translate

```text
92

↓

LYS_SWL
```

Translate

```text
311

↓

DATE_EXAM
```

using

```
spri_formsite_field_map
```

Without this

Calculator couldn't understand

```
92

311

133
```

The new procedure uses the mapping table to build a flat JSON object from the FormSite `items` array. 

---

# Step 8

Calculator starts

Now calculator finally sees

Instead of

```json
{
"id":"92"
}
```

It sees

```json
{
"LYS_SWL":"8"
}
```

Now

```sql
JSON_EXTRACT(...)
```

works.

Score calculated.

---

# Step 9

Back to Manager

Manager receives

```text
Harris = 92

IKDC = 87

WOMAC = 78
```

Now

```sql
INSERT

or

UPDATE
```

into

```
pro_score
```

The validation guide in the Jira even includes running `CALL spri_apply_pro_score_for_form_response('<FORM_RESPONSE_ID>');` and then checking that the `pro_score` row exists. 

---

# Stored Procedure Relationship

Think of this hierarchy.

```text
spri_apply_pro_score_for_form_response()
                │
                │ calls
                ▼
spri_pro_calculate_for_response()
                │
                │ calls
                ▼
spri_pro_payload_json()
                │
                │ calls (only for FormSite)
                ▼
spri_pro_flatten_formsite_items()
                │
                ▼
Returns normalized JSON
                │
                ▼
Calculator computes scores
                │
                ▼
Returns result
                │
                ▼
pro_score updated
```

---

# Where did SOPMP-1609 fail?

Everything above was working.

Problem occurred here

```text
Patient

↓

form_response

↓

Trigger

↓

Manager SP

↓

Calculator SP

↓

JSON Parser

↓

❌ Could not understand FormSite JSON

↓

NULL

↓

pro_score
```

That means

* Trigger ✅
* Manager SP ✅
* Calculator SP ✅
* JSON Parser ❌

The parser could not extract values because the incoming JSON format had changed.

---

# Think of it like a factory

```
Patient

↓

Raw Material

↓

Machine 1

↓

Machine 2

↓

Machine 3

↓

Finished Product
```

Real project

```
Patient

↓

form_response

↓

Trigger

↓

Manager SP

↓

Calculator SP

↓

JSON Translator

↓

Score Calculator

↓

pro_score
```

---

# As a DBA, when debugging an issue, ask these questions in order:

1. Did a row get inserted into `form_response`?
2. Did the trigger execute?
3. Was `spri_apply_pro_score_for_form_response()` called?
4. Did `spri_pro_calculate_for_response()` return any values?
5. Did `spri_pro_payload_json()` produce valid normalized JSON?
6. Was a row inserted or updated in `pro_score`?
7. If the row exists but scores are `NULL`, did the JSON parser successfully extract the required fields?

This step-by-step approach is exactly how you narrow the problem to the correct layer instead of guessing.

In **Step 4**, we'll go even deeper by tracing **one actual survey submission** from the `form_response.answers` JSON all the way to individual columns in `pro_score`, using real SQL examples from the project. That will make tickets like SOPMP-1609 much easier to troubleshoot.
