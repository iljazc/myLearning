# myLearning
# myLearning

A web-based Learning Management and Learner Progress Tracking platform for Level 2 Electrical Installation learners.

---

# Project Overview

The project originally started as a Claude-generated artifact that relied on:

```javascript
window.claude
```

for:

- User Authentication
- Database Access
- Downloads

The system has now been migrated to:

- **Vercel** (Hosting)
- **GitHub** (Source Control)
- **Supabase** (Authentication, Database & Future File Storage)

---

# Live Environment

## Hosting

Platform:

```text
Vercel
```

URL:

```text
https://mylearning-fawn.vercel.app
```

---

## Source Control

Platform:

```text
GitHub
```

Repository:

```text
myLearning
```

---

# Architecture

```text
User
 │
 ▼
Login Page
 │
 ▼
Supabase Auth
 │
 ▼
Dashboard
 │
 ├── Learners
 │
 ├── Activities
 │
 ├── Assessments
 │
 └── Progress Tracking
 │
 ▼
Supabase Database
```

---

# Supabase Setup

## Authentication

Enabled:

```text
Email / Password
```

Login page:

```text
login.html
```

Authentication flow:

```text
Login
   ↓
Supabase Session
   ↓
Dashboard Access
```

---

# Database Schema

## learners

```sql
create table learners (
    id text primary key,
    name text,
    learner_group text,
    target text,

    units jsonb,
    acts jsonb,
    tests jsonb,
    notes jsonb,
    hub jsonb,

    created bigint,
    updated bigint
);
```

### Purpose

Stores:

- Learner Information
- Unit Progress
- Activities
- Assessments
- Notes
- Targets
- Learning Hub Statistics

---

## config

```sql
create table config (
    id text primary key,
    activities jsonb,
    assessments jsonb
);
```

Initial record:

```sql
insert into config
(
    id,
    activities,
    assessments
)
values
(
    'main',
    '[]'::jsonb,
    '[]'::jsonb
);
```

### Purpose

Stores:

- Activity Definitions
- Assessment Definitions

---

## profiles

```sql
create table profiles (
    id uuid primary key,
    email text unique,
    role text,
    created_at timestamptz
);
```

### Planned Roles

```text
super_admin
staff
student
```

---

# Claude → Supabase Migration

## Authentication

### Removed

```javascript
window.claude.use("user")
```

### Replaced With

```javascript
const {
  data: { session }
} = await sb.auth.getSession();
```

---

## Database Reads

### Removed

```javascript
DB.doc()
DB.collection()
```

### Replaced With

```javascript
sb.from()
```

---

# Learner Management

## Create Learner

Migrated from:

```javascript
DB.doc().set(...)
```

To:

```javascript
sb
  .from("learners")
  .insert(...)
```

Status:

```text
✅ Working
```

---

## Read Learners

Migrated from:

```javascript
DB.collection("learners")
```

To:

```javascript
sb
  .from("learners")
  .select("*")
```

Status:

```text
✅ Working*```

---

## Update Learner

Migra*ed from:

```javascript
DB.doc().u*date(...)
```

To:

```javascript
*b
  .from("learners")
  .update(..*)
  .eq("id", id)
```

Status:

``*text
✅ Working
```

---

## Delete*Learner

Migrated from:

```javasc*ipt
DB.doc().delete()
```

To:

``*javascript
sb
  .from("learners")
* .delete()
  .eq("id", id)
```

St*tus:

```text
✅ Working
```

---

* Progress Tracking

## Unit Status*s

```javascript
n = Not Started
p*= In Progress
d = Done
```

### Ex*mple

```json
{
  "201": "d",
  "2*2": "p",
  "204": "d"
}
```

---

*# Important Fix

### Previous Bug
*Only one unit progress value was s*ved.

Example:

```json
{
  "201":*"d"
}
```

became:

```json
{
  "2*2": "p"
}
```

and Unit 201 progre*s was lost.

---

### Fix Applied
*Merged existing values before upda*ing:

```javascript
Object.assign(*}, l.units)
```

instead of overwr*ting the entire object.

Status:

*``text
✅ Fixed
```

---

# Activit* Tracking

Stores:

```json
{
  "a*tivity_id": {
    "s": "d",
    "d*: 1761751000000
  }
}
```

Status:*
```text
✅ Working
```

---

# Ass*ssment Tracking

Stores:

```json
*
  "assessment_id": {
    "score":*34,
    "date": 1761751000000
  }
*
```

Status:

```text
✅ Working
`*`

---

# Group Tracking

Database*column:

```sql
learner_group
```
*Dashboard expected:

```javascript*group
```

Compatibility layer add*d:

```javascript
group: l.learner*group || ""
```

Status:

```text
* Working
```

---

# Configuration*Management

## Activities

Stored *n:

```sql
config.activities
```

*tatus:

```text
✅ Working
```

---*
## Assessments

Stored in:

```sq*
config.assessments
```

Status:

*``text
✅ Working
```

---

## saveCfg()

Migrated from:

```javascript
DB.doc("config/main").set(...)
```

To:

```javascript
sb
  .from("config")
  .update(...)
  .eq("id","main")
```

Status:

```text
✅ Working
```

---

# Row Level Security (RLS)

## learners

Policies created for:

```text
SELECT
INSERT
UPDATE
DELETE
```

Role:

```text
authenticated
```

---

## config

Policies created for:

```text
SELECT
UPDATE
```

Role:

```text
authenticated
```

---

# Current Feature Status

## Authentication

- ✅ Login Page
- ✅ Session Protection
- ✅ Supabase Auth

---

## Learners

- ✅ Add Learner
- ✅ Edit Learner
- ✅ Delete Learner
- ✅ Import Learners
- ✅ Group Management

---

## Progress

- ✅ Unit Tracking
- ✅ Activity Tracking
- ✅ Assessment Tracking
- ✅ Progress Persistence

---

## Dashboard

- ✅ Learner Dashboard
- ✅ Needs Attention View
- ✅ Activity Management
- ✅ Assessment Management

---

## Infrastructure

- ✅ Vercel Hosting
- ✅ GitHub Source Control
- ✅ Supabase Database
- ✅ Supabase Authentication

---

# Next Development Task

## File Uploads (Planned)

### Goal

Allow tutors to attach:

```text
PDF
DOCX
```

to activities.

---

## Supabase Storage

Planned bucket:

```text
learning-files
```

---

## Activity Structure

Current:

```json
{
  "id": "a1",
  "title": "Worksheet",
  "unit": "201"
}
```

Future:

```json
{
  "id": "a1",
  "title": "Worksheet",
  "unit": "201",
  "fileName": "HealthAndSafety.pdf",
  "fileUrl": "https://..."
}
```

---

## Dashboard Enhancement

Activities will support:

```text
View PDF
Download DOCX
```

links directly from the dashboard.

---

# Future Roadmap

## Roles & Permissions

Implement:

```text
super_admin
staff
student
```

Dashboard access:

```text
super_admin ✅
staff ✅
student ❌
```

---

## Learning Hub Integration

Current:

```text
Learning Hub
Dashboard
```

operate separately.

Planned:

```text
Learner completes activity
          ↓
Supabase
          ↓
Dashboard updates automatically
```

---

# Development Milestone

✅ Successfully migrated the dashboard from Claude's proprietary APIs to a standalone Supabase-backed application.

The platform now supports:

- Authentication
- Persistent Database Storage
- Learner CRUD
- Progress Tracking
- Configuration Management
- Deployment on Vercel

with no dependency on Claude's database services.