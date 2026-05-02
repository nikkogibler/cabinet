---
name: screenshot-cleanup
description: Cleans up a folder of screenshots and images on macOS by generating a temporary OCR-assisted helper script, sorting files into category folders, and renaming files to readable slugs. Use when the user says "clean up my desktop", "organize my screenshots", "sort my images", or wants a screenshot dump triaged.
version: 1.0.0
user-invocable: true
argument-hint: "[source folder path, or leave blank for default]"
license: MIT
metadata:
  author: Nik Gibler
  copyright: Copyright (c) 2026 Nik Gibler
---

# Screenshot Cleanup

Clean up a screenshot or image dump on macOS.

This is a bounded local-automation skill. It writes a temporary helper script to `/tmp/sort_and_rename.py`, may compile a temporary Swift OCR binary to `/tmp/desktop_ocr_bin`, and renames or moves image files inside the chosen source folder. Use it when the user actually wants the folder reorganized, not when they need a preview-only audit.

## When To Use

Use this skill when the user says things like:

- `clean up my desktop`
- `organize my screenshots`
- `sort my images`
- `screenshot cleanup`
- `organize my downloads/screenshots folder`

## When Not To Use It

Do not use this skill when:

- the user wants a dry run or non-destructive preview only
- the folder contains files they do not want renamed or moved
- a cross-platform workflow is required
- the user needs a general DAM system rather than a one-time cleanup pass

## Safety Boundary

- Confirm the source folder explicitly. Default to `~/Desktop` only when the user leaves it unspecified.
- Explain that files will be renamed and moved inside that folder.
- If the user cannot tolerate immediate reorganization, recommend duplicating the folder first.
- Never assume Desktop once the user names a different folder.
- The canonical OCR path is macOS-only because it uses Swift and Vision.
- Always write the helper script fresh. Do not trust a stale copy in `/tmp`.

## Workflow

### 1. Confirm The Source Folder

Ask the user which folder to clean up. Default is `~/Desktop`, but it can be any folder such as Downloads or a dedicated screenshots directory.

Everything that follows happens **inside that folder**. Subfolders are created there, and matching files are renamed and moved there.

### 2. State The Side Effects

Before running anything, say plainly that this workflow:

- creates category subfolders inside the source folder
- renames matching image files
- moves those files into the created category folders
- may compile a temporary OCR helper binary under `/tmp`

If the user wants a reversible safety step, recommend copying the source folder before cleanup.

### 3. Write The Canonical Helper Script

Write the script below to `/tmp/sort_and_rename.py`, then run it. This helper is generated at runtime. Do not store it in the repo as a standalone script artifact.

The script creates any missing taxonomy subfolders inside the source folder before processing. If some already exist, they are left untouched with `exist_ok=True`.

**Script** (write to `/tmp/sort_and_rename.py`):

```python
#!/usr/bin/env python3
"""
sort_and_rename.py — Screenshot Cleanup skill helper
Usage: python3 /tmp/sort_and_rename.py [source_folder]
Default source_folder: ~/Desktop
"""
import sys, re, subprocess, time
from pathlib import Path
from datetime import datetime
from collections import defaultdict

IMAGE_EXTENSIONS = {'.png', '.jpg', '.jpeg', '.webp', '.avif', '.svg'}
OCR_BINARY = Path('/tmp/desktop_ocr_bin')
OCR_SOURCE = Path('/tmp/desktop_ocr.swift')
OCR_TIMEOUT_SECONDS = 6

SWIFT_SOURCE = r"""
import Foundation
import Vision
import AppKit
let path = CommandLine.arguments[1]
let url = URL(fileURLWithPath: path)
guard let image = NSImage(contentsOf: url) else { fputs("image load failed\n", stderr); exit(1) }
guard let tiff = image.tiffRepresentation,
      let bitmap = NSBitmapImageRep(data: tiff),
      let cgImage = bitmap.cgImage else { fputs("cgimage failed\n", stderr); exit(1) }
let request = VNRecognizeTextRequest()
request.recognitionLevel = .accurate
request.usesLanguageCorrection = true
let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])
do {
    try handler.perform([request])
    let results = (request.results as? [VNRecognizedTextObservation]) ?? []
    for obs in results { if let c = obs.topCandidates(1).first { print(c.string) } }
} catch { fputs("vision failed: \(error)\n", stderr); exit(1) }
"""

CATEGORY_KEYWORDS = [
    ('finance-pricing', ['invoice','receipt','payment','price','stripe','billing','subscription','charge','revenue','cost','budget','expense','transaction','refund','tax','vat','total due','amount due']),
    ('e-commerce',      ['checkout','apple pay','paypal','shipping','delivery','order confirmation','free shipping','add to cart','buy now']),
    ('shoppingcarts',   ['cart','wishlist','saved items','shopping bag','continue shopping','items in your cart','your cart']),
    ('to-buy',          ['buy','shop','purchase','order','wish','amazon','product listing','price tag','add to wishlist']),
    ('product-proposals',['proposal','pitch','deck','roadmap','strategy','feature spec','requirements','specification','prd','scope of work']),
    ('documents',       ['pdf','document','report','slide','presentation','form','contract','policy','terms','certificate','readme','guide','instructions','agreement','privacy policy']),
    ('app-screenshots', ['dashboard','sidebar','widget','panel','settings','modal','button','navbar','toolbar','tab','menu','notification','error','alert','loading','login','signup','sign in','sign up','register','onboarding']),
    ('reminder',        ['reminder','todo','to-do','task','deadline','due','calendar','schedule','meeting','appointment','event','alarm']),
    ('travel',          ['flight','hotel','airbnb','booking','itinerary','passport','ticket','trip','rental','airport','destination','check-in','boarding pass']),
    ('branding',        ['logo','brand','identity','mockup','banner','campaign','asset','marketing','advertisement','creative brief']),
    ('personal',        ['health','fitness','workout','calories','weight','sleep','journal','diary','note','tracker','habit','steps','heart rate']),
    ('funny',           ['lol','meme','joke','funny','haha','laugh','humor','comic','rofl','lmao']),
    ('art',             ['illustration','artwork','drawing','sketch','painting','poster','graphic','vector','icon']),
    ('inspiration',     ['quote','idea','concept','design','mood','palette','color','font','typography','layout','reference','inspo','inspired']),
    ('photos',          ['portrait','selfie','photo','photograph','picture']),
    ('images',          []),
    ('misc',            []),
]

DEFAULT_TAXONOMY = ['images','documents','finance-pricing','reminder','inspiration','funny','personal','art','to-buy','app-screenshots','photos','branding','e-commerce','travel','product-proposals','shoppingcarts','misc']

def ensure_ocr_binary():
    if OCR_BINARY.exists(): return True
    OCR_SOURCE.write_text(SWIFT_SOURCE)
    r = subprocess.run(['swiftc','-o',str(OCR_BINARY),str(OCR_SOURCE)], capture_output=True, text=True)
    if r.returncode != 0:
        print(f'[ERROR] swiftc compile failed:\n{r.stderr}', file=sys.stderr); return False
    return True

def ocr_file(path):
    if not OCR_BINARY.exists(): return ''
    try:
        proc = subprocess.Popen([str(OCR_BINARY), str(path)], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        deadline = time.time() + OCR_TIMEOUT_SECONDS
        while proc.poll() is None:
            if time.time() > deadline: proc.kill(); return ''
            time.sleep(0.05)
        stdout, _ = proc.communicate()
        return stdout.decode('utf-8', errors='replace')
    except Exception: return ''

def classify(ocr_text, ext):
    lower = ocr_text.lower()
    for cat, kws in CATEGORY_KEYWORDS:
        if cat in ('images','misc'): continue
        if any(kw in lower for kw in kws): return cat
    if not ocr_text.strip():
        return 'photos' if ext in ('.jpg','.jpeg') else 'misc'
    return 'misc'

def make_slug(ocr_text, max_words=5):
    if not ocr_text.strip(): return 'untitled'
    lines = [l.strip() for l in ocr_text.splitlines() if l.strip()]
    if not lines: return 'untitled'
    slug = '-'.join(lines[0].split()[:max_words]).lower()
    slug = re.sub(r'[^a-z0-9\-]','',slug)
    slug = re.sub(r'-{2,}','-',slug).strip('-')
    return slug or 'untitled'

def unique_dest(dest_dir, name):
    stem, ext = Path(name).stem, Path(name).suffix
    candidate = dest_dir / name
    counter = 2
    while candidate.exists():
        candidate = dest_dir / f'{stem}-{counter}{ext}'; counter += 1
    return candidate

def main():
    source = Path(sys.argv[1]).expanduser().resolve() if len(sys.argv) > 1 else Path.home() / 'Desktop'
    if not source.is_dir():
        print(f'[ERROR] Not a directory: {source}', file=sys.stderr); sys.exit(1)
    print(f'Source: {source}\n')
    for folder in DEFAULT_TAXONOMY:
        (source / folder).mkdir(parents=True, exist_ok=True)
    files = [f for f in source.iterdir() if f.is_file() and f.suffix.lower() in IMAGE_EXTENSIONS]
    if not files:
        print('No image files found.'); return
    print(f'Found {len(files)} file(s). Compiling OCR binary...')
    ocr_ok = ensure_ocr_binary()
    if not ocr_ok: print('[WARN] OCR unavailable — extension-only classification.')
    print()
    counts = defaultdict(int)
    errors = []
    for i, file in enumerate(sorted(files), 1):
        ext = file.suffix.lower()
        print(f'[{i}/{len(files)}] {file.name}', end=' ... ', flush=True)
        if ext == '.svg':
            ocr_text, category = '', 'branding'
        else:
            ocr_text = ocr_file(file) if ocr_ok else ''
            category = classify(ocr_text, ext)
        slug = make_slug(ocr_text)
        dest_name = f'{category}-{slug}-{datetime.fromtimestamp(file.stat().st_mtime).strftime("%Y-%m-%d")}{ext}'
        dest_path = unique_dest(source / category, dest_name)
        try:
            file.rename(dest_path)
            print(f'{category} → {dest_path.name}')
            counts[category] += 1
        except Exception as e:
            print(f'ERROR: {e}'); errors.append(f'{file.name}: {e}')
    print(f'\n{"Category":<22} {"Files":>6}')
    print('─'*30)
    total = 0
    for cat, _ in CATEGORY_KEYWORDS:
        n = counts.get(cat,0)
        if n: print(f'{cat:<22} {n:>6}'); total += n
    remaining = [f for f in source.iterdir() if f.is_file() and f.suffix.lower() in IMAGE_EXTENSIONS]
    print('─'*30)
    print(f'{"Total":<22} {total:>6}  ({len(remaining)} remaining)')
    if errors:
        print(f'\nErrors ({len(errors)}):')
        for e in errors: print(f'  {e}')
    if remaining:
        print('\nStill in source:')
        for f in sorted(remaining): print(f'  {f.name}')

if __name__ == '__main__':
    main()
```

### 4. Run The Script

After writing the script, run it with the confirmed source folder:

```bash
python3 /tmp/sort_and_rename.py /path/to/source/folder
```

Replace `/path/to/source/folder` with the actual folder the user specified, such as `~/Desktop`, `~/Downloads`, or `~/Documents/Screenshots`.

### 5. Report Results

After the script exits, print a tidy summary such as:

```text
Category           Files moved
──────────────────────────────
personal                    12
documents                    8
app-screenshots              6
branding                     4
misc                         3
...
Total                       42  (0 remaining in source)
```

Then list any files still in the source folder. If any remain, say why, such as unsupported format or permission error.

## Classification Priority Order

Most specific first. First match wins:

1. `finance-pricing` — invoice, receipt, payment, price, stripe, billing, subscription, charge, revenue, cost, budget, expense, transaction
2. `e-commerce` — checkout, apple pay, paypal, shipping, delivery, order confirmation
3. `shoppingcarts` — cart, wishlist, saved items, bag, continue shopping
4. `to-buy` — buy, shop, purchase, order, wish, amazon, product listing, item, price tag
5. `product-proposals` — proposal, pitch, deck, roadmap, strategy, feature, spec, requirements
6. `documents` — pdf, document, report, slide, presentation, form, contract, policy, terms, certificate, readme, guide, instructions
7. `app-screenshots` — dashboard, sidebar, widget, panel, settings, modal, button, navbar, toolbar, tab, menu, notification, error, alert, loading, login, signup
8. `reminder` — reminder, todo, task, deadline, due, calendar, schedule, meeting, appointment, event, alarm
9. `travel` — flight, hotel, airbnb, booking, itinerary, passport, ticket, trip, rental, airport, destination
10. `branding` — logo, brand, identity, mockup, banner, campaign, asset, marketing, advertisement, creative
11. `personal` — health, fitness, workout, calories, weight, sleep, journal, diary, note, tracker, habit
12. `funny` — lol, meme, joke, funny, haha, laugh, humor, comic
13. `art` — illustration, artwork, drawing, sketch, painting, poster, graphic, vector, icon
14. `inspiration` — quote, idea, concept, design, mood, palette, color, font, typography, layout, reference
15. `photos` — portrait, selfie, photo, also zero-OCR `.jpg` or `.jpeg`
16. `images` — generic image fallback
17. `misc` — final fallback

## Key Decisions

- **Source folder drives everything**: category folders are always created inside the user-supplied source folder.
- **Existing folders are safe**: all `mkdir` calls use `exist_ok=True`, so existing category folders are not cleared.
- **Generated helper only**: write the helper to `/tmp`, do not keep it in the repo as a standalone executable.
- **U+202F in macOS filenames**: use Python `pathlib` exclusively rather than shell `mv`.
- **OCR timeout**: kill Swift OCR after 6 seconds per file to avoid hangs on corrupt images.
- **No guessing for photos**: zero-OCR `.jpg` or `.jpeg` files go to `photos`, not `misc`.
- **SVG files**: skip OCR and classify as `branding` by default, unless the user explicitly wants a different rule.
- **Duplicate names**: append `-2`, `-3`, and so on until the destination is clear.