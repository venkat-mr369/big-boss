Excellent. Now we're moving from the **business flow** to the **actual codebase**. This is where you'll spend most of your time as a Database Developer.

# Step 5 – Repository Architecture

A repository (repo) is a separate codebase with a specific responsibility.

Instead of putting everything into one application, the SPRI project is split into multiple services.

```text
                    SPRI Project
                         │
 ┌──────────────┬──────────────┬──────────────┬──────────────┐
 │              │              │              │
 ▼              ▼              ▼              ▼
spri-client  patient-service survey-service common-library
(UI)            (API)            (API)         (Shared)
```

Each repository has its own purpose.

---

# Repository 1 – spri-client

### Type

```
Angular Application
```

### Responsibility

This is the web application used by:

* Doctors
* Receptionists
* Admins
* Clinical Staff

Patients do **not** use this application.

---

### What does it do?

Examples

```
Patient Search

Appointment Screen

Assign Survey

Reports

Dashboard

Charts
```

---

### Does it connect directly to MariaDB?

❌ No

Instead it calls APIs.

```text
Angular UI

↓

HTTP Request

↓

patient-service
```

Example

```text
Doctor clicks

Patient Details

↓

GET /patients/1001
```

The UI never executes SQL directly.

---

# Repository 2 – patient-service

### Type

```
NestJS Backend API
```

Think of it as the main business service.

---

### Responsibilities

Patient operations

Appointment operations

Doctor Dashboard

Patient History

Medical Scores

Reports

---

### Example APIs

```
GET /patients

POST /patients

PUT /patients

GET /appointments

GET /scores
```

---

### Database Access

Uses

```
Prisma
```

Example

```text
Angular

↓

patient-service

↓

Prisma

↓

MariaDB
```

---

# Repository 3 – survey-service

### Type

```
NestJS Backend API
```

Purpose

Everything related to surveys.

---

Responsibilities

Receive survey

Save survey

Validate survey

Manage survey status

Integrate with FormSite

---

Flow

```text
FormSite

↓

survey-service

↓

MariaDB
```

---

Typical APIs

```
POST /survey-response

GET /survey

PUT /survey
```

---

# Repository 4 – common-library

This repository is extremely important for Database Developers.

### Type

```
Shared Library
```

Instead of writing the same code in every service,

everyone uses common-library.

---

Contains

```
Prisma Schema

Database Models

Shared Types

Utilities

Database Migrations

Common Functions
```

Think of it as

```text
Common Toolbox
```

Every project uses it.

---

# Prisma

One of the most important technologies.

Instead of writing

```sql
SELECT *
FROM patient;
```

Developers write

```typescript
prisma.patient.findMany()
```

Prisma converts it into SQL.

---

Flow

```text
Application

↓

Prisma

↓

MariaDB
```

---

# Database

Technology

```
MariaDB
```

Contains

```
Tables

Views

Triggers

Stored Procedures

Functions

Indexes
```

This is where your work mainly happens.

---

# How Repositories Communicate

Suppose a doctor searches for a patient.

### Step 1

Doctor types

```
John
```

inside Angular.

---

### Step 2

Angular sends

```http
GET /patients?name=John
```

to

```
patient-service
```

---

### Step 3

patient-service

uses

```
Prisma
```

---

### Step 4

Prisma executes

```sql
SELECT *
FROM patient
WHERE name='John';
```

---

### Step 5

MariaDB returns

```
Patient Data
```

---

### Step 6

Prisma converts database rows into objects.

---

### Step 7

patient-service returns JSON.

```json
{
  "id": 1001,
  "name": "John"
}
```

---

### Step 8

Angular displays

```
John

DOB

MRN

Phone
```

---

# Complete Flow

```text
Doctor

↓

Angular UI

↓

patient-service

↓

Prisma

↓

MariaDB

↓

Prisma

↓

patient-service

↓

Angular

↓

Doctor
```

---

# Survey Flow

Now another example.

Patient submits survey.

```text
Patient

↓

FormSite

↓

survey-service

↓

Prisma

↓

MariaDB

↓

Stored Procedures

↓

pro_score

↓

patient-service

↓

Angular

↓

Doctor
```

Notice something.

There are **two backend services** involved.

The first

```
survey-service
```

collects data.

The second

```
patient-service
```

shows results.

---

# Where does the Database Developer work?

Mostly here.

```text
MariaDB

│

├── Tables

├── Views

├── Indexes

├── Triggers

├── Stored Procedures

├── Functions

├── Constraints

└── Migrations
```

Sometimes you also work in

```
common-library
```

because that's where Prisma schema changes and database migrations are often maintained.

---

# Where does a Backend Developer work?

```
patient-service

survey-service

Controllers

Services

Prisma Queries

Business Logic
```

---

# Where does a Frontend Developer work?

```
spri-client

Angular Components

HTML

CSS

TypeScript

Charts
```

---

# Overall Architecture

```text
                    Doctor
                       │
                       ▼
                spri-client
               (Angular UI)
                       │
             HTTP REST API Calls
                       │
      ┌────────────────┴───────────────┐
      ▼                                ▼
patient-service                  survey-service
(NestJS API)                     (NestJS API)
      │                                │
      └──────────────┬─────────────────┘
                     ▼
              common-library
         (Prisma + Shared Code)
                     │
                     ▼
                 MariaDB
     Tables | Views | SPs | Triggers
```

---

# Responsibilities Summary

| Repository        | Technology          | Responsibility                                               |
| ----------------- | ------------------- | ------------------------------------------------------------ |
| `spri-client`     | Angular             | Doctor and staff web interface                               |
| `patient-service` | NestJS              | Patient, appointments, dashboards, reporting                 |
| `survey-service`  | NestJS              | Survey intake, validation, FormSite integration              |
| `common-library`  | TypeScript + Prisma | Shared models, Prisma schema, migrations, reusable utilities |
| `MariaDB`         | Database            | Stores data and executes database logic                      |

---

## What you should remember from Step 5

As a Database Developer, you don't need to master every line of Angular or NestJS code, but you **do** need to understand how requests flow through the system:

* **`spri-client`** presents information to doctors and staff.
* **`patient-service`** manages patient-related business operations.
* **`survey-service`** receives and processes survey submissions.
* **`common-library`** provides shared Prisma models and database-related code used by multiple services.
* **MariaDB** is the central data store where tables, views, triggers, stored procedures, and functions execute the core database logic.

Understanding this flow makes it much easier to trace a request from the UI down to the SQL layer and back again.

### Next: Step 6 – Database Object Architecture

In the next step, we'll focus entirely on the MariaDB database itself. We'll learn the purpose of each database object—**tables, views, indexes, triggers, functions, stored procedures, events, and migrations**—and how they work together inside the SPRI project. This is the foundation you'll use when reading SQL scripts and troubleshooting database behavior.
