# hn-digest

[中文说明](README-zh.md)

A Claude Code skill that fetches the Hacker News front page, shows a full numbered topic list first, lets the user choose which items to inspect more closely with per-item modes, and saves a dated markdown digest locally.

## What changed

The current version adds a two-stage workflow:

- Step 1 fetches the full Hacker News front page and returns a numbered list of every visible item
- each item includes HN signals, a one-line summary, a why-it-matters line, and 2-3 suggested deep-dive angles
- Step 2 runs only after the user selects item numbers and chooses a mode: `academic`, `industry`, or `both`
- different selected items can use different modes in the same run
- targeted extraction now follows six focus types, with opinion synthesis always included first
- the final markdown archive keeps a fixed section order and records whether each selected thread was actually inspected

## What it does

- fetches the current Hacker News front page
- extracts visible story metadata such as rank, title, link, points, comments, submitter, and source domain when available
- returns a full numbered topic list in chat before any deep dive begins
- suggests item-specific deep-dive directions from six focus types
- inspects only the selected discussion threads in one batched pass
- writes a digest in English or Chinese for non-technical academic readers
- saves the result as a timestamped markdown file in `~/Downloads/hn-digest/`

## Modes and focus types

Step 2 supports three modes:

- `academic`: opinion synthesis, facts and quotes, research angle, business implications
- `industry`: opinion synthesis, tech stack and implementation, build ideas
- `both`: all six focus types

The six focus types are:

1. Facts and quotes
2. Opinion synthesis
3. Research angle
4. Business implications
5. Tech stack and implementation
6. Build ideas

## Who it is for

This skill is for people who want a quick, archived Hacker News summary without reading the entire front page. It is especially useful for business school PhD students, researchers, and educators who want help connecting technical news to markets, organizations, labor, regulation, and strategy.

## Files

- `skill.md`: the Claude Code skill definition
- `README.md`: English overview
- `README-zh.md`: Chinese overview
- `LICENSE`: MIT license
- `.gitignore`: ignores local editor and system clutter

## How to use

1. Put this skill in your Claude Code skills directory.
2. Invoke it with a request such as:
   - `/hn-digest`
   - `Summarize Hacker News and save an archive.`
   - `Give me a business-school-friendly summary of today's Hacker News.`
3. If you do not specify a language, the skill asks whether you want Chinese or English.
4. Step 1 returns the full numbered topic list.
5. Reply with the item numbers and mode you want, for example `4 industry; 18 both`.
6. Step 2 fetches the selected threads, writes the final digest, and saves a markdown archive locally.

Before use, turn on a VPN if Hacker News is not reachable from your network.

## Output

The archive is saved to:

`~/Downloads/hn-digest/`

with filenames in this format:

`hacker-news-digest-YYYY-MM-DD-HHMM.md`

The saved digest uses this fixed section order:

1. Title
2. Source
3. Fetched at
4. Part 1: Summary
5. Part 2: Targeted extraction
6. Caveats

## Notes

- This skill is for quick daily digestion and archiving.
- It is not a historical database or a full news crawler.
- Hacker News points and comments are attention signals, not proof that a claim is true or important.
