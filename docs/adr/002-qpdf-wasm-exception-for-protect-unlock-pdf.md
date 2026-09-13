# ADR-002: QPDF Compiled to WebAssembly for Protect/Unlock PDF

**Status:** Accepted
**Date:** 2026-07-26
**Deciders:** ZeroCloudPDF maintainers
**Context:** Narrow exception to ADR-001's browser-native JavaScript default, scoped to a single tool

---

## 1. Context and Problem Statement

ADR-001 established browser-native JavaScript as the default execution model for ZeroCloudPDF, specifically to keep every processing path auditable in DevTools with no compiled binary in the loop. That default holds for every tool except one.

Protect/Unlock PDF needs to apply and remove real password-based encryption on a PDF, specifically 256-bit AES per the PDF specification. No mature, actively maintained pure-JavaScript library implements PDF password encryption and decryption to that standard. The candidates evaluated either handle PDF structure without encryption (`pdf-lib`, `pdf.js`) or only read encrypted PDFs rather than write them.

QPDF is a widely used, open-source C++ library that implements this correctly, including the specific AES-256 encryption dictionary format the PDF spec requires. The question was not "WASM vs JS" in general, ADR-001 already answered that, it was: is there a JS alternative for this one function, or does the general rule need a scoped, documented exception.

## 2. Decision Drivers

| Driver | Weight | Rationale |
| --- | --- | --- |
| **Correctness** | Critical | Password protection must produce a PDF that actually opens correctly in Acrobat and other readers, not a custom approximation |
| **No server round-trip** | Critical | The password is more sensitive than the file itself and must never leave the device, WASM-in-a-Web-Worker still satisfies this |
| **Auditability** | High | Even though the engine is compiled, the boundary around it must stay inspectable: what goes in, what comes out, when it's fetched |
| **Scope of exception** | High | The exception must be narrow, one tool, one function, not a precedent for using WASM generally |
| **On-demand loading** | Medium | The WASM payload should not affect load time for the 15 other tools that don't need it |

## 3. Considered Options

### Option A: Pure JavaScript password protection

**Approach:** Implement or adopt a pure-JS library to write PDF encryption dictionaries and apply RC4/AES-256 manually.

**Pros:**
- Consistent with ADR-001, no compiled binary anywhere on the site.
- Fully readable source, no separate audit boundary needed.

**Cons:**
- No actively maintained pure-JS library implements PDF encryption to the AES-256 standard the tool needed to ship.
- Hand-rolling PDF encryption is a correctness and security risk that is hard to justify for a small team maintaining 16 other tools.
- Would likely under-deliver on the "real encryption, not a gimmick" bar the rest of the product holds itself to.

### Option B: QPDF compiled to WebAssembly, loaded on demand

**Approach:** Use `@neslinesli93/qpdf-wasm`, a WebAssembly build of the real QPDF library, running inside a dedicated Web Worker, loaded from CDN only when a user opens the Protect/Unlock PDF tool and clicks confirm.

**Pros:**
- Real, spec-correct 256-bit AES encryption and decryption, the same engine many professional PDF tools rely on.
- Runs entirely on-device, inside a Web Worker, the password and file bytes never leave the browser tab.
- Loaded on demand only, the WASM payload has zero cost for any of the other 15 tools or the homepage.
- The boundary is still auditable: the worker only ever fetches the QPDF engine itself and never makes any other network call, which can be verified in DevTools' Network tab during use.

**Cons:**
- Reintroduces the exact opacity trade-off ADR-001 rejected: the compiled `.wasm` binary itself is not human-readable, a user cannot open DevTools and read the encryption logic line by line the way they can with `pdf.js` or `jsPDF`.
- First use of this specific tool requires a network fetch of the QPDF engine from CDN, this is different from the rest of the site, where the airplane-mode ("5-Second Privacy Test") claim holds from the very first page load.
- Adds a second execution model to the codebase, which the team must remember is the one intentional exception.

## 4. Decision

**Chosen option:** Option B — QPDF compiled to WebAssembly, scoped to this tool only.

The exception is accepted because the alternative, hand-built PDF encryption, would have been a worse outcome for the exact property ADR-001 is trying to protect: user trust. Shipping approximate or non-standard encryption under a "protect your PDF" label would be a bigger credibility risk than one clearly-scoped, clearly-documented WASM boundary.

This decision does not change the default. Every other tool remains pure JavaScript per ADR-001. WASM is used in exactly one place in the codebase: `frontend/qpdf-worker.js`, instantiated only from the Protect/Unlock PDF tool page.

## 5. Consequences

### Positive

- Protect/Unlock PDF ships real, standards-correct 256-bit AES encryption rather than a weaker or custom substitute.
- The password never leaves the device: it is used once, inside the Web Worker, to run QPDF's encrypt or decrypt operation, then discarded.
- On-demand loading means the other 15 tools' load time and bundle size are unaffected.
- The network boundary around the WASM module is narrow enough to still be verified manually: open DevTools, use the tool, confirm the only outbound request is the one-time engine fetch.

### Negative / Mitigations

- **Opacity of the compiled binary:** unlike every other tool, a user cannot read the actual encryption logic in plain text. Mitigation: the surrounding boundary, what is fetched, when, and what network calls happen during use, is kept fully inspectable and documented on the tool page itself, so the "verify it yourself" property is preserved at the boundary even though it can't extend inside the binary.
- **First-use network dependency:** the "works from the very first page load in airplane mode" claim does not hold for this tool the first time it's used, since the engine must be fetched once. Mitigation: this is stated plainly rather than implied otherwise; the tool's own copy should describe this as "works offline after the engine loads once," not an unqualified airplane-mode claim, to avoid overstating what ADR-001's validation test proved for the rest of the site.
- **Single point of exception to track:** as the codebase grows, this must not quietly become precedent for using WASM elsewhere. Mitigation: this ADR exists specifically so any future WASM proposal is compared against a documented, narrow bar, correctness need with no adequate JS alternative, not general performance preference.

## 6. Validation

The decision is validated by two checks, both narrower than ADR-001's general airplane-mode test:

1. **Network boundary check:** Open DevTools' Network tab, use Protect/Unlock PDF, confirm the only requests are the one-time fetch of `qpdf.js`/`qpdf.wasm` from CDN, no request ever contains the file bytes or the password.
2. **Correctness check:** Protect a PDF, confirm a general-purpose reader (not just ZeroCloudPDF's own detector) requires the password to open it; unlock a protected PDF with the correct password and confirm it opens without one; attempt a wrong password and confirm QPDF reports failure with no retry or guessing behavior.

## 7. Related Decisions

- **ADR-001:** Browser-Native JavaScript over WebAssembly for ZeroCloudPDF (the general default this ADR is a scoped exception to)

## 8. Future Improvements

- If a pure-JS library ever implements PDF AES-256 encryption/decryption to a standard equivalent to QPDF, re-evaluate this exception against ADR-001's original bar.
- Consider caching the QPDF WASM payload (e.g. via a service worker) so repeat visits to this specific tool don't require a re-fetch, this would narrow the first-use network dependency noted above.

## 9. References

- [ZeroCloudPDF Website](https://zerocloudpdf.com)
- [QPDF](https://github.com/qpdf/qpdf)
- [@neslinesli93/qpdf-wasm](https://www.npmjs.com/package/@neslinesli93/qpdf-wasm)
- ADR-001: Browser-Native JavaScript over WebAssembly for ZeroCloudPDF
