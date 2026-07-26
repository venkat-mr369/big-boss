Excellent. Since **Step 2** now includes the complete microservices architecture, **Step 3** should no longer start directly from `survey-service`. Instead, it should explain **how a request travels through the system before reaching the database**.

I recommend replacing the previous Step 3 with the version below.

---

# Step 3 – Request Lifecycle & Service Communication

Now that we understand the system architecture, let's see **how a single request travels through the application**.

Think of every user action as a journey.

Example actions:

* Login
* Create Patient
* Assign Survey
* Submit Survey
* View Dashboard

Although these are different operations, they all follow a similar lifecycle.

---

# High-Level Request Flow

```text
                 User
                  │
                  ▼
          Angular UI (spri-client)
                  │
                  ▼
          auth-service (JWT Validation)
                  │
                  ▼
           Business Service
(patient-service / survey-service /
 admin-service / tenant-service)
                  │
                  ▼
          common-library (Prisma)
                  │
                  ▼
               MariaDB
                  │
                  ▼
      Triggers / Stored Procedures
                  │
                  ▼
              Tables Updated
                  │
                  ▼
          JSON Response Returned
                  │
                  ▼
             Angular UI
                  │
                  ▼
                 User
```

---

# Request 1 – User Login

### Step 1

User opens

```text
https://spri.company.com
```

---

### Step 2

Angular displays

```text
Login Screen
```

---

### Step 3

User enters

```text
Username

Password
```

---

### Step 4

Angular sends

```http
POST /login
```

to

```text
auth-service
```

---

### Step 5

auth-service

* validates credentials
* checks user
* verifies password

---

### Step 6

If successful

```text
Generate JWT Token
```

---

### Step 7

Angular stores

```text
JWT Token
```

Every future request includes this token.

---

# Request 2 – Search Patient

```text
Doctor

↓

Angular

↓

JWT Token

↓

patient-service

↓

Prisma

↓

MariaDB

↓

Patient Table

↓

JSON

↓

Angular

↓

Doctor
```

---

# Request 3 – Create Patient

```text
Receptionist

↓

Angular

↓

auth-service

↓

patient-service

↓

Validation

↓

Prisma

↓

INSERT INTO patient

↓

MariaDB

↓

Success

↓

Angular
```

---

# Request 4 – Assign Survey

```text
Doctor

↓

Angular

↓

patient-service

↓

Create Survey Assignment

↓

Database

↓

notification-service

↓

Email Sent

↓

Patient Receives Survey Link
```

Notice that **patient-service** is responsible for creating the assignment, while **notification-service** is responsible for delivering it.

---

# Request 5 – Patient Submits Survey

This is the most important flow for a Database Developer.

```text
Patient

↓

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

↓

INSERT INTO form_response

↓

Trigger

↓

Stored Procedures

↓

Score Calculation

↓

UPDATE pro_score

↓

Success
```

---

# Request 6 – Doctor Views Dashboard

```text
Doctor

↓

Angular

↓

patient-service

↓

Prisma

↓

MariaDB

↓

Read pro_score

↓

JSON

↓

Angular Dashboard
```

---

# How Services Communicate

Each service has a dedicated responsibility.

```text
Angular UI
     │
     ├──────────────► auth-service
     │
     ├──────────────► patient-service
     │
     ├──────────────► survey-service
     │
     ├──────────────► admin-service
     │
     └──────────────► tenant-service
```

Backend services communicate with:

```text
common-library

↓

Prisma

↓

MariaDB
```

Some services also communicate with each other.

Example

```text
patient-service

↓

notification-service

↓

Email
```

or

```text
scheduler-service

↓

notification-service

↓

Reminder Emails
```

---

# Complete Survey Processing Flow

This combines everything you've learned so far.

```text
Patient
   │
   ▼
FormSite
   │
   ▼
survey-service
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
form_response (TABLE)
   │
   ▼
AFTER INSERT Trigger
   │
   ▼
Stored Procedure
   │
   ▼
Functions (if required)
   │
   ▼
pro_score (TABLE)
   │
   ▼
patient-service
   │
   ▼
Angular UI
   │
   ▼
Doctor Dashboard
```

---

# Service Responsibilities

| Service                  | Responsibility                    |
| ------------------------ | --------------------------------- |
| **auth-service**         | Authentication, JWT, Roles        |
| **patient-service**      | Patients, Appointments, Dashboard |
| **survey-service**       | Survey Processing                 |
| **notification-service** | Email & SMS Notifications         |
| **scheduler-service**    | Background Jobs                   |
| **admin-service**        | Administration                    |
| **tenant-service**       | Multi-tenant Configuration        |
| **common-library**       | Shared Prisma, Utilities, DTOs    |
| **MariaDB**              | Data Storage & Business Logic     |

---

# Key Takeaways

As a Database Developer, you should think in layers:

```text
User
  │
  ▼
Angular UI
  │
  ▼
Authentication
  │
  ▼
Business Service
  │
  ▼
Prisma (common-library)
  │
  ▼
MariaDB
  │
  ▼
Triggers
  │
  ▼
Stored Procedures
  │
  ▼
Tables
```

This layered view helps you quickly identify where a problem belongs. For example:

* Login issue → `auth-service`
* Survey submission issue → `survey-service`
* Email not received → `notification-service`
* Patient data issue → `patient-service`
* Data not stored or calculated correctly → MariaDB (tables, triggers, stored procedures)

This updated Step 3 now aligns with the expanded microservices architecture introduced in the updated Step 2, while still preserving the detailed database processing flow that is most relevant to your role as a Database Developer.
