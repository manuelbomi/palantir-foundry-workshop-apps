# Palantir Workshop — Engineering Operational Applications on the Ontology

**A Forward-Deployed Engineer's field guide to building operational, decision-ready apps with Foundry Workshop**

[![Platform](https://img.shields.io/badge/Platform-Palantir%20Foundry-1a1a2e?style=flat-square)](https://www.palantir.com/platforms/foundry/)
[![Tool](https://img.shields.io/badge/Tool-Workshop-0f6fde?style=flat-square)]()
[![Domain](https://img.shields.io/badge/Domain-Agnostic-6a1b9a?style=flat-square)]()
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

> Part 3 of a 4-part series. 📦 Part 1 — [Data Pipelines](https://github.com/manuelbomi/palantir-foundry-data-pipelines) → 🧠 Part 2 — [Ontology](https://github.com/manuelbomi/palantir-foundry-ontology) → 🖥️ **Part 3 — Workshop Apps** (this repo) → 🧭 Part 4 — [The FDE Playbook](https://github.com/manuelbomi/palantir-foundry-fde-playbook) (capstone)

---

## Table of Contents

1. [From Data Model to Decision-Making Tool](#from-data-model-to-decision-making-tool)
2. [What Workshop Actually Is](#what-workshop-actually-is)
3. [What We're Building](#what-were-building)
4. [Walkthrough: Building the Fulfillment Dashboard](#walkthrough-building-the-fulfillment-dashboard)
5. [Design Decisions an FDE Has to Defend](#design-decisions-an-fde-has-to-defend)
6. [Why This Generalizes Far Beyond Order Fulfillment](#why-this-generalizes-far-beyond-order-fulfillment)
7. [Why Foundry's Approach to App-Building Is Different](#why-foundrys-approach-to-app-building-is-different)
8. [How to Reuse This Pattern in Other Domains](#how-to-reuse-this-pattern-in-other-domains)
9. [Repo Contents](#repo-contents)
10. [Related Repositories](#related-repositories)

---

## From Data Model to Decision-Making Tool

By the end of [Part 2](https://github.com/manuelbomi/palantir-foundry-ontology), the Ontology has a governed, queryable `Order` Object Type. That solves the *modeling* problem. It still doesn't solve the fulfillment manager's actual, daily problem: **they need a screen they can open every morning that shows what's overdue, what's unassigned, and lets them act on it** — not a data model, an application.

This is the gap **Workshop** is built to close: turning Ontology Objects into a real, interactive, decision-ready application without writing a line of frontend code.

## What Workshop Actually Is

[Workshop](https://www.palantir.com/docs/foundry/workshop/overview) is Foundry's low-code application builder for rapidly assembling operational tools directly on top of Ontology Objects. Instead of hand-rolling a React app that calls a REST API that queries a database, a builder assembles the same outcome from **widgets** (tables, filters, charts, buttons, property panels) wired to **object sets** (queries over Ontology Objects) and **variables** (the state that connects widgets together) — all inside the browser, all governed by the same permissions as the rest of the platform.

## What We're Building

A single-page **Fulfillment Dashboard** for the `Order` Object modeled in Part 2:

- A sortable, filterable **table of every order**
- A **filter panel** the fulfillment manager can use to slice by item, status, assignee, and more
- An **object-details panel** that shows every property of whichever order is selected
- A **Pie Chart** breaking down orders by status (`assigned` / `closed` / `open`)
- A **Bar Chart** showing order volume by days-until-due, segmented by status — the view that visually surfaces what's at risk

```
Order (Ontology Object) ──► Object Set variable ──► Object Table ──┐
                                    │                                ├─► one interactive dashboard
                                    ├─► Filter List ─► Filtered set ─┤     (Workshop module)
                                    ├─► Pie Chart (by Status)        │
                                    └─► Bar Chart (Days Until Due)  ─┘
```

## Walkthrough: Building the Fulfillment Dashboard

*All 47 screenshots below were recovered directly from the source tutorial document and restored here in build order, so this walkthrough is complete end-to-end — including the widgets, filters, and chart configuration steps that aren't always exported as standalone images.*

### 1. Start the module

Every Workshop module starts as a blank page; the fulfillment dashboard is assembled section by section from here.

![Workshop overview](images/01-introduction-workshop-overview.png)

### 2. Build the Object Table — the dashboard's backbone

The **Object Table** widget is added first, since almost everything else on the page (filters, details panel, charts) will end up reading from the same underlying object set.

![Add Object Table widget](images/02-add-object-table-widget.png)
![Select Order object set](images/03-select-order-object-set.png)
![Rename Order object set variable](images/04-rename-order-object-set-variable.png)

Every property from the `Order` Object is added to the table, sorted by `Item Name`, with the default `Title` column relabeled to something a fulfillment manager would actually recognize.

![Sort by Item Name](images/05-sort-by-item-name.png)
![Rename Title property to Item Name](images/06-rename-title-property-to-item-name.png)
![Rename Title property, confirm](images/07-rename-title-property-confirm.png)
![Configure section dimensions, Flex 2](images/08-configure-section-dimensions-flex-2.png)

### 3. Build the Filter panel — and wire it to the table

This is the step that's easy to skip past in a demo but is the crux of the whole exercise: **adding a Filter List widget does nothing on its own.** A filter has to be explicitly composed into a *new* object set variable, and that new variable — not the original one — has to be plugged back into the Object Table. This decoupling (filters produce a derived object set; widgets consume object set variables) is exactly what lets the same filtered result simultaneously drive the table, the pie chart, and the bar chart later.

![Add Filter widget](images/09-add-filter-widget.png)
![Add Filter List widget](images/10-add-filter-list-widget.png)
![Select object set for filter](images/11-select-object-set-for-filter.png)
![Search and add Item Name filter](images/12-search-and-add-item-name-filter.png)
![Change filter component to multi-select](images/13-change-filter-component-to-multiselect.png)
![Allow users to add/remove filters](images/14-allow-users-to-add-remove-filters.png)
![Rename filter output to Order Filters](images/15-rename-filter-output-to-order-filters.png)
![Create object set definition variable](images/16-create-object-set-definition-variable.png)
![Select Order Filter as variable filter](images/17-select-order-filter-as-variable-filter.png)
![Replace Object Table input with filtered orders](images/18-replace-object-table-input-with-filtered-orders.png)

With the wiring in place, the filter panel gets its final styling pass — background, icon, collapsibility, header format — so it reads as a purpose-built control panel rather than a raw widget.

![Set filter background color, Light Gray 4](images/19-set-filter-background-color-light-gray-4.png)
![Add filter icon to section](images/20-add-filter-icon-to-section.png)
![Configure filter header format, Contained](images/21-configure-filter-header-format-contained.png)

### 4. Build the Object Details panel

Selecting a row in the table should show everything about that one order — the pattern behind every "master-detail" operational screen ever built.

![Split section, new section on right](images/22-split-section-new-section-on-right.png)
![Toggle off title, contained header](images/23-toggle-off-title-contained-header.png)
![Add Object Set Title widget](images/24-add-object-set-title-widget.png)
![Set input object set to Active Object](images/25-set-input-object-set-to-active-object.png)

A **Button Group** is placed here as a deliberate placeholder — the eventual home for the write-back actions (reassign, close order) that the fulfillment team needs, which are built out in the [capstone repo](https://github.com/manuelbomi/palantir-foundry-fde-playbook) once the Ontology Actions from Part 2 are formalized.

![Add Button Group widget](images/26-add-button-group-widget.png)
![Configure Button Group row height](images/27-configure-button-group-row-height.png)

Underneath the button, a **Property List** shows the full detail of whichever order is currently selected in the table above.

![Add Property List widget](images/28-add-property-list-widget.png)
![Search and add Item Name property](images/29-search-and-add-item-name-property.png)
![Set Property List number of columns](images/30-set-property-list-number-of-columns.png)
![Set Property List display, Flex 1](images/31-set-property-list-display-flex-1.png)

### 5. Polish the layout

A few renames and header adjustments turn a working prototype into something that reads as a finished product — the difference between "a demo" and "a tool the team will actually adopt."

![Rename Object Table section, All Orders](images/32-rename-object-table-section-all-orders.png)
![Remove duplicate section header](images/33-remove-duplicate-section-header.png)
![Split section, new section below](images/34-split-section-new-section-below.png)
![Set section padding, Regular](images/35-set-section-padding-regular.png)
![Add Charts section, set layout columns](images/36-add-charts-section-set-layout-columns.png)

### 6. Add the Pie Chart — orders by status

The chart is wired to the **same `Filtered Orders` variable** that drives the table, not a fresh, unfiltered query — meaning every chart on the page respects whatever the fulfillment manager currently has filtered. This single design choice is what turns a set of independent widgets into one coherent, filterable dashboard.

![Add Pie Chart widget](images/37-add-pie-chart-widget.png)
![Configure Pie Chart, Group By Status](images/38-configure-pie-chart-group-by-status.png)

### 7. Add the Bar Chart — the "what's at risk" view

The bar chart plots order count against **Days Until Due**, segmented by **Status** — the single visualization a fulfillment manager will actually check first each morning, because it's the one that visually surfaces orders sliding toward being overdue.

![Add Chart: XY (bar chart) widget](images/39-add-chart-xy-bar-chart-widget.png)
![Open Plot Layers panel](images/40-open-plot-layers-panel.png)
![Configure bar layer data input and axes](images/41-configure-bar-layer-data-input-and-axes.png)
![Set bar orientation, Vertical](images/42-set-bar-orientation-vertical.png)

### 8. Final layout pass

Charts get moved above the fold — the summary view a manager should see *before* scrolling into row-level detail — and the table's column order is tuned so the fields fulfillment cares about most (`Customer Name`, `Item Name`, `Status`, `Assignee`, `Days Until Due`) appear first, left to right.

![Move Charts section to top](images/43-move-charts-section-to-top.png)
![Configure Object Table column order](images/44-configure-object-table-column-order.png)
![Reorder Object Table key columns](images/45-reorder-object-table-key-columns.png)

### 9. Save, publish, and view the result

![Save and publish Workshop module](images/46-save-and-publish-workshop-module.png)

The finished module: a live, filterable, chart-backed operational dashboard, reading directly off the Ontology `Order` Object — no custom frontend code, no separate database, no BI tool licensing, built by one person in one sitting.

![Completed Workshop dashboard view](images/47-completed-workshop-dashboard-view.png)

## Design Decisions an FDE Has to Defend

- **Why decouple the filter into its own object set variable instead of filtering the table directly?** Because the same filtered result needs to drive the table *and* both charts simultaneously. Filtering each widget independently would let the table and the charts silently disagree with each other — exactly the "which number do we trust" problem this whole series set out to eliminate in Part 1.
- **Why is the Button Group added now but left unconfigured?** Because the write-back Action it will trigger (reassigning an order) hasn't been formalized in the Ontology yet. Wiring a button to an action that doesn't exist yet produces a button that either does nothing or does the wrong thing — worse than no button at all. The right sequencing is: model the Object (Part 2), build the read surface (this repo), *then* formalize the Action once real usage clarifies exactly what "reassign" should mean (permissions, audit trail, side effects) — covered in the [capstone](https://github.com/manuelbomi/palantir-foundry-fde-playbook).
- **Why put the charts above the table instead of below?** Dashboard layout should mirror decision order, not build order. A manager scans the summary (what's the overall health of the order book?) before drilling into individual rows — so the charts, once built, are deliberately promoted above the fold rather than left wherever they happened to be added.
- **Why reorder table columns to `Customer Name, Item Name, Status, Assignee, Days Until Due`?** Column order in an operational table is a UX decision, not a cosmetic one — the leftmost columns are what a user scans first, so they should be the fields that answer "do I need to act on this row right now?"

## Why This Generalizes Far Beyond Order Fulfillment

Swap the noun and this exact widget pattern rebuilds an operational cockpit in any domain:

- **Healthcare operations:** a table of `Patient Encounter` Objects, filtered by unit and acuity, with a status pie chart and a wait-time bar chart — a bed-management or triage dashboard
- **Manufacturing:** a table of `Asset` Objects, filtered by plant and line, with a health-status pie chart and a maintenance-due bar chart — a predictive-maintenance cockpit
- **Financial services:** a table of `Case` Objects (fraud/AML alerts), filtered by risk tier, with a disposition pie chart and an age-of-case bar chart — an investigator's queue
- **Logistics:** a table of `Shipment` Objects, filtered by carrier and lane, with an on-time-status pie chart and a days-to-ETA bar chart — a control-tower view
- **Government/public sector:** a table of `Case` Objects (benefits, permits), filtered by program, with a disposition pie chart and an aging bar chart — a caseworker queue

The specific properties change; the *pattern* — object table + decoupled filter + status pie chart + risk/aging bar chart + a details panel with a placeholder for write-back actions — is the same 90% solution an FDE reaches for as the starting point of nearly every operational app engagement.

## Why Foundry's Approach to App-Building Is Different

- **The app and the data model are the same thing.** There's no separate app database to keep in sync with the Ontology — every widget queries live Ontology Objects, so the app is never stale relative to the semantic layer built in Part 2.
- **Low-code doesn't mean low-ceiling.** Workshop's widget/variable model handles this entire dashboard with zero custom code, but the same canvas supports custom TypeScript widgets and full write-back Actions when a use case genuinely needs them — an FDE doesn't hit a wall and have to "rewrite it properly" later.
- **Governance rides along for free.** Because the app reads and (eventually) writes through Ontology Objects and Actions, every permission, audit log entry, and data-quality guarantee already established in Parts 1–2 applies automatically inside the app — nothing has to be re-implemented in application code.
- **This is the same substrate AIP builds on.** An AI agent that later needs to "show me overdue orders for Region X" is querying the identical `Order` Object Set this dashboard is built on — the operational app and a future AI copilot share one governed foundation instead of two divergent ones.

## How to Reuse This Pattern in Other Domains

1. **Start with the object table.** It's the widget everything else — filters, charts, details — will end up referencing.
2. **Always route filters through a derived object set variable**, never filter individual widgets independently, so every visualization on the page stays in sync.
3. **Reserve a details panel with a placeholder action slot early**, but don't wire real write-back logic until the underlying Ontology Action is actually modeled — sequence matters.
4. **Pick exactly one summary chart and one risk/aging chart** as the default "what should I look at first" view, and place them above the row-level table.
5. **Order table columns by decision relevance**, not by the order columns happen to exist in the source dataset.

## Repo Contents

```
├── images/     # all 47 screenshots recovered from the source tutorial, in build order
└── README.md
```

> This module operates on the `Order` Object Type modeled in [Part 2](https://github.com/manuelbomi/palantir-foundry-ontology), which in turn is backed by the `all_orders` dataset from [Part 1](https://github.com/manuelbomi/palantir-foundry-data-pipelines).

## Related Repositories

| Part | Repo | Focus |
|---|---|---|
| 1 | [palantir-foundry-data-pipelines](https://github.com/manuelbomi/palantir-foundry-data-pipelines) | Data integration with Pipeline Builder |
| 2 | [palantir-foundry-ontology](https://github.com/manuelbomi/palantir-foundry-ontology) | Modeling the unified dataset as a live Ontology Object |
| 3 | **palantir-foundry-workshop-apps** (this repo) | Turning the Ontology into an operational application |
| 4 | [palantir-foundry-fde-playbook](https://github.com/manuelbomi/palantir-foundry-fde-playbook) | The end-to-end case study and generalized FDE playbook |

---

*Author: [manuelbomi](https://github.com/manuelbomi) — built while working through Palantir's official Foundry tutorial content, reframed around the kinds of problems a Forward-Deployed Engineer solves in the field.*
