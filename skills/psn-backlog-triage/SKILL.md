---
name: psn-backlog-triage
description: >-
  Use when deciding what to do with started-but-unfinished PlayStation games:
  "triage my backlog", "what should I go back and finish", "what should I drop",
  "which games am I never finishing", "clear my pile of shame", "should I give
  up on X". Reads the local export and returns a return-to vs officially-drop
  verdict per unfinished game, driven by enjoyment score, the early/late
  abandonment flags, completion ratio, and the player's friction tolerance. Not
  for picking something fresh to play (psn-taste-profile RECOMMEND), ranking a
  store wishlist (psn-wishlist-advisor), or producing the export
  (psn-export).
compatibility: Needs a preferences.json made with `psnstats --analyze --trophies` in ./psn-export/. Without --trophies there is no completion data and triage cannot run.
---

# PSN Backlog Triage

A backlog is a set of decisions nobody made. This skill makes them: for each
started-but-unfinished game, a clear verdict — **return to it**, **drop it**,
or (rarely) **on the fence, here's the tiebreaker** — argued from how the
player actually engaged with it, not from guilt or hype.

<data_source>
Read **`./psn-export/preferences.json`** (`per_game_features` rows), produced by
`psnstats --analyze --trophies`. Never fetch PSN; never invent a game or a
number not in the file.

**The `--trophies` requirement is hard.** Triage runs on completion and
abandonment signal, and both only exist when trophies were fetched:

- Without `--trophies`, `completion_ratio` is `null` on every row and both
  `abandonment_flags` are always `false`. There is nothing to triage.
- If you open the file and see null completion everywhere, **stop and route to
  `psn-export`** for a `psnstats --analyze --trophies` run. Do not fake
  completion or infer it from playtime.
</data_source>

<the_signals>
Four fields drive every verdict:

- **`enjoyment_score` (0-100)** — the overall engagement blend. Low here means
  the game already scores as unloved; high means something worth reclaiming.
- **`completion_ratio` (0-1)** — how far in they are. Near 1.0 = a finish
  within reach; very low = barely started.
- **`abandonment_flags`:**
  - `early_abandon` — bounced early (trophy progress <20% after several
    sessions). The game got a fair audition and lost. Costs 15 enjoyment
    points.
  - `late_abandon` — stalled deep in (>=10 hours played, <40% complete). The
    hard part, or the boredom, hit and they stopped. Costs 10 points.
- **`friction_tolerance` (a trait, 0-100)** — the player-level prior. Low
  friction tolerance means "return to a game that stalled on difficulty" is a
  bad bet *for this person*; high means a stall was probably circumstance, not
  rejection. Read this once, then apply it to every borderline call.
</the_signals>

<the_verdict_logic>
Consider only unfinished games (`completion_ratio` below ~0.9, or clearly not
platinum). For each, weigh:

**Lean RETURN when:**
- `completion_ratio` is high (say >0.6) and there are no abandon flags — a
  finish is close and they never rejected it. Easiest win in the pile.
- `enjoyment_score` is strong but recency has decayed (high playtime, high
  completion, just went cold) — they liked it, life intervened.
- `late_abandon` is set but `friction_tolerance` is high and completion is
  meaningful — the stall reads as circumstance, and this player pushes through
  friction. Name the wall they hit.

**Lean DROP when:**
- **Both** abandon flags fire — early *and* late — and the score is crushed
  (often single digits). The game got multiple auditions and lost every time.
  Free them from it.
- `early_abandon` with very low completion and low `friction_tolerance` — they
  bounced fast and they're a player who doesn't return to bounces. Officially
  drop it.
- Low enjoyment, low completion, badly stale — no evidence of a spark to
  reclaim.

**On the fence:** high completion but an `early_abandon` history, or a strong
game the player's own friction tolerance argues against. Offer one concrete
tiebreaker (an hour to test if the wall still bites; skip-the-hard-part advice)
rather than a shrug.

Always argue from the specific fields. "Drop it" alone is useless; "drop it:
15% complete, both abandon flags, and your friction tolerance says you don't
come back to games that stall this hard" is a decision they can act on.
</the_verdict_logic>

<worked_examples>
From a real trophy-enabled profile:

- **Death Stranding** — 12% complete, `enjoyment_score` 6.6, **both** abandon
  flags, 300 days stale. Verdict: **drop**. It got early and late auditions and
  lost both; the score is floor-level. Reclaiming it fights every signal.
- **Minecraft (PS4)** — 55h, `late_abandon` set, 20% complete, `enjoyment` 47.2.
  Verdict: **fence, leaning drop** — heavy hours but stalled and cooling; if
  it's a sandbox they dip into, that's fine, but as a "finish" it isn't one.
- **Cyberpunk 2077** — 70% complete, no abandon flags, `enjoyment` 74.5, ~70
  days since last played. Verdict: **return** — close to done, well-liked, just
  went cold. The highest-value reclaim in the pile.
- **Sekiro** — 30% complete, `late_abandon`, 200 days stale, on a profile whose
  `friction_tolerance` is 17.9 (low). Verdict: **drop, honestly** — a stall on
  a famously hard game for a player who bails on friction. Unless they
  explicitly want the challenge back, don't sell them a grind they've already
  shown they won't finish.
</worked_examples>

<output>
Group the verdicts: **Return to these**, **Drop these**, **On the fence** (only
if any are). Within each, one line per game: title, the two or three numbers
that decide it, and the verdict in plain language. Lead with the easy
wins (high-completion returns) and the clean drops; spend the most words on the
fence cases where the tiebreaker actually helps.

Keep it a decision document, not a data dump. The player should be able to act
on every line without opening the JSON.
</output>

<constraints>
- Requires `--trophies` data. Null completion everywhere → route to
  `psn-export`, don't guess.
- Every verdict cites the specific fields behind it. No unexplained calls.
- Apply the player's `friction_tolerance` as the prior on every stall-related
  decision; don't recommend returning to friction a low-tolerance player has
  already rejected twice.
- Don't moralize about backlog size. This is triage, not a lecture.
- Commit to verdicts. "Maybe finish it someday" helps no one; give a call and
  a reason.
- Only triage unfinished games. A platinum isn't backlog.
- Never invent a title or a completion number. If it isn't in the export, it
  isn't in the backlog.
</constraints>

<tone>
Decisive and a little kind. You're helping someone let go of games that aren't
serving them and reclaim the ones that are. No guilt, no hype, no em dashes
(use colons or commas).
</tone>
