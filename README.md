# WBD Dealer Vault

Static asset vaults for **Wiedmann Bros Distributing** and **Orange Aftermarket** dealer programs. Hosted as registered-users-only pages on each site's Shift4Shop store; content is manifest-driven from this repo.

```
wbd-dealer-vault/
├── README.md                  ← you are here
├── wbdc/
│   ├── manifest.json          ← canonical asset list for WBDC
│   ├── vault-page.html        ← paste into S4S extra page
│   └── admin.html             ← open locally to add manifest entries
└── oa/
    ├── manifest.json          ← canonical asset list for Orange Aftermarket
    ├── vault-page.html        ← paste into S4S extra page
    └── admin.html             ← open locally to add manifest entries
```

---

## How it works

The vault page lives on each S4S site as a **registered-users-only extra page**. When a dealer visits the page, a small JS bundle fetches `manifest.json` from this GitHub repo's raw URL, then renders categories and asset cards from the data.

To update the vault: edit `manifest.json` and commit. The next dealer to visit the page sees the new content (cache-busted via query param).

The admin tool (`admin.html`) is a local utility that helps you generate properly-shaped manifest entries without hand-editing JSON.

---

## Initial setup (one-time per site)

1. **Create the S4S extra page:**
   - Login to Shift4Shop admin → Content → Pages → Add Page
   - Title: "Dealer Resources" (or whatever you want)
   - Slug: e.g., `dealer-resources` (results in URL `/dealer-resources.html`)
   - **Restrict access to: Registered Users Only**
   - Open the page's HTML view, paste in the entire contents of `vault-page.html`
   - Save

2. **Update the manifest URL in the page:**
   Open the pasted `vault-page.html` content in S4S admin and find this line near the top of the `<script>` block:

   ```javascript
   var MANIFEST_URL = 'https://raw.githubusercontent.com/wos1337/wbd-dealer-vault/main/wbdc/manifest.json';
   ```

   Make sure it points at the correct manifest for that site (`wbdc/manifest.json` or `oa/manifest.json`). Commit any URL changes back to this repo so you don't lose them.

3. **Create the asset folders in S4S File Manager:**
   - `/assets/images/vault/files/` — actual files dealers will download
   - `/assets/images/vault/thumbs/` — preview thumbnails (use 600×450 or similar 4:3)

4. **Add a link to the vault from the dealer's My Account page:**
   In `myaccount.html` (S4S theme template), add a tile or button linking to `/dealer-resources.html`. Example:

   ```html
   <a href="/dealer-resources.html" class="btn btn-primary">
     Dealer Resource Vault
   </a>
   ```

---

## Adding a new asset

The everyday workflow:

1. **Upload the file** to S4S File Manager → `/assets/images/vault/files/your-file.svg`
2. **Generate a thumbnail** (PNG/JPG, 4:3 aspect ratio works best) and upload to `/assets/images/vault/thumbs/your-file.png`
3. **Open `admin.html` locally** in your browser (just double-click the file)
4. **Fill out the form:**
   - Download URL → paste the path you uploaded to (e.g. `/assets/images/vault/files/your-file.svg`)
   - Thumbnail URL → the thumbnail path
   - Format auto-fills from extension (override if needed)
   - Title, description, category
5. **Click Copy JSON** → the entry is on your clipboard
6. **Open `manifest.json`** (in your editor or directly on github.com)
7. **Paste the entry** into the `assets` array (anywhere in it — order doesn't matter, the page renders in the order you list)
8. **Update the `updated` field** at the top of the manifest to today's date
9. **Commit and push** (or commit via the GitHub web UI)

That's it. The vault page picks up the change immediately on the next page load (cache-busted).

---

## Adding a new category

1. **Open `admin.html`** → Manage Categories tab
2. **Fill in the form** — name, optional description (id auto-generates from name)
3. **Click "Add Category"** — saves to the admin tool's local storage so it shows in the dropdown when you add assets
4. **Click "Copy Categories JSON"** — full categories array is on your clipboard
5. **Replace the entire `categories` array** in `manifest.json` with the pasted JSON
6. **Commit**

---

## Removing or updating an asset

Edit `manifest.json` directly. To remove an asset, delete its entry from the `assets` array. To update fields, just edit the entry inline. Commit.

---

## The manifest schema

```json
{
  "site": "wiedmannbros.com",
  "updated": "2026-05-06",
  "categories": [
    {
      "id": "logos",
      "name": "Logos",
      "description": "Optional category description"
    }
  ],
  "assets": [
    {
      "id": "unique-slug",
      "category": "logos",
      "title": "Display title",
      "description": "Long-form description shown under the title",
      "thumbnail": "/path/to/thumb.png",
      "download_url": "/path/to/file.svg",
      "format": "SVG",
      "size_kb": 12,
      "dimensions": null,
      "added": "2026-05-06"
    }
  ]
}
```

**Field notes:**
- `id` — unique slug, lowercase, hyphens only. The admin tool auto-generates from the title.
- `category` — must match one of the `id` values in the `categories` array.
- `thumbnail` — optional. If null/missing, a placeholder shows.
- `download_url` — required. Relative paths work on the live site.
- `format` — display badge (SVG, PNG, PDF, etc.).
- `size_kb` — optional, just for display.
- `dimensions` — optional. Useful for banners (`"728×90"`) and logos with specific size requirements.
- `added` — ISO date string for the "added" indicator.

---

## Troubleshooting

**Vault page shows "Could not load resources" error:**
- Check the `MANIFEST_URL` in the page's `<script>` matches your repo path
- Make sure `manifest.json` is valid JSON (use the Preview Manifest tab in `admin.html` to validate)
- Check browser console for the actual fetch error

**Asset shows but thumbnail is broken:**
- Verify the thumbnail URL works directly in a browser
- Check the path matches what you uploaded to S4S File Manager
- Falls back to a format badge automatically if the image fails to load

**New asset doesn't appear after committing:**
- The page cache-busts on every load (`?t=` query param), so it should be immediate
- GitHub raw URLs sometimes have a few-minute CDN delay — wait 1–2 minutes and refresh
- Make sure you committed AND pushed (if working locally)

**Asset references a category that doesn't exist:**
- The Preview Manifest tab in `admin.html` flags orphan assets
- Either add the missing category or fix the asset's `category` field

---

## Architecture choices and why

**Why GitHub for the manifest instead of S4S?**
- Better edit history — every change is logged with author and timestamp
- Easier to review and roll back changes
- Can edit from the GitHub web UI on any device — no S4S admin login needed
- Works offline (you can prepare batches in your editor, push when ready)

**Why a separate admin tool instead of editing JSON directly?**
- Reduces typos in `id`, `category`, and date fields
- Auto-generates the slug from the title (one less thing to think about)
- Live preview of how the card will look before you commit
- Categories tab persists between sessions via localStorage, so you don't lose your category list when you close the tool

**Why two separate vaults instead of one shared?**
- WBDC and OA serve different dealer programs with different content
- Categories and assets don't overlap meaningfully
- Visual style is different (WBDC dark/lime editorial vs OA Kubota orange utilitarian)
- Updating one doesn't risk affecting the other

**Why "registered users only" and not Cloudflare Access or shared password?**
- S4S already has a registered-users gate that's free and zero-maintenance
- Content is marketing assets, not pricing or confidential data — risk of leakage is low
- Dealers already have S4S accounts on these sites for ordering — no new credentials to manage
- Can upgrade to stricter auth later if content sensitivity increases

---

## Files

| File | Purpose |
|------|---------|
| `wbdc/manifest.json` | Canonical WBDC vault content |
| `wbdc/vault-page.html` | Pasted into wiedmannbros.com S4S extra page |
| `wbdc/admin.html` | Local-use admin tool for WBDC vault |
| `oa/manifest.json` | Canonical Orange Aftermarket vault content |
| `oa/vault-page.html` | Pasted into orangeaftermarket.com S4S extra page |
| `oa/admin.html` | Local-use admin tool for OA vault |
