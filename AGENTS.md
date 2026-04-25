# Repository Guidelines

## Project Structure & Module Organization
This repository is a Markdown knowledge base about Claude Code. The main content lives in root-level lesson files such as `01-Claude-Code-整体认知与工作模型.md` through `22-平台扩展-Desktop-Slack-Chrome-JetBrains-Computer-Use.md`. Keep new topics in this numbered sequence when they represent a new standalone module. Repository-local Claude settings live under `.claude/`; treat `.claude/settings.local.json` as local tooling support, not core documentation content.

## Build, Test, and Development Commands
There is no application build, package manager, or automated test suite in this repo. Use lightweight review commands instead:

- `rg --files` lists all lesson files and is the fastest way to inspect coverage.
- `Get-Content -Encoding UTF8 <file>` verifies Chinese text renders correctly in PowerShell.
- `git diff --check` catches trailing whitespace and malformed line endings.
- `git status --short` confirms the exact files changed before opening a PR.

Preview Markdown in your editor before submitting if you changed headings, lists, or code fences.

## Coding Style & Naming Conventions
Write in UTF-8 Markdown. Preserve the existing structure: one `#` title per file, short `##` sections, concise bullets, and direct instructional language. Keep filenames descriptive and ordered with a two-digit numeric prefix, for example `23-新主题.md`. Match the repository’s bilingual naming pattern when appropriate, using English keywords only where they improve searchability or align with product terminology.

## Testing Guidelines
Validation is manual. Check for:

- correct heading hierarchy
- readable Markdown rendering
- consistent terminology across related lessons
- accurate command names, paths, and dates

When editing factual content, verify neighboring modules for duplication or contradiction.

## Commit & Pull Request Guidelines
Recent commits use short imperative English summaries, for example `Add comprehensive learning modules for Claude Code`. Follow that pattern and keep one logical documentation change per commit.

PRs should include a brief purpose statement, the files changed, and any terminology or structural decisions that affect other modules. Screenshots are optional and only useful when showing Markdown rendering issues.

## Configuration Notes
Do not depend on `.claude/settings.local.json` for repository behavior; local permission allowlists may differ across contributors.
