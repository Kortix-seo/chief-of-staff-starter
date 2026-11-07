# chief-of-staff-starter

An open-source starter for an AI chief of staff: six scheduled agents that run a company's operating cadence, declared as files in one repo, with one rule about what may leave the building.

This is a working `kortix.yaml` for [Kortix](https://kortix.com), the open-source CompanyOS. Clone it, point the connectors at your own Gmail, Linear and metrics sheet, and the cadence runs on a schedule instead of on someone's memory.

## The six jobs

| Job | When it runs | What it produces |
| --- | --- | --- |
| `weekly-report` | Mondays 06:00 | The numbers report, with commentary, posted to `#metrics` |
| `standup-summary` | Weekdays 08:30 | The standup, grouped by person, posted to `#standup` |
| `meeting-notes` | Every 15 minutes | Notes and Linear action items for every meeting that ended |
| `investor-update` | The 1st, 09:00 | A drafted investor update, saved unsent |
| `inbox-triage` | Every 15 minutes | Labels, drafted replies and Linear tasks |
| `qbr-prep` | Quarterly, the 1st, 08:00 | The quarterly review briefing, saved unsent |

Each job is a trigger in `kortix.yaml` and an agent prompt in `agents/`. A trigger names the agent, the schedule and the first message. A run starts in a session of its own, on a machine spun up for it, so a slow Monday report never holds up the inbox sweep.

## The send rule

Two of the six jobs leave the building: the inbox and the investor update. The manifest says what happens when they try.

```yaml
connectors:
  - slug: gmail
    provider: composio
    app: gmail
    policies:
      - match: "gmail.send_*"
        action: require_approval
      - match: "gmail.reply_*"
        action: require_approval
      - match: "gmail.forward_*"
        action: require_approval
      - match: "gmail.delete_*"
        action: block

policy:
  default_mode: risk
approvals:
  ttl: 4h
```

`require_approval` holds the call until a person approves it. `block` means the call never runs. `policy.default_mode: risk` lets a read through and holds a write or a delete, so a connector with no rule of its own still cannot send. `approvals.ttl: 4h` is how long a held call waits before it expires.

A grant is a separate question from a policy. An agent with no grant for a connector cannot call it at all, and `kortix validate` prints `none (default-deny)` for that agent. The real choice is four-way: no grant, allow, ask, or block.

## Validate it

```console
$ kortix validate
  ✓  kortix.yaml is valid

What each agent enables:
  weekly-report  skills=[none (default-deny)]  subagents=[none (default-deny)]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[google_sheets, kortix_slack]  secrets=[none (default-deny)]
  standup-summary  skills=[none (default-deny)]  subagents=[none (default-deny)]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[linear, kortix_slack]  secrets=[none (default-deny)]
  meeting-notes  skills=[meeting-recap]  subagents=[none (default-deny)]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[linear, kortix_slack]  secrets=[none (default-deny)]
  investor-update  skills=[none (default-deny)]  subagents=[none (default-deny)]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[google_sheets, gmail]  secrets=[none (default-deny)]
  inbox-triage  skills=[none (default-deny)]  subagents=[none (default-deny)]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[gmail, linear]  secrets=[none (default-deny)]
  qbr-prep  skills=[none (default-deny)]  subagents=[none (default-deny)]  commands=[none (default-deny)]  extensions=[none (default-deny)]  connectors=[google_sheets, linear]  secrets=[none (default-deny)]
```

That output is from `kortix validate` on this repository, Kortix CLI v0.14.20.

## Run it

```bash
curl -fsSL https://kortix.com/install | bash
kortix init
# copy kortix.yaml, agents/ and skills/ into the new project
kortix validate
kortix ship
```

`kortix ship` validates the manifest, commits, offers to set missing secrets, pushes the branch and connects the declared connectors. Then connect Gmail, Linear and the metrics sheet in **Connectors**, and the six triggers start firing on their schedules.

## What to change first

1. **The metrics sheet.** `weekly-report`, `investor-update` and `qbr-prep` read it. Point them at your own sheet and rename the columns in the prompts.
2. **The Slack channels.** `#metrics` and `#standup` are placeholders.
3. **The send rule.** Widen it only where you mean to. `gmail.send_*` at `always_run` is a decision, and it belongs in a diff someone reviews.

## Files

```
kortix.yaml              the manifest: six agents, six triggers, the connector policies
agents/*.md              one prompt per job
skills/meeting-recap/    the skill the meeting-notes agent loads
```

## Choosing what to run it on

The starter is the build; the plan is a separate question. [The AI chief of staff tools comparison](https://aichiefofstaffguide.com/ai-chief-of-staff-tools) sets out what each product covers and where its slice ends, and [what an AI chief of staff costs](https://aichiefofstaffguide.com/ai-chief-of-staff-cost) works through the published plans and the work each one leaves on your desk.

## Licence

The manifest and the prompts in this starter are yours to change and ship. Kortix itself is open source; its licence sits in the [Kortix repository](https://github.com/kortix-ai/CompanyOS).

## More

- [What an AI chief of staff is, and the six jobs it owns](https://aichiefofstaffguide.com/what-is-an-ai-chief-of-staff) — the working guide this starter comes from.
- [Kortix documentation](https://kortix.com/docs) — the `kortix.yaml` reference, triggers, connectors and approvals.
