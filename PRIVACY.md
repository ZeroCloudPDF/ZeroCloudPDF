# Privacy Policy — ZeroCloudPDF

**Last updated:** 2026-07-25  
**Effective date:** 2026-07-25  

---

## 1. Privacy Model: Zero Server Contact

ZeroCloudPDF is built on a **privacy-first, browser-native** architecture.

| Activity | Data Leaves Your Device? | What We Collect |
|---|---|---|
| PDF conversion (all tools) | **No** | Nothing. Zero data. |
| PDF encryption/decryption (Protect/Unlock) | **No** | Nothing. qpdf.js runs locally. |
| Image-to-PDF conversions | **No** | Nothing. |
| Word-to-PDF conversion | **No** | Nothing. |
| HEIC-to-PDF | **No** | Nothing. |
| PDF editing (rotate, delete, sign, redact) | **No** | Nothing. |

**All processing happens in your browser.** Your files never leave your device during any conversion or editing operation.

---

## 2. What We Do Collect

### 2.1 Anonymous Usage Statistics
We use **Google Analytics 4 (GA4)** with tracking ID `G-ESZCDHN3HT` to collect anonymous usage statistics such as pages visited, browser type, session duration, and general location. GA4 may set analytics cookies. It does **not** receive any information about the files you process or the content of your conversions. See [Google's privacy policy](https://policies.google.com/privacy) for details on how GA4 data is handled.

### 2.2 Google Fonts
This site uses **Google Fonts** loaded from `fonts.googleapis.com` and `fonts.gstatic.com` for typography. These requests may expose your IP address and user agent to Google per their CDN policies. No file data is transmitted.

### 2.3 What We Do NOT Collect (Beyond the Unlisted Legacy Vault Feature Described in §2.4)
We do not use:

- ❌ Mixpanel or any additional visitor tracking scripts beyond GA4
- ❌ Fingerprinting or device identification
- ❌ Server-side logs of conversion activity
- ❌ Error telemetry from conversion tools
- ❌ Advertising cookies or retargeting pixels
- ❌ Third-party analytics beyond GA4
- ❌ User authentication or account systems (for PDF tools)
- ❌ File storage or cloud upload capabilities (for PDF tools)

### 2.4 Legacy Vault Feature (Unlisted, Not Promoted)
zerocloudpdf.com also operates a separate, unlisted account-based storage feature (not linked from any page). It uses Firebase Authentication and Google Cloud Storage with Google-managed encryption at rest — not zero-knowledge/client-side encryption. It is unrelated to the PDF tools this policy otherwise describes, and no PDF tool ever contacts it.

**CDN loading note:** Our processing libraries (pdf.js, jsPDF, mammoth.js, qpdf.js) load from **cdnjs.cloudflare.com** and/or **cdn.jsdelivr.net**. These CDNs may log IP addresses and request timestamps per their own policies. We do not control or receive those logs.

---

## 3. Data Retention & Deletion

### 3.1 No Data Retention
- **ZeroCloudPDF does not store, retain, or have access to any files you process.**
- All file processing occurs entirely within your browser's memory.
- Files are discarded when you close the browser tab or refresh the page.
- No temporary copies are sent to any server.

### 3.2 Browser Cache
Your browser may cache CDN-loaded libraries (pdf.js, jsPDF, etc.) for offline capability. This cache contains only the library code, not your documents. Clear your browser cache to remove these cached files.

---

## 4. Legal Basis & Compliance

### 4.1 GDPR (EU/EEA Visitors)
For visitors from the European Union:
- **Lawful basis:** Consent for Google Analytics 4 and Google Fonts usage.
- **Right to erasure:** Not applicable—no personal data is collected or stored by ZeroCloudPDF.
- **International transfer:** Google Analytics and Google Fonts involve data transfers outside the EEA via Google's infrastructure. Google's Standard Contractual Clauses apply per their terms of service.

---

## 5. Third-Party Services

| Service | Provider | Purpose | Data Shared |
|---|---|---|---|
| Google Analytics 4 | Google | Anonymous usage statistics | Page views, browser type, session data, IP address |
| Google Fonts | Google | Typography | IP address, user agent (CDN logs) |
| CDN (cdnjs.cloudflare.com) | Cloudflare | Library delivery | IP address, user agent (CDN logs) |
| CDN (cdn.jsdelivr.net) | jsDelivr | Library delivery | IP address, user agent (CDN logs) |

We have **no Data Processing Agreements** beyond the standard terms of service provided by these vendors.

---

## 6. Cookies & Local Storage

| Technology | Purpose | Duration |
|---|---|---|
| Google Analytics 4 (_ga, _gid, etc.) | Anonymous usage tracking | 1–2 years (Google-managed) |
| Browser cache | Offline capability for CDN libraries | Until cleared by user |

We do **not** use `localStorage` or `sessionStorage` for tracking purposes.

---

## 7. Children's Privacy

ZeroCloudPDF is not directed at individuals under 18. We do not knowingly collect data from minors. Since our tools process files entirely in the browser without server contact, there is no mechanism for us to collect data from any user regardless of age.

---

## 8. Changes to This Policy

We will update this policy if:
- We add or remove analytics providers.
- We change our CDN sources.
- We introduce any feature that involves server contact (currently not planned).

**Notification:** Material changes will be announced via GitHub Discussions (Announcements category) and updated on this page with a new `Last updated` date.

---

## 9. Contact

**Privacy inquiries:** privacy@zerocloudpdf.com  
**Security issues:** security@zerocloudpdf.com  
**GitHub Discussions:** https://github.com/ZeroCloudPDF/ZeroCloudPDF/discussions/categories/privacy-compliance

---

## 10. Transparency Checklist

Before uploading sensitive documents (bank statements, passports, medical records, green cards, school certificates), verify:

- [ ] I understand all conversion and editing tools process files **only in my browser**.
- [ ] I understand **zero files are uploaded** to any server.
- [ ] I have read [SECURITY.md](SECURITY.md).
- [ ] I have read [ADR-001: Why Browser-Native, Not WebAssembly](docs/adr/001-why-browser-native-not-wasm.md).
- [ ] I have read [ADR-002: Client-Side Only Architecture](docs/adr/002-client-side-only-architecture.md).

---

*This policy is a living document. If you believe any statement is inaccurate or misleading, please open a GitHub Discussion in the "Privacy & Compliance" category.*
