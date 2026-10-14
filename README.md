# odc-brd

A Claude Code skill that writes (or reviews) a **Business Requirements Document for an OutSystems Developer Cloud (ODC) app**, structured so it can be built reliably through Mentor and the `outsystems` MCP, and tested afterwards.

It produces a text-and-tables Markdown build spec: roles and access, data model, numbered business rules with error codes, flows, screens, seed data with expected values, test cases, and a staged build order with test gates.

Derived from real ODC builds (a CRM and an insurance portal) and the problems hit while building them with Mentor.

## What you get

- An intake step that asks only the questions your input leaves open
- A 15-section BRD template (`odc-brd/references/brd-template.md`)
- ODC and Mentor-specific guidance: atomic transactions, no polymorphic FKs, test personas, REST status codes, Mentor stall limits
- A self-audit checklist and a decisions log (D-xx) so corrections do not silently rewrite the spec
- A per-stage brief to hand the BRD to a build agent

## Requirements

- Claude Code (terminal, desktop Code tab, or IDE extension)
- For building from the BRD (optional): the `outsystems` MCP connected and authorized, plus the companion skills `odc-build-playbook` and `odc-best-practices`

## Install

**Personal skill (all projects), macOS / Linux / Git Bash**

```bash
git clone https://github.com/rohitsingh-sg/outsystems-odc-brd.git
mkdir -p ~/.claude/skills
cp -r outsystems-odc-brd/odc-brd ~/.claude/skills/
```

**Windows PowerShell**

```powershell
git clone https://github.com/rohitsingh-sg/outsystems-odc-brd.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse outsystems-odc-brd\odc-brd "$env:USERPROFILE\.claude\skills\"
```

**Project skill (shared with a team):** copy the inner `odc-brd` folder to `.claude/skills/odc-brd/` in the project repo and commit it.

**Claude Desktop chat / claude.ai:** zip the inner `odc-brd` folder (with `SKILL.md` inside) and upload it under Settings, Capabilities, Skills. Menu names may differ by version.

Start a new session afterwards so the skill loads.

## Usage

Type `/odc-brd`, or ask in plain words:

- "Write an ODC BRD for a leave-management app"
- "Review this requirements doc and turn it into an ODC build spec"
- "Here are my mock-ups, what's missing before I build in ODC?"

Claude asks the intake questions, drafts the BRD, audits it, and shows a summary for you to confirm. Corrections are logged as decisions.

## Repository layout

```
outsystems-odc-brd/          <- repo root
  README.md
  LICENSE
  odc-brd/                    <- the skill folder to install
    SKILL.md
    references/
      brd-template.md
      intake-questions.md
```

## Limitations

- Not yet tested on many projects; improve it from what goes wrong.
- Chunk sizes and Mentor limits are observations, not measured figures.
- ODC hints in generated BRDs must be checked against your tenant's current features.

## License

MIT. See `LICENSE`.
