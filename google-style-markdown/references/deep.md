# Google Markdown Style Guide — Deep Reference

## Table of Contents

[TOC]

---

## 1. Goals

The Google Markdown Style Guide is built on three core goals:

### Source readable and portable

Markdown is designed to be readable as plain text. Raw Markdown should be intelligible without being rendered. At the same time, it must render correctly across the widest possible set of tools — GitHub, Gitiles, code editors, static site generators, and documentation browsers. This means using well-supported syntax and avoiding tool-specific extensions where possible.

### Maintainable corpus

Google's documentation corpus is large and long-lived. Consistent formatting conventions reduce friction when editing, reviewing, and migrating content. A maintainable corpus means:

- New contributors can predict the style of any file.
- Automated tools (linters, formatters, validators) can enforce rules.
- Reviews focus on content and correctness, not formatting debates.
- Bulk updates (e.g., link migrations, branding changes) are feasible.

### Simple syntax

Prefer the simplest Markdown construct that achieves the goal. For example:

```markdown
<!-- Prefer: -->
*   Simple bullet list

<!-- Over: -->
- [ ] Complex task list syntax when you only need a bullet
```

When there are multiple ways to express something in Markdown, choose the option that is easiest to read, easiest to write, and least likely to break across renderers.

---

## 2. Minimum viable documentation

Documentation is a living artifact. It must be kept fresh, accurate, and concise.

### Fresh and accurate

Stale documentation is worse than no documentation — it misleads readers and erodes trust. When updating features, always update the corresponding docs. If you cannot keep a document current, consider removing or deprecating it.

### Delete cruft

Outdated examples, deprecated API references, orphaned pages, and redundant explanations accumulate over time. Be aggressive about removing content that no longer serves a clear purpose. Every line of documentation carries a maintenance cost.

### Better is better than best

This principle applies to both reviewers and authors:

- **For authors**: Ship good documentation now rather than perfect documentation later. Iterate in follow-up changes.
- **For reviewers**: Approve documentation that meets the bar of "better than what existed before" and improves the overall state. Do not block on stylistic perfection.

The corollary: "perfect is the enemy of good." A 90% improvement that ships today is worth more than a 100% solution that ships next quarter.

---

## 3. Capitalization

### Preserve original names

Always use the official capitalization of product names, tools, technologies, and brands:

```markdown
<!-- Correct -->
JavaScript
npm
GitHub
iOS
macOS
Node.js
TypeScript
HTML
CSS
Google Cloud Platform

<!-- Incorrect -->
Javascript
NPM
Github
Ios
Macos
Node.JS
Typescript
Html
Css
Google cloud platform
```

### Sentence case in documentation text

Within running prose, use sentence case unless referring to a proper noun or a specific UI label. For headings, see the [Headings deep dive](#8-headings-deep-dive) section on capitalization conventions.

Do not apply title case to document titles or headings unless a proper noun requires it.

---

## 4. Document layout

Every Markdown document should follow a consistent structural template:

```markdown
# Document Title

A short introduction paragraph that states the purpose and scope of the document.

[TOC]

## First Topic

Content.

## Second Topic

Content.

## See also

- [Related document](related.md)
- [External reference](https://example.com)
```

### Title

A single H1 (`# Title`) at the top of the document. The title should be descriptive and concise. Only one H1 per document.

### Author (optional)

If authorship or ownership metadata is needed, use a comment or YAML front matter — not visible text styling.

### Short introduction

One or two paragraphs immediately after the title, before the table of contents. This should tell the reader:

- What this document covers
- Who it is for
- What prerequisite knowledge is assumed (if any)

### Topic sections

Each major topic gets an H2 (`## Topic`). Sub-topics use H3 (`### Sub-topic`), H4, etc. Do not skip levels (e.g., do not jump from H2 to H4).

### See also

A final `## See also` section with links to related documentation. This helps readers navigate the broader corpus.

---

## 5. Table of contents

### The `[TOC]` directive

Use the `[TOC]` directive to auto-generate a table of contents. Place it after the introductory paragraph(s) and before the first H2:

```markdown
# My Document

This document covers the FooBar API.

[TOC]

## Getting Started
```

### Placement rules

- Always after the introductory paragraph.
- Always before the first H2.
- On its own line with blank lines before and after.

### Limitations

The `[TOC]` directive is supported by Gitiles and many Markdown renderers but not all. If your target renderer does not support `[TOC]`, provide a manual table of contents using a bullet list of links to headings.

---

## 6. Character line limit

### 80 characters maximum

All lines should be wrapped at **80 characters** unless they fall under an explicit exception.

```markdown
<!-- Lines should wrap like this. The 80-char limit keeps diffs readable and    -->
<!-- side-by-side editing possible without horizontal scrolling.                 -->
```

### Exceptions

The following elements may exceed 80 characters:

| Element        | Rationale                                                       |
|----------------|----------------------------------------------------------------|
| **Links**      | Breaking a URL across lines can break the link.                |
| **Tables**     | Table formatting depends on column alignment.                  |
| **Headings**   | Forced wrapping may render oddly.                              |
| **Code blocks**| Breaking code at 80 chars may introduce syntax errors.         |

When an exception applies, still try to keep lines reasonably short. For example, prefer reference-style links when a URL is excessively long.

### Implementation tips

- Many editors support "rewrap" or "hard wrap at 80" commands.
- Use a linter (e.g., markdownlint rule MD013) to automate enforcement.
- When wrapping prose, break at sentence boundaries or natural phrase boundaries rather than in the middle of a word.

---

## 7. Trailing whitespace

### Avoid trailing whitespace

Trailing whitespace at the end of a line is invisible in most editors and renders inconsistently across Markdown processors. It can cause:

- Unintended line breaks in some renderers
- Noise in diffs
- Linter warnings

```markdown
<!-- Correct: no trailing spaces -->
This line ends cleanly.

<!-- Incorrect: trailing spaces (invisible but present) -->
This line has trailing whitespace···
```

### Line breaks with trailing backslash

If you need an intentional line break (a `<br>` equivalent), use a trailing backslash (`\`) sparingly:

```markdown
First line\
Second line
```

Reserve this for cases where a line break is semantically meaningful (e.g., addresses, poetry, song lyrics). For most prose, simply start a new paragraph (double newline) instead.

### Hard line breaks in tables

Within table cells, use `<br>` HTML tags for line breaks if needed, since trailing backslash does not work reliably inside tables.

---

## 8. Headings deep dive

### ATX style only

Use ATX-style headings (`#`, `##`, `###`, etc.). Do not use setext-style headings (underlined with `===` or `---`):

```markdown
<!-- Correct: ATX -->
## Heading

<!-- Incorrect: setext -->
Heading
=======
```

ATX headings are unambiguous, easier to search for, and work consistently across all Markdown renderers.

### Unique heading names

Every heading in a document must have a unique name. Duplicate heading names break anchor links and confuse readers:

```markdown
<!-- Incorrect: duplicate headings -->
## Configuration
...
## Configuration  <!-- same name, different section -->

<!-- Correct: disambiguate -->
## Build Configuration
...
## Runtime Configuration
```

### Spacing rules

- One space between `#` and heading text: `## Heading` not `##Heading`.
- Blank line before the heading (unless it is the first line of the file).
- Blank line after the heading (before the body content).

```markdown
# Title

(blank line)
## Section
(blank line)
Content starts here.
```

### Single H1

Use exactly one `# Title` per document. The H1 represents the document title. Additional top-level headings should be H2 or below.

```markdown
# Document Title   ← the only H1

## Section          ← H2

### Sub-section     ← H3
```

### Heading capitalization

Follow the Google Developer Documentation Style Guide for heading capitalization:

- Use sentence case (capitalize only the first word and proper nouns).
- Do not use title case.

```markdown
<!-- Correct: sentence case -->
## Configuring the build pipeline

<!-- Incorrect: title case -->
## Configuring the Build Pipeline
```

### Heading hierarchy

Do not skip levels. An H3 must follow an H2 (or another H3), not an H1 directly:

```markdown
<!-- Correct -->
# Title
## Section
### Sub-section

<!-- Incorrect: skipping level -->
# Title
### Sub-section  (no H2 between H1 and H3)
```

---

## 9. Lists deep dive

### Ordered lists: lazy numbering

For ordered (numbered) lists with more than a few items, use lazy numbering — mark every item as `1.`:

```markdown
1. First step
1. Second step
1. Third step with a longer description that wraps
   to the next line.
1. Fourth step
```

Lazy numbering makes reordering items trivial — you never need to renumber. Most Markdown renderers display sequential numbers regardless of the source markup.

For short lists (2–3 items) where reordering is unlikely, explicit numbering is acceptable:

```markdown
1. First
2. Second
3. Third
```

### Unordered lists: spacing

Use `*` for bullet items (not `-` or `+`). Follow `*` with 3 spaces before the text:

```markdown
*   First item
*   Second item
*   Third item
```

### Nested content: 4-space indent

When a list item contains wrapped text, nested lists, code blocks, or paragraphs, indent the continuation content by 4 spaces:

```markdown
1. Item with a paragraph continuation.

    This is a second paragraph under the same list item,
    indented by 4 spaces.

1. Item with a nested list:

    *   Nested bullet
    *   Another nested bullet

1. Item with a code block:

    ```python
    print("hello")
    ```
```

### List consistency

- Within a single list, use the same marker type (`*` or `1.`) throughout.
- Do not mix ordered and unordered markers at the same nesting level.
- Blank lines before and after lists help readability but are not required by most renderers. Use them when a list is preceded or followed by a paragraph.

---

## 10. Code deep dive

### Inline code

Use single backticks for inline code references:

```markdown
Use the `parse()` function to process input.
```

### Code spans containing backticks

If the code itself contains backticks, use double backticks as delimiters:

```markdown
To reference a variable named `` `foo` ``, use double backticks.
```

### Fenced code blocks

Use fenced code blocks (triple backticks with a language declaration) for multiline code. Always declare the language when possible:

```markdown
```python
def greet(name):
    return f"Hello, {name}"
```

```json
{
  "key": "value"
}
```

```bash
npm install
```
```

Language declaration enables syntax highlighting and helps screen readers. If no language applies, use `text` or `none`:

```markdown
```text
Raw output or plain text.
```
```

### Avoid indented code blocks

Indented code blocks (4-space indent) are valid Markdown but should be avoided. They do not support language declaration, cannot be fenced, and are visually ambiguous:

```markdown
<!-- Avoid: indented code block -->
    const x = 1;

<!-- Prefer: fenced code block -->
```javascript
const x = 1;
```
```

### Escape newlines in shell

When showing shell commands that span multiple lines, use a backslash to escape the newline:

```markdown
```bash
docker run \
  --name my-container \
  --publish 8080:80 \
  nginx:latest
```
```

### Code in lists

When a code block appears inside a list item, indent it by 4 spaces from the list marker, plus the usual fenced block markers:

```markdown
1. Configure the application:

    ```yaml
    key: value
    ```

1. Start the service.
```

---

## 11. Links deep dive

### Explicit paths within Markdown

When linking to another Markdown file, use the explicit file path:

```markdown
<!-- Correct -->
See the [setup guide](setup.md) for details.

<!-- Incorrect -->
See the [setup guide](../other-repo/setup.md).
```

### Avoid relative paths outside the same directory

Relative paths that traverse up directories (`../`) or across the repository structure are brittle and break when files are moved. Prefer:

- Full paths from the repository root (e.g., `/docs/guide/setup.md`)
- Absolute URLs (e.g., `https://developers.google.com/...`)

### Informative link titles

Link text should describe the destination page or resource. Do not use generic phrases:

```markdown
<!-- Correct -->
See the [installation instructions](install.md) for setup steps.

<!-- Incorrect -->
Click [here](install.md) for setup steps.
```

### Reference links

For long URLs or links that appear multiple times, use reference-style links:

```markdown
The [Configuration API][config-api] provides runtime settings.
See also the [deployment options][config-api] reference.

[config-api]: https://example.com/api/config
```

### Reference link placement

Define reference link targets after the paragraph or section where they first appear, grouped together:

```markdown
First paragraph with a [reference][ref-a] and another [reference][ref-b].

[ref-a]: https://example.com/a
[ref-b]: https://example.com/b

Second paragraph with a [reference][ref-c].

[ref-c]: https://example.com/c
```

Do not place all references at the bottom of the document — put them near their first use to improve local readability.

### Link to headings within the same document

Use anchor links for intra-document navigation:

```markdown
See the [configuration section](#configuration) for details.
```

The anchor text must match the heading text (lowercase, spaces replaced with hyphens, special characters removed).

---

## 12. Images

### Standard syntax

Use the standard Markdown image syntax:

```markdown
![Alt text](path/to/image.png)
```

Provide meaningful alt text that describes the image content for accessibility. If the image is purely decorative, use an empty alt attribute:

```markdown
![Decorative divider line](divider.png)
```

A title attribute is optional but can provide additional context on hover:

```markdown
![Architecture diagram](arch.png "System architecture overview")
```

### Image paths

Follow the same path conventions as [links](#11-links-deep-dive). Store images in a dedicated directory (e.g., `images/` or `img/`) co-located with the document.

### File formats

- PNG for screenshots and diagrams
- SVG for icons and diagrams (preferred when available)
- JPEG for photographs (rare in documentation)
- GIF only when animation is required (use with caution)

---

## 13. Tables

### Standard Markdown tables

Use standard pipe-based Markdown tables:

```markdown
| Header 1     | Header 2     | Header 3     |
|--------------|---------------|--------------|
| Row 1 Cell 1 | Row 1 Cell 2  | Row 1 Cell 3 |
| Row 2 Cell 1 | Row 2 Cell 2  | Row 2 Cell 3 |
```

### Alignment

Use the colon syntax in the separator row for alignment:

```markdown
| Left-aligned | Center-aligned | Right-aligned |
|:-------------|:--------------:|--------------:|
| Left         | Center         | Right         |
```

### Formatting within tables

Most inline Markdown formatting works inside table cells:

- Bold: `**bold**`
- Inline code: `` `code` ``
- Links: `[text](url)`

For line breaks within a cell, use `<br>` HTML tags.

### Empty cells

Use a single space (or nothing) for empty cells:

```markdown
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   |          |
```

### Table width

Tables are exempt from the 80-character line limit because column alignment and readability may require wider lines. However, keep tables as narrow as practical.

---

## 14. Prefer Markdown to HTML

### Strong preference for Markdown

Use Markdown syntax whenever the language provides an equivalent. Raw HTML should be a last resort:

```markdown
<!-- Correct: Markdown -->
**Bold text**
*Italic text*
[Link](url)
![Image](img.png)

<!-- Avoid: HTML equivalent -->
<b>Bold text</b>
<i>Italic text</i>
<a href="url">Link</a>
<img src="img.png" alt="Image">
```

### When HTML is acceptable

HTML is acceptable in these cases:

- Line breaks inside table cells (`<br>`)
- Advanced table features (colspan, rowspan)
- Wrapping complex content in `<div>` for styling
- Inline formatting that Markdown cannot express (e.g., `<kbd>`, `<sup>`, `<sub>`)

When HTML is necessary, keep it minimal and well-formatted.

### Security note

Be cautious with HTML that can execute scripts or load external resources. In documentation contexts, `<script>`, `<iframe>`, and `<object>` tags are typically disallowed by renderers.

---

## 15. Review checklist

Use this checklist when reviewing Markdown documentation:

### Structure

- [ ] Single H1 at the top
- [ ] Short introduction paragraph before TOC
- [ ] `[TOC]` present and correctly placed
- [ ] Heading hierarchy does not skip levels
- [ ] `## See also` section at the end

### Headings

- [ ] ATX style only (`#`, not underlines)
- [ ] All heading names are unique
- [ ] Space after `#` markers
- [ ] Blank lines before and after headings
- [ ] Sentence case capitalization

### Line formatting

- [ ] Lines wrapped at 80 characters (exceptions noted)
- [ ] No trailing whitespace
- [ ] No trailing whitespace in code blocks either

### Lists

- [ ] Lazy numbering for ordered lists with 4+ items
- [ ] 4-space indent for nested content
- [ ] Consistent marker type within each list
- [ ] Bullet items: 3 spaces after `*`

### Code

- [ ] Inline code uses backticks
- [ ] Fenced code blocks have language declarations
- [ ] No indented code blocks (unless unavoidable)
- [ ] Shell newlines escaped with backslash

### Links

- [ ] Informative link text (not "click here")
- [ ] Reference links grouped near first use
- [ ] No relative paths traversing outside the directory
- [ ] Anchor links match heading text

### Images

- [ ] Meaningful alt text
- [ ] Standard `![alt](url)` syntax

### Tables

- [ ] Standard Markdown pipe tables
- [ ] Appropriate alignment markers

### General

- [ ] Markdown used in preference to HTML
- [ ] Product/tool names capitalized correctly
- [ ] No stale, outdated, or crufty content
- [ ] Document passes a linter (e.g., markdownlint)

---

Full source: `G:\styleguide\docguide\style.md`
