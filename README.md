# ZeroCloudPDF

&gt; Privacy-first PDF tools that run entirely in your browser.  
&gt; Zero uploads. Zero servers. Zero trust required.

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
| Native Browser APIs | Image decoding (`Canvas`, `ImageBitmap`, `OffscreenCanvas`) |

No WebAssembly. No serverless functions. No hidden WebSocket streams. Just libraries you can `npm install` and audit yourself.

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

## Current Tools

- **JPG to PDF** — Merge multiple images into a single PDF
- **PDF to JPG** — Extract pages as high-quality images
- **Merge PDF** — Combine multiple PDFs client-side
- **Compress PDF** — Reduce file size with iPhone-optimized settings
- **Word to PDF** — Convert .docx without Microsoft Office
- **HEIC to PDF** — Convert iPhone HEIC images (rarely supported elsewhere)

**Roadmap:** Rotate PDF, Edit PDF text

---

## Why Not WASM?

We deliberately chose **vanilla JavaScript** over WebAssembly:

- **Auditability:** Source maps and unminified libraries are inspectable in DevTools
- **No compilation step:** Faster iteration, simpler debugging
- **Universal compatibility:** Works in every modern browser without `.wasm` MIME type headaches
- **Same privacy guarantee:** Your file still never leaves your device

The privacy claim is equally strong—and now it is **falsifiable** by any developer in 30 seconds.

---

## Contributing

This repository currently serves as the public documentation and architecture hub for ZeroCloudPDF. If you are interested in the client-side PDF processing space, open an issue to discuss:

- Additional format support
- Performance benchmarks
- Privacy audit methodologies

---

## License

MIT

---

*ZeroCloudPDF is a privacy-first project built in India. No venture capital. No surveillance business model. Just architecture that makes privacy the default.*
