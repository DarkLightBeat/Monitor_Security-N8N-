# Monitor Security — AI-Powered Security Advisory Automation

An n8n workflow that continuously monitors external security advisory feeds, uses an AI agent to
assess relevance against a target tech stack, and automatically routes critical issues to the right
people — cutting the time between "a vulnerability was disclosed" and "the team knows about it"
from hours to minutes.

Built for **HexCoreArena AI Hackathon** by a team of 2–3.

## How it works

1. **Collect** — Pulls advisories on a schedule (every hour) from three sources:
   - Palo Alto security advisories (RSS)
   - CISA Known Exploited Vulnerabilities (KEV) catalog (HTTP)
   - GitHub Security Advisories (HTTP)
2. **Normalize** — Each source has a different shape; dedicated code nodes normalize all three into
   a single consistent format.
3. **Deduplicate** — Checks a Google Sheets–backed database to skip advisories already seen and
   ignore anything stale (older than 24 hours).
4. **Assess relevance** — An AI agent (Groq-hosted LLM) compares each advisory against the
   organization's tech stack and extracts structured severity/relevance data.
5. **Route by severity**:
   - **Not relevant** → discarded
   - **Relevant + High/Critical** → creates an urgent Jira ticket **and** emails SecOps/management
     immediately
   - **Relevant + Lower severity** → creates a backlog Jira ticket for later triage
6. **Log** — Every processed advisory is saved back to Google Sheets, both as an audit trail and as
   the dedup source for future runs.

A secondary chat-trigger flow also lets a team member ask the AI agent questions on demand outside
the scheduled run.

## Tech stack

- **n8n** — workflow orchestration
- **AI Agent (Groq-hosted LLM)** — relevance/severity assessment via `@n8n/n8n-nodes-langchain`
- **Jira API** — automatic ticket creation (urgent vs. backlog)
- **Gmail API** — SecOps/management alerting
- **Google Sheets API** — advisory database, dedup, and audit log
- **RSS / HTTP Request nodes** — advisory ingestion from Palo Alto, CISA KEV, and GitHub

## Why this matters

Security teams are flooded with advisories from dozens of sources, most of which are irrelevant to
their actual stack. This workflow filters the noise automatically and only interrupts a human when
something both (a) applies to systems they run and (b) is severe enough to need urgent action —
while keeping a full audit trail of everything that was seen and how it was triaged.

## Setup

1. Import `Monitor_Security.json` into an n8n instance.
2. Configure credentials for: Google Sheets, Jira, Gmail, and your Groq API key.
3. Replace the "Get Tech Stack (Mock)" node with a real data source (e.g., a CMDB or asset
   inventory) for production use.
4. Activate the schedule trigger to run hourly, or trigger manually for testing.

## Team

Built as a hackathon submission for HexCoreArena AI Hackathon.
