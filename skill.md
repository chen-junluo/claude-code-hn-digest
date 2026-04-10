---
name: hn-digest
description: Fetch Hacker News front page, identify what people care about most, explain it in plain language for business school PhD students with limited technical background, optionally output in Chinese or English, and save a dated markdown archive.
tools: Bash, Write, AskUserQuestion
---

# HN Digest

Use this skill when the user wants a fast, archived summary of the current Hacker News front page.

## When to use

Trigger this skill when the user asks to:
- summarize Hacker News
- fetch today's Hacker News front page
- explain what Hacker News users care about most
- save an HN digest as markdown
- create a daily or one-off HN archive

Examples:
- `/hn-digest`
- `Summarize Hacker News and save an archive.`
- `Fetch HN, tell me what people care about most, and save a markdown digest.`
- `Give me a business-school-friendly summary of today's Hacker News.`

## Core task

When invoked, do the following in order:

1. Ask the user whether they want the digest in Chinese or English if they did not already specify the language.
2. Fetch `https://news.ycombinator.com/`.
3. Extract the front-page stories.
4. Identify what appears to have the strongest audience attention.
5. Write a plain-language summary for business school PhD students with limited technical background in the user's chosen language.
6. Save the result as a timestamped markdown file.
7. Return the saved file path.

## Language selection

If the user clearly asked for Chinese, write the digest in Chinese.
If the user clearly asked for English, write the digest in English.

If the user did not specify a language, ask a short question before drafting:
- Chinese
- English

Use `AskUserQuestion` for this choice.

The saved markdown should match the chosen language throughout, including headings, bullets, and final response.

## Fetch and extraction rules

Do not rely on `WebFetch` for Hacker News.
Use `Bash` with a small local Python script to fetch `https://news.ycombinator.com/` and parse the front page HTML.

The script should:
- use Python 3
- prefer only standard-library modules such as `urllib.request`, `html.parser`, `json`, `re`, and `html`
- send a normal browser-like user agent
- fetch the page HTML
- parse the HN front page by pairing each `athing` row with its following subtext row
- extract as much of the following as is visible on the front page:
  - rank
  - title
  - outbound URL
  - source domain
  - points
  - comment count
  - submitter
- keep job posts or other items with missing fields, but mark those fields as missing
- build a structured in-memory list of story records before writing the markdown

Implementation preference:
- prefer `html.parser.HTMLParser` or another standard-library parsing approach
- avoid adding third-party Python dependencies
- if lightweight regex is used, use it only for small field extraction tasks such as pulling numbers from strings like `123 points` or `45 comments`
- print or otherwise surface the structured records first, then draft the markdown from that parsed data

The parsing step should treat the extracted story list as the source of truth for the later summary.

Recommended Python skeleton:

```python
import json
import re
import urllib.request
from html import unescape
from html.parser import HTMLParser

URL = "https://news.ycombinator.com/"
USER_AGENT = "Mozilla/5.0"

class HNParser(HTMLParser):
    def __init__(self):
        super().__init__()
        self.rows = []
        self.current_story = None
        self.pending_subtext = None
        self.in_titleline = False
        self.in_subtext = False
        self.capture_rank = False
        self.capture_title = False
        self.capture_domain = False
        self.capture_points = False
        self.capture_user = False
        self.capture_comments = False

    def handle_starttag(self, tag, attrs):
        attrs = dict(attrs)
        cls = attrs.get("class", "")

        if tag == "tr" and "athing" in cls:
            self.current_story = {
                "rank": None,
                "title": None,
                "url": None,
                "domain": None,
                "points": None,
                "submitter": None,
                "comments": None,
            }
            self.rows.append(self.current_story)
        elif self.current_story is not None and tag == "span" and cls == "rank":
            self.capture_rank = True
        elif self.current_story is not None and tag == "span" and cls == "titleline":
            self.in_titleline = True
        elif self.current_story is not None and self.in_titleline and tag == "a" and "href" in attrs and self.current_story["title"] is None:
            self.current_story["url"] = attrs["href"]
            self.capture_title = True
        elif self.current_story is not None and self.in_titleline and tag == "span" and cls == "sitestr":
            self.capture_domain = True
        elif tag == "td" and cls == "subtext" and self.rows:
            self.pending_subtext = self.rows[-1]
            self.in_subtext = True
        elif self.in_subtext and tag == "span" and cls == "score":
            self.capture_points = True
        elif self.in_subtext and tag == "a" and attrs.get("class") == "hnuser":
            self.capture_user = True
        elif self.in_subtext and tag == "a" and attrs.get("href", "").startswith("item?id="):
            self.capture_comments = True

    def handle_endtag(self, tag):
        if tag == "span":
            self.capture_rank = False
            self.capture_domain = False
            self.capture_points = False
            if self.in_titleline:
                self.in_titleline = False
        if tag == "a":
            self.capture_title = False
            self.capture_user = False
            self.capture_comments = False
        if tag == "td" and self.in_subtext:
            self.in_subtext = False
            self.pending_subtext = None

    def handle_data(self, data):
        text = unescape(data).strip()
        if not text:
            return
        if self.capture_rank and self.current_story is not None:
            self.current_story["rank"] = text.rstrip(".")
        elif self.capture_title and self.current_story is not None:
            self.current_story["title"] = (self.current_story["title"] or "") + text
        elif self.capture_domain and self.current_story is not None:
            self.current_story["domain"] = text
        elif self.capture_points and self.pending_subtext is not None:
            m = re.search(r"(\d+)", text)
            if m:
                self.pending_subtext["points"] = int(m.group(1))
        elif self.capture_user and self.pending_subtext is not None:
            self.pending_subtext["submitter"] = text
        elif self.capture_comments and self.pending_subtext is not None:
            m = re.search(r"(\d+)\s*comment", text)
            if m:
                self.pending_subtext["comments"] = int(m.group(1))
            elif "discuss" in text.lower():
                self.pending_subtext["comments"] = 0

req = urllib.request.Request(URL, headers={"User-Agent": USER_AGENT})
with urllib.request.urlopen(req, timeout=20) as response:
    html = response.read().decode("utf-8", errors="replace")

parser = HNParser()
parser.feed(html)
stories = [row for row in parser.rows if row.get("title")]

print(json.dumps(stories, ensure_ascii=False, indent=2))
```

This skeleton is a starting point, not a rigid requirement. Adjust only if the current HN HTML structure demands it.

Aim to capture the top 20 to 30 stories if available.

If some fields are missing, say they are missing. Do not invent values.

If the fetch result is incomplete, continue with the best visible data and state the limitation in the final markdown.

## How to decide what people care about most

Create a section named exactly:

`## What people care about most`

Use visible Hacker News attention signals, especially:
- comment count
- points
- story rank

Do not use rank alone if comments or points suggest a different pattern.

For each item in this section:
- name the story or topic clearly
- give one short evidence line using the visible HN signals
- explain in plain language why it likely drew attention
- explain why a business school PhD student may care

If points or comments are unavailable for some or all stories, say so directly.

## Writing rules

Write for smart readers with limited technical background.

Follow these rules:
- use plain language in the chosen output language
- avoid jargon when possible
- when a story is technical, explain the practical meaning first
- focus on business relevance, organizational implications, labor, markets, incentives, regulation, AI adoption, entrepreneurship, or research opportunities when relevant
- do not overclaim from headlines alone
- keep sentences fairly short
- prefer concrete explanation over hype

If the output language is Chinese:
- write natural Chinese, not translation-like Chinese
- explain technical topics in everyday terms first
- keep headings and bullets in Chinese

If the output language is English:
- keep the current plain-English style

## Required markdown structure

Save a markdown file with this structure when the output language is English:

```md
# Hacker News Digest — YYYY-MM-DD HH:MM

Source: https://news.ycombinator.com/
Fetched at: YYYY-MM-DD HH:MM local time

## What people care about most

1. **Story or topic**
   - Evidence: X points, Y comments, rank Z.
   - Why it drew attention: ...
   - Why a business school PhD student may care: ...

## Executive summary

- ...
- ...
- ...

## Stories worth reading

### 1. Story title
- Link: ...
- Source domain: ...
- HN signal: X points, Y comments, rank Z
- Plain-English summary: ...
- Why it matters: ...
- Research angle: ...

## Patterns across the front page

- AI and automation: ...
- Startups, product strategy, and competition: ...
- Labor, organizations, and productivity: ...
- Regulation, platforms, and markets: ...

## Caveats

- This digest is a snapshot of the HN front page at one point in time.
- HN comments and points are attention signals, not proof that a claim is true or important.
- Some stories may require reading the original link before using them in research or teaching.
```

You may omit a pattern bullet if it is not supported by the stories you extracted.

If the output language is Chinese, keep the same information structure but translate the headings and bullets into natural Chinese. The Chinese version must still include a section that clearly marks what people care about most.

Recommended Chinese headings:
- `# Hacker News 摘要 — YYYY-MM-DD HH:MM`
- `## 大家最关注什么`
- `## 摘要`
- `## 值得一读的内容`
- `## 首页上的几个共同主题`
- `## 说明与局限`

## Archive rules

Save the markdown archive in the user's local Downloads folder:

`~/Downloads/hn-digest/`

Use a timestamped filename in this format:

`hacker-news-digest-YYYY-MM-DD-HHMM.md`

Before saving:
- create the directory if it does not exist
- generate the current local timestamp
- expand `~` to the real local home directory before writing

Use `Bash` for:
- simple filesystem setup
- timestamp generation
- the local Python fetch-and-parse step

Use `Write` for the final markdown file.

## Final response

After saving the file, respond with:
- a one-line summary of the digest
- the archive path
- 2 to 3 bullets on the strongest themes you found

## Boundaries

This skill is for quick daily digestion and archiving.
It is not a full news crawler, not a historical database, and not a substitute for reading the source articles.
