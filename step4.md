Perfect. We'll completely ignore Jira and production bugs from now on. Let's learn the project like a new employee joining the team.

---

# Step 4 – Complete Application Flow (End-to-End)

In the previous steps, we focused mostly on the database.

Now let's understand **how the entire application works**, from the moment a patient is created until a doctor views the results.

Think of the system as different departments working together.

```text
                Receptionist
                     │
                     ▼
              Patient Module
                     │
                     ▼
          Appointment Module
                     │
                     ▼
             Survey Assignment
                     │
                     ▼
             Email Notification
                     │
                     ▼
            Patient Completes Survey
                     │
                     ▼
              Survey Processing
                     │
                     ▼
            PRO Score Calculation
                     │
                     ▼
             Doctor Dashboard
```

Every box above is a module.

---

# Module 1 — Patient Management

### Type

Business Module

### Main Responsibility

Maintain patient information.

---

### Main Table

```text
patient
(TABLE)
```

Stores

* Patient Name
* DOB
* Gender
* MRN
* Phone
* Email

Example

```text
Patient

↓

Create Patient

↓

patient Table
```

---

### APIs

Examples

```text
POST /patients

GET /patients/{id}

PUT /patients/{id}
```

---

### UI

Receptionist Screen

```text
+-------------------------+

Create Patient

Name

DOB

Phone

MRN

Save

+-------------------------+
```

---

# Module 2 — Appointment Management

Once patient exists,

doctor appointment is created.

---

### Table

```text
appointment
(TABLE)
```

Stores

* Appointment Date
* Doctor
* Facility
* Visit Type

Relationship

```text
patient

↓

appointment
```

One patient

↓

Many appointments

---

### API

```text
POST /appointments

GET /appointments
```

---

### UI

Doctor Calendar

```text
Monday

10 AM

John

11 AM

Smith

12 PM

Mary
```

---

# Module 3 — Survey Management

Now doctor wants patient to answer a questionnaire.

System assigns survey.

---

### Table

```text
patient_survey
(TABLE)
```

Stores

```text
Patient

Survey

Status

Assigned Date
```

Status example

```text
Assigned

Sent

Completed

Expired
```

---

### API

```text
POST /patient-surveys
```

---

### UI

Doctor clicks

```text
Assign Survey
```

---

# Module 4 — Notification

Survey is assigned.

Now patient should receive it.

System sends

Email

or

SMS

---

Usually contains

```text
Survey Link

Patient Token

Expiry Date
```

Example

```text
https://fs7.formsite.com/...
```

---

Patient clicks.

---

# Module 5 — FormSite

Notice something important.

Patient is **NOT inside SPRI UI** anymore.

Patient is now inside

```text
FormSite
```

External Application.

Flow

```text
SPRI

↓

Email

↓

FormSite

↓

Patient fills survey
```

---

### UI

Example

```text
Pain Level

( )

0

( )

1

( )

2

...

10
```

Next Question

```text
Walking

Yes

No
```

Next

```text
Sports

Daily

Weekly

Never
```

---

# Module 6 — Survey Processing

Patient clicks

```text
Submit
```

Now backend starts working.

---

### Service

```text
survey-service

(API)
```

Responsibilities

Receive survey

↓

Validate

↓

Save

↓

Call Database

---

### Database

```text
form_response
(TABLE)
```

Stores

```text
Survey Answers
```

Nothing else.

---

# Module 7 — Business Logic

Now database starts processing.

Objects involved

```text
TRIGGER

↓

Stored Procedures

↓

Functions

↓

Calculations
```

Purpose

Convert

```text
Patient Answers

↓

Medical Scores
```

---

### Tables involved

```text
form_response

(TABLE)

↓

pro_score

(TABLE)
```

---

# Module 8 — Reporting

Doctor doesn't want

100 survey answers.

Doctor wants

```text
Patient Score

↓

Previous Score

↓

Improvement
```

---

Application reads

```text
pro_score
(TABLE)
```

---

### API

Usually

```text
patient-service
```

Example

```text
GET

/patients/{id}/scores
```

---

Returns

```json
{
  "harris":91,
  "ikdc":84,
  "womac":78
}
```

---

# Module 9 — Doctor Dashboard

Doctor opens patient.

UI

```text
Patient

↓

Appointments

↓

Surveys

↓

Scores

↓

Charts
```

Graph

```text
Harris Score

95

|

90

|

85

|

80

|

75

|

_____________________

Jan

Feb

Mar

Apr
```

Doctor immediately understands

Recovery is improving.

---

# Overall Application Architecture

```text
                 Receptionist
                       │
                       ▼
             Patient Management
                 (patient)
                       │
                       ▼
          Appointment Management
              (appointment)
                       │
                       ▼
             Survey Assignment
            (patient_survey)
                       │
                       ▼
             Email Notification
                       │
                       ▼
         External FormSite Application
                       │
                       ▼
              survey-service (API)
                       │
                       ▼
          form_response (TABLE)
                       │
                       ▼
      Trigger + Stored Procedures
                       │
                       ▼
           pro_score (TABLE)
                       │
                       ▼
           patient-service (API)
                       │
                       ▼
          Doctor Dashboard (UI)
```

---

# Technology Stack

Now let's map everything to technologies.

| Layer               | Technology                 | Purpose                                                |
| ------------------- | -------------------------- | ------------------------------------------------------ |
| UI                  | Angular (`spri-client`)    | Doctor and staff web application                       |
| External UI         | FormSite                   | Patient survey forms                                   |
| Backend API         | NestJS (`patient-service`) | Patient, appointments, scores                          |
| Backend API         | NestJS (`survey-service`)  | Survey processing                                      |
| Shared Library      | `common-library`           | Prisma schema, shared database logic, migrations       |
| ORM                 | Prisma                     | Database access and migrations                         |
| Database            | MariaDB                    | Stores all application data                            |
| Database Logic      | Stored Procedures          | Medical score calculations                             |
| Database Automation | Triggers                   | Automatically invoke business logic after data changes |

---

# Responsibilities by Layer

| Layer             | Main Responsibility                            |
| ----------------- | ---------------------------------------------- |
| Angular UI        | User interaction for doctors and staff         |
| FormSite          | Collect survey responses from patients         |
| APIs              | Business operations and database communication |
| Prisma            | Execute SQL and manage schema changes          |
| MariaDB Tables    | Persist application data                       |
| Stored Procedures | Implement complex medical scoring logic        |
| Triggers          | Start database workflows automatically         |
| Dashboard         | Display patient recovery information           |

---

## What you've learned so far

At this point, you understand the **entire business journey**:

1. A patient is created.
2. An appointment is scheduled.
3. A survey is assigned.
4. The patient completes the survey in FormSite.
5. The backend saves the response.
6. The database processes the response.
7. Scores are calculated and stored.
8. Doctors view those scores in the SPRI application.

You now have the high-level picture.

### Next: Step 5 – Repository Architecture

We'll leave the business flow behind and move into the source code. We'll examine each repository (`spri-client`, `patient-service`, `survey-service`, `common-library`) and answer questions like:

* Which repository owns which functionality?
* Which repository talks directly to MariaDB?
* Where is Prisma used?
* Where are migrations stored?
* How do the repositories communicate with each other?

This will connect the architecture you've learned with the actual codebase you'll work on every day.
