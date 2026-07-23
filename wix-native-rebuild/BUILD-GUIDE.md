# Native Wix Rebuild — Build Guide

Goal: the dashboard's content lives in real, crawlable Wix page HTML (wrapped in
your actual header/footer) instead of a sandboxed iframe — so Google, ChatGPT,
Claude, and Perplexity can all see and cite it.

## 0. Prerequisite — check this first

This requires **Wix Velo (Dev Mode)**, which needs a Wix plan that supports it
(Wix Studio, or a Premium plan with Dev Mode access — not the free/basic
Connect Domain tier). In the Editor, look for a **Dev Mode** toggle in the top
bar. If you don't see it, check Settings → Premium, or contact Wix support
before going further — everything below assumes Dev Mode is available.

## 1. Data files in this folder

| File | Rows | Use |
|---|---|---|
| `merged_companies.csv` | 17 | **Import this one** — Dashboard cards + Table view combined into a single collection (same 17 companies, so one source of truth instead of two collections to keep in sync) |
| `global_stage_table.csv` | 7 | Supply-chain-stage comparison table on the Global Context page |
| `global_stat_tiles.json` | 4 | The 4 headline stat tiles (China/Allies/US/multiple) — small enough to hand-build as static elements, schema below in case you'd rather import it |
| `companies.csv`, `table_rows.csv` | 17 each | Raw unmerged extracts, kept for reference only — don't import these separately, use `merged_companies.csv` |

All data was parsed directly out of the existing index.html/table.html/global-context.html, so it matches what's currently live.

## 2. Wix Content Collections to create

Go to **CMS (Content Manager)** in the Editor → **Create Collection**.

### Collection: `MagnetCompanies`

Import `merged_companies.csv`. Field mapping (Wix auto-detects most on import,
but set types explicitly where noted):

| Field key | Type | Notes |
|---|---|---|
| `name` | Text | Company name |
| `ticker` | Text | e.g. "NYSE: MP" or "Private" |
| `location` | Text | HQ / site locations |
| `foundedStatus` | Text | "Founded 2017 · Public" |
| `segments` | **Tags** | Split `"Upstream; Midstream; Downstream"` into tags on import — used for filter buttons |
| `dataSegments` | Text | Lowercase space-separated version (`upstream midstream downstream`) — keep as a plain filter key |
| `dataMaturity` | Text | Same pattern for maturity filtering |
| `stageTags` | Text | Full stage-tag list, dashboard card view |
| `headline` | **Rich Text** | Card headline paragraph |
| `fundingLine` | Text | Funding/customer summary line |
| `keyFacts` | **Rich Text** | Pipe-separated key facts — convert to a bulleted list when importing, or leave as-is and split with `\|` in the repeater |
| `detailSectionsJson` | Text | JSON string of `{heading, items[]}` blocks — parse in Velo (`JSON.parse`) to render Government Relationships / Commercial Agreements style sections |
| `analystNote` | Rich Text | Analyst note paragraph |
| `tableMaturityLabel` | Text | e.g. "Commercial / Commissioning" — for the Table view maturity badge |
| `keyStagesAbbrev` | Text | Short stage-pill list, Table view |
| `currentVolumeNumber`/`Unit`/`Note` | Text | Current production volume, Table view |
| `projectedVolumeNumber`/`Unit`/`Note` | Text | Projected volume, Table view |
| `targetYear` | Text | Target year, Table view |
| `currentVolumeSort` | **Number** | Numeric current volume for sorting the Table view |
| `projectedVolumeSort` | **Number** | Numeric projected volume for sorting |

### Collection: `SupplyChainStages`

Import `global_stage_table.csv` — only 7 rows, straightforward 1:1 field mapping
(`stage`, `us_current_volume`, `china_global_position`, `key_us_companies`).

### Stat tiles — skip the collection

`global_stat_tiles.json` has only 4 items that rarely change. Simplest: build
these as 4 static text/number elements directly on the page instead of a
Repeater + Collection. If you'd rather have them editable without touching the
Editor's layout, create a `GlobalStats` collection with fields `label`,
`value`, `note` and import the JSON (convert to CSV first, or add manually —
it's only 4 rows).

## 3. Page-by-page build steps

### Page 1 — Company Dashboard

1. Add a **Repeater** to the page, connected via a **Dataset** to `MagnetCompanies`.
2. Design one repeater item's layout by dragging in text elements bound to
   each field (name, ticker, location, segment badges, stage tags, headline,
   funding line). Use a collapsible container for the detail section (bound
   via Velo, see code below) to replicate the current "Details ▾" expand.
3. Add filter buttons (Upstream/Midstream/Downstream, maturity stages) and a
   search input above the repeater — wire them with Velo (starter code below).
4. Add the "17 companies tracked" counter as a text element updated in Velo
   whenever the filter changes.

### Page 2 — Volume & Stage Table

1. Same `MagnetCompanies` collection, new Dataset instance on this page
   (Wix lets you reuse one collection across multiple page datasets).
2. **Use a Repeater, not Wix's native Table element.** The native Table
   element's column headers aren't individually selectable — you can't give
   them element IDs, and Wix doesn't support click-to-sort-by-header on it
   natively (it's an open feature request, not shipped). Build a Repeater for
   the data rows (columns: Company, Segment, Maturity, Key Stages, Current
   Volume, Projected Volume, Target Year), and a **separate, manually-built
   header row above it** using individual text/button elements — one per
   column. Those are ordinary elements with normal ID fields, unlike the
   native Table's headers.
3. Column-header sorting: bind each header element's click to a `wixData`
   sort call on `currentVolumeSort` / `projectedVolumeSort` (numeric) or the
   text fields (alphabetical) — starter code below.

### Page 3 — US vs. China vs. Allies

1. Build the 3 narrative sections ("The Scale Gap Today," "Global Market
   Share," "The U.S. + Allied Buildout Race") as ordinary Wix text/heading
   sections — this is prose content, not repeating data, so there's no
   benefit to a collection here.
2. Build the 4 stat tiles as static elements (see §2).
3. For "U.S. Capacity by Supply Chain Stage," add a Repeater/Table bound to
   `SupplyChainStages` (only 7 rows — no filtering or sorting needed).

## 4. Starter Velo code

**Finding / setting an element ID:** click the element on the canvas in the
Wix Editor. In the Properties panel there's a small editable ID field near
the top (Wix defaults it to something like `text1` or `header1`) — click it
and rename it to something identifiable, e.g. `companyHeader`. That exact
string (case-sensitive) is what goes inside `$w('#...')` in the code below —
it's the element's ID field, not its visible button/label text.

**Where this code goes:** open the page's code panel (bottom of the Wix
Editor, `</>` icon) — each page has its own code file. Every line that
references `$w('#...')` must be inside the `$w.onReady(() => { ... })` block,
not floating outside it — code outside `onReady` runs before the page's
elements are attached and will silently fail or error. Element IDs
(`#segmentFilter`, `#companyRepeater`, etc.) below are placeholders — rename
to match whatever you actually named your elements per the step above.

This applies to Page 1 (Company Dashboard):

```js
import wixData from 'wix-data';

let segmentFilter = 'all';
let maturityFilter = 'all';
let searchQuery = '';

$w.onReady(function () {
  $w('#companyDataset').onReady(() => {
    updateCount();
  });

  $w('#segmentFilterGroup').onChange((event) => {
    segmentFilter = event.target.value; // 'all' | 'upstream' | 'midstream' | 'downstream'
    applyFilters();
  });

  $w('#maturityFilterGroup').onChange((event) => {
    maturityFilter = event.target.value;
    applyFilters();
  });

  $w('#companySearch').onKeyPress(() => {
    searchQuery = $w('#companySearch').value.toLowerCase();
    applyFilters();
  });
});

function applyFilters() {
  let filter = wixData.filter();

  if (segmentFilter !== 'all') {
    filter = filter.contains('dataSegments', segmentFilter);
  }
  if (maturityFilter !== 'all') {
    filter = filter.contains('dataMaturity', maturityFilter);
  }
  if (searchQuery) {
    filter = filter.contains('name', searchQuery);
  }

  $w('#companyDataset').setFilter(filter).then(updateCount);
}

function updateCount() {
  const total = $w('#companyDataset').getTotalCount();
  $w('#filterCount').text = `Showing ${total} of 17 companies`;
}
```

**Expand/collapse detail section inside each repeater item** (add inside the
same `$w.onReady()` block as the filter code above, on Page 1):

```js
$w.onReady(function () {
  // ...filter code from above also lives in here...

  $w('#companyRepeater').onItemReady(($item, itemData) => {
    $item('#expandBtn').onClick(() => {
      const detail = $item('#detailSection');
      if (detail.collapsed) {
        detail.expand();
        $item('#expandBtn').label = 'Details ▴';
      } else {
        detail.collapse();
        $item('#expandBtn').label = 'Details ▾';
      }
    });

    // Parse the JSON-blob field for Government Relationships / Commercial Agreements style sections
    const sections = JSON.parse(itemData.detailSectionsJson || '[]');
    // Render into a rich-text or repeater element inside the item as needed
  });
});
```

**Column sort — Page 2 (Volume & Stage Table), separate page, separate code
file.** This is the full file for that page — `sortBy` can sit outside
`onReady` since it's just a function definition with no `$w` calls in its own
body until invoked, but the `.onClick()` bindings that call it must be inside:

```js
import wixData from 'wix-data';

let sortCol = 'name';
let sortDir = 'asc';

function sortBy(field) {
  sortDir = (sortCol === field && sortDir === 'asc') ? 'desc' : 'asc';
  sortCol = field;
  let sort = wixData.sort();
  sort = sortDir === 'asc' ? sort.ascending(field) : sort.descending(field);
  $w('#tableDataset').setSort(sort);
}

$w.onReady(function () {
  $w('#currentVolumeHeader').onClick(() => sortBy('currentVolumeSort'));
  $w('#projectedVolumeHeader').onClick(() => sortBy('projectedVolumeSort'));
  $w('#companyHeader').onClick(() => sortBy('name'));
});
```

## 5. Design tokens (match current look)

Apply these in the Wix Design panel per element — there's no shared stylesheet
in the native approach, so each element's color needs to be set individually
(or use Wix's Site Theme colors to define them once and reuse).

| Token | Hex | Use |
|---|---|---|
| Background (deep) | `#031f38` | Page background |
| Background | `#053A5C` | Section/card background |
| Accent | `#1F959A` | Links, active filter state |
| Upstream | `#5eadff` | Segment badge |
| Midstream | `#fbbf24` | Segment badge |
| Downstream | `#a0aec0` | Segment badge |
| Commercial | `#34d399` | Maturity badge |
| Commissioning | `#60a5fa` | Maturity badge |
| Pilot | `#fbbf24` | Maturity badge |
| Development | `#fb923c` | Maturity badge |
| Announced | `#94a3b8` | Maturity badge |
| Paused | `#f87171` | Maturity badge |
| China (comparison) | `#e2552e` | Stat tile |
| Allies (comparison) | `#3b82f6` | Stat tile |
| US (comparison) | `#0d9488` | Stat tile |

## 6. What this guide deliberately simplifies

- The original pages have some layout polish (hover states, transitions,
  responsive breakpoints) that doesn't translate 1:1 to Wix's native element
  system. Expect some visual differences from the current HTML version —
  budget time for design pass in the Editor.
- `detailSectionsJson` keeps variable per-company sections (some companies
  have different detail headings than others) as a JSON blob rather than
  fixed columns, since forcing that into uniform Collection fields would
  create a lot of empty columns. Parse it in Velo as shown above.
- Print/watermark/copyright footer treatment from the iframe version still
  applies conceptually — add the copyright line as a static text element in
  each page's footer area, and set page SEO (title/description/structured
  data) per page as already covered separately.
