# AI Agent Security Research

Independent, hands-on research into how autonomous AI agents behave in real security work: what they can actually do on their own, where they break or hold back, and what that means for the people who defend systems for a living.

**Author:** Carlos Alvarado, Technology Consultant · *Cybersecurity, Applied AI & Research* · Founder of [Nova Cybersec](https://novacybersec.com)
**Profile:** [github.com/deepc0re85](https://github.com/deepc0re85)

---

## What this is

I run my own experiments. I build controlled, isolated labs on equipment I own, point AI systems at real problems, and write down what happens, the method, the results, and the limitations, so the work holds up and other people can learn from it.

The field around AI and security is full of claims right now. My goal with this repository is simple: add evidence to that conversation instead of more noise. Each entry is a self-contained case study, documented the way I'd want to read it if someone else had done the work.

This is my personal research. Nova Cybersec is where I apply this professionally, but the work here is mine and is not a product pitch.

## Responsible-research posture

Everything in this repository is written to be read, not reused as an attack.

- Experiments run in **isolated, authorized labs on my own equipment**, fully separated from any production system or third party.
- Targets use **known, already-patched vulnerabilities**. Nothing here claims a new or undisclosed flaw.
- I do **not** publish operational exploit chains, payloads, crafted requests, credentials, or step-by-step instructions that would help compromise a real device. Raw captures, full recordings, and sensitive artifacts are kept private.
- What you'll find instead are **sanitized excerpts and analysis**, enough to understand and verify the findings, not to weaponize them.

## Research log

### 01. Inside an AI Multi-Agent System: From Reconnaissance to FortiGate Administrative Access

A study of an autonomous multi-agent system (ARTEX, running on Claude) that was given only a target and a high-level goal, then coordinated a set of specialized agents through a shared knowledge graph to reach administrative access on a lab firewall affected by a known, patched vulnerability ([CVE-2022-40684](https://www.fortiguard.com/psirt/FG-IR-22-377)).

The focus is the **agent orchestration**, how the agents divided the work, shared facts, steered each other, and verified their own results, and what that behavior implies for defenders. It is **not** a how-to for the attack. It's also honest about the messy parts: getting the system to actually complete a run took real effort and several attempts, and I deliberately don't publish the specific model-and-prompt recipe that got it there.

→ [Read the case study](research/01-fortigate-multi-agent/REPORT.md)

## Repository structure

```
ai-agent-security-research/
├── README.md                     ← this file
└── research/
    └── 01-fortigate-multi-agent/
        ├── REPORT.md             ← the full case study
        └── evidence/             ← selected, sanitized artifacts
            ├── screenshots/
            └── logs/
```

## About the author

I'm a technology consultant based in Cancún, Mexico, working at the intersection of cybersecurity, applied AI, and research. I've spent years securing and running infrastructure for demanding environments, and these days a lot of my own time goes into applied research and building things in public. I care about evidence over hype, and about being honest regarding what a result does and doesn't show.

## License

Documentation and write-ups in this repository are released under **CC BY 4.0**. You're free to share and adapt with attribution. This license covers the written analysis only; no operational tooling or exploit material is included.

---

*Carlos Alvarado, Cybersecurity · Applied AI · Research*
