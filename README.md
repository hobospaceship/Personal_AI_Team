# AI Agent Team — AXS / COE

> **What is this?** A personal autonomous AI agent team that runs every morning, reads your calendar, meeting transcripts, Slack DMs, and priority channels, and delivers a structured briefing to a private Slack channel — so you can start each day knowing exactly what needs your attention and what you should focus on to advance your career goals.

---

## The Problem It Solves

As an FDE or AXS specialist you're juggling multiple clients, a firehose of Slack channels, meeting transcripts you never get back to, and V2MOM goals that get buried under daily execution work. Every morning costs 20-30 minutes just getting oriented. Great ideas shared in DMs or #fde-show-and-tell disappear into the archive. Career opportunities scroll past unnoticed.

This system fixes all of that. It runs while you sleep.

---

## What You Get

### Daily Brief (Tue–Sun, 7 AM)
A single Slack message in your private agent channel with:
- **Today's meetings** with pre-written context briefs drawn from prior meeting transcripts
- **Priority decisions** that need your approval (with drafts ready)
- **Comms** — actionable messages from DMs and key channels, nothing invented
- **R&D intel** — verified Slack findings from practitioner channels relevant to your active client work
- **Project pulse** — per-client status in one line
- **Career nudge** — one actionable move to advance a V2MOM measure

### Monday Planning Brief
Replaces the daily brief. Sources the full prior week — calendar, meeting transcripts, Slack, DMs — and produces:
- What actually moved last week (from transcripts + Slack, not memory)
- Carry-forward decisions still unresolved
- This week's 3-5 focus areas, ranked by V2MOM impact + client urgency
- Calendar preview with meeting briefs
- Opportunities to engage (channels, people, events) ranked by career signal
- Full V2MOM pulse: which measures moved, which are stale, one action per gap

---

## How It Works

```
7:03 AM → system cron fires → claude -p "Run today's brief"
                                          ↓
                              Orchestrator reads the day
                                          ↓
              ┌──────────┬──────────┬──────────┬──────────┬──────────┐
              │  Admin   │  Comms   │    PM    │   R&D    │  Career  │
              │ Calendar │ DMs +    │ Meeting  │ Slack    │ V2MOM    │
              │ + notes  │ channels │ notes +  │ intel    │ tracking │
              │          │          │ Slack    │          │          │
              └──────────┴──────────┴──────────┴──────────┴──────────┘
                                          ↓
                         Synthesized brief → #your-ai-team
```

**Data sources used:**
| Source | What agents read |
|--------|-----------------|
| Google Calendar | Today + tomorrow's events |
| Google Meet transcripts | Prior meeting notes (via Gemini) for every meeting brief |
| Slack DMs | All key contacts + AXS team members |
| Slack channels | Tiered — AXS home channels, insight channels, client channels |
| Gmail | External unread email |
| Memory files | Decisions, relationships, V2MOMs, prior agent notes |

**Nothing is invented.** If a tool returns no results, the agent says so. All Slack searches use verified MCP tool names. All calendar events come from the Google Calendar API.

---

## What You Need to Set It Up

**Required:**
- Claude Code CLI (with Pro or Max subscription)
- AI Expert Suite (for Google Calendar, Gmail, Google Meet, Slack MCP connections)
- A private Slack channel (your agent working surface)

**Time to set up:** ~2 hours  
**Ongoing maintenance:** ~10 min/week

---

## Files in This Folder

| File | What it is |
|------|-----------|
| `SETUP_GUIDE_v2.md` | Full step-by-step blueprint to reproduce this system |
| `QUICK_START.md` | One-page overview for sharing with teammates |
| `reference/agent-definitions/` | The actual agent .md files powering this system |
| `reference/example-daily-brief.md` | Real example of a daily brief output |
| `reference/example-planning-brief.md` | Real example of a Monday planning brief |

---

## Key Design Decisions (and Why)

**Why a private Slack channel, not Claude Code directly?**  
The brief posts to Slack so it's waiting for you when you open your phone or laptop. You don't need Claude Code running to see it. Approval replies (`approve`, `edit:`, `skip`) work inline in thread.

**Why separate specialist agents instead of one prompt?**  
Each agent has a narrow domain and deep context for that domain. A single prompt trying to do everything produces shallow, generic output. Parallel agents run simultaneously and produce richer, more accurate results.

**Why meeting transcripts as a primary source?**  
Slack searches miss a huge amount of context — decisions made verbally, client feedback given in calls, action items agreed in meetings. Google Meet Gemini transcripts capture all of that. Making them the PM and Admin agents' primary source dramatically improves the quality of prior-meeting context and project status.

**Why V2MOM-anchored Monday planning?**  
FDEs and AXS specialists do a lot of execution work that matters but never gets connected to career measures. The Monday brief explicitly maps last week's work to V2MOM methods and surfaces which measures are going stale — so you don't arrive at your mid-year check-in with gaps you never noticed building up.

---

*Built by Dave Harding, AXS / FDE | May 2026*  
*Questions: #experience-specialists or DM @Dave Harding*
