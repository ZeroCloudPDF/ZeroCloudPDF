# ADR-002: Client-Side-Only Architecture

**Status:** Accepted  
**Date:** 2026-05-28  
**Deciders:** ZeroCloudPDF maintainer  
**Context:** ADR-001 (Browser-Native Architecture), ADR-003 (Zero Server Contact Verification)

---

## 1. Context

Most modern web applications use Service Workers, Progressive Web App (PWA) manifests, and engineered offline strategies to improve reliability and user experience. ZeroCloudPDF explicitly rejects this stack.

This ADR documents why we remain a **pure client-side application** with no Service Worker, no PWA registration, and no engineered offline capability — and what that means for users.

---

## 2. Decision

We will **not** implement:

- A Service Worker (`sw.js`)
- A Web App Manifest (`manifest.json`)
- PWA registration or install prompts
- An engineered offline/network fallback strategy

Our only offline mechanism is the **standard browser HTTP cache** of CDN-loaded libraries. If the cache is warm, conversion tools work without internet. If the cache is cold, they do not.

---

## 3. What We Do NOT Use

### 3.1 No Service Worker
There is no `sw.js` file in the repository. There is no `navigator.serviceWorker.register()` call. There is no background sync, no push notification infrastructure, and no cache API usage.

**Why:** Service Workers introduce a persistent proxy layer between the user and the application. For a privacy-first tool, this creates ambiguity: a Service Worker could theoretically intercept and log file data, or be updated silently by the server to change behavior. Removing it entirely eliminates that trust surface.

### 3.2 No PWA Manifest
There is no `manifest.json`. There is no `apple-touch-icon` metadata for standalone installation. The site cannot be "installed" to a home screen as a standalone app.

**Why:** PWAs blur the line between web and native application. Users who install a PWA may assume it has native-level sandboxing or offline guarantees that our architecture does not provide. We prefer the transparency of a standard browser tab.

### 3.3 No Engineered Offline Strategy
We do not use:
- The Cache API
- IndexedDB for library storage
- Local bundling of processing libraries
- Network fallback patterns (`fetch` → cache → error)

**Why:** All of these require additional code complexity and storage permissions. Our threat model prioritizes **minimal code paths** and **zero storage access** for conversion tools. The fewer systems that touch user files, the fewer systems that can be compromised.

---

## 4. How Offline Actually Works

Our offline capability is a **side effect**, not a feature.

### 4.1 Standard Browser HTTP Cache
When a user first loads a tool page, the browser downloads these libraries and assets from CDN:

| Library / Asset | CDN Source | Cache Behavior |
|---|---|---|
| jsPDF 2.5.1 | cdnjs.cloudflare.com | Standard HTTP cache |
| pdf.js 3.11.174 | cdnjs.cloudflare.com | Standard HTTP cache |
| mammoth.js 1.6.0 | cdnjs.cloudflare.com | Standard HTTP cache |
| html2canvas 1.4.1 | cdnjs.cloudflare.com | Standard HTTP cache (only when Word Image mode triggers load) |
| Firebase 8.10.0 | gstatic.com | Standard HTTP cache |
| Google Fonts | fonts.googleapis.com / fonts.gstatic.com | Standard HTTP cache |

If the user returns to the site (or keeps the tab open) while offline, and the browser cache has not expired or been cleared, the libraries and assets are served from cache and conversion proceeds normally.

### 4.2 The Airplane Mode Test
This is documented in ADR-003. The test works **only if**:
1. The initial page load happened while online.
2. The browser cache has not been cleared since that load.
3. The user does not refresh the page while offline (a refresh may attempt to revalidate CDN resources).

If any of these conditions fail, conversion fails with a "Library not ready" toast. This is expected behavior.

---

## 5. Cache Behavior & Limitations

### 5.1 Cache Volatility
Standard HTTP cache is **user-controlled and fragile**:

| Scenario | Result |
|---|---|
| User clears browser cache | Libraries must re-download. Offline breaks. |
| CDN cache expires (Cache-Control header) | Libraries must re-download. Offline breaks. |
| Corporate firewall blocks cdnjs.cloudflare.com | Libraries never load. Conversion impossible. |
| User opens site in Incognito/Private mode | Cache discarded on close. Offline breaks. |

### 5.2 No Cache Guarantees
We do not set `Cache-Control` headers (we do not control the CDN). We do not use `localStorage` or `IndexedDB` to persist libraries. We do not prompt users to "download for offline use."

**The offline experience is best-effort and incidental.**

---

## 6. Trade-offs & Consequences

### Positive
- **Minimal attack surface:** No Service Worker means no persistent proxy that could be compromised.
- **No storage permissions:** Users are never asked to grant persistent storage access.
- **Transparent behavior:** Users understand they are on a website, not an installed app.
- **Simpler debugging:** No Service Worker lifecycle issues, no cache invalidation wars, no manifest update delays.

### Negative
- **Unreliable offline:** Works only if cache is warm. No guarantee.
- **CDN dependency:** If cdnjs.cloudflare.com is unreachable, the site is entirely non-functional. We do not bundle libraries locally as a fallback.
- **No background processing:** Cannot process files while the tab is closed or the device is locked.
- **No installability:** Cannot be added to home screen for quick access like a native app or PWA.

---

## 7. Related Documents

- [ADR-001: Why Browser-Native, Not WebAssembly](001-why-browser-native-not-wasm.md)
- [ADR-003: Zero Server Contact Verification Methodology](003-zero-server-contact-verification.md)
- [SECURITY.md](https://github.com/ZeroCloudPDF/ZeroCloudPDF/blob/main/SECURITY.md)
- [PRIVACY.md](https://github.com/ZeroCloudPDF/ZeroCloudPDF/blob/main/PRIVACY.md)

---

*This ADR is a living document. If we ever add a Service Worker, PWA manifest, or engineered offline strategy, this ADR must be superseded.*
