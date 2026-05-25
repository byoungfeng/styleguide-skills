# Google JSON Style Guide — Deep Reference

## 1. Introduction

The Google JSON Style Guide defines conventions for building JSON APIs at Google. JSON (JavaScript Object Notation) is a lightweight data-interchange format defined at json.org. This guide applies to both RPC-style and REST-style APIs that use JSON as their wire format. Following these conventions promotes consistency, interoperability, and clarity across services.

## 2. Definitions

- **Property**: A name/value pair inside a JSON object. Also referred to as a field or member.
- **Property Name**: The string on the left side of the colon in a property. Must be unique within the enclosing object.
- **Property Value**: The value on the right side of the colon. May be any valid JSON type.
- **Number**: All numeric values in JSON are represented as JavaScript numbers. There is no built-in integer type; a "number" may be an integer or a floating-point value. Implementations that require a true integer distinction should document this convention separately.

## 3. General Guidelines

### No Comments

JSON is a data format, not a programming language. Comments are not part of the JSON specification and must not be included in JSON payloads. Any metadata about a property should be conveyed via documentation or schema definitions, not inline comments.

### Double Quotes Required

All property names and string values must be enclosed in double quotation marks. Single quotes and unquoted identifiers are not valid JSON.

```json
{
  "name": "example",
  "count": 42
}
```

### Flattened vs. Structured Hierarchy

Prefer flat data structures over deeply nested hierarchies. Deep nesting increases parsing complexity and makes individual properties harder to access.

**Preferred — flattened:**

```json
{
  "id": "12345",
  "firstName": "John",
  "lastName": "Doe",
  "city": "New York",
  "state": "NY"
}
```

**Avoid — over-structured:**

```json
{
  "id": "12345",
  "name": {
    "first": "John",
    "last": "Doe"
  },
  "address": {
    "city": "New York",
    "state": "NY"
  }
}
```

However, grouping related properties into a sub-object is acceptable when the grouping has clear semantic meaning, improves readability, or corresponds to a separate resource. Use judgment: flatten by default and nest only when it adds value.

## 4. Property Name Guidelines

### Meaningful Names

Property names must be descriptive and convey the semantics of the value they contain. Abbreviations should be avoided unless they are universally understood (e.g., `id`, `url`).

### camelCase ASCII

All property names should use camelCase. The first character must be a letter, underscore (`_`), or dollar sign (`$`). Subsequent characters may be letters, digits, underscores, or dollar signs.

```json
{
  "firstName": "Alice",
  "lastName": "Smith",
  "someProperty": 100
}
```

### JavaScript Identifier Compatibility

Property names should be valid JavaScript identifiers so they can be accessed with dot notation (e.g., `obj.firstName`). If a property name contains characters that are not valid in a JavaScript identifier, consumers must use bracket notation (`obj["non-standard-name"]`). Avoid such names unless necessary (e.g., map keys).

### Key Names in JSON Maps

When a JSON object is used as a map (i.e., keys are dynamic and not known in advance), the key strings may contain any Unicode characters. This is an exception to the ASCII-only guideline. Such usage should be clearly documented.

```json
{
  "München": { "population": 1500000 },
  "東京":     { "population": 13960000 }
}
```

### Reserved Property Names

Certain property names are reserved at the top level and within sub-objects. These are listed in subsequent sections. Avoid using reserved names for application-specific properties to prevent ambiguity.

### Singular vs. Plural Naming

- Properties whose values are arrays should use plural names (e.g., `items`, `users`, `errors`).
- All other properties should use singular names (e.g., `id`, `name`, `totalItems`).

### Naming Conflicts (Versioning)

If a property must change its semantics in a way that breaks backward compatibility, either introduce a new property with a different name (deprecating the old one) or use API versioning. Do not reuse the same property name with different semantics across versions.

## 5. Property Value Guidelines

### Valid Types

A JSON value must be one of:
- `boolean`
- `number`
- `string`
- `object` (JSON object)
- `array`
- `null`

Types like `Date`, `Map`, `Set`, or `Function` do not exist in JSON. Language-specific types must be serialised into one of the above types (e.g., a Date becomes an RFC 3339 string).

### No JavaScript Expressions

JSON is a data format. Do not include JavaScript expressions, function calls, or executable code in values.

```json
{
  "callback": "someFunction()"
}
```

The above is technically valid JSON but violates the intent of the guide if the value is meant to be executed.

### Empty and Null Values

Consider omitting optional properties whose value would be `null` or empty (`""`, `[]`, `{}`) rather than including them. This reduces payload size and avoids ambiguity between "not provided" and "empty". If the distinction between absent and empty is semantically important, keep the property and document the convention.

### Enum Values as Strings

Enumerated values should be represented as strings, not numbers.

```json
{
  "status": "active"
}
```

## 6. Property Value Data Types

### Dates — RFC 3339

Dates and timestamps must use the RFC 3339 format, which is a profile of ISO 8601. The format is:

```
YYYY-MM-DDTHH:MM:SS[.sss]Z
```

- `Z` indicates UTC. Offsets like `+05:30` are also permitted.
- Fractional seconds are optional but recommended.

```json
{
  "createdAt": "2007-11-06T16:34:41.000Z",
  "updatedAt": "2024-01-15T09:30:00Z"
}
```

### Time Durations — ISO 8601

Durations must follow ISO 8601 duration format:

```
P[n]Y[n]M[n]DT[n]H[n]M[n]S
```

- `P` marks the start of the duration.
- `Y` = years, `M` = months (after `P`), `D` = days.
- `T` separates date from time.
- `H` = hours, `M` = minutes (after `T`), `S` = seconds.

```json
{
  "duration": "P3Y6M4DT12H30M5S",
  "timeout": "PT30S",
  "lease": "P1D"
}
```

### Latitude / Longitude — ISO 6709

Geographic coordinates must follow ISO 6709 in the form:

```
±DD.DDDD±DDD.DDDD
```

- Latitude first, then longitude.
- Positive values should be prefixed with `+`.
- Negative values use `-`.
- No comma or space between the two values.

```json
{
  "location": "+40.6894-074.0447"
}
```

Coordinates for the Statue of Liberty in the example above: +40.6894 (latitude), -074.0447 (longitude).

## 7. JSON Structure & Reserved Property Names

The top-level JSON object uses reserved property names to define the envelope of an API request or response. The following schema (written in Orderly format) defines the structure:

```
object {
  string apiVersion?;
  string context?;
  string id?;
  string method?;
  object params?;
  object data?;
  object error?;
}
```

- `data` and `error` are mutually exclusive. A response must contain either a `data` object (success) or an `error` object (failure), but never both.
- All top-level reserved properties are optional at the protocol level, though individual APIs may require certain fields (e.g., `method` in a request, `id` in a response).

## 8. Top-Level Reserved Properties

### apiVersion (string)

The version of the API. Should be a string to accommodate version schemes like `"v1"`, `"v2.1"`, `"2024-01-01"`.

```json
{
  "apiVersion": "v2"
}
```

### context (string)

An opaque string provided by the client and echoed unchanged by the server. Used for request correlation and debugging.

```json
{
  "context": "req-abc-123"
}
```

### id (string)

A server-supplied identifier for the request or response. Typically a UUID or other unique value.

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### method (string)

The operation to perform. Used in RPC-style APIs to indicate the endpoint or function name.

```json
{
  "method": "users.list"
}
```

### params (object)

Input parameters for the RPC method. The contents are API-specific.

```json
{
  "method": "users.list",
  "params": {
    "pageSize": 50,
    "filter": "status:active"
  }
}
```

### data (object)

The successful response payload. Contains the response data and reserved sub-properties (see Section 9).

```json
{
  "apiVersion": "v2",
  "id": "req-001",
  "data": {
    "kind": "user#list",
    "items": [
      { "id": "1", "name": "Alice" },
      { "id": "2", "name": "Bob" }
    ]
  }
}
```

### error (object)

The error response payload. Present only when the request fails. Never coexists with `data`.

```json
{
  "apiVersion": "v2",
  "id": "req-001",
  "error": {
    "code": 404,
    "message": "Resource not found"
  }
}
```

## 9. data Object Reserved Properties

### Standard Properties

| Property | Type | Description |
|----------|------|-------------|
| `kind` | string | A discriminator or resource type (e.g., `"user#list"`, `"tasks#item"`) |
| `fields` | string | Mask indicating which fields are present (used with partial responses) |
| `etag` | string | Entity tag for caching and conditional requests |
| `id` | string | Server-supplied identifier for the data resource |
| `lang` | string | Language tag (e.g., `"en-US"`, `"fr"`) |
| `updated` | string | Last update timestamp in RFC 3339 format |
| `deleted` | boolean | Indicates the resource was deleted |

```json
{
  "data": {
    "kind": "note#item",
    "id": "n-001",
    "lang": "en-US",
    "updated": "2024-06-01T12:00:00Z",
    "deleted": false
  }
}
```

### Pagination Properties

| Property | Type | Description |
|----------|------|-------------|
| `currentItemCount` | number | Number of items returned in this page |
| `itemsPerPage` | number | Maximum items per page requested/configured |
| `startIndex` | number | Index of the first item in this page (1-based) |
| `totalItems` | number | Total number of items across all pages |
| `pageIndex` | number | Current page index (0-based or 1-based, document convention) |
| `totalPages` | number | Total number of pages available |

```json
{
  "data": {
    "kind": "user#list",
    "items": [ /* ... */ ],
    "currentItemCount": 25,
    "itemsPerPage": 25,
    "startIndex": 1,
    "totalItems": 142,
    "pageIndex": 1,
    "totalPages": 6
  }
}
```

### Link Properties

| Property | Type | Description |
|----------|------|-------------|
| `selfLink` | string | URL to this resource |
| `nextLink` | string | URL to the next page |
| `previousLink` | string | URL to the previous page |
| `editLink` | string | URL to edit/update this resource |
| `pageLinkTemplate` | string | URL template for constructing page links |

```json
{
  "data": {
    "kind": "user#list",
    "items": [ /* ... */ ],
    "selfLink": "https://api.example.com/users?page=1",
    "nextLink": "https://api.example.com/users?page=2",
    "previousLink": null
  }
}
```

### Items Array

The `items` property contains an array of data objects. Each object in the array may itself contain any of the reserved properties listed above.

```json
{
  "data": {
    "kind": "user#list",
    "items": [
      {
        "kind": "user#item",
        "id": "u-001",
        "name": "Alice",
        "email": "alice@example.com"
      },
      {
        "kind": "user#item",
        "id": "u-002",
        "name": "Bob",
        "email": "bob@example.com"
      }
    ],
    "totalItems": 2
  }
}
```

## 10. error Object Reserved Properties

### Top-Level Error Properties

| Property | Type | Description |
|----------|------|-------------|
| `code` | integer | Numeric error code (e.g., HTTP status code or application-specific code) |
| `message` | string | Human-readable error message |

```json
{
  "error": {
    "code": 400,
    "message": "Invalid request parameters"
  }
}
```

### errors Array

The `errors` array provides detailed per-field or per-reason error information. Each element in the array is an object with the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `domain` | string | The domain or service where the error originated |
| `reason` | string | Machine-readable error reason identifier |
| `message` | string | Human-readable description of the specific error |
| `location` | string | The field or location that caused the error |
| `locationType` | string | Type of location (`"field"`, `"parameter"`, `"header"`, etc.) |
| `extendedHelp` | string | URL or reference to extended documentation |
| `sendReport` | string | URL or instructions for reporting the issue |

```json
{
  "error": {
    "code": 400,
    "message": "Validation failed",
    "errors": [
      {
        "domain": "global",
        "reason": "invalidParameter",
        "message": "Invalid value for 'email'",
        "location": "email",
        "locationType": "parameter",
        "extendedHelp": "https://api.example.com/docs/errors#invalidParameter",
        "sendReport": "https://api.example.com/errors/report"
      },
      {
        "domain": "global",
        "reason": "required",
        "message": "Required parameter 'name' is missing",
        "location": "name",
        "locationType": "parameter"
      }
    ]
  }
}
```

## 11. Examples

### Successful List Response (with pagination and links)

```json
{
  "apiVersion": "v2",
  "context": "client-trace-001",
  "id": "resp-abc-123",
  "data": {
    "kind": "book#list",
    "etag": "\"abc123def456\"",
    "items": [
      {
        "kind": "book#item",
        "id": "b-001",
        "title": "The Great Gatsby",
        "author": "F. Scott Fitzgerald",
        "published": "1925-04-10"
      },
      {
        "kind": "book#item",
        "id": "b-002",
        "title": "To Kill a Mockingbird",
        "author": "Harper Lee",
        "published": "1960-07-11"
      }
    ],
    "currentItemCount": 2,
    "itemsPerPage": 10,
    "startIndex": 1,
    "totalItems": 2,
    "totalPages": 1,
    "selfLink": "https://api.example.com/books",
    "editLink": "https://api.example.com/books"
  }
}
```

### Error Response

```json
{
  "apiVersion": "v2",
  "id": "resp-xyz-789",
  "error": {
    "code": 403,
    "message": "Forbidden",
    "errors": [
      {
        "domain": "global",
        "reason": "rateLimitExceeded",
        "message": "Rate limit exceeded. Retry after 60 seconds.",
        "location": "Authorization",
        "locationType": "header",
        "extendedHelp": "https://api.example.com/docs/rate-limiting"
      }
    ]
  }
}
```

### Minimal Successful Response

```json
{
  "data": {
    "kind": "status#item",
    "id": "health-1",
    "updated": "2024-06-15T08:30:00Z"
  }
}
```

---

Full source: `G:\styleguide\jsoncstyleguide.xml`
