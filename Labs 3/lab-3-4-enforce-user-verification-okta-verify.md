# Lab 3.4 — Enforce User Verification When Enrolling in Okta Verify

**Path:** Implement Security Policies  
**Platform:** Okta Admin Console  
**Lab Guide Version:** Okta 2025.10  
**Skill level:** Intermediate  
**Estimated completion time:** 15–20 minutes  
**Status:** ✅ Completed

---

## Objective

Modify the Okta Verify authenticator settings to require device passcode
or biometric confirmation during enrollment — ensuring that every device
on which Okta Verify is registered is protected by a local user
verification mechanism before it can be used as an authentication factor.

---

## Business scenario

The organization requires that when users enroll Okta Verify on a
device, they must enable either a device passcode or biometric
confirmation (fingerprint or face recognition) as part of that
enrollment. This ensures that Okta Verify is not enrolled on
unprotected devices — where anyone with physical access to the device
could authenticate on behalf of the account owner.

As the IAM administrator, I modified the Okta Verify authenticator
settings to set the device passcode or biometric user verification
enrollment requirement to **Required**, and verified that the supported
verification methods — TOTP, Push notification, and Okta FastPass —
remained available after the change.

This is a single configuration change with significant security
implications: it raises the baseline security standard for every
Okta Verify enrollment across the entire org.

---

## What I configured

### Step 1 — Accessed Okta Verify authenticator settings

From the Okta Admin Console, I navigated to **Security > Authenticators**
and located Okta Verify in the authenticator list. I selected
**Actions > Edit** to open the authenticator configuration.

### Step 2 — Validated supported verification methods

Before making any changes, I confirmed that Okta Verify was configured
to support three verification methods:

- **TOTP** — Time-based one-time password (6-digit rotating code)
- **Push notification** — One-tap approval sent to the enrolled device
- **Okta FastPass** — Cryptographically bound, phishing-resistant
  device authentication

All three methods remained available after the configuration change —
the user verification requirement applies to the enrollment process,
not to the verification methods themselves.

### Step 3 — Set device passcode or biometric enrollment to Required

In the **Device passcode or biometric user verification** section,
I changed the Enrollment setting from **Preferred** to **Required**
and saved the changes.

**The difference between Preferred and Required:**

| Setting | Behavior |
|---------|----------|
| Preferred | The system encourages users to enable a device passcode or biometric but does not block enrollment if the device lacks this capability |
| Required | Enrollment in Okta Verify is blocked if the device does not have a passcode or biometric enabled — the user must enable device protection before proceeding |

Setting this to **Required** means Okta Verify can no longer be enrolled
on a device that has no screen lock, no PIN, no password, and no
biometric protection. The device itself must be secured before it can
become an authentication factor.

**Important environment note:**  
In lab and testing environments where the test device does not have a
passcode or biometric enabled, this setting should be left as
**Preferred** to avoid blocking the lab workflow. In production, the
determination of whether to use Required or Preferred should be based
on the organization's device management posture and acceptable use
policy.

---

## Why device-level user verification matters for MFA security

Okta Verify operates on a specific security model: the device itself
is the possession factor. A user proves they possess the enrolled
device by receiving a push notification or generating a TOTP code on
it. But this model has a vulnerability if the device is not protected:

```
Without device user verification:
─────────────────────────────────
Attacker steals or borrows unlocked device
         │
         ▼
Attacker opens Okta Verify app on device
         │
         ▼
Attacker approves push notification or reads TOTP code
         │
         ▼
Attacker successfully authenticates as the victim ❌

With device user verification required:
────────────────────────────────────────
Attacker steals or borrows locked device
         │
         ▼
Device requires passcode or biometric to unlock
         │
         ▼
Attacker cannot access Okta Verify without device credentials
         │
         ▼
Authentication attempt fails ✅
```

Device user verification transforms Okta Verify from a single-layer
possession factor into a two-layer control: the attacker needs both
the physical device (something you have) and the device unlock
credential (something you know or something you are). This is
effectively MFA within the MFA factor itself.

---

## Relationship to Okta FastPass and phishing resistance

The device passcode or biometric requirement is especially significant
for **Okta FastPass**, which uses device-bound cryptographic keys for
phishing-resistant authentication:

- FastPass generates a private key that is bound to the enrolled device
- The private key never leaves the device
- User verification (biometric or passcode) is required to use the
  private key for authentication

Without the device user verification requirement, FastPass could
theoretically be used on a device with no screen lock — meaning the
cryptographic key is accessible to anyone who picks up the device.
Requiring device user verification ensures the key is protected by
a local credential, maintaining the "something you have + something
you know/are" security model that makes FastPass a strong authenticator.

For organizations pursuing phishing-resistant MFA as a security
objective, enabling the device passcode or biometric requirement is
not optional — it is a prerequisite for FastPass to deliver its
full security value.

---

## Authenticator settings vs. enrollment policy vs. authentication policy

This lab introduces a third policy layer in Okta's MFA architecture —
authenticator-level settings — which sits alongside the enrollment
policy (Lab 3.5) and authentication policy (Lab 3.8):

| Policy layer | Where configured | Controls |
|--------------|-----------------|----------|
| Authenticator settings | Security > Authenticators > Edit | How the authenticator behaves during enrollment and use (e.g., device verification requirement, supported methods) |
| Enrollment policy | Security > Authenticators > Enrollment tab | Who can enroll in which authenticators, and under what network or group conditions |
| Authentication policy | Security > Authentication Policies | What factors are required to access a specific application or resource |

**The distinction matters for troubleshooting.** If a user cannot
enroll in Okta Verify, the issue could be in any of these three layers:
the authenticator settings may block enrollment on their device type,
the enrollment policy may deny enrollment from their network location,
or the authentication policy may require an authenticator the user
has not yet enrolled. Each layer requires a different investigation
path and a different resolution.

Understanding all three layers — and how they interact — is what
enables effective MFA troubleshooting in production environments.

---

## Why this matters

**Unprotected devices are the weakest link in a possession-factor
MFA model.** An organization that deploys Okta Verify without requiring
device protection has reduced the security of its MFA to whatever
physical security exists around users' devices. In open office
environments, co-working spaces, or shared device scenarios, this
is a meaningful gap. Requiring device user verification closes it.

**This setting applies org-wide instantly.** Unlike policies that are
scoped to groups or applications, authenticator-level settings apply
to all future enrollments across the entire org. A single configuration
change in the authenticator settings raises the security baseline for
every new Okta Verify enrollment from that point forward. This is high-
leverage configuration — one change, org-wide effect.

**Device protection requirements support broader security frameworks.**
Many security frameworks and compliance standards — NIST 800-63B, SOC 2,
ISO 27001 — include requirements for device security as part of
authentication assurance. Requiring device passcode or biometric
enrollment supports these requirements at the identity layer, providing
evidence of device security controls that auditors can verify through
Okta's system log and device registry.

**The Preferred vs. Required distinction is a risk acceptance decision.**
Choosing Preferred means the organization accepts the risk that some
users will enroll Okta Verify on unprotected devices. Choosing Required
means the organization mandates device protection as a prerequisite for
MFA enrollment. This is not a technical decision — it is a risk
management decision that should be made with input from security,
compliance, and IT leadership, with the authenticator setting as the
enforcement mechanism.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| Authenticator-level settings | Modified Okta Verify configuration to enforce device user verification |
| Device passcode / biometric enforcement | Set enrollment requirement from Preferred to Required |
| Possession factor security model | Understood how device protection strengthens the security of a device-based factor |
| FastPass security prerequisites | Connected device verification requirement to phishing-resistant FastPass security model |
| Three-layer MFA architecture | Distinguished authenticator settings from enrollment policy and authentication policy |
| Org-wide configuration impact | Recognized that authenticator settings apply globally to all future enrollments |

---

## Lessons learned

The most significant insight from this lab was understanding the security
model behind the setting — not just what the toggle does, but what attack
it defends against. The scenario of an attacker with physical access to
an unlocked device being able to approve a push notification is a real
and underappreciated threat. Device user verification is the control that
closes that gap, and it operates at a layer below the Okta policy system —
at the device itself.

The relationship between device user verification and FastPass was also
clarifying. FastPass is positioned as a phishing-resistant authenticator,
and its cryptographic architecture deserves that designation. But the
private key bound to the device is only as secure as the device's local
protection. Without a passcode or biometric, the key is accessible to
anyone who picks up the device. The Required setting is what makes the
FastPass security model complete — not just theoretically sound but
operationally enforced.

The three-layer MFA architecture table also crystallized something I
had been building across this path: Okta's MFA system is not a single
policy but a stack of interacting layers. Authenticator settings define
the authenticator's behavior. Enrollment policies define who can enroll
and when. Authentication policies define what factors are required for
access. A complete MFA design requires decisions at all three layers —
and troubleshooting requires knowing which layer a problem belongs to.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: Security > Authenticators page, Okta Verify Actions > Edit
> configuration screen, supported verification methods (TOTP, Push,
> FastPass) confirmation, Device passcode or biometric user verification
> section showing Enrollment set to Required, and saved configuration
> confirmation.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Related labs

- **Lab 3.3** — Add Google Authenticator *(introduced the authenticator
  enrollment and authentication policy interaction — this lab adds a
  third layer: authenticator-level settings that govern enrollment
  behavior independently of both policy layers)*
- **Lab 3.5** — Add an enrollment policy for Okta Verify *(builds the
  enrollment policy layer that sits above the authenticator settings
  configured here — together they define the complete Okta Verify
  enrollment experience)*
- **Lab 3.6** — Manage registered devices *(device registry is the
  downstream record of Okta Verify enrollments — device user verification
  settings configured here affect what devices can appear in that registry)*
- **Lab 3.8** — Add rules based on network zones to the Okta Dashboard
  authentication policy *(authentication policy that references hardware-
  protected possession factors — device user verification is what makes
  Okta Verify qualify as hardware-protected in applicable scenarios)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
