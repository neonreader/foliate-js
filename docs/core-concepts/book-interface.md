# Book Interface

The Book Interface is the core abstraction that all e-book format parsers must implement. This interface provides a unified way to access book content, metadata, and navigation regardless of the underlying format.

## Overview

Every book format (EPUB, MOBI, FB2, etc.) is parsed into an object that implements the Book Interface. This allows the rendering components to work with any supported format without needing to know the specifics of each format.

## Required Properties

### `sections` (Array)

An array of section objects representing the content sections of the book. Each section must implement the following interface:

#### Section Properties

- **`load()`** (Function): Returns a string containing the URL that will be rendered. May be async.
- **`unload()`** (Function, optional): Called to free section resources. Returns nothing.
- **`createDocument()`** (Function, optional): Returns a `Document` object of the section. Used for searching. May be async.
- **`size`** (Number): The byte size of the section. Used for showing reading progress.
- **`linear`** (String, optional): If `"no"`, the section is not part of the linear reading sequence.
- **`cfi`** (String, optional): Base CFI string of the section (EPUB-specific).
- **`id`** (Any): An identifier for the section, used for getting TOC items. Must be usable as a Map key.

### `dir` (String, optional)

Represents the page progression direction of the book:
- `"rtl"` - Right-to-left (Arabic, Hebrew, etc.)
- `"ltr"` - Left-to-right (English, etc.)

### `toc` (Array, optional)

Table of contents structure. Each item has:
- **`label`** (String): Display label for the item
- **`href`** (String): Destination reference (doesn't need to be a valid URL)
- **`subitems`** (Array, optional): Nested TOC items

### `pageList` (Array, optional)

Page list structure (similar to TOC but for page references).

### `metadata` (Object, optional)

Book metadata following the Readium webpub manifest schema. Properties can include:
- **`title`** (String or Object): Book title (can be multilingual)
- **`creator`** (String, Object, or Array): Author(s)
- **`language`** (String): Language code
- **`identifier`** (String): Unique identifier (ISBN, etc.)
- **`publisher`** (String): Publisher name
- **`description`** (String): Book description
- **`subject`** (Array): Subject categories
- **`rights`** (String): Copyright information

### `rendition` (Object, optional)

Rendering properties that correspond to EPUB rendition properties:
- **`layout`** (String): `"pre-paginated"` for fixed layout, `"reflowable"` for flowing text
- **`orientation`** (String): `"auto"`, `"landscape"`, or `"portrait"`
- **`spread`** (String): `"auto"`, `"none"`, or `"landscape"`
- **`flow`** (String): `"auto"`, `"paginated"`, or `"scrolled"`

## Required Methods

### `resolveHref(href)` (Function)

Given an href string, returns an object representing the destination:

```javascript
{
  index: number,    // Index of the referenced section
  anchor: function  // Function that returns the document fragment
}
```

The `anchor` function takes a `Document` object and returns either an `Element`, a `Range`, or `null`.

### `resolveCFI(cfi)` (Function, optional)

Same as `resolveHref` but takes a CFI string instead of href.

### `isExternal(href)` (Function, optional)

Returns a boolean indicating if the link should be opened externally.

## Progress Tracking Methods

These methods are used by the progress tracking system:

### `splitTOCHref(href)` (Function, optional)

Given an href string from the TOC, returns an array:
- First element: section `id`
- Second element: fragment identifier (can be any type)

### `getTOCFragment(doc, id)` (Function, optional)

Given a `Document` object and fragment identifier, returns a `Node` representing the target.

## Data Transformation

### `transformTarget` (EventTarget, optional)

An `EventTarget` that can be used to transform book content as it loads. It dispatches custom `"data"` events with:

```javascript
{
  data: string|Blob|Promise,  // The content data
  type: string,               // Content type
  name: string                // Resource identifier
}
```

Event handlers should mutate the `data` property to transform the content.

## Example Implementation

Here's a minimal example of a book object:

```javascript
const book = {
  sections: [
    {
      id: 'chapter1',
      size: 1024,
      linear: 'yes',
      load: async () => 'blob:http://localhost:8000/chapter1.html',
      createDocument: async () => {
        const doc = new DOMParser().parseFromString('<html>...</html>', 'text/html')
        return doc
      }
    }
  ],
  
  dir: 'ltr',
  
  toc: [
    {
      label: 'Chapter 1',
      href: 'chapter1.xhtml'
    }
  ],
  
  metadata: {
    title: 'Sample Book',
    creator: 'John Doe',
    language: 'en'
  },
  
  rendition: {
    layout: 'reflowable'
  },
  
  resolveHref: (href) => {
    // Implementation to resolve href to section index and anchor
    return { index: 0, anchor: (doc) => doc.body }
  },
  
  isExternal: (href) => href.startsWith('http')
}
```

## Format-Specific Implementations

### EPUB

The EPUB parser (`epub.js`) implements the full interface with:
- CFI support for precise location tracking
- Media overlay support for text-to-speech
- Font deobfuscation
- Encryption support

### MOBI/KF8

The MOBI parser (`mobi.js`) implements:
- Both MOBI and KF8 (AZW3) formats
- Huffman compression decompression
- Font decompression
- Page break detection

### FB2

The FB2 parser (`fb2.js`) implements:
- FictionBook 2 XML format
- Binary content handling
- Image extraction

### CBZ

The CBZ parser (`comic-book.js`) implements:
- Comic book archive format
- Image-based content
- Fixed layout rendering

### PDF

The PDF parser (`pdf.js`) implements:
- PDF.js integration
- Page-based navigation
- Fixed layout rendering

## Best Practices

1. **Memory Management**: Implement `unload()` methods to free resources when sections are no longer needed.

2. **Error Handling**: Handle parsing errors gracefully and provide meaningful error messages.

3. **Performance**: Load content on-demand rather than all at once.

4. **Accessibility**: Ensure proper metadata and navigation structure for screen readers.

5. **Internationalization**: Support multilingual content and right-to-left languages.

## Extending the Interface

To add support for a new format:

1. Create a parser that implements the Book Interface
2. Add format detection in `view.js`
3. Register the parser with the appropriate MIME type or file extension
4. Test with various books in that format

See [Custom Format Support](../advanced/custom-formats.md) for detailed instructions. 