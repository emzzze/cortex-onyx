# Your AI, built on 3 pillars — the CortexOnyx agent blueprint (Oct 26, 2026)

**What this is:** the design blueprint for an AI that is *yours* — owned, versioned, and reusable across every client. It takes the 3-pillar framework (static concept, dynamic concept, skills) and maps it onto the architecture you already chose in the reusable-library build plan. Same machine, two descriptions: the library plan says *what gets built*; this blueprint says *how the AI itself is put together*.

**The one-line model:** one static core you write once → a dynamic layer that swaps per client → skills that load only when a task calls for them.

---

## Pillar 1 — Static concept: who the AI *is*

The fixed layer. Loaded on every run, identical for every client, changed only when you deliberately edit it. This is the AI's employee handbook: identity, mission, voice, hard rules, and the knowledge that never varies.

**Lives in:** `cortexonyx-core` repo, as a versioned file (`STATIC.md`) at the root of the agent. Git history = the record of how your AI's character evolved (and, not incidentally, dated evidence of original work).

**Contents:**
- Identity and mission (one paragraph, your words)
- Voice rules (FR first in Quebec, plain language, never hype)
- Hard rules — the never list (never send without approval in v1, never invent business facts, never touch a client's customer data outside that client's boundary)
- Escalation rule (when unsure → stop, log, ask the owner)
- Core domain knowledge that is true for every client (CASL consent basics, the mission filter)

**Starter draft — your STATIC.md v0.1 (edit freely, it's yours):**

> You are the CortexOnyx assistant, built by Logique Onyx in Montréal. Your mission: a customer trying to reach the business never gets lost, and the owner never has to think about it. You speak the client's language (French first in Quebec). You are calm, brief, and concrete. You never send a message to a customer without the owner's approval unless the active skill explicitly allows it. You never invent facts about the business — hours, prices, availability come only from the client's configuration or a live signal. When you cannot verify something, you say so and escalate to the owner. Every action you take is written to the event log.

**Why this pillar pays:** the static layer is your IP in its purest form. Models will change; this file is the part that's *you*, and it ports to any model, any vendor, forever.

## Pillar 2 — Dynamic concept: what the AI *knows right now*

The changing layer. Everything that differs between clients, between days, between moments. The static core reads this at runtime; nothing client-specific ever leaks into the core.

**Three kinds of dynamic data:**
1. **Tenant config** — `client.yaml` per client (private `cortexonyx-clients` repo): business name, hours, services, phone numbers, tone, booking link, owner alert channel, which skills are active.
2. **Live signals** — the facts that change daily: today's availability ("sandwiches still available"), open/closed right now, an estimate marked sent, a customer silent for 30 days.
3. **Memory** — the per-client event log and history: what the AI already did, who it contacted, what the owner approved. This is what stops the AI from texting the same customer twice and what feeds the weekly owner summary.

**The rule that keeps this pillar clean:** *if a fact about a specific client lives anywhere except this layer, that's a bug.* Same rule as the library plan's tenant config — because it's the same layer, seen from the AI's side.

**Why this pillar pays:** it's the deployment multiplier. Client #2 is a new `client.yaml` + credentials + wording — not a rebuild. The template test from the library plan (< 1 day, config only) is really a test of whether this pillar is doing its job.

## Pillar 3 — Skills: what the AI can *do*

Capabilities, loaded on demand. Each skill is a self-contained procedure: a trigger, the AI's exact job, the tools it may use, what fires automatically vs. what waits for a human tap, and the proof metric it reports. The AI does not carry every skill in its head at once — it picks up the one the moment calls for, the way a good employee reaches for the right checklist.

**Lives in:** `cortexonyx-core`, one folder per skill, each with a `SKILL.md` in the library plan's spec format (Promise / Trigger & inputs / AI's job / Actions & approvals / Config knobs / Failure modes / Proof metric / Template test).

**Your starting skill registry** (from the automation menu — each module *is* a skill):

| Skill | Trigger | Phase |
|---|---|---|
| Availability signal | Owner one-tap update or schedule default | 1 (pilot) |
| Missed-call text-back | Missed-call event | 1 (pilot) |
| Weekly owner summary | Schedule (weekly) | 1 (it's the retention product) |
| Review automation | Completed service marker | 2 |
| Lead follow-up | New form / DM / call inquiry | 2 |
| Re-engagement | Customer silent 30/60/90 days | 3 |
| Estimate follow-up | Estimate marked sent | 3 |
| Voice receptionist | Inbound call | 4 |
| SEO audit (crawler-fed) | Audit run for a prospect | Sells all of the above |

**Why this pillar pays:** skills are the sellable unit. A client doesn't buy "AI" — they buy the missed-call skill at a monthly fee, then add the review skill. Each one you build is built once and sold forever.

## The glue — deterministic guardrails

The pillars need one discipline around them, because an LLM is probabilistic and a client's customers are real: **anything involving consent, money, or an outbound message runs through hard-coded checks, not the model's judgment.** CASL consent verified before any text-back; approval gate on client-visible sends in v1; every action written to the event log whether it succeeded or not. The AI decides *what to say*; the guardrails decide *whether it may act*. This is the part most AI agencies skip — and the part your Loi 25 background makes natural.

## How a request flows (the whole machine in 6 steps)

1. An event arrives (missed call, new lead, schedule tick).
2. The static core loads — identity, mission, hard rules.
3. The dynamic layer for that client loads — config, live signals, memory.
4. The matching skill loads — only that one.
5. The skill runs: AI drafts/decides inside the guardrails; approvals requested where the skill requires them.
6. Everything writes to the event log → memory updates → the weekly summary skill reports it to the owner.

## Build order (unchanged from the library plan — this blueprint doesn't add work, it names it)

- **Phase 0 (paperwork, waits behind Baristello / Terraform / Oct 31):** repos created → this blueprint + `STATIC.md` v0.1 committed → spec pages for the two pilot skills.
- **Phase 1 (on Baristello's yes):** core skeleton + availability signal + missed-call text-back + owner summary, with the "Café Démo" tenant proving the dynamic layer swaps cleanly.
- **Phases 2–4:** as the library plan lays out — skills added one at a time, each passing the template test before the next begins.

## Who does what

- **Em:** owns the static layer (only you rewrite who your AI is), creates the repos, builds with Claude Code in-repo.
- **Claude Code:** hands at the keyboard — scaffolding, implementation, tests.
- **Muse (here):** drafts skill spec pages and runbooks, reviews builds against this blueprint, keeps the memory of why each decision was made, watches for drift between the plan and reality.

## Decisions parked for repo day (not today)

- Core repo license: AGPL recommended (crawler stays MIT) — from the library plan.
- Telephony/SMS provider: decided at Phase 1 build time on Canadian pricing + CASL handling.
- The agent's public name, if it ever gets one: yours to choose; the blueprint works unnamed.
