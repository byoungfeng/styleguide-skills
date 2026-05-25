# Google HTML/CSS Style Guide — Deep Reference

## 1. Background

This guide defines coding standards for HTML and CSS across Google. It applies to all HTML, CSS, Sass, and GSS source files. Following a consistent style improves readability, maintainability, and reduces the risk of bugs.

---

## 2. General Style Rules

### 2.1 Protocol

Use HTTPS for embedded resources. HTTPS provides security, privacy, and is the recommended protocol for all web content.

```html
<!-- Good -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>

<!-- Bad -->
<script src="http://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
```

### 2.2 Indentation

Indent by 2 spaces at a time. Do not use tabs. This ensures consistent rendering across all editors.

```html
<!-- Good -->
<div>
  <span>text</span>
</div>

<!-- Bad -->
<div>
    <span>text</span>
</div>
```

```css
/* Good */
.selector {
  property: value;
}

/* Bad */
.selector {
    property: value;
}
```

### 2.3 Capitalization

Use only lowercase. HTML element names, attributes, attribute values, CSS selectors, properties, and property values must be lowercase.

```html
<!-- Good -->
<a href="https://example.com">link</a>

<!-- Bad -->
<A HREF="https://example.com">link</A>
```

```css
/* Good */
color: #f00;

/* Bad */
COLOR: #F00;
```

### 2.4 Trailing Whitespace

Remove trailing whitespace. Empty lines should not contain spaces.

### 2.5 Encoding

Use UTF-8 (no BOM). Specify the encoding in HTML templates and documents via `<meta charset="utf-8">`.

```html
<!-- Good -->
<meta charset="utf-8">

<!-- Bad -->
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
```

### 2.6 Comments

Explain code as needed. Use `TODO:` for action items.

```css
/* TODO: add fallback for older browsers */
```

```html
<!-- TODO: replace placeholder image -->
<img src="placeholder.jpg" alt="user avatar">
```

---

## 3. HTML Style Rules

### 3.1 Document Type

Use `<!doctype html>` to enable no-quirks mode.

```html
<!-- Good -->
<!doctype html>
<html lang="en">
  ...
</html>
```

### 3.2 HTML Validity

Use valid HTML where possible. Use tools like the W3C HTML Validator.

```html
<!-- Good -->
<!doctype html>
<meta charset="utf-8">
<article>This is valid.</article>

<!-- Bad -->
<!doctype html>
<meta charset="utf-8">
<article>This is not <.</article>
```

### 3.3 Semantics

Use HTML elements for their intended purpose. This improves accessibility and maintainability.

```html
<!-- Good -->
<a href="https://example.com">Visit Example</a>
<h1>Page Title</h1>

<!-- Bad -->
<div onclick="location.href='https://example.com'">Visit Example</div>
<div class="heading">Page Title</div>
```

### 3.4 Multimedia Fallback

Provide alternative content for multimedia elements.

```html
<!-- Good -->
<img src="avatar.jpg" alt="User profile picture">

<!-- Bad -->
<img src="avatar.jpg">
```

For video and audio, provide captions and transcripts.

### 3.5 Separation of Concerns

Keep structure (HTML), presentation (CSS), and behavior (JavaScript) separate. Inline styles and scripts should be avoided.

```html
<!-- Good -->
<link rel="stylesheet" href="style.css">
<script src="app.js"></script>

<!-- Bad -->
<div style="color: red">text</div>
<script>alert('hi')</script>
```

### 3.6 Entity References

Do not use entity references like `&mdash;`, `&rdquo;`, or `&#x260e;`. Use actual UTF-8 characters. Only escape `&` and `<` (and `>` , `"`, `'` as needed for attribute values).

```html
<!-- Good -->
<p>Curly "quotes" — em dash • bullet</p>

<!-- Bad -->
<p>Curly &ldquo;quotes&rdquo; &mdash; em dash &#x2022; bullet</p>
```

### 3.7 Optional Tags

Omit optional tags to reduce file size and improve parseability.

```html
<!-- Good -->
<!doctype html>
<meta charset="utf-8">
<title>Page</title>
<p>Content

<!-- Bad -->
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Page</title>
</head>
<body>
  <p>Content</p>
</body>
</html>
```

### 3.8 `type` Attributes

Omit `type` attributes for stylesheets and scripts. HTML5 treats `text/css` and `text/javascript` as defaults.

```html
<!-- Good -->
<link rel="stylesheet" href="style.css">
<script src="app.js"></script>

<!-- Bad -->
<link rel="stylesheet" type="text/css" href="style.css">
<script src="app.js" type="text/javascript"></script>
```

### 3.9 `id` Attributes

Avoid unnecessary `id` attributes. Prefer classes for styling and data attributes for scripting. If an `id` is required, include at least one hyphen for JavaScript safety (avoids matching DOM API methods like `submit`, `focus`, etc.).

```html
<!-- Good -->
<div class="panel" data-expanded="true">...</div>

<!-- Good (with id, includes hyphen) -->
<input id="search-box" type="text">

<!-- Bad -->
<div id="panel">...</div>
<div id="submit">...</div>
```

### 3.10 HTML Formatting

Use a new line for every block, list, or table element. Indent children.

```html
<!-- Good -->
<blockquote>
  <p>Quote text here.</p>
</blockquote>

<ul>
  <li>Item one</li>
  <li>Item two</li>
</ul>

<!-- Bad -->
<blockquote><p>Quote text here.</p></blockquote>

<ul><li>Item one</li><li>Item two</li></ul>
```

### 3.11 HTML Line-Wrapping

Break long lines at a reasonable column (optional). When wrapping, indent wrapped lines.

```html
<!-- Good -->
<button
    class="btn btn-primary"
    type="submit"
    data-action="save">
  Save
</button>
```

### 3.12 HTML Quoting

Use double quotes `""` for attribute values.

```html
<!-- Good -->
<a href="https://example.com" class="link">Link</a>

<!-- Bad -->
<a href='https://example.com' class=link>Link</a>
```

---

## 4. CSS Style Rules

### 4.1 CSS Validity

Use valid CSS. Use a validator tool.

```css
/* Good */
.alert {
  color: #c00;
  font-weight: bold;
}
```

### 4.2 Class Naming

Use meaningful, generic class names. Be as short as possible while remaining descriptive.

```css
/* Good */
.nav { }
.author { }
.avatar { }

/* Bad */
.s-926 { }
.green { }
.blue { }
```

### 4.3 Class Name Style

Use hyphen-delimited names. For large projects, prefix class names.

```css
/* Good */
.video-title { }
.g-reply { }

/* Bad */
.videotitle { }
.video_title { }
```

### 4.4 Type Selectors

Avoid qualifying class names with type selectors. Prefer `.error` over `div.error`.

```css
/* Good */
.error { color: #c00; }

/* Bad */
div.error { color: #c00; }
```

### 4.5 ID Selectors

Avoid ID selectors in CSS. They are overly specific and hard to reuse. Prefer classes.

```css
/* Good */
.highlight { background: #ff0; }

/* Bad */
#highlight { background: #ff0; }
```

### 4.6 Shorthand Properties

Use shorthand properties where possible for brevity.

```css
/* Good */
.box {
  border: 1px solid #000;
  margin: 10px;
  padding: 0 5px;
  font: 1em/1.5 sans-serif;
}

/* Bad */
.box {
  border-width: 1px;
  border-style: solid;
  border-color: #000;
  margin-top: 10px;
  margin-right: 10px;
  margin-bottom: 10px;
  margin-left: 10px;
}
```

### 4.7 0 and Units

Omit units after `0` values unless required (e.g., transition duration).

```css
/* Good */
margin: 0;
padding: 0 10px;

/* Bad */
margin: 0px;
padding: 0em 10px;
```

### 4.8 Leading 0

Include a leading `0` in decimal values.

```css
/* Good */
font-size: 0.8em;
opacity: 0.5;

/* Bad */
font-size: .8em;
opacity: .5;
```

### 4.9 Hexadecimal Notation

Use 3-character hexadecimal notation where possible.

```css
/* Good */
color: #f00;
background: #fff;

/* Bad */
color: #ff0000;
background: #ffffff;
```

### 4.10 `!important`

Avoid `!important`. It breaks the natural cascade and makes debugging difficult.

```css
/* Good */
.error {
  color: #c00;
}

/* Bad */
.error {
  color: #c00 !important;
}
```

### 4.11 Hacks

Avoid user-agent detection and CSS hacks. Use graceful degradation or feature detection instead.

```css
/* Good */
@supports (display: grid) {
  .container { display: grid; }
}

/* Bad */
* html .container { display: grid; } /* IE6 hack */
```

---

## 5. CSS Formatting Rules

### 5.1 Declaration Order

Alphabetize declarations within a rule set (optional but recommended). Vendor prefixes should be grouped at the top.

```css
/* Good */
.selector {
  -webkit-transition: all 0.2s;
  background: #fff;
  border: 1px solid #000;
  color: #333;
  display: block;
  margin: 10px;
  padding: 5px;
}

/* Bad */
.selector {
  padding: 5px;
  margin: 10px;
  background: #fff;
  color: #333;
  border: 1px solid #000;
  display: block;
}
```

### 5.2 Block Content Indentation

Indent all block content — selectors and declarations — inside a rule.

```css
/* Good */
.media {
  overflow: hidden;
}

.media-body {
  float: left;
}
```

### 5.3 Declaration Stops

End every declaration with a semicolon.

```css
/* Good */
.selector {
  color: #333;
  margin: 0;
}

/* Bad */
.selector {
  color: #333;
  margin: 0
}
```

### 5.4 Property Name Stops

Always use a space after a colon in property declarations.

```css
/* Good */
font-weight: bold;

/* Bad */
font-weight:bold;
```

### 5.5 Selector and Declaration Separation

Always use a space before the opening brace `{`.

```css
/* Good */
.video { width: 100%; }

/* Bad */
.video{width:100%;}
```

### 5.6 Rule Separation

Put one selector per line for multi-selector rules. Put each declaration on its own line.

```css
/* Good */
.author,
.date {
  color: #666;
  font-size: 0.8em;
}

/* Bad */
.author, .date {
  color: #666; font-size: 0.8em;
}
```

### 5.7 Blank Lines

Separate rule sets with a blank line.

```css
/* Good */
.first {
  color: #000;
}

.second {
  color: #333;
}

/* Bad */
.first {
  color: #000;
}
.second {
  color: #333;
}
```

### 5.8 Quoting

Use single `'` quotes for attribute selectors and property values. Do not use quotes in `url()`.

```css
/* Good */
input[type='text'] { }
background: url(logo.png);
font-family: 'Open Sans', sans-serif;

/* Bad */
input[type="text"] { }
background: url('logo.png');
font-family: "Open Sans", sans-serif;
```

### 5.9 Section Comments

Optionally group related styles with section comments.

```css
/* Header */
.header { ... }
.header-title { ... }

/* Footer */
.footer { ... }
.footer-link { ... }
```

---

## 6. Parting Words

**Be consistent.** If you're editing existing code, follow its local conventions even if they differ from this guide. Consistency within a file or project matters more than strict adherence to any single style guide.

---

Full source: `G:\styleguide\htmlcssguide.html`
