# LinkedIn Lead Generation Automation (n8n)

An automated cold outreach and lead qualification pipeline built with [n8n](https://n8n.io). It takes a list of prospects, sends LinkedIn connection requests, follows up with a structured messaging sequence, and classifies each lead based on how they respond — all without manual intervention.

## Overview

Given a list of prospects (with their LinkedIn profile URLs), the workflow:

1. Sends connection requests while respecting LinkedIn's sending limits.
2. Waits for the request to be accepted.
3. Sends a personalized first message (with an icebreaker) once accepted.
4. Follows up automatically if there's no reply.
5. Classifies each lead as **Qualified**, **Negative**, or **Archived** based on outcome.

## Workflow

### 1. Input
- A list of cold leads is provided, each with a LinkedIn profile URL.

### 2. Connection Request
- A connection request is sent to each lead.
- Sending volume/pacing respects LinkedIn's outreach limits to avoid account restrictions.

### 3. Acceptance Check
- The workflow waits for the request to be accepted.
- **If not accepted within 15 days** → the pending request is withdrawn and the lead is dropped from the active sequence.

### 4. First Message
- **Once accepted**, the workflow waits **24 hours**, then sends the first message, which includes an icebreaker.

### 5. Reply Handling
- **If the lead replies** → the lead is classified immediately as:
  - `Qualified` — response indicates interest.
  - `Negative` — response indicates disinterest/rejection.
- **If there is no reply after 2 days** → a second (follow-up) message is sent.
- **If there is still no reply after 4 more days** → a third and final follow-up message is sent.
- **If there is no reply 5 days after the last message** → the lead is classified as `Archived`.

### Lead Status Summary

| Status      | Trigger Condition                                              |
|-------------|------------------------------------------------------------------|
| `Qualified` | Lead replied and response indicates interest                    |
| `Negative`  | Lead replied and response indicates rejection/disinterest        |
| `Archived`  | No reply received after the full message sequence (connection → 3 messages → 5-day silence) |
| *(dropped)* | Connection request not accepted within 15 days                   |

### Timeline at a Glance

```
Day 0        Connection request sent
Day 0–15     Waiting for acceptance (drop if not accepted by day 15)
Accepted     +24h -> Message 1 (icebreaker)
No reply     +2d  -> Message 2 (follow-up)
No reply     +4d  -> Message 3 (final follow-up)
No reply     +5d  -> Archived
Any reply    -> Classified as Qualified / Negative
```

## Tech Stack

- **[n8n](https://n8n.io)** — workflow automation / orchestration
- **LinkedIn** — outreach channel (via automation-compatible integration)
- **Database** — Baserow

## Setup

1. Import the workflow JSON into your n8n instance (`Workflows > Import from File`).
2. Configure credentials for your LinkedIn connection/automation node.
3. Set your lead source (Baserow) as the input trigger.
4. Adjust timing parameters (15-day acceptance window, 24h/2d/4d/5d delays) in the wait nodes if you want a different cadence.
5. Configure the destination for classified leads (CRM, spreadsheet, database, Slack notification, etc.).
6. Activate the workflow.

## Customization

- **Icebreaker message**: edit the template in the first-message node; supports dynamic fields (name, company, etc.) from your input list.
- **Classification logic**: reply classification can be rule-based (keywords).
- **Timing**: all wait durations (15 days, 24 hours, 2 days, 4 days, 5 days) are configurable to match your outreach strategy.

## ⚠️ Compliance Note

This workflow automates actions on LinkedIn. Automating connection requests and messaging may conflict with LinkedIn's Terms of Service and can put your account at risk of restriction or suspension. Use conservative sending volumes, add randomized delays, and review LinkedIn's current policies before deploying this in production. Use at your own risk.
