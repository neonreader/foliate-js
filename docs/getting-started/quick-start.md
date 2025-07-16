# Quick Start Guide

This guide will help you get up and running with Foliate.js in minutes.

## Minimal Setup

### 1. Create HTML File

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>E-book Reader</title>
    <meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none';">
    <style>
        body { margin: 0; padding: 0; }
        foliate-view { width: 100vw; height: 100vh; }
    </style>
</head>
<body>
    <div id="app"></div>
    <script type="module" src="reader.js"></script>
</body>
</html>
```

### 2. Create JavaScript File

```javascript
// reader.js
import './foliate-js/view.js'

// Create the reader view
const view = document.createElement('foliate-view')
document.getElementById('app').appendChild(view)

// Handle location changes
view.addEventListener('relocate', e => {
    console.log('Reading progress:', e.detail.fraction)
})

// Open a book
async function openBook(file) {
    try {
        await view.open(file)
        console.log('Book opened successfully!')
    } catch (error) {
        console.error('Failed to open book:', error)
    }
}

// Add file input
const input = document.createElement('input')
input.type = 'file'
input.accept = '.epub,.mobi,.azw3,.fb2,.cbz,.pdf'
input.addEventListener('change', e => {
    const file = e.target.files[0]
    if (file) openBook(file)
})
document.body.appendChild(input)
```

### 3. Serve and Test

Start a local server and open the HTML file:

```bash
python -m http.server 8000
# Then visit http://localhost:8000
```

## Basic Navigation

Once you have a book open, you can navigate programmatically:

```javascript
// Go to next page
view.next()

// Go to previous page
view.prev()

// Go to specific location
await view.goTo({ index: 0, anchor: null })

// Go to table of contents item
const tocItem = { href: 'chapter1.xhtml#section1' }
await view.goTo(view.resolveNavigation(tocItem))
```

## Reading Progress

Track reading progress with the `relocate` event:

```javascript
view.addEventListener('relocate', e => {
    const { fraction, index, range } = e.detail
    
    // Update progress bar
    const progress = Math.round(fraction * 100)
    console.log(`Progress: ${progress}%`)
    
    // Update page indicator
    console.log(`Section: ${index + 1}`)
})
```

## Search Functionality

Implement basic search:

```javascript
// Search for text
async function searchText(query) {
    const results = []
    for await (const result of view.search({ query })) {
        results.push(result)
    }
    console.log(`Found ${results.length} results`)
    return results
}

// Clear search highlights
view.clearSearch()
```

## Custom Styling

Apply custom styles to the reader:

```css
/* Dark theme */
foliate-view::part(filter) {
    filter: invert(1) hue-rotate(180deg);
}

/* Custom fonts */
foliate-view {
    --foliate-font-family: 'Georgia', serif;
    --foliate-font-size: 18px;
    --foliate-line-height: 1.6;
}
```

## Complete Example

Here's a complete, functional e-book reader:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple E-book Reader</title>
    <meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none';">
    <style>
        body { 
            margin: 0; 
            padding: 20px; 
            font-family: Arial, sans-serif; 
        }
        .controls {
            margin-bottom: 20px;
            padding: 10px;
            background: #f5f5f5;
            border-radius: 5px;
        }
        .progress {
            margin: 10px 0;
            padding: 10px;
            background: #e0e0e0;
            border-radius: 3px;
        }
        foliate-view { 
            width: 100%; 
            height: 70vh; 
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        button {
            margin: 0 5px;
            padding: 8px 16px;
            border: none;
            border-radius: 3px;
            background: #007bff;
            color: white;
            cursor: pointer;
        }
        button:hover { background: #0056b3; }
        button:disabled { background: #ccc; cursor: not-allowed; }
    </style>
</head>
<body>
    <div class="controls">
        <input type="file" id="fileInput" accept=".epub,.mobi,.azw3,.fb2,.cbz,.pdf">
        <button id="prevBtn" disabled>Previous</button>
        <button id="nextBtn" disabled>Next</button>
        <button id="searchBtn" disabled>Search</button>
        <input type="text" id="searchInput" placeholder="Search..." style="display: none;">
        <button id="clearSearchBtn" style="display: none;">Clear</button>
    </div>
    
    <div class="progress" id="progress"></div>
    <div id="reader"></div>

    <script type="module">
        import './foliate-js/view.js'

        class SimpleReader {
            constructor() {
                this.view = document.createElement('foliate-view')
                this.init()
            }

            init() {
                document.getElementById('reader').appendChild(this.view)
                
                // Event listeners
                this.view.addEventListener('relocate', this.onLocationChange.bind(this))
                this.view.addEventListener('load', this.onLoad.bind(this))
                
                // Button listeners
                document.getElementById('fileInput').addEventListener('change', this.onFileSelect.bind(this))
                document.getElementById('prevBtn').addEventListener('click', () => this.view.prev())
                document.getElementById('nextBtn').addEventListener('click', () => this.view.next())
                document.getElementById('searchBtn').addEventListener('click', this.toggleSearch.bind(this))
                document.getElementById('clearSearchBtn').addEventListener('click', () => {
                    this.view.clearSearch()
                    this.toggleSearch()
                })
            }

            async onFileSelect(event) {
                const file = event.target.files[0]
                if (file) {
                    try {
                        await this.view.open(file)
                        this.enableControls(true)
                    } catch (error) {
                        alert('Failed to open book: ' + error.message)
                    }
                }
            }

            onLocationChange(event) {
                const { fraction, index } = event.detail
                const progress = Math.round(fraction * 100)
                document.getElementById('progress').textContent = 
                    `Progress: ${progress}% | Section: ${index + 1}`
            }

            onLoad(event) {
                console.log('Book loaded:', event.detail)
            }

            enableControls(enabled) {
                document.getElementById('prevBtn').disabled = !enabled
                document.getElementById('nextBtn').disabled = !enabled
                document.getElementById('searchBtn').disabled = !enabled
            }

            toggleSearch() {
                const input = document.getElementById('searchInput')
                const clearBtn = document.getElementById('clearSearchBtn')
                const isVisible = input.style.display !== 'none'
                
                if (isVisible) {
                    input.style.display = 'none'
                    clearBtn.style.display = 'none'
                    input.removeEventListener('input', this.onSearch.bind(this))
                } else {
                    input.style.display = 'inline'
                    clearBtn.style.display = 'inline'
                    input.addEventListener('input', this.onSearch.bind(this))
                    input.focus()
                }
            }

            async onSearch(event) {
                const query = event.target.value.trim()
                if (query.length < 2) {
                    this.view.clearSearch()
                    return
                }
                
                try {
                    const results = []
                    for await (const result of this.view.search({ query })) {
                        results.push(result)
                    }
                    console.log(`Found ${results.length} results for "${query}"`)
                } catch (error) {
                    console.error('Search failed:', error)
                }
            }
        }

        // Initialize the reader
        new SimpleReader()
    </script>
</body>
</html>
```

## Next Steps

- Explore [Basic Usage Examples](./basic-usage.md) for more patterns
- Learn about the [Book Interface](../core-concepts/book-interface.md)
- Check out the [API Reference](../api/) for detailed documentation
- Read about [Security Considerations](../advanced/security.md) 