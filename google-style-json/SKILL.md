---
name: google-style-json
description: Google JSON Style Guide — property naming, data types, structure, and reserved property names for JSON APIs. Use when designing or reviewing JSON API requests/responses to apply Google's standards. Triggers on any JSON API design task.
---

# Google JSON Style Guide — Quick Reference

## General

- No comments in JSON objects
- Double quotes for all property names and string values
- Flattened data preferred over arbitrary structured hierarchy

## Property Names

- Meaningful names with defined semantics
- camelCase ASCII strings
- First char: letter, underscore, or dollar sign
- Subsequent chars: letter, digit, underscore, dollar sign
- Avoid reserved JavaScript keywords
- Maps can use any Unicode in keys (document when used as map)
- Array types: plural names; others: singular
- Avoid naming conflicts via new names or API versioning

## Property Values

- Must be: boolean, number, string, object, array, or null
- No JavaScript expressions or functions
- Consider removing empty/null optional properties
- Enum values as strings

## Data Types

- Dates: RFC 3339 format (e.g., `"2007-11-06T16:34:41.000Z"`)
- Time durations: ISO 8601 (e.g., `"P3Y6M4DT12H30M5S"`)
- Lat/Long: ISO 6709 (e.g., `"+40.6894-074.0447"`)

## Reserved Properties (top-level)

- `apiVersion` (string): API version
- `context` (string): Client echo for correlation
- `id` (string): Server-supplied identifier
- `method` (string): Operation to perform
- `params` (object): RPC input parameters
- `data` (object): Successful response payload
- `error` (object): Error response payload (never both data and error)

## data object reserved properties

- `kind`, `fields`, `etag`, `id`, `lang`, `updated`, `deleted`
- Pagination: `currentItemCount`, `itemsPerPage`, `startIndex`, `totalItems`, `pageIndex`, `totalPages`
- Links: `selfLink`, `nextLink`, `previousLink`, `editLink`
- Items: `items` (array of data objects)

## error object reserved properties

- `code` (integer): Error code
- `message` (string): Error message
- `errors` (array): Detailed errors with `domain`, `reason`, `message`, `location`, `locationType`, `extendedHelp`, `sendReport`

Link to full source: `G:\styleguide\jsoncstyleguide.xml`
