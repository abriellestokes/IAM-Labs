# Lab 1.2 — Create Users in Okta

**Path:** Define Your Users in Okta  
**Platform:** Okta Admin Console  
**Lab Guide Version:** Okta 2025.10  
**Skill level:** Foundational  
**Estimated completion time:** 20–30 minutes  
**Status:** ✅ Completed

---

## Objective

Create and activate a new user account in Okta, populate the user's profile
with organizational attributes, and verify the full lifecycle event trail
in the system log.

---

## Business scenario

A new IT Administrator named Nina Shah requires access to the organization's
Okta environment. As the IAM administrator, I need to create her account,
assign the correct profile attributes, initiate the activation workflow,
and confirm the account reaches an Active status — all while ensuring audit
logs capture the lifecycle events correctly.

This lab simulates one of the most common real-world IAM tasks: new employee
onboarding. Every user who joins an organization begins with this workflow.
Getting it right — and understanding what happens at each step — is core to
the IAM administrator role.

---

## What I configured

### Step 1 — Created the user account

From the Okta Admin Console, I went to **Directory > People** and added a
new user with the following initial values:

| Field | Value |
|-------|-------|
| First name | Nina |
| Last name | Shah |
| Username | nina.shah@oktaice.com |
| Primary email | *(personal or work email for activation)* |

**Key decision — no password set at creation.**

I intentionally did not set a password during account creation. When no
password is set, Okta sends an activation email to the primary email address,
which is the standard and more secure onboarding flow. Setting a password
at creation skips the activation email — a shortcut typically only used for
test accounts. In a production environment, allowing users to set their own
passwords from the start is both a security best practice and a compliance
requirement in many frameworks.

I also verified that the account was set to **Activate now**, meaning the
activation email would be sent immediately upon saving.

### Step 2 — Verified the pending status

After saving the account and refreshing the browser, I confirmed that Nina
Shah's status showed as **Pending user action**. This is the expected state:
the account exists in Okta, but the user has not yet clicked the activation
link in their email to claim the account.

Understanding user account statuses is critical for troubleshooting. If a
user reports they never received their activation email, the admin can
verify the account is still in **Pending user action** status and resend the
activation link — without creating a duplicate account.

### Step 3 — Populated the user profile with organizational attributes

After the initial account creation, I navigated to Nina Shah's profile and
populated her organizational attributes:

| Field | Value |
|-------|-------|
| Title | IT Admin |
| Primary phone | 925-555-1000 |
| User type | Employee |
| Cost center | IT |
| Organization | OktaIce |
| Department | IT |
| Region | AMER |

The `Region` field was available here because of the custom attribute created
in Lab 1.1. This is the first time the upstream configuration decision
directly surfaces in a downstream workflow — building the attribute schema
first enables clean, consistent data entry here.

Setting **User type** to `Employee` is particularly important. In later labs,
group rules use `userType` as a filtering condition to distinguish employees
from contractors. Assigning the correct user type at onboarding ensures the
user is automatically placed in the correct groups when those rules are active.

### Step 4 — Activated the account as Nina Shah

Switching to a separate browser session to simulate the user experience, I:

- Accessed the activation email sent to the primary email address
- Clicked the activation link from the Welcome to Okta email
- Set a new password for the account
- Enrolled in Okta Verify on a mobile device for MFA

After completing these steps, I verified that I was successfully signed into
Okta as Nina Shah, then signed out and closed the browser tab.

### Step 5 — Verified Active status from the admin account

Returning to the admin console, I refreshed the People page and confirmed
that Nina Shah's status had changed from **Pending user action** to
**Active**. This confirms the full activation flow completed successfully.

### Step 6 — Reviewed the system log for lifecycle events

I navigated to **Reports > Reports** and applied the **User lifecycle
activity** system log filter. I verified the log captured both expected
events:

| Event | Target | Result |
|-------|--------|--------|
| Create Okta user | Nina Shah (User) | SUCCESS |
| Activate Okta user | Nina Shah (User) | SUCCESS |

The system log is the authoritative audit trail. In a production environment,
these log entries are what compliance teams, security operations centers, and
auditors reference to verify that onboarding events occurred correctly and
were not tampered with.

---

## Why this matters

**User provisioning is the entry point for every security decision downstream.**
Every access policy, group membership, and application assignment flows from
the moment a user account is created. Errors made at provisioning — wrong
user type, missing department, incorrect region — can silently break
automation and policy enforcement in ways that are hard to diagnose later.

**The activation email flow is a security control, not just UX.** Requiring
users to activate their own accounts via email confirms that the person
claiming the account has access to the registered email address. This is a
basic but meaningful identity verification step. Skipping it by setting a
password at creation removes that verification and creates accounts that
could be activated without the intended user's knowledge.

**Account status awareness prevents provisioning errors.** Understanding the
difference between Pending user action, Active, Password reset, Locked out,
Suspended, and Deactivated is essential for day-to-day IAM operations. Each
status reflects a different point in the identity lifecycle and requires a
different administrative response.

**System log verification closes the loop.** Creating a user is not complete
until the log confirms it happened correctly. In regulated industries —
healthcare, finance, government — the system log is evidence. Administrators
who verify log entries after provisioning operations are practicing audit
readiness as a habit, not a once-a-year exercise.

**Profile completeness drives downstream automation.** A user account with
missing attributes is not just incomplete — it may be excluded from group
rules, receive incorrect application assignments, or fail policy evaluations.
Complete, accurate profiles at creation reduce remediation work later.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| User provisioning | Created a new Okta user account using the Admin Console |
| Activation flow | Initiated email-based activation rather than admin-set password |
| Account status lifecycle | Observed transition from Pending user action → Active |
| Profile attribute population | Applied organizational attributes including custom `region` field |
| User type assignment | Set `Employee` to enable correct downstream group rule behavior |
| System log auditing | Verified Create and Activate lifecycle events in Reports |
| Least privilege (provisioning) | Did not set password at creation — preserved user-initiated activation |

---

## Lessons learned

This lab made clear that user provisioning is a multi-step workflow, not a
single action. There is a meaningful difference between creating an account
and activating an account — and the system correctly enforces that boundary
through the Pending user action status.

The most instructive moment was connecting Lab 1.1 to Lab 1.2. The `region`
attribute I added to the profile schema in the previous lab appeared here as
an available field during profile population. This reinforced how IAM
configuration is cumulative — earlier decisions shape what is possible in
later workflows. It also reinforced why getting the schema right before
onboarding users matters.

The system log review at the end shifted how I think about administrative
tasks. Completing an action in the UI is only part of the job. Confirming
that the action is captured in the audit log is the professional standard —
especially in environments where access to systems is subject to compliance
review or security monitoring.

In a real organization, I would also note that the primary email field used
for activation is separate from the username. This is deliberate — it allows
the activation link to go to a personal or temporary email address while the
Okta username follows the organization's naming convention. That separation
matters in onboarding flows where the user may not yet have a corporate email
account at the moment of provisioning.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: New user creation form, Pending user action status, profile
> attribute population, Active status confirmation, and system log showing
> Create and Activate lifecycle events.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Related labs

- **Lab 1.1** — Add a custom attribute *(established the `region` attribute
  used in Step 3 of this lab)*
- **Lab 1.3** — Manage user account statuses *(continues Nina Shah's
  lifecycle with password resets, lockouts, suspension, and deactivation)*
- **Lab 2.1** — Manually assign users to a group *(uses users provisioned
  in this lab as group members)*
- **Lab 2.3** — Add a group rule based on a user attribute *(the `Department`
  and `userType` values set in this lab drive automated group assignment)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
