---
name: "project-init"
description: "Use this skill when starting a new project or product that will involve a planning agent (e.g. Claude in Cowork) and a coding agent (e.g. Claude Code) working together, and a lightweight command-center of planning documents is wanted instead of ad-hoc chat. Trigger on phrases like 'start a new project', 'set up a command center for this', 'let's write a PRD/decision log/backlog', even if the user never names the methodology. Skip this for single-file tasks or casual conversation about an existing project."
---

# Precedent — Command-Center Setup for AI-Assisted Projects

This skill sets up a consistent "command center" when starting a new product or project involving both a planning agent and a coding agent. The goal: while code and (optionally) design happen in separate environments, there's a single place where decisions get made and recorded.

Note: this is the planning/documentation side of the workflow. Code gets written elsewhere (e.g. Claude Code); both sides read and write the same `docs/` folder. The real "command center" isn't a tool — it's the documents in the repo (PRD, DECISIONS, TECH_SPEC).

## Philosophy (why this works the way it does)

The planning agent running this skill isn't just planning — it's filling the Project Historian role: preserving decisions, remembering the project, managing its history, reviewing what's gone stale, and sustaining its own methodology, kept distinct from whichever agent writes the code. Everything below is that role broken into concrete steps.

This is a deliberate, systematic approach: **clarify and document first, build second.** There's no technical urgency driving this — the goal is building good habits. Before code gets written, what the project is, which decisions were made and why, and what the real risk is get put into writing. That way, months later, "why did we do it this way?" always has an answer.

Core principles:

- **Favor an MVP, avoid over-engineering.** Recommendations should be concrete and actionable. Aim for the leanest version that actually works.
- **Every significant decision is recorded with its rationale** (ADR discipline). Decision = what + why + consequences.
- **Unverified assumptions aren't presented as fact.** Don't treat an untested data/technical assumption as certain — say explicitly what still needs to be validated.
- **Blockers get resolved first.** The core uncertainty of a project (usually data source feasibility) needs to be clear before it's built around.
- **The command center is the single source of truth.** Downstream environments (code, design) consume briefs; decisions flow back to the center.
- **Decisions don't flow one way only.** The flow from center to code matters as much as the flow of deviations and new decisions coming back — otherwise the documents silently go stale.
- **Context economy.** Only the core documents belong in the agent's context; raw data and logs stay out of it.

## Folder layout

Prepare the new project folder however the user's setup requires (Cowork accesses whatever folder is connected in this session). Try writing documents **directly to `docs/` in the repo** (see the access note below); if the file tools genuinely can't reach the connected folder, produce them in a temporary location and hand them to the user as files, and they'll place them in the repo. If a new folder needs to be created, ask about the user's existing project-creation flow first — don't create folders unilaterally.

Organize projects by whatever category structure fits your workspace (e.g. by product line, by client, by repo type) — define this with the user rather than assuming a fixed set of categories.

Target structure:
```
<category>/<project>/
  AGENTS.md            ← project brief (<200 lines)
  docs/
    PRD_<project>.md
    TECH_SPEC.md
    DECISIONS.md
    DECISIONS_DRAFT.md  ← inbox: raw decision notes from the coding agent (cleared once processed)
    FEASIBILITY.md      ← only if there's an open blocker (e.g. a data source)
    BACKLOG.md
  briefs/
    DESIGN_BRIEF.md    ← if design work is involved
```
Use ASCII in folder/slug names to avoid friction with store or tooling searches.

### Access: if the project folder is connected to Cowork, work directly

If the user has connected the project folder to Cowork (via a folder-connection feature), both file tools (Read/Write/Edit/Glob/Grep) and the sandbox shell access the same synced folder directly. In that setup there's no mount/path mismatch — the command center can edit documents and, if needed, run commands inside the repo too. Assume this by default.

Be careful only in the exceptional case where the repo isn't connected to Cowork as a folder and lives on a path the sandbox shell can't resolve (e.g. an unmounted network/WSL path). In that case file tools, which go through the host layer, will usually still reach it, but the sandboxed shell may not and will report it can't access the path. In that scenario: test access with the file tools (not the shell), and leave running commands/tests inside the repo to the user or the coding agent.

## Step-by-step protocol

### 1. Read and understand existing context
If this is an existing project, run the Session Start Checklist (below) first. If a PRD, prior notes, or custom instructions exist, read those too. Understand what's already there before proposing anything from scratch: what's the project, which decisions are already fixed, what's the open blocker? Flag contradictions before proposing anything.

### 2. Clarify (if missing)
If what the product is, its target audience, scope (in/out of MVP), or the biggest risk is unclear, ask the user. Don't produce documents before scope is fixed.

### 3. Pick a category, decide placement
Place the project according to the category structure agreed with the user.

### 4. Set up the document set
Use the templates below. Fill in the fixed parts (decisions, backlog) with real content; leave the uncertain parts (tech spec, feasibility) as skeletons — they fill in once the blocker is resolved.

### 5. Apply the sequencing rule
`feasibility/blocker → TECH_SPEC → code`. Don't move into architecture or code before the core uncertainty of the project is resolved. Mark this as "Phase 0" in the backlog.

### 6. Record decisions
Write the first fixed decisions (scope, priority, stack, name, etc.) to DECISIONS.md in ADR format with their rationale.

### 7. Update the root map, if one exists
If the workspace maintains a root index/map of projects, add the new project to it. If the user will update it themselves, give them the exact line to add.

## Workflow: command center ↔ downstream environments

The command center **produces briefs**; the coding agent and (optionally) a design environment **consume briefs**:
1. Decide here → update PRD/TECH_SPEC/DECISIONS → produce a clear brief for that feature.
2. The design environment works from DESIGN_BRIEF; the coding environment works from AGENTS.md + TECH_SPEC (and any design-tool link, if present).
3. New decisions surfacing in code or design flow back into DECISIONS.md (see the inbox mechanism below). The center stays the single source of truth.
4. **Don't open executable output as files in the repo.** Coding-agent prompts or design prompts are delivered **as chat output**; the user relays them to the downstream environment. The division of labor is clean: edit documents here, hand off runnable prompts as text.

The biggest efficiency lever: stop re-explaining context from scratch to downstream environments every time. Once a brief is written well, it gets reused.

## Session Start Checklist

Every session where the planning agent picks up work on an existing project runs this before any new planning happens:

1. **Read ADR.** Read `docs/DECISIONS.md` to get current with what's already been decided and why.
2. **Read Draft.** Check `docs/DECISIONS_DRAFT.md`. Triage anything in it — Accepted, Rejected, or Ignored (see the DECISIONS_DRAFT.md template below) — and clear processed entries.
3. **Look for expired reviews.** For each ADR with a `Review trigger`, check it against what's true now (a stated metric, a shipped version). For ADRs with no explicit trigger, check whether `BACKLOG.md` just crossed a phase boundary — that's the default trigger. Flag anything whose assumptions may no longer hold.
4. **Continue.** Proceed with whatever the user actually asked for this session.

This is a standing instruction, not a hard gate — it depends on the planning agent actually running it, the same way any instruction in AGENTS.md does. It's written here once so the project's own AGENTS.md can point to it with a single line instead of restating it.

## Feedback: DECISIONS_DRAFT.md (inbox)

Decisions flowing only from the center to code isn't enough — deviations that come up during implementation need to flow back too, or DECISIONS.md drifts out of sync with reality over time. A low-friction inbox handles this, with the triage and review mechanics living in the Session Start Checklist above:

- **Coding-agent side:** when it deviates from the brief or makes a significant decision during implementation (a library swap, an unplanned architectural choice, an unexpected constraint), it drops a raw one-line note into `docs/DECISIONS_DRAFT.md` without worrying about ADR formatting: `- <date> <what happened, why>`. This rule is fixed into the "Non-negotiable rules" section of AGENTS.md, so it doesn't need repeating in every handoff prompt.
- **Command-center side:** the checklist's "Read Draft" step handles triage (Accepted / Rejected / Ignored) and its "Look for expired reviews" step handles staleness — against an ADR's own `Review trigger` field, or, if none was set, against the project's next BACKLOG.md phase transition as the default.

This expectation is written into the project's own AGENTS.md (see the role-labeled rules under "Working practice"), not just this skill's one-time setup flow — that's what makes it hold on ordinary continuation sessions, not only the session where the project was first created.

## Document templates

### DECISIONS.md
```
# Decision Log — <Project>

This file records WHY significant decisions were made.

Core fields (every entry): Status → Context → Decision → Consequences → Importance
Optional fields (fill in when useful, skip otherwise): Labels · Confidence · Review trigger · Owner

Statuses: Proposed · Accepted · Rejected · Implemented · Validated · Deprecated · Superseded
  (not every decision passes through every status — e.g. a pure scope decision may go
  straight from Accepted to Validated, skipping Implemented, since there's nothing to build)
Importance: Minor · Major · Critical

Rule: new decisions go at the top. Log liberally — once something reaches a planning
conversation or the DECISIONS_DRAFT.md inbox, it's worth a line here, with Importance
signaling its weight rather than a write-time judgment call about whether to log it at all.
Genuine non-decisions (typos, cosmetic tweaks) never reach that point to begin with, so this
doesn't turn into noise.

---

## ADR-001 — <title>
- Status: <status> (<date>)
- Importance: <Minor | Major | Critical>
- Context: <why this decision came up>
- Decision: <what was decided>
- Consequences: (+) <upsides> (−) <downsides / risks>
- Labels: <optional — architecture, performance, security, api, database, ux, cost, ...>
- Confidence: <optional — Low | Medium | High>
- Review trigger: <optional — e.g. "after 1000 users," "after v1 ships." Defaults to the
  project's next backlog phase transition if left blank.>
- Owner: <optional — Planning Agent | Human | Developer>
```

### DECISIONS_DRAFT.md (inbox — unprocessed notes)
```
# Decision Drafts (Unprocessed) — <Project>

This is a raw scratchpad. The coding agent drops a one-line note here
whenever it deviates from the brief or makes a significant decision
during implementation, with no formatting overhead.

The command center triages each note at the start of the next session
(see the Session Start Checklist) into one of three outcomes:
  - Accepted  -> promoted to a full ADR in DECISIONS.md
  - Rejected  -> still recorded in DECISIONS.md, as a minimal entry with
                 Status: Rejected and a one-line reason -- considered, not lost
  - Ignored   -> genuine noise (typos, cosmetic notes), cleared without a trace

Format: - <date> <what happened, why>

---
```

### BACKLOG.md
```
# Backlog / Roadmap — <Project>

Status: [ ] todo · [~] in progress · [x] done · [!] blocked

## Phase 0 — Feasibility (BLOCKER: must resolve before architecture)
- [ ] <test the core uncertainty of the project>

## Phase 1 — MVP
- [ ] <tasks for the leanest working version>

## Phase 2+ — later
- [ ] <deferred features>
```
Note: before moving to a new phase, review the assumptions behind the ADRs accumulated so far (see the feedback section above).

### FEASIBILITY.md (only if there's an open blocker)
```
# Feasibility Report — <Project> (Phase 0)

Status: ⬜ Not tested yet
Goal: <prove the core dependency of the project actually works>

## Test scope and method
- <what gets tested, how>

## Findings
> To be filled in after testing.

## Assessment and decision
- [ ] Result: sufficient / partial / insufficient
- [ ] Fallback needed?
- [ ] Update the relevant ADR in DECISIONS.md
```

### TECH_SPEC.md
```
# Technical Specification — <Project>

Status: Skeleton — not filled in until Phase 0 (feasibility) is done.
Relationship: the PRD says "what," this document says "how." Decisions link back to DECISIONS.md by reference.

## 1. Architecture overview
## 2. Data model
## 3. Critical algorithm(s)
## 4. API surface
## 5. Open technical questions
```

### AGENTS.md (project brief — <200 lines)
```
# <Project> — Project Brief

## Project (one line)
<what, for whom, as plainly as possible>

## Stack
- Backend / Client / Infra / Auth

## Commands
- Setup / Run / Test / Lint   (fill in once the repo exists — the highest-value section)

## Non-negotiable rules
- Clarify + document first, build second. Favor MVP.
- Flag any contradiction with the PRD before proposing a fix.
- Don't present an unverified assumption as fact.
- If you deviate from the brief, drop a raw note in `docs/DECISIONS_DRAFT.md` (don't worry about ADR formatting, the command center processes it later).

## Working practice (command center ↔ coding agent)
- The command center reads/edits docs directly.
- If the project folder is connected to Cowork, the command center can also run commands/tests inside the repo; if not (unmounted-path exception), that's left to the user or the coding agent.
- Executable output (coding-agent or design prompts) isn't opened as files in the repo — it's delivered as chat output.
- If you are the planning/command-center agent: at the start of every session, run the Session Start Checklist (read ADR, read Draft and triage it into Accepted/Rejected/Ignored, check for expired reviews).
- If you are the coding agent: only append raw notes to `docs/DECISIONS_DRAFT.md` when you deviate from this brief. Don't curate, triage, or promote entries yourself — that's the command center's job.

## Reference documents (under docs/)
- PRD · TECH_SPEC · DECISIONS · DECISIONS_DRAFT (inbox) · BACKLOG

## Status / blocker
<current phase, and any open blocker>
```

### DESIGN_BRIEF.md (if design work is involved)
```
# Design Brief — <Project>

## Product summary
## Design principles
## MVP screen list
- [ ] <screens>
## Out of scope
## Open design questions
## Related documents
```

## Context economy
Keep the fixed core that goes into context to: **PRD + TECH_SPEC + DECISIONS**. Raw data dumps, experiment logs, query output, and the unprocessed contents of `DECISIONS_DRAFT.md` can stay as files but shouldn't be pulled into context — bloating context degrades decision quality. Keep AGENTS.md under 200 lines (it's working memory, not documentation).

## Optional: proactive checks in Cowork

If you're running this in Cowork specifically, the Session Start Checklist can be turned into a real scheduled task (e.g. weekly) that reports unprocessed inbox notes or expired review triggers without waiting for a session to start. This isn't part of the portable skill itself — AGENTS.md needs to stay usable by any AGENTS.md-aware tool, not just Cowork — but it's worth setting up separately if reliability here matters to you.

## When done
Present the generated files to the user. The next step is usually resolving the Phase 0 blocker (e.g. testing a data source) — remind them of this and offer to help.
