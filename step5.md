Excellent. Since we've updated **Steps 2, 3, and 4**, the old Step 5 no longer fits. Step 5 should now focus on the **actual repository structure** and explain how each repository is organized internally. This is exactly what a new developer joining the project needs to understand.

---

# Step 5 – Repository Architecture & Code Organization

Now that we understand the business flow and request lifecycle, let's learn the source code organization.

The SPRI platform is built using a **microservices architecture**. Each repository has a single responsibility, but together they provide the complete application.

---

# Overall Repository Architecture

```text
                               SPRI Platform
                                     │
    ┌──────────────┬──────────────┬──────────────┬──────────────┐
    │              │              │              │
    ▼              ▼              ▼              ▼
spri-client   auth-service  patient-service  survey-service
    │              │              │              │
    ├──────────────┼──────────────┼──────────────┤
    ▼              ▼              ▼
admin-service tenant-service notification-service
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
```

---

# Repository Responsibilities

## 1. spri-client

### Technology

Angular

### Purpose

Provides the web application used by:

* Doctors
* Receptionists
* Clinical Staff
* Administrators

Patient does **not** use this application.

Typical modules

```text
Login

Dashboard

Patients

Appointments

Surveys

Reports

Administration
```

---

## 2. auth-service

### Technology

NestJS

### Purpose

Authentication and Authorization.

Responsibilities

* Login
* Logout
* JWT Token
* Password validation
* User authentication
* Role validation
* Permission checks

Flow

```text
Angular

↓

auth-service

↓

JWT Token
```

---

## 3. patient-service

This is the main business service.

Responsibilities

```text
Patients

Appointments

Medical History

Dashboard

Clinical Data

Reports
```

Typical API examples

```http
GET /patients

POST /patients

PUT /patients/{id}

GET /appointments

GET /dashboard
```

---

## 4. survey-service

Dedicated to survey operations.

Responsibilities

```text
Survey Assignment

Survey Processing

Survey Validation

Survey Submission

FormSite Integration
```

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

Responsible for all communications.

Responsibilities

```text
Email

SMS

Survey Invitation

Appointment Reminder

Notifications
```

Example

```text
Survey Assigned

↓

notification-service

↓

Email Sent
```

---

## 6. scheduler-service

Executes scheduled jobs.

Examples

```text
Every Day

↓

Reminder Emails
```

```text
Every Hour

↓

Pending Survey Check
```

```text
Weekly

↓

Maintenance Jobs
```

Unlike a Trigger, which executes immediately after a database event, the Scheduler executes based on time.

---

## 7. admin-service

Used by administrators.

Responsibilities

```text
Users

Roles

Clinics

Departments

Configuration

Administration
```

---

## 8. tenant-service

Supports multi-tenant deployments.

Responsibilities

```text
Hospital Configuration

Tenant Configuration

Branding

Domain Management

Feature Settings
```

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

---

## 9. common-library

This repository is shared by all backend services.

Responsibilities

```text
Prisma Schema

Database Models

Shared DTOs

Utilities

Logging

Validation

Constants
```

Instead of duplicating code, services import from this repository.

---

# Common Folder Structure

Most backend repositories follow a similar structure.

```text
repository-name/

├── src/
│   ├── controllers/
│   ├── services/
│   ├── modules/
│   ├── dto/
│   ├── entities/
│   ├── middleware/
│   ├── guards/
│   ├── interceptors/
│   ├── filters/
│   ├── config/
│   └── main.ts
│
├── prisma/
│   ├── schema.prisma
│   └── migrations/
│
├── test/
│
├── Dockerfile
├── docker-compose.yml
├── package.json
├── tsconfig.json
└── README.md
```

---

# Understanding Each Folder

## controllers/

Receives HTTP requests.

Example

```http
GET /patients
```

↓

Controller

---

## services/

Contains business logic.

Example

```text
Create Patient

Assign Survey

Calculate Statistics
```

Controller calls Service.

---

## modules/

Groups related functionality.

Example

```text
Patient Module

Survey Module

Auth Module
```

---

## dto/

Data Transfer Objects.

Used for request validation.

Example

```text
CreatePatientDto

UpdatePatientDto

LoginDto
```

---

## entities/

Represents database models or domain objects.

---

## middleware/

Runs before the request reaches the controller.

Examples

* Logging
* Authentication
* Request modification

---

## guards/

Checks permissions.

Example

```text
Doctor

↓

Can Access Dashboard

YES
```

or

```text
Receptionist

↓

Delete User

NO
```

---

## interceptors/

Executes before and after request processing.

Examples

* Logging
* Response formatting
* Execution time measurement

---

## filters/

Handles exceptions.

Example

```text
Database Error

↓

Exception Filter

↓

HTTP Response
```

---

## config/

Application configuration.

Examples

```text
Database

JWT

Email

Environment Variables
```

---

## prisma/

Contains

```text
schema.prisma

Migrations

Database Models
```

This is one of the most important folders for Database Developers.

---

# Database Layer

Every backend service eventually reaches the same database layer.

```text
Controller

↓

Service

↓

Prisma

↓

MariaDB
```

---

# Repository Communication

```text
Angular

↓

auth-service

↓

patient-service

↓

common-library

↓

Prisma

↓

MariaDB
```

Another example

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
```

---

# Backend Service Communication

Sometimes services work together.

Example

```text
patient-service

↓

notification-service

↓

Email Sent
```

Another

```text
scheduler-service

↓

notification-service

↓

Reminder Emails
```

---

# Complete Repository Interaction

```text
                           Angular
                               │
                     Authentication
                               │
                               ▼
                        auth-service
                               │
      ┌────────────────────────┼────────────────────────┐
      ▼                        ▼                        ▼
patient-service          survey-service          admin-service
      │                        │                        │
      ├──────────────┐          │                        │
      ▼              ▼          ▼                        ▼
notification-service tenant-service scheduler-service
           │               │
           └───────────────┴──────────────┐
                                          ▼
                                   common-library
                                          │
                                       Prisma ORM
                                          │
                                       MariaDB
```

---

# Which Repository Should You Open?

| Requirement                         | Repository             |
| ----------------------------------- | ---------------------- |
| Login issue                         | `auth-service`         |
| Patient creation                    | `patient-service`      |
| Appointment issue                   | `patient-service`      |
| Survey submission                   | `survey-service`       |
| Survey invitation email             | `notification-service` |
| Reminder jobs                       | `scheduler-service`    |
| User & role management              | `admin-service`        |
| Tenant configuration                | `tenant-service`       |
| Prisma schema or shared utilities   | `common-library`       |
| Tables, Stored Procedures, Triggers | MariaDB                |

---

# Daily Workflow for a Database Developer

When you're assigned a task, this decision tree helps you know where to start.

```text
Requirement Received
        │
        ▼
Is it a Database Change?
        │
   ┌────┴────┐
   │         │
  Yes        No
   │         │
   ▼         ▼
MariaDB   Identify Service
   │         │
   ▼         ▼
Tables   auth / patient / survey /
Indexes  admin / tenant / notification /
SPs      scheduler
Triggers
Functions
Views
```

---

# Key Takeaways

By now you should understand:

* The SPRI platform is composed of **multiple independent microservices**, each with a clear responsibility.
* **Angular (`spri-client`)** is the frontend used by hospital staff.
* **NestJS services** implement business capabilities such as authentication, patient management, surveys, notifications, administration, scheduling, and tenant management.
* **`common-library`** provides shared code, including Prisma models and utilities, so backend services remain consistent.
* **MariaDB** is the central data store where business data, stored procedures, triggers, functions, and indexes reside.
* As a **Database Developer**, your primary work is in MariaDB and the Prisma-related database layer, but understanding which service owns a feature helps you quickly locate the right code when investigating or implementing changes.

This updated Step 5 now aligns with the complete microservice architecture and provides a practical guide for navigating the codebase.
