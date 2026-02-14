# Security Guidelines

## Overview

This document outlines the security model and best practices for using Clamp Calculator.

## Security Properties ✅

### What is Secure

- **Local Execution**: All JavaScript runs in your browser; no data leaves your computer
- **No External Dependencies**: No npm packages, no CDN libraries, no network calls
- **XSS Protection**: Uses `textContent` instead of `innerHTML` to prevent injection attacks
- **No Eval**: No `eval()`, `Function()`, or dynamic code execution
- **No Persistent Storage**: No cookies, localStorage, or IndexedDB usage
- **No Telemetry**: No analytics, tracking, or diagnostic data collection
- **Input Validation**: Uses `type="number"` and regex parsing with safe DOM manipulation

### When It's Safe to Use

✅ **Recommended:**
- Local development on your machine
- Team/internal use on intranets
- Self-hosted deployment with HTTPS
- Sensitive project design systems
- Offline environments

## Deployment Considerations

### Local File Access (file://)

```html
<!-- SECURE: Open directly in browser -->
<double-click clamp-calculator.html>
```

**Why it's safe:**
- Runs entirely in browser memory
- No backend or server involved
- File:// protocol has same-origin restrictions

### Public HTTPS Hosting

If deploying publicly, implement security headers:

```nginx
# Example nginx configuration
add_header Content-Security-Policy "default-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline'" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
```

**Or update HTML meta tags:**
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline'">  
<meta http-equiv="X-UA-Compatible" content="ie=edge">
```

### Internet ❌ Not Recommended

Do NOT use on public internet without additional measures:
- 🔴 No server authentication
- 🔴 Users might copy/paste sensitive data via clipboard API
- 🔴 No rate limiting or DDoS protection
- 🔴 Man-in-the-middle risks (use HTTPS only)

## Input Validation

### Current Validations

```javascript
// Type checking
const min = parseFloat(document.getElementById('mobileMin').value);
if (isNaN(min)) { /* skip calculation */ }

// Regex parsing (Reverse Tool)
const regex = /clamp\(\s*([\d.]+)rem,\s*([\d.]+)vw\s*\+\s*([\d.]+)rem,\s*([\d.]+)rem\s*\)/i;
const match = input.match(regex);
if (!match) { resultEl.textContent = "Invalid format..."; }
```

### Recommended Enhancements

If modifying code for enhanced validation:

```javascript
// Strict bounds checking
function validateFontSize(px) {
  const num = parseFloat(px);
  if (isNaN(num)) return false;
  return num >= 4 && num <= 500; // Reasonable font size range
}

// Validate viewport widths
function validateWidth(width) {
  const num = parseFloat(width);
  return num >= 320 && num <= 2560; // Realistic viewport range
}
```

## Clipboard Integration

**Feature**: One-click copy of calculated values

```javascript
navigator.clipboard.writeText(textToCopy).then(() => { /* ... */ });
```

**Security Notes:**
- Requires HTTPS on public deployments
- User permission popup on first use (browser-controlled)
- Only copies internally-generated values (safe)
- No access to clipboard read operations

## Future Enhancements

### Recommended Improvements

1. **Content Security Policy (CSP)**
   - Migrate inline scripts to external `app.js` file
   - Update meta tags and remove `'unsafe-inline'`
   
2. **Subresource Integrity (SRI)**
   - If adding any external assets, use SRI hashes
   
3. **Form-based Event Handlers**
   - Replace `onclick="..."` attributes with `addEventListener()`
   - See [example implementation](#event-handler-modernization)

4. **Input Range Validation**
   - Add numerical bounds checking per field
   - Prevent degenerate cases (negative widths, etc.)

### Event Handler Modernization

```javascript
// Recommended pattern instead of onclick="handleMobileInput('min')"
document.getElementById('mobileMin').addEventListener('input', () => {
  handleMobileInput('min');
  updateAll();
});
```

## Vulnerability Disclosure

If you discover a security issue:

1. **Do NOT** open a public GitHub issue
2. **Email** with subject: `[SECURITY] Clamp Calculator Vulnerability`
3. Include:
   - Description of vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested remediation

## Browser Security Features

This tool benefits from built-in browser protections:

- **Same-Origin Policy**: Prevents cross-origin access
- **Sandbox Restrictions**: Local files have limited API access
- **Type Safety**: JavaScript type coercion is predictable
- **CSP Fallback**: `default-src 'self'` (if server-enforced)

## Compliance

This tool does NOT require:
- ✅ GDPR compliance (no personal data)
- ✅ HIPAA compliance (no health data)
- ✅ PCI-DSS compliance (no payment data)
- ✅ SOC 2 certification

**Data Handling**: User is the sole custodian of all data; nothing is transmitted.

## Version

**Security Document Version**: 1.0  
**Last Updated**: 2026-02-15  
**Clamp Calculator Version**: 3.1+
