# Day 26: Dashboard Design & Publishing — Complete Notes

## The Mindset

> Design isn't decoration. A messy dashboard makes a manager distrust your NUMBERS — even when they're right. Clean layout signals "this person is careful." That perception is worth as much as the analysis.

Working visuals scattered on a canvas = a workshop. A portfolio piece = a showroom.

---

## Part 1: Layout Hierarchy

Eyes land **top-left first**, then scan down. Structure follows importance (newspaper logic — headline up top, details below):

| Zone | Contents | Why |
|---|---|---|
| **Top strip** | Title + KPI cards (Total Revenue, Orders, Premium Revenue, AOV) | Headline numbers, seen first |
| **Top-right corner** | Navigation (bookmark buttons) | Controls out of the data's way |
| **Middle** | The core story (trend line — "is revenue growing?") | Most important business question gets prominence + size |
| **Bottom** | Supporting breakdowns (bar, donut, scatter) | Detail for those who scan further |

**The judgment call:** the visual answering the CORE business question deserves the most visual weight. "Is revenue growing over time?" usually beats category breakdowns for the prime spot. But layout is arguable — what matters is having a REASON. An analyst makes deliberate choices and can defend them ("I gave trend and category equal weight because...").

**Design decisions are defensible, not random.** If an interviewer asks "why is this here?", you need an answer.

---

## Part 2: Titles — Business Language, Not Machine Language

Machine titles describe FIELDS. Business titles describe INSIGHTS.

| Machine (default) | Business (rewritten) |
|---|---|
| Total Revenue by Year-Month | Revenue Trend Over Time |
| Count of order_id by order_status | Order Status Breakdown |
| Total Revenue by product_category_name | Top Revenue Categories |
| Average of product_weight_g and Average of freight_value by... | **Heavier Products Cost More to Ship** |

The scatter title is the senior-level move: it **states the finding**. The viewer knows the takeaway before reading a single dot.

**How:** click visual → Format pane → General → Title → Text.

**Main dashboard title:** Insert → Text box → large (28–32pt), bold, top of page. Ours: *"Olist Sales & Fulfillment Overview (2016–2018)"* — scope + date range reads professional.

---

## Part 3: The Final Polish Pass

1. **English labels** — joined product_category_name_translation (Day 19 merge skill reused) → health_beauty, watches_gifts instead of Portuguese. Highest-impact fix: any manager can now read it.
2. **Styled buttons** — default grey boxes → navy fill, white text. Intentional, not leftover.
3. **Consistent palette** — one coordinated scheme (navy/purple/black) instead of default-blue-everything. Reads as ONE designed thing.

---

## Part 4: Publishing Reality (Free vs Pro)

**The limitation:** publishing to Power BI Service with shareable links requires a Pro license (~$10–14/month). Free accounts: personal workspace only, no sharing, no Publish-to-web embed codes.

**The honest take: you don't need Pro for a portfolio.** A Pro link often requires the VIEWER to sign in too — friction that kills portfolio views. Free alternatives are arguably better:

| Method | Value |
|---|---|
| **Screenshot in GitHub README** | HIGHEST ROI — recruiters look at READMEs first, zero friction |
| **Export → PDF** | Full layout preserved, anyone opens it |
| **Commit the .pbix file** | Technical reviewers with free Desktop can open it LIVE — bookmarks, drill-through, everything works |
| README section explaining the dashboard | What it shows, tools used, insights found |

**When Pro matters (interview knowledge, not action):** in a company, you'd publish to a Service **Workspace**, set **scheduled refresh**, share via org Pro/Premium capacity. Know the concept; the company provides the license.

---

## One-line takeaway
> Design turns correct analysis into TRUSTED analysis — English labels, insight-driven titles, and a consistent palette signal care, and care is what a hiring manager is really reading.
