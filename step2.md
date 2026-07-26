Perfect. Since you've updated **Step 1** with the new services, let's make **Step 2** more accurate based on the complete architecture instead of the simplified version.

---

# Step 2 – SPRI System Architecture (Detailed)

Before learning tables or SQL, understand **who talks to whom**.

A database developer should always know:

* Which service owns which functionality?
* Which service writes to the database?
* Which service reads from the database?
* Which service communicates with another service?

---

# High-Level Architecture

```text
                        Users
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
 Receptionist          Doctor             Patient
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                     Angular UI
                     (spri-client)
                           │
                    Authentication
                           │
                           ▼
                    auth-service
                    (JWT / Roles)
                           │
                           ▼
                    API Gateway*
                           │
 ┌────────────┬────────────┼────────────┬────────────┬────────────┐
 ▼            ▼            ▼            ▼            ▼
patient     survey      admin      tenant     notification
service     service     service     service      service
      │          │            │            │
      └──────────┴────────────┴────────────┘
                    │
            scheduler-service
                    │
             common-library
                    │
                 Prisma ORM
                    │
                 MariaDB
```

> *If your project doesn't use a dedicated API Gateway, the UI communicates directly with each backend service.

---

# Components Overview

## 1. spri-client

### Type

Frontend Application

### Technology

Angular

### Responsibilities

* Login screen
* Patient search
* Dashboard
* Appointment screens
* Survey assignment
* Reports
* Administration

---

## 2. auth-service

### Type

Backend API

### Technology

NestJS

### Responsibilities

* User login
* JWT generation
* User authentication
* Authorization
* Role validation
* Token verification

Without authentication, no other service should be accessed.

---

## 3. patient-service

This is one of the core business services.

Responsibilities include:

* Patient management
* Appointments
* Medical history
* Dashboard data
* Patient profile
* Reading calculated scores

Typical flow:

```text
Doctor

↓

patient-service

↓

MariaDB
```

---

## 4. survey-service

Dedicated to surveys.

Responsibilities

* Survey creation
* Survey assignment
* Receiving survey responses
* Survey validation
* Survey status

Flow

```text
FormSite

↓

survey-service

↓

Database
```

---

## 5. notification-service

This service communicates with users.

Responsibilities

* Email
* SMS
* Survey invitations
* Appointment reminders
* Notifications

Example

```text
Assign Survey

↓

notification-service

↓

Email

↓

Patient
```

---

## 6. scheduler-service

Runs background jobs automatically.

Examples

```text
12:00 AM

↓

Reminder Emails
```

```text
Every Hour

↓

Pending Survey Check
```

```text
Every Day

↓

Generate Reports
```

---

## 7. admin-service

Administrative operations.

Examples

* User management
* Roles
* Physicians
* Clinics
* Departments
* System configuration

Usually accessed only by administrators.

---

## 8. tenant-service

This service indicates the application supports multiple organizations.

Responsibilities

* Tenant configuration
* Hospital configuration
* Branding
* Domain settings
* Feature configuration

Example

```text
Hospital A

↓

Tenant A
```

```text
Hospital B

↓

Tenant B
```

Each tenant can have different settings while using the same platform.

---

## 9. common-library

Shared code used by all backend services.

Contains

* Prisma schema
* Shared DTOs
* Utility functions
* Logging
* Validation
* Constants
* Common models

Instead of duplicating code, every service imports from here.

---

## 10. MariaDB

The central database.

Contains

* Tables
* Views
* Indexes
* Stored Procedures
* Functions
* Triggers
* Events

Every service ultimately reads or writes data here through Prisma or database logic.

---

# Request Flow Example – Patient Search

```text
Doctor

↓

Angular UI

↓

auth-service
(Token Validation)

↓

patient-service

↓

Prisma

↓

MariaDB

↓

Patient Data Returned

↓

Angular UI
```

---

# Request Flow Example – Survey Submission

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

Score Calculation

↓

Database Updated
```

---

# Request Flow Example – Survey Assignment

```text
Doctor

↓

Angular UI

↓

patient-service

↓

Survey Created

↓

notification-service

↓

Email Sent

↓

Patient Receives Survey Link
```

---

# Overall Communication Diagram

```text
                      Angular UI
                           │
                    auth-service
                           │
      ┌────────────┬─────────────┬─────────────┐
      ▼            ▼             ▼             ▼
patient-service survey-service admin-service tenant-service
      │            │             │             │
      └────────────┴─────────────┴─────────────┘
                    │
         notification-service
                    │
         scheduler-service
                    │
            common-library
                    │
                 Prisma ORM
                    │
                 MariaDB
```

---

# Responsibilities Matrix

| Component            | Responsibility                   | Database Access               |
| -------------------- | -------------------------------- | ----------------------------- |
| spri-client          | User Interface                   | ❌ No                          |
| auth-service         | Authentication & Authorization   | ✅ Yes                         |
| patient-service      | Patient & Appointment Management | ✅ Yes                         |
| survey-service       | Survey Processing                | ✅ Yes                         |
| notification-service | Email & SMS Notifications        | Usually for logging/templates |
| scheduler-service    | Scheduled Background Jobs        | ✅ Yes                         |
| admin-service        | System Administration            | ✅ Yes                         |
| tenant-service       | Multi-tenant Configuration       | ✅ Yes                         |
| common-library       | Shared Prisma & Utilities        | Used by all services          |
| MariaDB              | Data Storage & Business Logic    | Core database                 |

---

## What you should remember from Step 2

Think of the system in three layers:

```text
Presentation Layer
    ↓
spri-client (Angular)

Business Layer
    ↓
auth-service
patient-service
survey-service
admin-service
tenant-service
notification-service
scheduler-service

Data Layer
    ↓
common-library
Prisma ORM
MariaDB
```

This layered architecture is the foundation for everything else. As we continue, every API, table, stored procedure, and Jira issue can be placed into one of these layers, making it much easier to understand where a problem or feature belongs.

For **Step 3**, we'll drill into the **request lifecycle**—following a single user action (such as "Create Patient" or "Assign Survey") across all these services until the data is committed to the database. This will help you understand exactly how the microservices collaborate.
