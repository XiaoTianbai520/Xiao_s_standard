# Repository Guidelines

## Project Structure & Module Organization

This repository is a flat collection of Chinese-language Markdown standards and operational notes. Keep one subject per root-level file and use a descriptive Chinese filename, such as `Ubuntu服务器规范.md`, `RocketMQ.md`, or `命名规范.md`. There are currently no source, test, or asset directories. If a topic becomes large, create a clearly named folder and keep its images in a local `assets/` subdirectory; use relative links so the documentation remains portable.

## Local Review Commands

There is no build system or application runtime. Review documentation directly from the repository root:

```powershell
Get-ChildItem -File *.md                    # list documentation files
rg -g '*.md' '^#{1,6} ' .                    # inspect heading structure
rg -g '*.md' '密码|token|secret|api[_-]?key' . # detect possible credentials
```

If a Markdown linter is introduced later, commit its configuration and document the exact invocation here. Do not make contributors depend on an unrecorded global tool.

## Writing Style & Naming Conventions

Write concise Simplified Chinese unless a technical term is conventionally English. Start each document with one `#` title, use `##` and `###` headings in order, and leave blank lines around headings, lists, tables, and fenced code blocks. Use backticks for commands, paths, ports, identifiers, and short examples. Prefer tables for structured service information and fenced blocks for multi-line commands. Keep terminology and capitalization consistent within a document (for example, `RocketMQ`, `Docker Compose`, and `ConsumerGroup`).

## Validation Guidelines

Documentation review is the current test process. Before submitting, verify that commands are safe and syntactically complete, links and relative paths resolve, tables render correctly, and examples match the stated convention. Re-read changed sections in a Markdown preview. When changing operational instructions, include prerequisites and distinguish start, stop, restart, inspection, and destructive commands.

## Commits & Pull Requests

Git history is not available in this checkout, so no existing commit convention can be inferred. Use short, imperative, scoped messages such as `docs: clarify RocketMQ naming rules`. Keep each commit focused. Pull requests should summarize the affected documents, explain why the guidance changed, identify commands or examples that were verified, and include screenshots only when rendering is relevant.

## Security & Configuration

Never commit real passwords, tokens, private keys, public endpoints, or production account details. Replace them with placeholders such as `<server-ip>` and `<password>`. Treat destructive examples such as `docker compose down` carefully: state their impact and any persistence or backup assumptions next to the command.
