# oreilly-reader

Claude Code skill — reads and synthesizes O'Reilly Learning books via Playwright browser automation.

## What it does

Navigates an O'Reilly book chapter by chapter, extracts the full text via JavaScript evaluation, and produces a technical chapter-by-chapter synthesis tuned to the reader's background.

O'Reilly renders content dynamically — standard HTTP fetch returns a shell. This skill uses the Playwright MCP browser to evaluate `document.querySelector('main').innerText` on each chapter page, which gets the actual content.

## Requirements

- [Claude Code](https://claude.ai/code) with the Playwright MCP plugin enabled
- An active O'Reilly Learning subscription (logged in via Playwright browser)

## Install

```bash
claude mcp install oreilly-reader.skill
```

Or download `oreilly-reader.skill` from the [releases page](../../releases) and install via Claude Code settings.

## Usage

Drop an O'Reilly book URL into Claude Code:

```
https://learning.oreilly.com/library/view/build-a-large/9781633437166/
```

Or from a playlist:

```
https://learning.oreilly.com/playlists/d1def9e4-815c-4a24-9db5-db7da9ebc16d
```

The skill handles:
- Playlist → book list extraction
- TOC parsing
- Chapter-by-chapter text extraction
- Synthesis via a sub-agent (depth tunable to audience)

## Auth

If Playwright redirects to the login page, log into O'Reilly in your browser. The Playwright session needs a live O'Reilly cookie. Once logged in, navigate back to the book URL.

## Built by

[NuClide Research](https://nuclide-research.com) — nicholas@nuclide-research.com
