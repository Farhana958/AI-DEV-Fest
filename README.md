# TenderPack — Tender Document Package Builder

> **AI DevFest 2026 Solo Vibe-Coding Contest**  
> **Repository:** [devfest-solo-tenderpack](https://github.com/)  
> **Participant Name:** [Farhana Meherin Rimu]  
> **Registration Number:** [242-15-958]  
> **Public HTTPS Live Link:** [https://tenderpack.vercel.app / https://[username].github.io/devfest-submission/](https://tender-pack-document-builder--farhana958.replit.app)

---

## 1. Overview & Purpose

**TenderPack** is a 100% frontend-only enterprise web application designed for non-technical procurement and office staff to compile, validate, order, and generate professional tender document packages directly inside the browser.

When an organization participates in formal bidding, small oversights (such as missing mandatory documents, expired licenses, duplicate file attachments, or out-of-order papers) result in automatic bid rejection. **TenderPack** automates the entire audit and compilation process with strict deterministic validation.

---

## 2. Core Architecture & Privacy Guarantee

- **100% Client-Side Processing:** All document operations, SHA-256 binary cryptographic hashing, PDF parsing, page extraction, scaling, and merging occur purely within the user's browser.
- **Zero Backend / Zero Storage:** Documents are never uploaded to any remote server, database, cloud bucket, or external API.
- **No Dependencies on Cloud Services:** The entire pipeline works completely offline once loaded.

---

## 3. Main Features Completed (P0 Specification)

1. **Flexible Tender Specification Loading:**
   - Supports arbitrary, unseen `requirements.json` formats.
   - Graceful JSON error handling with user-friendly alerts.
   - Displays tender metadata (Tender ID, Tender Title, Procuring Entity, Bidder Name, Submission Deadline).
   - Dynamic deadline status pill (Active, Today, Past).
   - Sorts requirements strictly by numeric `order` before display and packaging.

2. **PDF Upload & Validation:**
   - Multi-file drag-and-drop and native file picker.
   - Non-PDF file rejection with clear error messages.
   - Strict limit guards: enforces maximum 30 PDF files and maximum 50 MB total upload size.
   - Corrupted and password-protected PDF safety: safely identifies encrypted/malformed files and prevents them from corrupting the package.
   - Calculates exact page counts for every valid document.

3. **Cryptographic Content Duplicate Detection:**
   - Uses browser **Web Crypto API SHA-256** binary hashing (`crypto.subtle.digest`).
   - Compares actual file bytes rather than filenames (e.g. `VAT.pdf` and `VAT-copy.pdf` with identical content are detected as duplicates).
   - Clear visual duplicate alerts identifying matching files.
   - **Strict Constraint Enforcement:** Duplicate files cannot be assigned to different requirements.

4. **Strict One-to-One Document Matching:**
   - Each requirement gets at most one PDF file.
   - Each uploaded PDF goes to at most one requirement.
   - Files assigned to other requirements (or sharing a duplicate hash with an assigned file) are disabled in dropdown selectors.
   - One-click "Unmatch" action to clear or reassign documents.

5. **Exact Deterministic Status Rules (Section 5 & 6):**
   - **Missing (🔴 Blocker):** Mandatory requirement with no matched file.
   - **Expiry date needed (🟠 Blocker):** Requirement with `has_expiry: true` matched with a file, but no expiry date entered.
   - **Expired (🔴 Blocker):** Requirement where `expiryDate < submissionDeadline`.
   - **Not provided (⚪ Non-blocking):** Optional requirement with no file matched (safely skipped during package generation).
   - **OK (🟢 Non-blocking):** File matched, and (if `has_expiry`) expiry date is on or after the deadline.
   - **Edge Case Verified:** When `expiryDate === submissionDeadline`, status is strictly **OK**.

6. **Blocking Logic & Generation Gate:**
   - The **Generate Final Package PDF** button is disabled whenever any blocker exists.
   - A dedicated **Blocking Issues Panel** itemizes all blocking problems with clear explanations and a 1-click **"Fix issue"** jump link that scrolls and highlights the row.

7. **Standardized PDF Package Compilation (Section 6 & 13–16):**
   - Output filename strictly formatted as `<tender_id>_Package.pdf` (e.g. `T-2026-0417_Package.pdf`).
   - **Page 1: Formal English Cover Page** (regardless of UI language):
     - Tender ID, Title, Procuring Entity, Bidder, Deadline, Dynamic Package Created Date.
     - Formatted table of included documents in verified numerical order with start page numbers.
   - **Original Page Order & Multi-page Preservation:**
     - Imports all pages of matched documents in strict requirement order.
     - Skips optional unmatched documents; excludes unassigned files.
   - **Non-Obscuring Dedicated Footer Engine:**
     - Renders `<tender_id> | Page X of Y` across **every page**, including cover and index.
     - `Y` represents the exact total package page count, calculated before rendering.
     - Dedicated 38 pt footer margin with hairline separator; source document pages are scaled proportionally above the footer to guarantee original content is never obscured.

8. **Complete Bilingual Interface (English & বাংলা):**
   - Instant language switcher in header.
   - All navigation, headings, table columns, badges, buttons, empty states, and blocker messages switch seamlessly between English and বাংলা.
   - Requirement titles dynamically resolve from `title_en` and `title_bn`.
   - Cover page remains in formal English as per specification.

---

## 4. High-Value Bonus Features Completed

- **Intelligent Filename Auto-Match Suggestions (Section 11):**
  - Token-similarity algorithm matches file names (e.g. `trade_license_2026.pdf` ➔ Trade License).
  - Displays interactive suggestion banner with confidence scores, `[Accept]`, `[Ignore]`, and `[Accept All Suggestions]`. Never forces or silently auto-assigns.
- **Table of Contents / Index Page (Section 18):**
  - Optional index page placed immediately after the cover page showing each document title, dotted leader lines, and exact starting page numbers (`Page X`).
- **CSV Checklist Export (Section 19):**
  - Generates RFC 4180 CSV export with UTF-8 BOM for Microsoft Excel / Google Sheets compatibility.
  - Columns: Order, Document Name, File Name, Pages, Expiry Date, Status.
- **Official Seal / Signature Stamp Tool (Section 21):**
  - In-browser PNG seal/signature upload with customizable target pages (All Pages, Last Page, Cover Page) and width adjustment.
- **In-Browser PDF Previewer:**
  - Full-screen modal to preview source PDFs and the generated package before or after downloading.
- **One-Click Quick Test Demo Generator:**
  - Programmatically generates 6 sample PDFs (including multi-page PDFs and an intentional content duplicate) to test the whole workflow in 5 seconds.

---

## 5. Tests Performed & Verification Results

Automated test suites (`tests/verify-all.ts` and `tests/browser-test.ts`) were executed to rigorously test the full contest matrix:

| Test Case | Description | Result |
|-----------|-------------|--------|
| **TEST 1** | Mandatory requirement with no file ➔ Missing (Blocked) | ✅ Passed |
| **TEST 2** | Optional requirement with no file ➔ Not provided (Not Blocked) | ✅ Passed |
| **TEST 3** | Expiry required matched file without date ➔ Expiry date needed (Blocked) | ✅ Passed |
| **TEST 4** | Expiry date before submission deadline (2026-10-19 < 2026-10-20) ➔ Expired (Blocked) | ✅ Passed |
| **TEST 5** | Expiry date strictly equal to deadline (2026-10-20 == 2026-10-20) ➔ OK (Not Blocked) | ✅ Passed |
| **TEST 6** | Expiry date after deadline (2026-12-31 > 2026-10-20) ➔ OK | ✅ Passed |
| **TEST 7** | Two identical PDFs with different names ➔ SHA-256 duplicate detected | ✅ Passed |
| **TEST 8** | Duplicate PDFs attempted for different requirements ➔ Prohibited & disabled | ✅ Passed |
| **TEST 9** | Single file attempted for multiple requirements ➔ Strict 1-to-1 enforced | ✅ Passed |
| **TEST 10** | Non-PDF file upload ➔ Rejected with alert | ✅ Passed |
| **TEST 11** | File limit > 30 PDFs ➔ Rejected with limit warning | ✅ Passed |
| **TEST 12** | File size > 50 MB ➔ Rejected with size warning | ✅ Passed |
| **TEST 13** | Optional matched PDF ➔ Included in output | ✅ Passed |
| **TEST 14** | Optional unmatched PDF ➔ Safely skipped | ✅ Passed |
| **TEST 15** | Random requirement order in JSON ➔ Sorted strictly by numeric order | ✅ Passed |
| **TEST 16** | Multi-page source PDF ➔ All pages included in original order | ✅ Passed |
| **TEST 17** | Final package footer ➔ `<tender_id> \| Page X of Y` on every page | ✅ Passed |
| **TEST 18** | Cover page ➔ Page 1, English, all required metadata present | ✅ Passed |
| **TEST 19** | Generated package filename ➔ `<tender_id>_Package.pdf` | ✅ Passed |
| **TEST 20** | Bilingual UI toggle ➔ All labels, tables, and statuses switch accurately | ✅ Passed |
| **TEST 21** | Removing matched file ➔ Status recalculates instantly | ✅ Passed |
| **TEST 22** | Changing match ➔ Prior assignment cleared cleanly | ✅ Passed |
| **TEST 23** | Corrupted / password-protected PDF ➔ Caught cleanly without crashing | ✅ Passed |

---

## 6. How to Run Locally

### Prerequisites
- Node.js (v18+ recommended; tested on v24)
- npm (v9+ recommended; tested on v11)

### Steps
```bash
# 1. Install dependencies
npm install

# 2. Run automated test suite & generate output package
npx tsx tests/verify-all.ts

# 3. Start local development server
npm run dev
```
Open [http://localhost:5173/](http://localhost:5173/) in Google Chrome.

---

## 7. How to Build for Production

```bash
npm run build
```
The optimized static production bundle is generated inside `dist/`.

---

## 8. Deployment Readiness

The application is completely static (HTML, CSS, JS, WebAssembly/PDF engines) and can be deployed with zero backend configuration:
- **Vercel:** Run `vercel --prod` or link the repository.
- **Netlify:** Run `netlify deploy --prod --dir=dist`.
- **GitHub Pages:** Deploy the `dist/` folder directly.

---

## 9. AI Tools Used & Most Useful Prompt

- **AI Tools Used:** Gemini 3.8 Flash (High) via Antigravity Agentic IDE.
- **Most Useful Prompt:**  
  *"Implement strict deterministic status calculation where expiryDate strictly less than submissionDeadline is Expired, same-day expiry is OK, duplicate detection uses Web Crypto SHA-256 binary hash preventing duplicate assignments, and output PDF pages scale proportionally above a dedicated 38pt footer margin so source content is never obscured."*

---

## 10. License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.
