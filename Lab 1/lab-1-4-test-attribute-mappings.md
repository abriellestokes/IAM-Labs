# Lab 1.4 — Test Attribute Mappings

**Path:** Define Your Users in Okta  
**Platform:** Okta Admin Console + Okta Workflows  
**Lab Guide Version:** Okta 2025.10  
**Skill level:** Foundational — Intermediate  
**Estimated completion time:** 30–40 minutes  
**Status:** ✅ Completed

---

## Objective

Test and verify how user profile attributes flow from Okta's core identity
store to a connected application — Okta Workflows — using profile mappings.
Observe how mapping settings control whether attribute changes in Okta
propagate automatically to downstream applications.

---

## Business scenario

The organization uses Okta Workflows to automate business processes. Okta
Workflows maintains its own user profile, which must stay synchronized with
the master Okta user profile. As the IAM administrator, I need to verify
that attribute mappings between the two systems are correctly configured,
understand how mapping behavior settings affect data synchronization, and
confirm that changes made to a user's Okta profile propagate to Okta
Workflows as expected.

This lab simulates a real-world integration governance task: ensuring that
the identity data flowing into a connected application is accurate,
synchronized, and behaving according to the configured rules — not just
assumed to be working.

---

## What I configured

### Step 1 — Reviewed the existing attribute mappings

From the Okta Admin Console, I navigated to **Directory > Profile Editor**
and selected **Mappings** next to the Okta Workflows User profile. I then
opened the **Okta User to Okta Workflows** tab to review how attributes
flow from the master Okta profile to the Workflows profile.

The existing mappings were:

| Okta User Profile | Okta Workflows Profile |
|-------------------|----------------------|
| `user.firstName` | `givenName` |
| `user.lastName` | `familyName` |
| `user.email` | `email` |
| `isSuperOrgAdmin() ? 'superOrgAdmin' : isWorkflowsAdmin() ? 'workflowsAdmin' : 'member'` | `primaryRole` |

**The `primaryRole` mapping is notable.** Rather than mapping a static
attribute, it uses an Okta Expression Language (OEL) conditional expression
that evaluates the user's admin role at runtime:

- If the user holds the Super Org Administrator role → `primaryRole` is set
  to `superOrgAdmin`
- If the user holds the Workflows Administrator role → `primaryRole` is set
  to `workflowsAdmin`
- Otherwise → `primaryRole` defaults to `member`

This is a dynamic mapping — the value isn't stored anywhere in the user
profile, it is computed from the user's current role assignments each time
the mapping runs. This pattern is common in real integrations where
downstream applications need role-context from Okta that doesn't map
cleanly to a single stored attribute.

For the `user.firstName → givenName` mapping, I selected the green arrow
to review its behavior setting and confirmed it was set to **Apply mapping
on user create and update** — meaning every time the first name changes in
Okta, the change flows automatically to Okta Workflows.

### Step 2 — Verified the admin user's attributes in Okta Workflows

I navigated to **Workflow > Workflows Console**, opened the **Settings**
tab, and selected **Role assignments**. I verified that:

- The first name, last name, and email in Okta Workflows matched the admin
  user's profile attributes in Okta exactly
- The org-level role assignment showed **Super Org Administrator**, which
  confirmed the `primaryRole` OEL expression was resolving correctly

This step established a verified baseline before testing any mapping
behavior changes.

### Step 3 — Tested "Apply mapping on user create and update"

With the `user.firstName → givenName` mapping set to **Apply mapping on
user create and update**, I modified the admin user's first name in the
Okta profile and saved the change. I then refreshed the Okta Workflows
browser tab and confirmed that the updated first name appeared immediately
in the Workflows profile.

**Result:** Attribute change in Okta propagated to Okta Workflows
automatically. The mapping behaved as configured.

### Step 4 — Changed the mapping behavior to "Apply mapping on user create only"

I returned to the Profile Editor, opened the mappings for Okta Workflows
User, and changed the `user.firstName → givenName` mapping from **Apply
mapping on user create and update** to **Apply mapping on user create
only**. I saved the mappings and applied the updates.

With this setting active, I again modified the admin user's first name in
Okta and refreshed the Workflows tab. This time, the first name in Okta
Workflows did **not** change — it retained the previous value.

**Result:** The "create only" setting confirmed that attribute
synchronization is deliberately one-directional at the point of
provisioning. After initial account creation, subsequent changes to the
first name in Okta no longer propagate to Workflows automatically. This is
the expected behavior — and understanding *why* this setting exists is more
important than the setting itself.

### Step 5 — Reset the mapping behavior to "Apply mapping on user create and update"

To restore the correct production configuration, I returned the mapping
to **Apply mapping on user create and update** and applied the updates.
I modified the admin user's first name one final time and confirmed the
change propagated to Okta Workflows, verifying the mapping was fully
restored.

I then reset the admin user's first name to its original value to return
the environment to its baseline state.

---

## Understanding mapping behavior settings

This lab introduced two mapping behavior modes. Understanding the difference
is essential for integration configuration and troubleshooting:

| Setting | Behavior | When to use |
|---------|----------|-------------|
| Apply mapping on user create and update | Attribute syncs from Okta to the app every time it changes in Okta | When the app should always reflect the current Okta profile value |
| Apply mapping on user create only | Attribute syncs once at provisioning; subsequent Okta changes do not propagate | When the app manages the attribute independently after initial provisioning |

**Real-world example of "create only":** An HR system provisioned through
Okta may need to receive the employee's start date at the time of account
creation. But after onboarding, the HR system manages the employee record
independently. Setting the start date mapping to "create only" prevents
Okta from overwriting HR system data on every profile update.

**Real-world example of "create and update":** A helpdesk ticketing system
needs to always show the user's current display name and email. If an
employee's name changes due to a legal name change or marriage, the
ticketing system should reflect that automatically. "Create and update"
ensures the downstream app stays synchronized without manual intervention.

---

## Why this matters

**Attribute mappings are the data pipeline between Okta and every connected
application.** When mappings are misconfigured, the consequences are not
always visible immediately — but they accumulate. A user's name may be
correct in Okta but wrong in the ITSM tool. A role assignment may be
accurate in the identity store but not reflected in the application the
user is trying to access. Mapping errors are a silent source of access
and data quality problems.

**Dynamic mappings using Okta Expression Language are powerful and common.**
The `primaryRole` mapping demonstrated that not every attribute passed to
a downstream application comes from a stored field. Computed values derived
from role assignments, group memberships, or conditional logic are a
standard pattern in enterprise Okta implementations. Administrators who
understand OEL can design more intelligent integrations.

**Mapping behavior settings are a governance decision, not just a technical
one.** Choosing between "create and update" vs. "create only" has
implications for data ownership, system of record authority, and
integration reliability. IAM administrators who understand these
implications can advise on integration design, not just execute it.

**Testing integrations is a professional discipline.** It is not sufficient
to configure a mapping and assume it works. This lab demonstrated the
importance of verifying attribute flow end-to-end: from the source profile,
through the mapping configuration, to the target application profile.
Verification should be part of every integration deployment and every
significant configuration change.

**Baseline verification before testing prevents false conclusions.** By
confirming the initial state of attributes in Okta Workflows before making
changes, I ensured that observed differences were caused by the mapping
behavior change — not by a pre-existing discrepancy. This is the correct
methodology for integration testing in any environment.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| Profile mappings | Reviewed and tested attribute flow from Okta to Okta Workflows |
| Okta Expression Language (OEL) | Analyzed the `primaryRole` dynamic mapping using conditional OEL logic |
| Mapping behavior settings | Compared "create and update" vs. "create only" behavior in a live environment |
| Integration verification | Confirmed attribute synchronization end-to-end across two systems |
| Baseline testing methodology | Established known state before making configuration changes |
| Configuration restoration | Reset mapping to original setting and verified restored behavior |
| System of record governance | Understood when Okta should and should not overwrite downstream app data |

---

## Lessons learned

This lab introduced a concept that reframed how I think about identity
data: the difference between a stored attribute and a computed attribute.
The `primaryRole` mapping does not read a field — it evaluates a condition
and derives a value. This distinction matters because it means the value
seen in a downstream application is only as current as the last time the
mapping ran. If a user's role changes, the downstream value may not update
until the next provisioning event triggers the mapping.

The mapping behavior test was the most directly applicable exercise in
Path 1 to real-world troubleshooting. If a user reports that their name
is wrong in a connected application even though it is correct in Okta,
the mapping behavior setting is the first thing to check. A "create only"
setting on a name attribute would explain exactly that symptom. Having run
this test in a lab means I have direct experience with the failure mode —
not just a theoretical understanding of it.

The step of resetting the environment to its original state at the end
of the lab also reinforced a professional discipline: leave the system
in a known good state after testing. In a production environment, failing
to restore configuration after a test can create downstream inconsistencies
that are difficult to diagnose later. Restoration and verification are as
important as the test itself.

Finally, this lab completed Path 1 with a meaningful progression: Lab 1.1
built the profile schema, Lab 1.2 used that schema to provision a user,
Lab 1.3 managed that user's lifecycle, and Lab 1.4 verified how that
user's attributes flow to connected applications. Each lab built on the
last — which is exactly how identity management works in production.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: Profile Editor mappings view, Okta User to Okta Workflows tab,
> `primaryRole` OEL expression, Workflows Role assignments panel showing
> baseline attributes, first name propagation with "create and update"
> setting, first name retention with "create only" setting, and restored
> mapping confirmation.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Path 1 complete — what was built

Lab 1.4 completes the **Define Your Users in Okta** learning path. Across
four labs, the following was built and verified:

| Lab | What was built |
|-----|----------------|
| 1.1 | Extended the Okta user profile schema with a custom `region` attribute |
| 1.2 | Provisioned a user through the full activation lifecycle |
| 1.3 | Managed a user account across six status states and four operational scenarios |
| 1.4 | Verified attribute mapping behavior between Okta and a connected application |

Together, these four labs cover the complete user identity lifecycle in
Okta — from schema design through provisioning, status management, and
integration verification.

---

## Related labs

- **Lab 1.1** — Add a custom attribute *(established the profile schema
  that underlies all attribute mapping in this lab)*
- **Lab 1.3** — Manage user account statuses *(user lifecycle events trigger
  mapping runs — status changes and provisioning events are directly
  connected)*
- **Lab 2.3** — Add a group rule based on a user attribute *(extends the
  attribute-driven logic introduced here into group membership automation)*
- **Lab 2.4** — Use the Okta Expression Language in a group rule *(builds
  directly on the OEL pattern observed in the `primaryRole` mapping)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
