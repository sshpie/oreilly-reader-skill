---
name: oreilly-reader
description: Use when reading O'Reilly Learning books, extracting chapter content from oreilly.com, synthesizing technical book content via browser automation, or researching multiple books for a specific engineering goal. Triggers on O'Reilly book URLs, "read this book", "what's in this O'Reilly book", requests to extract/summarize O'Reilly content, or "look for books on X" research sessions.
allowed-tools: mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_evaluate, mcp__plugin_playwright_playwright__browser_snapshot, mcp__plugin_playwright_playwright__browser_click, mcp__plugin_playwright_playwright__browser_take_screenshot, Bash, Agent
---

# O'Reilly Reader

Read and synthesize O'Reilly Learning books via Playwright browser automation. Requires an active O'Reilly session in the Playwright browser (user must be logged in).

## Two Modes

**Single book**: User provides a URL or title → read targeted chapters → synthesize.

**Research goal**: User has an engineering goal ("find books for X") → search O'Reilly → evaluate TOCs → read targeted chapters across multiple books → cross-book synthesis. This is the more powerful mode.

---

## How It Works

O'Reilly renders chapter content via JavaScript — static HTTP fetch returns a shell. The only reliable extraction path is `document.querySelector('main').innerText` via browser evaluation after the page loads.

**Never use `browser_snapshot` to extract chapter text** — snapshots exceed token limits on any real chapter (171K+ chars triggers truncation). Always use `browser_evaluate` with the `filename` parameter.

---

## Single Book Workflow

### 1. Navigate to book

```
https://learning.oreilly.com/library/view/[slug]/[ISBN]/
```

If given a playlist URL, navigate to the playlist first, extract book titles/links from the snapshot, then pick the target book.

### 2. Extract TOC

Use `browser_evaluate` — do **not** rely on snapshot YAML for TOC. Books use different URL patterns:

```javascript
() => {
  const links = [];
  // Works for all book formats
  document.querySelectorAll('a[href*="/library/view/"]').forEach(a => {
    const text = a.innerText.trim();
    if (text.length > 3 && a.href.includes(ISBN) && !links.find(l => l.href === a.href))
      links.push({ text: text.substring(0, 100), href: a.href });
  });
  return JSON.stringify(links, null, 2);
}
```

Common chapter URL patterns (varies by publisher):
- Packt: `/Text/Chapter_N.xhtml` or `/BN_NN.xhtml`
- O'Reilly native: `/c01.xhtml`, `/c02.xhtml`
- Apress: `/XHTML/B978...000001X/...xhtml`

### 3. Select targeted chapters

Read the TOC and **select only the chapters relevant to the goal**. Do not read every chapter. Chapter titles are the filter — pick the 2-4 that match the specific technical need.

### 4. Read chapters

For each chapter URL, navigate then evaluate with filename:

```javascript
() => { return (document.querySelector('main') || document.body).innerText; }
```

Use the `filename` parameter to save to disk:
```
filename: bookabbrev-chNN-topic.txt
```

Name files with a short book abbreviation + chapter number + topic slug, e.g.:
- `hpp-ch08-async-io.txt` (High Performance Python ch8)
- `oti-ch07-enrichment.txt` (Operationalizing Threat Intelligence ch7)

Files save to the current working directory. Navigate → evaluate → save, one chapter at a time.

### 5. Synthesize

Spawn a synthesis `Agent` (Explore subagent) with all file paths. See Synthesis Agent Prompt Template below.

---

## Multi-Book Research Workflow

Use when building something and want to know what the literature says before writing code.

### 1. Define the research goal

State it precisely: "Building an ASN-to-institution attribution pipeline using async Python + WHOIS + RDAP." This determines which chapters matter.

### 2. Search O'Reilly

Run 2-4 targeted searches. Be specific:

```
https://learning.oreilly.com/search/?q=TOPIC+SUBTOPIC&type=book&rows=50&language=en
```

Good search terms for technical topics: include author names, specific library names, or problem-domain terms (not just "python").

Extract results with:
```javascript
() => {
  const results = [];
  document.querySelectorAll('a[href*="/library/view/"]').forEach(a => {
    const text = a.innerText.trim();
    if (text.length > 5 && !results.find(r => r.href === a.href))
      results.push({ title: text.substring(0, 100), url: a.href });
  });
  return JSON.stringify(results.slice(0, 30), null, 2);
}
```

### 3. Triage TOCs before reading

For each candidate book, navigate to its main page and extract the TOC (see step 2 above). Decide which chapters are relevant **before** reading any content. Skip books where no chapter matches the goal.

### 4. Read targeted chapters only

Read 2-4 chapters per book, never the full book. Goal: extract the specific techniques that apply to the engineering problem. Name files by book + chapter.

### 5. Create a playlist

Add all books read to a named O'Reilly playlist for future reference:

**Create:**
- Navigate to `https://learning.oreilly.com/playlists/`
- Click `[data-testid="publicCreateButton"]`
- Fill title input `#«r6»` and description input `#«r7»`
- Click `role=button[name='Create Playlist']`

**Add a book:**
- Navigate to the book's main page
- Click `text=Add to playlist`
- Click the playlist name in the dialog
- Click `role=button[name='Done']` (use role selector — `text=Done` is ambiguous)

### 6. Cross-book synthesis

Spawn a synthesis `Agent` (Explore subagent) with all file paths from all books. See Synthesis Agent Prompt Template below.

---

## Auth Handling

If navigation redirects to `learning.oreilly.com/member/login/`, the Playwright session is not authenticated. Ask the user to log in via their browser. The Playwright session must share the browser profile with the logged-in session.

---

## Quick Reference

| Step | Tool | Notes |
|------|------|-------|
| Navigate | `browser_navigate` | Full book URL |
| Get TOC | `browser_evaluate` JS | `querySelectorAll('a[href*="/library/view/"]')` |
| Extract chapter | `browser_evaluate` + `filename` | `main.innerText` → file |
| Playlist create | `browser_click [data-testid="publicCreateButton"]` | Then fill `#«r6»` + `#«r7»` |
| Playlist add | `browser_click text=Add to playlist` | Then `role=button[name='Done']` |
| Synthesize | `Agent` Explore subagent | Feed all file paths |

---

## Common Issues

**Snapshot overflow (171K+ chars, truncated)**: Chapter snapshots always overflow. Use `browser_evaluate` with `filename` parameter for chapter text — never `browser_snapshot` for content extraction.

**`text=Done` is ambiguous**: The playlist dialog Done button conflicts with card descriptions. Use `role=button[name='Done']` instead.

**`text=New` is ambiguous**: Multiple elements match on the playlists page. Use `[data-testid="publicCreateButton"]` for creating new playlists.

**TOC links not in `/Text/` pattern**: Packt books use `/Text/Chapter_N.xhtml`, O'Reilly native use `/c01.xhtml`, Apress use long ISBN-based paths. Use the broad `a[href*="/library/view/"]` selector and filter by ISBN instead of path pattern.

**Empty snapshot / 131 lines only**: Content not loaded yet. Use `browser_evaluate` on `main.innerText`, not the snapshot YAML.

**`Show More Titles` in playlist**: Click it iteratively. Extract group names from snapshot with `re.findall(r'group \"([^\"]+)\" \[ref=', text)`.

**Cross-origin iframe**: O'Reilly renders some content in iframes — `contentDocument` will be null. `main.innerText` from the top-level document is the correct target.

**File save path**: Playwright saves files relative to the Playwright working directory (`~/.playwright-mcp/` or cwd). Find with `find /home -name "chapter-1-text.txt" 2>/dev/null`.

---

## Synthesis Agent Prompt Template

### Single book

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

### Multi-book research synthesis

```
Read ALL of these [N] files completely and produce a focused technical synthesis for [GOAL].

Context: [explain what you're building and what decisions the synthesis should inform]

FILES:
[list all file paths with book and chapter annotations]

Produce:

1. NEW insights not obvious from first principles (avoid: "use async", DO: specific function/class names, concrete tradeoffs, measured numbers from the text)

2. Specific code patterns worth implementing (exact technique, library call, or algorithm from the text — not pseudocode)

3. [Domain-specific section for your goal, e.g. "IP attribution techniques", "ASN data sources mentioned"]

4. Comparison table: with vs without book research
   Columns: Aspect | Without Books | With Books | Impact
   Minimum 6 rows. Be concrete — function names, order-of-magnitude differences, false positive rates.

5. Any surprising cross-book convergences or contradictions

Audience: [specify]
Do NOT summarize file by file. Synthesize across all sources.
Only report what's actually in the texts.
```

The comparison table ("with vs without books") is the highest-value output for engineering decisions — it forces the synthesis to be concrete and actionable rather than encyclopedic.
