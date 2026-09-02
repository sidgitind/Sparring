# Sparring -- OpenSpec Schema

Adds adversarial spec review to your OpenSpec workflow. Five personas run
as separate artifacts between `design` and `tasks`. Each one is a named
step in your change folder -- skippable, resumable, part of the record.

```
design -> sparring_pm -> sparring_architect -> sparring_ui
       -> sparring_qa_functional -> sparring_qa_nfq -> tasks
```

Each persona runs one at a time. `/opsx:continue` advances the pipeline.
Human resolves blocking items after each persona before continuing.
`tasks` is blocked until all five artifacts exist (or are explicitly skipped).

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

**Manual:**

```bash
# 1. Clone Sparring
git clone https://github.com/sidgitind/Sparring /tmp/sparring

# 2. Copy schema into your project
cp -r /tmp/sparring/openspec-schema/openspec/schemas/sparring \
      openspec/schemas/sparring

# 3. Validate
openspec schema validate sparring

# 4. Configure
cat >> openspec/config.yaml << 'EOF'
schema: sparring
sparring_library_path: /tmp/sparring
EOF

# 5. Apply
openspec update
```

---

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
/opsx:continue  -> tasks.md                  <- only if all personas passed
/opsx:apply     -> implementation
```

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
  -> sparring_qa_functional -> sparring_qa_nfq
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

A six-AC spec entered the full pipeline. It exited with 17 ACs, four defined
error states with message content, explicit external service contracts, a
testable definition of "prominent", and operational monitoring alerts.

Every addition traced to the persona that caught it:
[Example/Delta_Spec.md](https://github.com/sidgitind/Sparring/blob/main/Example/Delta_Spec.md)

---

*Built on the principle that the most expensive bugs are written into specs, not code.*
