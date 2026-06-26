# Lab 1.3 — Manage User Account Statuses

**Path:** Define Your Users in Okta  
**Platform:** Okta Admin Console  
**Lab Guide Version:** Okta 2025.10  
**Skill level:** Foundational  
**Estimated completion time:** 30–45 minutes  
**Status:** ✅ Completed

---

## Objective

View and manage user account statuses across four real-world scenarios:
password reset, password expiration, account lockout, and user
suspension and deactivation.

---

## Business scenario

This lab follows the continued lifecycle of Nina Shah, the IT Administrator
provisioned in Lab 1.2. Four distinct operational scenarios are simulated,
each representing a common helpdesk and IAM administration situation:

- **Scenario A:** Nina forgot her password and needs it reset
- **Scenario B:** Nina's password was suspected to be compromised — it
  needs to be expired and a temporary password issued
- **Scenario C:** Nina entered the wrong password too many times and her
  account is locked out
- **Scenario D:** Nina left the company — her account must be suspended
  immediately, then deactivated after 30 days

Together, these four scenarios cover the majority of account status
management tasks an IAM administrator handles in day-to-day operations.

---

## What I configured

### Scenario A — Reset the password for a user

**Situation:** Nina Shah forgot her password.

From the Okta Admin Console, I navigated to Nina Shah's user account and
selected **Reset or Remove password**. I chose the option to **send a reset
password email**, which:

- Sends a password reset link directly to Nina's primary email address
- Signs Nina out of all active devices and browsers immediately
- Transitions her account status to **Password reset**

I verified the status change, then simulated the user experience by accessing
the reset email, clicking the reset link, and setting a new password. After
completing the reset, I confirmed from the admin console that Nina's status
returned to **Active**.

**Why this method over setting a temporary password:**  
Sending a reset email keeps the password known only to the user. The
administrator never sees or handles the new password, which is both a
security best practice and often a compliance requirement. The admin's role
is to initiate the reset — not to know the credential.

---

### Scenario B — Expire the password for a user

**Situation:** Nina's password was suspected to be exposed.

This scenario required a different approach from Scenario A. Rather than
sending a reset email, I selected the option to **create a temporary
password**. This:

- Immediately expires Nina's current password
- Generates a temporary credential that I, as the admin, provide to the user
  directly (via a secure channel)
- Forces the user to set a new permanent password on their next login
- Transitions the account status to **Password expired**

I copied the temporary password, closed the dialog, and verified the
**Password expired** status. I then simulated Nina's login using the
temporary password, which required her to immediately set a new password
before accessing Okta. After the new password was set, her status returned
to **Active**.

**Key distinction from Scenario A:**  
Password expiration with a temporary credential is used when the admin
needs immediate, controlled revocation — particularly when a credential is
suspected to be compromised. The email-based reset in Scenario A is
self-service; the temporary password in Scenario B is admin-controlled and
appropriate for security incidents.

---

### Scenario C — Handle an account lockout

**Situation:** Nina entered the wrong password too many times.

To simulate this scenario, I signed in as Nina Shah and intentionally entered
an incorrect password 11 times. The default Okta password policy locks an
account after **10 unsuccessful attempts**. On the 11th attempt, the account
transitioned to **User is locked out** status.

From the admin console, I confirmed the locked status, then unlocked the
account using the unlock action on Nina's profile. After unlocking, her
status returned to **Active** — no password change was required.

**Operational note:**  
Account lockouts are one of the highest-volume helpdesk requests in any
organization. Understanding the difference between a lockout (temporary,
reversible by admin unlock) and a suspension (deliberate, requires an admin
action to reverse) is essential. Many users and even junior IT staff confuse
these two states, which can lead to incorrect remediation steps.

The lockout threshold — 10 attempts in this environment — is a configurable
policy setting. In the security policies labs, password policy configuration
is explored in depth, including setting custom lockout thresholds.

---

### Scenario D — Suspend and deactivate a user

**Situation:** Nina Shah left the company.

This scenario follows a two-phase offboarding workflow that reflects real
enterprise policy:

**Phase 1 — Immediate suspension (day of departure):**

From Nina's user account, I selected **More Actions > Suspend**. This
immediately prevented Nina from signing into Okta or any of her assigned
applications. Her status transitioned to **Suspended**.

I verified the suspension was effective by attempting to sign in as Nina —
the login was rejected. The account still exists in Okta and can be
reactivated if needed, but access is completely blocked.

**Phase 2 — Deactivation (30 days later):**

After the simulated 30-day retention period, I selected **More Actions >
Deactivate** from Nina's account. Her status transitioned to
**Deactivated**.

A deactivated account is no longer accessible but remains in Okta's
directory. It can be permanently deleted or reactivated if circumstances
change (for example, if the employee is rehired). Until permanently deleted,
the account and its history remain available for audit purposes.

---

## The Okta user account status lifecycle

This lab introduced six distinct account statuses. Understanding the full
lifecycle — and the transitions between statuses — is foundational to IAM
administration:

```
[Provisioned] ──► [Pending user action] ──► [Active]
                                                │
                          ┌─────────────────────┼────────────────────┐
                          ▼                     ▼                    ▼
                   [Password reset]     [Password expired]   [User is locked out]
                          │                     │                    │
                          └─────────────────────┴────────────────────┘
                                                │
                                                ▼
                                          [Active] ──► [Suspended] ──► [Deactivated]
                                                                              │
                                                                              ▼
                                                                    [Permanently deleted]
```

| Status | Meaning | Admin action to resolve |
|--------|---------|------------------------|
| Pending user action | Account created, activation email sent, user has not activated | Resend activation email |
| Active | Account fully operational | N/A |
| Password reset | Admin initiated email-based password reset | User completes reset via email |
| Password expired | Admin issued temporary password | User logs in with temp password and sets new one |
| User is locked out | Too many failed login attempts | Admin unlocks the account |
| Suspended | Deliberate access revocation, account preserved | Admin unsuspends |
| Deactivated | Account disabled, retained for audit | Admin reactivates or permanently deletes |

---

## Why this matters

**Account status management is the most frequent IAM task in production.**
Provisioning a user happens once. Managing their account status — resets,
lockouts, offboarding — happens continuously throughout their tenure. An IAM
administrator who cannot quickly and correctly navigate these status
transitions will create friction for users and risk for the organization.

**The suspension-before-deactivation pattern is a compliance and security
standard.** Immediately deactivating a departing employee's account without
a suspension phase can cause problems: active sessions may not terminate
immediately, and some integrations do not process deactivation events as
quickly as suspension. The two-phase approach — suspend first, deactivate
after a retention window — is the established best practice in enterprise
IAM and is required by many regulatory frameworks.

**Offboarding is a security-critical event.** According to industry research,
a significant portion of data breaches involve former employees with
lingering access. The speed and completeness of the offboarding workflow
directly affects an organization's security posture. Suspension must happen
on the day of departure, not days later.

**Temporary passwords are a security incident response tool.** When a
credential is suspected to be compromised, the admin needs to revoke access
immediately and issue a controlled temporary credential. The email-based
reset is not appropriate in this scenario because the attacker may have
access to the user's email. The temporary password flow bypasses email and
puts control directly in the administrator's hands.

**System log verification is not optional.** Every status change in this lab
was verifiable in the system log. In a regulated environment, the audit trail
of who performed which action on which account and when is as important as
the action itself. Administrators should develop the habit of verifying log
entries after status changes, especially for offboarding events.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| Password reset (email-based) | Initiated self-service reset workflow via activation email |
| Password expiration (admin-controlled) | Issued temporary credential for suspected compromise scenario |
| Account lockout | Observed automatic lockout after 10 failed attempts; performed admin unlock |
| User suspension | Immediately revoked access on simulated departure day |
| User deactivation | Completed two-phase offboarding after simulated 30-day retention |
| Status lifecycle fluency | Navigated and verified six distinct account statuses |
| Audit log verification | Confirmed status transitions in Reports > System Log |
| Least privilege (offboarding) | Applied suspension before deactivation to preserve auditability |

---

## Lessons learned

This was the most operationally dense lab in Path 1. Four distinct scenarios
in a single lab meant moving quickly between admin and user contexts — which
is exactly what production IAM work feels like. Juggling the admin console
and simulated user sessions side by side gave me a clearer picture of how
these workflows look from both sides of the identity relationship.

The scenario that generated the most insight was Scenario B — password
expiration. I had previously understood "reset" and "expire" as roughly
equivalent. This lab made the distinction concrete: a reset returns control
to the user via email; an expiration returns control to the admin via a
temporary credential. Choosing the wrong method in a real incident could
leave a compromised account accessible if an attacker also controls the
user's email.

The two-phase offboarding in Scenario D also reframed how I think about
deactivation. I had assumed deactivation was the first and only step when
someone leaves. Understanding that suspension is the immediate control —
because it blocks access faster and more reliably than deactivation alone
in some integration scenarios — changes how I would approach an offboarding
request in production.

Finally, watching the status lifecycle play out across all four scenarios in
sequence reinforced that account status is not just an administrative label.
Each status is a security posture. Active means the identity is trusted.
Suspended means trust is revoked but the record is preserved. Deactivated
means the lifecycle has ended. Treating these as meaningful security states,
not just UI indicators, is the right mental model for IAM administration.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: Password reset status confirmation, password expired status,
> locked out status, admin unlock action, suspended status with failed
> login verification, deactivated status, and system log entries for each
> scenario.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Related labs

- **Lab 1.2** — Create users in Okta *(provisioned the Nina Shah account
  used across all four scenarios in this lab)*
- **Lab 1.4** — Test attribute mappings *(continues working with the same
  user profile in the Okta Workflows context)*
- **Lab 3.7** — Add a rule to the default global session policy *(session
  lifetime and idle timeout settings directly affect how quickly lockouts
  and suspensions terminate active sessions)*
- **Lab 3.9** — Set up a password policy *(the 10-attempt lockout threshold
  encountered in Scenario C is a configurable password policy setting
  explored in depth in that lab)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
