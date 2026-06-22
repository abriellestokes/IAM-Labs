# Lab 1.1 — Add a Custom Attribute

**Path:** Define Your Users in Okta  
**Platform:** Okta   
**Status:** ✅ Completed

---

## Objective

Add a custom attribute to the Okta universal user profile to capture
geographic region data for all users in the organization.

---

## Business scenario

The organization needs to track which geographic region each user belongs
to — Americas (AMER), Asia-Pacific (APAC), or Europe, Middle East & Africa
(EMEA). This data will be used downstream for access control policies,
reporting, and group rule automation.

Rather than storing this information externally, the attribute is added
directly to the Okta user profile schema so it travels with the identity
record and can be referenced in policies, mappings, and group rules.

---

## What I configured

### Step 1 — Navigated to the Profile Editor

From the Okta Admin Console, I went to **Directory > Profile Editor** and
selected the **Okta User (default)** profile. This is the base schema that
applies to all users in the org.

### Step 2 — Added a new custom attribute

I added a new attribute with the following values:

| Field | Value |
|-------|-------|
| Data type | String |
| Display name | Region |
| Variable name | `region` |
| Description | Geographic region |
| User permission | Read Only |

Setting the data type to **string** allows the field to store text values.
Using **Read Only** for the user permission means end users can view their
region assignment but cannot change it — only administrators can set or
update this value. This is an important access control decision, not just
a configuration detail.

### Step 3 — Defined an enumerated value list

Rather than allowing free-text entry (which creates data quality problems),
I defined a constrained list of acceptable values:

| Display name | Value |
|--------------|-------|
| AMER | AMER |
| APAC | APAC |
| EMEA | EMEA |

Using an enumerated list ensures consistency across all user records.
Without this constraint, the same region could be entered as "Americas",
"america", "US", or "North America" — making downstream policy rules and
reporting unreliable.

### Step 4 — Saved and verified

After saving the attribute, I verified it appeared correctly in the Okta
User profile schema before proceeding to subsequent labs that depend on
this field.

---

## Why this matters

Custom attributes are fundamental to identity governance. In a production
environment, the `region` attribute serves several real-world functions:

**Access control** — Authentication and authorization policies can reference
region to apply different rules based on where a user is geographically
assigned. For example, users in EMEA may be subject to different compliance
requirements than users in AMER.

**Group automation** — Once `region` is in the user profile, group rules
can automatically assign users to region-specific groups (as demonstrated
in the group rules labs). This removes the need for manual group management
at scale.

**Attribute mapping** — Profile attributes can be mapped to downstream
applications via SAML or OIDC. An application that needs to know a user's
region can receive it as a claim rather than requiring a separate lookup.

**Audit and reporting** — Structured, enumerated attributes make reporting
meaningful. You can filter system logs and user reports by region to
investigate access patterns or compliance posture.

**Read-only enforcement** — Setting user permission to Read Only reflects
the principle of least privilege applied to identity data itself. Users
should not be able to self-assign regions, which could be used to
circumvent location-based access controls.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| Profile schema management | Extended the default Okta user profile with a custom field |
| Data type selection | Chose `string` to support text-based enumerated values |
| Enumerated attributes | Constrained input to AMER, APAC, EMEA to ensure data integrity |
| User permission model | Applied Read Only to prevent self-service modification |
| Least privilege | Restricted attribute write access to administrators only |

---

## Lessons learned

Before this lab, I understood user profiles at a surface level — name,
email, department. What this lab made clear is that the profile schema
is the foundation of everything downstream: group rules, access policies,
and application integrations all depend on clean, structured identity data.

The decision to use an enumerated list instead of free text is an example
of proactive governance. It costs very little to implement at setup and
prevents significant data quality problems at scale. In a real organization
with thousands of users, inconsistent attribute values can break automated
policies in ways that are difficult to trace.

The Read Only permission was also more significant than it appeared. It's
not just a UX decision — it's an access control boundary. Allowing users
to edit their own region could create a path to bypass location-based
authentication rules, which is a real security consideration.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: Profile Editor view, custom attribute configuration,
> enumerated value list, and saved attribute in user schema.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Related labs

- **Lab 1.2** — Create users in Okta *(uses the `region` attribute when
  building user profiles)*
- **Lab 2.3** — Add a group rule based on a user attribute *(automates
  group membership using profile attributes like `region`)*
- **Lab 3.2** — Add a dynamic network zone for allowed countries
  *(complements region-based identity governance with network-level
  controls)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
