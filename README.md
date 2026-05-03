# Microsoft Entra ID — Identity & Access Management Lab

**Platform:** Microsoft Entra ID (Azure AD) | **Tenant:** gauravkoshtiyahoo.onmicrosoft.com | **Date:** January 2026

---

## Overview

This project documents a hands-on Identity and Access Management (IAM) lab built entirely inside a free Microsoft Entra ID tenant. The goal was to simulate how identity governance is implemented in a real enterprise environment — from provisioning structured user accounts and enforcing Role-Based Access Control, to enabling Security Defaults for MFA and validating the entire setup through sign-in log auditing.

Four video walkthroughs document each phase of the lab: an intro to the tenant and environment, RBAC implementation with user provisioning and security group assignment, enabling MFA via Security Defaults and validating enforcement on a test user login, and finally reviewing sign-in logs and authentication details to confirm policy compliance.

---

## Tenant Environment

| Field | Value |
|---|---|
| Tenant Name | Default Directory |
| Primary Domain | gauravkoshtiyahoo.onmicrosoft.com |
| Admin Account | Gaurav Koshti (Global Administrator) |
| Total Users | 21 |
| Security Groups | 4 |
| Registered Apps | 1 (RBAC-Test-IT) |
| Identity Secure Score | 68.42% |

---

## Video 1 — Lab Introduction & Tenant Overview

**Duration:** ~1 min 17 sec

This video introduces the Microsoft Entra admin center and provides an overview of the lab environment. The recording opens on the Entra ID home dashboard showing the Default Directory with 21 provisioned users, 4 security groups, 0 registered devices, and 1 enterprise application. The admin account (Gaurav Koshti — Global Administrator) is visible in the top-right, confirming Global Admin privileges for the session.

Key elements shown:
- Tenant ID and Primary Domain displayed on the Entra home page
- Role assignment panel showing 1 high-privileged role assignment (Global Administrator)
- Identity Secure Score at 68.42% visible in the Tenant Status section
- Left-side navigation confirming access to Users, Groups, Devices, Enterprise Apps, App Registrations, Roles & Admins, and Domain Services
- The "View Users" and "View Groups" quick links previewing the full 21-user, 4-group structure built for the lab

This video sets the context for all subsequent demos — the tenant is pre-configured with structured department-based users and groups before any RBAC or MFA policies are applied.

---

## Video 2 — User Provisioning, RBAC & Security Groups

**Duration:** ~2 min 58 sec

This video documents the creation of 20 structured Entra ID users across four departments and the assignment of those users to four RBAC-aligned security groups. It also demonstrates the creation of the `RBAC-Test-IT` enterprise application and the process of assigning users to it.

### User Provisioning

The Users blade is shown with all 21 accounts listed (20 department users + 1 Global Admin). Users were created with a consistent naming convention across four departments:

| Department | Users | UPN Format |
|---|---|---|
| HR | hruser01 – hruser05 | hruser0X@gauravkoshtiyahoo.onmicrosoft.com |
| IT | ituser01 – ituser05 | ituser0X@gauravkoshtiyahoo.onmicrosoft.com |
| Dev | devuser01 – devuser05 | devuser0X@gauravkoshtiyahoo.onmicrosoft.com |
| Finance | financeuser01 – financeuser05 | financeuser0X@gauravkoshtiyahoo.onmicrosoft.com |

All accounts are Member-type users with no Agent privileges.

### Security Groups

The Groups blade confirms 4 security groups were created — all Cloud groups with no dynamic membership (all assigned manually):

| Group | Members | Purpose |
|---|---|---|
| hr-sg | hruser01 – hruser05 | HR department access group |
| it-sg | ituser01 – ituser05 | IT department access group |
| de-sg | devuser01 – devuser05 | Development team access group |
| finance-sg | financeuser01 – financeuser05 | Finance department access group |

The Groups Overview panel confirms: Total groups = 4, Security groups = 4, Cloud groups = 4, Dynamic groups = 0, M365 groups = 0.

### RBAC Enterprise Application

The `RBAC-Test-IT` enterprise application was created to simulate role-based access control. The video shows the **Users and groups** blade inside `RBAC-Test-IT` with all five IT users (IT User 01 through IT User 05) assigned to the application with Default Access role.

A notable limitation observed during the lab: when attempting to assign groups directly to the app via **Add Assignment**, Entra displayed a warning that group-based assignment requires a higher Azure AD plan level. Individual users were assigned instead. This is a real-world constraint of the free/trial tier and was documented accordingly.

The **Owners** blade for the application shows no application owners assigned, which is expected for a test application managed directly by the Global Admin.

---

## Video 3 — MFA Enforcement via Security Defaults

**Duration:** ~2 min 58 sec

This video demonstrates enabling Security Defaults in the tenant to enforce MFA across all users, and validates that the policy is working by signing in as a standard user (ituser01) and observing the MFA challenge prompt.

### Security Defaults Enabled

Security Defaults were enabled from the Entra ID tenant Properties panel. Once enabled, all users in the tenant are required to register and use MFA — enforced via Microsoft Authenticator by default. The policy cannot be granularly scoped under the free tier (Conditional Access policies require Entra ID P1 or P2).

The **Authentication Methods | Policies** blade is shown briefly. When accessed as `ituser01` (a non-admin user), the blade returns a 401 Access Denied error — this is expected behavior, as authentication method policy management is restricted to privileged administrators. This confirms RBAC access boundaries are working correctly.

### MFA Login Validation

To validate enforcement, the session switches to `ituser01@gauravkoshtiyahoo.onmicrosoft.com` via the Switch directory / Sign in with a different account flow. The login sequence shows:

1. User signs in with username and password at `login.microsoftonline.com`
2. After successful password entry, the Microsoft authentication screen prompts **"Enter code"** — requiring an authenticator app code from the user's mobile device
3. The MFA challenge confirms Security Defaults are actively intercepting logins for non-admin users

This validates that the policy is enforced at sign-in, not just configured in the portal.

---

## Video 4 — Sign-In Log Audit & Authentication Verification

**Duration:** ~2 min 49 sec

This video walks through the Entra ID sign-in logs for `IT User 01` to verify the MFA event captured in Video 3 and review the full authentication activity detail, including location data, authentication methods applied, and Conditional Access policy results.

### Sign-In Log Overview

Navigating to **Users → IT User 01 → Sign-in logs**, the log shows one interactive sign-in event in the last 24 hours sourced from Montreal, Quebec, CA. The Authentication result column confirms **Multifactor authentication** was applied.

### Activity Details — Location Tab

The sign-in detail panel is opened and the **Location** tab shows:

| Field | Value |
|---|---|
| Location | Montreal, Quebec, CA |
| IP Address | 24.202.127.207 |
| Through Global Secure Access | No |
| Autonomous System Number | 5769 |

No named network locations are configured in the tenant, so the location shows as a standard residential ISP.

### Activity Details — Authentication Details Tab

The **Authentication Details** tab confirms two authentication steps were satisfied under the Security Defaults policy:

| Time | Authentication Method | Result | Detail |
|---|---|---|---|
| 03:26 PM | Previously satisfied | True | First factor requirement satisfied |
| 03:26 PM | Previously satisfied | True | MFA requirement satisfied by claim |

The "Previously satisfied" status indicates the MFA claim was carried from a prior session token — confirming the MFA registration was completed successfully in Video 3.

### Activity Details — Conditional Access Tab

The **Conditional Access** tab shows one policy applied to this sign-in:

| Policy Name | Grant Controls | Result |
|---|---|---|
| Security defaults | Require multifactor authentication | Success |

This confirms end-to-end enforcement: Security Defaults policy was evaluated at sign-in, MFA was required, MFA was satisfied, and access was granted. The audit trail is complete and inspectable.

---

## IAM Configuration Summary

| Component | Details |
|---|---|
| Users Created | 20 department users across HR, IT, Dev, Finance |
| Security Groups | 4 groups (hr-sg, it-sg, de-sg, finance-sg) — manually assigned, 5 users each |
| Enterprise App | RBAC-Test-IT — IT users (01-05) assigned with Default Access role |
| MFA Policy | Security Defaults enabled — enforces MFA for all users tenant-wide |
| MFA Validation | ituser01 prompted for authenticator code on login |
| Sign-In Audit | Authentication Details and Conditional Access tabs confirm policy enforcement |

---

## Real-World Observations

**Group assignment limitation on free tier.** The `RBAC-Test-IT` app showed a warning that group-based assignment is not available at the current Azure AD plan level. In an enterprise environment with Entra ID P1 or P2 licensing, security groups can be assigned directly to application roles — a critical feature for scalable RBAC at hundreds or thousands of users. The lab used individual user assignment as the equivalent workaround.

**Security Defaults vs. Conditional Access.** Security Defaults provide a simple, free MFA enforcement mechanism but offer no granularity — they apply to all users with no exceptions for locations, device compliance, or risk levels. Enterprises requiring scoped MFA (e.g., require MFA only from outside the corporate network) would need Conditional Access policies, which require Entra ID P1 licensing.

**Authentication Policies blade access control.** A non-admin user (ituser01) attempting to view the Authentication Methods | Policies blade received a 401 error. This confirmed that Entra ID correctly restricts sensitive identity policy pages to privileged roles — even when the user is logged into the same admin center URL.

---

## Repository Structure

```
Entra_Iam_Lab/
├── README.md
├── 1. Intro-IAM-Lab.mp4                     # Tenant overview and environment intro
├── 2. RBAC, User Provision, Sec groups.mp4  # User creation, groups, and app RBAC
├── 3. MFA, policy enabled.mp4               # Security Defaults + MFA login demo
└── 4. Logs check.mp4                        # Sign-in log audit and auth validation
```

---

## Tools Used

- Microsoft Entra ID (Azure Active Directory) — Free Tier
- Microsoft 365 Admin Center
- Microsoft Authenticator (MFA)
- Microsoft Edge (lab browser)

---

*Gaurav Koshti | Montreal, Canada | [LinkedIn](https://linkedin.com/in/gaurav-koshti)*
