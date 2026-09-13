# ZeroCloudPDF

&gt; Privacy-first PDF tools that run entirely in your browser.  
&gt; Zero uploads, zero servers for every conversion or edit.

[![Website](https://img.shields.io/badge/Website-zerocloudpdf.com-00b894?style=flat-square)](https://zerocloudpdf.com)
[![German](https://img.shields.io/badge/Article-Deutsch-brightgreen?style=flat-square)](https://zerocloudpdf.blogspot.com/2026/05/zerocloudpdf-vs-smallpdf-ilovepdf-pdf24.html)
[![Hashnode](https://img.shields.io/badge/Article-Hashnode-2962ff?style=flat-square)](https://zerocloudpdf.hashnode.dev/i-analyzed-the-privacy-architecture-of-popular-pdf-tools-here-s-why-i-built-a-browser-first-alternative)

---

## The Problem

Every mainstream PDF converter—Smallpdf, iLovePDF, Adobe Acrobat Online, PDF24, Online2PDF—requires you to upload your file to a remote server. The moment you click "Upload," your document begins a journey through DNS, CDN edges, load balancers, object storage, and shared processing containers. Each hop is a potential retention point you cannot audit.

**Privacy policies are mutable. Architecture is physics.**

---

## The Solution

ZeroCloudPDF processes all files **inside your browser** using standard JavaScript libraries:

| Library | Purpose |
|---------|---------|
| [pdf.js](https://github.com/mozilla/pdf.js) | PDF parsing, rendering, and structure analysis |
| [jsPDF](https://github.com/parallax/jsPDF) | PDF generation from images, text, and HTML |
| [mammoth.js](https://github.com/mwilliamson/mammoth.js) | Word document (.docx) to HTML/PDF conversion |
| [qpdf.js](https://github.com/jsejcksn/qpdf.js) | PDF encryption and decryption (Protect/Unlock) |
| Native Browser APIs | Image decoding (`Canvas`, `ImageBitmap`, `OffscreenCanvas`) |

Processing libraries load on-demand per tool via deferred script tags from trusted CDNs (jsDelivr and Cloudflare).

---

## Architecture

### The 5-Second Privacy Test

1. Open [zerocloudpdf.com](https://zerocloudpdf.com)
2. Enable Airplane Mode
3. Convert a file
4. It works

If it works offline, your file never touched `eth0`. Try that on any competitor.

---

## Comparison: Where Your Bytes Actually Go

| Tool | File Leaves Device? | Execution | Retention | Account Required? |
|------|---------------------|-----------|-----------|-----------------|
| Smallpdf | ✅ Yes | Remote cloud | Hours | Freemium gate |
| iLovePDF | ✅ Yes | Remote cloud | Hours | No |
| Adobe Acrobat Online | ✅ Yes | Multi-tenant cloud | Adobe ecosystem | Often |
| PDF24 (Web) | ✅ Yes | German server | Temporary | No |
| Online2PDF | ✅ Yes | German server | Temporary | No |
| **ZeroCloudPDF** | **❌ Never** | **Your browser** | **Instant discard** | **Never** |

*Applies to all PDF conversion and editing tools. A separate, unlisted legacy account feature exists outside this comparison's scope.*

German localization: [Deutschsprachiger Vergleich](https://zerocloudpdf.blogspot.com/2026/05/zerocloudpdf-vs-smallpdf-ilovepdf-pdf24.html)

Founder narrative: [Why I Had to Build My Own](https://zerocloudpdf.hashnode.dev/i-analyzed-the-privacy-architecture-of-popular-pdf-tools-here-s-why-i-built-a-browser-first-alternative)

---

## Performance: The Hidden Tax of Uploading

| Step | Server-Side Tool | Browser-Native (ZeroCloudPDF) |
|------|------------------|-------------------------------|
| Upload | 10–60+ seconds | **<< 1 second** |
| Queue | Variable | **None** |
| Processing | Shared vCPU | **Your local cores** |
| Download | 10–60+ seconds | **Instant** |
| **Total (50 MB PDF)** | **2–4 minutes** | **<< 30 seconds** |

---

## Current Tools (15 Total)

### Core Conversion Tools (10)
- **JPG to PDF** — Merge multiple images into a single PDF
- **PNG to PDF** — Convert PNG images to PDF
- **TIFF to PDF** — Convert TIFF images to PDF
- **WEBP/Images to PDF** — Convert WEBP and other image formats via generic image-to-PDF tool
- **PDF to JPG** — Extract pages as high-quality images
- **PDF to PNG** — Extract pages as PNG images
- **Merge PDF** — Combine multiple PDFs client-side
- **Compress PDF** — Reduce file size with optimized settings
- **Word to PDF** — Convert .docx without Microsoft Office
- **HEIC to PDF** — Convert iPhone HEIC images (rarely supported elsewhere)

### PDF Editing & Security (4)
- **Protect/Unlock PDF** — Add or remove password protection with 256-bit AES encryption (qpdf.js WASM)
- **Delete PDF Pages** — Remove specific pages from a PDF
- **Sign PDF** — Add digital signatures to PDF documents
- **Redact PDF** — Permanently remove sensitive content from PDFs

### Utilities (1)
- **Add Page Numbers** — Insert page numbers at custom positions

---

## Why We Use WebAssembly for Protect/Unlock

We deliberately chose **vanilla JavaScript** for most tools, but use **qpdf.js (WebAssembly)** for Protect and Unlock PDF:

- **Cryptographic security:** 256-bit AES encryption requires battle-tested implementations
- **Industry standard:** qpdf is widely audited and used in production environments
- **Performance:** WASM provides near-native speed for encryption operations
- **Same privacy guarantee:** Your file still never leaves your device

All other tools run on pure JavaScript for maximum auditability and debugging simplicity.

---

## Contributing

This repository currently serves as the public documentation and architecture hub for ZeroCloudPDF. If you are interested in the client-side PDF processing space, open an issue to discuss:

- Additional format support
- Performance benchmarks
- Privacy audit methodologies

See our [Architecture Decision Records (ADRs)](docs/adr/) for technical rationale behind key design choices.

---

## License

MIT

---

*ZeroCloudPDF is a privacy-first project built in India. No venture capital. No surveillance business model. Just architecture that makes privacy the default.*
