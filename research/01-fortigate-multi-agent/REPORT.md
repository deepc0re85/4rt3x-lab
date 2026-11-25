# Inside an AI Multi-Agent System: From Reconnaissance to FortiGate Administrative Access

**Author:** Carlos Alvarado, Technology Consultant · Founder, Nova Cybersec
**Date:** November 2026
**Status:** First public case study. Observations drawn from a single experimental run in a controlled lab.

> **Scope and safety note.** This document analyzes the *behavior* of an autonomous multi-agent AI system during a security test against a device I own, in an isolated lab, using a publicly known and already-patched vulnerability. It deliberately omits operational detail: there are no crafted requests, payloads, commands, credentials, or step-by-step exploitation instructions. The goal is to understand how the agents worked, not to enable the attack.

---

## Abstract

I gave an autonomous agent framework a single address and one instruction, reach administrative access on the target, and watched what it did without walking it through the steps. The system coordinated a set of specialized agents, mapped the target, identified a known authentication-bypass condition, and worked its way to full administrative control of the device, which it then verified on its own.

What makes this worth writing up is not the vulnerability. It's old, public, and patched. It's *how* the system got there: the division of labor between agents, the shared knowledge graph they used as a common memory, the way one agent's findings became another's next task, and the fact that the system verified its own result before reporting it. This case study documents that behavior and is honest about what the run did and did not prove, including the messy part, because getting the system to actually complete a run took real work (§8).

---

## 1. Context and objective

Most conversations about "AI doing security" skip the part that actually matters to a defender: when you hand an autonomous system a goal and step back, what does it really do, how far does it get, and where does it stall? I wanted to observe that directly, with my own eyes and my own equipment, rather than take anyone's word for it.

The objective of this run was narrow and concrete: **starting from only a network address and the goal of obtaining administrative access, let an autonomous multi-agent system attempt it on its own, and record the sequence and, above all, how the agents coordinated.**

This is a capability-and-behavior study. It is not a vulnerability disclosure (the flaw is already public) and it is not a benchmark of one tool against another.

---

## 2. Lab environment

- **Target:** a FortiGate-VM appliance running FortiOS 7.2.0, affected by **CVE-2022-40684**, a known authentication-bypass vulnerability that Fortinet disclosed and patched in 2022. The device is mine, deployed only for this research.
- **Isolation:** the lab is separated from any production or third-party system. Internal addressing, serial numbers, and device identifiers are withheld from this write-up.
- **Autonomous system:** **ARTEX**, an autonomous offensive-security agent framework, driven by a **Claude model**. ARTEX supplies the orchestration layer, the agents, their shared memory, and the coordination tools, while the model supplies the reasoning. (Getting a run to complete took experimentation across several models and settings, see §8. The specific model version and the prompt that worked are deliberately not disclosed.)
- **Available tooling:** standard offensive tooling was present in the environment. (See §7, the run's own records show which capabilities were actually exercised, which is not the same as what was installed.)

*Observed fact: target software version and the associated CVE are confirmed in the system's own findings records. Specific device identifiers are intentionally not reproduced here.*

---

## 3. Methodology and conditions

- **Input to the system:** one target address and one high-level goal ("gain administrative access"). No exploitation steps, no hints about the vulnerability class, were provided up front.
- **Human involvement: the setup, not the execution.** I provided the target, the goal, and the task framing up front. What I did **not** do was steer it step by step during the run. The *hints* and *steering* that show up in the records were posted by the agents themselves (see §4), agent-to-agent coordination, not me nudging it along. The honest caveat sits elsewhere: the framing I chose at the start mattered (§8). So the accurate description is autonomous *execution* within a human-set, deliberately-framed goal, not zero-touch autonomy.
- **Observation:** the run was watched live through ARTEX's web interface and supporting consoles, and is reconstructed here from the system's structured records (its knowledge graph, agent activity, tool usage, and generated findings) plus saved evidence artifacts.
- **Repetition:** this is a **single run**. The findings describe what happened once, not a measured success rate across trials.

---

## 4. Multi-agent architecture and coordination

ARTEX doesn't run as one model in a loop. It splits the job across specialized agents that share a common memory and hand work to each other.

### The agents

| Agent | Role in the run |
|---|---|
| **goals** | Decomposes the overall objective into independent, verifiable sub-goals. |
| **planner** | Reads the current situation and decides what to explore next, adding new *intents* only when there's genuinely uncovered ground. |
| **worker** | Picks up one intent, carries it out, writes the facts and findings it discovers back into the shared graph, then stops. |
| **pentest** | A self-contained agent able to run a full chain on its own: recon → attack surface → deeper work → verification → wrap-up, planning and adversarially checking itself. |
| **mainagent** | The human interface: turns an operator's intent into a *hint* or a high-priority task for the others. |
| **auto** | Platform operations: manages the task and assets, and can extend the system's own tools. |
| **reporter** | Triggered when a finding appears; reads the evidence and the execution trace and writes it up. |
| **retester** | Independently re-checks a finding against its original evidence. |

### How they coordinated

The agents did not talk in free-form chat. They coordinated through structured mechanisms that the system records, which is what let me reconstruct the run honestly:

- A **shared graph that doubles as the agents' message board.** Every unit of coordination is a typed node on it, the schema itself defines the kinds: `begin`, `goal`, `intent`, `fact`, `finding`, `hint`, and `digest`. The agents read and write this board; it is both their common memory and their channel to each other.
- **Intents**, one-sentence exploration directions the planner posts and workers claim.
- **Facts**, discoveries workers record back to the board, which become the basis for the next intent.
- **Hints**, a first-class node type: an agent posts a hint to flag a higher-priority direction for the others. In this run, the hints and steering were posted **by the agents themselves**, the planner and the operations agent. The agents were steering each other.
- **Worker traces**, the execution record one agent leaves, which other agents (and the reporter) read.
- **Goal proving**, explicit confirmation that a sub-goal was met before moving on.

In this run, the system's own activity records show the planner doing the heaviest coordination, including posting hints and steering the work, with workers executing, the reporter documenting, and the auto agent handling platform operations. The result is closer to a small team leaving each other notes on a shared board than to a single chatbot following a script.

*Observed fact: agent roles and the coordination mechanisms above are taken directly from the system's configuration and activity records.*

---

## 5. Observed sequence (high level)

Reconstructed from the knowledge graph and findings, deliberately kept at the level of *what* happened, not *how*:

1. **Reconnaissance and attack-surface mapping.** The system enumerated the target's exposed services and identified the technology and version in use.
2. **Identification of a known weakness.** It recognized that the target was exposed to a known authentication-bypass condition.
3. **Unauthenticated information disclosure.** It reached configuration data on the device without valid credentials.
4. **A privileged configuration change.** It used that access to establish administrative control and a means of persistence.
5. **Validation.** It confirmed administrative access independently, then handed off to the reporter, which wrote up the findings.

Each step above corresponds to recorded findings and saved evidence artifacts (kept private; see §9).

---

## 6. Results

The run produced **four findings, all rated critical** by the system, forming a single chain that ended in complete administrative takeover of the target:

1. **Authentication bypass** (CVE-2022-40684), the entry condition.
2. **Configuration disclosure**, sensitive device configuration read without authentication.
3. **Privilege escalation and persistence**, an administrative change that established lasting control.
4. **Full administrative takeover**, validated administrative access to the device.

The end state: starting from an address and a goal, the system reached and confirmed administrative control of the target on its own, and documented the result.

*Observed fact: four critical findings and the final administrative-access state are recorded in the system's findings data and supported by saved evidence artifacts.*

---

## 7. What the data shows, and what it doesn't

This section separates what the records prove from what they don't.

- **The coordination is real and measurable.** The multi-agent behavior in §4 comes straight from the system's own activity records, not from a narrative I wrote after the fact.
- **No external exploitation framework was invoked in this run.** The system's tool-usage records contain only its internal orchestration tools (graph reads/writes, intents, hints, worker traces, reporting). Offensive tooling was available in the environment, but the records do **not** show a separate framework such as Kali-MCP, nmap, or sqlmap being called through the tracked tool layer. The offensive actions were carried out by the agents directly. **I therefore do not claim that any particular external toolkit drove this result.**
- **The in-run coordination was agent-driven, and I verified it.** The hints and steering in the record were posted by the agents, the planner and the operations agent, not relayed from me. My input was the setup: the target, the goal, and the framing. The accurate description is autonomous *execution* within a human-set, deliberately-framed goal: not zero-touch autonomy, but not me nudging it through either.
- **The vulnerability was known and provided context, not discovered.** This is not a zero-day and not original vulnerability research.
- **One run is not a success rate.** Nothing here measures reliability, repeatability, or how the system performs on an unknown or patched target.
- **No comparison was made** between this framework and any other. Claims of relative capability would need a controlled comparison this run did not do.

*Interpretation (mine): what stands out is the coordination. A set of cooperating agents, given a goal, organized the work, kept a shared understanding of the target, and checked their own result before reporting, with only light human steering.*

---

## 8. Notes from the run

The clean summary above can make this sound easier than it was. Here is how the run actually went.

**Getting the system to really work took effort.** ARTEX is not plug-and-play. It stalled on me more than once, going quiet, or spinning without making real progress, and getting a run to carry the whole chain through to a verified result took patience and several attempts. An early run went well and I stopped it on purpose; later ones were less cooperative, and I kept having to adjust.

**I experimented with several models and configurations** before a run completed cleanly. The behavior varied a lot depending on the setup: which model was behind the agents, and how the task was framed. That variability is itself part of the finding, this is not a reliable push-button result, at least not in my hands.

**What I'd read versus what I actually needed.** Going in, I'd seen the common assumption that a job like this needs a full offensive toolchain wired up, a Kali environment, an MCP integration with a proxy like Burp, the usual kit. In this run, none of that turned out to be necessary. The system's own records (§7) back this up: the agents reached the result through their own execution and ARTEX's orchestration, with no separate external toolkit in the tracked loop. That surprised me, and I think it's worth saying plainly.

**On framing.** How I presented the task to the model mattered. I was working in a legitimate, authorized lab on my own equipment, and I described it that way. I'm not going to pretend that's only a technicality. That same dependence on framing means someone with stronger techniques than mine, better at getting a model to go along with a request, could likely push further than I did. That's a real safety concern, and it's the main reason I am **not** publishing the specific model version or the prompt that got this run to complete. I'll describe what the system did and how it coordinated; I won't hand anyone the exact combination that worked. If that detail matters to the people who should have it, it belongs in a private channel, not a public recipe.

---

## 9. Limitations and security considerations

- **Single, lab-only run** on a deliberately vulnerable, isolated device I own. Results do not generalize to patched systems, other devices, or real environments.
- **Operational detail is withheld by design.** This write-up cannot be used to reproduce the attack, and that is intentional.
- **Model and framework behavior can change** between versions and configurations; a specific setup's behavior is not a universal property.
- **Defensive takeaway:** the exposure that made this possible, an unpatched, internet-reachable device with a long-known critical vulnerability, is exactly the kind of thing routine patching and attack-surface management address. The interesting shift is that organizing and executing a chain like this no longer requires a skilled human at every step; a goal and a capable agent system can cover a lot of the middle. That raises the value of reducing known exposure, not of any single tool.

---

## 10. Evidence

Selected artifacts are under [`evidence/`](evidence/), captioned and mapped to the sections above. Raw captures, full recordings, device configuration, credentials, and the system's raw database are **kept private** and are not part of this repository.

- `evidence/screenshots/`, stills from the recorded session showing the opening prompt, the exploration-path graph, the broadcast board and its typed nodes, the tool-usage record, the findings, and the end state. See [`evidence/README.md`](evidence/README.md) for the full list and captions.
- `evidence/screenshots/zoom/`, close-ups for legibility (graph legend, hints column, tool table, traffic and service detail).

## References

- Fortinet PSIRT advisory **FG-IR-22-377**, CVE-2022-40684: https://www.fortiguard.com/psirt/FG-IR-22-377
- NVD, CVE-2022-40684: https://nvd.nist.gov/vuln/detail/CVE-2022-40684
- CISA Known Exploited Vulnerabilities Catalog (CVE-2022-40684 listed)
- Anthropic, Claude: https://www.anthropic.com

---

*Carlos Alvarado · Cybersecurity · Applied AI · Research · [github.com/deepc0re85](https://github.com/deepc0re85)*
