# STARTER KIT: SaaS B2B

> For: business software with multiple user roles, external integrations,
> and paying customers who expect reliability.
> Examples: team productivity tools, CRM, project management, HR platforms.
> This is the Flowdesk model — the worked example in this library uses this kit.

---

## Persona Stack

```
1. Amazon PM           personas/pm/amazon-pm-working-backwards.md
2. Google Architect    personas/architect/google-distributed-architect.md
3. UI Designer         personas/ui-designer/ui-designer-interaction-clarity.md
4. QA Functional       personas/qa-functional/qa-functional-path-coverage.md
5. QA NFQ              personas/qa-nonfunctional/qa-nfq-resilience.md
```

---

## Behavior Switch Configuration

```yaml
Amazon PM:
  STRICTNESS:   high
  OUTPUT:       detailed
  GATE_MODE:    block

Google Architect:
  STRICTNESS:   high
  OUTPUT:       detailed
  GATE_MODE:    block
  SCALE_MODE:   proportional    # B2B SaaS scale — not planet scale

UI Designer:
  STRICTNESS:   high
  OUTPUT:       detailed
  GATE_MODE:    block
  COPY_MODE:    standard        # B2B users tolerate more technical language

QA Functional:
  STRICTNESS:   high
  OUTPUT:       detailed
  GATE_MODE:    block
  SECURITY_MODE: auth-only      # Standard OAuth — upgrade to full for
                                # enterprise compliance features

QA NFQ:
  STRICTNESS:   high
  OUTPUT:       detailed
  GATE_MODE:    block
  SCALE_MODE:   proportional    # Paying customers: SLO blocks on critical paths
```

---

## PROJECT CONFIG — Fill In Before Running

```
## SaaS B2B Project Config

### Product context
Product name:           [your product name]
User roles:             [e.g. Team Member, Team Lead, Admin]
Customer type:          [SMB / mid-market / enterprise / mixed]
Product maturity:       [MVP / paying customers / scaling]

### Metrics
Analytics tool:         [e.g. Amplitude, Mixpanel, custom]
North Star metric:      [e.g. weekly active teams, tasks completed]
Current OKRs:           [or "not formally defined"]
SLA commitments:        [uptime SLA if any, or "none yet"]

### Architecture
External services:      [list: e.g. Google OAuth, SendGrid, Stripe]
Design system:          [name or "not yet defined"]
Defined timeouts:       [from ARCHITECTURE.md]
Database:               [e.g. PostgreSQL, MySQL]

### QA context
Auth mechanism:         [e.g. Google OAuth 2.0 + JWT]
User roles in auth:     [e.g. "all roles use same login flow"]
Known regression areas: [e.g. "JWT middleware, all protected routes"]
Compliance:             [SOC 2 / GDPR / none]
```

---

## Why This Stack for SaaS B2B

**Amazon PM over Google PM:**
B2B products serve multiple buyer roles. The Team Member, Team Lead,
and Admin may all use the same feature differently. Amazon PM's demand
for explicit customer definition catches the assumption that "users"
means the same thing in every context. It rarely does.

B2B features also tend toward Type 1 decisions — data model changes,
API contracts, permission structures. These are hard to reverse.
Amazon PM's rigour on irreversible decisions is appropriate here.

**UI Designer with standard copy mode:**
B2B users are more tolerant of technical language than consumers.
"Authentication failed" is acceptable where "We couldn't sign you in"
is preferable for consumer products. Standard mode still blocks on
missing error states — it just allows slightly more technical wording.

**QA NFQ proportional with SLO blocks:**
B2B products with paying customers need measurable reliability commitments.
SLOs on critical paths (login, core workflow, data writes) are not optional
once revenue exists. QA NFQ proportional mode enforces this.

**SECURITY_MODE auth-only:**
Standard B2B OAuth and JWT. Upgrade to full security mode if:
- Your product handles regulated data (health, financial, legal)
- An enterprise customer requires SOC 2 / ISO 27001
- You are adding SAML or enterprise SSO

---

## The Two Features This Kit Was Built On

The Flowdesk worked example in `/mockup-project/worked-example/` uses
this starter kit. Reading the worked example before starting your own
project is strongly recommended — it shows what each persona catches
on a realistic B2B auth feature.

Specific findings from the Flowdesk example relevant to all B2B auth:
- State parameter generation is almost never in the first-draft spec
- Error messages default to machine-readable codes without UI Designer review
- Token expiry boundary behaviour (EDGE-003 pattern) recurs on every
  JWT implementation — it is not an edge case

---

## What to Watch For

**The trap specific to B2B:**
Amazon PM will ask "who is the customer?" B2B products have multiple
answers — the user, the team lead, and the admin may all be customers
with competing needs. The PM review will surface this tension.
Do not collapse all three into "users." Name them separately.

**The role escalation trap:**
B2B features often have role-based access. A feature available to
Team Members and Team Leads but not Admins (or vice versa) has
three customer scenarios, not one. Amazon PM will flag this.
QA Functional will flag missing permission failure test cases.
Both are correct. Plan for the permutation count.

**The compliance escalation path:**
If a feature touches user data in a regulated category (personal,
financial, health), upgrade QA Functional to SECURITY_MODE=full
and add explicit data handling ACs to the spec before Architect review.
Retrofitting compliance after build is the most expensive kind of spec gap.
