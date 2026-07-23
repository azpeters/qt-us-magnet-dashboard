> **SUPERSEDED (2026-07-23):** the dashboard is now hosted externally at
> dashboard.qtmagneticsolutions.com instead of embedded in Wix pages. The
> JSON-LD blocks below are already baked directly into index.html and
> global-context.html's `<head>` — no Wix Structured Data panel needed.
> Kept for reference only.

# JSON-LD Structured Data — Wix Setup

## Where it goes in Wix

This is separate from the HTML you paste into the Embed widget — Wix has a
dedicated field for this per page:

1. In the Wix Editor, select the page (or open it from the Pages panel).
2. Click the **SEO** button for that page (usually a small "SEO (Google)"
   icon/panel, or Pages panel → page settings → **SEO**).
3. Go to the **Advanced** tab → **Structured Data markup**.
4. Click **Add New Markup**, give it a name, paste the JSON-LD below into the
   code box, then **Apply** and **Save**.

Notes:
- Wix only accepts **JSON-LD** format (not Microdata/RDFa).
- Each markup must be **under 7,000 characters** — all three below are well
  under that.
- Do this once per page, matching the JSON-LD to that page's own content.
- After you publish each page, replace the placeholder `"url"` value below
  with the real published Wix URL for that page.

---

## Page 1 — Company Dashboard (index.html)

```json
{
  "@context": "https://schema.org",
  "@type": "Dataset",
  "name": "U.S. Domestic Rare Earth Permanent Magnet Production Dashboard",
  "description": "A comprehensive reference tracking 17 companies involved in U.S. domestic rare earth permanent magnet production, spanning the full supply chain from rare earth mining and beneficiation through separation, metallization, alloy manufacturing, and finished NdFeB and SmCo magnet production. Includes company stage maturity ratings, funding data, government contracts, and key commercial agreements. Updated July 2026.",
  "url": "REPLACE-WITH-WIX-DASHBOARD-URL",
  "creator": {
    "@type": "Organization",
    "name": "QT Magnetic Solutions",
    "url": "https://qtmagneticsolutions.com"
  },
  "dateModified": "2026-07-15",
  "temporalCoverage": "2026",
  "spatialCoverage": "United States",
  "keywords": ["rare earth magnets", "NdFeB", "neodymium magnets", "rare earth supply chain", "domestic manufacturing", "critical minerals", "DFARS compliance", "DoD supply chain"],
  "copyrightNotice": "© 2026 QT Magnetic Solutions. All rights reserved.",
  "copyrightHolder": {
    "@type": "Organization",
    "name": "QT Magnetic Solutions",
    "url": "https://qtmagneticsolutions.com"
  }
}
```

---

## Page 2 — Volume & Stage Table (table.html)

```json
{
  "@context": "https://schema.org",
  "@type": "Dataset",
  "name": "U.S. Rare Earth Magnet Supply Chain — Volume & Stage Table",
  "description": "Sortable table of all 17 U.S. rare earth magnet supply chain companies by stage maturity, current production volume, and projected future capacity. Updated July 2026.",
  "url": "REPLACE-WITH-WIX-TABLE-URL",
  "creator": {
    "@type": "Organization",
    "name": "QT Magnetic Solutions",
    "url": "https://qtmagneticsolutions.com"
  },
  "dateModified": "2026-07-15",
  "temporalCoverage": "2026",
  "spatialCoverage": "United States",
  "keywords": ["rare earth magnet production volume", "NdFeB magnet capacity", "US magnet manufacturing stages", "critical minerals supply chain"],
  "copyrightNotice": "© 2026 QT Magnetic Solutions. All rights reserved.",
  "copyrightHolder": {
    "@type": "Organization",
    "name": "QT Magnetic Solutions",
    "url": "https://qtmagneticsolutions.com"
  }
}
```

---

## Page 3 — US vs. China vs. Allies (global-context.html)

```json
{
  "@context": "https://schema.org",
  "@type": "Dataset",
  "name": "U.S. vs. Allies vs. China — Rare Earth Magnet Production",
  "description": "How U.S. rare earth magnet production compares to China and to allied nations (Japan, EU, South Korea, Australia) — current market share, the 2026-2030 buildout trajectory, and a stage-by-stage view of the supply chain.",
  "url": "REPLACE-WITH-WIX-GLOBAL-CONTEXT-URL",
  "creator": {
    "@type": "Organization",
    "name": "QT Magnetic Solutions",
    "url": "https://qtmagneticsolutions.com"
  },
  "dateModified": "2026-07-15",
  "temporalCoverage": "2026",
  "spatialCoverage": "Global",
  "keywords": ["China rare earth magnet market share", "rare earth supply chain allies", "US rare earth independence", "critical minerals geopolitics"],
  "copyrightNotice": "© 2026 QT Magnetic Solutions. All rights reserved.",
  "copyrightHolder": {
    "@type": "Organization",
    "name": "QT Magnetic Solutions",
    "url": "https://qtmagneticsolutions.com"
  }
}
```
