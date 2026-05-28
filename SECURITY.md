# Security Policy — ZeroCloudPDF

**Last updated:** 2026-05-28  
**Scope:** All browser-native PDF conversion tools, the Vault (cloud storage layer), and the thumbnail generation pipeline.

---

## 1. Security Model: Browser-Native by Design

ZeroCloudPDF processes **all** PDF conversions inside your browser using pure JavaScript.  
No file ever leaves your device during conversion.

| Component | Technology | Server Contact? |
|---|---|---|
| PDF rendering | Mozilla pdf.js 3.11.174 | ❌ None |
| PDF generation | jsPDF 2.5.1 | ❌ None |
| Word-to-PDF | mammoth.js 1.6.0 | ❌ None |
| DOM-to-image (Word image mode) | html2canvas 1.4.1 | ❌ None |
| HEIC decoding | Browser-native `createImageBitmap()` | ❌ None¹ |

¹ Native HEIC decoding is supported on Safari and iOS only. Chrome and Firefox desktop do not natively decode HEIC; conversions may silently fail or produce blank output.

**Verification:** Open DevTools → Network tab. Perform any conversion. You will see zero outbound requests containing file data. We call this the **Zero Server Contact Verification**.

---

## 2. Vault Security Architecture

The Vault is the **only** server-touching feature. It allows authenticated users to store original files and generated PDFs in cloud storage.

### 2.1 Authentication
- Firebase Auth (email/password or Google OAuth)
- Session persistence: `SESSION` only
- **Email verification is enforced.** Unverified accounts cannot access the Vault.

### 2.2 Upload Flow
1. Client requests a signed upload URL from our Cloud Run backend (`POST /get-upload-url`).
2. Backend validates the Firebase ID token.
3. Client `PUT`s the file **directly** to Google Cloud Storage via the signed URL.
4. Files are stored at: `User/{uid}/original/{timestamp}-{filename}`

### 2.3 Data-at-Rest Status
| Question | Status | Note |
|---|---|---|
| Is Vault data encrypted at rest by ZeroCloudPDF? | **No.** | We rely on GCS default encryption (Google-managed keys). We do not apply client-side or server-side additional encryption before upload. |
| Is Vault data encrypted in transit? | Yes. | All traffic is TLS 1.2+. |
| Are files automatically deleted? | **No.** | No GCS lifecycle rules are configured. Files persist until the user deletes them via the Vault UI. |
| Are thumbnails retained? | Yes. | Thumbnails and previews are generated and stored alongside originals. |

### 2.4 Thumbnail Pipeline
- `generate-thumbnail` Cloud Run service (`asia-south1`)
- Downloads the **full original file** to `/tmp/` for processing
- Temp files (`/tmp/thumb-*`, `/tmp/preview-*`) are **not explicitly cleaned up** and remain until the Cloud Run instance recycles
- Only `originalTemp` is explicitly `unlinkSync`-ed

### 2.5 Logging Policy
- **Zero explicit logging** in the success path for upload/download/delete operations
- No filenames, UIDs, file sizes, or content types are logged
- Only `console.error(err)` in catch blocks for debugging
- Thumbnail service logs emoji-only success indicators (e.g., "✅ Image processed") and `console.error` on failure

### 2.6 Known Risk: Orphaned Files
If a user deletes their Firebase account **without** first deleting Vault files via the UI, the GCS objects become permanent orphans. There is currently no automated cleanup mechanism.

---

## 3. What We Do NOT Do

To avoid scope confusion, we explicitly state:

- ❌ **No WebAssembly** in any processing path
- ❌ **No Service Worker** — no `sw.js`, no `manifest.json`, no PWA registration
- ❌ **No ES module dynamic `import()`** — jsPDF, pdf.js, and mammoth.js load via deferred script tags on every page load. html2canvas loads dynamically on demand when Word Image mode is selected.
- ❌ **No client-side encryption** before Vault upload
- ❌ **No server-side PDF processing** for conversion tools

---

## 4. Supported Versions

| Version | Supported |
|---|---|
| Latest `main` branch | ✅ |
| Any pinned release | ✅ |
| Older commits | ❌ |

We do not maintain LTS branches. Always use the latest deployed version.

---

## 5. Reporting a Vulnerability

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

## 6. Security Checklist for Self-Audit

If you are evaluating ZeroCloudPDF for sensitive documents (bank statements, passports, medical records, school certificates), verify:

- [ ] You are using the conversion tools **without logging into the Vault** (zero server contact)
- [ ] If using the Vault, your email is verified and you understand files are stored unencrypted at rest
- [ ] You have reviewed the Network tab in DevTools during conversion
- [ ] You have read [ADR-001: Why Browser-Native, Not WebAssembly](docs/adr/001-why-browser-native-not-wasm.md)
- [ ] You have read [ADR-003: Zero Server Contact Verification Methodology](docs/adr/003-zero-server-contact-verification.md) *(coming soon)*

---

## 7. Credits & References

- Mozilla pdf.js Security: https://github.com/mozilla/pdf.js/security
- jsPDF: https://github.com/parallax/jsPDF
- Google Cloud Storage Encryption: https://cloud.google.com/storage/docs/encryption

---

*This policy is a living document. If you believe any statement here is inaccurate, please open a GitHub Discussion in the "Privacy & Compliance" category or email security@zerocloudpdf.com.*
