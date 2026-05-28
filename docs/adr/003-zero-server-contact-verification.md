# ADR-003: Zero Server Contact Verification Methodology

**Status:** Accepted  
**Date:** 2026-05-28  
**Deciders:** ZeroCloudPDF maintainer  
**Context:** ADR-001 (Browser-Native Architecture), SECURITY.md, PRIVACY.md

---

## 1. Context

ZeroCloudPDF claims that **PDF conversions happen entirely inside the browser with zero server contact**. This is our core differentiator against server-based competitors (Smallpdf, iLovePDF, Adobe Acrobat Online).

However, "trust us" is not a valid security model. This ADR defines a **reproducible, manual verification methodology** that any user, auditor, or journalist can perform in under five minutes using standard browser DevTools.

---

## 2. Decision

We will document and maintain a public, step-by-step verification protocol that proves:

1. No file data leaves the device during conversion.
2. No WebAssembly module is loaded for processing.
3. No `fetch()`, `XMLHttpRequest`, or `WebSocket` transmits converted content.
4. All processing libraries are auditable, version-pinned, and loaded from known CDNs.

---

## 3. Verification Methodology

### 3.1 Environment Setup

| Requirement | Value |
|---|---|
| Browser | Chrome, Firefox, Safari, or Edge (latest stable) |
| DevTools | Network tab + Console tab |
| Test file | Any non-sensitive file (JPG, PNG, Word .docx, HEIC) |
| Network | Broadband (for initial CDN load); optional Airplane Mode |

**Pre-test:** Open DevTools → Network tab → click the **🚫 Clear** button. Ensure "Preserve log" is **unchecked** so you see only fresh requests.

---

### 3.2 Step 1: Baseline Page Load

Load `https://zerocloudpdf.com` (or the relevant tool page). Observe the Network tab.

**Expected requests:**
| Request | Purpose | Contains File Data? |
|---|---|---|
| `gtag/js?id=G-ESZCDHN3HT` | Google Analytics 4 page tracking | No |
| `firebase/8.10.0/firebase-app.js` | Firebase Auth library | No |
| `firebase/8.10.0/firebase-auth.js` | Firebase Auth library | No |
| `jspdf/2.5.1/jspdf.umd.min.js` | PDF generation library | No |
| `pdf.js/3.11.174/pdf.min.js` | PDF rendering library | No |
| `mammoth/1.6.0/mammoth.browser.min.js` | Word-to-PDF library | No |
| `html2canvas/1.4.1/html2canvas.min.js` | **Only if Word Image mode is triggered** | No |

**What to verify:**
- `html2canvas` does **not** appear on initial page load. It is injected dynamically only when Word Image mode is selected.
- No request contains file data, file metadata, or conversion parameters.

---

### 3.3 Step 2: Isolate the Conversion

**Do not log into the Vault.** Stay on the public tool page.

Select a tool and upload a test file. Watch the Network tab during the entire conversion.

**What you should see:**
- **Zero new network requests** containing your file.
- The only new requests may be:
  - GA4 `collect` pings (anonymous page metrics, no file content)
  - Firebase Auth heartbeat (only if you are logged in)

**What to look for (red flags):**
| Red Flag | Meaning |
|---|---|
| `POST` or `PUT` request with `Content-Type: multipart/form-data` | File uploaded to server |
| `fetch()` to non-CDN domain with payload &gt; 1 KB | File data transmitted |
| WebSocket connection opened | Real-time data streaming |
| WebAssembly `.wasm` file downloaded | WASM processing path active |

**ZeroCloudPDF exhibits none of these during conversion.**

---

### 3.4 Step 3: Payload Inspection

If you see any request you are unsure about:

1. Click the request in the Network tab.
2. Go to the **Payload** or **Request** sub-tab.
3. Verify the payload size is &lt; 1 KB (typical for analytics pings).
4. Verify the payload does not contain Base64-encoded image data, binary blobs, or file names.

**All conversion output is written to a local `Blob` or `URL.createObjectURL()` and triggered via `anchor.download`.** No network transmission occurs.

---

### 3.5 Step 4: Airplane Mode Test

This test verifies that conversion logic is **truly local** and does not depend on a server handshake.

**Prerequisite:** Complete Step 1 (page load) while online so CDN libraries cache in the browser.

1. Keep the tab open.
2. Disconnect your machine from the internet (Wi-Fi off / Airplane Mode).
3. Clear the Network tab log.
4. Upload a new file and convert.
5. Download the resulting PDF.

**Expected result:** Conversion completes successfully. The Network tab shows **zero requests** (or only failed GA4 pings). The PDF downloads normally.

**Known caveat:** If browser cache was cleared between the online page load and the offline test, libraries will not be available and conversion will fail with a "Library not ready" toast. This is expected — we do not use a Service Worker or engineered offline strategy. Standard browser HTTP cache is the only offline mechanism.

---

### 3.6 Step 5: Code Audit Points

For developers who want to verify at the source level, inspect these functions in `app.js`:

| Function | What to Verify |
|---|---|
| `wordToPdf()` | Uses `mammoth.convertToHtml()` on a local `ArrayBuffer`. No `fetch()`. |
| `wordToPdfImage()` | Injects `html2canvas` dynamically, then renders a local DOM clone. No `fetch()`. |
| `mergePdfs()` | Uses `pdfjsLib.getDocument()` + `jsPDF`. No `fetch()`. |
| `pdfToImages()` | Uses `pdfjsLib.getDocument()` + `&lt;canvas&gt;`. No `fetch()`. |
| `finishPdf()` | Creates `URL.createObjectURL(blob)`. Triggers download via `anchor.click()` (Safari), `window.open()` (Chrome/Firefox), or a persistent download button (iOS). No `fetch()`. |
| `initPdfTool()` | Dispatches all image-to-PDF, compress, merge, Word, and PDF-to-image conversions based on the active ribbon tab or current pathname. No `fetch()`. |

**Global check:** Search `app.js` for `fetch(`, `XMLHttpRequest`, `WebSocket(`, `.wasm`. The only `fetch()` calls should be in Vault-related functions (`uploadSingle`, `loadGallery`, and the download confirm handler, including a direct `fetch()` to signed GCS URLs when downloading Vault files).

---

## 4. Library Loading Verification

### 4.1 Static Script Tags (Deferred)
The following libraries load via `&lt;script defer&gt;` on every page load:

```html
&lt;script defer src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"&gt;&lt;/script&gt;
&lt;script defer src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"&gt;&lt;/script&gt;
&lt;script defer src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.6.0/mammoth.browser.min.js"&gt;&lt;/script&gt;
