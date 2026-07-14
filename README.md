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

## Roadmap

- [ ] Auto-trigger on Jira status change (e.g. "Ready for Refinement" column) instead of manual chat trigger
- [ ] Optional short-summary comment back on the Jira ticket itself, linking to the full Slack report
- [ ] Evaluation harness to track scoring consistency across a larger ticket sample

---

## Author

**Nancy Bhardwaj** — Lead QA Engineer & Test Automation Architect
[LinkedIn](https://linkedin.com/in/nancy-bhardwaj) · [Topmate](https://topmate.io/nancy_bhardwaj) · er.nancybhardwaj@gmail.com

*Building AI Agents That Reduce Test Effort by 70%.*
