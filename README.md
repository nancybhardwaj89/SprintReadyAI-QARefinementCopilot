# SprintReadyAI - QA Refinement Copilot

**Building AI Agents That Reduce Test Effort by 70%**

An AI agent that reviews Jira stories for QA refinement readiness — before your team's refinement meeting even starts. Type one line in chat, get a structured 14-point QA analysis posted straight to Slack.

---

## The Problem

Stories often reach sprint refinement half-baked — missing acceptance criteria, no thought given to edge cases, no one has asked the hard "what if..." questions until QA raises them live, in the meeting, in front of everyone.

Manually reviewing every story for QA readiness — reading through descriptions, spotting gaps, writing clarification questions, estimating QA effort — takes real time out of every sprint cycle.

## The Solution

SprintReadyAI is an n8n-powered AI agent that automates this review. Send it a Jira issue key in chat, and it:

1. Fetches the full ticket from Jira (title, description, acceptance criteria)
2. Analyzes it from a senior QA lead's perspective
3. Generates a structured, 14-section QA refinement report
4. Posts the report to a Slack channel for the whole team to see
5. Replies with a quick summary — readiness status, score, QA estimate, and top clarification questions

No manual ticket-reading. No "let me get back to you after I review this."

---

## How It Works

**Trigger:** `Refine story SCRUM-23` typed in chat

**Flow:**

```
When chat message received
        ↓
   AI Agent  ──────────────┬──────────────┬────────────────────┐
        │                  │              │                    │
   OpenAI Chat Model   Simple Memory   Get an issue        Post Report
   (reasoning)          (context)      in Jira Software     in Slack
```

| Node | Role |
|---|---|
| **When chat message received** | Trigger — user sends an issue key |
| **AI Agent** | Runs the system prompt logic below |
| **OpenAI Chat Model** | Language model powering the analysis |
| **Simple Memory** | Keeps conversation context across messages |
| **Get an issue in Jira Software** | Tool — fetches ticket details from Jira |
| **Post Report in Slack** | Tool — posts the formatted report to Slack |

The full exported workflow is in [`qarefinementcopilot.json`](./qarefinementcopilot.json) — import it directly into n8n.

---

## The QA Refinement Report

Every report follows a fixed 14-section structure:

1. QA Understanding
2. Story Readiness Status (Ready / Partially Ready / Not Ready)
3. Story Readiness Score (out of 100)
4. Missing Acceptance Criteria
5. QA Clarification Questions
6. Functional Test Scenarios
7. Negative and Edge Test Scenarios
8. Regression Impact
9. API Impact
10. UI/UX Impact
11. Test Data Needs
12. Dependencies / Blockers
13. Suggested QA Estimate
14. QA Recommendation

**The system prompt is built around one hard rule: never invent Jira data or assume unstated requirements.** If details are missing, the agent flags them and asks — it doesn't fill gaps with plausible-sounding guesses.

### Tested across the readiness spectrum

| | Well-defined story | Vague story |
|---|---|---|
| **Status** | Ready | Not Ready |
| **Score** | 92 / 100 | 40 / 100 |
| **QA Points** | 5 | 3 |
| **Behavior** | Minor gaps only | Flags missing error handling, token expiry, validation rules — with specific clarification questions |

See `Slack Report - Ready Story.png` and `SlackReport - Not Ready Story.png` for real output examples.

---

## Screenshots

- `Chat Interface.png` — triggering a refinement from chat
- `n8n Workflow.png` — the full agent workflow
- `Slack Report - Ready Story.png` — a Ready (92/100) report
- `SlackReport - Not Ready Story.png` — a Not Ready (40/100) report

---

## Tech Stack

- **n8n** — workflow orchestration and AI Agent node
- **OpenAI** — reasoning / language model
- **Jira Software (Cloud) API** — fetching ticket data
- **Slack API** — posting reports to a team channel
- **Simple Memory** — conversation context across the session

---

## Setup

1. Import `qarefinementcopilot.json` into your n8n instance.
2. Connect credentials:
   - Jira Software Cloud account (API token)
   - Slack app with `chat:write` and `chat:write.public` scopes, installed to your workspace
   - OpenAI API key
3. Invite your Slack app/bot to the target channel (e.g. `#qa-refinement-copilot`).
4. Paste the system prompt into the AI Agent node's System Message field.
5. Activate the workflow and test with a message like `Refine story PROJ-123`.

**Notes:**
- Jira's API posts comments/fields as plain text — it does not render Markdown or wiki markup.
- Slack renders basic Markdown natively (`*bold*`, `- ` bullets, `---` dividers) — the prompt is tuned to take advantage of this for the report formatting.

---

## Evaluation

Manual testing only proves an agent worked *that one time*. To actually trust the readiness scoring, this project includes a [Promptfoo](https://www.promptfoo.dev/) evaluation suite in [`eval_promptfoo/promptfooconfig.yaml`](./eval_promptfoo/promptfooconfig.yaml) that calls the live n8n webhook and checks the response against defined pass/fail criteria — a regression suite you rerun any time the system prompt changes.

### Readiness scoring rubric

Score bands are fixed in advance, not derived from any single output:

| Score | Status | Meaning |
|---|---|---|
| 80–100 | Ready | Acceptance criteria are specific and cover the main flow plus edge cases |
| 50–79 | Partially Ready | A basic flow exists, but important details are missing |
| 20–49 | Not Ready | Only a high-level idea; acceptance criteria are vague or generic |
| 0–19 | Not Ready | Little more than a title — no testable detail at all |

### Test cases

| # | Case | Checks |
|---|---|---|
| 1 | Well-defined story | Scores 80+, marked Ready |
| 2 | Vague story | Score and status agree with each other (e.g. a 45 can't be labeled "Ready"), and must not land in the Ready band |
| 3 | Invalid Jira issue key | No fabricated report is generated; agent explains it couldn't find the issue |
| 4 | No issue key provided | Agent asks the user for one instead of guessing |
| 5 | Same well-defined story, run twice | Gives a consistent verdict on repeat runs |
| 6 | Extremely vague story (near-zero detail) | Scores below 50 — meaningfully lower than a "somewhat vague" story |

### What the eval suite caught

Two real issues surfaced that manual testing had missed:

- **Run-to-run inconsistency (Case 5):** the same ticket returned different scores on different runs (40/100 vs. 60/100). Root cause: LLM outputs are probabilistic by default. Fixed by lowering the model's temperature setting to reduce randomness in scoring.
- **Low scoring resolution at the vague end (Case 6):** a near-empty ticket scored identically to a moderately vague one — the agent wasn't discriminating between "missing some details" and "missing almost everything." Fixed in the system prompt, not the test, by adding the explicit scoring rubric above and instructing the agent not to default to a mid-range "safe" score for every vague ticket.

### Running it

```bash
cd eval_promptfoo
promptfoo cache clear
promptfoo eval --no-cache
promptfoo view
```

Before running, set your n8n Production webhook URL in `promptfooconfig.yaml`, and bump the `runTag` value at the top of the file before each fresh run — this guarantees new n8n memory sessions so the agent doesn't recall a "completed" session from a previous run.

---

## Roadmap

- [ ] Auto-trigger on Jira status change (e.g. "Ready for Refinement" column) instead of manual chat trigger
- [ ] Optional short-summary comment back on the Jira ticket itself, linking to the full Slack report
- [x] Evaluation harness to track scoring consistency across a larger ticket sample

---

## Author

**Nancy Bhardwaj** — Lead QA Engineer & Test Automation Architect
[LinkedIn](https://linkedin.com/in/nancy-bhardwaj) · [Topmate](https://topmate.io/nancy_bhardwaj) · er.nancybhardwaj@gmail.com

*Building AI Agents That Reduce Test Effort by 70%.*
