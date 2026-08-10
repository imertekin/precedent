# Decision Log — Precedent (the skill itself)

This file tracks the evolution decisions of the Precedent skill itself — the discipline this
skill teaches to projects, applied reflexively to the skill's own history. Format: Context →
Decision → Consequences → Status.

Note: this log started on 2026-07-31, generalized from the original Turkish, company-specific
version ("proje-baslat") that predates it. That earlier version's own decisions weren't recorded
retroactively — this log picks up from the generalization point forward.

---

## ADR-S10 — Design requests need the same "ask before deciding" discipline as code
- Status: Accepted (2026-08-04)
- Importance: Major
- Context: A live dry run (Sonnet, not Opus, running project-init on a real project) surfaced an
  asymmetry: the coding handoff has a well-specified protocol (gate, inbox, "deliver prompts as
  chat output, not files"), but the design handoff was just a DESIGN_BRIEF.md template with no
  equivalent rule. Asked to improve the UI with "various design model approaches" in mind, the
  planning agent skipped straight to picking colors and a design system and pushed toward
  implementation, instead of asking what the user actually wanted and producing a brief. This
  looks like a real skill gap, not just a weaker model being less cautious — nothing in the skill
  told any model to hold back here the way it's told to on the code side.
- Decision: Added a "Design requests: ask before deciding" section. It splits design asks into
  two kinds the command center must distinguish rather than assume: a throwaway mockup/visual
  check (no prep, fine to sketch directly in chat) versus an actual design decision (follows the
  same clarify-then-brief-then-handoff discipline as code, via DESIGN_BRIEF.md). If it's unclear
  which the user wants, ask.
- Consequences: (+) Closes the asymmetry between the coding and design handoffs — design
  decisions now go through the same one-way gate as code decisions. (+) Quick mockups stay cheap
  and don't require ceremony. (−) Adds one more judgment call the command center has to make
  correctly (throwaway vs. real decision) rather than a hard rule; a model could still misjudge
  which bucket a request falls into.

---

## ADR-S9 — Claude Code doesn't read AGENTS.md natively; ask, don't auto-create
- Status: Accepted (2026-08-04)
- Importance: Major
- Context: A live dry run (project-init run on a real dummy project) raised the question of
  whether AGENTS.md is actually consumed by a coding agent the way this skill assumes. Research
  confirmed Codex, Cursor, Windsurf, and Cline all read a repo's AGENTS.md natively, but Claude
  Code does not — as of 2026 this is the single largest unresolved feature request on its issue
  tracker (#6235, 5,200+ reactions, no roadmap commitment). Left unaddressed, this silently
  breaks the skill's core premise ("planning agent and coding agent share AGENTS.md") for anyone
  whose coding agent happens to be Claude Code. The officially documented fix is trivial: a
  one-line `@AGENTS.md` import at the top of CLAUDE.md, which Claude Code loads at session start.
- Decision: Rather than generating a CLAUDE.md bridge file by default for every project (dead
  weight for the majority of coding agents that need no bridge at all), the skill now asks which
  coding agent will consume AGENTS.md during document setup, and creates the one-line CLAUDE.md
  import only when the answer is Claude Code.
- Consequences: (+) No unnecessary files for Codex/Cursor/Windsurf/Cline users. (+) Closes a real
  correctness gap for Claude Code users instead of silently assuming AGENTS.md gets read. (−) One
  more clarifying question added to setup. (−) If the user switches coding agents mid-project,
  the bridge file doesn't automatically appear or disappear — that's a manual follow-up.

---

## ADR-S8 — Richer Decision Lifecycle, Draft triage, and a consolidated Session Start Checklist
- Status: Accepted (2026-07-31)
- Importance: Major
- Context: A structured review (external feedback plus internal audit) surfaced several real
  gaps in the original 4-field ADR format and the inbox mechanism: no way to signal how much a
  decision matters (trivial and load-bearing entries read identically, which is exactly the
  documented cause of ADR-trust erosion); no way to record that a draft note was reviewed and
  declined rather than silently deleted; no way for a decision to specify its own re-check
  condition instead of relying solely on the generic phase-transition review; and the
  session-start behavior (check ADRs, check the inbox, check for staleness) was scattered
  across three separate places in this file, restating overlapping instructions.
- Decision: Expanded the ADR format to five core fields (Status, Context, Decision,
  Consequences, Importance) plus four optional fields (Labels, Confidence, Review trigger,
  Owner) filled in only when useful. Status became a 7-value lifecycle (Proposed, Accepted,
  Rejected, Implemented, Validated, Deprecated, Superseded); decisions don't need to pass
  through every value. Loosened the write-time "skip trivial decisions" rule — anything that
  reaches a planning conversation or the inbox gets logged, with Importance (Minor/Major/
  Critical) carrying the filtering weight instead of an irreversible write-time judgment call.
  Gave DECISIONS_DRAFT.md notes an explicit triage outcome — Accepted, Rejected, or Ignored —
  instead of uniformly clearing everything; Rejected notes get a minimal permanent record
  instead of disappearing. Consolidated the three scattered session-start behaviors into one
  Session Start Checklist section that AGENTS.md's role-labeled rule now points to instead of
  restating.
- Consequences:
  - (+) Load-bearing and trivial decisions are now visually distinguishable without deleting
    anything, directly targeting a documented cause of ADR abandonment.
  - (+) A declined draft leaves a trace instead of vanishing — closes a real information-loss
    gap the "clear everything" behavior had.
  - (+) Review triggers are now decision-specific (a metric, a version, a phase) rather than
    forcing every decision onto the same generic phase-transition schedule.
  - (+) Session-start behavior lives in one place instead of three, reducing the chance future
    edits create contradictions between the copies.
  - (−) The ADR format is heavier than the original 4-field version — mitigated by keeping only
    5 fields mandatory and 4 genuinely optional, but this is a real increase in what a
    conscientious user is expected to fill in.
  - (−) None of this is self-enforcing beyond what AGENTS.md already wasn't — it still depends
    on the planning agent actually running the checklist, same limitation as ADR-S7.

## ADR-S7 — Role-labeled the inbox-processing rule inside AGENTS.md
- Status: Accepted (2026-07-31)
- Context: The instruction "check DECISIONS_DRAFT.md at the start of a session" only lived in
  SKILL.md's one-time setup flow, which is scoped to *starting a new project* — not to ordinary
  continuation sessions on an existing one. Since project-init's own trigger explicitly skips
  "casual conversation about an existing project," nothing guaranteed the inbox actually got
  checked on later sessions. Moving the rule into AGENTS.md (which reloads every session) fixes
  that reach problem, but AGENTS.md is a single file both the planning agent and the coding agent
  read — a blanket rule there would be ambiguous about which agent is supposed to act on it, and
  risked the coding agent trying to curate ADRs itself, which breaks the "curation stays with the
  command center" principle.
- Decision: Added two explicitly role-labeled bullets to AGENTS.md's "Working practice" section —
  one starting "If you are the planning/command-center agent," one starting "If you are the coding
  agent" — instead of splitting the brief into two separate files.
- Consequences:
  - (+) The inbox-check now persists across ordinary sessions, not just initial setup.
  - (+) A single shared brief stays the single source of truth; no second file to keep in sync.
  - (+) Each agent reading the same file has an unambiguous instruction for its own role.
  - (−) Still depends on the agent actually reading and following AGENTS.md — this is a standing
    instruction, not a hard architectural gate like the docs-before-code rule.

## ADR-S6 — Corrected AGENT.md to AGENTS.md
- Status: Accepted (2026-07-31)
- Context: The project-brief template file was named "AGENT.md" (singular), presented as the
  generic equivalent of CLAUDE.md. The actual open standard this maps to is "AGENTS.md" (plural),
  and the spec explicitly supports nested/per-directory AGENTS.md files, not just one at a repo
  root. Naming it "AGENT.md" was an unforced error — it looks like the standard but isn't, so
  no AGENTS.md-aware tool (Codex, Cursor, etc.) would actually pick it up.
- Decision: Renamed the template file to `AGENTS.md` throughout the skill and both READMEs, to
  match the real standard exactly.
- Consequences:
  - (+) The project brief is now genuinely interoperable with any AGENTS.md-aware tool, not just
    Claude-based ones.
  - (+) No invented near-standard sitting next to the real one to confuse readers.
  - (−) None identified — this was a straightforward correctness fix.

## ADR-S5 — Split the skill's invocation name from the methodology's brand name
- Status: Accepted (2026-07-31)
- Context: "Precedent" works well as the methodology's/repo's public brand — evocative, carries the
  legal-precedent metaphor. But as a skill's own invocation name (a kebab-case identifier used as
  a storage key), an abstract brand word is less practical than a name that signals what triggers
  it.
- Decision: Kept "Precedent" as the repo/methodology name. Renamed the skill's internal `name`
  field to `project-init` — mirroring a familiar convention (`git init`, `npm init`) and stating
  directly what the skill does.
- Consequences:
  - (+) The skill name now reads as an action, consistent with how skill names are used elsewhere.
  - (+) The methodology keeps a distinct, memorable public identity.
  - (−) Two names now need to be kept straight when referring to "the thing" (repo: Precedent,
    skill: project-init) — worth spelling out clearly in the README.

## ADR-S4 — Generalized and renamed to "Precedent," split into two skills
- Status: Accepted (2026-07-31)
- Context: The original skill ("proje-baslat") was written in Turkish, hardcoded to a specific
  workspace's categories (mobile/saas/ai-experiments) and language requirement. Turning it into
  a public repo under its original name/content would have been misleading — it would present a
  company-specific tool as a general-purpose methodology.
- Decision: Kept the original Turkish, company-specific skill untouched for its actual daily use.
  Created a separate, generalized, English skill (this one) for the public repo, with the
  categories and language requirement replaced by user-defined configuration, and renamed the
  methodology "Precedent" (a legal metaphor: past decisions bind future ones without
  re-litigating).
- Consequences:
  - (+) The working skill keeps functioning exactly as before, untouched.
  - (+) The public version is honestly generic — no leftover references to a specific company or
    fixed category set.
  - (−) Two skills now need to be kept loosely in sync if the underlying methodology changes;
    there's no automatic propagation between them.

## ADR-S3 — Phase-transition-based ADR review rule
- Status: Accepted (2026-07-31)
- Context: Decisions in DECISIONS.md carry a real risk of going stale over time (see the
  literature: ADRs are commonly abandoned or never revisited). A calendar-based review (e.g. a
  weekly reminder) is disconnected from a project's actual rhythm and adds extra overhead.
- Decision: Revalidation was tied to phase transitions in BACKLOG.md (Phase 0 → 1 → 2) instead of
  a calendar.
- Consequences:
  - (+) Requires no extra infrastructure — it fits into the backlog structure that already exists.
  - (+) Review gets triggered at genuinely meaningful moments in the project.
  - (−) If phase transitions are infrequent (e.g. Phase 1 runs very long), no review gets
    triggered for a long stretch — a time-based backstop may still be needed eventually.

## ADR-S2 — Updated the WSL/UNC access note
- Status: Accepted (2026-07-31)
- Context: The earlier version of the skill presented the possibility of the repo living on an
  unmounted WSL/UNC path as the primary scenario. In actual use, once the project folder is
  connected to Cowork via its folder-connection feature, both file tools and the sandbox shell
  access the same synced folder directly — the WSL problem doesn't occur in that setup.
- Decision: The access note was rewritten. "Works directly if connected to Cowork" is now the
  primary scenario; the WSL/UNC warning was demoted to an exceptional case.
- Consequences:
  - (+) The skill now reflects actual usage accurately, removing unnecessary caution.
  - (−) In a genuinely different setup (repo not connected), the old warning still applies — that
    scenario was kept in the file rather than removed.

## ADR-S1 — Added the DECISIONS_DRAFT.md inbox mechanism
- Status: Accepted (2026-07-31)
- Context: DECISIONS.md only captured decisions flowing from the center to code. Deviations the
  coding agent made during implementation never flowed back to the center — a structural gap that
  let the record drift out of sync with reality over time (staleness).
- Decision: Added a low-friction inbox, `docs/DECISIONS_DRAFT.md`. The coding agent drops
  deviations from the brief as raw notes here; the command center reads it at the start of every
  session, promotes real entries into ADRs, and clears it. The rule was fixed into the
  "Non-negotiable rules" section of the project brief template.
- Consequences:
  - (+) The feedback loop is now structural, not dependent on discipline.
  - (+) The coding agent doesn't need to think about proper ADR formatting in the moment — low
    friction.
  - (−) The command center still needs to process the inbox regularly; this isn't fully
    automatic, it's semi-automatic.
  - (−) There's a real risk of a one-off, unimportant note getting promoted into a permanent
    decision ("poisoning" risk) — curation reduces this but doesn't guarantee against it.
