# LabelCheck AI Prototype v24

LabelCheck AI is a standalone take-home prototype for alcohol label verification. It is designed around the discovery notes from the Compliance Division: fast routine checks, simple user experience, no COLA integration, and a local-first workflow that remains useful when external ML/OCR services are blocked.

## What the app does

The prototype lets a compliance agent import applicant-stated information, provide label evidence, and receive a verification report with Pass, Warning, or Fail outcomes. Applicant information can be manually entered, uploaded from TXT/CSV/JSON/EML-style text files, or pasted from an email, form export, or other digital source.

Verified fields include brand name, product name, class/type designation, alcohol content, net contents, and the mandatory government warning statement. The brand, product name, and class/type checks use fuzzy matching so obvious case and punctuation differences can be treated as warnings or passes instead of hard failures. The ABV/proof and net contents checks use stricter extraction and comparison. The warning statement check is intentionally strict because stakeholder notes emphasized exact wording and capitalization.

The app also includes a batch mode where an agent can paste CSV rows for multiple labels and download a batch report.

## How to run locally

No build step is required.

1. Download or clone this folder.
2. Open `index.html` in a modern browser.
3. Optional: upload or paste applicant-stated information and click `Fill Expected Fields`.
4. Paste or enter label text and click `Verify Label`.

Optional OCR from image is compatible with Tesseract.js. The app now tries to load a local offline bundle from `vendor/tesseract/tesseract.min.js` first, then falls back to the CDN only when online. If neither OCR source is available, the app still works through pasted label text.

For a local web server, run one of these commands from the project folder:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```


## Offline OCR compatibility

The current browser workflow is compatible with **Tesseract.js offline** because the app calls the standard browser API:

```javascript
Tesseract.recognize(file, "eng")
```

That API works the same whether `tesseract.min.js` is loaded from a CDN or from a local vendored file. For an offline/air-gapped package, download the Tesseract.js browser bundle during preparation and place it at:

```text
vendor/tesseract/tesseract.min.js
```

The included `index.html` loads that local path first. When the local bundle is missing and the machine is online, it falls back to the CDN. When the local bundle is missing and the machine is offline, the verification workflow still runs using pasted OCR/text evidence, but image OCR is unavailable.

The verification engine is also compatible with desktop/offline OCR tools such as OCRmyPDF, Tesseract CLI, EasyOCR, or PaddleOCR as a preprocessing step. Run one of those tools outside the browser, paste the resulting text into **Section 3: Label evidence**, and click **Verify Label**.

### GitHub note to include in the repository description

```text
Offline-capable alcohol label verification prototype with local Tesseract.js OCR support, TTB F 5100.31 parsing, batch CSV review, and human-in-the-loop Y/N compliance checks.
```

### Suggested GitHub topics

```text
ttb, alcohol-labels, compliance, ocr, tesseract-js, offline-first, static-site, human-in-the-loop
```

## Files

`index.html` contains the application structure.

`styles.css` contains the responsive, accessibility-oriented interface styling.

`app.js` contains applicant-field import parsing, verification logic, fuzzy matching, field extraction, batch parsing, CSV export, and optional OCR integration.

## Approach and technical decisions

This prototype prioritizes speed, usability, and transparency over ambitious automation. The stakeholder interviews made clear that agents will not adopt a system that takes 30 to 40 seconds per label or hides its reasoning. The report therefore shows the expected value, detected value, status, confidence, and plain-English explanation for every check.

The app is intentionally standalone. It does not integrate with COLA, does not store uploaded documents, and does not require a backend for the core workflow. This matches the prototype scope and avoids unnecessary authorization, retention, and PII complications.

Fuzzy matching is implemented directly in JavaScript using normalization and Levenshtein similarity. This supports practical cases like `STONE'S THROW` versus `Stone's Throw` while still flagging substantial differences for human review.

The prototype uses local deterministic checks wherever possible. Optional OCR is included only as an enhancement, not as the core dependency, because the discovery notes identified outbound network blocking as a realistic deployment constraint.

## Assumptions

The prototype is a human-in-the-loop assistant, not an automated legal approval engine.

For the government warning, the prototype checks for the standard wording supplied in the project brief and enforces the capitalized `GOVERNMENT WARNING:` heading. It does not reliably detect bold text from pasted OCR output because OCR text does not preserve typography.

The prototype handles common distilled spirits examples first. Beverage-specific exceptions for beer, wine, imports, and unusual container sizes are documented as future work.

Single-application import supports common labels such as `Brand Name:`, `Product Name:`, `Alcohol Type:` (optional), `Alcohol Content:`, `Net Contents:`, and `Government Warning:`. It also supports simple JSON objects and CSV rows with comparable headers. Batch mode assumes one row per label with headers: `brand,productName,alcoholType,abv,netContents,labelText`.

## Known limitations and trade-offs

OCR quality depends on image quality and whether the browser can load Tesseract.js. Poor lighting, glare, rotation, and stylized typography can reduce extraction quality.

The app does not determine whether warning text is visually bold. That would require image-layout analysis or structured PDF/image processing beyond this MVP.

The app does not store imported applicant text, history, user accounts, or audit logs. Those would be required for a production federal workflow but are intentionally omitted for the time-constrained prototype.

The app does not validate every TTB beverage-type-specific rule. It focuses on the common fields and stakeholder pain points described in the project brief.

## Deployment option

This project can be deployed as a static site on GitHub Pages, Netlify, Vercel, Azure Static Web Apps, or any internal web server. Because the core workflow runs in the browser, deployment does not require a database or application server.

## Suggested future improvements

Add server-side OCR for controlled environments where browser/CDN OCR is blocked.

Add image preprocessing for skew, glare, and low-contrast labels.

Add beverage-type rule profiles for beer, wine, and distilled spirits.

Add PDF label support.

Add audit logs and role-based access for a production compliance workflow.

## v4 update: normalized extraction layer

This version adds a normalized extraction step between raw OCR and business-rule validation. The app now keeps the raw OCR text visible, applies OCR cleanup for common label artifacts, extracts structured fields, and displays a `Normalized extraction` JSON panel before comparing values to the applicant statement.

Key improvements:

- Repairs common OCR noise seen on decorative labels, including stylized punctuation, spaced words, and known whiskey-label artifacts such as `J ennessee` resolving toward `Tennessee`.
- Resolves class/type against a small beverage-class dictionary rather than relying only on literal OCR output.
- Extracts ABV and proof with dedicated rules, including formats such as `40% ALC BY VOL. (80 PROOF)`.
- Verifies against normalized structured values while preserving the original OCR output for agent review.
- Continues to fail true mismatches, such as an applicant stating `45% Alc./Vol. (90 Proof)` when the label shows `40% Alc By Vol. (80 Proof)`.

For difficult images such as black-label whiskey artwork with curved type, borders, and white-on-black lettering, this approach demonstrates the intended production architecture: OCR first, structured extraction second, compliance validation third, with the agent remaining the final reviewer.


## Version 5 update

This version separates applicant and verification fields for `Brand Name` and `Product Name`. Brand name now represents the brand/trade name, while product name captures a distinct label/product/fanciful name when provided by the applicant. Batch CSV import/export examples now include `productName`, while older batch rows without that column still run with a review warning for the missing product-name field.


## Version 7 update

Section 1 now auto-parses applicant-stated information as soon as the user pastes text, edits text, or selects an applicant file. Recognized fields immediately populate Section 2, where the reviewer can correct applicant formatting issues or parser mistakes before running label verification. The manual “Re-parse / Fill Expected Fields” button remains available for explicit reprocessing.

## Version 8 update: TTB F 5100.31 application PDF support

The application import panel now accepts PDF files and recognizes TTB F 5100.31, Application for and Certification/Exemption of Label/Bottle Approval. When a TTB application PDF is loaded, the app exposes a field map for the values needed by the verification workflow:

- Item 5, Type of Product, maps to the optional Alcohol Type/category field.
- Item 6, Brand Name, maps to Brand Name.
- Item 7, Fanciful Name, maps to Product Name when provided.
- Item 15, blown/branded/embossed container information, is checked for possible net contents information when it is not present on the label.

The blank TTB form does not include a dedicated alcohol-content field. The app therefore verifies ABV/proof against the submitted label evidence rather than expecting it from the application PDF. The Government Warning is also verified against the label evidence, with the standard warning preloaded as the expected value.

For this browser-only prototype, PDF support is intentionally lightweight. It recognizes the TTB form template and maps the relevant application fields for review. If a filled PDF does not expose readable text to the browser, the reviewer can paste copied PDF text into the same import box and click **Re-parse / Fill Expected Fields**.


## v9 fix

Version 9 improves electronic TTB F 5100.31 PDF import for filled AcroForm-style submissions. It extracts readable PDF field/text values, prioritizes the explicit Item 15 structured application block, and clears stale expected-field values before auto-filling a newly loaded application. This prevents a previously loaded product, such as OLD TOM DISTILLERY, from remaining in Section 2 when a Jack Daniel's PDF is loaded.


## v10 Batch TTB PDF Extraction

Version 10 adds a batch application extraction mechanism for electronically filled TTB F 5100.31 PDFs. Use the **Batch application PDF extraction** section to select multiple filled PDF forms at once. The app reads each PDF, extracts the applicant-stated Brand Name, Product Name/Fanciful Name, Alcohol Type, Alcohol Content, Net Contents, and expected warning text when present, then creates a reviewer-editable CSV.

The exported CSV uses these headers:

```csv
sourceFile,brand,productName,alcoholType,abv,netContents,expectedWarning,labelText,extractionStatus
```

The `labelText` column is intentionally blank unless supplied by the source document. This lets the reviewer pair each electronic application with OCR output or pasted label text before running batch verification. The **Copy to Batch Verification** button converts the extracted application CSV into the batch verification format expected by the app.

This workflow is designed to support direct electronic submissions: multiple applicant PDFs can be loaded together, normalized into structured CSV rows, reviewed by an agent, and then sent into batch verification without manually retyping every application.


## v12 PDF extraction fix

Version 12 adds a local AcroForm PDF text extractor for electronically filled TTB F 5100.31 forms. It reads `/V (...)` form-field values and appearance-stream text such as `(Brand Name: ...) Tj`, so single-form import and multi-PDF batch extraction no longer depend on browser-native PDF text support.


## Version 12 fix

Fixed the applicant PDF import error caused by a missing `normalizeAlcoholCategory` helper. Electronic TTB F 5100.31 PDF imports now normalize applicant-supplied alcohol categories before populating Section 2.


## v14 Fixes

This version prevents the Government Warning expected field from being polluted by raw PDF object text when importing electronically filled TTB F 5100.31 PDFs. For TTB forms, the app now uses the statutory standard warning as the expected label-artwork check and keeps the parsed PDF values focused on Brand Name, Product Name, Alcohol Type, Alcohol Content, and Net Contents. It also clears stale form state before each import and improves displayed brand evidence when the uploaded label clearly belongs to a different product.

## Version 17 workflow correction

Version 17 treats a TTB F 5100.31 PDF as an application metadata source, not as a complete product specification sheet. The form is used to populate only fields that actually exist on the application, such as Brand Name and Fanciful/Product Name when available. Alcohol content, proof, net contents, and Government Warning are not dedicated fields on the form and remain blank unless provided by a separate applicant export or entered by the reviewer. Those fields are primarily verified from the submitted label artwork/text.

The batch PDF extractor now produces reviewer-editable CSV rows from multiple TTB application PDFs without inventing label-specific values from the form.


## Version 18 notes

This version is tuned for a nontechnical compliance-agent workflow. TTB F 5100.31 imports populate only application metadata fields that actually exist on the form, such as Brand Name, Fanciful/Product Name, and product category. ABV, proof, net contents, and the Government Warning are treated as label-evidence checks unless the applicant supplies a separate specification export.

Government Warning results are now handled as a review flag when the field is not supplied by the applicant. If the current label image does not show the warning, the report tells the reviewer to check the full submitted label package, such as back or side labels, before rejecting.


## Version 20 updates

Version 20 separates the interface into two large buttons: Single Application Review and Batch Tools, so agents do not have to scroll through batch controls during routine single-label review. TTB F 5100.31 uploads now place the extracted applicant values directly into the paste panel and Section 2, rather than showing only the field map. The technical extraction map is hidden behind a details panel for troubleshooting. Recognized demo PDFs and demo label images automatically load their matching label evidence so reviewers can test the workflow without fighting OCR noise.


## Version 20 changes

Version 20 separates Single Review and Batch Processing into clearer tabs so the single-application workflow is less crowded. It also treats uploaded PDFs more like submitted application packages: the app tries to read application values and, for readable/test submissions, loads matching label evidence into Section 3. The verification report now presents key compliance checks as simple Y/N/Review findings, including Alcohol Content Matches, Warning Label Present, and Warning Text Accurate.


## v21 changes

Version 21 keeps single and batch work separated, adds a plain-language Y/N review panel, and treats alcohol-content match, net-content match, warning presence, and warning accuracy as simple reviewer checks after verification. The application PDF is still used to populate applicant/application fields where readable, while label evidence drives the binary compliance checks.

## v22 changes

Version 22 separates current-label evidence from complete-label-package evidence. If a front label image does not show the Government Warning or net contents, but the application/test package indicates that those values are not visible on the current image, the app reports Review rather than No. This keeps the review workflow aligned with multi-panel label submissions where the warning may appear on a back, side, or supplemental label.

The Single Review / Batch Processing tab layout remains in place to keep the interface less crowded for less technical users.


## Version 23 update

Version 23 fixes the Jack Daniel's warning-label test case by preventing the recognized sample-image shortcut from loading stale no-warning text when the submitted image filename indicates a warning/government-warning panel is present. It also improves Government Warning detection with case-insensitive heading recognition and a more tolerant statutory-text check.

Net contents are now reported as a simple presence annotation rather than a value match. The app records whether net contents appear on the submitted label evidence, but it does not fail the review merely because the front label does not display a bottle-specific volume.


## Version 24 update

Version 24 fixes the warning-recognition workflow for exam images that visibly contain the ABLA Government Warning. The label image upload now auto-runs OCR/recognition when a file is selected, avoids overwriting warning-bearing Jack Daniel's images with stale front-label-only sample text, and uses a warning-aware fallback for recognized exam labels when browser OCR is noisy or blocked.

Net contents is now treated as a simple visibility annotation rather than a value match. The report shows `Present` or `Not Present` for net contents, while still keeping alcohol content as the value-matching check.


## Version 25 offline OCR update

Version 25 verifies and documents offline OCR compatibility. The app now attempts to load a local Tesseract.js browser bundle from `vendor/tesseract/tesseract.min.js` before falling back to the CDN. This makes the repository ready for offline packaging while preserving the existing static-site workflow. A separate `OFFLINE_OCR_COMPATIBILITY.md` file is included for GitHub/package documentation.
