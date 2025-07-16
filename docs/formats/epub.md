# EPUB Support

Foliate.js provides comprehensive support for EPUB (Electronic Publication) format, including EPUB 2, EPUB 3, and EPUB 3.2 specifications.

## Overview

The EPUB parser (`epub.js`) implements the full Book Interface and provides additional EPUB-specific features like CFI (Canonical Fragment Identifier) support, media overlays, font deobfuscation, and encryption handling.

## Features

- **EPUB 2, 3, and 3.2 Support**: Full compatibility with all EPUB versions
- **CFI Support**: Precise location tracking with EPUB CFI
- **Media Overlays**: Text-to-speech support with SMIL synchronization
- **Font Deobfuscation**: IDPF font obfuscation algorithm support
- **Encryption**: DRM-free encryption support
- **Fixed Layout**: Support for pre-paginated EPUBs
- **Navigation**: TOC, page list, and landmark navigation
- **Metadata**: Comprehensive metadata extraction

## Basic Usage

### Opening an EPUB

```javascript
import { EPUB } from './foliate-js/epub.js'

// Create EPUB parser
const epub = new EPUB({ loadText, loadBlob, getSize })

// Parse EPUB file
const book = await epub.init()
```

### Using with View Component

```javascript
import './foliate-js/view.js'

const view = document.createElement('foliate-view')
document.body.appendChild(view)

// Open EPUB file
await view.open('book.epub')
```

## EPUB Parser Options

### Constructor Options

```javascript
const epub = new EPUB({
    loadText,      // Function to load text files
    loadBlob,      // Function to load binary files
    getSize,       // Function to get file sizes
    sha1           // SHA-1 implementation (optional)
})
```

### Loader Functions

#### `loadText(filename)` (Async)

Loads a text file from the EPUB archive.

```javascript
const loadText = async (filename) => {
    // Implementation to load text content
    return textContent
}
```

#### `loadBlob(filename, type)` (Async)

Loads a binary file from the EPUB archive.

```javascript
const loadBlob = async (filename, type) => {
    // Implementation to load binary content
    return blob
}
```

#### `getSize(filename)` (Sync)

Gets the size of a file in bytes.

```javascript
const getSize = (filename) => {
    // Implementation to get file size
    return sizeInBytes
}
```

## EPUB-Specific Features

### CFI (Canonical Fragment Identifier)

CFI provides precise location tracking within EPUB documents.

#### Getting CFI

```javascript
// Get CFI for current location
const cfi = view.getCFI(sectionIndex, range)
console.log(cfi) // "/6/4!/4"
```

#### Resolving CFI

```javascript
// Navigate to CFI location
const location = view.resolveCFI('/6/4!/4')
await view.goTo(location)
```

#### CFI Structure

CFIs are represented as arrays of parts:

```javascript
// Simple CFI: "/6/4!/4"
[
    [
        { index: 6 },
        { index: 4 }
    ],
    [
        { index: 4 }
    ]
]

// Range CFI: "/6/4!/2,/2,/4"
{
    parent: [
        [
            { index: 6 },
            { index: 4 }
        ],
        [
            { index: 2 }
        ]
    ],
    start: [
        [
            { index: 2 }
        ]
    ],
    end: [
        [
            { index: 4 }
        ]
    ]
}
```

### Media Overlays

EPUB media overlays provide synchronized text-to-speech functionality.

#### Starting Media Overlay

```javascript
// Start media overlay playback
view.startMediaOverlay()
```

#### Media Overlay Events

```javascript
// Listen for media overlay events
view.addEventListener('mediaoverlay-start', (e) => {
    console.log('Media overlay started')
})

view.addEventListener('mediaoverlay-end', (e) => {
    console.log('Media overlay ended')
})
```

### Font Deobfuscation

EPUB fonts may be obfuscated using the IDPF algorithm.

#### Automatic Deobfuscation

The parser automatically handles font deobfuscation when the Web Crypto API is available:

```javascript
// Fonts are automatically deobfuscated
const book = await epub.init()
```

#### Custom SHA-1 Implementation

If Web Crypto is not available, provide a custom SHA-1 implementation:

```javascript
import { sha1 } from 'your-sha1-library'

const epub = new EPUB({
    loadText,
    loadBlob,
    getSize,
    sha1: sha1
})
```

### Encryption Support

EPUB supports DRM-free encryption for content protection.

#### Encrypted Content

The parser automatically handles encrypted content:

```javascript
// Encryption is handled transparently
const book = await epub.init()
```

## EPUB Metadata

### Accessing Metadata

```javascript
const book = await epub.init()

console.log(book.metadata.title)        // Book title
console.log(book.metadata.creator)      // Author(s)
console.log(book.metadata.language)     // Language code
console.log(book.metadata.identifier)   // ISBN or other ID
console.log(book.metadata.publisher)    // Publisher
console.log(book.metadata.description)  // Book description
console.log(book.metadata.subject)      // Subject categories
console.log(book.metadata.rights)       // Copyright information
```

### Multilingual Metadata

EPUB supports multilingual metadata:

```javascript
// Multilingual title
const title = book.metadata.title
if (typeof title === 'object') {
    console.log(title.en) // English title
    console.log(title.ja) // Japanese title
} else {
    console.log(title) // Single language title
}

// Multiple creators
const creators = book.metadata.creator
if (Array.isArray(creators)) {
    creators.forEach(creator => {
        console.log(creator.name, creator.role)
    })
} else {
    console.log(creators.name, creators.role)
}
```

## Navigation

### Table of Contents

```javascript
const toc = book.toc

function displayTOC(items, level = 0) {
    items.forEach(item => {
        const indent = '  '.repeat(level)
        console.log(`${indent}- ${item.label}`)
        
        if (item.subitems) {
            displayTOC(item.subitems, level + 1)
        }
    })
}

displayTOC(toc)
```

### Page List

```javascript
const pageList = book.pageList

pageList.forEach(page => {
    console.log(`Page ${page.label}: ${page.href}`)
})
```

### Navigation Resolution

```javascript
// Navigate to TOC item
const tocItem = { href: 'chapter1.xhtml#section1' }
const location = book.resolveHref(tocItem.href)
await view.goTo(location)

// Navigate to page
const pageItem = { href: 'page-10.xhtml' }
const pageLocation = book.resolveHref(pageItem.href)
await view.goTo(pageLocation)
```

## Rendition Properties

### Layout Types

```javascript
const rendition = book.rendition

if (rendition.layout === 'pre-paginated') {
    console.log('Fixed layout book')
} else {
    console.log('Reflowable book')
}
```

### Rendition Options

```javascript
console.log(rendition.orientation)  // 'auto', 'landscape', 'portrait'
console.log(rendition.spread)       // 'auto', 'none', 'landscape'
console.log(rendition.flow)         // 'auto', 'paginated', 'scrolled'
```

## Content Processing

### Resource Loading

The EPUB parser automatically handles:

- **HTML/XHTML Content**: Parsed and rendered
- **CSS Stylesheets**: Applied to content
- **Images**: Loaded on demand
- **Fonts**: Deobfuscated and applied
- **Audio/Video**: Supported for media overlays

### Content Transformation

```javascript
// Transform content as it loads
book.transformTarget.addEventListener('data', (event) => {
    const { data, type, name } = event.detail
    
    if (type === 'text/html') {
        // Transform HTML content
        event.detail.data = transformHTML(data)
    } else if (type === 'text/css') {
        // Transform CSS content
        event.detail.data = transformCSS(data)
    }
})
```

## Advanced Features

### Custom Content Filtering

```javascript
// Filter out specific elements
const filter = (node) => {
    if (node.nodeType !== 1) return NodeFilter.FILTER_ACCEPT
    
    // Skip footnotes
    if (node.matches('.footnote')) return NodeFilter.FILTER_REJECT
    
    // Skip navigation elements
    if (node.matches('nav')) return NodeFilter.FILTER_REJECT
    
    return NodeFilter.FILTER_ACCEPT
}

// Use filter in CFI operations
const cfi = view.getCFI(sectionIndex, range, filter)
```

### External Link Handling

```javascript
// Check if link should open externally
const isExternal = book.isExternal('https://example.com')
if (isExternal) {
    window.open('https://example.com', '_blank')
} else {
    // Handle internal navigation
    const location = book.resolveHref('chapter2.xhtml')
    await view.goTo(location)
}
```

### Progress Tracking

```javascript
// Get progress information
const progress = view.getProgressOf(sectionIndex, range)
console.log(`Overall progress: ${Math.round(progress.fraction * 100)}%`)
console.log(`Section: ${progress.section.current}/${progress.section.total}`)
console.log(`Location: ${progress.location.current}/${progress.location.total}`)
```

## Error Handling

### Common EPUB Errors

```javascript
try {
    const book = await epub.init()
} catch (error) {
    if (error.message.includes('container.xml')) {
        console.error('Invalid EPUB: missing container.xml')
    } else if (error.message.includes('package.opf')) {
        console.error('Invalid EPUB: missing package.opf')
    } else if (error.message.includes('encryption')) {
        console.error('EPUB encryption not supported')
    } else {
        console.error('EPUB parsing failed:', error)
    }
}
```

### Validation

```javascript
// Basic EPUB validation
function validateEPUB(epub) {
    const errors = []
    
    if (!epub.metadata.title) {
        errors.push('Missing title')
    }
    
    if (!epub.sections.length) {
        errors.push('No content sections')
    }
    
    if (!epub.toc.length) {
        errors.push('No table of contents')
    }
    
    return errors
}

const book = await epub.init()
const errors = validateEPUB(book)
if (errors.length > 0) {
    console.warn('EPUB validation warnings:', errors)
}
```

## Performance Considerations

### Memory Management

```javascript
// Unload sections when done
book.sections.forEach(section => {
    if (section.unload) {
        section.unload()
    }
})

// Destroy book when closing
book.destroy()
```

### Lazy Loading

The parser automatically implements lazy loading:

- Content is loaded only when needed
- Images are loaded on demand
- Fonts are loaded when referenced
- Large files are streamed

### Caching

```javascript
// Implement caching for better performance
const cache = new Map()

const loadText = async (filename) => {
    if (cache.has(filename)) {
        return cache.get(filename)
    }
    
    const content = await loadFromArchive(filename)
    cache.set(filename, content)
    return content
}
```

## Best Practices

1. **Always use HTTPS**: Required for font deobfuscation
2. **Implement CSP**: Prevent script injection from EPUBs
3. **Handle large files**: Use streaming for large EPUBs
4. **Validate content**: Check for malformed EPUBs
5. **Provide fallbacks**: Handle missing resources gracefully
6. **Monitor memory**: Unload sections when not needed
7. **Test thoroughly**: Test with various EPUB versions and features

## Examples

### Complete EPUB Reader

```javascript
import './foliate-js/view.js'

class EPUBReader {
    constructor() {
        this.view = document.createElement('foliate-view')
        this.setupEventListeners()
    }
    
    setupEventListeners() {
        this.view.addEventListener('relocate', (e) => {
            this.updateProgress(e.detail)
        })
        
        this.view.addEventListener('load', (e) => {
            this.updateTOC(e.detail)
        })
    }
    
    async openEPUB(file) {
        try {
            await this.view.open(file)
            console.log('EPUB opened successfully')
            
            // Display metadata
            this.displayMetadata()
            
            // Setup navigation
            this.setupNavigation()
            
        } catch (error) {
            console.error('Failed to open EPUB:', error)
        }
    }
    
    displayMetadata() {
        const { metadata } = this.view.book
        
        console.log('Title:', metadata.title)
        console.log('Author:', metadata.creator)
        console.log('Language:', metadata.language)
        console.log('Publisher:', metadata.publisher)
    }
    
    setupNavigation() {
        const { toc } = this.view.book
        
        // Create TOC navigation
        const tocElement = document.createElement('div')
        tocElement.innerHTML = this.buildTOC(toc)
        document.body.appendChild(tocElement)
    }
    
    buildTOC(items, level = 0) {
        return items.map(item => {
            const indent = '&nbsp;'.repeat(level * 4)
            const link = `<a href="#" data-href="${item.href}">${item.label}</a>`
            const subitems = item.subitems ? this.buildTOC(item.subitems, level + 1) : ''
            
            return `<div>${indent}${link}${subitems}</div>`
        }).join('')
    }
    
    updateProgress(detail) {
        const { fraction, index } = detail
        const progress = Math.round(fraction * 100)
        
        console.log(`Progress: ${progress}% (Section ${index + 1})`)
        
        // Save progress
        localStorage.setItem('epub-progress', JSON.stringify({
            fraction,
            section: index,
            timestamp: Date.now()
        }))
    }
    
    updateTOC(detail) {
        // Update TOC highlighting based on current section
        const { index } = detail
        // Implementation to highlight current TOC item
    }
}

// Usage
const reader = new EPUBReader()
document.body.appendChild(reader.view)

// Open EPUB file
const fileInput = document.createElement('input')
fileInput.type = 'file'
fileInput.accept = '.epub'
fileInput.addEventListener('change', (e) => {
    const file = e.target.files[0]
    if (file) reader.openEPUB(file)
})
document.body.appendChild(fileInput)
``` 