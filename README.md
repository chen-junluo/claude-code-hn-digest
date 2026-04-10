# hn-digest

[中文说明](README-zh.md)

A Claude Code skill that fetches the Hacker News front page, identifies what people care about most, explains it in plain language for business school PhD students with limited technical background, and saves a dated markdown archive locally.

## What it does

- fetches the current Hacker News front page
- extracts visible story metadata such as rank, title, link, points, comments, and source domain when available
- identifies the topics drawing the most attention on Hacker News
- writes a plain-English or Chinese digest for non-technical academic readers
- saves the result as a timestamped markdown file in `~/Downloads/hn-digest/`

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
4. The skill fetches Hacker News, drafts the digest, and saves a markdown archive locally.

## Output

The archive is saved to:

`~/Downloads/hn-digest/`

with filenames in this format:

`hacker-news-digest-YYYY-MM-DD-HHMM.md`

## Notes

- This skill is for quick daily digestion and archiving.
- It is not a historical database or a full news crawler.
- Hacker News points and comments are attention signals, not proof that a claim is true or important.
