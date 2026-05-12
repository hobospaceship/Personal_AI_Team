# AI Agent Team — Quick Start
*Set up your autonomous daily brief in ~2 hours*

---

## What You're Building

A private Slack channel where a team of AI agents posts every morning at 7 AM:
- Today's meeting briefs (with context from your prior transcripts)
- What needs a decision today (with drafts ready to approve)
- What your key contacts shared in DMs and key channels
- One V2MOM-anchored career nudge
- On Mondays: a full weekly planning brief instead

You interact with it entirely through Slack. Reply `approve`, `edit:`, or `skip` to any draft. Everything else is autonomous.

---

## Prerequisites

- [ ] Claude Code CLI installed and logged in (`claude --version`)
- [ ] AI Expert Suite connected (provides Google Calendar, Gmail, Meet, Slack MCP)
- [ ] A Salesforce Enterprise Slack workspace
- [ ] ~2 hours

---

## Step 1 — Create Your Agent Channel in Slack

Create a private Slack channel: `#yourname-ai-team`  
Add only yourself. This is where all briefs and drafts will land.

Note the channel ID (visible in the channel URL or via Claude Code search).

---

## Step 2 — Install Claude Code and Connect AI Expert Suite

```bash
# Install Claude Code
npm install -g @anthropic-ai/claude-code

# Verify
claude --version
```

In Claude Code, connect AI Expert Suite:
- Opens Google OAuth for Calendar, Gmail, and Meet transcript access
- Opens Slack OAuth for workspace access

Verify connections work:
```
Ask Claude: "What's on my calendar today?"
Ask Claude: "Search Slack for a recent message from [your manager]"
```

---

## Step 3 — Create Your Memory System

Claude Code uses files in `~/.claude/` as persistent memory. Create this structure:

```bash
mkdir -p ~/.claude/memory/core/v2moms
mkdir -p ~/.claude/memory/agents
mkdir -p ~/.claude/memory/episodic
mkdir -p ~/.claude/memory/briefings/archive
mkdir -p ~/.claude/logs
```

Then populate the core files. Templates are in `reference/agent-definitions/` — copy and edit:

**`~/.claude/memory/core/relationships.md`**  
Your clients, manager, colleagues, and their Slack user IDs.

**`~/.claude/memory/core/preferences.md`**  
Your timezone, communication style, working hours.

**`~/.claude/memory/core/decisions.md`**  
Key project and architectural decisions (start with your active clients).

**V2MOMs** — `~/.claude/memory/core/v2moms/your-v2mom.md`  
Paste in your FY V2MOM. Add your manager's V2MOM and at least one level above. These power the career nudge and Monday planning brief.

---

## Step 4 — Create CLAUDE.md

In your working directory, create `CLAUDE.md`. This is the master config file all agents read.

Minimum required sections:
```markdown
## About [Your Name]
- Role: [Your role]
- Timezone: America/[Your timezone]

## Briefing Channel
- #yourname-ai-team (channel ID: [your channel ID])

## Slack Configuration
- Primary workspace: salesforce.enterprise.slack.com
- Key people to always surface: [names + Slack user IDs]
- Priority channels: [list your key channels]

## Brief Schedule
- Monday: Planning Brief (covers full prior week, no daily brief)
- Tue–Sun: Daily Brief

## Approval Rules
- External Slack messages: DRAFT ONLY
- Internal #yourname-ai-team: post freely
- DMs to colleagues: DRAFT ONLY
```

---

## Step 5 — Install the Agent Definitions

Copy all agent `.md` files from `reference/agent-definitions/` to `~/.claude/agents/`:

```bash
cp reference/agent-definitions/*.md ~/.claude/agents/
```

Then edit each file — find and replace `Dave Harding` / `dharding` with your name, and update:
- Your briefing channel ID
- Your key contacts and their Slack user IDs
- Your AXS/team member user IDs
- Your V2MOM file paths

---

## Step 6 — Set Up the Daily Cron

In your Mac terminal:
```bash
crontab -e
```

Add this line (replace path if your `claude` binary is elsewhere):
```
3 7 * * * /Users/yourusername/.local/bin/claude -p "Run today's brief" --dangerously-skip-permissions >> ~/.claude/logs/brief-cron.log 2>&1
```

Save and exit. The brief will now run every morning at 7:03 AM automatically, even if Claude Code is not open — as long as your Mac is on.

**Verify the binary path:**
```bash
which claude
```

---

## Step 7 — Test Manually

Before waiting for tomorrow morning, run a brief now:

```
In Claude Code:
"Run today's brief"
```

Check `#yourname-ai-team` in Slack. You should see the brief appear within 2-3 minutes.

**What to verify:**
- [ ] Calendar events appear (real, not fabricated)
- [ ] Meeting notes section pulls from recent transcripts
- [ ] DMs from key contacts are scanned
- [ ] Priority channels show recent content
- [ ] Brief is posted to your Slack channel
- [ ] Any 🔴 decision items have threaded drafts

---

## Step 8 — Tune It

The first few briefs will show you what's working and what needs adjustment. Common tweaks:

**Too much noise in a channel?** Lower its tier in `comms.md`.

**Missing a key person's DMs?** Add their Slack user ID to the key contacts list in `comms.md`.

**Career nudges not relevant?** Make sure your V2MOM file is fully populated with all methods and measures.

**Calendar not showing?** Check MCP auth and make sure `calendars="all"` is set in admin agent calls.

---

## Approval Flow

When the brief surfaces a 🔴 decision with a draft, reply in thread:

| Reply | What happens |
|-------|-------------|
| `approve` | Draft is sent as-is |
| `edit: [your change]` | Draft is revised and reposted for another review |
| `skip` | Draft is discarded, reason logged to memory |

---

## Monday Planning Brief

On Mondays the brief is replaced by a weekly planning view. It sweeps:
- Every meeting transcript from the prior week
- All Slack DM activity with key contacts
- Priority channel highlights
- Your V2MOM against what actually happened

Output: 3-5 focus areas for the week, ranked by V2MOM impact + client urgency.

To get the most out of it — **make sure Gemini note-taking is enabled in your Google Meet settings**. The transcripts are the highest-signal input the system has.

---

## Files Reference

All the agent definition files you need are in `reference/agent-definitions/`. Each one is a standalone `.md` file that goes in `~/.claude/agents/`. They define what each agent does, what tools it uses, and how it should behave.

The setup guide has the full details: `SETUP_GUIDE_v2.md`

---

*Questions? Post in #experience-specialists or DM Dave Harding*
