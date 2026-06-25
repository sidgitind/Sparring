# STARTER KIT: Consumer App

> For: mobile-first or web consumer products where user experience
> quality directly drives retention and growth.
> Examples: productivity apps, social tools, habit trackers, content apps.

---

## Persona Stack

```
1. Google PM           Persona/PM_Persona/google_pm_data_driven.md
2. Google Architect    Persona/Architect_Persona/google_distributed_architect.md
3. UI Designer         Persona/Designer_Persona/ui_designer_interaction_clarity.md
4. QA Functional       Persona/QA_Persona/qa_functional_path_coverage.md
5. QA NFQ              Persona/QA_Persona/qa_nfq_resilience.md
```

> **Worked example:** This is the stack used in `/Example/` — the notification preferences spec runs through this exact pipeline. Read it before your first run.

---

## Behavior Switch Configuration

```yaml
Google PM:
  STRICTNESS:   high
  OUTPUT:       summary
  GATE_MODE:    block

Google Architect:
  STRICTNESS:   high
  OUTPUT:       summary
  GATE_MODE:    block
  SCALE_MODE:   proportional    # Consumer scale, not enterprise

UI Designer:
  STRICTNESS:   high
  OUTPUT:       summary
  GATE_MODE:    block
  COPY_MODE:    strict          # Consumer apps: microcopy is UX

QA Functional:
  STRICTNESS:   high
  OUTPUT:       summary
  GATE_MODE:    block
  SECURITY_MODE: auth-only      # Standard OAuth, not enterprise SSO

QA NFQ:
  STRICTNESS:   high
  OUTPUT:       summary
  GATE_MODE:    block
  SCALE_MODE:   proportional
```

---

## PROJECT CONFIG — Fill In Before Running

Copy this block into every persona's PROJECT CONFIG section:

```
## Consumer App Project Config

### Product context
Product name:           [your product name]
Platform target:        [iOS / Android / Web / cross-platform]
Current user base:      [e.g. "early access, ~500 users"]
Product maturity:       [MVP / post-launch / growth]

### Metrics
Analytics tool:         [e.g. Amplitude, Mixpanel, Firebase]
North Star metric:      [e.g. DAU, task completion rate, session length]
Current OKRs:           [or "not formally defined"]

### Architecture
External services:      [list: e.g. Firebase Auth, Stripe, SendGrid]
Design system:          [name or "not yet defined"]
Defined timeouts:       [from Architecture.md or "not yet defined"]

### QA context
Auth mechanism:         [e.g. Firebase Auth, Auth0, custom OAuth]
Known regression areas: [e.g. "all protected routes"]
Security standard:      [e.g. OWASP Top 10 auth, "not formally defined"]
```

---

## Why This Stack for Consumer Apps

**Google PM over Amazon PM:**
Consumer apps iterate fast. Launch-and-learn beats upfront specification
depth for most features. The key need is instrumentation — knowing what
users actually do. Google PM enforces this.

**UI Designer with strict copy mode:**
In consumer apps, microcopy is the product experience.
"Something went wrong" loses users. "We couldn't save that — tap to retry"
retains them. Error messages, empty states, and loading copy are not details.
They are the quality signal that determines ratings and reviews.

**QA Functional auth-only security mode:**
Most consumer apps use established auth providers (Firebase, Auth0, Cognito).
The provider handles the OAuth security surface. Auth-only mode focuses
security testing on the integration layer, not a full OWASP audit.
If you are building custom OAuth: switch to SECURITY_MODE=full.

**Proportional NFQ:**
Consumer apps at early scale do not need enterprise SRE rigour.
They do need defined timeouts and failure shapes — these apply at any scale.
SLOs are measured, not mandated, until paying customers arrive.

---

## What to Watch For

**The trap specific to consumer apps:**
Google PM's comfort with implicit customer knowledge means assumptions
go undocumented. Run the Assumptions Log from the Amazon PM output
template manually if the feature touches a new user segment or a new
behaviour you have not validated.

**The tension specific to consumer apps:**
UI Designer's strict copy mode will produce a long list of microcopy
requirements. This is correct — consumer UX lives in copy. But it can
feel like over-specification on internal or developer-facing features.
Apply strict copy mode to user-facing features. Switch to standard for
anything the end user never sees.
