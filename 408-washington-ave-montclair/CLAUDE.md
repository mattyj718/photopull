# Real Estate Photo Recovery

This directory contains recovered listing photos for properties. See `playbook.md` for the step-by-step workflow.

## Quick Start
Given a Zillow URL (or address), follow `playbook.md` to find and download photos, then open `viewer.html` in a browser.

## Key Lessons
- Most major real estate sites (Zillow, Redfin, Compass, Coldwell Banker, RE/MAX) return 403 on automated fetch — skip them
- Wayback Machine rarely has snapshots of listing pages (JS-heavy, robots.txt blocked) — try but don't rely on it
- Sites with accessible CDN image URLs are the goldmine — see playbook for the ranked list
- Always `file` check downloads to confirm they're real images, not HTML error pages
