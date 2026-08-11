# Product Scan Label

A mobile-first, single-page web app for scanning shipping/delivery labels (adidas-style
"Article / Origin / Size/Qty" tags), reviewing the extracted data, and exporting a
project as an Excel report.

**No backend, no build step.** It's one file — `index.html` — that runs entirely in the
browser. That's what makes it deployable to GitHub Pages in about a minute.

## What it does

- **Camera capture** — opens your phone's native camera, no app install.
- **OCR** — reads the photo client-side with Tesseract.js and pulls out a SKU, origin,
  and size/quantity pairs using pattern matching tuned to the two sample labels you sent
  (e.g. `KR9761 / VN / 58/5 54/7`, `JN7094 / JO / 28/1`).
- **Review before saving** — every field is editable before it's added, since OCR on a
  creased, ink-smudged label will not be perfect every time.
- **Projects** — each project is saved in the browser's local storage on your phone, so
  closing the tab or coming back tomorrow keeps your data. You can have multiple projects
  and switch between them, or start a new one.
- **Excel export** — one click produces a `.xlsx` with columns `SKU / SIZE / QUANTITY /
  IMAGE / ORIGIN` using ExcelJS, entirely in the browser.

## The one thing it can't fully automate: images

You asked for it to search "adidas [SKU]" and drop the image straight into the sheet.
A static site with no server and no API key can't legally or technically do a silent
Google Images fetch on your behalf — Google's image search has no free, keyless API, and
most product photos live on hosts that block cross-origin fetches (which is exactly what
would be needed to embed the image into the workbook).

So the app does the next best thing:

1. Each item has a **"find image ↗"** link that opens a Google Images search for
   `adidas [SKU]` in a new tab.
2. You copy the image address of the photo you want and **paste it** into the field.
3. The app tries to fetch it in the background and embed it in the Excel export. If the
   source blocks that (many CDNs do), it falls back to storing the image as a clickable
   link in that cell instead, so nothing is lost — it's just a link rather than a
   thumbnail.

If down the line you want this fully automated, that needs a small server-side function
holding an API key (e.g. Google Custom Search JSON API, Bing Image Search, or SerpAPI) —
happy to build that as a second phase if useful.

## Deploying to GitHub Pages

I don't have a connection to your GitHub account from here, so I can't push this for you
directly — but it's a two-minute manual step:

1. Create a new repository on GitHub (e.g. `product-scan-label`), public.
2. Upload `index.html` (and this `README.md`) to the repository — either drag-and-drop
   in the GitHub web UI ("Add file" → "Upload files"), or via git:
   ```bash
   git init
   git add index.html README.md
   git commit -m "Product Scan Label v1"
   git branch -M main
   git remote add origin https://github.com/<your-username>/product-scan-label.git
   git push -u origin main
   ```
3. In the repository, go to **Settings → Pages**, set **Source** to `Deploy from a
   branch`, branch `main`, folder `/ (root)`, then **Save**.
4. GitHub gives you a URL like `https://<your-username>.github.io/product-scan-label/`
   within a minute or two — open that on your phone and it's ready to use.

Because everything is client-side, it works from any static host, not just GitHub Pages
(Netlify, Vercel, Cloudflare Pages, or just opening the file locally all work too — camera
capture does need `https://` or `localhost` to be allowed by the browser, so a local
`file://` open won't trigger the camera on most phones).

## Notes on data

- Data lives in `localStorage` on the specific browser/device you use. It won't sync
  across devices, and clearing site data/browser data will remove it. If you want the
  same project on your phone and laptop, exporting to Excel after each session is the
  safest way to keep a durable copy — I'm happy to add a simple import/backup-file feature
  next if that matters to you.
- OCR quality depends a lot on lighting and how flat the label is — the review step is
  there so a bad read never becomes a bad row in your report.
