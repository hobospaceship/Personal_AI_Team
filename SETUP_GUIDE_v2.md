# AI Agent Team — Setup Guide v2.0
*Updated May 2026 | Replaces AI_TEAM_SETUP_GUIDE.md v1.0*

---

## What This Guide Covers

A complete blueprint for setting up an autonomous AI agent team that:
- Runs every morning at 7 AM without you opening Claude Code
- Reads your real calendar events, Google Meet transcripts, Slack DMs, and priority channels
- Posts a structured brief to a private Slack channel with decisions, drafts, R&D intel, and V2MOM-anchored career nudges
- On Mondays, posts a weekly planning brief instead — covering the full prior week and setting focus for the week ahead

**Estimated setup time:** 2-3 hours  
**Ongoing maintenance:** ~10 min/week

---

## Architecture Overview

```
System cron (7:03 AM daily)
  → claude -p "Run today's brief"
      → Orchestrator agent
          → Spawns 5 specialists in parallel:

Admin          Comms           PM              R&D             Career
─────          ─────           ──              ───             ──────
Google         Slack DMs       Meeting         Slack channel   Memory files
Calendar       (key contacts   transcripts     searches        + V2MOM files
               + AXS team)     + Slack         (verified tool
                               channels        calls only)

          → Synthesizes all outputs
          → Posts brief to #yourname-ai-team
          → Threads decisions with ready-to-approve drafts
          → Archives to ~/.claude/memory/briefings/archive/
```

**Key principle: Nothing is invented.** Every agent uses verified MCP tool names. If a tool returns no results, the agent reports that honestly and surfaces nothing from that source.

---

## Section 1 — Prerequisites

### 1.1 Claude Code CLI

```bash
npm install -g @anthropic-ai/claude-code
claude --version
claude login
```

You need a Claude Pro or Max subscription.

### 1.2 AI Expert Suite (MCP Connections)

AI Expert Suite provides the MCP servers that give agents access to Google Workspace and Slack. Connect it in Claude Code settings.

**What it provides:**
- `mcp__plugin_google_google__calendar_events` — Google Calendar
- `mcp__plugin_google_google__meeting_notes_search` — Google Meet transcripts (Gemini-generated)
- `mcp__plugin_google_google__docs_get` — Read Google Docs
- `mcp__plugin_gmail_gmail__gmail_search` — Gmail
- `mcp__plugin_slack_slack__slack_read_channel` — Read Slack channels and DMs
- `mcp__plugin_slack_slack__slack_search_public_and_private` — Search Slack
- `mcp__plugin_slack_slack__slack_search_channels` — Find channel IDs
- `mcp__plugin_slack_slack__slack_search_users` — Find user IDs
- `mcp__plugin_slack_slack__slack_send_message` — Post to Slack

### 1.3 Enable Gemini Note-Taking in Google Meet

This is the highest-value input for the system. Without it, the admin and PM agents lose their best source of prior meeting context.

In Google Meet settings → enable "Take notes with Gemini" by default for all meetings.

Transcripts appear automatically in Google Drive and are searchable via `meeting_notes_search`.

### 1.4 Create Your Agent Channel in Slack

Create a private Slack channel: `#yourname-ai-team`  
Add only yourself.

To find the channel ID: right-click the channel → Copy link. The ID is the string starting with `C` at the end of the URL.

---

## Section 2 — Memory System

All agents read from and write to a shared file-based memory system at `~/.claude/memory/`.

### 2.1 Create Directory Structure

```bash
mkdir -p ~/.claude/memory/core/v2moms
mkdir -p ~/.claude/memory/agents
mkdir -p ~/.claude/memory/episodic
mkdir -p ~/.claude/memory/briefings/archive
mkdir -p ~/.claude/logs
```

### 2.2 Core Memory Files

#### `~/.claude/memory/core/relationships.md`

Store everyone you work with regularly — clients, leadership, teammates. Include their Slack user IDs (needed for DM reads).

To find a Slack user ID: click their profile → three dots menu → Copy member ID. Or ask Claude: "Find the Slack user ID for [Name]"

```markdown
# Key Relationships

## Active Clients

### [Client Name]
- [Contact Name] (@SlackUserID) — [Role] — [Context]
- Channels: #client-channel-1, #client-channel-2

## Leadership
- [Manager Name] (@SlackUserID) — [Role]

## AXS / Team Members
- [Name] (@SlackUserID) — [Role]
```

#### `~/.claude/memory/core/preferences.md`

```markdown
# Preferences

## Communication
- Async-first. Push only what needs a decision.
- Timezone: America/[Your timezone]

## Briefing
- Length: concise, one scannable screen
- Tone: direct, no preamble

## Scheduling
- Client-facing calls: prefer mornings
- Buffer between meetings: 15 min minimum
```

#### `~/.claude/memory/core/decisions.md`

Start with any key decisions already made on active projects. Format:

```markdown
## [Date] — [Decision]
- What: [What was decided]
- Why: [Reasoning]
- How to apply: [When this pattern applies]
```

### 2.3 V2MOM Files

This is what powers the career nudge and Monday planning brief's V2MOM pulse.

```bash
# Create files for your V2MOM and your manager's
touch ~/.claude/memory/core/v2moms/your-name-v2mom.md
touch ~/.claude/memory/core/v2moms/manager-name-v2mom.md
```

Download your FY V2MOM PDF and paste the content in. The more complete it is (all methods, all measures), the better the Monday brief's V2MOM pulse will be.

Format:
```markdown
# FY27 V2MOM — [Your Name]

## Vision
[Paste vision text]

## Methods & Measures

### M1: [Method title]
[Method description]
- 1.1 — [Measure] | Progress: [%]
- 1.2 — [Measure] | Progress: [%]

### M2: [Method title]
...
```

### 2.4 Agent Memory Files

These start empty and get populated automatically as agents run. Create them:

```bash
for agent in admin comms pm rnd career; do
  touch ~/.claude/memory/agents/${agent}-memory.md
done
```

---

## Section 3 — CLAUDE.md Configuration

`CLAUDE.md` is the master configuration file. Create it in your primary working directory (e.g., `~/`).

```markdown
# Team Operating Context

## About [Your Name]
- Name: [Name]
- Role: [Your role]
- Timezone: America/[Timezone]
- Working style: Async-first. Push only what needs a decision.

## Current Clients & Projects
1. **[Client]** — [Status]. [Brief description of engagement].
   Slack: #client-channel-1, #client-channel-2

## Current Priorities
1. [Priority 1]
2. [Priority 2]

## Brief Schedule
- **Monday:** Planning Brief only — forward-looking, V2MOM-anchored, covers full prior week
- **Tue–Sun:** Daily Brief

## Tool Integrations
- Primary comms: salesforce.enterprise.slack.com (MCP connected)
- Email: Gmail (MCP connected)
- Calendar: Google Calendar (MCP connected)
- Briefing channel: #yourname-ai-team (private, Slack channel ID: [YOUR_CHANNEL_ID])

## Memory Sources (load at session start)
Always load:
  - ~/.claude/memory/core/relationships.md
  - ~/.claude/memory/core/decisions.md
  - ~/.claude/memory/core/preferences.md
  - ~/.claude/memory/agents/[your-agent-name]-memory.md

Career Agent also always loads:
  - ~/.claude/memory/core/v2moms/your-v2mom.md
  - ~/.claude/memory/core/v2moms/manager-v2mom.md

## Slack Configuration
- Primary workspace: salesforce.enterprise.slack.com
- Briefing channel: #yourname-ai-team

- Key people (always surface their messages):
  - [Manager Name] (@SlackID) = direct manager
  - [Colleague] (@SlackID) = [context]

## Slack Behaviour Rules
- NEVER post to external or customer-facing channels autonomously
- Internal Salesforce channels: draft and flag for approval before posting
- #yourname-ai-team: post freely
- DMs to colleagues: draft only, always surface for approval

## Approval Flow
- Reply `approve` in thread → send as-is
- Reply `edit: [change]` → revise and repost for approval
- Reply `skip` → discard and log reason in episodic memory
- No reply after 4 hours → re-surface in next briefing
```

---

## Section 4 — Agent Definitions

Agents are `.md` files in `~/.claude/agents/`. Each defines an agent's role, tools, and instructions.

Copy the files from `reference/agent-definitions/` and edit them:

```bash
cp reference/agent-definitions/*.md ~/.claude/agents/
```

**Required edits in each file:**

| What to find | What to replace with |
|-------------|---------------------|
| `Dave Harding` | Your name |
| `dharding` | Your username |
| `C0AUAS09ZA6` | Your briefing channel ID |
| `WADPAG4DP` (Armita's ID) | Your manager's Slack user ID |
| AXS team user IDs | Your team's Slack user IDs |
| SWA channel IDs | Your client channel IDs |
| V2MOM file paths | Paths to your V2MOM files |

### The Six Agents

**Orchestrator** (`orchestrator.md`)  
Team lead. Routes work to specialists, synthesises outputs, posts the brief. The only agent you address directly. Knows whether to run a Daily or Monday Planning Brief based on the day.

**Admin** (`admin.md`)  
Reads Google Calendar with `mcp__plugin_google_google__calendar_events` (parameters: `start_date`, `end_date`, `calendars="all"`). Searches prior meeting transcripts with `mcp__plugin_google_google__meeting_notes_search` to populate "prior history" in each meeting brief. Flags scheduling conflicts.

**Comms** (`comms.md`)  
Reads DMs from all key contacts and AXS team members using `mcp__plugin_slack_slack__slack_read_channel` with user_id as channel_id. Reads tiered priority channels. Searches Gmail. Triages: URGENT / ACTIONABLE / INFORMATIONAL. Drafts replies held for approval.

**PM** (`pm.md`)  
Searches meeting transcripts for decisions, blockers, and action items. Scans client Slack channels. Reports per-client status: on track / at risk / blocked.

**R&D** (`rnd.md`)  
Searches internal Slack first, web second. Only surfaces findings traceable to real tool results. Updates rnd-memory.md with SME contacts and validated patterns.

**Career** (`career.md`)  
Reads V2MOM files and career-memory.md. On daily brief: surfaces one actionable career nudge. On Monday planning brief: full V2MOM pulse — every measure rated MOVED / STALE / DEAD with one specific action per gap.

---

## Section 5 — Channel and Contact Configuration

### Finding Slack Channel IDs

In Claude Code:
```
"Find the Slack channel ID for #channel-name"
```

Or search manually: right-click channel → Copy link → ID is the `C...` string at the end.

### Finding Slack User IDs

```
"Find the Slack user ID for [Full Name]"
```

Or: click profile → three dots → Copy member ID.

### Channel Tiering (in comms.md)

Organize your channels into tiers so agents know what to prioritize:

| Tier | Purpose | Read frequency |
|------|---------|---------------|
| Tier 1 | Your team's home channels | Every brief |
| Tier 2 | Ideas, insights, innovations | Every brief — surface standouts |
| Tier 3 | Client + project channels | Every brief — scan for action items |
| Tier 4 | Broadcasts, org-wide | R&D agent, keyword search only |

For AXS/FDE roles, recommended Tier 1 channels:
- `#experience-specialists`
- `#agentforce-fde-experts-all-team`

Recommended Tier 2:
- `#fde-show-and-tell`
- `#ai-club`
- `#af-voice-active-projects-team-knowledge-sharing`
- `#exp-ai-hq`
- `#mastering-agentforce-for-solution-engineers`

---

## Section 6 — Automation Setup

### 6.1 System Cron (Runs Without Claude Code Open)

This is the recommended setup if your Mac stays on overnight:

```bash
# Find your claude binary path
which claude
# → /Users/yourusername/.local/bin/claude

# Open crontab
crontab -e

# Add this line (adjust path if needed)
3 7 * * * /Users/yourusername/.local/bin/claude -p "Run today's brief" --dangerously-skip-permissions >> ~/.claude/logs/brief-cron.log 2>&1
```

The `3 7 * * *` expression means "7:03 AM every day." Adjust the time as needed (use 24-hour format).

**Check if it's working:**
```bash
# After the first morning run, check the log
cat ~/.claude/logs/brief-cron.log
```

**macOS permissions note:** The first time cron fires, macOS may prompt for Full Disk Access. Go to System Settings → Privacy & Security → Full Disk Access → add `cron`.

### 6.2 In-Session Cron (Backup, Requires Claude Code Open)

If you want a backup cron that runs when Claude Code is already open:

In Claude Code:
```
"Set up a recurring cron job that runs 'Run today's brief' every morning at 7:03 AM, durable true"
```

Note: this auto-expires after 7 days and needs renewal. The system cron in 6.1 is more reliable.

---

## Section 7 — Testing Your Setup

Run a brief manually before waiting for tomorrow morning:

```
In Claude Code: "Run today's brief"
```

**Checklist:**

- [ ] Brief appears in your Slack channel within 3 minutes
- [ ] Calendar section shows real events (not fabricated)
- [ ] Meeting briefs reference actual prior transcripts (check admin agent)
- [ ] DMs from at least one key contact are scanned
- [ ] Comms section shows real channel content or "0 results" — not invented messages
- [ ] R&D section shows sources with channel names and usernames, or is omitted
- [ ] Any 🔴 items have threaded drafts
- [ ] Brief is archived to `~/.claude/memory/briefings/archive/[date].md`

**If calendar returns no data:** Check that `mcp__plugin_google_google__calendar_events` is called with `start_date`, `end_date`, and `calendars="all"`. The previous incorrect parameter was `date` — that causes silent failure.

**If Slack returns 0 results everywhere:** The comms agent uses `mcp__plugin_slack_slack__slack_search_public_and_private` for keyword searches and `mcp__plugin_slack_slack__slack_read_channel` with channel IDs for direct reads. Confirm both tool names are spelled correctly in your agent files.

**If DMs return nothing:** Confirm the user_id values in `comms.md` are correct. Test manually: `mcp__plugin_slack_slack__slack_read_channel` with a colleague's user_id as channel_id.

---

## Section 8 — The Monday Planning Brief

The Monday brief is the highest-value output of the system. It differs from the daily brief in three ways:

1. **Looks back** — sweeps the full prior week before looking forward
2. **Sources transcripts first** — reads every Google Meet note from Mon–Sun before touching Slack
3. **V2MOM-anchored** — maps everything that happened to career measures and surfaces gaps

To get the most value from it:
- Populate your V2MOM files completely (all methods, all measures with progress percentages)
- Enable Gemini note-taking in Google Meet so transcripts exist
- Review the Monday brief before your first meeting — it will set your week's priorities better than any to-do list

See `reference/example-planning-brief.md` for a full example of what a Monday brief looks like.

---

## Section 9 — Common Issues and Fixes

**Issue: Agents fabricate messages or content**  
This happens when an agent uses wrong tool names or pseudocode instead of real MCP calls. Every agent definition in this system includes exact MCP tool names and a CRITICAL rule: "If a tool returns no results, report that honestly. Never invent."

If you see an agent making up content: check that the tool names in its `.md` file exactly match the MCP tool names available in your session.

**Issue: Calendar section always empty**  
Wrong parameter name. Must use `start_date` / `end_date`, not `date`. Check `admin.md` for the correct call.

**Issue: DM reads return nothing**  
Two causes: (1) wrong user_id, or (2) `slack_read_channel` being called with channel name instead of ID. For DMs, the channel_id IS the user's user_id. Verify IDs with `slack_search_users`.

**Issue: Monday brief doesn't have much from last week**  
Either meeting transcripts don't exist (Gemini note-taking not enabled) or the V2MOM files are empty. Both are data sources — the more complete they are, the richer the output.

**Issue: Cron fires but brief doesn't appear in Slack**  
Check `~/.claude/logs/brief-cron.log` for errors. Most common: Slack MCP session token expired. Re-authenticate via AI Expert Suite.

**Issue: Claude Code cron expired (7-day limit)**  
The system cron in Section 6.1 doesn't expire. The in-session CronCreate job does expire after 7 days. Renew it with: "Set up a recurring cron job that runs 'Run today's brief' every morning at 7:03 AM, durable true"

---

## Section 10 — Maintenance

### Weekly (5 minutes, Fridays)
- Glance at the weekly career nudge in Monday's brief — did anything actually move?
- If V2MOM progress changed, update the percentage in your V2MOM file

### Monthly (15 minutes)
- Review `~/.claude/memory/agents/` files — are they accumulating useful patterns?
- Archive old episodic logs (keep last 8 weeks)
- Update `relationships.md` with new contacts
- Check cron is still running: `crontab -l`

### When onboarding a new client
- Add them to `CLAUDE.md` Current Clients section
- Add their Slack channel IDs to the Tier 3 channel list in `comms.md`
- Add key contacts (name + Slack user ID) to `relationships.md` and the DM read list in `comms.md`
- Add their PM memory entry to `pm-memory.md`

### When ending a client engagement
- Move them from active to closed in `CLAUDE.md`
- Remove their channels from active monitoring tiers in `comms.md`
- Log the closeout date in `pm-memory.md`

---

## Appendix A: Full MCP Tool Reference

| Tool | Purpose | Key Parameters |
|------|---------|---------------|
| `mcp__plugin_google_google__calendar_events` | Get calendar events | `start_date`, `end_date`, `calendars="all"` |
| `mcp__plugin_google_google__calendar_list` | List all calendars | none |
| `mcp__plugin_google_google__meeting_notes_search` | Find Meet transcripts | `query`, `date_from`, `date_to` |
| `mcp__plugin_google_google__docs_get` | Read a Google Doc | `file_id` |
| `mcp__plugin_google_google__docs_search` | Search Drive | `query` |
| `mcp__plugin_gmail_gmail__gmail_search` | Search Gmail | `query`, `maxResults` |
| `mcp__plugin_gmail_gmail__gmail_get` | Get an email | `id` |
| `mcp__plugin_slack_slack__slack_read_channel` | Read channel or DM history | `channel_id` (use user_id for DMs) |
| `mcp__plugin_slack_slack__slack_read_thread` | Read a thread | `channel_id`, `message_ts` |
| `mcp__plugin_slack_slack__slack_search_public_and_private` | Search messages | `query`, `count` |
| `mcp__plugin_slack_slack__slack_search_channels` | Find channel IDs | `query` |
| `mcp__plugin_slack_slack__slack_search_users` | Find user IDs | `query` |
| `mcp__plugin_slack_slack__slack_send_message` | Post to Slack | `channel_id`, `message`, `thread_ts` |

---

## Appendix B: Approval Flow Reference

All decisions surface as threaded messages in your agent channel. Reply inline:

| Reply | What happens |
|-------|-------------|
| `approve` | Draft is sent as-is |
| `edit: [your change]` | Draft is revised, reposted for review |
| `skip` | Discarded, reason logged to episodic memory |
| *(no reply after 4h)* | Re-surfaced in next briefing |

---

## Appendix C: What's Different from v1.0

| Area | v1.0 | v2.0 |
|------|------|------|
| Data sources | Slack search + calendar | + Google Meet transcripts + DMs |
| Calendar tool | Broken (wrong params) | Fixed: `start_date`/`end_date`/`calendars` |
| Comms | Channel keyword search | + Direct DM reads from key contacts |
| Fabrication risk | Not explicitly guarded | CRITICAL rule in every agent + exact tool names |
| Monday brief | Not implemented | Full V2MOM-anchored weekly planning |
| Channel config | List in CLAUDE.md | Tiered with verified IDs in comms.md |
| Automation | CronCreate only (7-day expiry) | System cron (permanent, no Claude Code needed) |
| V2MOM tracking | Weekly career check-in | Every measure rated MOVED/STALE/DEAD weekly |

---

*Built by Dave Harding, AXS / FDE | May 2026*  
*Built with Claude Code + AI Expert Suite*  
*Questions: #experience-specialists or DM @Dave Harding*
