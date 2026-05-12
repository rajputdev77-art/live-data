# live-data

Public status feed for Dev Rajput's automation projects. Each project's local script writes a small JSON status file here after every run. The Vercel dashboards fetch these JSONs over plain HTTP and display the data.

This is a "git-as-database" pattern: no servers, no cloud DB, no signups, no monthly bills. The local automations push, the public dashboards pull. Cadence: minutes-to-hours, never real-time.

## Layout

```
/soul-in-motion
  latest.json    # totals (blogs, videos, reels, shorts), streak, today's published items
  history.jsonl  # append-only log: one line per publish event
/youtube-pipeline
  latest.json    # last run status, total videos, channel stats, next scheduled run
  history.jsonl  # append-only log: one line per pipeline run
/trading
  latest.json    # account state, open positions, recent LLM decisions
  history.jsonl  # append-only log: closed positions + decision rationales
```

## How dashboards consume this

```
fetch('https://raw.githubusercontent.com/rajputdev77-art/live-data/main/soul-in-motion/latest.json')
```

Public files, no auth, cache-busted by Next.js `revalidate: 60`.

## How automations write to this

Each project has a `publish_status.py` (or equivalent) that:

1. Builds the JSON payload from project state
2. `git pull --rebase` to absorb concurrent writes from other projects
3. Writes `latest.json` (overwrites) and appends one line to `history.jsonl`
4. `git commit -am "data: <project> update"`
5. `git push`

Failures are logged and never block the parent automation — status is observability, not a critical path.

## Schemas

### soul-in-motion/latest.json
```json
{
  "updated_at": "ISO-8601 UTC",
  "totals": {
    "blogs": 0,
    "videos": 0,
    "reels": 0,
    "shorts": 0,
    "platforms_active": 9
  },
  "streak_days": 0,
  "last_publish_at": "ISO-8601 UTC",
  "today": {
    "date": "YYYY-MM-DD",
    "entries_processed": 0,
    "published": [
      { "platform": "linkedin", "url": "...", "type": "post", "at": "ISO-8601" }
    ]
  },
  "health": {
    "watcher_running": true,
    "last_error": null,
    "checked_at": "ISO-8601"
  }
}
```

### youtube-pipeline/latest.json
```json
{
  "updated_at": "ISO-8601 UTC",
  "channel": {
    "id": "UC...",
    "title": "AI News Daily",
    "subscriber_count": 0,
    "video_count": 0,
    "view_count": 0
  },
  "last_run": {
    "started_at": "ISO-8601 UTC",
    "finished_at": "ISO-8601 UTC",
    "status": "success | failure",
    "video_url": "https://youtube.com/watch?v=...",
    "video_title": "...",
    "duration_seconds": 0,
    "stage_timings": {}
  },
  "next_run_at": "ISO-8601 UTC",
  "schedule": {
    "monday": "14:00 IST",
    "tuesday": "14:00 IST",
    "wednesday": "15:30 IST",
    "thursday": "15:30 IST",
    "friday": "15:30 IST",
    "saturday": "17:00 IST",
    "sunday": "17:00 IST"
  },
  "totals": {
    "runs_attempted": 0,
    "runs_succeeded": 0,
    "videos_published": 0
  }
}
```

### trading/latest.json
```json
{
  "updated_at": "ISO-8601 UTC",
  "mode": "paper",
  "account": {
    "balance_usd": 0,
    "total_value_usd": 0,
    "pnl_usd": 0,
    "pnl_pct": 0
  },
  "open_positions": [
    { "asset": "BTC", "side": "long", "size": 0, "entry": 0, "tp": 0, "sl": 0, "unrealized_pnl_pct": 0 }
  ],
  "last_decision": {
    "at": "ISO-8601 UTC",
    "asset": "BTC",
    "action": "buy | sell | hold",
    "rationale": "..."
  }
}
```

## Privacy

No PII. No credentials. No journal text (Soul in Motion publishes URLs to public platforms, not the raw entries). No live wallet keys (Trading is paper-mode only). Safe to keep public.
