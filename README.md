<h1 align="center">jsonld.mq</h1>

A [JSON-LD](https://www.w3.org/TR/json-ld11/) `<script type="application/ld+json">` extractor implemented as an [mq](https://github.com/harehare/mq) module.

Markdown nodes can't hold HTML attributes, and `<script>` tags don't survive HTML-to-Markdown conversion at all. This module reads JSON-LD blocks straight out of the raw HTML string instead, via mq's built-in `css_text` selector and `json` module, so it works upstream of (or instead of) any Markdown conversion.

## Features

- Extracts every `<script type="application/ld+json">` block's text content, in document order
- Parses each block with mq's built-in `json` module
- Flattens blocks whose top-level value is an array (a JSON-LD "graph" of several nodes serialized in one script) into individual nodes
- Filters nodes by `@type`

## Installation

Copy `jsonld.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp jsonld.mq ~/.local/mq/config/
```

## Usage

```sh
mq -L /path/to/modules -I raw \
  'import "jsonld" | jsonld::jsonld_parse(.)' page.html
```

If you copied it to the mq built-in module directory:

```sh
mq -I raw 'import "jsonld" | jsonld::jsonld_parse(.)' page.html
```

## API

### `jsonld_extract(html)`

Returns the raw (unparsed) text content of every `<script type="application/ld+json">` tag in `html`, in document order.

### `jsonld_parse(html)`

Parses every JSON-LD script block and returns the resulting values, in document order. A block may be a single object or an array of objects.

### `jsonld_nodes(html)`

Flattens `jsonld_parse(html)` into one array of individual JSON-LD nodes, expanding any script block whose top-level value is an array.

### `jsonld_nodes_by_type(html, type_name)`

Returns every node in `html` whose `@type` equals `type_name` (a plain string comparison — JSON-LD's full type-alias/context resolution is not implemented).

## Example

Given `page.html`:

```html
<html><head>
  <script type="application/ld+json">
    {"@context": "https://schema.org", "@type": "Article", "headline": "Hello World"}
  </script>
  <script type="application/ld+json">
    [{"@type": "Person", "name": "Alice"}, {"@type": "Person", "name": "Bob"}]
  </script>
</head></html>
```

```sh
mq -L . -I raw 'import "jsonld" | jsonld::jsonld_nodes_by_type(., "Person")' page.html
# => [{"@type": "Person", "name": "Alice"}, {"@type": "Person", "name": "Bob"}]

mq -L . -I raw 'import "jsonld" | jsonld::jsonld_nodes(.) | map(fn(n): n["@type"];)' page.html
# => ["Article", "Person", "Person"]
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.5 or later, built with the (default-enabled) `css-selector` feature for `css_text`.

## License

MIT
