# I Turned My AI Planning Agent Into the Project's Historian — And Made It Keep Receipts on Itself, Too

*This isn't an AI memory system. It's about making sure a decision, once made, actually reaches the place it's needed next — a problem that will outlast whatever memory architecture is fashionable this year.*

Three weeks into a project, I asked my own AI assistant why we'd picked a particular data model. It didn't know. I didn't know either. We'd talked it through in a chat session that had long since scrolled into the void, made the call, moved on, and never written it down anywhere either of us — human or agent — would think to look again.

That's a small thing to lose. It's also exactly the kind of small thing that, multiplied across a few dozen decisions over a few months, turns a project into a place where nobody can explain why anything is the way it is. Everyone who's worked on a codebase for more than a year has felt this. What's new is that now an AI agent is making some of these calls too, in a chat window that disappears, at 2am, without asking anyone.

So I built a small, deliberately unglamorous fix. It's called [**Precedent**](https://github.com/imertekin/precedent), and the core idea fits in one sentence: an AI agent doesn't get to hand off runnable work until the decision behind it is written down where the next session — human or agent — can find it.

The agent that talks to me, clarifies scope, and writes the brief isn't just planning anymore. It's acting as the project's historian.

## Two kinds of forgetting

Working with AI agents across sessions, you lose context in two distinct ways.

The first is **session loss** — you open a new chat and have to re-explain the project from scratch. This one gets a lot of attention. It's why "memory" features and long-context models get so much airtime.

The second is quieter and, I'd argue, more expensive: **decision loss**. Nobody — not the human, not the agent — remembers *why* something was built a certain way. Not the fact that a decision was made, but the reasoning behind it, the alternative that got rejected, the constraint that made it necessary. This is the stuff that determines whether you can safely change something six months later, and it's exactly the stuff that evaporates first.

There are two broad ways to fight this. One is automatic: run a memory system that embeds and retrieves everything, so the agent can pull up "similar past situations" on demand. The other is manual: keep a small number of hand-curated documents that a human actually wrote and actually trusts. I went with the second, and it's worth being honest about why — it's not because it's better in general, it's because it fits my scale. A handful of projects, decisions made slowly and on purpose, one or two people involved. If you're running dozens of parallel experiments with high-frequency, low-stakes decisions, an automatic system will probably beat this. This is a tool for a specific kind of shop, not a universal answer.

But I want to flag something here, because it changes how you should read the rest of this: I don't think this is actually a memory problem, and framing it that way undersells it. Memory is about retrieval — pulling up what happened. What I kept running into wasn't a retrieval failure, it was a *routing* failure: a decision got made, and then never reliably reached the place or the moment where it was needed next. Models will keep changing. Context windows will keep growing. Whatever we're calling "agent memory" in five years will look nothing like today's embeddings-and-vector-stores approach. None of that touches the actual question — why did we decide this — which will need an answer just as much in ten years as it does now. That's the part worth building a habit around, independent of the tooling underneath it.

## What it actually does

Every project gets a small set of files:

```
<project>/
  AGENTS.md                ← project brief, under 200 lines
  docs/
    PRD.md                  ← what we're building
    TECH_SPEC.md             ← how
    DECISIONS.md             ← why (the decision log)
    DECISIONS_DRAFT.md       ← inbox for unprocessed notes
    BACKLOG.md               ← when, in what order
  briefs/
    DESIGN_BRIEF.md          ← only if design work is involved
```

None of this is novel by itself. `DECISIONS.md` is an Architecture Decision Record, a format Michael Nygard wrote up in 2011 and that's been quietly standard in serious engineering orgs ever since. `AGENTS.md` is an actual open standard now — released by OpenAI in August 2025, quickly adopted by Cursor, Factory, Google's Jules, and most other major coding agents, and donated to the Linux Foundation's Agentic AI Foundation in December 2025. The PRD-then-spec-then-backlog ordering is close to what GitHub's own Spec-Kit recommends for AI-assisted development. I didn't invent any of these pieces. What I built is the specific way they connect.

Each `DECISIONS.md` entry also carries more than the classic four fields. A status that moves through a real lifecycle — proposed, accepted, implemented, validated, or rejected, deprecated, superseded — instead of a flat yes/no. And an importance rating, so a decision to use a UUID instead of an auto-incrementing ID doesn't sit in the log with the same visual weight as a decision that turns the whole backend event-driven. That distinction sounds cosmetic until you're the one scrolling past forty entries trying to figure out which ten actually mattered.

That connection has two parts, and the second one is the part I actually care about.

```
Planning Agent → writes ADR → Coding Agent
      ↑                              │
      └──────── Draft (inbox) ───────┘
```

**Part one is the gate.** The planning agent — the one that talks to me, clarifies scope, writes the brief — will not generate a runnable prompt for the coding agent until the relevant decision already exists in `docs/`. Not "should" write it down. Can't hand off work without it. I want to be precise about what "can't" means here, because it's the honest version of a claim I'd otherwise overstate: the gate is a standing instruction, enforced at the prompt level, so it's exactly as reliable as the agent reading and following its brief — same as everything else in this system. What separates it from a rule someone has to remember is *where it sits*: it's defined as a precondition of producing the handoff, not as a separate step on a checklist. There's no moment where anyone has to remember to check; a handoff without the record behind it is, by definition, not a finished handoff.

**Part two is the part everyone gets wrong, including me the first time.** A one-way gate isn't enough. Code doesn't stay obedient to the plan — it can't, because the plan is always incomplete. A library doesn't do what you expected, a constraint shows up that nobody saw coming, and the agent implementing the feature makes a small, sensible call that never travels back to the document that's supposed to be the source of truth. Six months later that document is wrong, and worse, nobody knows it's wrong.

So there's a second file, `DECISIONS_DRAFT.md`, that exists for exactly one purpose: catching those small, in-the-moment calls before they vanish. Not everything that lands there survives — triage has three outcomes, not two. A note either gets promoted to a full decision record, or gets declined with a one-line reason that stays in the log, or — if it's genuine noise — gets cleared without a trace. The middle outcome is the one that matters: a *declined* decision still leaves a record, but the log doesn't turn into a landfill of every passing remark either. Here's what that looks like in practice.

## A walk-through

**Session 1.** I ask the planning agent to set up a new project — a small internal tool for tracking something. We go back and forth on scope, it writes a PRD, and we log the first real decision: we're using a particular external API as the data source, even though it's rate-limited, because the volume is small enough that it won't matter. That goes into `DECISIONS.md` as ADR-001, with the reasoning attached.

**Later that week.** The coding agent is implementing the integration and discovers the API's rate limit is enforced per-IP, not per-key — something nobody checked. It works around this with a small caching layer that wasn't in the spec. Instead of silently doing this and moving on, it drops one line into `DECISIONS_DRAFT.md`: *"2026-07-14 — API rate-limits by IP not key, added a 10-minute cache layer to stay under it."* No formatting, no ceremony. Just a note.

**Session 2**, a few days later, back in the planning agent. The project's brief — the file that gets loaded at the start of any session touching this project — carries a standing rule for whichever agent is reading it as the planning side: check the inbox first. It finds that note, decides it's actually significant (it constrains future scaling, not just a cosmetic detail), and promotes it to ADR-002 in proper form. The inbox gets cleared. Worth being honest about the mechanism here: this isn't magic, it's one line of standing instruction in a file that reloads every session — which is exactly as reliable as the agent actually reading and following it.

**A month later**, the project moves from its MVP phase into the next one. That transition is a built-in checkpoint: before moving forward, we do a quick pass over the ADRs made so far and ask whether the assumptions still hold. ADR-002 gets marked Validated — the cache layer held, the assumption behind it is confirmed. ADR-001's "volume is small enough" gets flagged instead — it's grown 4x, worth another look — its status stays open while everything else moves forward.

Nothing about this required me to remember to ask, or the agent to remember to mention it unprompted at some arbitrary later date. The structure did the work.

![The Precedent loop](./assets/flow-diagram.svg)

## The role, not the artifact

Everything above is a reasonably direct application of things the industry already knows — decision logs aren't new. What took me longer to see is that the artifact was never really the point. The planning agent running all of this isn't just planning. It's acting as the project's historian.

A historian's job isn't just "write things down." It's preserving decisions well enough that their reasoning survives, not just the fact that something was decided (the ADR format, with a real status lifecycle instead of a flat yes/no). It's remembering the project itself, cheaply, without re-explaining it every session (the brief that reloads each time). It's managing a messy, ongoing history rather than a clean final one (the inbox, and the fact that a declined note leaves a trace instead of disappearing). It's reviewing — actually going back and checking whether an old decision still holds, on a schedule tied to the decision itself rather than a calendar nobody follows.

And then there's the part that made me want to write this up at all, the part that's smaller and, I think, more interesting than everything above: **the skill that runs this whole thing keeps a decision log about its own design.**

When I added the inbox mechanism, I didn't just update the instructions — I wrote an ADR explaining why, what the alternative was, and what trade-off I was accepting. When I later caught a naming mistake (I'd called the project brief `AGENT.md`, not the actual standard `AGENTS.md`), that correction got its own entry too. The tool that teaches "write down why you decided things" applies that rule to itself, recursively, with the same discipline it asks of everything else.

This sounds like a small flex, but it solves a real problem: methodologies drift, and usually nobody can say when or why. A year from now, if a rule in this system looks strange, the answer isn't buried in a Slack thread or a half-remembered conversation. It's in a file, in the same repo, in the same format as every other decision.

## What I'd tell you to be skeptical of

I'd rather undersell this than oversell it, so here's what I don't have confidence in yet.

One thing before the specific risks: this system intentionally optimizes for explainability over speed. If your context rewards moving fast with less friction more than it rewards being able to explain a choice six months later, some of what follows will read as overhead, not safety.

**Decision logs get abandoned. This is extremely well documented, not a hypothetical.** Teams adopt ADRs enthusiastically and quietly stop maintaining them within months, consistently enough that it's a known pattern in the literature, not an edge case. The reasons are things like: the maintenance burden lands on whoever has the least context, or trivial decisions pile up next to load-bearing ones until nobody trusts the log anymore. The one-way gate targets the record living outside the actual workflow; the importance rating on every entry targets trivial decisions drowning out load-bearing ones. The maintenance burden landing on whoever has the least context — that one I still don't have a good answer for.

**The inbox can be poisoned.** If a one-off, unimportant deviation gets promoted into a permanent decision, the agent learns the wrong lesson and keeps re-learning it. Curation helps — a human or the planning agent deciding what's actually worth keeping — but it's a mitigation, not a guarantee.

**I haven't tested this past small scale.** This is built for a handful of projects with infrequent, deliberate decisions. I genuinely don't know where it breaks if you throw a large team or a high-velocity, many-small-decisions environment at it. My guess is it breaks somewhere, and an automatic memory system takes over from there — but that's a guess, not something I've verified.

## Take the piece that's missing, skip the rest

One more thing worth saying plainly: none of these pieces require each other. If all you take from this is the one-way gate — don't let an agent hand off runnable work until the decision behind it exists somewhere durable — that's useful entirely on its own, in whatever stack you're already running. Same with the inbox, or tying revalidation to your own natural checkpoints instead of a calendar. Or just the idea itself — that one agent, not both, owns the project's history — is worth carrying into whatever setup you already have, even without any of the specific file formats here. This isn't a methodology to adopt wholesale, with a name and a manifesto. It's closer to a small set of habits. Take the one that's missing from what you've already got, and leave the rest.

## Why bother writing this down

None of this makes AI agents smarter. It makes the *record* better, which is a duller but more durable kind of improvement. The agent isn't going to reason better about your architecture because you gave it a decision log. But six months from now, when someone — human or agent — asks why the system does something strange, there will be an answer that isn't "nobody remembers."

That's a low bar. It's also one that most projects, AI-assisted or not, quietly fail to clear.

---

*Precedent is open source — the write-up, the installable skill (it's called `project-init`), and its own self-referential decision log are on [GitHub](https://github.com/imertekin/precedent).*
