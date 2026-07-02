# Lab 3.3 — Add Google Authenticator

**Path:** Implement Security Policies  
**Platform:** Okta Admin   
**Status:** ✅ Completed

---

## Objective

Add Google Authenticator as an optional authenticator in the Okta org,
verify its enrollment policy behavior, and observe how authentication
policy requirements interact with authenticator availability — specifically
how satisfying one authenticator requirement can change the enrollment
status of another from required to optional.

---

## Business scenario

The organization currently uses Okta Verify as its primary MFA
authenticator. To give users more flexibility — particularly for users
whose devices may not support Okta Verify — the organization wants to
offer Google Authenticator as an additional option.

As the IAM administrator, I added Google Authenticator to the org's
authenticator list, confirmed its enrollment policy settings, and
tested how the enrollment experience presents to a user when the
authentication policy requires two factor types but the user has already
satisfied one requirement with Okta Verify.

This lab introduces a critical concept: the difference between an
authenticator being *available* (added to the org) and being *required*
(mandated by an authentication policy). Understanding that distinction
is foundational to designing MFA strategies that are both secure and
usable.

---

## What I configured

### Step 1 — Added Google Authenticator as an authenticator

From the Okta Admin Console, I navigated to **Security > Authenticators**
and added Google Authenticator to the org's authenticator list.

After adding it, I selected the **Enrollment** tab and reviewed the
Default Policy to confirm that both Okta Verify and Google Authenticator
were set as **optional** authenticators for everyone in the org.

**What "optional" means in this context:**  
An optional authenticator is available for users to enroll in at any
time — but the authentication policy, not the enrollment policy, determines
whether enrollment becomes effectively required. If an authentication
policy requires a factor type that only Google Authenticator can satisfy,
a user who has not enrolled in Google Authenticator will be prompted to
do so before gaining access. Optional in the enrollment policy does not
mean the authenticator will never be required — it means it is not
unconditionally required for all users at all times.

### Step 2 — Tested the default enrollment policy as a standard user

I opened an incognito browser window and signed in as Hugo Santos
(hugo.santos) to simulate the end-user experience.

Upon sign-in, I observed that both Google Authenticator and Okta Verify
were presented as **Required now**. This initially appears to contradict
the enrollment policy setting — both authenticators are marked optional
in the enrollment policy, yet the user is being prompted to enroll in
both immediately.

**Why this happens — authentication policy interaction:**  
The Okta Dashboard authentication policy requires users to authenticate
with two factor types. To satisfy that requirement, users must have at
least one qualifying authenticator enrolled. Because Hugo Santos had
neither Okta Verify nor Google Authenticator enrolled, the system
presented both as required in order to meet the authentication policy's
two-factor requirement.

The enrollment policy sets availability. The authentication policy
drives urgency. When an authentication policy cannot be satisfied
without enrolling in a specific authenticator, that authenticator
becomes effectively required at the moment of sign-in — regardless of
its enrollment policy setting.

### Step 3 — Enrolled in Okta Verify and observed the status change

I set up Okta Verify on a mobile device for the Hugo Santos account.
After completing the Okta Verify enrollment, I observed that Google
Authenticator's status changed from **Required now** to **Optional**.

This status change is the most instructive moment in the lab. Once Okta
Verify was enrolled, the two-factor authentication policy requirement
was satisfied. Google Authenticator was no longer needed to meet the
requirement — so it reverted to its true enrollment policy status:
optional.

I selected **Continue** to skip Google Authenticator enrollment,
verified that Hugo Santos was successfully signed into Okta, then
signed out.

---

## The authenticator enrollment and authentication policy interaction

This lab surfaces a nuanced relationship between two distinct policy
layers in Okta that is critical to understand for MFA design:

```
Enrollment Policy                    Authentication Policy
─────────────────                    ─────────────────────
Sets which authenticators            Sets what factor types are
are available and whether            required to access a specific
users can enroll in them             application or resource
        │                                       │
        │                                       │
        ▼                                       ▼
"Google Authenticator                "Users must authenticate
is Optional for all users"           with 2 factor types"
        │                                       │
        └───────────────────┬───────────────────┘
                            │
                            ▼
              Runtime enrollment experience
              ─────────────────────────────
              If user has no authenticators enrolled
              and policy requires 2 factors:
              → Both authenticators appear as Required now

              If user enrolls Okta Verify (satisfies policy):
              → Google Authenticator reverts to Optional
```

**The key principle:** Enrollment policy controls availability.
Authentication policy controls necessity. When necessity exceeds
availability, the system bridges the gap by prompting enrollment
at sign-in time.

---

## Authenticator types and their security properties

Google Authenticator is a TOTP (Time-based One-Time Password)
authenticator. Understanding the security profile of each authenticator
type is important for MFA strategy design:

| Authenticator | Type | Factor category | Phishing resistant | Hardware protected |
|---------------|------|-----------------|-------------------|-------------------|
| Password | Knowledge | Something you know | No | No |
| Google Authenticator | TOTP | Something you have | No | No |
| Okta Verify (TOTP) | TOTP | Something you have | No | No |
| Okta Verify (Push) | Push notification | Something you have | No | No |
| Okta Verify (FastPass) | Device-bound | Something you have | Yes | Yes |
| Hardware security key (FIDO2) | WebAuthn | Something you have | Yes | Yes |

**Why TOTP authenticators are not phishing-resistant:**  
TOTP codes (the 6-digit rotating codes generated by Google Authenticator
and Okta Verify TOTP) can be intercepted by a real-time phishing attack.
An attacker who creates a convincing fake login page can capture both
the user's password and their TOTP code and replay them immediately
to the real site before the code expires. This is a known attack vector
called a real-time phishing proxy attack.

Phishing-resistant authenticators — FastPass, FIDO2 hardware keys —
use cryptographic binding to the legitimate site's domain, making
them immune to this attack. This is why Lab 3.8's authentication
policy distinguishes between possession factors that are
hardware-protected and those that are not.

Understanding the security properties of each authenticator is what
allows an IAM administrator to design MFA policies that match
protection level to risk — not just check a box for "MFA enabled."

---

## Why this matters

**Offering multiple authenticators improves security adoption.**
Users who cannot or will not enroll in a specific authenticator are
a gap in the MFA coverage. Offering Google Authenticator alongside
Okta Verify gives users a fallback option — particularly useful for
users on devices where Okta Verify is not supported, or users who
prefer a standalone TOTP app they already use for other services.
Higher enrollment rates mean more comprehensive MFA coverage across
the org.

**The enrollment-authentication policy interaction is a common source
of user confusion.** End users who see "Required now" for an
authenticator marked as optional in the enrollment policy will be
confused — and may raise helpdesk tickets. IAM administrators who
understand why this happens can explain it clearly, update user
communications, and design onboarding flows that set expectations
correctly. This lab provides the mental model for that explanation.

**Authenticator choice is a security architecture decision.**
Adding Google Authenticator is not just a UX improvement — it is a
decision about which factor types the organization supports and at
what security level. TOTP authenticators like Google Authenticator
improve over password-only authentication but do not provide
phishing resistance. For high-risk access scenarios, organizations
should layer phishing-resistant authenticators on top of TOTP options
rather than treating all MFA methods as equivalent.

**The "optional but effectively required" pattern is by design.**
Okta's separation of enrollment policy and authentication policy is
intentional. It allows organizations to make authenticators broadly
available without mandating enrollment for users who do not need
them — while still being able to require enrollment at the moment
a user attempts to access a resource that demands a specific factor
type. This flexibility is a feature of mature MFA architecture, not
a loophole.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| Authenticator addition | Added Google Authenticator to the org's available authenticator list |
| Enrollment policy configuration | Confirmed optional enrollment status for both Okta Verify and Google Authenticator |
| Authentication policy interaction | Observed how authentication policy requirements drive enrollment urgency at sign-in |
| Authenticator status dynamics | Witnessed Google Authenticator transition from Required now to Optional after Okta Verify enrollment |
| TOTP authenticator type | Understood Google Authenticator's factor category and security properties |
| Phishing resistance awareness | Distinguished between TOTP authenticators and phishing-resistant alternatives |

---

## Lessons learned

The status change from Required now to Optional after Okta Verify
enrollment was the clearest demonstration in this lab of how the two
policy layers interact. Seeing it happen in real time — Google
Authenticator going from urgent to optional the moment Okta Verify
was enrolled — made the conceptual relationship between enrollment
policy and authentication policy concrete in a way that reading about
it would not have.

The phishing resistance distinction also reframed how I think about
MFA. Before this lab, I understood MFA as a binary: either a user
has it or they do not. This lab introduced a spectrum: TOTP is better
than no MFA, but push notifications are better than TOTP, and
device-bound phishing-resistant methods are better still. Designing
an MFA strategy means choosing where on that spectrum each access
scenario should sit — not just turning MFA on and considering the
work done.

The interaction between a Google product (Google Authenticator) and
an Okta product (Okta Verify) in the same policy framework also
highlighted that enterprise IAM is inherently multi-vendor. Real
organizations use authenticators from multiple providers, and the
IAM platform needs to manage them all coherently. Understanding how
Okta accommodates third-party authenticators within its own policy
framework is practically relevant for any IAM role.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: Security > Authenticators page after adding Google
> Authenticator, Enrollment tab showing Default Policy with both
> authenticators as optional, Hugo Santos sign-in experience showing
> both authenticators as Required now, Okta Verify setup completion,
> Google Authenticator status change to Optional after Okta Verify
> enrollment, and successful Hugo Santos sign-in confirmation.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Related labs

- **Lab 3.4** — Enforce user verification when enrolling in Okta Verify
  *(builds on Okta Verify enrollment introduced here — adds biometric
  and device passcode requirements to the enrollment process)*
- **Lab 3.5** — Add an enrollment policy for Okta Verify *(creates a
  custom enrollment policy that replaces the Default Policy behavior
  observed in this lab — scopes enrollment requirements by network zone
  and user group)*
- **Lab 3.8** — Add rules based on network zones to the Okta Dashboard
  authentication policy *(authentication policy that drove the Required
  now behavior in this lab — Lab 3.8 builds a more sophisticated version
  of that policy with network zone conditions)*
- **Lab 3.9** — Set up a password policy *(password is the knowledge
  factor that pairs with possession factors like Google Authenticator
  and Okta Verify in two-factor authentication scenarios)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
