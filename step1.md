### Step 1 – Business Overview (SPRI)

### Imagine this real-world scenario

A patient injures their knee while playing cricket.

The patient visits **Steadman Philippon Research Institute (SPRI)**.

The doctor examines the patient and performs surgery.

However, surgery is not the end.

The doctor wants to monitor the patient's recovery for several months.

Questions like:

* Is the pain decreasing?
* Can the patient walk normally?
* Can they climb stairs?
* Can they play sports again?
* Is the surgery successful?

Instead of asking these questions during every hospital visit, the hospital sends an **online survey** to the patient.

---

# Business Flow

```text
Patient gets surgery
        │
        ▼
Hospital creates patient
        │
        ▼
Survey is assigned
        │
        ▼
Patient receives email
        │
        ▼
Patient opens survey
        │
        ▼
Patient submits answers
        │
        ▼
System calculates medical score
        │
        ▼
Doctor reviews recovery
```

This is the complete business process.

---

# Who are the users?

## 1. Patient

Uses only the survey.

Example:

* Opens survey link
* Answers questions
* Clicks Submit

Patient never logs into the SPRI application.

---

## 2. Doctor

Uses the SPRI web application.

Doctor can:

* Search patients
* View appointments
* View surgery details
* View PRO Scores
* Compare previous scores

---

## 3. Receptionist

Uses the application to:

* Register patients
* Schedule appointments
* Update demographics

---

## 4. Database Developer (You)

You don't create surveys.

You don't treat patients.

Your job is to make sure:

* Data is saved correctly
* Stored Procedures work
* Scores are calculated
* Reports are accurate
* Migrations run successfully
* Production issues are fixed

That's why Jira tickets like SOPMP-1609 and SOPMP-1669 come to you.

---

# Main Modules

Think of the project as several departments.

```text
SPRI
│
├── Patient Management
├── Appointment Management
├── Survey Management
├── PRO Score Calculation
├── Reporting
└── Database Migration
```

---

# Patient Journey

Let's follow one patient.

Patient:

```text
John
```

### Step 1

Receptionist creates John.

Database:

```text
patient
(TABLE)
```

Stores:

* Name
* DOB
* MRN
* Phone

---

### Step 2

Doctor creates appointment.

Database:

```text
appointment
(TABLE)
```

Stores:

* Appointment Date
* Doctor
* Facility
* Joint Type

---

### Step 3

Doctor decides

> "Send Knee Survey."

Database creates

```text
patient_survey
(TABLE)
```

This tells the system:

> John should complete a Knee Survey.

---

### Step 4

System sends email.

Email contains

```text
https://fs7.formsite....
```

Patient clicks it.

---

### Step 5

Patient answers survey.

Example

```text
Pain = 7

Walking = Yes

Running = No
```

Clicks

```text
Submit
```

---

### Step 6

Survey reaches your database.

Stored in

```text
form_response
(TABLE)
```

Nothing has been calculated yet.

---

### Step 7

Stored Procedures calculate scores.

Example

```text
IKDC

WOMAC

Harris

VHS
```

Results are stored in

```text
pro_score
(TABLE)
```

---

### Step 8

Doctor opens Dashboard.

The application reads

```text
pro_score
```

and displays

```text
Patient Score

Previous Score

Improvement

Graphs
```

---

# Why are PRO Scores important?

Doctors don't want to read 80 survey answers every time.

Instead of seeing:

```text
Pain = 6

Walking = Yes

Sports = Sometimes

Running = No

Swelling = Mild

...
```

They see:

```text
Harris Hip Score = 91

WOMAC = 84

IKDC = 76
```

These are standardized medical scores that summarize the patient's recovery.

---

# Where does your work begin?

As a Database Developer, your work starts **after the patient clicks Submit**.

```text
Patient
   │
Submit Survey
   │
   ▼
Database
   │
   ▼
Tables
   │
   ▼
Triggers
   │
   ▼
Stored Procedures
   │
   ▼
PRO Score
```

This is the part you are responsible for.

---

# In the SPRI project, you mainly work with these database objects

| Object           | Example                                  | Purpose                                                                 |
| ---------------- | ---------------------------------------- | ----------------------------------------------------------------------- |
| TABLE            | `patient`                                | Patient information                                                     |
| TABLE            | `appointment`                            | Appointment details                                                     |
| TABLE            | `patient_survey`                         | Survey assigned to a patient                                            |
| TABLE            | `form_response`                          | Raw survey answers                                                      |
| TABLE            | `pro_score`                              | Calculated medical scores                                               |
| TRIGGER          | `trg_form_response_ai_pro_score`         | Automatically starts score calculation after a survey response is saved |
| STORED PROCEDURE | `spri_apply_pro_score_for_form_response` | Creates or updates the PRO score                                        |
| STORED PROCEDURE | `spri_pro_calculate_for_response`        | Calculates the medical scores                                           |
| STORED PROCEDURE | `spri_pro_payload_json`                  | Reads and normalizes the survey JSON                                    |

---

## Homework (important)

Before we move to Step 2, make sure you can answer these questions yourself:

1. What is the purpose of the SPRI project?
2. Who uses the SPRI application?
3. What is a PRO Score?
4. Why is the `form_response` table needed?
5. Why is the `pro_score` table needed?
6. At what point does a Database Developer become involved in the workflow?

Once you're comfortable with these, we'll continue to **Step 2: Database Architecture**, where we'll examine the major tables one by one, their relationships, and how data flows between them.
