# Privacy Policy — ZeroCloudPDF

**Last updated:** 2026-05-28  
**Effective date:** 2026-05-28  
**Jurisdiction:** India (compliance with Digital Personal Data Protection Act, 2023)

---

## 1. Privacy Model: Zero Server Contact by Default

ZeroCloudPDF is built on a **privacy-first, browser-native** architecture.

| Activity | Data Leaves Your Device? | What We Collect |
|---|---|---|
| PDF conversion (all tools) | **No** | Nothing. Zero data. |
| Word-to-PDF (text mode) | **No** | Nothing. |
| Word-to-PDF (image mode) | **No** | Nothing. html2canvas runs locally. |
| HEIC-to-PDF | **No** | Nothing. |
| JPG/PNG-to-PDF | **No** | Nothing. |
| Vault upload | **Yes** | See Section 3. |
| Vault download | **Yes** | See Section 3. |
| Thumbnail generation | **Yes** | File is sent to Cloud Run for processing. |

**The default experience is zero-knowledge.** If you never log into the Vault, we have no way to know you visited, what files you converted, or what was in them.

---

## 2. What We Do Collect

### 2.1 Anonymous Usage Statistics
We use **Google Analytics 4 (GA4)** to collect anonymous usage statistics such as pages visited, browser type, session duration, and general location. GA4 may set analytics cookies. It does **not** receive any information about the files you process or the content of your conversions. See [Google's privacy policy](https://policies.google.com/privacy) for details on how GA4 data is handled.

### 2.2 What We Do NOT Collect
We do not use:

- ❌ Mixpanel or any additional visitor tracking scripts beyond GA4
- ❌ Fingerprinting or device identification
- ❌ Server-side logs of conversion activity
- ❌ Error telemetry from conversion tools
- ❌ Advertising cookies or retargeting pixels
- ❌ Third-party analytics beyond GA4

**CDN loading note:** Our processing libraries (pdf.js, jsPDF, mammoth.js, html2canvas) load from **cdnjs.cloudflare.com**. Firebase Auth loads from **gstatic.com**. Google Fonts load from **fonts.googleapis.com** and **fonts.gstatic.com**. These CDNs may log IP addresses and request timestamps per their own policies. We do not control or receive those logs.

---

## 3. Vault Data Collection

The Vault is the **only** feature that requires data collection.

### 3.1 Account Data
When you create an account, we collect:

| Data Point | Purpose | Stored Where |
|---|---|---|
| Email address | Authentication, account recovery, verification | Firebase Auth |
| Firebase UID | Internal user identification | Firebase Auth + GCS object paths |
| Authentication provider | Email/password or Google OAuth | Firebase Auth |

We do **not** collect: name, phone number, physical address, government ID, or payment information.

### 3.2 File Data
When you upload to the Vault:

| Data Point | Purpose | Stored Where |
|---|---|---|
| Original file content | Storage and retrieval | Google Cloud Storage (GCS) |
| Generated PDF content | Storage and retrieval | GCS |
| Thumbnail/preview images | Vault UI display | GCS |
| File metadata (name, size, content-type, upload timestamp) | Vault organization | GCS object metadata |

**Storage path:** `User/{firebase-uid}/original/{timestamp}-{filename}`

### 3.3 Data-at-Rest Transparency
| Question | Answer |
|---|---|
| Are Vault files encrypted with a key only I control? | **No.** |
| Are Vault files encrypted at all? | GCS applies Google-managed server-side encryption by default. We do **not** add client-side encryption or additional server-side encryption layers. |
| Can ZeroCloudPDF staff read my files? | Technically, GCS administrators with sufficient privileges could access raw objects. We do not have internal policies or technical controls that prevent this. |
| Are files shared with third parties? | No. We do not sell, rent, or share file data. |

---

## 4. Data Retention & Deletion

### 4.1 Retention Period
- **A 25 MB free storage quota is enforced per account.** Uploads that would exceed this limit are rejected. Your current usage is visible in the Vault UI.
- **No automatic deletion.** Files persist indefinitely until you delete them.
- **No lifecycle rules** are configured on our GCS buckets.

### 4.2 How to Delete
1. Log into the Vault.
2. Select the file(s) you want to remove.
3. Click **Delete**.

This removes the GCS object permanently. Thumbnails and previews associated with that file are also deleted.

### 4.3 Known Risk: Orphaned Data
If you delete your Firebase account **without first deleting Vault files**, the GCS objects remain as **permanent orphans**. There is currently no automated cleanup job linking Firebase account deletion to GCS object removal.

**Recommendation:** Always empty your Vault before deleting your account.

---

## 5. Legal Basis & Compliance

### 5.1 India — DPDP Act 2023
As an Indian-founded service, we acknowledge obligations under the **Digital Personal Data Protection Act, 2023**:

- **Consent:** Vault usage constitutes explicit consent for processing and storage of file data.
- **Purpose limitation:** File data is used only for storage, retrieval, and thumbnail generation.
- **Data principal rights:** You may request deletion of your account data by emailing privacy@zerocloudpdf.com.
- **Grievance officer:** See Section 10. **Must be populated with a real name and Indian address before this policy is legally enforceable under DPDP Act 2023.**

### 5.2 GDPR (EU/EEA Visitors)
For visitors from the European Union:
- **Lawful basis:** Consent (Vault) and legitimate interest (security, fraud prevention).
- **Right to erasure:** You may request deletion of all personal data.
- **Right to access:** You may request a copy of data we hold about you.
- **International transfer:** File data is stored in Google Cloud (region: `asia-south1` and multi-region US). This constitutes a transfer outside the EEA. Google Cloud's standard terms include Standard Contractual Clauses for international transfers. **We have not executed a separate Data Processing Agreement with Google Cloud.**

---

## 6. Third-Party Services

| Service | Provider | Purpose | Data Shared |
|---|---|---|---|
| Google Analytics 4 | Google | Anonymous usage statistics | Page views, browser type, session data, IP address |
| Firebase Authentication | Google | User login, session management | Email, UID, auth provider |
| Google Cloud Storage | Google | File storage | File content, metadata, UID-derived paths |
| Cloud Run (get-upload-url) | Google | Signed URL generation | Firebase ID token (validated, not stored) |
| Cloud Run (generate-thumbnail) | Google | Thumbnail generation | Full file content (temporarily) |
| CDN (cdnjs.cloudflare.com) | Cloudflare | Library delivery | IP address, user agent (CDN logs) |
| CDN (gstatic.com) | Google | Firebase Auth library delivery | IP address, user agent (CDN logs) |
| Google Fonts | Google | Typography | IP address, user agent (CDN logs) |

We have **no Data Processing Agreements** beyond the standard Google Cloud Terms of Service. We are not a Google Cloud enterprise customer.

---

## 7. Cookies & Local Storage

| Technology | Purpose | Duration |
|---|---|---|
| Google Analytics 4 (_ga, _gid, etc.) | Anonymous usage tracking | 1–2 years (Google-managed) |
| Firebase Auth session cookie | Maintains login state | Session only (`SESSION` persistence) |
| `localStorage` / `sessionStorage` | Not used for tracking | — |
| Browser cache | Offline capability for CDN libraries | Until cleared by user |

---

## 8. Children's Privacy

ZeroCloudPDF is not directed at individuals under 18. We do not knowingly collect data from minors. If you believe a minor has provided personal data, contact us for deletion.

---

## 9. Changes to This Policy

We will update this policy if:
- We add server-side conversion features (currently not planned).
- We change encryption practices.
- We add or remove analytics.

**Notification:** Material changes will be announced via GitHub Discussions (Announcements category) and updated on this page with a new `Last updated` date.

---

## 10. Contact & Grievance Officer

**Privacy inquiries:** privacy@zerocloudpdf.com  
**Security issues:** security@zerocloudpdf.com  
**GitHub Discussions:** https://github.com/ZeroCloudPDF/ZeroCloudPDF/discussions/categories/privacy-compliance

**Grievance Officer (India):**  
⚠️ **REQUIRED FOR DPDP ACT 2023 COMPLIANCE — MUST BE POPULATED BEFORE PUBLICATION**  
Name: [Founder name to be added]  
Address: [Indian address to be added]  
Email: privacy@zerocloudpdf.com

---

## 11. Transparency Checklist

Before uploading sensitive documents (bank statements, passports, medical records, green cards, school certificates), verify:

- [ ] I understand conversion tools process files **only in my browser**.
- [ ] I understand the Vault stores files **unencrypted** with Google-managed keys.
- [ ] I have read [SECURITY.md](SECURITY.md).
- [ ] I have read [ADR-001: Why Browser-Native, Not WebAssembly](docs/adr/001-why-browser-native-not-wasm.md).

---

*This policy is a living document. If you believe any statement is inaccurate or misleading, please open a GitHub Discussion in the "Privacy & Compliance" category.*
