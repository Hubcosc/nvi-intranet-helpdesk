# NVI Internal Management System

> **Enterprise intranet + IT help desk platform** for the National Veterinary Institute (NVI), Bishoftu, Ethiopia.

![Status](https://img.shields.io/badge/Status-Production-brightgreen) ![Django](https://img.shields.io/badge/Django-4.x-092E20?logo=django) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14+-336791?logo=postgresql) ![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python) ![Bootstrap](https://img.shields.io/badge/Bootstrap-5-7952B3?logo=bootstrap)

---

## Overview

A complete digital operations platform serving **200+ employees** across **5 departments**. Replaced manual paper-based workflows with an integrated system combining IT help desk ticketing, asset management, employee directory, content management, sports event tracking, and a full intranet portal.

| | |
|---|---|
| **Status** | ✅ Deployed in production |
| **Role** | Solo full-stack developer |
| **Stack** | Django · PostgreSQL · Bootstrap 5 · HTMX · JavaScript |
| **Deployment** | Windows Server 2022 + IIS |
| **Apps** | `helpdesk` · `intranet` |

---

## Table of Contents

- [Core Systems](#core-systems)
  - [IT Help Desk](#1-it-help-desk-module)
  - [Asset Management](#2-asset-management-system)
  - [Intranet Portal](#3-intranet-portal)
  - [User & Notification System](#4-user--notification-system)
- [Technology Stack](#technology-stack)
- [Database Schema Architecture](#database-schema-architecture)
- [Model Index Summary](#model-index-summary)
- [Key Architectural Decisions](#key-architectural-decisions)
- [Challenges & Solutions](#challenges--solutions)
- [Code Metrics](#code-metrics)
- [Future Improvements](#future-improvements)

---

## Core Systems

### 1. IT Help Desk Module

Built on Django with structured ticketing, technician assignment, and a full conversation system.

| Feature | Detail |
|---------|--------|
| **Ticket hierarchy** | 4-level cascade: `IssueCategory` → `IssueSubcategory` → `IssueType` → `IssueOption` |
| **Custom issue types** | Employees can submit `IssueType(is_custom=True)` — auto-routed to a "User Defined" subcategory |
| **Priority levels** | `CRITICAL` / `HIGH` / `MEDIUM` / `LOW` |
| **Status workflow** | `INCOMING` → `ASSIGNED` → `ONGOING` → `RESOLVED` / `UNRESOLVED` / `PENDING_ERP` / `PENDING_ASSET` → `CLOSED` |
| **Unresolved tracking** | Stores `unresolved_reason`, `unresolved_by`, `unresolved_at` separately from resolution fields |
| **Technician availability** | `TechnicianStatus` model: `ONLINE` / `BUSY` / `AWAY` / `OFFLINE`, configurable `available_from`/`available_to`, `max_conversations` cap |
| **Load balancing** | `can_accept_new` property enforces concurrent conversation limits per technician |
| **Direct messaging** | `Conversation` + `DirectMessage` models; supports `TEXT`, `SYSTEM`, and `FILE` message types with file attachments |
| **Issue catalog** | `IssueCatalog` knowledge base with `use_count`, `unique_view_count`, and `CatalogView` per-user tracking |
| **Activity logging** | `ActivityLog` captures: `CREATED`, `ASSIGNED`, `UPDATED`, `RESOLVED`, `REOPENED`, `COMMENT`, `PRIORITY_CHANGED`, `STATUS_CHANGED`, `MESSAGE_SENT`, `CONVERSATION_STARTED` |
| **Soft delete** | `SupportRequest` uses `is_archived` + `marked` (employee-side hide) for data retention |
| **Reply tracking** | `replied_by` and `replied_at` stored on each ticket |
| **Progress notes** | Internal `progress_notes` field visible to staff only |

**Ticket status transitions (from actual model):**

```
INCOMING
  └── ASSIGNED
        └── ONGOING
              ├── RESOLVED        (resolved_at auto-set; unresolved fields cleared)
              ├── UNRESOLVED      (unresolved_at auto-set; unresolved_reason required)
              ├── PENDING_ERP
              └── PENDING_ASSET
                    └── CLOSED
```

---

### 2. Asset Management System

Full lifecycle inventory tracking — from warehouse acceptance through assignment, return, and decommissioning.

**Models:**

```
Asset
  ├── AssetAssignment  (one asset → many assignments over time)
  └── StockRequest     (technician requests stock from an active ticket)
```

**Asset types** (from `ASSET_TYPE_CHOICES`):

`COMPUTER` · `LAPTOP` · `PRINTER` · `MONITOR` · `KEYBOARD` · `MOUSE` · `PHONE` · `TABLET` · `NETWORK` · `SOFTWARE` · `FURNITURE` · `VEHICLE` · `TOOL` · `OTHER`

**Asset statuses:**

`AVAILABLE` · `ASSIGNED` · `DAMAGED` · `MAINTENANCE` · `RETIRED` · `LOST` · `PENDING_REPAIR` · `IN_TRANSIT`

**Locations:**

`WAREHOUSE` · `IT_DEPARTMENT` · `HR_DEPARTMENT` · `FINANCE` · `OPERATIONS` · `SALES` · `MARKETING` · `EXECUTIVE` · `REMOTE` · `STORAGE` · `REPAIR_CENTER` · `OTHER`

| Feature | Detail |
|---------|--------|
| **Quantity-aware assignment** | `available_quantity` property aggregates unreturned `AssetAssignment` quantities dynamically |
| **Acceptance workflow** | `is_accepted` + `accepted_at` + `accepted_by` — assets must be accepted into inventory before use |
| **Warranty tracking** | `warranty_expiry_date` with `is_under_warranty` property |
| **Maintenance scheduling** | `last_maintenance_date` + `next_maintenance_date` with validation |
| **Decommissioning** | `decommissioned_at` + `decommission_reason` for retired/lost/damaged assets |
| **Technical specs** | `specifications` stored as `JSONField` for flexible per-asset metadata |
| **Assignment purposes** | `SUPPORT` (linked to ticket) · `GENERAL` · `TEMPORARY` · `REPLACEMENT` |
| **Return tracking** | `returned_date`, `condition_on_return`, `returned_to` (who received it back) |
| **Overdue detection** | `is_overdue()` checks `expected_return_date` against today |
| **Stock request workflow** | `PENDING` → `APPROVED` / `PENDING_ERP` → `REJECTED` / `CANCELLED` |
| **ERP integration hook** | `erp_reference` + `erp_processed_at` for external procurement system handoff |
| **Unavailable asset requests** | Technicians can request assets not yet in inventory (`request_type=UNAVAILABLE`) with `unavailable_asset_name` + specs |

**Stock request state machine (enforced in `clean()`):**

```
PENDING  →  APPROVED
         →  REJECTED
         →  CANCELLED
         →  PENDING_ERP  →  APPROVED
                          →  REJECTED
                          →  CANCELLED
```

---

### 3. Intranet Portal

| Model | Key Fields & Behaviour |
|-------|------------------------|
| **`Announcement`** | Priority: `Urgent` / `General` · Status: `Published` / `Draft` / `Archived` · Optional `expiry_date` · Department targeting · M2M to `Media` |
| **`News`** | Three content areas: `top_description`, `middle_description`, `bottom_description` · `is_featured` flag · M2M to `Media` |
| **`Document`** | Version string (e.g. `1.0`) · Media types: `Document` / `Image` / `Video` / `Audio` · Full version history via `DocumentVersion` (unique per `document + version`) |
| **`PhotoAlbum` + `Photo`** | Album → Photo (cascade) · Per-photo `caption` · `ImageField` with date-partitioned upload path |
| **`Media`** | Shared media pool · Types: `Photo` / `Video` · Sections: `Top` / `Middle` / `Bottom` · File validation: `png`, `jpg`, `jpeg`, `mp4`, `avi` |
| **`EmployeeDirectory`** | One-to-one with `Profile` · Auto-synced via `post_save` signal on `Profile` · Stores `telephone`, `email`, `position_name`, `department`, `labor_data` |
| **`Match`** | Home/away teams (FK to `Department`) · Stages: `Regular Match` → `Group Stage` → `Quarter Finals` → `Semi Finals` → `Final` · Stores `scorers`, `cards` as text · Status: `Upcoming` / `Completed` |
| **`Standing`** | Auto-calculated from completed matches: `played`, `wins`, `draws`, `losses`, `points` (3/1/0) · `update_standings()` recomputes from scratch |
| **`ContentApproval`** | Single approval model covering: `News`, `Announcement`, `Document`, `Match`, `Event`, `PhotoAlbum` · Statuses: `pending` / `approved` / `rejected` · Tracks `date_submitted`, `approved_date`, `rejected_date` |
| **`Event`** | Categories: `meeting` / `training` / `social` / `conference` / `other` · Duration choices from 30 min to 5+ hours · Recurrence: `daily` / `weekly` / `monthly` / `yearly` with `recurrence_interval` and `recurrence_end_date` |
| **`Page`** | CMS static pages with `slug`, `content`, and `created_by` |

---

### 4. User & Notification System

```python
class Profile(models.Model):
    # Identity
    emp_id          = CharField(unique=True, db_index=True)
    identity_number = CharField(unique=True)
    person_id       = CharField(unique=True)
    first_name      = CharField()
    father_name     = CharField()

    # Role-Based Access Control
    ROLES = [
        ('employee',     'Employee'),      # Can submit tickets
        ('technician',   'Technician'),    # Can respond and resolve tickets
        ('head',         'Head'),          # Department head, approves content
        ('poster',       'Poster'),        # Publishes announcements and news
        ('sport_poster', 'Sport Poster'),  # Manages sports content
        ('it_manager',   'IT Manager'),    # Full system access
        ('admin',        'Admin'),         # Superuser
    ]

    # Notification Channels
    email_notifications = BooleanField(default=True)
    push_notifications  = BooleanField(default=True)
    sms_notifications   = BooleanField(default=False)

    # Per-event notification toggles
    notify_on_new_request    = BooleanField(default=True)
    notify_on_assignment     = BooleanField(default=True)
    notify_on_status_change  = BooleanField(default=True)
    notify_on_priority_change= BooleanField(default=True)
    notify_on_comment        = BooleanField(default=True)
    notify_on_resolution     = BooleanField(default=True)
    notify_on_escalation     = BooleanField(default=True)
    notify_on_announcement   = BooleanField(default=True)

    # Delivery frequency
    notification_frequency = CharField(choices=['IMMEDIATE','HOURLY','DAILY'])
    digest_frequency       = CharField(choices=['DISABLED','DAILY','WEEKLY'])

    # Quiet hours
    quiet_hours_enabled = BooleanField(default=False)
    quiet_hours_start   = TimeField(default='22:00')
    quiet_hours_end     = TimeField(default='08:00')

    # Category exclusions
    excluded_notification_categories = JSONField(default=list)

    # Messaging availability
    is_available_for_messaging = BooleanField(default=True)
    auto_away_minutes          = PositiveIntegerField(default=30)
```

**Notification routing logic** (`should_receive_notification()`):
1. Check per-event toggle (e.g. `notify_on_comment`)
2. Check `excluded_notification_categories` JSONField
3. Check quiet hours — supports overnight spans (e.g. 22:00–08:00)
4. Return `True` only if all three pass

**`TechnicianStatus` model:**

| Field | Detail |
|-------|--------|
| `status` | `ONLINE` / `BUSY` / `AWAY` / `OFFLINE` |
| `current_conversations` | Live count updated on assignment |
| `max_conversations` | Configurable cap (default: `5`) |
| `available_from` / `available_to` | Shift window (default: `08:00–17:00`) |
| `can_accept_new` | Property: `is_available AND status in [ONLINE, AWAY] AND current < max` |

**Notification model types:**

`REQUEST_UPDATE` · `ASSIGNMENT` · `SYSTEM` · `REMINDER` · `NEW_MESSAGE`

---

## Technology Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Backend | Django (Python) | 4.x |
| Database | PostgreSQL | 14+ |
| Frontend | Bootstrap 5, JavaScript, HTMX | — |
| Authentication | Django built-in `User` + one-to-one `Profile` with role-based permissions | — |
| File Storage | Django `FileSystemStorage` with date-partitioned upload paths | — |
| PDF Export | ReportLab | — |
| Deployment | Windows Server + IIS | 2022 |
| Version Control | Git (private repository) | — |
| Containerization | Docker | Planned |

---

## Database Schema Architecture

```
📦 NVI Internal Management System
│
├── 🎫 helpdesk (app)
│   │
│   ├── Department
│   ├── Position
│   ├── Profile  ──────────────────── OneToOne ──► User (django.contrib.auth)
│   ├── TechnicianStatus ──────────── OneToOne ──► Profile
│   │
│   ├── IssueCategory
│   │   └── IssueSubcategory
│   │         └── IssueType  (supports is_custom flag)
│   │               ├── IssueOption
│   │               └── IssueCatalog
│   │                     └── CatalogView  (unique per catalog+user)
│   │
│   ├── SupportRequest
│   │   ├── → IssueType, IssueOption
│   │   ├── → Profile (employee), User (assigned_to)
│   │   ├── ActivityLog
│   │   ├── AssetAssignment ──► Asset
│   │   └── StockRequest    ──► Asset
│   │
│   ├── Conversation  (M2M → Profile participants)
│   │   └── DirectMessage  (TEXT / SYSTEM / FILE)
│   │
│   └── Notification  ──► Profile, SupportRequest, Conversation
│
└── 🏢 intranet (app)
    │
    ├── Page  (CMS static pages)
    ├── Media  (shared Photo/Video pool, Top/Middle/Bottom sections)
    │
    ├── Announcement  ──► Department, Profile, Media (M2M)
    ├── News          ──► Department, Profile, Media (M2M)
    ├── Document      ──► Department, Profile
    │   └── DocumentVersion  (unique_together: document + version)
    │
    ├── PhotoAlbum  ──► Department, Profile
    │   └── Photo
    │
    ├── EmployeeDirectory  ──── OneToOne ──► Profile  (auto-synced via post_save signal)
    │
    ├── Match    ──► Department (home_team, away_team)
    ├── Standing ──► Department  (recalculated by update_standings())
    │
    ├── Event  (meeting/training/social/conference, with recurrence)
    │
    └── ContentApproval  (covers News, Announcement, Document, Match, Event, PhotoAlbum)
```

---

## Model Index Summary

All performance-critical indexes defined directly on models:

```python
# Profile
Index(['emp_id']), Index(['role']), Index(['department']),
Index(['is_available_for_messaging']), Index(['email_notifications']),
Index(['push_notifications']), Index(['notification_frequency'])

# SupportRequest
Index(['status', 'created_at']), Index(['employee', 'created_at']),
Index(['priority', 'status']), Index(['assigned_to', 'status'])

# Conversation
Index(['status', 'updated_at']), Index(['created_at'])

# DirectMessage
Index(['conversation', 'created_at']), Index(['sender', 'created_at']),
Index(['is_read'])

# Asset
Index(['asset_type']), Index(['status']), Index(['serial_number']),
Index(['location', 'status']), Index(['arrival_date']),
Index(['current_holder']), Index(['is_accepted'])

# AssetAssignment
Index(['asset', 'employee']), Index(['assigned_date']),
Index(['returned_date']), Index(['purpose', 'assigned_date']),
Index(['employee', 'purpose', 'returned_date']),
Index(['support_request', 'purpose']), Index(['expected_return_date'])

# StockRequest
Index(['status', 'created_at']), Index(['asset', 'status']),
Index(['technician', 'status']), Index(['support_request', 'status']),
Index(['priority', 'status']), Index(['request_type', 'status']),
Index(['created_at']), Index(['required_by_date'])

# IssueCatalog
Index(['issue_type']), Index(['status']),
Index(['priority']), Index(['created_at'])

# CatalogView
Index(['catalog', 'user']), Index(['viewed_at']),
Index(['catalog', '-viewed_at'])

# Notification
Index(['user', 'is_read']), Index(['timestamp']),
Index(['notification_type'])
```

---

## Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| Separate `helpdesk` and `intranet` Django apps | Clean separation of concerns; each app can be maintained and scaled independently |
| `Profile` one-to-one with Django's `User` | Extends auth without overriding it — preserves Django admin, sessions, and permission machinery |
| `EmployeeDirectory` auto-synced via `post_save` signal on `Profile` | Directory always reflects latest employee data without manual sync steps |
| `JSONField` for `excluded_notification_categories` and `specifications` | Flexible per-record data without additional migrations or join tables |
| `available_quantity` as a computed `@property` on `Asset` | Aggregates live `AssetAssignment` data; no risk of stale cached counts |
| State machine validation in `clean()` and `save()` | Enforces legal status transitions at the model layer, not just the view layer |
| `is_archived` + `marked` soft-delete pattern on `SupportRequest` | `is_archived` = staff-side archive; `marked` = employee hides from their own list — preserves audit trail |
| `ContentApproval` as a single polymorphic model | One approval workflow covers all content types (News, Announcements, Documents, Events, Matches, Albums) |
| `IssueType(is_custom=True)` with auto-routing in `save()` | Lets employees describe novel issues without admin creating new categories first |
| `TechnicianStatus.can_accept_new` property with shift window | Prevents over-assignment; respects working hours without cron jobs |

---

## Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| Unreliable internet connectivity at NVI | Offline-capable forms with `localStorage` synchronization |
| Non-technical staff struggled with UI | Tooltips, video tutorials, and simplified dashboard views |
| Managers needed custom reports | Dynamic PDF export using the `ReportLab` library |
| Asset quantity tracking across bulk items | `available_quantity` property aggregates live `AssetAssignment` data via `Sum()` — no cached fields to go stale |
| Stock requests for assets not yet in inventory | `StockRequest(request_type='UNAVAILABLE')` captures name + specs and routes to procurement without blocking the ticket |
| Multiple user roles with conflicting permissions | Custom role-based permission decorators per view; role checked via `profile.role` property |
| Technician overloading | `TechnicianStatus.can_accept_new` enforces `max_conversations` cap before auto-assignment |
| Content publishing without review chaos | Single `ContentApproval` model with `pending → approved / rejected` workflow covering all six content types |

---

## Code Metrics

| Component | Lines of Code (approx.) |
|-----------|------------------------|
| Models (`helpdesk` + `intranet`) | ~1,000 |
| Views / API | ~800 |
| Templates (HTML) | ~1,500 |
| JavaScript | ~400 |
| CSS / SCSS | ~300 |
| **Total** | **~4,000** |

---

## Future Improvements

- [ ] **Containerization** — Dockerize the application for consistent deployment across environments
- [ ] **REST API** — Migrate to Django REST Framework to support a future mobile app
- [ ] **Real-Time Notifications** — Replace polling with WebSockets via Django Channels
- [ ] **CI/CD Pipeline** — Automated testing and deployment with GitHub Actions
- [ ] **Unit Testing** — Increase model and view coverage from ~40% to 80%+
- [ ] **Full-Text Search** — Implement Elasticsearch for ticket and catalog search
- [ ] **Mobile / PWA** — Progressive Web App support for field staff
- [ ] **ERP Integration** — Complete the `erp_reference` / `erp_processed_at` handoff with the external procurement system

---

*Built solo as a full-stack developer for the National Veterinary Institute, Bishoftu, Ethiopia.*
