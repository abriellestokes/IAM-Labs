# Lab 2.2 — Add a Group Rule Based on Group Membership

**Path:** Organize Users with Groups  
**Platform:** Okta Admin
**Status:** ✅ Completed

---

## Objective

Create two new groups and configure an automated rule that assigns users
to those groups based on their membership in an existing group — eliminating
the need for manual assignment and ensuring access stays synchronized as
group membership changes.

---

## Business scenario

The organization's executive team requires access to two sensitive
application categories: revenue applications and intellectual property
resources. Rather than maintaining three separate group memberships
manually for each executive, the organization wants membership in the
Executives group to automatically grant membership in the Revenue and
Intellectual Property groups.

As the IAM administrator, I created both target groups and built a
rule that uses the Executives group as its condition. Any user added
to or removed from the Executives group is automatically added to or
removed from both downstream groups — with no manual intervention
required.

This lab directly addresses the access drift risk identified in Lab 2.1:
when group membership drives downstream access automatically, there is no
gap between a user's role assignment and their actual access state.

---

## What I configured

### Step 1 — Created the target groups

From the Okta Admin Console, I navigated to **Directory > Groups** and
created two new groups:

| Name | Description |
|------|-------------|
| Revenue | Users who need access to revenue apps |
| Intellectual Property | Users who need access to intellectual property |

Both groups were created empty. Their membership will be managed entirely
by the group rule configured in the next step — no users will ever be
manually assigned to these groups.

**Design decision — rule-managed groups should not accept manual members:**
In a production environment, groups governed by automation rules should
be clearly documented as rule-managed. Manually adding users to a
rule-managed group creates inconsistency: the manual member may not be
removed when the rule condition no longer applies, leading to lingering
access that the rule alone cannot clean up. Governance documentation
should flag these groups so administrators know not to add members
manually.

### Step 2 — Created the group rule

From the Groups page, I selected the **Rules** tab and added a new rule
with the following configuration:

**Rule name:** Group Assignments for Executives

**IF condition:**
- Condition type: Group membership
- Source group: Executives

**THEN condition:**
- Assign users to: Revenue, Intellectual Property

I saved the rule, then activated it by selecting **Actions > Activate**
from the Group Assignments for Executives rule row.

### Step 3 — Verified automatic membership propagation

After activating the rule, I navigated to both the Revenue group and the
Intellectual Property group and confirmed that all three members of the
Executives group — Jay Kumar, Sarah James, and Ping Yang — had been
automatically added to both groups.

The rule evaluated immediately upon activation and populated both target
groups without any additional administrator action.

### Step 4 — Reviewed the event log

From the Intellectual Property group, I selected **View logs** and
verified the system log captured the full sequence of events:

| Event | Target | Result |
|-------|--------|--------|
| Create Okta group | Intellectual Property (UserGroup) | SUCCESS |
| Group membership rule triggered | Group Assignments for Executives (PolicyRule) / Intellectual Property (UserGroup) | SUCCESS |
| Add user to group membership | Jay Kumar (User) / Intellectual Property (UserGroup) | SUCCESS |
| Add user to group membership | Sarah James (User) / Intellectual Property (UserGroup) | SUCCESS |
| Add user to group membership | Ping Yang (User) / Intellectual Property (UserGroup) | SUCCESS |

The log captures not just the membership additions but the rule trigger
event itself — **Group membership rule triggered** — which identifies the
policy rule responsible for the change. This is critical for auditability:
when an auditor asks why a user has access to a sensitive group, the log
answers with both the membership event and the rule that caused it.

---

## How group membership rules work

A group membership rule in Okta evaluates continuously. When the IF
condition changes — a user is added to or removed from the source group —
the rule re-evaluates and updates target group memberships accordingly.

```
[Executives group]
       │
       │  IF user is member of Executives
       ▼
[Group Assignments for Executives Rule]
       │
       ├──► [Revenue group]
       └──► [Intellectual Property group]
```

**What happens when a user is removed from Executives:**
If Jay Kumar leaves the executive team and is removed from the Executives
group, the rule automatically removes him from both Revenue and Intellectual
Property as well. The access revocation is immediate and complete — no
manual cleanup required.

**What happens when a new user joins Executives:**
If a new executive is added to the Executives group, the rule
automatically adds them to Revenue and Intellectual Property. Onboarding
to the executive access tier requires only a single group assignment.

This is the core value of rule-based group management: **a single source
of truth drives access across multiple downstream groups simultaneously.**

---

## Why this matters

**Rule-based group membership eliminates the operational lag between
role change and access change.** In a manual model, when an executive
leaves the team, an administrator must remember to remove them from the
Executives group, the Revenue group, and the Intellectual Property group
separately. Forgetting any one of those steps leaves residual access.
With rule-based automation, removing the user from Executives propagates
the revocation everywhere the rule applies — instantly and completely.

**Nested group logic mirrors real organizational structures.** Organizations
naturally have role hierarchies: executives are a subset of employees;
finance team members are a subset of all staff. Modeling those hierarchies
as group rules means access follows role, not manual process. When the
organizational structure changes, updating the group membership is
sufficient — the access model updates itself.

**The rule event in the audit log adds a layer of explainability.**
Manual group assignments show who was added and when. Rule-triggered
assignments show who was added, when, and *why* — because the triggering
rule is captured in the log. This makes access reviews significantly
faster: instead of tracing back through tickets and emails to understand
why a user has access, an auditor can see the rule that granted it
directly in the system log.

**Rule-managed groups scale without adding administrative overhead.**
A manual model requires administrator effort proportional to the number
of users and groups. A rule-based model requires effort once — to design
and configure the rule — and then scales automatically. Adding a tenth
executive to the organization requires exactly the same administrative
effort as adding the first: one group assignment in Executives.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| Rule-based group assignment | Automated membership in Revenue and IP groups via group membership condition |
| Group membership inheritance | Executives group members automatically populate downstream groups |
| Rule activation | Activated rule and observed immediate membership propagation |
| Access synchronization | Verified that rule-managed groups reflect source group membership accurately |
| Audit trail for automated actions | Confirmed rule trigger event captured alongside membership events in log |
| Access drift prevention | Eliminated manual maintenance requirement for two downstream groups |

---

## Lessons learned

The moment that landed most clearly in this lab was watching the rule
activate and both target groups populate immediately — without touching
either group directly. That transition from manual effort to automated
propagation made the operational value of rule-based assignment concrete
rather than theoretical.

The audit log entry for **Group membership rule triggered** was also
significant. In Lab 2.1, the log showed membership events attributed to
an administrator action. In this lab, the log shows membership events
attributed to a policy rule. That distinction matters in an access review:
knowing that access was granted by a rule — rather than a one-off admin
decision — tells the auditor the access is governed, intentional, and
repeatable. Rule-attributed access is easier to defend than manually
attributed access.

The design decision about rule-managed groups not accepting manual members
also felt important to document. It is easy to imagine a scenario where
an administrator manually adds someone to a rule-managed group as a
temporary measure — and then forgets to remove them. The rule will not
clean up manually added members. Understanding this boundary is essential
for maintaining group integrity in production.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: Revenue and Intellectual Property group creation, Rules tab
> with Group Assignments for Executives rule, rule configuration showing
> IF/THEN conditions, populated Revenue group membership, populated
> Intellectual Property group membership, and system log showing rule
> trigger and membership events.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Related labs

- **Lab 2.1** — Manually assign users to a group *(created the Executives
  group used as the source condition in this lab's rule)*
- **Lab 2.3** — Add a group rule based on a user attribute *(extends
  rule-based assignment from group membership conditions to user profile
  attribute conditions)*
- **Lab 2.4** — Use the Okta Expression Language in a group rule *(combines
  multiple attribute conditions into a single rule using OEL — the next
  level of rule complexity beyond this lab)*
- **Lab 3.5** — Add an enrollment policy for Okta Verify *(demonstrates
  how group membership — specifically the Pilot Users group — is used to
  scope security policy enforcement, the same principle applied here)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
