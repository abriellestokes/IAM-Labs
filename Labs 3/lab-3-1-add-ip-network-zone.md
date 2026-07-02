# Lab 3.1 — Add an IP Network Zone for the Corporate Network

**Path:** Implement Security Policies  
**Platform:** Okta Admin  
**Status:** ✅ Completed

---

## Objective

Create an IP-based network zone in Okta that identifies and defines the
corporate network by its gateway IP address — establishing a trusted
network boundary that can be referenced in authentication and enrollment
policies to apply different access rules based on where a user is
signing in from.

---

## Business scenario

The organization wants to apply different authentication requirements
depending on whether a user is signing in from inside the corporate
network or from a public network. Before any network-based policy can
be enforced, the network itself must be defined in Okta as a named zone.

As the IAM administrator, I created a Corporate Network zone using the
current gateway IP address. Once defined, this zone becomes a reusable
policy condition — referenced in authentication policies, enrollment
policies, and session policies throughout the remaining labs in Path 3.

This lab is foundational. Every network-aware security policy built in
Labs 3.2 through 3.9 depends on the zones defined here and in Lab 3.2.

---

## What I configured

### Step 1 — Navigated to Security > Networks

From the Okta Admin Console, I navigated to **Security > Networks**,
which is the central location for managing all network zone definitions
in the org.

### Step 2 — Created the Corporate Network IP zone

I selected **Add zone > IP Zone** and configured the zone with the
following values:

| Field | Value |
|-------|-------|
| Zone name | Corporate Network |
| Gateway IPs | Current IP address (added via the "Add your current IP address" option) |

I saved the configuration and verified from the Networks page that the
Corporate Network zone showed as **active**.

**Why gateway IPs, not individual device IPs:**  
An organization's devices share a common external-facing IP address
assigned by their internet service provider or network gateway — this
is the gateway IP. Individual device IPs (assigned internally via DHCP)
are not visible to Okta, which evaluates network zones based on the
public-facing IP of the connection. Defining the zone by gateway IP
means all devices connecting through that gateway are recognized as
being on the corporate network — regardless of which internal IP they
hold.

---

## Understanding Okta network zones

Okta supports two types of network zones, both introduced across Labs
3.1 and 3.2:

| Zone type | Defined by | Best for |
|-----------|-----------|----------|
| IP Zone | Specific IP addresses or CIDR ranges | Corporate offices, data centers, VPNs with known IP ranges |
| Dynamic Zone | Geographic locations (countries, regions) | Allowlisting or blocklisting access by country |

Network zones are not enforcement mechanisms on their own — they are
named references. The enforcement happens in policies that use those
zone names as conditions. This separation of definition from enforcement
is an important design principle: zones can be updated (for example,
when an office moves and gets a new IP) without modifying every policy
that references them. Update the zone once; all policies that reference
it inherit the change automatically.

---

## Network zones as a Zero Trust building block

Zero Trust security operates on the principle that no network location
is inherently trusted — every access request must be verified regardless
of where it originates. Network zones in Okta do not contradict this
principle — they extend it.

Rather than trusting the corporate network unconditionally, Okta uses
network zones to *differentiate* authentication requirements based on
location. A user on the corporate network may be required to use a
possession factor; a user on a public network may be required to use
a hardware-protected possession factor. The corporate network zone
reduces friction for verified employees without eliminating the
authentication requirement entirely.

This is the practical implementation of Zero Trust: **verify always,
but calibrate the verification requirement based on risk context.**
Network location is one signal in that risk calculation — not a
free pass.

---

## Why this matters

**Network-aware authentication is a foundational enterprise security
control.** Organizations that apply the same authentication requirements
regardless of network location are either over-securing low-risk access
(creating friction) or under-securing high-risk access (creating
vulnerability). Network zones allow authentication policy to be
calibrated to the risk context of each access request.

**Zone definitions are reusable policy infrastructure.** Creating the
Corporate Network zone once makes it available as a condition across
every authentication policy, enrollment policy, and session policy in
the org. This is infrastructure, not just configuration — the zone
defined here is referenced in Labs 3.5, 3.7, 3.8, and 3.9.

**IP-based zones support VPN enforcement patterns.** In organizations
where remote employees connect through a corporate VPN, the VPN exit
IP can be added to the Corporate Network zone. This means remote
employees who connect through the VPN are treated as corporate network
users and receive the same policy treatment as on-site employees —
a common and practical remote work security pattern.

**Centralized zone management reduces policy maintenance overhead.**
If an organization's office moves and its gateway IP changes, only the
zone definition needs to be updated. Every policy that references the
Corporate Network zone automatically reflects the new IP without any
policy-level changes. This is the operational efficiency benefit of
separating zone definition from policy enforcement.

---

## Key concepts applied

| Concept | Application in this lab |
|---------|------------------------|
| IP network zone creation | Defined the Corporate Network zone using the current gateway IP |
| Gateway IP vs. device IP | Understood why public-facing IP is used rather than internal device IP |
| Zone as reusable policy reference | Established a named zone for use across multiple policies in subsequent labs |
| Network-aware authentication | Understood how zone definitions enable location-based policy differentiation |
| Zero Trust context | Framed network zones as risk signals, not trust grants |

---

## Lessons learned

The most significant conceptual shift in this lab was understanding the
separation between zone definition and policy enforcement. Before this
lab, I would have assumed that defining a network zone was itself a
security action — that marking something as "Corporate Network" implied
that traffic from that network was trusted or treated differently by
default. It does not. The zone is simply a named reference. What happens
to users connecting from that zone is determined entirely by the policies
that reference it.

This separation is elegant from a governance perspective. It means a
security team can change what happens when a user connects from the
corporate network — by updating policy — without touching the zone
definition. And they can change the IP addresses that constitute the
corporate network — by updating the zone — without rewriting every
policy. The two concerns are decoupled, which makes the system more
maintainable and less error-prone.

The gateway IP concept also clarified something I had not fully
considered before: Okta does not see individual device IPs. It sees
the public-facing IP of the connection. This means network zone
evaluation is inherently about the network path, not the device — which
has implications for VPN configurations, split-tunnel setups, and
multi-office environments where different locations have different
gateway IPs.

---

## Screenshots

> 📁 Screenshots for this lab are available in the
> [Lab Screenshots folder](https://drive.google.com/your-folder-link-here)
>
> Includes: Security > Networks page before zone creation, Add zone >
> IP Zone configuration form with Corporate Network name and gateway IP,
> and Networks page after creation showing Corporate Network zone as active.

*Replace the link above with your actual Google Drive or GitHub-hosted
screenshot folder link.*

---

## Related labs

- **Lab 3.2** — Add a dynamic network zone for allowed countries *(creates
  the Allowed Countries zone — the second zone definition that pairs with
  Corporate Network across all security policy labs)*
- **Lab 3.5** — Add an enrollment policy for Okta Verify *(references the
  Allowed Countries zone to control where MFA enrollment is permitted)*
- **Lab 3.7** — Add a rule to the default global session policy *(session
  policy applies globally — network zones provide the context for
  differentiating session behavior in more advanced configurations)*
- **Lab 3.8** — Add rules based on network zones to the Okta Dashboard
  authentication policy *(directly references both Corporate Network and
  Allowed Countries zones as policy conditions — the primary consumer of
  the zones defined in Labs 3.1 and 3.2)*
- **Lab 3.9** — Set up a password policy *(references the Allowed Countries
  zone to control where self-service account recovery is permitted)*

---

*Part of the [IAM-Labs portfolio](https://github.com/abriellestokes/IAM-Labs)
by Abrielle Stokes*
