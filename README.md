<h1 align="center">oreilly-reader</h1>

<h4 align="center">Claude Code skill that reads and synthesizes O'Reilly Learning books.</h4>

<p align="center">
  <a href="https://github.com/zellkernel/oreilly-reader-skill/blob/main/LICENSE"><img src="https://img.shields.io/github/license/zellkernel/oreilly-reader-skill?style=flat-square" alt="license"></a>
  <a href="https://claude.ai/code"><img src="https://img.shields.io/badge/Claude_Code-skill-blueviolet?style=flat-square" alt="Claude Code"></a>
  <a href="https://playwright.dev"><img src="https://img.shields.io/badge/Playwright-MCP-2EAD33?style=flat-square" alt="Playwright"></a>
  <a href="https://zellkernel.com"><img src="https://img.shields.io/badge/by-NuClide-blue?style=flat-square" alt="NuClide"></a>
</p>

<p align="center">
  <a href="#features">Features</a> •
  <a href="#installation">Installation</a> •
  <a href="#usage">Usage</a> •
  <a href="#modes">Modes</a> •
  <a href="#how-it-works">How It Works</a>
</p>

---

oreilly-reader is a Claude Code skill. It navigates an O'Reilly Learning book chapter by chapter through Playwright browser automation, extracts the full text from each chapter, and produces a technical synthesis tuned to the reader's background.

O'Reilly renders chapter content with JavaScript. A standard HTTP fetch returns a shell. The skill drives the Playwright MCP browser, evaluates `document.querySelector('main').innerText` on each chapter page, and pulls the actual content. The skill avoids `browser_snapshot` for chapter extraction: real chapters exceed 170K characters and the snapshot path truncates.

# Features

- Two modes: single book (URL or title to synthesis) and research goal (engineering goal to cross-book synthesis)
- Playlist URL handling: extracts the book list, picks the target book
- TOC parsing via `browser_evaluate`, robust across publisher URL patterns (Packt, Apress, O'Reilly native)
- Per-chapter text extraction direct from the rendered DOM
- Sub-agent synthesis with tunable depth
- No Python dependencies. Skill bundle ships as a single `.skill` archive
- Lives alongside the rest of the NuClide research toolchain

# Installation

Requires Claude Code with the Playwright MCP plugin enabled and an active O'Reilly Learning subscription (logged in via the Playwright browser).

```bash
claude mcp install oreilly-reader.skill
```

Or download `oreilly-reader.skill` from the [releases page](../../releases) and install through Claude Code settings.

# Usage

Drop an O'Reilly book URL into Claude Code:

```
https://learning.oreilly.com/library/view/build-a-large/9781633437166/
```

Or a playlist URL:

```
https://learning.oreilly.com/playlists/d1def9e4-815c-4a24-9db5-db7da9ebc16d
```

The skill handles playlist to book-list extraction, TOC parsing, chapter-by-chapter text extraction, and synthesis through a sub-agent at tunable depth.

# Modes

**Single book.** User provides a URL or a title. The skill reads targeted chapters and synthesizes. Best for "what's the meat of this book" sessions.

**Research goal.** User has an engineering goal ("find books for X"). The skill searches O'Reilly, evaluates TOCs, reads targeted chapters across multiple books, then produces a cross-book synthesis. This is the more powerful mode, designed for read-apply-repeat cycles on a specific engineering target.

# How it works

O'Reilly renders chapter content via JavaScript. Static HTTP fetch returns a shell. The only reliable extraction path is `document.querySelector('main').innerText` via browser evaluation after the page loads.

`browser_snapshot` is never used for chapter text. Snapshots blow past Claude Code's token limit on any real chapter (171K characters trigger truncation). The skill calls `browser_evaluate` with the `filename` parameter for every chapter read.

TOC discovery uses `browser_evaluate` against `a[href*="/library/view/"]` and filters by ISBN. The same approach holds across publishers; only the per-chapter URL pattern varies (Packt uses `/Text/Chapter_N.xhtml` or `/BN_NN.xhtml`, O'Reilly native uses `/c01.xhtml`, Apress uses `/XHTML/B978...`).

# Auth

If Playwright redirects to the login page, log into O'Reilly in your browser. The Playwright session needs a live O'Reilly cookie. Once logged in, navigate back to the book URL.

# Skill bundle

The skill ships as a Claude Code skill bundle. The bundle contains:

```
oreilly-reader/
  SKILL.md          # skill definition: name, description, allowed-tools, workflow
oreilly-reader.skill  # the packaged bundle
```

The `allowed-tools` field gates the skill to the Playwright MCP browser tools plus `Bash` and `Agent` for the synthesis sub-agent.

# Our other projects

- [aimap](https://github.com/zellkernel/aimap) — fingerprint scanner for AI and ML infrastructure
- [pharos](https://github.com/zellkernel/pharos) — autonomous offensive research agent
- [sentinel](https://github.com/zellkernel/sentinel) — CVE-reactive exposure pipeline
- [scanner](https://github.com/zellkernel/scanner) — fast active-banner stage between Shodan and aimap
- [visorlog](https://github.com/zellkernel/visorlog) — finding ledger

# License

MIT. Part of the NuClide toolchain. Contact: [zellkernel.com](https://zellkernel.com)
