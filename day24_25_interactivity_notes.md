# Days 24–25: Interactivity — Complete Notes

## The Big Picture

Interactivity is what turns "a page of charts" into "a tool a manager can explore." Three tools, in order of importance:

| Tool | What it does |
|---|---|
| **Cross-filtering** | Charts filter each other on click |
| **Drill-through** | Right-click a data point → jump to a dedicated detail page |
| **Bookmarks** | Saved page states, switchable via buttons |

---

## Part 1: Cross-Filtering (Day 24)

**On by default.** Click any data point (e.g., a bar) → every other visual on the page filters to that selection. Click again to deselect.

> Every visual is a clickable filter. The user explores by clicking, not configuring.

### Edit Interactions — controlling HOW visuals react
Select a visual → Format → **Edit interactions**. Each other visual shows 3 icons:

| Icon | Effect |
|---|---|
| **Filter** (funnel) | Target fully recalculates to selection only |
| **Highlight** | Target keeps full data visible, dims non-matching, brightens matching |
| **None** (⊘) | Target ignores the click |

**Filter vs Highlight for part-to-whole (donut):**
- **Highlight** preserves context — you see the selection AGAINST the full total (faded ring + bright portion). Good for exploration.
- **Filter** recalculates slices to 100% within the selection. Good for focus ("within this category, what's the split?").
- Neither is wrong — know WHY you chose. (We kept Highlight: context wins for this dashboard.)

### THE BUG WE DEBUGGED — filter direction
**Symptom:** clicking the bar chart (products-based) filtered everything EXCEPT the donut (orders-based).

**Diagnosis:** filters flow from the ONE side to the MANY side of a relationship.
- products (1) → order_items (many) ✅ flows
- orders (1) → order_items (many) ✅ flows
- But order_items (many) → orders (1) ❌ BLOCKED — filter can't climb many-to-one

So the click traveled products → order_items and STOPPED. Never reached orders → donut ignored it.

**Fix:** double-click the order_items ↔ orders relationship line → Cross-filter direction: **Both** (bidirectional).

> ⚠️ Bidirectional is a deliberate tool, not a default — can create ambiguity in complex models. Use when needed, know why.

---

## Part 2: Drill-Through (Day 24)

**What it solves:** cross-filtering crowds detail into one page. Drill-through gives a **dedicated, uncluttered detail page** — overview stays clean, detail appears on demand.

> Summary page = the "what." Drill-through page = the "why/details." Separation of altitude.

### Build steps
1. New page (e.g., "Category Details")
2. On that page: Visualizations pane → **Drill through** section → drag the field (product_category_name) into "Add drill-through fields here"
3. Power BI auto-creates a back-arrow button (top-left)
4. Build detail visuals: revenue card, trend line, status table — they auto-filter to whatever was drilled from
5. "Keep all filters" = On (default, good — carries source-page filters over)

### How the user triggers it
**RIGHT-CLICK** a data point (e.g., the beleza_saude bar) → **Drill through** → page name.

**Traps we hit:**
- Hover shows a TOOLTIP (different feature) — not drill-through
- Left-click cross-filters — not drill-through
- Right-click is the gesture

**Why the detail page filters itself:** the data point you right-click becomes the filter context for the entire destination page. One detail page serves ALL categories dynamically.

---

## Part 3: Bookmarks + Buttons (Day 25)

**Mental model:** a bookmark is a **photo of the page's state** — active filters/slicers, visible/hidden visuals, selections. One click restores the photo exactly.

### Build a year-toggle (2017/2018)
1. View tab → check **Bookmarks** (and **Selection**) panes
2. Set slicer to 2017 → Bookmarks pane → **Add** → rename "2017 view"
3. Set slicer to 2018 → **Add** → rename "2018 view"
4. Insert → **Buttons** → Blank. Button text: Style → Text → On → "2017" (or use Title → On)
5. Button **Action** → On → Type: **Bookmark** → select "2017 view"
6. Repeat for 2018 (copy-paste the first button, change text + bookmark)

### Testing quirk
- In Desktop (edit mode): **Ctrl + Click** fires the button (plain click just selects it for editing)
- Published/reading mode: normal click works

### THE "SAVES TOO MUCH" TRAP
A bookmark photographs EVERYTHING active at save time. If a stray cross-filter (e.g., beleza_saude clicked) was active when you saved "2017 view," the button will re-apply that hidden filter every time — forever.

**Prevention:** before saving any bookmark → click empty canvas (clears selections) → set ONLY the state you want → then Add.

**Fine control:** each bookmark's "..." menu → toggle what it captures (Data / Display / Current page).

---

## One-line takeaways
- Day 24: Filters travel one-side → many-side; when a click can't reach a visual, check the relationship direction — and choose Filter vs Highlight deliberately.
- Day 25: A bookmark is a photo of the entire page state — clear stray selections before saving, or it captures filters you never meant to keep.
