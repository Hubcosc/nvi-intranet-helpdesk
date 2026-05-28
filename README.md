> # 🔒 SOURCE CODE PRIVATE
>
> <p align="center">
>   <strong>The source code in this repository is not publicly shared.</strong><br>
>   <em>Only screenshots and documentation are displayed for portfolio/reference purposes.</em>
> </p>

---

<p align="center"> <img src="https://via.placeholder.com/800x5/2c3e50/2c3e50" alt="divider" width="800" height="4"> </p>

<h1 align="center">🏢 NVI Internal Management System</h1>

<p align="center"> <strong>Enterprise intranet + IT help desk platform for the National Veterinary Institute (NVI), Bishoftu, Ethiopia</strong> </p>

<p align="center"> 
  <img src="https://img.shields.io/badge/Status-Production-brightgreen?style=for-the-badge" alt="Status"> 
  <img src="https://img.shields.io/badge/Django-5.2.4-092E20?style=for-the-badge&logo=django&logoColor=white" alt="Django"> 
  <img src="https://img.shields.io/badge/PostgreSQL-14+-336791?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL"> 
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"> 
  <img src="https://img.shields.io/badge/Bootstrap%20CSS-5-7952B3?style=for-the-badge&logo=bootstrap&logoColor=white" alt="Bootstrap CSS"> 
  <img src="https://img.shields.io/badge/Team-Solo%20Full--Stack-ff6b35?style=for-the-badge" alt="Team"> 
</p>

---

## 📋 Overview

| | |
|---|---|
| **Status** | ✅ Deployed in production |
| **Role** | Solo full-stack developer |
| **Stack** | Django · PostgreSQL · Bootstrap CSS · HTML · JavaScript · jQuery |
| **Deployment** | Windows Server 2012 R2 + IIS |
| **Apps** | `helpdesk` · `intranet` |

---

## 🎯 Core Features

| Module | Key Features |
|--------|--------------|
| **Help Desk** | Ticket creation · Priority levels (Critical/High/Medium/Low) · Status tracking (Incoming/Assigned/Ongoing/Resolved/Unresolved/Closed) · Technician assignment · Reply system · Progress notes |
| **Issue Categorization** | Category → Subcategory → Issue Type → Options · Custom issue types |
| **Asset Management** | Asset types (Computer/Laptop/Printer/Network/Software) · Status (Available/Assigned/Damaged/Retired/Lost) · Location tracking · Warranty · Bulk quantity |
| **Asset Assignments** | Purpose-based (Support/General/Temporary/Replacement) · Return dates · Condition tracking · Overdue detection |
| **Stock Requests** | Available/Unavailable assets · Approval workflow · ERP integration |
| **Employee Profiles** | Roles (Employee/Admin/Technician/Head/Poster/Sport Poster/IT Manager) · Department & position · Retired status |
| **Messaging** | Conversations · Direct messages · Read/unread · File attachments |
| **Technician Status** | Online/Busy/Away/Offline · Conversation limits · Availability hours |
| **Notifications** | Email/Push/SMS channels · Event-based triggers · Frequency settings · Quiet hours |
| **Intranet Content** | Pages · News · Announcements · Media gallery · Documents with version control · Photo albums |
| **Content Approval** | Pending/Approved/Rejected workflow for all content types |
| **Sports Tracking** | Matches · Scores · Stages (Group/Quarter/Semi/Final) · League standings |
| **Event Calendar** | Categories · Date/time · Location · Recurring events |
| **Activity Logging** | All actions tracked with user and timestamp |
| **Knowledge Base** | Issue catalog with usage counter and view tracking |

---

## Technology Stack

| Layer | Technology |
|-------|------------|
| Backend | Django 4.x (Python) |
| Database | PostgreSQL 14+ |
| Frontend | Bootstrap 5, JavaScript, HTML, jQuery |
| Authentication | Django User + Profile (one-to-one) |
| File Storage | Django FileSystemStorage |
| Deployment | Windows Server 2012 R2 + IIS |

---

## 🗂 System Architecture

```
📦 NVI Internal Management System
│
├── 🎫 helpdesk (app)
│   ├── Profile ──── OneToOne ──► User (django.contrib.auth)
│   ├── TechnicianStatus ──── OneToOne ──► Profile
│   ├── IssueCategory → IssueSubcategory → IssueType → IssueOption
│   │                                               └── IssueCatalog → CatalogView
│   ├── SupportRequest → ActivityLog, AssetAssignment, StockRequest
│   ├── Conversation (M2M Profiles) → DirectMessage
│   └── Notification → Profile, SupportRequest, Conversation
│
└── 🏢 intranet (app)
    ├── Announcement, News, Document → DocumentVersion
    ├── PhotoAlbum → Photo
    ├── EmployeeDirectory ──── OneToOne ──► Profile (auto-synced via signal)
    ├── Match, Standing (auto-recalculated)
    ├── Event (with recurrence)
    ├── ContentApproval (covers all content types)
    └── Page, Media
```
## 🔑 Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Separate `helpdesk` / `intranet` apps | Clean separation; independently maintainable |
| `Profile` one-to-one with `User` | Extends auth without overriding Django's permission system |
| `EmployeeDirectory` via `post_save` signal | Always in sync with Profile — no manual steps |
| `JSONField` for specs & notification exclusions | Flexible per-record data without extra migrations |
| `available_quantity` as `@property` | Live aggregation — no stale cached counts |
| State machines in `clean()` / `save()` | Status transitions enforced at model layer |
| Soft-delete: `is_archived` + `marked` | Staff archive vs. employee hide — preserves audit trail |
| `ContentApproval` as single polymorphic model | One workflow for all content types |
| `IssueType(is_custom=True)` | Employees describe novel issues without admin creating categories |
| `TechnicianStatus.can_accept_new` property | Prevents over-assignment; respects shift hours without cron jobs |

---

## 📞 Contact

<div align="center">

<a href="mailto:boxctf76@gmail.com">
  <img src="https://img.shields.io/badge/Email-boxctf76%40gmail.com-red?style=for-the-badge&logo=gmail&logoColor=white">
</a>
&nbsp;
<a href="https://linkedin.com/in/abduselam-kedir-5945732a6" target="_blank">
  <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
</a>

</div>
