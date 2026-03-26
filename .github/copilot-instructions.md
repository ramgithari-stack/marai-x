# MARAI-X Encryption Platform - AI Agent Guide

## Project Overview

MARAI-X is a **dual-architecture** encryption platform combining a static HTML landing page with an embedded React app. The project offers client-side encryption for multiple data types including text, images, files, audio, and video with advanced features like steganography, dual-file encryption, and time-lock mechanisms.

## Architecture

### Dual Entry Points
- **Static site**: [index.html](index.html) - Full-featured marketing page with embedded demos powered by [script.js](script.js)
- **React app**: [src/main.tsx](src/main.tsx) → [src/App.tsx](src/App.tsx) - Minimal starter (currently a placeholder)

The static site is the **primary working application**. React components are not yet integrated with the encryption engine.

### Key Components

1. **Encryption Engine** ([script.js](script.js), lines 1-150)
   - Web Crypto API with AES-256-GCM
   - PBKDF2 key derivation (150,000 iterations)
   - Hard time-lock enforcement checked **before** decryption attempts
   - All operations are client-side only

2. **Steganography Module** ([script.js](script.js), lines 220-270)
   - LSB (Least Significant Bit) image steganography
   - Zero-width character stealth for text files
   - Binary marker-based payload hiding in audio/video

3. **Dual-File Encryption** ([script.js](script.js), lines 900-1231)
   - Decoy + secret file pattern using magic bytes (MRXI, MRXF, etc.)
   - Payload appended to decoy with length prefix
   - Format: `[DECOY_DATA][ENCRYPTED_PAYLOAD][LENGTH:4bytes][MAGIC:4bytes]`

## Development Workflow

### Setup & Build
```bash
npm install          # Install dependencies
npm run dev          # Vite dev server (React app only)
npm run build        # Build for production
npm run typecheck    # TypeScript validation
```

### Static Site Development
- Edit [index.html](index.html) for UI/structure
- Edit [script.js](script.js) for encryption logic
- Edit [styles.css](styles.css) for styling (custom CSS, not Tailwind-based)
- No build step needed - open [index.html](index.html) directly in browser

### React Integration (Future)
- React setup exists but is **not yet connected** to the encryption engine
- To integrate: import crypto functions from [script.js](script.js) into React components
- Vite config excludes `lucide-react` from optimization due to bundling issues

## Critical Patterns

### Time-Lock Implementation
```javascript
// ALWAYS enforce time-lock BEFORE attempting decryption
function enforceTimeLock(payloadObj) {
  if (payloadObj.m?.unlockAt && Date.now() < payloadObj.m.unlockAt) {
    throw `⏳ Locked. Try again in ${Math.ceil((payloadObj.m.unlockAt - Date.now()) / 1000)}s`;
  }
}
```

### File Size Handling
- Images: 100MB limit
- Audio: 200MB limit  
- Video/Files: 500MB limit
- Always prompt user confirmation for large files to avoid browser crashes

### Encryption Metadata Structure
```javascript
{
  v: "MARAI-X-1",        // Version identifier
  s: [...],               // Salt (16 bytes)
  i: [...],               // IV (12 bytes)
  d: "base64_data",       // Encrypted data
  m: {                    // Metadata
    unlockAt: timestamp,  // Optional time-lock
    name: "file.ext",     // Original filename
    type: "mime/type"     // MIME type
  }
}
```

## UI Conventions

- **Demo tabs**: Use `data-tab` attribute for panel switching
- **Security options**: Consistent `.opt-timelock`, `.opt-time`, `.opt-onetime` class names
- **Theme toggle**: Persists via `localStorage.setItem("theme", value)`
- **Upload areas**: Hide file inputs, trigger via click events on styled divs

## Integration Points

- **Supabase**: Dependency installed but not actively used
- **Lucide React**: Icon library for React (excluded from Vite optimization)
- **Tailwind CSS**: Configured but only used in React app, static site uses custom CSS

## Testing Encryption Features

1. **Text encryption**: Use text/password panels with time-lock option
2. **Stealth TXT**: Requires decoy text + file upload
3. **Dual modes**: Always require TWO file uploads (decoy + secret)
4. **Stego image**: Message embedded in LSB, visually identical output

## Common Pitfalls

- Don't modify encryption payload structure - decryption depends on exact format
- Time-lock must be checked in `decryptPayload()`, not just UI validation
- React components don't have access to `script.js` functions without explicit imports
- File downloads use `URL.createObjectURL()` - always revoke URLs after use

## File Organization

```
├── index.html           # Static site (primary app)
├── script.js            # Complete encryption engine
├── styles.css           # Custom CSS for static site
├── password-vault.js    # Empty placeholder file
├── src/
│   ├── App.tsx          # React placeholder
│   ├── main.tsx         # React entry
│   └── index.css        # Tailwind imports
└── vite.config.ts       # Build config
```

## Security Principles

- **Zero-knowledge**: All encryption/decryption happens in browser
- **No tracking**: No analytics, telemetry, or server-side storage
- **Client-side only**: Never transmit unencrypted data
- **Time-lock enforcement**: Hard-coded checks prevent bypass attempts
