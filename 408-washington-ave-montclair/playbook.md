# Real Estate Photo Recovery Playbook

## Input
User provides a Zillow URL, Redfin URL, or street address.

## Step 1: Extract property details
From the URL or address, identify:
- Full street address
- City, state, zip
- Zillow zpid or Redfin home ID (if available)

## Step 2: Create output directory
```
mkdir -p ~/dev/tmp/<address-slug>
```
Use kebab-case address as the directory name.

## Step 3: Search for accessible listings (ranked by success rate)

### Tier 1 — Most likely to have fetchable image URLs

1. **Zumper** (rental listings only)
   - URL pattern: `https://www.zumper.com/address/<address-slug>`
   - Images on CDN: `https://img.zumpercdn.com/<id>/1280x960`
   - Fetch the page, extract image IDs, download directly

2. **Weichert**
   - Search: `"<address>" site:weichert.com`
   - Images on CDN: `https://wdcimagestorageprodeast.blob.core.windows.net/mls009/Large/<id>.jpg`
   - Only first ~2 images in static HTML; rest need JS rendering

3. **Homes.com**
   - Search: `"<address>" site:homes.com`
   - May have CDN image URLs in static HTML

4. **Realtor.com**
   - Search: `"<address>" site:realtor.com`
   - Images sometimes on: `https://ap.rdcpix.com/`

5. **Movoto**
   - Search: `"<address>" site:movoto.com`
   - Sometimes has accessible image galleries

### Tier 2 — Worth trying but often blocked

6. **Trulia** (owned by Zillow, usually blocked)
7. **RE/MAX Gateway** (local broker sites sometimes less restrictive)
8. **Local brokerage sites** — search for the MLS number + "photos"

### Tier 3 — Unlikely but possible

9. **Wayback Machine CDX API**
   ```
   curl -s "https://web.archive.org/cdx/search/cdx?url=www.zillow.com/homedetails/<path>&output=json&limit=20"
   ```
   Rarely has real estate listings but worth a quick check.

10. **Google Images** — search `"<full address>" house listing`
    Can surface cached thumbnails but not full-res.

### DO NOT bother with (403 every time):
- Zillow direct fetch
- Redfin direct fetch
- Compass direct fetch
- Coldwell Banker direct fetch

## Step 4: Download photos
```bash
cd ~/dev/tmp/<address-slug>
for url in <list-of-urls>; do
  curl -s -o "photo_$(basename $url)" "$url" &
done
wait
```

## Step 5: Validate downloads
```bash
file *.jpg  # confirm they're actual JPEG image data, not HTML error pages
ls -la *.jpg | wc -l  # count successful downloads
```
Delete any files that are HTML error pages or zero-byte.

## Step 6: Generate viewer
Copy `viewer.html` into the directory and open it. The viewer auto-detects all `.jpg`/`.png`/`.webp` files in its directory.

## Notes
- If the property was never a rental, Zumper won't have it — go straight to Weichert/Homes.com/Realtor.com
- MLS numbers appear as watermarks on photos (e.g., "6418504") — useful for identifying which listing era the photos are from
- Some sites serve WebP instead of JPEG — the viewer handles both
