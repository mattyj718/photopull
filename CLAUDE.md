# Real Estate Photo Recovery Tool

## On startup
When the user starts a conversation in this directory, immediately ask:

> Paste a real estate listing URL (Zillow, Redfin, etc.) or a street address and I'll recover the listing photos.

Then follow the runbook below automatically — no further prompting needed.

## Runbook

### 1. Parse the input
Extract: full street address, city, state, zip, and any site-specific IDs (Zillow zpid, Redfin home ID).

### 2. Create output directory
```
mkdir -p ~/dev/tmp/<address-slug>/
```
Use lowercase kebab-case (e.g. `123-main-st-springfield-nj`).

### 3. Search for listings with photos
Run a WebSearch for `"<full address>" photos listing` to find which sites have the property.

### 4. Fetch photos — try sites in this order, stop once you get a good set

**Skip these (always 403):** Zillow, Redfin, Compass, Coldwell Banker, RE/MAX

**Try these:**
1. **Zumper** (rental only) — WebFetch the page, extract `img.zumpercdn.com/<id>/1280x960` URLs
2. **Weichert** — WebFetch the page, extract `wdcimagestorageprodeast.blob.core.windows.net` URLs
3. **Homes.com** — WebFetch, look for CDN image URLs
4. **Realtor.com** — WebFetch, look for `ap.rdcpix.com` URLs
5. **Movoto** — WebFetch, look for image URLs
6. **Any local brokerage sites** that appeared in search results — try WebFetch on those
7. **Wayback Machine** — quick CDX API check via curl (rarely works but fast to try)

When fetching a page, prompt: "List all photo URLs or image URLs for this property listing. I want direct image URLs from CDNs."

### 5. Download photos
```bash
cd ~/dev/tmp/<address-slug>/
for url in <urls>; do curl -s -o "photo_$(basename $url)" "$url" & done && wait
```

### 6. Validate
```bash
file *.jpg *.png *.webp 2>/dev/null
```
Delete any files that are HTML error pages or zero-byte.

### 7. Generate viewer
Copy the viewer.html template from `~/dev/tmp/408-washington-ave-montclair/viewer.html` into the new directory. Update the `CONFIG` block with:
- `address`: the full address
- `details`: bed/bath/type if known from search results
- `source`: which site the photos came from
- `photos`: array of downloaded filenames

### 8. Launch
```bash
xdg-open ~/dev/tmp/<address-slug>/viewer.html
```

### 9. Report results
Tell the user how many photos were recovered, from which source, and where they're saved.
