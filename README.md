# Microsoft Entra ID — SC-300 Hands-On Lab Project

A self-built identity and access management lab simulating a mid-size Canadian company, created while studying for the **Microsoft SC-300: Identity and Access Administrator** certification.

> **Tenant type:** Microsoft Entra ID (Workforce) — Free tier + Entra ID P2 (Microsoft 365 Developer Program trial)
> **Scenario company:** NorthBridge Technologies Inc. — a fictional Canadian tech company, 50 employees, offices in Toronto, Vancouver, Montreal, Edmonton, Winnipeg, and Halifax

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Lab Environment Setup](#1-lab-environment-setup)
3. [Users](#2-users)
4. [Groups](#3-groups)
5. [Role-Based Access Control (RBAC)](#4-role-based-access-control-rbac)
6. [App Registration](#5-app-registration)
7. [Conditional Access](#6-conditional-access)
8. [Privileged Identity Management (PIM)](#7-privileged-identity-management-pim)
9. [Identity Protection](#8-identity-protection)
10. [Key Concepts Learned](#key-concepts-learned)
11. [Issues Encountered & Fixes](#issues-encountered--fixes)
12. [SC-300 Exam Domain Mapping](#sc-300-exam-domain-mapping)
13. [Next Steps](#next-steps)

---

## Project Overview

This project simulates the identity infrastructure of a mid-size organization inside a real, free-tier Microsoft Entra ID tenant. Instead of just reading exam objectives, every concept below was **built, broken, debugged, and verified** inside the Entra admin center.

**Goal:** Cover as much of the SC-300 exam blueprint as possible using only:
- A free Microsoft Entra ID tenant
- A free 1-month Entra ID P2 trial (no recurring payment — cancelled before billing)

```
┌─────────────────────────────────────────────────────────┐
│                  NorthBridge Technologies Inc.            │
│                  tahirshaik.onmicrosoft.com                │
├─────────────────────────────────────────────────────────┤
│  50 Users  →  6 Security Groups  →  Role Assignments      │
│       ↓                                                    │
│  1 App Registration (NorthBridge Employee Portal)          │
│       ↓                                                    │
│  Conditional Access Policies (MFA enforcement)              │
│       ↓                                                    │
│  PIM (Just-In-Time privileged access)                      │
│       ↓                                                    │
│  Identity Protection (risk-based sign-in policies)          │
└─────────────────────────────────────────────────────────┘
```

---

## 1. Lab Environment Setup

### Tenant
| Property | Value |
|---|---|
| Tenant name | TH (Default Directory) |
| Domain | `tahirshaik.onmicrosoft.com` |
| Tenant type | Workforce |
| Base license | Microsoft Entra ID Free |
| Upgrade | Microsoft Entra ID P2 (1-month free trial) |

### Why P2 instead of Free-only
The Free tier doesn't support Conditional Access (full version), Identity Protection, or Privileged Identity Management — three of the most heavily tested SC-300 topics. A no-cost **1-month P2 trial** was activated for a single admin license to unlock these features for lab purposes.

> ⚠️ **Cost note:** The P2 trial requires payment details and auto-converts to a paid subscription if not cancelled. Only **1 license** was requested (not 50), and the trial must be cancelled before the renewal date to avoid charges.

### Alternative considered
The [Microsoft 365 Developer Program](https://developer.microsoft.com/microsoft-365/dev-program) offers a fully free 90-day E5 sandbox (no card required) and is worth trying first — it provides the same P2-level features without any billing risk.

---

## 2. Users

**50 fictional users** were created to simulate a real organizational headcount, distributed across 9 departments.

### Department Breakdown

| Department | Headcount | Example Roles |
|---|---|---|
| Engineering | 10 | CTO, Eng Manager, Software/DevOps/QA/Security Engineers |
| Sales | 7 | Sales Director, Account Executives, SDRs, CSM |
| Finance | 5 | CFO, Finance Manager, Accountants, Analysts |
| IT | 5 | IT Manager, SysAdmin, Cloud Architect, Security Analyst |
| Marketing | 5 | CMO, Content Strategist, SEO, Social Media, Design |
| Human Resources | 4 | HR Director, HRBP, Recruiter, Coordinator |
| Operations | 4 | COO, Project Manager, Business Analyst, Office Manager |
| Legal | 3 | General Counsel, Compliance Officer, Legal Assistant |
| Support | 2 | Support Team Lead, Support Specialist |
| **Total** | **50** | |

### Method: Bulk CSV Import

Users were created using Entra's native **Bulk create** feature rather than manually, to simulate a realistic onboarding process.

**Path:** `Entra admin center → Users → Bulk operations → Bulk create`

Microsoft's bulk import requires an exact column template (downloaded from the portal itself):

```
Username, First name, Last name, Display name, Job title, Department,
Office number, Office phone, Mobile phone, Fax, Alternate email address,
Address, City, State or province, ZIP or postal code, Country or region
```

> 💡 **Lesson learned:** Entra's bulk import only accepts `.csv`, not `.xlsx`. Always download the current template directly from the portal first and match column headers exactly — Microsoft updates this template periodically, and mismatched headers cause upload failures.

Each user was generated with realistic Canadian profile data:
- Canadian usage location (`CA`)
- City/province/postal code drawn from real Canadian cities (Toronto, Vancouver, Montreal, Edmonton, Winnipeg, Halifax)
- Canadian-format phone numbers with real area codes (416, 604, 514, 403, etc.)
- Auto-generated temporary password, forced reset on first login

📄 *Sample data file: [`/sample-data/entra_bulk_users_template.csv`](./sample-data/entra_bulk_users_template.csv)*

---

## 3. Groups

**6 Security groups** were created to manage access at scale instead of per-user.

| Group Name | Purpose | Approx. Members |
|---|---|---|
| `GRP-IT-Admins` | IT staff — used for Conditional Access & PIM testing | 5 |
| `GRP-Engineering` | Engineering department | 10 |
| `GRP-Finance` | Finance department | 5 |
| `GRP-Sales` | Sales department | 7 |
| `GRP-Managers` | All directors/managers (cross-department) | ~12 |
| `GRP-AllEmployees` | Entire organization | 50 |

### Configuration details
- **Group type:** Security (not Microsoft 365 — no email/collaboration features needed for access-control purposes)
- **Membership type:** Assigned (manual) — Dynamic membership rules require P1/P2 *licenses assigned to users*, not just the tenant, so manual assignment was used
- **Owner:** Tenant admin account, for delegated management practice

> 💡 **Lesson learned:** Choosing "Microsoft 365" as the group type forces a mandatory group email address field. If you don't need Outlook/Teams/SharePoint collaboration features, always choose **Security** group type instead.

---

## 4. Role-Based Access Control (RBAC)

Built-in Entra directory roles were assigned to specific users to mirror real responsibilities — demonstrating **least-privilege** access design.

| User | Job Title | Assigned Role | Why |
|---|---|---|---|
| Ahmed Hassan | IT Manager | User Administrator | Manages user/group lifecycle |
| Priya Sharma | CTO | Global Reader | Org-wide read-only visibility |
| Lily Jackson | Cloud Architect | Cloud App Security Administrator | Manages cloud app security posture |
| Sebastian White | IT Security Analyst | Security Reader | Read-only access to security data |
| Penelope King | General Counsel | Compliance Administrator | Manages compliance policies |
| Elijah Martin | Security Engineer | Security Administrator | Manages security configuration |

**Path:** `Entra admin center → Roles & admins → Roles & admins → [Role] → Add assignments`

### Key concept: Built-in roles vs. custom roles
This lab used Microsoft's **built-in roles** exclusively (sufficient for SC-300 fundamentals). Custom roles — built from granular permissions — are a Premium feature and a natural "next step" extension of this project.

---

## 5. App Registration

A fictional internal application, **"NorthBridge Employee Portal,"** was registered to demonstrate how organizations connect their own apps to Entra ID for authentication (SSO).

**Path:** `Entra admin center → Applications → App registrations → New registration`

| Setting | Value |
|---|---|
| Name | NorthBridge Employee Portal |
| Supported account types | Single tenant (this organizational directory only) |
| Redirect URI | `https://localhost` (Web platform) |

### Configuration steps performed

**1. Client Secret**
A client secret was generated (`Certificates & secrets → New client secret`) to act as the app's own credential when authenticating to Microsoft Entra ID — analogous to a password, but for the application itself rather than a user.

**2. API Permissions (Delegated)**
| Permission | Purpose |
|---|---|
| `User.Read` | Read the signed-in user's own profile (granted by default) |
| `User.ReadBasic.All` | Read basic profile info of other org members |
| `profile` | Access basic profile claims (name, photo) |

**Delegated** permissions were used (not Application permissions) because the portal acts *on behalf of* the signed-in employee — it has no need to act autonomously in the background.

**3. Admin Consent**
Consent was granted tenant-wide (`Grant admin consent`) so individual employees aren't shown a permission-approval prompt on every login — a standard enterprise practice for internal, trusted apps.

### Authentication flow this demonstrates

```
Employee → clicks "Sign in" on Employee Portal
        ↓
App redirects to Entra ID for authentication
        ↓
Entra ID validates credentials (+ MFA if required by policy)
        ↓
Entra ID issues a token confirming identity + permitted claims
        ↓
Employee is signed in to the Employee Portal (SSO)
```

---

## 6. Conditional Access

Conditional Access is Entra's **"if this, then that"** policy engine — the modern, recommended replacement for legacy per-user MFA and Security Defaults.

### Policy 1 — `CA001-Require MFA for IT Admins`

| Setting | Value |
|---|---|
| Users (Include) | `GRP-IT-Admins` only |
| Target resources | All cloud apps |
| Grant control | Require multifactor authentication |
| State | **Report-only → tested → switched to On** |

**Logic:** *If a member of GRP-IT-Admins signs in to any cloud app, require MFA.*

### Policy 2 — `CA002-Require MFA for Risky Sign-ins`

| Setting | Value |
|---|---|
| Users (Include) | All users |
| Target resources | All cloud apps |
| Condition | Sign-in risk: Medium and above |
| Grant control | Require multifactor authentication |
| State | Report-only |

**Logic:** *If a sign-in looks risky (e.g. unfamiliar location, leaked credentials, anonymous IP), require MFA.*

> 💡 **Lesson learned:** Microsoft is retiring the standalone Identity Protection "Sign-in risk policy" / "User risk policy" pages (read-only as of 2026, full retirement October 2026). Risk-based policies should now be built **inside Conditional Access** as a policy condition, not the legacy ID Protection page.

### Testing methodology (Report-only → On)

Every policy was tested safely before enforcement, following Microsoft's recommended rollout pattern:

```
1. Build policy  →  set State = Report-only
2. Trigger a real sign-in matching the policy conditions
3. Check Entra ID → Monitoring & health → Sign-in logs
   → open the sign-in event → "Conditional Access" tab
   → confirms: Policy Name | Grant Controls | Result: "Report-only: Success"
4. Once confirmed correct  →  switch State = On
```

This avoided the risk of misconfigured policies locking out real users — a core SC-300 best practice.

### MFA layers — what controls what (a common point of confusion)

| Layer | Scope | Forces MFA registration? | Controlled by |
|---|---|---|---|
| Per-user MFA (legacy) | Individual users | Yes, if enabled | Admin (now considered legacy) |
| Security Defaults | Entire tenant, all-or-nothing | Yes | Admin (toggle on/off) |
| Authentication Methods — Registration Campaign | Tenant-wide nudge | Optional nudge | Admin |
| Conditional Access | Granular, per group/app/condition | Only if policy specifies | Admin (recommended approach) |
| Microsoft Mandatory MFA (2024–2026 rollout) | Tenant-wide, admin portals first | Yes | **Microsoft** — postponable, not avoidable |

These layers are independent and can overlap. The sign-in log's **Conditional Access tab** is the definitive way to determine which layer triggered a given MFA prompt.

---

## 7. Privileged Identity Management (PIM)

PIM converts **standing (permanent) admin access** into **just-in-time (JIT) access** — a core Zero Trust principle and one of the highest-value SC-300 topics.

### Before PIM
Ahmed Hassan had `User Administrator` as a **permanent, Active** role assignment — meaning the privilege existed 24/7, whether in use or not.

### After PIM
The standing assignment was removed and replaced with an **Eligible** assignment:

| Setting | Value |
|---|---|
| Role | User Administrator |
| Member | Ahmed Hassan |
| Assignment type | **Eligible** (not Active) |
| Duration | Permanently eligible |

**Path:** `Privileged Identity Management → Microsoft Entra roles → Roles → User Administrator → Add assignments`

### Self-service activation (tested end-to-end)

1. Signed in as Ahmed → navigated to `Privileged Identity Management → My roles → Microsoft Entra roles`
2. Clicked **Activate** next to User Administrator
3. Provided a justification reason (and MFA, since CA001 was live)
4. Role became **Active** for a fixed window — verified in the admin's **Active assignments** tab:

| Principal | State | Start time | End time |
|---|---|---|---|
| Ahmed Hassan | Activated | 19/06/2026, 7:31 PM | 19/06/2026, **8:31 PM** (auto-expiry) |

After expiry, Ahmed automatically reverts to **Eligible only** — no manual revocation required.

### Break Glass Accounts (security design concept)

A key realization during this lab: **the tenant's own Global Administrator account remains a standing/permanent assignment** — and that's *intentional*, not an oversight.

> Even in fully PIM-adopted organizations, **2+ "break glass" accounts** are kept as permanent Global Administrators, excluded from Conditional Access, with offline-stored credentials — so that if PIM or Conditional Access ever misconfigures and locks everyone out, these accounts can still restore access.

| Practice | Reason |
|---|---|
| 2+ accounts, not 1 | Redundancy if one is compromised/unavailable |
| Permanent (not PIM-eligible) | Must work even if PIM service is down |
| Excluded from Conditional Access | Must work even if CA policy misfires |
| Credentials stored offline | Not phishable |
| Sign-in alerts configured | Should almost never be used — any use is notable |

---

## 8. Identity Protection

Identity Protection uses Microsoft's threat intelligence and behavioral signals to detect **two types of risk**:

| Risk Type | Detects |
|---|---|
| **User risk** | This identity may be compromised (e.g. credentials found in a leak) |
| **Sign-in risk** | This specific sign-in attempt looks suspicious (impossible travel, anonymous IP, unfamiliar device) |

In this lab, sign-in risk detection was implemented via **Conditional Access** (`CA002`, see [Section 6](#6-conditional-access)), reflecting Microsoft's 2026 platform direction of consolidating risk policies into the Conditional Access engine.

### Dashboard observation
With a brand-new tenant and no real-world malicious traffic, the Identity Protection dashboard correctly showed **0 risk detections across all metrics** — expected behavior, not a misconfiguration. Risk detection depends on accumulated behavioral signals and global threat intelligence that a fresh lab tenant hasn't yet generated.

---

## Key Concepts Learned

A condensed list of the conceptual takeaways from this build — useful as exam revision notes.

- **Tenant types:** Workforce (employees/internal) vs. External (B2B/B2C customer-facing) — choose based on who you're managing.
- **Bulk operations:** CSV-only, exact column headers required, template should always be re-downloaded from the portal (it changes).
- **Security vs. Microsoft 365 groups:** Security groups for access control (no email needed); M365 groups for collaboration (email required).
- **Dynamic vs. Assigned group membership:** Dynamic rules require P1/P2 licenses *assigned to users*, not just enabled on the tenant.
- **Delegated vs. Application permissions:** Delegated = acts as the signed-in user; Application = acts as itself, no user context.
- **Admin consent:** Approves API permissions tenant-wide so individual users aren't prompted.
- **Report-only mode:** The safe way to test any Conditional Access policy before enforcing it — logs intended actions without blocking anyone.
- **MFA can come from multiple independent layers** (per-user, Security Defaults, registration campaigns, Conditional Access, and now Microsoft's own mandatory enforcement) — the sign-in log is the source of truth for which layer fired.
- **PIM = Just-In-Time access.** Eligible vs. Active is the core distinction; activation is time-bound and self-service.
- **Break glass accounts** are a deliberate exception to "everything must be PIM/CA-governed" — required for resilience.
- **Identity Protection risk policies are migrating into Conditional Access** as of 2026 — the standalone risk policy pages are being retired.

---

## Issues Encountered & Fixes

A real troubleshooting log from this build — useful for understanding *why* things work, not just *that* they work.

| Issue | Root Cause | Fix |
|---|---|---|
| "Microsoft Entra ID" tenant type greyed out | Required a paid license for "Workforce (legacy)" creation flow | Used the already-existing free "Default Directory" tenant instead of creating a new one |
| Excel upload rejected by Bulk Create | Entra's bulk import only accepts `.csv` | Re-exported data as CSV with `utf-8-sig` encoding |
| Bulk upload errors despite correct-looking CSV | Custom column headers didn't match Microsoft's exact template | Downloaded Microsoft's official template first and matched headers exactly |
| "Group email address" required field blocking group creation | Had selected "Microsoft 365" group type instead of "Security" | Restarted group creation, selected **Security** type (no email needed) |
| P2 trial tried to assign 26 paid licenses automatically | Selecting "Assign licenses" during bulk user creation defaults to licensing everyone | Selected **"Don't assign any licenses"** — only the admin account needs P2 for policy configuration |
| Forgot bulk-generated user passwords | Microsoft's official template doesn't include a password column — passwords are auto-generated and shown only once | Used **admin password reset** to generate a new temporary password for testing |
| MFA setup prompt appeared despite Conditional Access being in Report-only | Multiple independent layers can trigger MFA registration (Security Defaults, Registration Campaign, and Microsoft's tenant-wide mandatory MFA rollout) | Checked sign-in log → Conditional Access tab to confirm the custom policy was *not* the cause; identified Microsoft's mandatory MFA enforcement (2024–2026 rollout) as the actual trigger |
| Sign-in risk policy page had no "Report-only" option | Microsoft has marked the legacy ID Protection risk policy pages **read-only**, retiring October 2026 | Rebuilt the same logic as a Conditional Access policy instead (`CA002`), which retains full Report-only support |
| PIM "Activate" button missing from admin-side role assignment page | That page is the **admin management view** — self-service activation lives on a separate page | Navigated to `Privileged Identity Management → My roles` (not `→ Microsoft Entra roles → All roles`) |

---

## SC-300 Exam Domain Mapping

How this project maps to the official SC-300 exam objective areas:

| Exam Domain | Covered By |
|---|---|
| **Manage Microsoft Entra identities** | Users (bulk creation), Groups, RBAC role assignment |
| **Manage access to applications** | App Registration, API permissions, Admin consent, SSO concepts |
| **Plan and implement identity governance** | PIM (eligible/active roles), Break Glass account design |
| **Implement and manage authentication** | Conditional Access, MFA layering, Authentication Methods |
| **Plan, implement, and administer Conditional Access** | CA001, CA002 — Report-only testing methodology, sign-in log verification |
| **Manage identity protection** | Identity Protection dashboard, risk-based Conditional Access |

---

## Next Steps

Planned extensions to deepen this lab further:

- [ ] Build a **User Risk Policy** (force secure password reset on high-risk users) to complete the risk-policy pair
- [ ] Set up **Access Reviews** for periodic recertification of `GRP-IT-Admins` membership
- [ ] Create a **second break-glass admin account**, exclude it from Conditional Access, and convert the daily-use admin account to PIM-eligible
- [ ] Explore **Custom Roles** (granular permission sets beyond built-in roles)
- [ ] Document **Entra Connect / hybrid identity** concepts (sync from on-prem AD) — conceptual only, no on-prem environment available

---

## Disclaimer

This is a personal study lab using fictional data (NorthBridge Technologies Inc.) for SC-300 exam preparation. No real organization, employee, or customer data is used. The Entra ID P2 trial used for premium features was cancelled before any billing occurred.

---

*Built while studying for Microsoft Certified: Identity and Access Administrator Associate (SC-300).*
