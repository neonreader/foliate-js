# Security Considerations

Security is a critical aspect when working with e-books, especially EPUB files that can contain executable content. This document outlines the security risks and best practices for using Foliate.js safely.

## ⚠️ Critical Security Warning

**EPUB books can contain scripted content (JavaScript) which is potentially dangerous.** Foliate.js does not support scripted content due to security concerns, but you must implement proper security measures to protect your users.

## Security Risks

### 1. Script Injection

EPUB files can contain JavaScript code that could:
- Access user data and cookies
- Perform network requests
- Modify the DOM
- Access browser APIs
- Execute malicious code

### 2. Content Security Policy (CSP) Bypass

Without proper CSP, scripts in EPUBs could bypass your application's security controls.

### 3. Font Obfuscation

EPUB fonts may be obfuscated, requiring cryptographic operations that could be exploited.

### 4. External Resources

EPUBs can reference external resources that may be malicious or compromised.

## Required Security Measures

### 1. Content Security Policy (CSP)

**You MUST implement CSP** to block all scripts except those from your own origin:

```html
<meta http-equiv="Content-Security-Policy" 
      content="script-src 'self'; object-src 'none';">
```

#### Comprehensive CSP Example

```html
<meta http-equiv="Content-Security-Policy" 
      content="
        default-src 'self';
        script-src 'self';
        object-src 'none';
        style-src 'self' 'unsafe-inline';
        img-src 'self' data: blob:;
        font-src 'self' data:;
        media-src 'self' data: blob:;
        connect-src 'self';
        frame-src 'none';
        base-uri 'self';
        form-action 'self';
      ">
```

#### CSP for Different Environments

**Development:**
```html
<meta http-equiv="Content-Security-Policy" 
      content="script-src 'self' 'unsafe-eval'; object-src 'none';">
```

**Production:**
```html
<meta http-equiv="Content-Security-Policy" 
      content="script-src 'self'; object-src 'none';">
```

### 2. HTTPS Requirement

**Always serve your application over HTTPS** because:
- Font deobfuscation requires Web Crypto API
- Web Crypto API is only available in secure contexts
- Protects against man-in-the-middle attacks

```javascript
// Check if running in secure context
if (!window.isSecureContext) {
    console.error('Application must run over HTTPS for font deobfuscation')
}
```

### 3. File Validation

Validate EPUB files before processing:

```javascript
class EPUBValidator {
    static async validateFile(file) {
        const errors = []
        
        // Check file size
        if (file.size > 100 * 1024 * 1024) { // 100MB limit
            errors.push('File too large')
        }
        
        // Check file type
        if (!file.name.toLowerCase().endsWith('.epub')) {
            errors.push('Invalid file type')
        }
        
        // Check file signature
        const buffer = await file.slice(0, 4).arrayBuffer()
        const signature = new Uint8Array(buffer)
        const epubSignature = [0x50, 0x4B, 0x03, 0x04] // ZIP signature
        
        if (!signature.every((byte, i) => byte === epubSignature[i])) {
            errors.push('Invalid EPUB file signature')
        }
        
        return errors
    }
}

// Usage
const file = event.target.files[0]
const errors = await EPUBValidator.validateFile(file)
if (errors.length > 0) {
    console.error('File validation failed:', errors)
    return
}
```

### 4. Sandboxed Rendering

Use iframe sandboxing when possible:

```javascript
// Create sandboxed iframe for content
const iframe = document.createElement('iframe')
iframe.sandbox = 'allow-same-origin allow-scripts'
iframe.src = 'about:blank'
document.body.appendChild(iframe)
```

**Note**: Due to [WebKit Bug 218086](https://bugs.webkit.org/show_bug.cgi?id=218086), iframe sandboxing may not be fully effective for EPUB content.

## Security Best Practices

### 1. Input Sanitization

Always sanitize user inputs:

```javascript
class InputSanitizer {
    static sanitizeFilename(filename) {
        // Remove path traversal attempts
        return filename.replace(/\.\./g, '').replace(/[\/\\]/g, '')
    }
    
    static sanitizeURL(url) {
        // Validate URLs
        try {
            const parsed = new URL(url)
            if (!['http:', 'https:'].includes(parsed.protocol)) {
                throw new Error('Invalid protocol')
            }
            return url
        } catch {
            throw new Error('Invalid URL')
        }
    }
}
```

### 2. Resource Loading

Implement safe resource loading:

```javascript
class SafeResourceLoader {
    constructor() {
        this.allowedDomains = ['trusted-domain.com']
        this.blockedPatterns = [/\.exe$/, /\.bat$/, /\.sh$/]
    }
    
    async loadResource(url) {
        // Check if URL is allowed
        if (!this.isAllowedURL(url)) {
            throw new Error('Resource not allowed')
        }
        
        // Check for blocked file types
        if (this.isBlockedFile(url)) {
            throw new Error('File type not allowed')
        }
        
        // Load resource safely
        return await this.fetchResource(url)
    }
    
    isAllowedURL(url) {
        try {
            const parsed = new URL(url)
            return this.allowedDomains.includes(parsed.hostname)
        } catch {
            return false
        }
    }
    
    isBlockedFile(url) {
        return this.blockedPatterns.some(pattern => pattern.test(url))
    }
}
```

### 3. Error Handling

Implement secure error handling:

```javascript
class SecureErrorHandler {
    static handleError(error, context) {
        // Log error securely (don't expose sensitive information)
        console.error('Error in', context, error.name, error.message)
        
        // Don't expose internal details to users
        if (error instanceof ResponseError) {
            return 'Network error occurred'
        } else if (error instanceof UnsupportedTypeError) {
            return 'File format not supported'
        } else {
            return 'An error occurred while processing the file'
        }
    }
}
```

### 4. Memory Management

Prevent memory-based attacks:

```javascript
class MemoryManager {
    constructor() {
        this.maxFileSize = 100 * 1024 * 1024 // 100MB
        this.maxMemoryUsage = 500 * 1024 * 1024 // 500MB
    }
    
    checkMemoryUsage() {
        if (performance.memory) {
            const used = performance.memory.usedJSHeapSize
            if (used > this.maxMemoryUsage) {
                throw new Error('Memory usage exceeded limit')
            }
        }
    }
    
    async validateFileSize(file) {
        if (file.size > this.maxFileSize) {
            throw new Error('File size too large')
        }
    }
}
```

## Security Configuration

### 1. Application Security Headers

Set appropriate security headers:

```javascript
// Server-side headers (Node.js/Express example)
app.use((req, res, next) => {
    res.setHeader('X-Content-Type-Options', 'nosniff')
    res.setHeader('X-Frame-Options', 'DENY')
    res.setHeader('X-XSS-Protection', '1; mode=block')
    res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin')
    next()
})
```

### 2. CORS Configuration

Configure CORS properly:

```javascript
// Server-side CORS configuration
app.use(cors({
    origin: ['https://yourdomain.com'],
    methods: ['GET', 'POST'],
    allowedHeaders: ['Content-Type'],
    credentials: false
}))
```

### 3. File Upload Security

Secure file uploads:

```javascript
class SecureFileUpload {
    static allowedTypes = ['.epub', '.mobi', '.azw3', '.fb2', '.cbz', '.pdf']
    static maxSize = 100 * 1024 * 1024 // 100MB
    
    static validateUpload(file) {
        // Check file type
        const extension = file.name.toLowerCase().substring(file.name.lastIndexOf('.'))
        if (!this.allowedTypes.includes(extension)) {
            throw new Error('File type not allowed')
        }
        
        // Check file size
        if (file.size > this.maxSize) {
            throw new Error('File too large')
        }
        
        // Check MIME type
        if (!this.isValidMimeType(file.type)) {
            throw new Error('Invalid MIME type')
        }
        
        return true
    }
    
    static isValidMimeType(mimeType) {
        const validTypes = [
            'application/epub+zip',
            'application/x-mobipocket-ebook',
            'application/x-fictionbook+xml',
            'application/vnd.comicbook+zip',
            'application/pdf'
        ]
        return validTypes.includes(mimeType) || mimeType === ''
    }
}
```

## Testing Security

### 1. Security Testing Checklist

```javascript
class SecurityTester {
    static async runSecurityTests() {
        const tests = [
            this.testCSP(),
            this.testHTTPS(),
            this.testFileValidation(),
            this.testResourceLoading(),
            this.testErrorHandling()
        ]
        
        const results = await Promise.all(tests)
        return results.every(result => result.passed)
    }
    
    static async testCSP() {
        // Test if CSP is properly configured
        const meta = document.querySelector('meta[http-equiv="Content-Security-Policy"]')
        return {
            passed: !!meta && meta.content.includes('script-src'),
            message: 'CSP not properly configured'
        }
    }
    
    static async testHTTPS() {
        // Test if running over HTTPS
        return {
            passed: window.isSecureContext,
            message: 'Not running over HTTPS'
        }
    }
}
```

### 2. Malicious File Testing

Test with known malicious EPUBs:

```javascript
// Test files (DO NOT use in production)
const testFiles = [
    'epub-test.epub', // Contains scripted content
    'malicious-font.epub', // Contains malicious fonts
    'external-resource.epub' // References external resources
]

// Run security tests
for (const file of testFiles) {
    try {
        await view.open(file)
        console.error('Security test failed: malicious file opened')
    } catch (error) {
        console.log('Security test passed: malicious file blocked')
    }
}
```

## Incident Response

### 1. Security Incident Plan

```javascript
class SecurityIncidentHandler {
    static async handleIncident(incident) {
        // 1. Log the incident
        this.logIncident(incident)
        
        // 2. Block the source
        this.blockSource(incident.source)
        
        // 3. Notify administrators
        await this.notifyAdmins(incident)
        
        // 4. Take corrective action
        this.takeCorrectiveAction(incident)
    }
    
    static logIncident(incident) {
        console.error('Security incident:', {
            type: incident.type,
            source: incident.source,
            timestamp: new Date().toISOString(),
            userAgent: navigator.userAgent
        })
    }
    
    static blockSource(source) {
        // Implement source blocking logic
        localStorage.setItem('blocked-sources', 
            JSON.stringify([...this.getBlockedSources(), source]))
    }
}
```

### 2. Monitoring and Alerting

```javascript
class SecurityMonitor {
    constructor() {
        this.suspiciousActivities = []
        this.setupMonitoring()
    }
    
    setupMonitoring() {
        // Monitor for suspicious activities
        window.addEventListener('error', (e) => {
            this.checkForSuspiciousActivity(e)
        })
        
        // Monitor network requests
        this.monitorNetworkRequests()
    }
    
    checkForSuspiciousActivity(event) {
        const suspicious = [
            'eval',
            'Function',
            'setTimeout',
            'setInterval'
        ]
        
        if (suspicious.some(term => event.message.includes(term))) {
            this.reportSuspiciousActivity(event)
        }
    }
}
```

## Compliance and Standards

### 1. OWASP Guidelines

Follow OWASP security guidelines:
- Input validation
- Output encoding
- Authentication and session management
- Access control
- Security configuration
- Secure communication

### 2. GDPR Compliance

Ensure GDPR compliance when handling user data:

```javascript
class GDPRCompliance {
    static getUserConsent() {
        // Get explicit consent for data processing
        return localStorage.getItem('gdpr-consent') === 'true'
    }
    
    static anonymizeData(data) {
        // Anonymize user data
        return {
            ...data,
            userId: this.hashUserId(data.userId),
            personalInfo: null
        }
    }
    
    static hashUserId(userId) {
        // Hash user ID for privacy
        return btoa(userId).replace(/[^a-zA-Z0-9]/g, '')
    }
}
```

## Security Checklist

Before deploying your application, ensure you have:

- [ ] Implemented Content Security Policy
- [ ] Serving over HTTPS
- [ ] Validating all file uploads
- [ ] Sanitizing user inputs
- [ ] Implementing proper error handling
- [ ] Setting up monitoring and alerting
- [ ] Testing with malicious files
- [ ] Documenting security procedures
- [ ] Training team on security practices
- [ ] Regular security audits

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [WebKit Bug 218086](https://bugs.webkit.org/show_bug.cgi?id=218086)
- [EPUB Security Considerations](https://www.w3.org/TR/epub/#sec-scripted-content) 