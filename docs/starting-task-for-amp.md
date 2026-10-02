we don't try to fix this repo or find any issues, it's public professional repo, study it first. We want to modify it as needed, so that we can use it in our Amp code (start new amp cloud threads) and with ChatGPT subscription (and BYOK also for other api keys). Then create a prompt for new Amp thread to start building it, it needs to provide a plan first for us to review before implementing. Goal is that I can monitor my Amp agents in it, start new threads, select model for new thread, etc.
https://github.com/devmik3/cubefarm

---

Yes — and after reading the repo and the current Amp docs, I think CubeFarm is a **very good base for exactly this**.

The key architectural point is that current CubeFarm already separates coding-agent launch behavior in `server/clis.ts`: Claude Code, Codex, and OpenCode are represented as agent CLIs, while the rest of CubeFarm handles floors, agents, work allocation, UI, GitHub, QA, etc. For Amp, though, I would **not simply add another local PTY CLI definition and stop there**, because your main goal is Amp **Cloud threads/orbs**.

Amp officially supports programmatically starting an orb thread with `amp -ox`, choosing its project, mode, orb size, title, etc.; it immediately returns the new `T-…` thread URL while the agent continues in the cloud. It also supports streaming JSON when we want richer live monitoring, continuing existing threads, listing/searching threads, and running threads on local/runners as alternatives. [AmpCode](https://ampcode.com/docs/cli/spawning-orbs?utm_source=chatgpt.com)

Even better, Amp itself now supports exactly the credential model you want: **ChatGPT subscription + BYOK provider keys + Amp credits/model routing**. The current Amp docs explicitly say ChatGPT subscriptions can provide OpenAI model usage, while separate connections can use OpenAI, Anthropic, Google, xAI, routers, custom endpoints, etc. [AmpCode](https://ampcode.com/docs/customize/model-routing?utm_source=chatgpt.com)

So I would tell the new Amp agent to design CubeFarm around **execution/provider backends**, rather than hard-coding a fourth local CLI:

**CubeFarm → Amp Cloud / Amp Runner / local coding CLI → selected agent mode/model routing**

That leaves the existing Claude/Codex/OpenCode support intact while making Amp Cloud a first-class backend.

Here is the prompt I would use for the new Amp thread.

We are going to adapt this repository for our own use.

Repository:
https://github.com/devmik3/cubefarm

IMPORTANT: This is a professional public open-source project. Do NOT approach this task as an audit, bug hunt, cleanup exercise, or attempt to "fix" the original author's work.

First study and understand the existing architecture and design decisions. Preserve the project wherever possible.

Our goal is to EXTEND CubeFarm so it can become a visual control center for our coding agents, especially Amp agents.

Do not implement anything yet.

Your first task is RESEARCH + ARCHITECTURE + PLAN.

Stop after presenting the plan and wait for my approval before changing code.

# Main goal

I want to be able to use the CubeFarm UI to monitor and control my coding-agent work.

The most important new integration is Amp.

I want CubeFarm to eventually let me:

- monitor my Amp agents/threads
- start a new Amp thread
- start Amp Cloud/orb threads
- associate a new thread with the correct GitHub project/repository
- enter the task/prompt
- choose appropriate Amp execution options
- choose the model / agent mode where Amp officially supports this
- see which agent/thread is working
- see thread status/progress
- open the real Amp thread
- continue/send instructions to an existing thread
- see completed/failed/waiting work
- ideally preserve CubeFarm's visual office metaphor for all of this

The existing local Claude Code, Codex and OpenCode functionality should not be unnecessarily removed.

Amp should become an additional first-class execution backend.

# Authentication and model usage

We specifically want to support:

1. Amp using my ChatGPT subscription
2. Amp BYOK/model-provider connections for other providers/API keys
3. Existing Amp model routing
4. Existing Amp account authentication
5. Existing Codex support where useful

Do NOT build our own model proxy, store provider API keys unnecessarily, or duplicate authentication/model-routing functionality Amp already provides.

Prefer Amp's official authentication, subscription and model-routing mechanisms.

Likewise, do not put API keys in CubeFarm state, browser storage, logs, URLs, GitHub, or source code.

CubeFarm should normally tell/use the installed Amp CLI and allow Amp itself to manage credentials and model routing.

# Important Amp capabilities to investigate

Read the CURRENT official Amp documentation before designing this.

At minimum investigate:

- Amp CLI
- execute mode
- `amp -ox`
- Amp orbs/cloud threads
- `--executor`
- `--project`
- `--orb-size`
- `--mode`
- `--fast`
- `--title`
- `--stream-json`
- `--no-archive-after-execute`
- `amp threads ...`
- continuing an existing thread
- listing/searching threads
- exporting/reading thread information
- Amp runners
- Amp thread URLs / IDs
- Amp model routing
- ChatGPT subscription connection
- BYOK/model-provider connections
- Amp access-token requirements for non-interactive use

Do not assume an option or API exists.

Verify everything against current official Amp documentation and/or the installed Amp CLI.

If Amp does NOT expose direct arbitrary per-thread model selection, do not invent it.

Determine the correct supported abstraction.

For example, it may be:

- Amp mode selection
- configured/tuned Amp modes
- model-routing configuration
- provider connections

rather than a raw `--model` flag.

Explain this clearly in the plan.

# First: understand CubeFarm properly

Study this repository before proposing changes.

Pay particular attention to:

- `server/clis.ts`
- `server/cliRunner.ts`
- `server/agentRunner.ts`
- `server/swarm.ts`
- `server/terminal.ts`
- `server/ptyHost.ts`
- `server/ptyClient.ts`
- `server/github.ts`
- `server/workspace.ts`
- `shared/types.ts`
- manager/settings UI
- agent/team UI
- websocket communication
- persistence/state
- issue → agent → PR → QA workflow
- current Claude Code integration
- current Codex integration
- current OpenCode integration
- model/effort/CLI selection
- terminal monitoring
- existing test architecture
- demo mode

Read:

- README.md
- CONTRIBUTING.md
- CLAUDE.md
- docs/how-it-works.md

Do not merely summarize documentation. Trace the relevant implementation.

# Architectural question to solve

The current CubeFarm model is primarily based around coding agents running as local processes/PTYs.

Amp Cloud is different.

An Amp orb thread:

- can execute remotely
- has its own Amp thread ID/URL
- may continue while CubeFarm or my computer is not running
- should not be treated as if it were simply a local terminal process

Design the integration accordingly.

Consider whether CubeFarm should introduce a clean abstraction such as:

- Agent provider
- Execution backend
- Runtime adapter
- Thread backend

or another design that naturally supports both:

LOCAL
- Claude Code
- Codex
- OpenCode
- possibly local Amp

REMOTE
- Amp Orb / Cloud
- Amp Runner where appropriate

Do not force this naming if the existing architecture suggests something better.

The goal is a minimal, maintainable extension of CubeFarm rather than a rewrite.

# Desired Amp workflow

A possible user flow is:

1. I enter a CubeFarm floor/project.
2. I choose "New agent/thread".
3. CubeFarm knows which GitHub repository/Amp project this floor corresponds to.
4. I enter the task.
5. I choose Amp.
6. I choose where it runs:
   - Amp Cloud / Orb
   - Amp Runner, if configured
   - local Amp, if useful
7. I choose supported Amp options such as mode / model configuration / orb size.
8. CubeFarm starts the Amp thread.
9. CubeFarm captures the returned Amp thread ID and URL.
10. A character/desk represents that Amp thread in the office.
11. I can see that it is active and what task it is working on.
12. I can click it for more details.
13. I can open the actual Amp thread when I want full native Amp interaction.
14. I can send a follow-up from CubeFarm if Amp officially supports doing so safely.
15. When the thread finishes, CubeFarm reflects that state.

Investigate how much of this is reliably possible using official interfaces.

Do not fake monitoring data that Amp does not expose.

# Monitoring design

Determine what CubeFarm can reasonably monitor.

Consider:

- thread ID
- thread URL
- title
- project
- execution backend
- started time
- current/last-known status
- working / idle / completed / errored
- final response
- Amp mode
- orb size
- changes/commit/PR information
- elapsed time
- thread activity
- continuation/follow-up

Determine whether this should come from:

- Amp CLI structured output
- `--stream-json`
- Amp thread commands
- official Amp APIs
- GitHub state
- some combination

Prefer structured supported output over scraping terminal text or HTML.

For a cloud thread that outlives the CubeFarm process, think about how CubeFarm restores/reconciles its state after restart.

# Native Amp vs CubeFarm responsibility

Keep responsibilities clean.

Amp should remain responsible for:

- Amp authentication
- ChatGPT subscription authentication
- BYOK credentials
- model-provider routing
- Amp cloud/orbs
- Amp thread storage
- Amp execution

CubeFarm should be responsible for:

- visual management
- choosing/configuring a task
- launching supported Amp commands
- associating threads with floors/repos/agents
- displaying status
- storing safe metadata such as thread IDs/URLs
- orchestration that genuinely adds value

Do not reproduce the Amp web application inside CubeFarm.

A button that opens the actual Amp thread is perfectly acceptable and probably desirable.

# ChatGPT subscription + BYOK

Research the cleanest setup experience.

Ideally CubeFarm should be able to detect things such as:

- Amp installed?
- Amp version
- Amp authenticated?
- configured Amp project
- available execution types
- configured model-provider routing, where safely queryable
- whether additional setup is required

But CubeFarm should NOT expose provider secrets.

If ChatGPT subscription or BYOK setup is best performed using Amp's official UI/CLI, CubeFarm can provide a Setup/Open Amp configuration action instead of duplicating that functionality.

Design this deliberately.

# Existing Codex integration

CubeFarm already supports Codex.

Study whether any of this work can be reused.

However, distinguish clearly between:

A. Codex CLI directly using a ChatGPT subscription

and

B. Amp using a ChatGPT subscription as one of its model-provider connections.

Those are different execution paths.

We want Amp to remain Amp when we choose Amp.

# UI

Study the current UI before proposing changes.

Keep the visual style and office metaphor.

Think about the smallest useful UI changes.

For example, there may eventually be:

New Thread / Hire Agent

Provider:
- Amp
- Claude Code
- Codex
- OpenCode

For Amp:

Execution:
- Cloud / Orb
- Runner
- Local

Project:
- inferred from current floor where possible

Task:
- prompt

Mode:
- supported Amp modes

Orb size:
- default / small / large etc., populated from supported configuration rather than hard-coded if possible

Advanced:
- title
- fast mode
- runner
- other genuinely useful supported options

The UI does NOT need every Amp CLI switch.

Optimize for the common workflow.

# Thread representation

Think carefully about the CubeFarm character model.

An Amp cloud agent should still be represented visually in the office even though there is no local PTY.

We may want its monitor/panel to show something more appropriate than a fake terminal:

- task
- status
- recent activity
- final response
- thread link
- Continue Thread
- Open in Amp
- GitHub branch/PR where available

Use the existing UI architecture as much as practical.

# Do not disturb upstream unnecessarily

This is our fork of a professional public repository.

Prefer:

- additive architecture
- small changes
- adapters
- clean interfaces
- preserving existing behavior
- keeping upstream changes mergeable when practical

Avoid:

- mass renaming
- formatting unrelated files
- architectural rewrites
- replacing working subsystems without a reason
- changing the existing visual design unnecessarily

We may continue pulling improvements from upstream later, so minimizing fork divergence is valuable.

# Testing strategy

Before implementation, identify how the new integration should be tested.

We want BOTH:

## Automated agent tests

Use the project's existing Vitest/Playwright architecture.

Design tests for the new Amp integration without spending real Amp/ChatGPT/API usage.

Amp CLI execution should be mockable/fakeable for automated tests.

Test things like:

- command construction
- parsing Amp output
- parsing thread URL/ID
- state transitions
- persisted thread metadata
- restart/reconciliation logic
- REST/WebSocket contracts
- UI state
- failure states

Do not make normal automated tests launch paid Amp orbs.

## Manual tests for me

Create a very clear manual testing sequence.

Start with the smallest possible real Amp test.

For example:

1. detect local Amp installation/auth state
2. launch ONE tiny Amp cloud thread
3. confirm CubeFarm captures its thread ID/URL
4. confirm it appears correctly in CubeFarm
5. open it in Amp
6. allow it to finish
7. confirm CubeFarm updates its state
8. send one follow-up if implemented
9. restart CubeFarm and verify the thread remains correctly represented

Then test model/subscription routing separately.

Be conscious of usage/cost.

# Security

Review the integration design specifically for secrets.

Do not store:

- ChatGPT auth tokens
- AMP_API_KEY
- OpenAI API keys
- Anthropic API keys
- other provider secrets

unless absolutely required by an official supported integration, and if something must be stored, explain why and how it will be protected BEFORE implementation.

Prefer inherited environment/authentication managed by the relevant CLI.

Do not print secrets into logs or browser messages.

# Phase planning

I expect this work to be divided into sensible phases rather than implemented all at once.

A likely shape might be:

Phase 1:
Amp detection + abstraction + start one Amp cloud thread + store ID/URL + basic UI representation.

Phase 2:
monitor/reconcile Amp thread status + restart persistence + open thread.

Phase 3:
continue thread / manager controls.

Phase 4:
mode/execution/orb options and polished setup.

Phase 5:
deeper CubeFarm workflow integration if justified.

But do not blindly use these phases.

After studying the actual code and Amp capabilities, propose the sequence you think is technically cleanest.

# What I want from you NOW

DO NOT CHANGE CODE YET.

Return a planning report with these sections:

## 1. How CubeFarm currently works

Only the architecture relevant to this integration.

Explain the agent lifecycle from UI → server → coding agent → GitHub/QA.

## 2. Best integration point

Tell me where Amp should enter the architecture and why.

Mention the specific files/components likely involved.

## 3. Amp capabilities verified

For every important capability we need, say:

- supported
- partially supported
- not exposed / uncertain

Include the official mechanism/command/API.

Especially cover:

- create cloud thread
- select project
- select execution environment
- select mode/model
- ChatGPT subscription
- BYOK
- get thread ID/URL
- monitor progress
- get final result
- continue thread
- restore after CubeFarm restart
- list/find existing threads

## 4. Proposed architecture

Show the proposed components/interfaces and data flow.

Keep it as small as practical.

A simple diagram is welcome.

## 5. Proposed CubeFarm UX

Describe exactly how I would:

- connect Amp
- start an agent
- select relevant options
- monitor it
- open the real thread
- continue it

## 6. Authentication / ChatGPT / BYOK design

Explain how credentials should work without CubeFarm becoming a secrets manager.

## 7. Data model changes

Show the important new/changed TypeScript types conceptually.

Do not write implementation yet.

## 8. Implementation phases

Break the implementation into reviewable phases.

For each phase:

- goal
- files/subsystems affected
- user-visible result
- automated tests
- manual test I should perform
- risks/dependencies

## 9. Automated testing plan

Tell me what tests agents can run and what new tests should be added.

No real paid cloud usage in normal automated tests.

## 10. Manual test plan for me

Give me a short sequential test procedure beginning with the safest tiny real Amp thread.

## 11. Decisions I need to make

Only list decisions that genuinely require product preference from me.

Do not ask questions whose answers can be obtained from the repository or official documentation.

## 12. Recommended first implementation milestone

Define the smallest end-to-end milestone that proves the architecture works.

It should ideally demonstrate:

CubeFarm → start Amp cloud thread → obtain Amp thread ID/URL → represent it in CubeFarm → open real Amp thread → recognize completion.

# Working rules

- Study first.
- Use current official Amp documentation.
- Verify assumptions.
- Evidence over confidence.
- Preserve upstream CubeFarm architecture where possible.
- No unrelated cleanup.
- No speculative rewrites.
- No secrets in code/state/logs.
- No paid API/orb testing without explicitly telling me first.
- Keep automated tests offline/faked wherever possible.
- Use the project's existing testing conventions.
- Do not implement until I approve the plan.

Research → Plan → Approve → Build → Verify → User Test → Ship.

STOP after the plan and wait for my approval.

One thing I intentionally changed from our previous onboarding prompt is that this one tells Amp **not to search for “issues” in CubeFarm at all**. Its job is to understand the upstream design and find the **smallest clean integration seam**.

I also strongly prefer the first milestone in that prompt: **CubeFarm launches one Amp Orb → captures `T-…` → displays it → opens the real thread → recognizes completion.** Once that works, model/mode selection, runners, continuation, richer monitoring, and GitHub workflow integration can be layered on without betting the whole fork on a large redesign.

And your ChatGPT-subscription goal is especially realistic now: Amp's current model routing explicitly supports a ChatGPT subscription alongside BYOK/provider connections, so I'd let **Amp own that entire credential/routing layer** rather than making CubeFarm manage OpenAI keys itself. [AmpCode](https://ampcode.com/docs/customize/model-routing?utm_source=chatgpt.com)

---

can I shortly add at the beginning or end that it needs to support login for both me and my teammate, maybe password or google login, password might be better since I don't want everyone to be able to login and use our tokens

---

Yes. I’d add this near the **Authentication and model usage** section:

## CubeFarm user access

This CubeFarm deployment must support secure login for at least two human users: me and my teammate.

Do not make the interface publicly accessible just because someone knows the URL.

Prefer a simple private authentication system suitable for a small trusted team, such as individual username/email + password accounts. Google login is optional, not required.

Requirements:

- separate account for each user
- passwords stored securely using modern password hashing; never plaintext
- authenticated session required before accessing CubeFarm
- logout/session expiry
- no public self-registration by default
- only explicitly created/approved users can sign in
- authentication must protect all sensitive UI/API/WebSocket routes, not just hide the frontend
- Amp, ChatGPT, BYOK and other provider credentials/tokens must never be exposed to another unauthorized visitor
- do not put authentication secrets or passwords in source code

For the initial version, optimize for a simple secure two-user deployment rather than building a large multi-tenant user-management system.

Study the existing server architecture and include the cleanest authentication approach in the implementation plan before coding.

I’d definitely use **individual passwords rather than one shared password**. That gives you separate sessions, easy access revocation for one person, and avoids sharing the same credential.

Also important: the agent should protect **the server APIs and WebSockets**, not merely add a login screen. Otherwise someone could potentially bypass the UI and still reach the CubeFarm backend.
