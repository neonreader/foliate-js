# View Component API

The `View` component (`view.js`) is the main entry point for Foliate.js. It provides a high-level interface for opening books, navigating, searching, and managing annotations.

## Overview

The `View` class extends `HTMLElement` and is registered as the custom element `<foliate-view>`. It automatically detects book formats and uses the appropriate renderer (paginator for reflowable books, fixed layout for fixed-layout books).

## Import

```javascript
import './foliate-js/view.js'
```

## Basic Usage

```javascript
// Create the view element
const view = document.createElement('foliate-view')
document.body.appendChild(view)

// Open a book
await view.open('example.epub')
```

## Constructor

```javascript
new View()
```

The constructor automatically sets up the view with default settings and event handling.

## Properties

### `book` (Object)

The currently loaded book object implementing the Book Interface.

```javascript
console.log(view.book.metadata.title)
```

### `isFixedLayout` (Boolean)

Indicates if the current book uses fixed layout rendering.

```javascript
if (view.isFixedLayout) {
    console.log('Using fixed layout renderer')
}
```

### `lastLocation` (Object)

The last known reading location.

```javascript
console.log(view.lastLocation)
// { index: 0, anchor: null, fraction: 0.5 }
```

### `history` (History)

Navigation history object for back/forward functionality.

```javascript
console.log(view.history.canGoBack) // true/false
console.log(view.history.canGoForward) // true/false
```

## Methods

### `open(book)` (Async)

Opens a book. The `book` parameter can be:

- **File/Blob object**: A file from file input or drag-and-drop
- **URL string**: A URL to fetch the book from
- **Book object**: An object implementing the Book Interface

```javascript
// Open from file input
const file = event.target.files[0]
await view.open(file)

// Open from URL
await view.open('https://example.com/book.epub')

// Open book object directly
await view.open(bookObject)
```

**Returns**: Promise that resolves when the book is loaded.

**Throws**: 
- `ResponseError` - Network error when fetching from URL
- `NotFoundError` - File not found
- `UnsupportedTypeError` - Unsupported book format

### `close()`

Closes the current book and cleans up resources.

```javascript
view.close()
```

### `goTo(target)` (Async)

Navigates to a specific location in the book.

```javascript
// Go to specific section
await view.goTo({ index: 0, anchor: null })

// Go to TOC item
const tocItem = { href: 'chapter1.xhtml#section1' }
await view.goTo(view.resolveNavigation(tocItem))

// Go to CFI location (EPUB only)
await view.goTo(view.resolveCFI('/6/4!/4'))
```

**Parameters**:
- `target` (Object): Location object with `index` and `anchor` properties

### `goToFraction(fraction)` (Async)

Navigates to a specific fraction through the book (0.0 to 1.0).

```javascript
await view.goToFraction(0.5) // Go to 50% through the book
```

### `goToTextStart()`

Navigates to the beginning of the text content (skips front matter).

```javascript
view.goToTextStart()
```

### `prev(distance)` / `next(distance)`

Navigate to the previous or next location.

```javascript
view.prev() // Go to previous page
view.next() // Go to next page
view.prev(2) // Go back 2 pages
```

**Parameters**:
- `distance` (Number, optional): Number of pages to skip (default: 1)

### `goLeft()` / `goRight()`

Navigate left or right (useful for fixed layout books).

```javascript
view.goLeft() // Previous page in fixed layout
view.goRight() // Next page in fixed layout
```

### `select(target)`

Selects text at the specified location.

```javascript
await view.select({ index: 0, anchor: range })
```

### `deselect()`

Clears the current text selection.

```javascript
view.deselect()
```

### `search(options)` (Async Generator)

Searches for text in the book.

```javascript
// Basic search
for await (const result of view.search({ query: 'search term' })) {
    console.log(result)
}

// Advanced search
for await (const result of view.search({
    query: 'search term',
    matchCase: false,
    matchDiacritics: true,
    matchWholeWords: false
})) {
    console.log(result)
}
```

**Parameters**:
- `options` (Object):
  - `query` (String): Search term
  - `matchCase` (Boolean, optional): Case-sensitive search
  - `matchDiacritics` (Boolean, optional): Match diacritical marks
  - `matchWholeWords` (Boolean, optional): Match whole words only

**Returns**: Async generator yielding search results.

### `clearSearch()`

Clears search highlights.

```javascript
view.clearSearch()
```

### `addAnnotation(annotation, remove)`

Adds an annotation to the book.

```javascript
const annotation = {
    type: 'highlight',
    range: range,
    color: 'yellow',
    note: 'Important passage'
}

view.addAnnotation(annotation)
```

**Parameters**:
- `annotation` (Object): Annotation object
- `remove` (Boolean, optional): Whether to remove existing annotations

### `deleteAnnotation(annotation)`

Removes an annotation from the book.

```javascript
view.deleteAnnotation(annotation)
```

### `showAnnotation(annotation)` (Async)

Navigates to and highlights an annotation.

```javascript
await view.showAnnotation(annotation)
```

### `getCFI(index, range)`

Gets the CFI (EPUB Canonical Fragment Identifier) for a location.

```javascript
const cfi = view.getCFI(0, range)
console.log(cfi) // "/6/4!/4"
```

**Returns**: String CFI or null if not supported.

### `resolveCFI(cfi)`

Resolves a CFI to a location object.

```javascript
const location = view.resolveCFI('/6/4!/4')
console.log(location) // { index: 0, anchor: function }
```

### `resolveNavigation(target)`

Resolves a navigation target (TOC item, page list item) to a location.

```javascript
const tocItem = { href: 'chapter1.xhtml#section1' }
const location = view.resolveNavigation(tocItem)
await view.goTo(location)
```

### `getSectionFractions()`

Gets the fraction boundaries for each section.

```javascript
const fractions = view.getSectionFractions()
console.log(fractions) // [0, 0.25, 0.5, 0.75, 1]
```

### `getProgressOf(index, range)`

Gets the reading progress for a specific location.

```javascript
const progress = view.getProgressOf(0, range)
console.log(progress) // { fraction: 0.25, section: {...}, location: {...}, time: {...} }
```

### `getTOCItemOf(target)` (Async)

Gets the TOC item corresponding to a location.

```javascript
const tocItem = await view.getTOCItemOf({ index: 0, anchor: null })
console.log(tocItem) // { label: 'Chapter 1', href: 'chapter1.xhtml' }
```

### `initTTS(granularity, highlight)` (Async)

Initializes text-to-speech functionality.

```javascript
await view.initTTS('word', true)
```

**Parameters**:
- `granularity` (String, optional): 'word', 'sentence', or 'paragraph'
- `highlight` (Boolean, optional): Whether to highlight spoken text

### `startMediaOverlay()`

Starts media overlay playback (EPUB only).

```javascript
view.startMediaOverlay()
```

## Events

### `relocate`

Fired when the reading location changes.

```javascript
view.addEventListener('relocate', (event) => {
    const { range, index, fraction } = event.detail
    console.log(`At section ${index}, ${Math.round(fraction * 100)}% through`)
})
```

**Event Detail**:
- `range` (Range): Current visible text range
- `index` (Number): Current section index
- `fraction` (Number): Progress through current section (0.0 to 1.0)

### `load`

Fired when a section is loaded.

```javascript
view.addEventListener('load', (event) => {
    const { doc, index } = event.detail
    console.log(`Section ${index} loaded`)
})
```

**Event Detail**:
- `doc` (Document): Loaded document
- `index` (Number): Section index

### `create-overlayer`

Fired when an overlay should be created for annotations.

```javascript
view.addEventListener('create-overlayer', (event) => {
    const { doc, index, attach } = event.detail
    const overlay = new Overlayer(doc)
    attach(overlay)
})
```

**Event Detail**:
- `doc` (Document): Document to overlay
- `index` (Number): Section index
- `attach` (Function): Function to attach the overlay

## Initialization

### `init(options)` (Async)

Initializes the view with specific options.

```javascript
await view.init({
    lastLocation: { index: 0, anchor: null, fraction: 0.5 },
    showTextStart: true
})
```

**Parameters**:
- `options` (Object):
  - `lastLocation` (Object, optional): Location to restore
  - `showTextStart` (Boolean, optional): Whether to go to text start

## Error Handling

The View component throws specific error types:

```javascript
try {
    await view.open(file)
} catch (error) {
    if (error instanceof ResponseError) {
        console.error('Network error:', error.message)
    } else if (error instanceof NotFoundError) {
        console.error('File not found')
    } else if (error instanceof UnsupportedTypeError) {
        console.error('Unsupported book format')
    } else {
        console.error('Unknown error:', error)
    }
}
```

## Complete Example

```javascript
import './foliate-js/view.js'

class EbookReader {
    constructor() {
        this.view = document.createElement('foliate-view')
        this.setupEventListeners()
    }
    
    setupEventListeners() {
        // Location changes
        this.view.addEventListener('relocate', (e) => {
            this.updateProgress(e.detail)
        })
        
        // Section loads
        this.view.addEventListener('load', (e) => {
            console.log(`Section ${e.detail.index} loaded`)
        })
        
        // Overlay creation
        this.view.addEventListener('create-overlayer', (e) => {
            this.setupOverlay(e.detail)
        })
    }
    
    async openBook(file) {
        try {
            await this.view.open(file)
            console.log('Book opened successfully')
        } catch (error) {
            console.error('Failed to open book:', error)
        }
    }
    
    updateProgress(detail) {
        const { fraction, index } = detail
        const progress = Math.round(fraction * 100)
        console.log(`Progress: ${progress}% (Section ${index + 1})`)
    }
    
    setupOverlay({ doc, index, attach }) {
        const overlay = new Overlayer(doc)
        
        // Add highlight rendering
        overlay.add('highlight', range, (ctx, rect) => {
            ctx.fillStyle = 'rgba(255, 255, 0, 0.3)'
            ctx.fillRect(rect.x, rect.y, rect.width, rect.height)
        })
        
        attach(overlay)
    }
    
    async searchText(query) {
        const results = []
        for await (const result of this.view.search({ query })) {
            results.push(result)
        }
        return results
    }
    
    async addHighlight(range, color = 'yellow') {
        const annotation = {
            type: 'highlight',
            range: range,
            color: color,
            timestamp: Date.now()
        }
        
        this.view.addAnnotation(annotation)
        return annotation
    }
}

// Usage
const reader = new EbookReader()
document.body.appendChild(reader.view)

// Open a book
const fileInput = document.createElement('input')
fileInput.type = 'file'
fileInput.accept = '.epub,.mobi,.azw3,.fb2,.cbz,.pdf'
fileInput.addEventListener('change', (e) => {
    const file = e.target.files[0]
    if (file) reader.openBook(file)
})
document.body.appendChild(fileInput)
```

## Performance Tips

1. **Memory Management**: The view automatically manages memory, but call `close()` when done with a book.

2. **Search Performance**: Search is performed asynchronously and yields results as they're found.

3. **Navigation**: Use `goToFraction()` for large jumps, `prev()`/`next()` for small movements.

4. **Event Handling**: Remove event listeners when destroying the view to prevent memory leaks. 