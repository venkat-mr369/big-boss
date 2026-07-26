Yes. Based on everything we've learned so far, I would **replace the old Step 7 completely**. The old Step 7 focused only on table relationships. Now we know the system is built with multiple microservices, so Step 7 should explain **how services own the database schema and how data flows across them**.

---

# Step 7 – Database Schema & Service Ownership

By now, we know the SPRI platform is not just one application connected to one database. It is a collection of **microservices** that share a common database architecture.

As a Database Developer, you need to answer two questions for every table:

1. **Which service owns this table?**
2. **Which other services read or update this table?**

Understanding service ownership makes debugging and development much easier.

---

# Database Architecture

```text
                           MariaDB
                              │
 ┌─────────────┬──────────────┬──────────────┬──────────────┐
 │             │              │              │
 ▼             ▼              ▼              ▼
Patient     Survey        Admin         Tenant
Tables       Tables       Tables         Tables
 │             │              │              │
 └─────────────┴──────────────┴──────────────┘
                │
         Shared Reference Tables
                │
                ▼
      Stored Procedures / Functions
                │
                ▼
        Views / Reports / Dashboard
```

Although all services use MariaDB, each service is primarily responsible for a specific functional area.

---

# Service Ownership

## auth-service

Owns authentication-related data.

Typical objects

```text
users

roles

permissions

user_role

role_permission
```

Responsibilities

* User login
* Password management
* Authentication
* Authorization

---

## patient-service

Owns patient-related business data.

Typical tables

```text
patient

appointment

patient_history

provider

clinic
```

Responsibilities

* Patient registration
* Appointment management
* Clinical history
* Dashboard information

---

## survey-service

Owns survey processing.

Typical tables

```text
survey_definition

patient_survey

form_response

survey_status
```

Responsibilities

* Survey assignment
* Survey submission
* Survey tracking

---

## notification-service

Owns notification-related data.

Typical tables

```text
notification

email_queue

sms_queue

notification_template
```

Responsibilities

* Email
* SMS
* Reminder tracking

---

## scheduler-service

Usually owns scheduled-job metadata.

Typical tables

```text
job_schedule

job_history

job_execution
```

Responsibilities

* Scheduled jobs
* Background processing
* Retry management

---

## admin-service

Owns administration data.

Typical tables

```text
department

physician

facility

organization_settings
```

Responsibilities

* Administrative configuration
* Master data
* User administration

---

## tenant-service

Owns tenant configuration.

Typical tables

```text
tenant

tenant_configuration

branding

domains
```

Responsibilities

* Multi-tenant configuration
* Branding
* Environment settings

---

# Shared Tables

Some tables are shared across multiple services.

Example

```text
patient
```

Used by

```text
patient-service

survey-service

notification-service
```

Example

```text
form_response
```

Written by

```text
survey-service
```

Read by

```text
patient-service
```

Processed by

```text
Stored Procedures
```

---

# Logical Database Flow

```text
Patient Created
       │
       ▼
patient
       │
       ▼
Appointment Created
       │
       ▼
appointment
       │
       ▼
Survey Assigned
       │
       ▼
patient_survey
       │
       ▼
Patient Completes Survey
       │
       ▼
form_response
       │
       ▼
Trigger
       │
       ▼
Stored Procedures
       │
       ▼
pro_score
       │
       ▼
Dashboard
```

---

# Cross-Service Communication

Although services have their own responsibilities, they collaborate.

```text
patient-service
        │
        ▼
notification-service
        │
        ▼
Email Sent
```

```text
survey-service
        │
        ▼
MariaDB
        │
        ▼
Stored Procedures
        │
        ▼
patient-service
```

```text
scheduler-service
        │
        ▼
notification-service
        │
        ▼
Reminder Emails
```

---

# Database Object Relationships

```text
patient
    │
    ├───────────────┐
    │               │
    ▼               ▼
appointment    patient_history
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

Notice something important.

The **patient** remains the center of the business model.

Everything ultimately relates back to the patient.

---

# Request Ownership

| Operation                 | Primary Service                  | Main Database Objects     |
| ------------------------- | -------------------------------- | ------------------------- |
| Login                     | auth-service                     | users, roles              |
| Create Patient            | patient-service                  | patient                   |
| Schedule Appointment      | patient-service                  | appointment               |
| Assign Survey             | patient-service + survey-service | patient_survey            |
| Submit Survey             | survey-service                   | form_response             |
| Calculate Clinical Scores | MariaDB (Stored Procedures)      | form_response → pro_score |
| Send Email                | notification-service             | notification, email_queue |
| Run Reminder Job          | scheduler-service                | job_schedule              |
| Configure Hospital        | tenant-service                   | tenant_configuration      |
| Manage Users              | admin-service                    | users, roles, departments |

---

# Repository vs Database Ownership

One mistake new developers often make is assuming every repository owns completely separate tables.

Instead, think of it like this:

```text
Repository
       │
       ▼
Business Responsibility
       │
       ▼
Primary Database Tables
       │
       ▼
Shared Database
```

For example:

```text
patient-service
       │
       ▼
patient

appointment

patient_history
```

Another service can **read** these tables, but **patient-service** is typically responsible for creating and maintaining them.

---

# Complete Data Flow

```text
Angular UI
      │
      ▼
auth-service
      │
      ▼
patient-service
      │
      ▼
survey-service
      │
      ▼
notification-service
      │
      ▼
scheduler-service
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
      │
      ▼
Triggers
      │
      ▼
Stored Procedures
      │
      ▼
Functions
      │
      ▼
Views
      │
      ▼
Dashboard
```

---

# Database Developer's Mindset

Whenever you're assigned a task, think in this order:

```text
Requirement
      │
      ▼
Which Service Owns It?
      │
      ▼
Which Tables Are Involved?
      │
      ▼
Which Stored Procedures Run?
      │
      ▼
Which APIs Read the Result?
      │
      ▼
Which UI Displays It?
```

This sequence will help you trace almost any feature or production issue through the system.

---

## One suggestion before finalizing Step 7

Now that you've uploaded additional repositories (`auth-service`, `admin-service`, `tenant-service`, `notification-service`, and `scheduler-service`), I recommend we inspect their actual **Prisma schema files**, **database diagrams**, and **migrations**. That will let us replace the example table names (such as `users`, `notification`, or `job_schedule`) with the **real table names used in your project**, making the training documentation match the codebase exactly.
