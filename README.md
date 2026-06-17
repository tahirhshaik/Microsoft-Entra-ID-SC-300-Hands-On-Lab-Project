# EntraID-IAM-Enterprise-Project-
An enterprise cloud identity infrastructure project deploying a 50-user directory with departmental RBAC, SAML application integration, and Zero-Trust security hardening using a free Microsoft Entra ID tenant.


# Enterprise IAM Deployment: 50-User Directory & RBAC Lifecycle

## Project Overview
This project demonstrates the end-to-end design, provisioning, and security configuration of a cloud identity infrastructure for a mid-sized enterprise using **Microsoft Entra ID**. Facing free-tier licensing constraints regarding group-based application assignments, I engineered a secure, scalable Least-Privilege access model utilizing strategic individual mappings, robust identity attribute tagging, and mandatory multi-factor authentication (MFA) enrollment.

---

## Architecture & Implementation Phases

### Phase 1: Automated Bulk Provisioning
To simulate a real-world enterprise onboarding scenario (such as an HR system data migration), I bypassed manual user creation by executing a directory-wide bulk import.
* Constructed a comprehensive identity schema mapping attributes including User Principal Name (UPN), Job Title, Department, and geographic parameters (Ontario, Canada).
* Ingested a 50-user corporate workforce roster using a standardized `.csv` data payload via the Entra Bulk Operations engine.

### Phase 2: Role-Based Access Control (RBAC)
The workforce was segmented into 5 operational security units to establish clear administrative boundaries: `IAM`, `Cybersecurity`, `Risk & Compliance`, `Infrastructure`, and `IT Operations`. I provisioned standalone Security Groups matching these departments to prepare the tenant for structured resource management.

### Phase 3: Application Integration & Licensing Workarounds
* Integrated a target line-of-business enterprise application (**HubSpot** / **CyberArk**) acting as a secure federated service provider.
* **Licensing Boundary Resolution:** Because the Entra ID Free tier gates *Group-Based Application Assignment* behind a Premium P1/P2 license, I adapted the deployment by safely assigning target departmental users to the application individually, proving an understanding of enterprise budget and platform constraints.

### Phase 4: Tenant Security Hardening (Zero Trust Alignment)
To eliminate credential stuffing and unauthorized lateral movement, company-wide baseline protection was established by enabling **Microsoft Security Defaults**. This policy enforces immediate MFA registration via the Microsoft Authenticator app for all 50 identities—including a dedicated Global Administrator account—while completely blocking vulnerable legacy authentication protocols.
