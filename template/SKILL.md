---
name: psn-yourskill
description: >-
  Use when [the specific situations, symptoms, and phrases that should trigger
  this skill]. Pack the words someone would actually type when they want this.
  Describe ONLY when to use it, never summarize the workflow inside. Keep the
  whole frontmatter under 1024 characters. End by naming what this skill is NOT
  for, pointing at the sibling skill that covers it.
compatibility: State what the skill needs (which psnstats run and flags, which export files in ./psn-export/).
---

# PSN Yourskill

One or two sentences: what this skill covers, and that it reads the export
`psn-export` produces (never PSN directly). Cross-reference sibling skills by
name (`psn-export`, `psn-taste-profile`, `psn-backlog-triage`,
`psn-wishlist-advisor`) rather than repeating their content.

## Guidelines for a good PSN skill

Delete this section in your real skill; it's guidance for authoring.

- **Name:** `psn-<area>`, lowercase and hyphens only. The directory name MUST
  equal the frontmatter `name`.
- **Description:** starts with "Use when…", lists concrete triggers and
  keywords, ends with a "not for X, use Y" pointer. No workflow summary (an
  agent follows the description instead of reading the body).
- **Markdown only.** This collection ships doctrine and fenced `psnstats`
  command recipes, not scripts. The agent runs the CLI and writes throwaway
  analysis per task; recipes stay honest where scripts rot.
- **Read the export, don't fetch PSN.** Every reader skill works only from the
  files in `./psn-export/`. If the data is missing or stale, route to
  `psn-export`, never invent it.
- **Never invent data.** No title, score, price, or trait that isn't in the
  export. If it isn't in the file, it isn't in the library.
- **Cite the signal.** Every recommendation or verdict names the specific field
  behind it (an enjoyment component, a trait, an abandon flag, a price string).
  "High score" is not a reason.
- **Generic examples only.** Worked examples use the synthetic profile shape,
  never a real npsso, a real account id, or a personal library.
- Keep `SKILL.md` under ~500 lines; push heavy detail into `references/*.md`.

## Overview

What this is and the core principle in 1-2 sentences.

## When to use

Symptoms and situations (bullets). When NOT to use.

## [Your sections]

The fields you read, the logic you apply, one worked example from the export
shape, and the specific traps. One excellent example beats five generic ones.
