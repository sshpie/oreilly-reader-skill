# oreilly-reader

Claude Code skill that reads and synthesizes O'Reilly Learning books via Playwright browser automation.

O'Reilly renders chapter content dynamically. A static HTTP fetch returns a shell; the actual text only exists after JavaScript runs. This skill drives the Playwright MCP browser to navigate each chapter, evaluates `(document.querySelector('main') || document.body).innerText` to extract the full text, and saves it to disk. A synthesis subagent then reads all saved files and produces a technical chapter-by-chapter synthesis. Snapshot-based extraction is explicitly avoided: chapter snapshots exceed 171K characters and truncate.

Two modes: single-book (user provides a URL or title, skill reads targeted chapters) and multi-book research (user states an engineering goal, skill searches O'Reilly, triages TOCs, reads targeted chapters across multiple books, and synthesizes across all sources).

## Requirements

- Claude Code with the Playwright MCP plugin enabled
- An active O'Reilly Learning subscription, logged in via the Playwright browser

## Install

```
claude mcp install oreilly-reader.skill
```

Or download `oreilly-reader.skill` from the releases page and install via Claude Code settings.

## Triggers

The skill fires on:
- An O'Reilly book URL (`learning.oreilly.com/library/view/...`)
- An O'Reilly playlist URL (`learning.oreilly.com/playlists/...`)
- "read this book", "what's in this O'Reilly book"
- Requests to extract or summarize O'Reilly content
- "look for books on X" research sessions

## How it works

### Single book

1. Navigate to the book URL.
2. Extract the TOC via `browser_evaluate` with a `querySelectorAll('a[href*="/library/view/"]')` selector.
3. Select the 2-4 chapters relevant to the goal (by title).
4. For each chapter: navigate, evaluate `main.innerText`, save to a named file on disk.
5. Spawn a synthesis `Agent` (Explore subagent) with all saved file paths.

### Multi-book research

1. User states a precise engineering goal.
2. Run 2-4 targeted O'Reilly searches.
3. For each candidate book: extract the TOC, decide which chapters match before reading any content.
4. Read 2-4 chapters per book, name files by book abbreviation + chapter + topic.
5. Optionally create an O'Reilly playlist holding all books read.
6. Spawn a synthesis subagent across all files from all books.

The synthesis agent produces: new insights not obvious from first principles, specific code patterns worth implementing, domain-specific findings, a comparison table (with vs without book research, minimum 6 rows), and cross-book convergences or contradictions.

## Auth

If Playwright redirects to the login page, the session is not authenticated. Log into O'Reilly in the Playwright browser. Once logged in, navigate back to the book URL and re-invoke the skill.

## What this skill is not

This skill does not cache or store book content beyond the current session's working directory files. It does not bypass O'Reilly's paywall; it requires a valid logged-in subscription. It reads only chapters the user targets; it does not auto-read every chapter in a book.

## License

MIT. Part of the NuClide toolchain. Contact: [nuclide-research.com](https://nuclide-research.com)
