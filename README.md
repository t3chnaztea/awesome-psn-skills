<!-- awesome PSN API skills for Claude Code -->

<p align="center">
  <img src="./media/hero.png" alt="awesome-psn-skills: agent skills for reading your PlayStation taste" width="840">
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-00e5ff" alt="MIT license"></a>
  <a href="https://github.com/t3chnaztea/awesome-psn-skills/releases"><img src="https://img.shields.io/github/v/release/t3chnaztea/awesome-psn-skills?color=ff2e97" alt="Latest release"></a>
  <img src="https://img.shields.io/badge/skills-4-ffcc00" alt="4 skills">
  <img src="https://img.shields.io/badge/markdown-only-8a2be2" alt="Markdown only">
  <img src="https://img.shields.io/badge/Claude%20Code-plugin-d97706" alt="Claude Code plugin">
</p>

<p align="center">
  <b>Claude Code agent skills for your PlayStation play history: read your taste, triage your backlog, rank your wishlist.</b>
</p>

> [`awesome-psnstats`](https://github.com/t3chnaztea/awesome-psnstats) exports your PlayStation play history and builds an LLM-ready taste profile (`preferences.json`, `library.csv`, `wishlist.json`). Its README ships a few starter prompts, and those prompts want to be a proper skills layer. This repo is that layer, packaged as [Agent Skills](https://agentskills.io): focused, model-readable guides your coding agent (Claude Code and other harnesses) loads on demand to produce the export, read your taste, triage your backlog, and rank your wishlist. The doctrine is what keeps the agent honest: rank by the real scoring formula, never invent a title, and route back to a fresh export when the data is stale.

```
/plugin marketplace add t3chnaztea/awesome-psn-skills
/plugin install psn@t3chnaztea-psn
```

## The four skills

Each is a self-contained home for one job. **Start with `psn-export`**: the
other three all read the export it produces and never touch PSN themselves.

<table>
  <tr>
    <td align="center" width="50%" valign="top">
      <a href="skills/psn-export/SKILL.md"><b>📤 psn-export</b></a><br />
      <sub>Start here. Install <code>psnstats</code>, acquire and rotate the npsso token safely, pick the flags for the goal (<code>--analyze</code>, <code>--trophies</code>, <code>--wishlist</code>, <code>--compare</code>), and stay under the rate limit. Every other skill assumes its output.</sub>
    </td>
    <td align="center" width="50%" valign="top">
      <a href="skills/psn-taste-profile/SKILL.md"><b>🍷 psn-taste-profile</b></a><br />
      <sub>The curator. Reads <code>preferences.json</code>: the enjoyment formula, five traits, and agent-features. Produces a palette read and recommendations that cite the exact signal behind each pick, plus <code>--compare</code> drift reads.</sub>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%" valign="top">
      <a href="skills/psn-backlog-triage/SKILL.md"><b>🗂️ psn-backlog-triage</b></a><br />
      <sub>Return-to vs officially-drop verdicts on started-but-unfinished games, from enjoyment score, the early/late abandon flags, completion ratio, and your friction tolerance. Needs a <code>--trophies</code> run and says so.</sub>
    </td>
    <td align="center" width="50%" valign="top">
      <a href="skills/psn-wishlist-advisor/SKILL.md"><b>🛒 psn-wishlist-advisor</b></a><br />
      <sub>Ranks <code>wishlist.json</code> by what you'll actually play <b>and finish</b>, not hype. Flags avoid-signal matches, makes buy-next and quietly-remove calls, and factors current discounts into timing.</sub>
    </td>
  </tr>
</table>

---

## Why skills, not just pasting the JSON into a chat?

Fair question. `psnstats` writes `preferences.json` to be pasted straight into
an LLM, and for a one-off "what should I play" that works. **If that's all you
want, paste away.** You don't need this repo for it.

These skills exist for the job the raw paste can't reach: **getting the reading
right, every time.** `preferences.json` is a scored artifact with real
semantics, and an agent with no doctrine will misread it.

| Task | Raw paste | These skills |
|------|:---:|:---:|
| Sort games by a number in the file | ✅ | ✅ |
| Explain a score from its *components* (log-playtime, 90-day recency, completion, replay) | ❌ | ✅ |
| Know that a bare `completionist_bias: 50` means "no trophy data", not "average" | ❌ | ✅ |
| Refuse to invent a title or score that isn't in the export | ❌ | ✅ |
| Route back to a `--trophies` run when backlog triage needs completion | ❌ | ✅ |
| Rank a wishlist by finish-likelihood against your avoid-signals | ❌ | ✅ |

They compose: `psn-export` keeps the data fresh, the other three read it the way
it was meant to be read.

---

## ⚠️ Read before you install

**These skills direct an agent to run the `psnstats` CLI and handle your npsso
token.** The npsso is a session cookie equivalent to your PlayStation password.

- **Review the skills before installing.** They're plain Markdown; read what
  they'll have your agent do. Nothing here phones home or auto-runs; they're
  reference guides. `psnstats` sends your token only to Sony's own endpoints.
- **The npsso is a password.** Anyone holding it can act as you on PSN. The
  skills keep it in a `chmod 600` file or an env var, never in a committed file
  or a chat transcript, and it expires on its own in ~60 days.
- **The API is unofficial.** `psnstats` wraps a reverse-engineered PSN API that
  self-limits to 300 requests / 15 minutes. Occasional runs are low-risk; the
  doctrine is "don't hammer it." Your account, your risk.

---

## Install

### Claude Code plugin (recommended)

```
/plugin marketplace add t3chnaztea/awesome-psn-skills
/plugin install psn@t3chnaztea-psn
```

The four skills activate automatically when your prompt matches (e.g. "export
my PSN library", "what should I play next", "triage my backlog", "rank my
wishlist").

### Manual copy (any Claude Code, no marketplace)

```bash
git clone https://github.com/t3chnaztea/awesome-psn-skills
cp -r awesome-psn-skills/skills/psn-* ~/.claude/skills/
```

### Other harnesses

Each `SKILL.md` is harness-neutral Markdown with standard Agent-Skills
frontmatter ([spec](https://agentskills.io/specification)). Drop the
`skills/*` directories wherever your agent framework discovers skills, or
point it at this repo.

---

## What's inside a skill

```
skills/psn-<area>/
  SKILL.md            # the guide (frontmatter + body, < 500 lines)
```

**Markdown only, deliberately.** No scripts to rot, no code to audit before
running near your account. Skills ship doctrine plus fenced `psnstats` command
recipes; your agent runs the CLI and writes whatever throwaway analysis the
task needs, fresh, against your actual export.

Skills are original prose: how to read the export correctly, not a copy of the
`psnstats` docs. The [`awesome-psnstats` README](https://github.com/t3chnaztea/awesome-psnstats)
remains the canonical reference for the CLI itself.

## Contributing

Have a sharper way to read the export, or a new angle on the data? Copy
[`template/SKILL.md`](template/SKILL.md) and follow its authoring notes:
original prose, generic examples (never a real npsso or a personal library),
markdown only, and show the reader the signal behind every call. PRs welcome.

## Companion repos

- [awesome-psnstats](https://github.com/t3chnaztea/awesome-psnstats): the CLI
  these skills drive: the PSN exporter and taste-profile engine.
- [home-assistant-skills](https://github.com/t3chnaztea/home-assistant-skills):
  the same idea for a Home Assistant smart home.
- [batocera-skills](https://github.com/t3chnaztea/batocera-skills): the same
  idea for a Batocera retro-gaming cabinet (and the structural template).

## Versions

Built against `awesome-psnstats` v1.x (`preferences.json` schema 1.0, the
enjoyment/trait/agent-features model). If `psnstats` changes its scoring or
output shape, confirm field names against a fresh export; the export is the
source of truth. When in doubt, read the file, not the docs.

## License

MIT; see [LICENSE](LICENSE).

> Not affiliated with Sony Interactive Entertainment. "PlayStation", "PSN", and
> "PS4"/"PS5" are used here descriptively; this is an independent,
> community-built collection that reads data you export from your own account.

---

Agent skills for reading your PlayStation taste, built on awesome-psnstats.
