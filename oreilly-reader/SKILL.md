---
name: oreilly-reader
description: Use when reading O'Reilly Learning books, extracting chapter content from oreilly.com, synthesizing technical book content via browser automation. Triggers on O'Reilly book URLs, "read this book", "what's in this O'Reilly book", or requests to extract/summarize O'Reilly content.
allowed-tools: mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_evaluate, mcp__plugin_playwright_playwright__browser_snapshot, mcp__plugin_playwright_playwright__browser_click, Bash, Agent
---

# O'Reilly Reader

Read and synthesize O'Reilly Learning books via Playwright browser automation. Requires an active O'Reilly session in the Playwright browser (user must be logged in).

## How It Works

O'Reilly renders chapter content via JavaScript — static HTTP fetch returns a shell. The only reliable extraction path is `document.querySelector('main').innerText` via browser evaluation after the page loads.

## Workflow

### 1. Navigate to book

```
https://learning.oreilly.com/library/view/[slug]/[ISBN]/
```

If given a playlist URL, navigate to the playlist first, extract book titles/links from the snapshot, then pick the target book.

### 2. Extract TOC

After navigating, take a snapshot. The TOC is in the sidebar — look for `listitem` nodes with chapter links like:
```
/library/view/[slug]/[ISBN]/Text/chapter-N.html
```

Collect all chapter URLs from the TOC before reading any content.

### 3. Read chapters

For each chapter URL, navigate then evaluate:

```javascript
() => { return (document.querySelector('main') || document.body).innerText; }
```

Save to a file using the `filename` parameter:
```
filename: chapter-N-text.txt
```

Files save to the current working directory. Navigate → evaluate → save, one chapter at a time.

### 4. Synthesize

After all chapters are saved, spawn a synthesis Agent with all file paths. Instruct it to produce a chapter-by-chapter technical breakdown covering:
- What gets built / implemented
- Key mechanisms with enough depth to understand HOW (not just WHAT)
- Non-obvious insights
- Specific code patterns worth noting

Calibrate depth to the audience — skip beginner framing for technical readers.

## Auth Handling

If navigation redirects to `learning.oreilly.com/member/login/`, the Playwright session is not authenticated. Ask the user to log in via their browser. The Playwright session must share the browser profile with the logged-in session.

## Quick Reference

| Step | Tool | Notes |
|------|------|-------|
| Navigate | `browser_navigate` | Full book URL |
| Get TOC | `browser_snapshot` | Look for chapter links in sidebar |
| Extract text | `browser_evaluate` | `document.querySelector('main').innerText` |
| Save to file | `filename` param on evaluate | Saves to cwd |
| Synthesize | `Agent` (Explore subagent) | Feed all chapter file paths |

## Common Issues

**Empty snapshot / 131 lines only**: Content not loaded yet. Use `browser_evaluate` on `main.innerText`, not the snapshot YAML.

**`Show More Titles` in playlist**: Click it iteratively. Extract group names from snapshot with `re.findall(r'group \"([^\"]+)\" \[ref=', text)`.

**Cross-origin iframe**: O'Reilly renders some content in iframes — `contentDocument` will be null. `main.innerText` from the top-level document is the correct target.

**File save path**: Playwright saves files relative to the Playwright working directory (`~/.playwright-mcp/` or cwd). Find with `find /home -name "chapter-1-text.txt" 2>/dev/null`.

## Synthesis Agent Prompt Template

```
Read these files completely and produce a chapter-by-chapter technical synthesis:
[list file paths]

For each chapter:
1. What the chapter builds (actual code/system constructed)
2. Key technical mechanisms with enough depth to understand HOW they work
3. Most interesting or non-obvious insight
4. Specific code patterns worth noting

Audience: [specify — security researcher / ML engineer / systems programmer / etc.]
Skip all beginner framing. Quote specific technical details, numbers, patterns.
```
