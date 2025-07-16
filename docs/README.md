# Foliate.js Documentation

Foliate.js is a comprehensive JavaScript library for rendering e-books in the browser. It supports multiple formats including EPUB, MOBI, KF8 (AZW3), FB2, CBZ, and PDF (experimental).

## Table of Contents

### Getting Started
- [Installation & Setup](./getting-started/installation.md)
- [Quick Start Guide](./getting-started/quick-start.md)
- [Basic Usage Examples](./getting-started/basic-usage.md)

### Core Concepts
- [Book Interface](./core-concepts/book-interface.md)
- [Renderer Interface](./core-concepts/renderer-interface.md)
- [Architecture Overview](./core-concepts/architecture.md)

### Format Support
- [EPUB Support](./formats/epub.md)
- [MOBI/KF8 Support](./formats/mobi.md)
- [FB2 Support](./formats/fb2.md)
- [CBZ Support](./formats/cbz.md)
- [PDF Support](./formats/pdf.md)

### API Reference
- [View Component](./api/view.md)
- [Paginator Component](./api/paginator.md)
- [Fixed Layout Renderer](./api/fixed-layout.md)
- [Search API](./api/search.md)
- [Progress Tracking](./api/progress.md)
- [Annotations & Overlays](./api/annotations.md)
- [Text-to-Speech](./api/tts.md)

### Advanced Topics
- [Custom Format Support](./advanced/custom-formats.md)
- [Performance Optimization](./advanced/performance.md)
- [Security Considerations](./advanced/security.md)
- [Styling & Theming](./advanced/styling.md)

### Examples & Tutorials
- [Building a Reader App](./examples/building-reader.md)
- [Custom Navigation](./examples/custom-navigation.md)
- [Search Implementation](./examples/search-implementation.md)
- [Progress Tracking](./examples/progress-tracking.md)

## Features

- **Multi-format Support**: EPUB, MOBI, KF8 (AZW3), FB2, CBZ, PDF
- **Pure JavaScript**: No external dependencies
- **Modular Architecture**: Use only what you need
- **Memory Efficient**: Doesn't require loading entire files into memory
- **Modern Browser Support**: ES modules, no legacy browser support
- **Customizable**: Highly configurable rendering and navigation
- **Search Capabilities**: Full-text search with various options
- **Progress Tracking**: Reading progress and location tracking
- **Annotations**: Support for highlights and notes
- **Text-to-Speech**: Media overlay support for EPUB

## Quick Example

```javascript
import './foliate-js/view.js'

const view = document.createElement('foliate-view')
document.body.append(view)

view.addEventListener('relocate', e => {
    console.log('Location changed:', e.detail)
})

// Open an EPUB file
await view.open('example.epub')
```

## Browser Support

Foliate.js requires modern browsers with support for:
- ES Modules
- Web Components
- Fetch API
- File API
- Web Crypto API (for font deobfuscation)

## Security Notice

⚠️ **Important**: EPUB books can contain scripted content which is potentially dangerous. Always use Content Security Policy (CSP) to block scripts except `'self'` when using this library.

## License

MIT License - see the [LICENSE](../LICENSE) file for details. 