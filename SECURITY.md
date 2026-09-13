# Security Policy — ZeroCloudPDF

**Last updated:** 2026-07-25  
**Scope:** All browser-native PDF conversion and editing tools.

---

## 1. Security Model: Browser-Native by Design

ZeroCloudPDF processes **all** PDF conversions and editing operations inside your browser using pure JavaScript and WebAssembly (for encryption only).  
No file ever leaves your device during any operation.

| Component | Technology | Server Contact? |
|---|---|---|
| PDF rendering | Mozilla pdf.js 3.11.174 | ❌ None |
| PDF generation | jsPDF 2.5.1 | ❌ None |
| Word-to-PDF | mammoth.js 1.6.0 | ❌ None |
| PDF encryption/decryption | qpdf.js (WebAssembly) | ❌ None |
| HEIC decoding | Browser-native `createImageBitmap()` (Safari/iOS only) | ❌ None¹ |
| Image processing | Native Canvas, ImageBitmap, OffscreenCanvas APIs | ❌ None |

¹ Native HEIC decoding is supported on Safari and iOS only. Chrome and Firefox desktop do not natively decode HEIC; conversions may silently fail or produce blank output.

**External dependencies:** Google Fonts (fonts.googleapis.com, fonts.gstatic.com) load on every page for typography. These requests may expose IP addresses and user agents to Google. They do not contain file data.

**Verification:** Open DevTools → Network tab. Perform any conversion or editing operation. You will see zero outbound requests containing file data. We call this the **Zero Server Contact Verification**.

See [ADR-003: Zero Server Contact Verification Methodology](docs/adr/003-zero-server-contact-verification.md) for detailed audit instructions.

---

## 2. Protect/Unlock PDF Security Architecture

The Protect PDF and Unlock PDF tools use **qpdf.js**, a WebAssembly port of the qpdf library, for cryptographic operations.

### 2.1 Encryption Specifications
- **Algorithm:** AES-256 in CBC mode
- **Key derivation:** Industry-standard password-based key derivation
- **Security level:** 256-bit encryption keys
- **Compliance:** qpdf is widely used in production environments and has been audited for security

### 2.2 How It Works
1. User selects a PDF file in their browser
2. qpdf.js WASM module loads from CDN (cdnjs.cloudflare.com or cdn.jsdelivr.net)
3. Password is entered locally (never transmitted)
4. Encryption/decryption occurs entirely in browser memory
5. Output PDF is generated and downloaded directly to user's device

### 2.3 Security Guarantees
| Question | Answer |
|---|---|
| Are passwords transmitted to any server? | **No.** Passwords never leave the browser. |
| Is the WASM module auditable? | Yes. Source maps and unminified versions available. |
| Are temporary files created on servers? | **No.** All processing is in-browser. |
| Can ZeroCloudPDF access encrypted files? | **No.** We have zero access to any files. |

---

## 3. Library Loading & CDN Security

### 3.1 CDN Sources
All processing libraries load from trusted, reputable CDNs:

| Library | Primary CDN | Fallback CDN |
|---------|-------------|--------------|
| pdf.js | cdnjs.cloudflare.com | cdn.jsdelivr.net |
| jsPDF | cdnjs.cloudflare.com | cdn.jsdelivr.net |
| mammoth.js | cdnjs.cloudflare.com | cdn.jsdelivr.net |
| qpdf.js | cdnjs.cloudflare.com | cdn.jsdelivr.net |

### 3.2 Loading Strategy
- Libraries load via **deferred script tags** (`defer` attribute)
- Each tool loads its required libraries **on-demand** (not all at once)
- No dynamic `import()` or ES module loading for core libraries
- No Service Worker registration
- No PWA manifest

### 3.3 Subresource Integrity (SRI)
We recommend implementing SRI hashes for all CDN-loaded libraries to prevent tampering. Check our deployed site for current SRI implementations.

---

## 4. What We Do NOT Do (For Any PDF Tool)

To avoid scope confusion, every statement below applies to all PDF conversion and editing tools:

- ❌ **No server-side PDF processing** for any conversion or editing tool
- ❌ **No file uploads** for any PDF tool (no Firebase, no GCS, no Cloud Run)
- ❌ **No account requirement** to use any PDF tool
- ❌ **No client-side encryption before upload** (because no PDF tool uploads anything)
- ❌ **No logging** of filenames, file sizes, or content types by any PDF tool
- ❌ **No error telemetry** that transmits file data

**Note:** zerocloudpdf.com also runs a separate, unlisted legacy feature (a private cloud vault) that does use Firebase Authentication and Google Cloud Storage for its own account holders. It is not linked anywhere on the site, is not under active development, and no PDF tool interacts with it in any way.

---

## 5. Supported Versions

| Version | Supported |
|---|---|
| Latest deployed version on zerocloudpdf.com | ✅ |
| Any pinned release tag | ✅ |
| Older commits | ❌ |

We do not maintain LTS branches. Always use the latest deployed version.

---

## 6. Reporting a Vulnerability

**Please do NOT open public issues for security bugs.**

Instead, email: **security@zerocloudpdf.com** (or contact the maintainer directly if this address is not yet active).

**What to include:**
- A clear description of the vulnerability
- Steps to reproduce
- Impact assessment (what data is at risk?)
- Whether you have tested against the live site or a local clone

**Response timeline:**
- Acknowledgment within 48 hours
- Initial assessment within 7 days
- Fix or mitigation plan within 30 days for confirmed critical/high issues
- Public disclosure coordinated with the reporter after fix deployment

We follow a **coordinated disclosure** model. We do not offer a bug bounty program at this time.

---

## 7. Security Checklist for Self-Audit

If you are evaluating ZeroCloudPDF for sensitive documents (bank statements, passports, medical records, school certificates), verify:

- [ ] You are using the tools **without any login requirement** (zero server contact)
- [ ] You have reviewed the Network tab in DevTools during conversion/editing
- [ ] You understand Protect/Unlock uses qpdf.js WASM with 256-bit AES encryption
- [ ] You have read [ADR-001: Why Browser-Native, Not WebAssembly](docs/adr/001-why-browser-native-not-wasm.md) (note: Protect/Unlock is the exception)
- [ ] You have read [ADR-002: Client-Side Only Architecture](docs/adr/002-client-side-only-architecture.md)
- [ ] You have read [ADR-003: Zero Server Contact Verification Methodology](docs/adr/003-zero-server-contact-verification.md)

---

## 8. Credits & References

- Mozilla pdf.js Security: https://github.com/mozilla/pdf.js/security
- jsPDF: https://github.com/parallax/jsPDF
- qpdf.js: https://github.com/jsejcksn/qpdf.js
- qpdf Documentation: https://qpdf.readthedocs.io/

---

*This policy is a living document. If you believe any statement here is inaccurate, please open a GitHub Discussion in the "Privacy & Compliance" category or email security@zerocloudpdf.com.*
