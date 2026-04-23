# Apple.com → Adobe EDS Migration

**Source:** https://www.apple.com  
**Scraped:** 2026-04-23  
**Tool:** playwright-cli (Chromium)

---

## Output Files

| File | Contents |
|------|----------|
| `index.md` | Main homepage content in EDS block format |
| `nav.md` | Global navigation (primary + mega-menu submenus) |
| `footer.md` | Full footer directory columns |
| `README.md` | This file — migration notes |

---

## Page Structure Analysis

Apple's homepage is composed of three major content sections, a persistent global nav, and a full-directory footer.

### Global Navigation (nav.md)

Apple uses a two-tier nav:
- **Primary bar:** Apple logo + 11 top-level links (Store, Mac, iPad, iPhone, Watch, Vision, AirPods, TV & Home, Entertainment, Accessories, Support) + shopping bag icon
- **Mega-menu flyouts** per category (Store, Mac, iPad, iPhone, etc.), each containing grouped sub-links

**EDS mapping:** The top-level links are represented as a `Nav` block with a bulleted list. Each mega-menu group is represented as a second-level heading (`## Section`) followed by a list. EDS's standard nav block renders the first list as the top-level bar and subsequent `##` sections as flyout menus.

The shopping bag / search icon links are omitted from the document as they are UI chrome, not content.

---

### Section 1 — Hero Slider (`.section-hero`)

The hero is a full-bleed carousel of three product tiles, each containing:
- Product name (headline)
- One-line tagline (subhead)
- Two CTAs: "Learn more" (product page) + "Buy" / "Shop" (store deep-link)
- Background image (JPG, served from apple.com CDN)

**EDS mapping:** `Hero Slider` block — a three-row table, one row per slide. Each row has three columns: headline, subhead, and CTA links. In production EDS, the block decorator would load the corresponding hero images from CDN or a `/media` folder.

Products featured at time of scrape:
1. **iPhone** — Meet the latest iPhone lineup.
2. **MacBook Neo** — Amazing Mac. Surprising price.
3. **iPad Air** — Now supercharged by M4.

---

### Section 2 — Promo Grid (`.section-promo`)

Six smaller promotional tiles in a masonry/grid layout:

| Tile | Type |
|------|------|
| Community Letter from Tim | Full-bleed editorial (no product tagline) |
| Apple Watch Series 11 | Product promo |
| MacBook Air (M5) | Product promo |
| AirPods Pro 3 | Product promo |
| Apple Trade In | Commerce/offer promo |
| Apple Card | Financial product promo |

**EDS mapping:** `Cards` block — each tile is a row. Columns: title, tagline, CTA links. The "Community Letter" tile has no subhead; its copy is the CTA label itself ("Read the letter").

---

### Section 3 — Endless Entertainment Gallery (`.section-endless-entertainment-gallery`)

A horizontal media carousel with dot-nav pagination (9 items). Content is split across two logical groups:

**Apple TV+ / Sports (items 1–9):**
- Margo's Got Money Troubles (Comedy)
- Your Friends & Neighbors (Drama)
- F1 on Apple TV
- Criminal Record (Crime — new season)
- MLS on Apple TV
- Imperfect Women (Thriller)
- Friday Night Baseball (Live MLB)
- Shrinking (Comedy)
- Monarch: Legacy of Monsters (Adventure)

**Services cross-promo grid (Apple Fitness+, Apple Music, Apple Arcade):**
A 3×3 grid cycling through Fitness+, Music, and Arcade content.

**EDS mapping:** `Carousel` block for the TV+ carousel, `Cards` block for the 3×3 services grid. Each row maps to one content item: title, description, CTA link.

---

### Footer (footer.md)

Five column groups, each with one or more sub-sections:

1. **Shop and Learn** + **Apple Wallet**
2. **Account** + **Entertainment**
3. **Apple Store** (full retail + commerce links)
4. **For Business / Education / Healthcare / Government**
5. **Apple Values** + **About Apple**

Bottom bar: copyright, Privacy Policy, Terms of Use, Sales and Refunds, Legal, Site Map.

**EDS mapping:** Multiple `Columns` blocks, one per column group. EDS renders these as responsive column layouts. The bottom bar is plain text + inline links.

---

## Migration Decisions

### Block Naming
- `Hero Slider` — non-standard EDS name; maps to a custom block that renders a full-bleed carousel. Standard EDS projects often call this `Hero` with a variation class (`Hero (Slider)`).
- `Carousel` — standard EDS carousel block for the entertainment section.
- `Cards` — standard EDS cards block for the promo grid and services grid.
- `Columns` — standard EDS columns block for the footer directory.
- `Fragment` — used for the footnotes/legal text, so it can be shared across pages (e.g., all pages using the Trade In or Apple Card offer share the same footnote document).
- `Metadata` — standard EDS metadata block at the bottom of index.md.

### Images
Apple's hero and promo images are served from `https://www.apple.com/v/home/cm/images/`. In a real EDS project these would be:
1. Downloaded and committed to `/media/` in the project repo, or
2. Referenced via external URL using EDS's external image support.

Image URLs are preserved in comments within the source data but not embedded in the Markdown files, since EDS Markdown does not use inline `![img](url)` for block images — the block decorator fetches imagery based on content context or explicit `Picture` columns.

### Content Not Migrated
- Country/region selector modal (UI chrome, not page content)
- Navigation search overlay (UI chrome)
- Shopping bag count (dynamic, session-specific)
- Animated/video backgrounds on hero tiles (assets would need separate treatment)
- Accessibility attributes (ARIA labels, roles — handled by EDS block decorators in JS/CSS)

### Footnote Numbering
Apple uses superscript ¹ inline in promo copy ("trade in iPhone 13 or higher.¹"). The footnote text is preserved verbatim in a `Fragment` block at the bottom of `index.md`.

---

## EDS Project Setup Notes

To use these files in an EDS project:

1. Place `index.md`, `nav.md`, `footer.md` in the root of your Google Drive / SharePoint folder (or repo for code-based projects).
2. Create block implementations for `hero-slider`, `carousel`, `cards`, `columns`, `fragment` in `/blocks/`.
3. Map `nav.md` in `fstab.yaml` as the global nav source.
4. Map `footer.md` as the global footer source.
5. Configure `metadata` rendering in the page template.
