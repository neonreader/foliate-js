# Renderer Interface

The Renderer Interface defines how e-books are displayed and navigated. Foliate.js provides two main renderers: one for reflowable books and one for fixed-layout books.

## Overview

Renderers are custom elements (web components) that handle the display and navigation of book content. They abstract away the complexity of different book formats and provide a consistent interface for user interaction.

## Renderer Types

### 1. Paginator (Reflowable Books)

The `Paginator` class (`paginator.js`) handles reflowable content like EPUB novels, MOBI books, and FB2 files. It uses CSS multi-column layout for pagination.

### 2. Fixed Layout Renderer

The `FixedLayout` class (`fixed-layout.js`) handles fixed-layout content like PDFs, CBZ comics, and fixed-layout EPUBs. It displays content as-is without reflowing.

## Core Interface

All renderers implement the following interface:

### Methods

#### `open(book)` (Async)

Opens a book object that implements the Book Interface.

```javascript
await renderer.open(book)
```

#### `goTo(target)` (Async)

Navigates to a specific location in the book.

```javascript
await renderer.goTo({ index: 0, anchor: null })
```

The `target` object has the same structure as returned by `book.resolveHref()`.

#### `prev()` / `next()`

Navigate to the previous or next page/section.

```javascript
renderer.prev()
renderer.next()
```

### Events

#### `load`

Fired when a section is loaded.

```javascript
renderer.addEventListener('load', (event) => {
    const { doc, index } = event.detail
    console.log(`Section ${index} loaded`)
})
```

#### `relocate`

Fired when the location changes (user scrolls, navigates, etc.).

```javascript
renderer.addEventListener('relocate', (event) => {
    const { range, index, fraction } = event.detail
    console.log(`At section ${index}, ${Math.round(fraction * 100)}% through`)
})
```

#### `create-overlayer`

Fired when an overlay should be created for annotations or highlights.

```javascript
renderer.addEventListener('create-overlayer', (event) => {
    const { doc, index, attach } = event.detail
    const overlay = new Overlayer(doc)
    attach(overlay)
})
```

## Paginator Specific Features

The Paginator renderer provides additional functionality for reflowable content:

### Attributes

#### `animated`

Boolean attribute that adds sliding transition effects.

```html
<foliate-view animated></foliate-view>
```

#### `flow`

Controls the reading flow:
- `"paginated"` - Multi-column pagination
- `"scrolled"` - Continuous scrolling

```html
<foliate-view flow="paginated"></foliate-view>
```

#### `margin`

CSS length value for header/footer height (must be in pixels).

```html
<foliate-view margin="40px"></foliate-view>
```

#### `gap`

CSS percentage value for space between columns.

```html
<foliate-view gap="5%"></foliate-view>
```

#### `max-inline-size`

Maximum width of text columns (must be in pixels).

```html
<foliate-view max-inline-size="800px"></foliate-view>
```

#### `max-block-size`

Maximum height of text columns (must be in pixels).

```html
<foliate-view max-block-size="600px"></foliate-view>
```

#### `max-column-count`

Maximum number of columns to display.

```html
<foliate-view max-column-count="3"></foliate-view>
```

### Properties

#### `scrolled`

Boolean indicating if the renderer is in scrolled mode.

```javascript
if (renderer.scrolled) {
    console.log('Currently in scrolled mode')
}
```

#### `page` / `pages`

Current page number and total pages.

```javascript
console.log(`Page ${renderer.page} of ${renderer.pages}`)
```

#### `atStart` / `atEnd`

Boolean properties indicating if at the beginning or end.

```javascript
if (renderer.atStart) {
    console.log('At the beginning of the book')
}
```

### Methods

#### `scrollBy(dx, dy)`

Scroll by the specified amount.

```javascript
renderer.scrollBy(0, 100) // Scroll down 100px
```

#### `scrollToAnchor(anchor, select)`

Scroll to a specific anchor element.

```javascript
await renderer.scrollToAnchor('#chapter1', true)
```

#### `prevSection()` / `nextSection()`

Navigate to the previous or next section.

```javascript
await renderer.prevSection()
```

#### `firstSection()` / `lastSection()`

Navigate to the first or last section.

```javascript
await renderer.firstSection()
```

## Fixed Layout Specific Features

The Fixed Layout renderer is simpler and focuses on displaying content as-is:

### Methods

#### `setImageSize()`

Adjusts image sizes to fit the viewport.

```javascript
renderer.setImageSize()
```

#### `expand()`

Expands the content to fill the available space.

```javascript
renderer.expand()
```

## Styling

### CSS Parts

Both renderers expose CSS parts for styling:

#### `filter`

Apply filters to the book content only (not overlays):

```css
foliate-view::part(filter) {
    filter: invert(1) hue-rotate(180deg); /* Dark theme */
}
```

#### `head` / `foot` (Paginator only)

Style headers and footers in paginated mode:

```css
foliate-view::part(head) {
    padding-bottom: 4px;
    border-bottom: 1px solid graytext;
}

foliate-view::part(foot) {
    padding-top: 4px;
    border-top: 1px solid graytext;
}
```

### CSS Custom Properties

Set custom properties for styling:

```css
foliate-view {
    --foliate-font-family: 'Georgia', serif;
    --foliate-font-size: 18px;
    --foliate-line-height: 1.6;
    --foliate-margin: 20px;
}
```

## Advanced Usage

### Custom Navigation

```javascript
class CustomReader {
    constructor() {
        this.renderer = document.createElement('foliate-view')
        this.setupNavigation()
    }
    
    setupNavigation() {
        // Keyboard navigation
        document.addEventListener('keydown', (e) => {
            switch(e.key) {
                case 'ArrowLeft':
                    this.renderer.prev()
                    break
                case 'ArrowRight':
                    this.renderer.next()
                    break
                case 'Home':
                    this.renderer.firstSection()
                    break
                case 'End':
                    this.renderer.lastSection()
                    break
            }
        })
        
        // Touch/swipe navigation
        let startX = 0
        this.renderer.addEventListener('touchstart', (e) => {
            startX = e.touches[0].clientX
        })
        
        this.renderer.addEventListener('touchend', (e) => {
            const endX = e.changedTouches[0].clientX
            const diff = startX - endX
            
            if (Math.abs(diff) > 50) { // Minimum swipe distance
                if (diff > 0) {
                    this.renderer.next()
                } else {
                    this.renderer.prev()
                }
            }
        })
    }
}
```

### Progress Tracking

```javascript
class ProgressTracker {
    constructor(renderer) {
        this.renderer = renderer
        this.setupTracking()
    }
    
    setupTracking() {
        this.renderer.addEventListener('relocate', (e) => {
            const { fraction, index } = e.detail
            this.updateProgress(fraction, index)
        })
    }
    
    updateProgress(fraction, sectionIndex) {
        const progress = Math.round(fraction * 100)
        const section = sectionIndex + 1
        
        // Update UI
        document.getElementById('progress').textContent = 
            `${progress}% complete (Section ${section})`
        
        // Save to localStorage
        localStorage.setItem('reading-progress', JSON.stringify({
            fraction,
            section: sectionIndex,
            timestamp: Date.now()
        }))
    }
    
    restoreProgress() {
        const saved = localStorage.getItem('reading-progress')
        if (saved) {
            const { fraction, section } = JSON.parse(saved)
            this.renderer.goTo({ index: section, anchor: null })
        }
    }
}
```

### Custom Overlays

```javascript
class AnnotationManager {
    constructor(renderer) {
        this.renderer = renderer
        this.setupOverlays()
    }
    
    setupOverlays() {
        this.renderer.addEventListener('create-overlayer', (e) => {
            const { doc, index, attach } = e.detail
            const overlay = new Overlayer(doc)
            
            // Add custom annotation rendering
            overlay.add('highlight', range, (ctx, rect) => {
                ctx.fillStyle = 'rgba(255, 255, 0, 0.3)'
                ctx.fillRect(rect.x, rect.y, rect.width, rect.height)
            })
            
            attach(overlay)
        })
    }
}
```

## Performance Considerations

1. **Memory Management**: Renderers automatically unload sections when they're no longer needed.

2. **Lazy Loading**: Content is loaded on-demand as the user navigates.

3. **Debounced Updates**: Location changes are debounced to prevent excessive updates.

4. **Efficient Rendering**: The paginator uses efficient algorithms for finding visible ranges.

## Best Practices

1. **Event Handling**: Always remove event listeners when destroying renderers to prevent memory leaks.

2. **Error Handling**: Wrap renderer operations in try-catch blocks.

3. **Accessibility**: Ensure keyboard navigation and screen reader support.

4. **Responsive Design**: Use CSS media queries to adjust renderer settings for different screen sizes.

5. **Performance Monitoring**: Monitor memory usage and rendering performance, especially with large books. 