# Installation & Setup

## Prerequisites

Foliate.js requires a modern browser with support for:
- ES Modules
- Web Components
- Fetch API
- File API
- Web Crypto API (for font deobfuscation)

## Installation Methods

### Method 1: Git Submodule (Recommended)

Since there's no official release yet, it's recommended to include the library as a git submodule in your project:

```bash
git submodule add https://github.com/johnfactotum/foliate-js.git
```

This allows you to easily update the library and track changes.

### Method 2: Direct Download

Clone the repository directly:

```bash
git clone https://github.com/johnfactotum/foliate-js.git
```

### Method 3: Copy Files

Copy the necessary files directly into your project structure.

## Project Structure

After installation, your project should include the following key files:

```
your-project/
├── foliate-js/
│   ├── view.js              # Main view component
│   ├── epub.js              # EPUB format support
│   ├── mobi.js              # MOBI/KF8 format support
│   ├── fb2.js               # FB2 format support
│   ├── comic-book.js        # CBZ format support
│   ├── pdf.js               # PDF format support (experimental)
│   ├── paginator.js         # Pagination engine
│   ├── fixed-layout.js      # Fixed layout renderer
│   ├── search.js            # Search functionality
│   ├── progress.js          # Progress tracking
│   ├── overlayer.js         # Annotations and overlays
│   ├── tts.js               # Text-to-speech support
│   ├── epubcfi.js           # EPUB CFI parsing
│   ├── text-walker.js       # Text traversal utilities
│   └── vendor/              # Third-party dependencies
│       ├── zip.js           # ZIP file handling
│       ├── fflate.js        # Compression utilities
│       └── pdfjs/           # PDF.js library
└── your-app/
    ├── index.html
    └── app.js
```

## Basic Setup

### 1. HTML Setup

Include the library in your HTML file:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>E-book Reader</title>
    
    <!-- Content Security Policy (IMPORTANT) -->
    <meta http-equiv="Content-Security-Policy" 
          content="script-src 'self'; object-src 'none';">
    
    <style>
        foliate-view {
            width: 100%;
            height: 100vh;
        }
    </style>
</head>
<body>
    <div id="reader"></div>
    
    <script type="module" src="app.js"></script>
</body>
</html>
```

### 2. JavaScript Setup

Create your main application file:

```javascript
// app.js
import './foliate-js/view.js'

class EbookReader {
    constructor() {
        this.view = document.createElement('foliate-view')
        this.init()
    }
    
    init() {
        // Add the view to the DOM
        document.getElementById('reader').appendChild(this.view)
        
        // Set up event listeners
        this.view.addEventListener('relocate', this.onLocationChange.bind(this))
        this.view.addEventListener('load', this.onSectionLoad.bind(this))
    }
    
    async openBook(file) {
        try {
            await this.view.open(file)
        } catch (error) {
            console.error('Failed to open book:', error)
        }
    }
    
    onLocationChange(event) {
        console.log('Location changed:', event.detail)
        // Update your UI with reading progress
    }
    
    onSectionLoad(event) {
        console.log('Section loaded:', event.detail)
    }
}

// Initialize the reader
const reader = new EbookReader()

// Example: Open a book from file input
document.addEventListener('DOMContentLoaded', () => {
    const fileInput = document.createElement('input')
    fileInput.type = 'file'
    fileInput.accept = '.epub,.mobi,.azw3,.fb2,.cbz,.pdf'
    fileInput.addEventListener('change', (event) => {
        const file = event.target.files[0]
        if (file) {
            reader.openBook(file)
        }
    })
    document.body.appendChild(fileInput)
})
```

## Development Server

Since Foliate.js uses ES modules, you need to serve your files through a web server. You cannot simply open HTML files directly in the browser.

### Using Python

```bash
# Python 3
python -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000
```

### Using Node.js

```bash
# Using npx
npx http-server

# Using npm
npm install -g http-server
http-server
```

### Using PHP

```bash
php -S localhost:8000
```

## Security Configuration

### Content Security Policy

It's **crucial** to implement a Content Security Policy to prevent script injection from e-books:

```html
<meta http-equiv="Content-Security-Policy" 
      content="script-src 'self'; object-src 'none';">
```

### HTTPS Requirement

For font deobfuscation to work properly, your application must be served over HTTPS, as it requires the Web Crypto API.

## Troubleshooting

### Common Issues

1. **CORS Errors**: Make sure you're serving files through a web server, not opening them directly.

2. **Module Import Errors**: Ensure your server supports ES modules and serves files with the correct MIME types.

3. **Font Deobfuscation Fails**: This usually means you're not running over HTTPS or the Web Crypto API is not available.

4. **PDF Not Loading**: PDF support requires PDF.js. Make sure the `vendor/pdfjs/` directory is included.

### Debug Mode

Enable debug logging by setting:

```javascript
localStorage.setItem('foliate-debug', 'true')
```

## Next Steps

- Read the [Quick Start Guide](./quick-start.md) for basic usage
- Check out [Basic Usage Examples](./basic-usage.md) for common patterns
- Explore the [API Reference](../api/) for detailed documentation 