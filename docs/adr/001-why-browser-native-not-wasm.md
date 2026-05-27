# ADR-001: Browser-Native JavaScript over WebAssembly for ZeroCloudPDF

**Status:** Accepted  
**Date:** 2026-05-28  
**Deciders:** ZeroCloudPDF maintainers  
**Context:** Architecture choice for client-side PDF processing engine

---

## 1. Context and Problem Statement

ZeroCloudPDF processes PDFs, images, and Word documents entirely inside the user's browser. The core constraint is simple: **the file must never leave the device**. This is not a privacy policy claim — it is an architectural invariant.

When choosing the execution environment for parsing and generating PDFs, we evaluated two client-side options:

1. **WebAssembly (WASM)** — compiled binaries (e.g., `pdfium`, `Poppler`, `qpdf`) running inside the browser's WASM sandbox.
2. **Native JavaScript** — standard JS libraries (`pdf.js`, `jsPDF`, `mammoth.js`) executed by the browser's V8/SpiderMonkey engine.

Both options keep files local. Both satisfy the "zero server contact" requirement. The question was which one produces a more **auditable, maintainable, and trustworthy** system.

## 2. Decision Drivers

| Driver | Weight | Rationale |
|--------|--------|-----------|
| **Auditability** | Critical | Users must be able to verify our privacy claim in under 30 seconds |
| **Debuggability** | High | We need to trace exactly what happens to every byte |
| **Compatibility** | High | Must work in every modern browser without configuration |
| **Performance** | Medium | Acceptable for documents up to 50 MB |
| **Bundle Size** | Medium | Initial load must stay under control |
| **Raw throughput** | Low | Not a benchmark war; privacy is the differentiator |

## 3. Considered Options

### Option A: WebAssembly (WASM) with C++ PDF libraries

**Approach:** Compile mature C/C++ PDF libraries (e.g., `pdfium` via `emscripten`) to `.wasm` modules. Ship the binary alongside a thin JS glue layer.

**Pros:**
- Near-native execution speed for complex rendering pipelines.
- Reuses decades of battle-tested C++ PDF code.
- Smaller JS bundle; heavy logic lives in the compiled binary.

**Cons:**
- **Opacity:** `.wasm` binaries are not human-readable. A user cannot open DevTools and verify that the module is not making `fetch()` calls or accessing `WebSocket`. Source maps for WASM are partial and require a separate debug build.
- **MIME type and CSP friction:** Some corporate proxies and strict CSP policies block `.wasm` downloads or require explicit `wasm-unsafe-eval` directives.
- **Memory model complexity:** WASM uses a linear memory heap. Large PDFs (50–100 MB) require careful heap sizing (`--initial-memory`, `--max-memory`) or the module crashes with an out-of-bounds trap.
- **Maintenance burden:** Every upstream C++ fix requires a full recompilation pipeline (`emscripten` toolchain, Docker images, ABI compatibility checks).

### Option B: Native JavaScript with audited npm libraries

**Approach:** Use well-known JavaScript libraries (`pdf.js` by Mozilla, `jsPDF`, `mammoth.js`) loaded directly from the same origin or bundled at build time.

**Pros:**
- **Full auditability:** Open DevTools → Sources → the entire parsing and generation logic is plain text. Any developer can set a breakpoint on `XMLHttpRequest` or `fetch` and confirm zero network calls occur during conversion.
- **No compilation step:** Library updates are `npm install` and `git diff`. No cross-compilation toolchain.
- **Universal compatibility:** Works in Chrome, Firefox, Safari, Edge, and mobile browsers without `.wasm` MIME type configuration or CSP exceptions.
- **Native browser APIs:** `Canvas`, `ImageBitmap`, `OffscreenCanvas`, and `FileReader` are first-class citizens. No FFI boundary between JS and WASM.

**Cons:**
- Slower raw throughput for massive PDFs compared to optimized C++.
- Larger initial JS bundle if all libraries are loaded eagerly.
- Some advanced PDF features (certain font subsets, complex vector paths) are less mature in JS than in `pdfium`.

## 4. Decision

**Chosen option:** Option B — Native JavaScript.

The auditability and debuggability advantages of plain JS outweigh the raw performance benefits of WASM for our use case. ZeroCloudPDF's value proposition is **"you can verify this yourself in 30 seconds."** A `.wasm` binary breaks that promise — it asks users to trust a black box. Plain JavaScript makes the architecture **falsifiable**.

ZeroCloudPDF does not use WebAssembly in any processing path. All conversion logic is pure JavaScript.

## 5. Consequences

### Positive
- Any developer can inspect the exact code path their file takes.
- The **5-Second Privacy Test** (enable airplane mode, convert a file, watch it succeed) works without caveats — there is no hidden WASM module that might cache a network call.
- Corporate and government users behind strict proxies can use the tool without `.wasm` download failures.
- Faster iteration cycle: library bug fixes ship in hours, not days (no recompilation).

### Negative / Mitigations
- **Performance ceiling:** For documents above 50 MB, JS is slower than WASM would be. Mitigation: we optimize for the 90th percentile (documents under 50 MB) and use `OffscreenCanvas` + `requestIdleCallback` to keep the main thread responsive.
- **Bundle size:** Processing libraries (`pdf.js`, `jsPDF`, `mammoth.js`) are loaded via deferred `<script>` tags on every page load. `defer` ensures they do not block HTML parsing, but they do download with the initial page request. This is a trade-off: users pay the download cost once up front, but every tool is instantly ready without additional network round-trips after the first load.
- **Feature gaps:** For edge-case PDFs (certain embedded fonts, XFA forms), JS libraries may fail where C++ would succeed. We accept this trade-off and document known limitations transparently.

## 6. Validation

The decision is validated by the **5-Second Privacy Test**:

1. Open `zerocloudpdf.com`
2. Enable Airplane Mode (or disconnect Wi-Fi)
3. Upload and convert a file
4. The conversion completes successfully

If the tool works offline, no bytes traversed `eth0`. This test is impossible to pass with a server-side tool, and trivial to pass with our browser-native JS architecture.

## 7. Related Decisions (Future)

The following ADRs are planned but not yet written:

- **ADR-002:** Client-Side-Only Architecture (no service worker network fallback)
- **ADR-003:** Zero Server Contact Verification Methodology

## 8. Future Improvements

- **Migrate library loading from deferred `<script>` tags to dynamic `import()` per tool**, so `pdf.js`, `jsPDF`, and `mammoth.js` only download when the relevant tool tab is first used. This ADR should be updated when that change ships.

## 9. References

- [ZeroCloudPDF Website](https://zerocloudpdf.com)
- [Privacy Architecture Comparison](https://zerocloudpdf.wordpress.com/2026/05/24/zerocloudpdf-vs-smallpdf-ilovepdf-adobe-a-privacy-first-architecture-comparison/)
- [Mozilla pdf.js](https://github.com/mozilla/pdf.js)
- [jsPDF](https://github.com/parallax/jsPDF)
- [mammoth.js](https://github.com/mwilliamson/mammoth.js)
