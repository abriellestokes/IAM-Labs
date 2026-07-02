# Lab 3.2 — Add a Dynamic Network Zone for Allowed Countries

**Path:** Implement Security Policies  
**Platform:** Okta Admin Console  
**Lab Guide Version:** Okta 2025.10  
**Skill level:** Intermediate  
**Estimated completion time:** 20–25 minutes  
**Status:** ✅ Completed

---

## Objective

Create a dynamic network zone in Okta that defines a set of allowed
countries based on geographic location — establishing a location-aware
policy boundary that can be used to permit or deny access and enrollment
based on where in the world a sign-in request originates.

---

## Business scenario

The organization operates in specific geographic regions and has no
legitimate business reason for users to authenticate from outside those
regions. Logins originating from unexpected countries represent a
meaningful risk signal — they may indicate credential theft, account
compromise, or unauthorized access attempts from threat actors operating
in restricted regions.

As the IAM administrator, I identified the country the organization
operates from using the Okta system log, created a dynamic network zone
called Allowed Countries scoped to that location, and verified the zone
was active and ready to be referenced in security policies.

This zone, paired with the Corporate Network zone from Lab 3.1, forms
the complete network policy infrastructure for all remaining labs in
Path 3.

---

## What I configured

### Step 1 — Identified the sign-in country from the system log

Before creating a location-based zone, I needed to confirm which country
the organization's authentication traffic originates from. I navigated
to **Reports > Reports** and applied the **Authentication activity**
system log filter.

For the most recent authentication event, I selected the expand arrow
and chose **Expand All** to view the full event details. In the
**Client** section, I located the **Country / region** field, which
identified the country of the active sign-in session.

**Why this step matters:**  
Creating a location-based zone without first verifying the actual sign-in
country risks locking out the administrator. If the zone is created with
the wrong country — or a country the administrator is not currently in —
and policies referencing that zone take effect, the administrator's own
access could be denied. Verifying the country in the system log before
creating the zone is a safety check, not just a configuration step.

### Step 2 — Created the Allowed Countries dynamic zone

I navigated to **Security > Networks** and selected **Add zone >
Dynamic Zone**, then configured the zone with the following values:

| Field | Value |
|-------|-------|
| Zone name | Allowed Countries |
| Locations | Country identified in system log (e.g., United States) |
| State / region | Not selected |

I saved the configuration and verified from the Networks page that the
Allowed Countries zone showed as **active**.

**Why no state or region was selected:**  
The policy intent is to allow access from anywhere within the permitted
country — not to restrict access to a specific state or metropolitan
area. Selecting a state or region would create an overly narrow zone
that could inadvertently block legitimate users who travel domestically
or work from different locations within the same country.

**Additional countries can be added:**  
For organizations with international operations, multiple countries can
be included in a single dynamic zone. The key requirement is that the
country from which the administrator is currently signing in must be
included — otherwise the administrator risks being locked out when
policies referencing the zone are activated.

---

## IP zones vs. dynamic zones — a complete comparison

Labs 3.1 and 3.2 together establish both types of network zones
available in Okta. Understanding when each is appropriate is an
important design decision:

| Attribute | IP Zone (Lab 3.1) | Dynamic Zone (Lab 3.2) |
|-----------|-------------------|------------------------|
| Defined by | Specific IP addresses or CIDR ranges | Geographic locations (countries, regions) |
| Precision | High — exact IPs or ranges | Moderate — geolocation-based |
| Best for | Corporate offices, VPNs, data centers | Country allowlisting / blocklisting |
| Updates needed when | Office moves, ISP changes gateway IP | Organization expands to new countries |
| Geolocation accuracy | Not applicable | Dependent on IP geolocation database accuracy |
| Typical policy use | Differentiate corporate vs. public network | Block restricted countries, allow known regions |

**A note on geolocation accuracy:**  
Dynamic zones rely on IP geolocation databases to map an IP address to
a country. These databases are highly accurate for most IPs but are not
perfect — VPNs, proxy servers, and Tor exit nodes may appear to originate
from a different country than the actual user. This is both a limitation
(legitimate users on VPNs may be incorrectly blocked) and a security
consideration (threat actors using geo-spoofing tools may bypass
country-based restrictions). Country-based zones are a meaningful
security layer — not an impenetrable one.

---

## How Allowed Countries is used across Path 3

The Allowed Countries zone is referenced in three of the remaining
labs in this path, each applying it in a different policy context:

| Lab | How Allowed Countries is used |
|-----|-------------------------------|
| 3.5 | Enrollment policy — Okta Verify enrollment is only permitted from allowed countries |
| 3.8 | Authentication policy — access to the Okta Dashboard is denied from countries not in the zone |
| 3.9 | Password policy — self-service account recovery (password reset, unlock) is only permitted from allowed countries |

Together, these three applications enforce a consistent geographic
boundary across enrollment, authentication, and recovery workflows —
ensuring that the Allowed Countries definition is not just a network
label but a meaningful security perimeter applied at every point in
the identity lifecycle.

---

## Geographic access control as a security layer

Country-based access restrictions are a well-established security
control in enterprise IAM, particularly for organizations that:

- Have no international workforce or customer base
- Operate in industries with strict data residency requirements
- Have experienced credential attacks originating from specific regions
- Are subject to regulatory requirements that restrict data access
  to specific geographies (GDPR, ITAR, FedRAMP, etc.)

For these organizations, blocking authentication attempts from outside
allowed countries reduces the attack surface meaningfully. A stolen
credential used from a restricted country will fail the geographic
policy check before it can be used to access systems — even if the
password itself is correct and MFA is not yet enrolled.

This does not make the system impenetrable — as noted above, geolocation
can be circumvented. But it raises the cost of an attack and reduces
the volume of automated credential-stuffing attempts that would otherwise
need to be filtered at the authentication layer.

---

## Why this matters

**Geographic access control is a proactive defense against credential
attacks.** The majority of automated credential-stuffing attacks and
brute-force attempts originate from IP ranges associated with known
threat actor infrastructure, many of which are concentrated in specific
geographic regions. A country allowlist does not require knowledge of
specific malicious IPs — it simply restricts access to the countries
where legitimate users operate, blocking everything else by default.

**The system log identification step is a governance practice.** Using
the authentication activity log to confirm the current sign-in country
before configuring a location-based policy is a methodical, evidence-based
approach to configuration. It ensures that zone definitions are grounded
in actual traffic data — not assumptions — and protects against
self-lockout, which is a real operational risk when configuring
network-based policies.

**Dynamic zones are maintenance-light once configured.** Unlike IP zones,
which require updates when office locations or ISP assignments change,
dynamic zones based on country are stable. A country does not change its
geolocation profile the way an office changes its gateway IP. For
organizations with stable geographic footprints, a country-based dynamic
zone requires minimal ongoing maintenance.

**Zone definitions are the foundation of policy layering.** With both
the Corporate Network zone (Lab 3.1) and the Allowed Countries zone
(this lab) defined, the organization now has two independent network
dimensions available for policy conditions: network type (corporate vs.
public) and geographic location (allowed vs. restricted). These two
dimensions can be combined in policies to create nuanced, context-aware
access rules — as demonstrated in Labs 3.8 and 3.9.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| Dynamic network zone creation | Defined Allowed Countries zone using geographic location |
| System log pre-verification | Used authentication activity log to confirm sign-in country before zone creation |
| Geolocation-based access control | Established country-level geographic boundary for policy enforcement |
| Self-lockout prevention | Verified current sign-in country is included in zone before policy activation |
| Zone as policy infrastructure | Established Allowed Countries as a reusable reference for enrollment, authentication, and password policies |
| IP zone vs. dynamic zone distinction | Understood the appropriate use case for each zone type |

---

## Lessons learned

The system log pre-verification step reframed how I think about
configuration sequencing. The instinct when creating a zone is to go
directly to Security > Networks and start configuring. This lab built
in a deliberate pause — check the log first, confirm the country, then
create the zone. That sequence protects against a consequential mistake:
if the zone is created with the wrong country and a restrictive policy
is applied immediately, the administrator could lose access to the org.
In a production environment, recovering from that scenario requires
contacting Okta support. The log check costs thirty seconds and prevents
a potentially hours-long recovery.

The geolocation accuracy limitation was also worth sitting with. Country-
based zones are a meaningful security layer — but they are not foolproof.
Documenting that limitation is important because it shapes how this
control should be positioned in a security strategy: as one layer among
several, not as a standalone defense. A credential attacker who knows
an organization uses country-based restrictions can route traffic through
a VPN in an allowed country. The zone blocks unsophisticated attacks
efficiently; sophisticated attackers require additional controls.

This lab also completed the network zone infrastructure that all
remaining Path 3 labs depend on. With Corporate Network and Allowed
Countries both defined and active, every security policy from Lab 3.5
onward has the network context it needs to enforce nuanced, location-
aware access rules. The foundational work done in Labs 3.1 and 3.2
will be invisible in the policy labs — which is exactly how good
infrastructure should work.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: Authentication activity system log showing Country / region
> field in Client section, Security > Networks page, Add zone > Dynamic
> Zone configuration form with Allowed Countries name and location
> selection, and Networks page after creation showing both Corporate
> Network and Allowed Countries zones as active.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Related labs

- **Lab 3.1** — Add an IP network zone for the corporate network *(created
  the Corporate Network zone — the companion zone that pairs with Allowed
  Countries across all security policy labs in Path 3)*
- **Lab 3.5** — Add an enrollment policy for Okta Verify *(uses Allowed
  Countries to restrict MFA enrollment to permitted geographic regions)*
- **Lab 3.8** — Add rules based on network zones to the Okta Dashboard
  authentication policy *(uses both Corporate Network and Allowed Countries
  as conditions in a layered authentication policy — the primary consumer
  of both zone definitions)*
- **Lab 3.9** — Set up a password policy *(uses Allowed Countries to
  restrict self-service account recovery to permitted geographic regions)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
