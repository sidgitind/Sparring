# STARTER KIT: Startup MVP

> For: pre-revenue products validating assumptions, where speed matters
> more than completeness and the cost of over-specification exceeds
> the cost of some production bugs.
> Examples: first version of anything, validation builds, prototype-to-product.

---

## Persona Stack

```
1. Google PM           Persona/PM_Persona/google_pm_data_driven.md
2. Google Architect    Persona/Architect_Persona/google_distributed_architect.md
3. QA Functional       Persona/QA_Persona/qa_functional_path_coverage.md  [user-facing only]
4. QA NFQ              Persona/QA_Persona/qa_nfq_resilience.md
```

UI Designer is optional. See guidance below.

---

## Behavior Switch Configuration

```yaml
Google PM:
  STRICTNESS:   medium          # Faster reviews — warn more, block less
  OUTPUT:       summary          # Briefer output — block items only in detail
  GATE_MODE:    block

Google Architect:
  STRICTNESS:   medium
  OUTPUT:       summary
  GATE_MODE:    block
  SCALE_MODE:   proportional

UI Designer (if included):
  STRICTNESS:   medium
  OUTPUT:       summary
  GATE_MODE:    warn             # Warn on missing states, don't block MVP velocity
  COPY_MODE:    standard

QA Functional:
  STRICTNESS:   medium
  OUTPUT:       summary
  GATE_MODE:    block
  SECURITY_MODE: auth-only

QA NFQ:
  STRICTNESS:   medium
  OUTPUT:       summary
  GATE_MODE:    block
  SCALE_MODE:   proportional    # MVP: SLO WARNs not BLOCKs
```

---

## PROJECT CONFIG — Fill In Before Running

```
## Startup MVP Project Config

### Product context
Product name:           [your product name]
What you are validating: [the assumption this build tests]
Time horizon:           [e.g. "2-week build to first users"]
Product maturity:       MVP

### Metrics
Analytics tool:         [or "not yet set up"]
The one metric that matters: [what tells you the MVP worked or didn't]
OKRs:                   [or "not applicable at this stage"]

### Architecture
External services:      [list]
Design system:          [or "none — using component library defaults"]
Defined timeouts:       [or "not yet defined — using Architecture.md defaults"]

### QA context
Auth mechanism:         [e.g. Auth0, Firebase Auth, custom]
Security standard:      [OWASP auth basics, "not formally defined"]
```

---

## When to Include UI Designer

Include UI Designer when:
- The feature is the primary user-facing flow (onboarding, login, core action)
- The feature will be shown to real users or investors in its current form
- Missing error states or loading states would cause real users to abandon

Skip UI Designer when:
- The feature is an internal API with no direct user interaction
- The feature is a backend process (job queue, data pipeline, webhook handler)
- You are building a throwaway prototype to validate a single assumption

When you skip UI Designer: add one manual check to your process.
Before handing the spec to the agent, answer:
"What does the user see when this fails?" If you cannot answer in one
sentence, you need the UI Designer. [Likely]

---

## Why This Stack for Startup MVP

**Google PM over Amazon PM:**
At MVP stage, you are validating assumptions, not delivering known solutions.
The most valuable PM contribution is instrumentation — can you measure
whether the feature achieved its purpose? Google PM enforces this.
Amazon PM's demand for full customer narrative adds overhead that MVP
velocity cannot absorb. Customer knowledge is often implicit at this stage.

**Medium strictness everywhere:**
High strictness produces the most thorough reviews. It also produces
the most blocking items. At MVP stage, the cost of each blocking item
is higher relative to the value of the feature. Medium strictness keeps
the library useful without turning a 2-day feature into a 5-day spec exercise.

**NFQ with proportional SLOs:**
The Minimum NFQ Contract still applies in full at MVP stage.
Timeouts, failure shapes, state write policy — these are non-negotiable
regardless of product maturity. The scaling is on SLOs only.
An MVP that crashes on every Google OAuth timeout is not an MVP —
it is a broken product that teaches nothing useful about user behaviour.

---

## The Non-Negotiable Minimum at MVP Stage

Even at MVP, these five things cannot be skipped:

```
1. Every external service call has an explicit timeout.
   (Not Node.js default. An explicit number.)

2. Every external service call has a defined failure response.
   (Not "handle errors somehow." A specific HTTP status and shape.)

3. No auth state written before external service confirmation.
   (Ghost sessions corrupt your user data from day one.)

4. The primary failure mode has a recovery path.
   (One failure + no recovery = user abandons + no learning.)

5. Success is measurable.
   (One metric. One way to know if the MVP worked.)
```

These five are the Minimum NFQ Contract applied to MVP context.
Everything else in the library is optional at this stage.
These five are not. [Certain]

---

## What to Watch For

**The velocity trap:**
Medium strictness and summary output make reviews faster.
They do not make the Minimum NFQ Contract optional.
QA NFQ will still block on missing timeouts and missing failure shapes
regardless of STRICTNESS setting. This is intentional.

**The instrumentation gap:**
Google PM will block on missing success metrics.
At MVP stage, the most common human response is "we'll add metrics later."
This is the single most expensive decision in product development.
Without a baseline metric at launch, you cannot distinguish
"this feature worked" from "this feature failed quietly."
Google PM's block on missing metrics is the right call even at MVP stage.

**The debt accumulation pattern:**
Medium strictness at MVP means some conditional items are flagged
but not blocked. Track these. They are not permanently dismissed —
they are deferred to post-validation. Create a debt register in
AGENT_CONTEXT.md and note every conditional item that was not resolved.
Before your first paying customer, run the full pipeline at high strictness
on the features that matter most. The debt register tells you where to start.
