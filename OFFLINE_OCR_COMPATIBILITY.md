# Offline OCR Compatibility

## Verified compatibility path: Tesseract.js offline bundle

LabelCheck AI uses the standard Tesseract.js browser API:

```javascript
Tesseract.recognize(file, "eng")
```

Because the app only needs `window.Tesseract` and calls `Tesseract.recognize(...)`, it is compatible with a locally vendored Tesseract.js browser build. The app now loads OCR in this order:

1. `vendor/tesseract/tesseract.min.js` for offline/air-gapped use.
2. CDN fallback when online and the local file is not present.
3. Manual pasted label text if no OCR library is available.

## Offline OCR options

### Option A: Browser-only offline OCR

Add the Tesseract.js browser bundle to:

```text
vendor/tesseract/tesseract.min.js
```

Then run the app locally:

```bash
python -m http.server 8000
```

Open:

```text
http://localhost:8000
```

Disconnect the network and test image OCR. If the local bundle is present, the OCR button should work without internet access.

### Option B: Desktop OCR preprocessing

The app is also compatible with OCRmyPDF, Tesseract CLI, EasyOCR, and PaddleOCR as external/offline preprocessing tools because the verification engine accepts pasted label text. Run OCR outside the browser, paste the output into Section 3, and click Verify Label.

Examples:

```bash
tesseract label.png stdout -l eng
ocrmypdf --sidecar output.txt input.pdf output_ocr.pdf
```

## GitHub repository description

Use this as the GitHub repository description:

```text
Offline-capable alcohol label verification prototype with local Tesseract.js OCR support, TTB F 5100.31 parsing, batch CSV review, and human-in-the-loop Y/N compliance checks.
```

Suggested GitHub topics:

```text
ttb, alcohol-labels, compliance, ocr, tesseract-js, offline-first, static-site, human-in-the-loop
```

## Offline verification checklist

Before calling the package offline-ready, disconnect the network and confirm:

1. `index.html` loads locally.
2. A TTB F 5100.31 PDF imports into Section 2.
3. A label image OCR attempt works if `vendor/tesseract/tesseract.min.js` is present.
4. Pasted label text verifies even if OCR is unavailable.
5. Batch CSV verification and CSV export work.
