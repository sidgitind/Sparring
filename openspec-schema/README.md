# Sparring -- OpenSpec Schema

Adds adversarial spec review to your OpenSpec workflow. Five personas run
as separate artifacts between `design` and `tasks`. Each one is a named
step in your change folder -- skippable, resumable, part of the record.

```
design -> sparring_pm -> sparring_architect -> sparring_ui
       -> sparring_qa_functional -> sparring_qa_nfq -> sparring_brief -> tasks
```

Each persona runs one at a time. `/opsx:continue` advances the pipeline.
Human resolves blocking items after each persona before continuing.
`tasks` is blocked until all five artifacts exist and `sparring_brief` shows
PIPELINE PASS (or the affected personas are explicitly skipped).

Full persona library: [sidgitind/Sparring](https://github.com/sidgitind/Sparring)

---

## Install

**One-shot (Claude Code) -- copy and paste into Claude Code in your project root:**

```
Install the Sparring schema for OpenSpec into this project:

1. Verify the project has an openspec/ directory (run openspec init if missing).
2. Clone https://github.com/sidgitind/Sparring to a temp directory.
3. Copy Sparring/openspec-schema/openspec/schemas/sparring/ to
   openspec/schemas/sparring/ in this project.
4. Run: openspec schema validate sparring
5. If validation passes, add to openspec/config.yaml:
     schema: sparring
     sparring_library_path: [absolute path to your Sparring clone]
6. Run: openspec update
7. Confirm with: openspec schema which sparring
```

**Manual (Mac/Linux):**

```bash
git clone https://github.com/sidgitind/Sparring /tmp/sparring
cp -r /tmp/sparring/openspec-schema/openspec/schemas/sparring \
      openspec/schemas/sparring
openspec schema validate sparring
# Open openspec/config.yaml and set:
#   schema: sparring
#   sparring_library_path: /tmp/sparring
openspec update
```

**Manual (Windows):**

```cmd
git clone https://github.com/sidgitind/Sparring C:\temp\sparring
mkdir openspec\schemas\sparring
xcopy /E /I C:\temp\sparring\openspec-schema\openspec\schemas\sparring\* openspec\schemas\sparring\
openspec schema validate sparring
rem Open openspec\config.yaml and set:
rem   schema: sparring
rem   sparring_library_path: C:\temp\sparring
openspec update
```

> **Duplicate key warning:** `openspec init` already writes a `schema:` line
> into `config.yaml`. Do not add a second one — YAML will throw a parse error
> and OpenSpec will silently fall back to `spec-driven`. Open `config.yaml`,
> find the existing `schema:` line, and replace it with `schema: sparring`.
> Same for `sparring_library_path` — add it once, on its own line.

---

## Avoiding context confusion with /brainstorm

The Sparring library lives at `sparring_library_path`. Claude Code can see
that path during any session in your project — which means skills like
Superpower's `/brainstorm` may read the Sparring library files and treat
them as describing what you are building. You will get a confused session
about building a "sparring partner app" instead of your actual feature.

The fix ships with Sparring: `CLAUDE.md` at the Sparring repo root tells
Claude Code that the library is a tool, not a project. If you cloned before
this file was added, pull the latest version.

**Use `/opsx:continue` to drive the pipeline — not `/brainstorm`.**
The Sparring pipeline is driven entirely by `/opsx:continue`. It knows
the artifact sequence and loads the right persona at the right step.
`/brainstorm` is not needed and should not be run in a Sparring project
unless you want to explore a design question completely outside the pipeline.

## Usage

Run `/opsx:continue` as normal. The pipeline pauses after each persona:

```
/opsx:continue  -> proposal.md
/opsx:continue  -> specs/
/opsx:continue  -> design.md
/opsx:continue  -> sparring_pm.md            <- PM reviews. Resolve BLOCK items.
/opsx:continue  -> sparring_architect.md     <- Architect reviews. Resolve BLOCK items.
/opsx:continue  -> sparring_ui.md            <- UI Designer reviews. Resolve BLOCK items.
/opsx:continue  -> sparring_qa_functional.md <- QA reviews. Resolve BLOCK items.
/opsx:continue  -> sparring_qa_nfq.md        <- NFQ reviews. PIPELINE PASS or BLOCK.
/opsx:continue  -> sparring_brief.md         <- Synthesis. Tier 1/2/3 output, final PIPELINE PASS.
/opsx:continue  -> tasks.md                  <- only if sparring_brief shows PIPELINE PASS
/opsx:apply     -> implementation
```

---

## Generation mode -- what to expect on your first design.md

If `sparring_library_path` points at a real clone of Sparring, the agent can see the
persona files while it writes `design.md` -- before any persona has actually run.
In testing, this meant the agent read the Architect and QA NFQ non-negotiables and
preemptively wrote in an explicit timeout, a state machine for generation status, and
a debounce strategy for event-triggered regeneration -- none of which had been asked for.

This is not a bug. It is a second mode of value on top of the review gate: the agent
raises its own floor before review even starts. It does not replace the review --
in the same test, the PM and Architect personas still found real gaps the agent's
preemptive design work had not touched (a missing failure condition, a monitoring
mechanism with no system behind it). Expect `design.md` to arrive partially hardened,
and expect the pipeline to still find things.

Full explanation: [`Persona/how_it_works.md`](https://github.com/sidgitind/Sparring/blob/main/Persona/how_it_works.md)

---

## Running specific personas only

Skip any persona by creating its output file manually before `/opsx:continue`:

```bash
# Skip UI Designer for a backend-only feature
echo "SKIPPED: no user-facing states in this feature" \
  > openspec/changes/my-feature/sparring_ui.md
```

OpenSpec sees the file exists and advances. The skip is part of the change record.
`tasks` notes which personas were skipped in its header.

**Suggested skips by feature type:**

| Feature type | Safe to skip |
|---|---|
| Pure backend / API-only | sparring_ui |
| No external service calls | sparring_qa_nfq (or run at reduced strictness) |
| Frontend-only, no new API | sparring_architect |
| Well-understood internal feature | sparring_pm (create manually with brief note) |

---

## Already using superpowers-bridge?

Same install. Sparring inserts before `tasks`. Superpower runs after:

```
proposal -> specs -> design
  -> sparring_pm -> sparring_architect -> sparring_ui
  -> sparring_qa_functional -> sparring_qa_nfq -> sparring_brief
  -> tasks
  -> [Superpower: writing-plans -> TDD -> code review]
  -> retrospective
```

---

## The five personas

```
PM (Amazon)      ->  Is the problem correctly defined? Is success measurable?
Architect        ->  Does this fit the system? Are external contracts defined?
UI Designer      ->  Are all user states specified? What does failure look like?
QA Functional    ->  Is every path testable? Are negative paths covered?
QA NFQ           ->  Will it survive reality? Timeouts, SLOs, failure visibility.
```

Each persona has non-negotiables, a bias section, and structured output format.
Read them before your first run: [/Persona](https://github.com/sidgitind/Sparring/tree/main/Persona)

---

## Proof

A four-AC spec for an AI ticket summary feature ran through this exact schema, live, via
`/opsx:continue` in Claude Code. Fourteen blocking items were raised and resolved across
five persona rounds. The spec exited with nine ACs. `design.md` picked up an idempotency
key on the event bus, a consecutive-failure counter, a unified failure-display rule, and
an explicit state machine -- some of it written by the agent before any persona ran
(see "Generation mode" above), the rest added in response to blocking items.

One persona's fix repeatedly created the next persona's finding -- the PM persona closed
a missing-failure-condition gap by adding a monitoring AC; the Architect persona, reading
that addition one round later, found it had no mechanism behind it. Neither persona alone
would have caught both ends of that gap.

Every persona output, in full, from this run:
[`Example/sparring-test/openspec/changes/ai-ticket-summary/`](https://github.com/sidgitind/Sparring/tree/main/Example/sparring-test/openspec/changes/ai-ticket-summary) --
start with `sparring_brief.md` for the 30-second version.

---

*Built on the principle that the most expensive bugs are written into specs, not code.*
