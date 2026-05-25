---
name: google-style-markdown
description: Google Markdown Style Guide — document structure, formatting, headings, lists, links, code, and tables. Use when writing or reviewing Markdown documentation to apply Google's standards. Triggers on any Markdown documentation task.
---

# Google Markdown Style Guide — Quick Reference

## Goals
- **Source readable and portable**: Markdown should be easy to read in raw form and render correctly across tools.
- **Maintainable corpus**: Consistent formatting makes large documentation sets easier to maintain.
- **Simple syntax**: Prefer simpler Markdown constructs over complex ones.

## Minimum viable documentation
- Keep documentation fresh and accurate.
- Delete cruft — outdated or irrelevant content harms the corpus.
- **Better is better than best**: Good documentation shipped today beats perfect documentation next quarter. This applies to both reviewers and authors.

## Capitalization
- Preserve original product and tool names (e.g., `JavaScript`, `npm`, `GitHub`, `iOS`, `macOS`).
- Do not apply title case or强行 alter proper names.

## Document layout
```
# Document Title
Short introduction paragraph.

[TOC]

## Topic
Content.

## See also
Related links.
```

## Character line limit
- **80 characters** maximum per line.
- Exceptions: links, tables, headings, code blocks (these may exceed 80 chars).

## Trailing whitespace
- **Do not use trailing whitespace**.
- Use a trailing backslash (`\`) for intentional line breaks (sparingly).

## Headings
- **ATX style only** — use `#` markers, not underlines (`===` or `---`).
- Each heading name must be unique within a document.
- Space after `#`: `## Heading` (not `##Heading`).
- Blank line before and after headings (except at file start).
- **Single H1** (`# Title`) per document.

## Lists

### Ordered lists (lazy numbering)
```
1. First item
1. Second item
1. Third item with
   continuation text indented 4 spaces
```

### Unordered lists
```
*   Bullet with 3 spaces after `*`
    Wrapped text indented 4 spaces.
*   Second bullet.
```

### Nested content
- Use **4-space indent** for nested paragraphs, code blocks, or sub-lists.

## Code

### Inline code
- Use backticks: \`code\`.
- For code containing backticks, use double backticks: \`\`code\`\`.

### Fenced code blocks
- Use fenced blocks with language declaration (preferred over indented blocks).
````markdown
```python
def hello():
    print("Hello, world!")
```
````

### Indented code blocks
- Avoid. Use fenced blocks instead.

### Shell output
- Escape newlines with `\` in shell commands shown inline.

## Links

### Explicit paths within Markdown
- Use explicit file paths (e.g., `[text](file.md)`).
- Avoid relative paths that traverse outside the same directory (prefer absolute repo paths or full URLs).

### Informative link titles
- Link text should describe the destination (not "click here").

### Reference links
- Use for long URLs or repeated links.
```markdown
Some text with a [reference][ref-id] link.

[ref-id]: https://example.com
```
- Define references after the paragraph or section where they first appear.

## Images
```markdown
![Alt text](url-or-path.png)
```

## Tables
- Use standard Markdown tables.
```
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
```

## Prefer Markdown to HTML
- Use Markdown syntax whenever possible.
- Avoid raw HTML except where Markdown has no equivalent (e.g., certain advanced table features).

---

Full source: `G:\styleguide\docguide\style.md`
