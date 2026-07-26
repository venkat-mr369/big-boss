Excellent. Since we've updated **Step 2 (Architecture)** and **Step 3 (Request Lifecycle)**, we should also replace the old **Step 4**.

The old Step 4 was mostly about "Patient → Survey → Database". Now we have enough understanding to explain the **complete business process** from beginning to end. This is how I would write the new Step 4.

---

# Step 4 – End-to-End Business Process

Now that we understand the system architecture and request lifecycle, let's understand the **complete business process** of the SPRI application.

This is the flow followed every day by doctors, receptionists, patients, and the backend services.

Think of this as watching one patient move through the entire system.

---

# Complete Business Flow

```text
Receptionist
      │
      ▼
Create Patient
      │
      ▼
Book Appointment
      │
      ▼
Doctor Consultation
      │
      ▼
Assign Survey
      │
      ▼
Notification Service
      │
      ▼
Patient Receives Email
      │
      ▼
Patient Completes Survey
      │
      ▼
Survey Processing
      │
      ▼
Database Processing
      │
      ▼
Score Calculation
      │
      ▼
Doctor Dashboard
      │
      ▼
Patient Follow-up
```

This is the complete lifecycle of one patient.

---

# Phase 1 – User Login

Every staff member begins here.

```text
Doctor

↓

Angular UI

↓

auth-service

↓

JWT Token

↓

Dashboard
```

Without authentication, the user cannot access the system.

---

# Phase 2 – Patient Registration

Usually performed by:

* Receptionist
* Front Desk
* Clinical Staff

Flow

```text
Receptionist

↓

Angular UI

↓

patient-service

↓

Create Patient

↓

MariaDB

↓

patient Table
```

Patient information includes:

* Name
* MRN
* DOB
* Gender
* Contact Information

---

# Phase 3 – Appointment Scheduling

Once the patient exists,

an appointment is created.

```text
Doctor

↓

Appointment Module

↓

patient-service

↓

appointment Table
```

Appointment stores

* Doctor
* Clinic
* Date
* Time
* Visit Type

---

# Phase 4 – Doctor Consultation

Patient visits hospital.

Doctor

* examines patient
* reviews history
* decides treatment
* decides whether a survey is required

No survey is generated automatically.

The doctor or clinical workflow determines when to send one.

---

# Phase 5 – Survey Assignment

Doctor selects

```text
Assign Survey
```

Flow

```text
Angular

↓

patient-service

↓

Survey Assignment

↓

Database

↓

notification-service
```

The system prepares

* Survey
* Patient
* Expiry Date
* Token

---

# Phase 6 – Patient Notification

notification-service now sends

```text
Email

or

SMS
```

Example

```text
Dear John,

Please complete your Knee Follow-up Survey.

Click Here
```

Patient receives

```text
Survey Link
```

---

# Phase 7 – Patient Completes Survey

Patient opens

```text
FormSite
```

This is an external application.

Patient answers questions.

Example

```text
Pain

Walking

Sports

Daily Activities

Quality of Life
```

Patient clicks

```text
Submit
```

---

# Phase 8 – Survey Processing

FormSite sends data to

```text
survey-service
```

Flow

```text
FormSite

↓

survey-service

↓

Validation

↓

common-library

↓

Prisma

↓

MariaDB
```

Survey answers are stored.

---

# Phase 9 – Database Processing

Once data reaches MariaDB,

database processing begins.

```text
form_response

↓

Trigger

↓

Stored Procedures

↓

Functions

↓

Business Rules
```

Responsibilities

* Read survey answers
* Validate data
* Calculate scores
* Update result tables

---

# Phase 10 – Score Generation

Database calculates

Example

```text
Clinical Scores

↓

Recovery Status

↓

Progress

↓

Assessment
```

Results stored in

```text
pro_score
```

---

# Phase 11 – Doctor Dashboard

Doctor opens dashboard.

Flow

```text
Angular

↓

patient-service

↓

Prisma

↓

MariaDB

↓

pro_score

↓

Dashboard
```

Doctor sees

* Current Score
* Previous Score
* Trend
* Recovery Progress

---

# Phase 12 – Follow-up

Recovery usually takes months.

Patient may receive

* 6 Week Survey
* 3 Month Survey
* 6 Month Survey
* 1 Year Survey

Each survey repeats the same process.

```text
Survey

↓

Database

↓

Score

↓

Dashboard
```

Doctor compares recovery over time.

---

# Scheduler Service Responsibilities

Not every task happens immediately.

Some jobs run automatically.

Example

```text
12:00 AM

↓

scheduler-service

↓

Find Pending Surveys

↓

notification-service

↓

Reminder Emails
```

Another example

```text
Every Morning

↓

Generate Reports
```

---

# Notification Flow

```text
Doctor

↓

Assign Survey

↓

patient-service

↓

notification-service

↓

Email

↓

Patient
```

---

# Authentication Flow

```text
User

↓

Angular

↓

auth-service

↓

JWT Token

↓

Business Services
```

Every request after login carries the JWT token.

---

# Complete End-to-End Architecture

```text
                    User
                      │
                      ▼
               Angular UI
                      │
                      ▼
               auth-service
                      │
      ┌───────────────┼────────────────┐
      │               │                │
      ▼               ▼                ▼
patient-service  survey-service  admin-service
      │               │
      │               ▼
      │        FormSite Integration
      │
      ├──────────────► notification-service
      │
      ▼
common-library
      │
      ▼
Prisma ORM
      │
      ▼
MariaDB
      │
      ▼
Tables
Triggers
Stored Procedures
Functions
      │
      ▼
Clinical Scores
      │
      ▼
patient-service
      │
      ▼
Angular Dashboard
      │
      ▼
Doctor
```

---

# Responsibility Matrix

| Phase                   | Primary Component         | Responsibility                             |
| ----------------------- | ------------------------- | ------------------------------------------ |
| Authentication          | auth-service              | User authentication and JWT generation     |
| Patient Registration    | patient-service           | Create and maintain patient records        |
| Appointment Management  | patient-service           | Schedule and manage appointments           |
| Survey Assignment       | patient-service           | Assign surveys to patients                 |
| Notifications           | notification-service      | Send survey invitations and reminders      |
| Survey Collection       | FormSite + survey-service | Collect and validate survey responses      |
| Shared Data Access      | common-library            | Shared Prisma models and utilities         |
| Database Processing     | MariaDB                   | Store responses and execute business logic |
| Background Processing   | scheduler-service         | Scheduled reminders and maintenance tasks  |
| Administration          | admin-service             | Manage users, roles, and configuration     |
| Multi-Tenant Management | tenant-service            | Tenant-specific settings and configuration |
| Reporting & Dashboard   | patient-service + Angular | Display patient recovery information       |

---

# Key Takeaways

At this point, you should be able to explain the SPRI platform to a new team member:

* **Authentication** is handled by `auth-service`.
* **Doctors and staff** use the Angular application.
* **Patients** complete surveys in **FormSite**, not in the Angular application.
* **patient-service** manages patient-related workflows such as registration, appointments, survey assignments, and dashboard data.
* **notification-service** delivers emails and reminders.
* **survey-service** receives and validates completed surveys.
* **common-library** provides shared Prisma models and utilities to backend services.
* **MariaDB** stores the data and runs the core business logic through tables, triggers, stored procedures, and functions.
* **scheduler-service** runs time-based background jobs.
* **admin-service** and **tenant-service** support administration and multi-tenant configuration.

This updated Step 4 provides the complete operational picture of how the SPRI platform functions from the moment a user logs in until a doctor reviews a patient's recovery progress.
