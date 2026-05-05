# Heartbeat — Poll Better Stack for New Incidents

The heartbeat fires every 2 minutes. There is no payload to
parse — your job is to find new Better Stack incidents since the
last sweep, post an audit-ready timeline for each, and reply to
the original post when a previously-tracked incident closes.

## What it does

1. Read `MEMORY.md` for `last_seen_incident_id`,
   `last_seen_started_at`, and the `open_incidents` map.
2. Use `betterstack-mcp` to list incidents sorted by
   `started_at` descending. Take any with `started_at` newer than
   the watermark, capped at 5 per fire.
3. For each new incident (oldest first), follow SOUL **Phase 2**
   — pull the timeline, severity, affected service, postmortem
   owner (with `DEFAULT_POSTMORTEM_OWNER` fallback), and the
   evidence pointers.
4. Compose the audit-ready post per SOUL **Phase 3** — Slack
   `mrkdwn`, under 2,500 chars, raw ISO timestamps preserved,
   timeline capped to the 10 most relevant events, every evidence
   pointer linked.
5. Resolve target channels per the SOUL **Where to post** rules
   and post one message per channel per incident. If a post fails
   for a particular channel, log and continue with the others —
   do not retry.
6. Record each `(channel, slack_message_ts)` in `MEMORY.md`
   under `open_incidents[<incident_id>]` so the resolution reply
   can find them. Update `last_seen_incident_id` and
   `last_seen_started_at` to the newest incident processed.
7. **Resolution sweep**: for each incident in `open_incidents`,
   re-fetch its current state from `betterstack-mcp`. If it is
   now `resolved`, reply to each recorded `(channel,
   slack_message_ts)` per SOUL **Phase 5** with the duration and
   resolution. Remove the incident from `open_incidents` after
   the reply posts.

## MEMORY.md state shape

The agent persists a small block in MEMORY.md to track what's
been processed. Shape:

```
## betterstack-incident-audit

last_seen_incident_id: 12345
last_seen_started_at: 2026-05-05T17:42:00Z

open_incidents:
  12345:
    - channel: C0123ABCD
      slack_message_ts: 1714928520.123456
    - channel: C0987WXYZ
      slack_message_ts: 1714928520.654321
  12346:
    - channel: C0123ABCD
      slack_message_ts: 1714929000.111111
```

Update the watermark in place each fire. Append to and trim from
`open_incidents` as incidents are posted and resolved — do not
let it grow unbounded.

## Where to post

Per SOUL **Where to post**:

- **New incident timelines**: post to every Slack channel the bot
  is a member of, one message per incident per channel. If the
  bot is in zero channels, DM the workspace install user with the
  timeline and a one-line invite hint.
- **Resolution updates**: reply to the original incident post
  using the recorded `slack_message_ts` for each channel. Never
  start a new top-level post for a resolution.
- Never hard-code a channel name.

## Skip conditions

Skip posting (and stop silently) if any of these are true:

- This is the first run after deploy. Seed the watermark to the
  most-recent existing incident and stop — the backlog of
  past incidents is not posted.
- Zero incidents have `started_at` newer than the watermark AND
  no tracked open incident has flipped to `resolved`. Stay
  silent — quiet runs are normal at 2m cadence.
- Better Stack is unreachable or returns an error. Log and wait
  for the next heartbeat.

## Hard rules

- Never post the same incident twice. The watermark + the
  `open_incidents` map are the only de-dup gates; respect them.
- Never relativize a timestamp in the audit post. Raw ISO only.
- Your turn ends after the posts and the `MEMORY.md` update. No
  follow-ups, no thread replies after the initial post (except
  the resolution reply, which is its own scheduled step).
