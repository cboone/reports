# Reports Repository

A collection of detailed reports on various topics, created by Claude.

## Structure

Each topic gets its own directory at the repository root. Reports are Markdown files within those directories.

```
topic-name/
  report-file.md
  another-report.md
```

## Adding New Reports

When adding new reports to the repository:

1. Place files in the appropriate topic directory, creating it if needed. Use lowercase kebab-case for directory names.
2. Each report file needs two date indicators:
   - **YAML frontmatter** with a `created` date in `YYYY-MM-DD` format:
     ```
     ---
     created: 2026-02-15
     ---
     ```
   - **Inline datestamp** in italics immediately below the `#` title, using `_Month Day, Year_` format:
     ```
     # Report title

     _February 15, 2026_
     ```
3. Update `README.md`:
   - Topics are listed alphabetically as `##` headings. No per-topic intro paragraph — the report summaries cover that.
   - Each report is its own `###` sub-section with a linked title and a summary paragraph (2-4 sentences, enough to convey scope and what the reader will find). No datestamps in the README — dates live in the reports themselves.
     ```
     ### [Report title](topic/filename.md)

     Summary paragraph describing the report's scope, what it covers, and what the reader will find.
     ```
   - Report titles must be self-explanatory without relying on the topic section for context. Avoid generic names like "Bibliography" or "Configuration files comparison" — use titles like "Agile methodology: sources and references" or "LLM coding agent configuration files: comparing Claude Code, Codex, Copilot, and OpenCode."

## Report Style

- Reports are comprehensive, long-form Markdown documents.
- Use standard Markdown formatting: headings, tables, lists, code blocks.
- Companion reports should cross-reference each other with relative links.
