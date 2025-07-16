# Search API

The Search API provides powerful text search capabilities across all supported e-book formats. It supports various search options including case sensitivity, diacritical mark matching, and whole word matching.

## Overview

The search functionality is implemented in `search.js` and provides two main search strategies:
- **Simple Search**: Basic string matching with case and accent options
- **Segmenter Search**: Advanced search using `Intl.Segmenter` for better word boundary detection

## Import

```javascript
import { search, searchMatcher } from './foliate-js/search.js'
```

## Basic Usage

### Direct Search

Search through an array of strings:

```javascript
import { search } from './foliate-js/search.js'

const textStrings = ['Hello world', 'This is a test', 'Another paragraph']
const query = 'world'

for (const result of search(textStrings, query)) {
    console.log(result)
    // {
    //   range: { startIndex: 0, startOffset: 6, endIndex: 0, endOffset: 11 },
    //   excerpt: { pre: 'Hello ', match: 'world', post: '' }
    // }
}
```

### Search with Options

```javascript
const options = {
    locales: 'en',
    granularity: 'word',
    sensitivity: 'base'
}

for (const result of search(textStrings, query, options)) {
    console.log(result)
}
```

## Search Options

### `locales` (String, optional)

The locale to use for search. Defaults to `'en'`.

```javascript
// Search in Spanish
const options = { locales: 'es' }
```

### `granularity` (String, optional)

The granularity of the search:
- `'grapheme'` (default): Character-by-character search
- `'word'`: Word-by-word search
- `'sentence'`: Sentence-by-sentence search

```javascript
const options = { granularity: 'word' }
```

### `sensitivity` (String, optional)

The sensitivity of the search:
- `'base'` (default): Case and accent insensitive
- `'accent'`: Accent sensitive, case insensitive
- `'case'`: Case sensitive, accent insensitive
- `'variant'`: Both case and accent sensitive

```javascript
const options = { sensitivity: 'case' }
```

## Search Results

Each search result contains:

### `range` (Object)

The location of the match:
- `startIndex` (Number): Index of the string containing the start
- `startOffset` (Number): Character offset within the start string
- `endIndex` (Number): Index of the string containing the end
- `endOffset` (Number): Character offset within the end string

### `excerpt` (Object)

Context around the match:
- `pre` (String): Text before the match
- `match` (String): The matched text
- `post` (String): Text after the match

## Advanced Usage

### Search Matcher

Create a reusable search matcher for documents:

```javascript
import { searchMatcher } from './foliate-js/search.js'

const matcher = searchMatcher(textWalker, {
    defaultLocale: 'en',
    matchCase: false,
    matchDiacritics: true,
    matchWholeWords: false,
    acceptNode: (node) => {
        // Custom node filtering
        return NodeFilter.FILTER_ACCEPT
    }
})

// Use the matcher
for (const result of matcher(document, 'search term')) {
    console.log(result)
}
```

### Matcher Options

#### `defaultLocale` (String, optional)

Default locale for search. Defaults to `'en'`.

#### `matchCase` (Boolean, optional)

Whether to perform case-sensitive search. Defaults to `false`.

#### `matchDiacritics` (Boolean, optional)

Whether to match diacritical marks. Defaults to `false`.

#### `matchWholeWords` (Boolean, optional)

Whether to match whole words only. Defaults to `false`.

#### `acceptNode` (Function, optional)

Custom node filter function. Should return `NodeFilter.FILTER_ACCEPT`, `FILTER_REJECT`, or `FILTER_SKIP`.

## Integration with View Component

The View component provides a high-level search interface:

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

## Search Examples

### Case-Insensitive Search

```javascript
const results = []
for (const result of search(textStrings, 'WORLD', { sensitivity: 'base' })) {
    results.push(result)
}
// Finds 'world', 'World', 'WORLD', etc.
```

### Whole Word Search

```javascript
const results = []
for (const result of search(textStrings, 'test', { 
    granularity: 'word',
    sensitivity: 'base'
})) {
    results.push(result)
}
// Finds 'test' but not 'testing' or 'contest'
```

### Accent-Sensitive Search

```javascript
const results = []
for (const result of search(textStrings, 'café', { 
    sensitivity: 'accent'
})) {
    results.push(result)
}
// Finds 'café' but not 'cafe'
```

### Multi-Language Search

```javascript
const results = []
for (const result of search(textStrings, 'search', { 
    locales: 'es'
})) {
    results.push(result)
}
// Uses Spanish language rules for word boundaries
```

## Performance Considerations

### Search Strategy Selection

The library automatically chooses the best search strategy:

1. **Simple Search** is used when:
   - `Intl.Segmenter` is not available
   - `granularity` is `'grapheme'` and `sensitivity` is `'variant'` or `'accent'`

2. **Segmenter Search** is used when:
   - `Intl.Segmenter` is available
   - `granularity` is `'word'` or `'sentence'`
   - `sensitivity` is `'base'` or `'case'`

### Memory Usage

- Search results are yielded as they're found (streaming)
- Large documents are processed efficiently
- Memory usage scales with the number of matches, not document size

### Search Speed

- Simple search is faster for basic text matching
- Segmenter search provides better accuracy for word boundaries
- Performance depends on document size and search complexity

## Custom Search Implementation

### Building a Search UI

```javascript
class SearchManager {
    constructor(view) {
        this.view = view
        this.currentResults = []
        this.currentIndex = -1
    }
    
    async performSearch(query, options = {}) {
        // Clear previous search
        this.view.clearSearch()
        this.currentResults = []
        this.currentIndex = -1
        
        if (!query.trim()) return
        
        // Perform search
        for await (const result of this.view.search({ query, ...options })) {
            this.currentResults.push(result)
        }
        
        // Highlight first result
        if (this.currentResults.length > 0) {
            this.goToResult(0)
        }
        
        return this.currentResults.length
    }
    
    goToResult(index) {
        if (index < 0 || index >= this.currentResults.length) return
        
        this.currentIndex = index
        const result = this.currentResults[index]
        
        // Navigate to the result
        this.view.goTo(result.range)
        
        // Update UI
        this.updateSearchUI()
    }
    
    nextResult() {
        this.goToResult(this.currentIndex + 1)
    }
    
    previousResult() {
        this.goToResult(this.currentIndex - 1)
    }
    
    clearSearch() {
        this.view.clearSearch()
        this.currentResults = []
        this.currentIndex = -1
        this.updateSearchUI()
    }
    
    updateSearchUI() {
        const count = this.currentResults.length
        const current = this.currentIndex + 1
        
        if (count > 0) {
            console.log(`Result ${current} of ${count}`)
        } else {
            console.log('No results found')
        }
    }
}

// Usage
const searchManager = new SearchManager(view)

// Search input handler
searchInput.addEventListener('input', async (e) => {
    const query = e.target.value
    const count = await searchManager.performSearch(query, {
        matchCase: false,
        matchWholeWords: true
    })
    
    console.log(`Found ${count} results`)
})

// Navigation buttons
nextButton.addEventListener('click', () => searchManager.nextResult())
prevButton.addEventListener('click', () => searchManager.previousResult())
clearButton.addEventListener('click', () => searchManager.clearSearch())
```

### Advanced Search with Filters

```javascript
class AdvancedSearch {
    constructor(view) {
        this.view = view
    }
    
    async searchWithFilters(query, filters = {}) {
        const results = []
        
        for await (const result of this.view.search({ query })) {
            // Apply custom filters
            if (this.applyFilters(result, filters)) {
                results.push(result)
            }
        }
        
        return results
    }
    
    applyFilters(result, filters) {
        // Filter by section
        if (filters.sections && !filters.sections.includes(result.range.startIndex)) {
            return false
        }
        
        // Filter by text length
        if (filters.minLength && result.excerpt.match.length < filters.minLength) {
            return false
        }
        
        // Filter by context
        if (filters.context) {
            const context = result.excerpt.pre + result.excerpt.post
            if (!context.toLowerCase().includes(filters.context.toLowerCase())) {
                return false
            }
        }
        
        return true
    }
}

// Usage
const advancedSearch = new AdvancedSearch(view)

const results = await advancedSearch.searchWithFilters('important', {
    sections: [0, 1, 2], // Only search in first 3 sections
    minLength: 5, // Minimum match length
    context: 'chapter' // Must appear near "chapter"
})
```

## Error Handling

```javascript
try {
    for await (const result of view.search({ query: 'search term' })) {
        console.log(result)
    }
} catch (error) {
    if (error.name === 'TypeError' && error.message.includes('Segmenter')) {
        console.error('Intl.Segmenter not supported, falling back to simple search')
    } else {
        console.error('Search failed:', error)
    }
}
```

## Browser Support

- **Simple Search**: Works in all modern browsers
- **Segmenter Search**: Requires `Intl.Segmenter` support (Chrome 87+, Firefox 102+, Safari 15+)

The library automatically falls back to simple search when `Intl.Segmenter` is not available.

## Best Practices

1. **Debounce Search Input**: Avoid performing search on every keystroke
2. **Limit Results**: Consider limiting the number of results for large documents
3. **Provide Feedback**: Show loading indicators and result counts
4. **Handle Errors**: Gracefully handle search failures
5. **Clear Previous Searches**: Always clear previous search highlights before new searches 