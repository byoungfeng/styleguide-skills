---
name: google-style-xml
description: Google XML Document Format Style Guide — schema design, naming, namespaces, elements vs attributes, and formatting for XML document formats. Use when designing or reviewing XML document format schemas to apply Google's standards. Triggers on any XML format design task.
---

# Google XML Document Format Style Guide — Quick Reference

## General
- **Reuse** existing formats whenever possible. Do not design a new format if an existing one suffices.
- When extending an existing format, **follow its implicit style** rather than overriding it with this guide's rules.

## Schemas
- Express schemas in **RELAX NG Compact Syntax** (`.rnc`).
- Use **Salami Slice** style — one rule per element, no nested rules.
- Use `pattern` for complex value constraints (regex).
- May also provide a DTD or W3C XML Schema for compatibility tooling.

## Namespaces
- Element names **MUST** be in a namespace.
- Prefer **default namespace** (`xmlns="..."`) to avoid prefix repetition.
- Attribute names **SHOULD NOT** be in a namespace (except `xml:` and `xsi:`).
- Namespace URI format: `https://example.com/name/year`
  - Must not change unless semantics change incompatibly.
- Prefixes: **short**, **lowercase ASCII**, **no single-letter prefixes**.

## Naming (`lowerCamelCase`)
- Elements, attributes, and enumerated values: **`lowerCamelCase`**
- ASCII characters only; max **25 characters**.
- No ad hoc abbreviations.
- Acronyms treated as words: `informationUri` not `informationURI`.

## Elements
- No mixed content (elements must not contain both text and child elements).
- Wrapping elements for repeating children SHOULD NOT be used (use multiple direct children instead).

## Attributes
- Order is not significant.
- Max ~10 attributes per element.
- No line breaks in attribute values.
- Allow both single and double quotes.

## Values
- Numeric types: `xsd:int`, `xsd:long`, `xsd:double` (base 10).
- Avoid booleans — prefer enumerations (`yes`/`no`, `true`/`false`).
- Dates: **RFC 3339**.
- Binary: Base64 encoded.

## Key-Value Pairs
- Use an empty element with a `value` attribute; optionally a `unit` attribute.
- For unbounded keys: use `key`, `value`, `unit`, `scheme` attributes.

## Processing Instructions
- Avoid creating new processing instructions.

## Document Instances
- **UTF-8** encoding.
- Namespaces declared on the **root element**.
- Consistent prefix mapping throughout.
- **2-space indentation** per nesting level.
- Empty tags: prefer `<tag/>` over `<tag></tag>`.
- Comments must not carry data semantics.
- Only standard entity references: `&amp;`, `&lt;`, `&gt;`, `&quot;`, `&apos;`.

## Full Source
`G:\styleguide\xmlstyle.html`
