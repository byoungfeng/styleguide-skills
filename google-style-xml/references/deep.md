# Google XML Document Format Style Guide — Deep Reference

> Full source: `G:\styleguide\xmlstyle.html`

---

## 1. Introduction

This style guide defines standards for **XML document formats** — XML-based data formats intended for machine-to-machine communication or structured data interchange. It is **not** intended for rich-text document formats such as XHTML, DocBook, or SVG, which have their own established conventions.

Goals of the guide:
- Consistency across Google XML document formats
- Interoperability and long-term readability
- Alignment with industry best practices

The guide covers schema design, naming conventions, namespaces, elements versus attributes, values and data types, and the representation of XML document instances.

---

## 2. To Design or Not to Design

Before creating a new XML format, exhaustively search for existing formats that solve the same problem. Reuse is always preferred over reinvention.

- If an existing standard (IETF RFC, W3C Recommendation, ISO standard, industry de facto) already covers your domain, adopt it.
- If you must extend an existing format, **follow the implicit style of that format** rather than overriding it with this guide. Consistency with the extended format takes precedence.
- When designing a wholly new format, apply this guide in full.

---

## 3. Schemas

### 3.1 Primary Schema Language

Express schemas in **RELAX NG Compact Syntax** (`.rnc` file extension). RELAX NG is chosen for:
- Readability and conciseness
- Strong datatyping via XML Schema Datatypes
- Composability and modularity

### 3.2 Schema Styles

| Style | Description | Recommendation |
|---|---|---|
| **Salami Slice** | One grammar rule per element, defined at the top level. No nesting of element declarations. | **Preferred** — maximizes reusability and readability. |
| Russian Doll | Element declarations nested inside their parent. Compact but hard to reuse. | Avoid. |
| Venetian Blind | Named pattern/parameter-entity definitions with a single top-level element. | Acceptable for compatibility DTDs. |

### 3.3 Example — Salami Slice (`.rnc`)

```xml
namespace atom = "https://www.w3.org/2005/Atom"
start = feed

feed = element feed {
  atom:author?,
  atom:category*,
  atom:contributor*,
  atom:generator?,
  atom:icon?,
  atom:id,
  atom:link*,
  atom:logo?,
  atom:rights?,
  atom:subtitle?,
  atom:title,
  atom:updated,
  entry*
}

entry = element entry { ... }
author = element author { atom:name, atom:uri?, atom:email? }
```

### 3.4 Complex Values with Regex

```xml
uri = element uri { xsd:anyURI }
email = element email {
  xsd:string { pattern = "[^@]+@[^@]+\.[^@]+" }
}
```

### 3.5 Compatibility Schemas

You may additionally provide:
- **DTD** (`.dtd`) — for legacy tooling compatibility
- **W3C XML Schema** (`.xsd`) — for tooling that does not support RELAX NG

When providing multiple schemas, the RELAX NG Compact version is the normative one.

---

## 4. Namespaces

### 4.1 Element Names Must Be in a Namespace

All element names **MUST** be qualified with a namespace. Unqualified element names are not permitted.

### 4.2 Default Namespace

Prefer the **default namespace** (`xmlns="..."`) for the primary vocabulary. This avoids repeating prefixes on every element.

```xml
<feed xmlns="https://example.com/atom/2025">
  <title>Example Feed</title>
</feed>
```

### 4.3 Attributes Should Not Be in a Namespace

Attribute names **SHOULD NOT** be in a namespace unless they belong to `xml:` (e.g., `xml:lang`, `xml:base`) or `xsi:` (e.g., `xsi:type`, `xsi:nil`, `xsi:schemaLocation`).

```xml
<!-- Correct -->
<entry xml:lang="en" xsi:type="atom:entry">

<!-- Incorrect — custom attribute in namespace -->
<entry my:attr="value">
```

### 4.4 Namespace URI Format

```
https://example.com/name/year
```

Rules:
- **Must not change** once published, unless the semantics of the vocabulary change in an incompatible way.
- Changing the namespace indicates a breaking change.
- Use the year of first publication or the latest breaking revision.

### 4.5 Namespace Prefix Rules

- **Short** — typically 2–5 characters
- **Lowercase ASCII only**
- **No single-letter prefixes** (except `x` for XML Schema and `a` for Atom)
- Must be mnemonic where possible

```xml
<!-- Good -->
xmlns:atom="https://www.w3.org/2005/Atom"
xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"

<!-- Bad -->
xmlns:a="https://example.com/ns"
xmlns:ATOM="https://www.w3.org/2005/Atom"
```

---

## 5. Names and Enumerated Values

### 5.1 lowerCamelCase

Element names, attribute names, and enumerated value strings use **lowerCamelCase**:

```xml
<lastModified>2025-01-15</lastModified>
<informationUri>https://example.com</informationUri>
```

### 5.2 Character Restrictions

- **ASCII only** — no Unicode letters outside ASCII range
- **Maximum 25 characters** per name
- **No ad hoc abbreviations** — `informationUri` not `infoUri`, `documentIdentifier` not `docId`

### 5.3 Acronyms

Acronyms are treated as regular words — only the first letter is lowercased:

| Correct | Incorrect |
|---|---|
| `informationUri` | `informationURI` |
| `htmlContent` | `HTMLContent` |
| `url` | `URL` |
| `xmlSchema` | `XMLSchema` |

### 5.4 Enumerated Values

```xml
<status>published</status>
<!-- Not: <status>Published</status> -->
<!-- Not: <status>PUBLISHED</status> -->
```

Allowed values for boolean-style attributes (when enums are used over actual booleans):
- `yes` / `no`
- `true` / `false`

---

## 6. Elements

### 6.1 No Mixed Content

Elements must contain **either** text **or** child elements, **not both**. Mixed content models are reserved for rich-text document formats and are prohibited in document format XML.

```xml
<!-- Correct: element contains only text -->
<title>My Title</title>

<!-- Correct: element contains only children -->
<author>
  <name>Jane Doe</name>
  <email>jane@example.com</email>
</author>

<!-- Incorrect: mixed content -->
<description>See the <link>details</link> for more.</description>
```

### 6.2 No Wrapping Elements for Repeating Children

When an element can appear multiple times, children repeat directly — do not introduce a wrapper element.

```xml
<!-- Correct -->
<feed>
  <entry>...</entry>
  <entry>...</entry>
  <entry>...</entry>
</feed>

<!-- Incorrect -->
<feed>
  <entries>
    <entry>...</entry>
    <entry>...</entry>
  </entries>
</feed>
```

---

## 7. Attributes

### 7.1 Order

Attribute order is **not significant**. Do not rely on attribute ordering in any processing logic.

### 7.2 Number of Attributes

Aim for a **maximum of ~10 attributes** per element. If an element requires more than 10 attributes, consider whether some should be child elements instead.

### 7.3 Line Breaks

Attribute values **MUST NOT** contain line breaks.

### 7.4 Quotes

Allow **both single and double quotes** in attribute values. Either style is acceptable:

```xml
<entry id="tag:example.com,2025:12345"/>
<entry id='tag:example.com,2025:12345'/>
```

If the value contains the quote character used for delimiters, use the other quote type or escape with `&quot;` or `&apos;`.

---

## 8. Values

### 8.1 Numeric Types

| Schema Type | Usage |
|---|---|
| `xsd:int` | 32-bit signed integer (−2,147,483,648 to 2,147,483,647) |
| `xsd:long` | 64-bit signed integer (−9,223,372,036,854,775,808 to 9,223,372,036,854,775,807) |
| `xsd:double` | 64-bit IEEE 754 floating point (base 10 representation in XML) |

All numeric values use **base 10**. No hexadecimal, octal, or other radices.

```xml
<count xsi:type="xsd:int">42</count>
<distance xsi:type="xsd:double">3.14159</distance>
```

### 8.2 Booleans

Avoid `xsd:boolean`. Use **enumerations** instead with explicit named values:

```xml
<enabled>true</enabled>
<enabled>false</enabled>
```

If `xsd:boolean` is unavoidable, only `true` and `false` (the lexical representations) may be used — never `1` or `0`.

### 8.3 Dates and Times

All date/time values MUST follow **RFC 3339** (a profile of ISO 8601):

```xml
<updated>2025-01-15T14:30:00Z</updated>
<published>2025-01-15T14:30:00-05:00</published>
<lastModified>2025-01-15</lastModified>
```

### 8.4 No Embedded Syntax

Element and attribute values must not contain embedded XML, HTML, or other structured syntax unless the schema explicitly defines the value as such (e.g., an XHTML content element).

### 8.5 Whitespace Handling

| `xsd` facet | Default handling |
|---|---|
| `xsd:string` | Preserve whitespace (not collapsed) |
| `xsd:token` | Collapse whitespace (trim + internal sequences to single space) |
| `xsd:normalizedString` | Normalize (replace line breaks and tabs with spaces) |

Choose the type that matches the semantic whitespace behavior of the value.

---

## 9. Key-Value Pairs

### 9.1 Simple Key-Value Pattern

Use an **empty element** with a `value` attribute, and optionally a `unit` attribute:

```xml
<length value="42" unit="cm"/>
<temperature value="98.6" unit="F"/>
```

### 9.2 Generic/Bounded Keys

For unbounded key-value sets or application-defined keys, use the following attributes on an empty element:

| Attribute | Description | Required |
|---|---|---|
| `key` | The key name | Yes |
| `value` | The value | Yes |
| `unit` | Unit of measurement | No |
| `scheme` | Disambiguating scheme/registry | No |

```xml
<property key="color" value="red"/>
<property key="height" value="180" unit="cm" scheme="https://example.org/units/height"/>
```

---

## 10. Binary Data

- All binary data must be **Base64 encoded**.
- Line breaks within the Base64 value are **optional** (the decoder must handle both).
- Declare the type explicitly with `xsi:type="xs:base64Binary"`.

```xml
<thumbnail xsi:type="xs:base64Binary">
  iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42m
  Nk+M9QDwADhgGAItRjYQAAAABJRU5ErkJggg==
</thumbnail>
```

XS: Use `xs:` (XML Schema namespace prefix) or `xsd:` consistently throughout the document.

---

## 11. Processing Instructions

- **Avoid creating new processing instructions** in XML document formats.
- Processing instructions are a legacy mechanism and should not be used for new designs.
- If you encounter an existing format that uses PIs, preserve their semantics but do not add new ones.

```xml
<!-- Avoid -->
<?myapp-version 2.0?>
<doc>...</doc>
```

---

## 12. Representation of XML Document Instances

### 12.1 Encoding

All XML documents **MUST** use **UTF-8** encoding.

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

### 12.2 Namespace Declaration

All namespaces MUST be declared on the **root element** — not on nested elements — unless dynamic namespace usage requires otherwise.

### 12.3 Consistent Prefix Mapping

The same namespace prefix MUST map to the same namespace URI throughout the document. No rebinding of prefixes.

```xml
<!-- Correct -->
<feed xmlns="https://example.com/feed/2025"
      xmlns:dc="https://purl.org/dc/elements/1.1/">
  <title>...</title>
  <dc:creator>...</dc:creator>
  <dc:creator>...</dc:creator>
</feed>
```

### 12.4 Indentation

Use **2 spaces** per indentation level. No tabs.

```xml
<feed>
  <entry>
    <title>Entry One</title>
  </entry>
  <entry>
    <title>Entry Two</title>
  </entry>
</feed>
```

### 12.5 Empty Tags

Prefer the self-closing tag syntax for empty elements:

```xml
<!-- Preferred -->
<br/>

<!-- Acceptable but not preferred -->
<br></br>
```

### 12.6 Attribute Quoting

Attribute values may use either single or double quotes. Be consistent within a document when possible.

### 12.7 Comments

Comments must not carry data semantics. No application logic should depend on comment content.

```xml
<feed>
  <!-- This is a human-readable note -->
  <entry>...</entry>
</feed>
```

### 12.8 CDATA Sections

CDATA sections are **allowed** but not required. They are useful when element text contains characters that would otherwise need escaping.

```xml
<code><![CDATA[if (a < b) { return x; }]]></code>
```

### 12.9 Entity References

Only the five standard XML entity references may be used:

| Entity | Character |
|---|---|
| `&amp;` | `&` |
| `&lt;` | `<` |
| `&gt;` | `>` |
| `&quot;` | `"` |
| `&apos;` | `'` |

Do not introduce custom entity references. Use numeric character references (`&#x...;` or `&#...;`) as a last resort.

---

## 13. Elements vs Attributes

### 13.1 General Principle

Elements express **structure and hierarchy**; attributes express **properties and metadata** of their parent element. This is a guideline, not a hard rule — context matters.

### 13.2 When to Use Elements

- Data that has or may later have substructure (child elements)
- Data that can appear multiple times (repeating values)
- Data whose type may change between text and structured in future versions
- Long text values
- Values that may need internationalization (`xml:lang`)

### 13.3 When to Use Attributes

- Simple, atomic, single-occurrence metadata about an element
- Identifiers, references, and links
- Language, base URI, and other `xml:` space attributes
- Values from a constrained enumeration
- Short values unlikely to ever need substructure

### 13.4 Examples

```xml
<!-- Element for structured data -->
<author>
  <name>Jane Doe</name>
  <email>jane@example.com</email>
</author>

<!-- Attribute for simple metadata -->
<entry xml:base="https://example.com/" xml:lang="en">
```

```xml
<!-- Repeating values → elements -->
<category term="tech"/>
<category term="science"/>

<!-- Single atomic property → attribute (open to interpretation) -->
<person age="29">...</person>
```

---

## 14. Review Checklist

Before finalizing an XML document format, verify:

- [ ] Does an existing format already solve this problem?
- [ ] If extending, does the design follow the extended format's style?
- [ ] Is the normative schema in RELAX NG Compact Syntax (Salami Slice style)?
- [ ] Are all element names in a namespace (default namespace preferred)?
- [ ] Are attribute names *not* in a namespace (except `xml:` / `xsi:`)?
- [ ] Does the namespace URI follow `https://example.com/name/year` and is it stable?
- [ ] Are all names `lowerCamelCase`, ASCII, ≤25 chars, with acronyms as words?
- [ ] Are there no mixed content models?
- [ ] Are wrapping elements for repeating children avoided?
- [ ] Does no element exceed ~10 attributes?
- [ ] Are booleans avoided in favor of enums?
- [ ] Are dates in RFC 3339 format?
- [ ] Are binary values Base64 encoded?
- [ ] Are there any new processing instructions?
- [ ] Does the instance representation use UTF-8, 2-space indent, root-level namespace declarations, and consistent prefix mapping?
- [ ] Are only standard entity references used?

---

*Full source: `G:\styleguide\xmlstyle.html`*
