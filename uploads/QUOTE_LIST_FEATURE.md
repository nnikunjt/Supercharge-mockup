# Quote List Feature — Public Shopify App Configuration

## 1. Overview

This document describes the **Quote List** ("Your Saved Quotes") storefront
feature for the B2B Quote App as it prepares to ship as a **public Shopify
app on [Gadget.dev](https://gadget.dev)** (the migration target defined in
`MIGRATION_PLAN.md`). This is the customer-facing page that lists all of a
buyer's saved quotes, with the same admin/customer access split that already
exists in the API.

Because the app installs across many merchants, every label, column, action,
and visual element of this list must be **merchant-configurable** from the
embedded admin UI, and the list itself must render as a **Shopify theme app
extension / app block** so merchants can place it on any page of their
storefront via the Shopify theme editor.

This document complements [`QUOTE_PDF_FEATURE.md`](./QUOTE_PDF_FEATURE.md) —
shared concerns (theme color, date format, currency format) are sourced from
the **same `appSettings` Gadget model record** so the storefront list and
the PDF stay visually consistent.

> All implementation references use Gadget's conventions (`api/actions/`,
> `api/routes/`, `api/models/`, `lib/`, `web/blocks/`) — not Laravel.

---

## 2. Reference Layout (from sample)

The supplied sample (ShopHoverTech "Your Saved Quotes") defines the canonical
layout. Top to bottom:

| # | Section          | Contents                                                                             |
|---|------------------|--------------------------------------------------------------------------------------|
| 1 | Page header      | Page title (e.g., `Your Saved Quotes`)                                               |
| 2 | (Optional) Toolbar | Search box, status filter, date range filter, sort dropdown                        |
| 3 | Table            | Columns: Quote Number, Quote Title, Date Created, Actions (kebab menu)               |
| 4 | Pagination       | Page numbers / load more / cursor                                                    |
| 5 | Empty state      | Shown when buyer has no saved quotes                                                 |

The kebab menu in each row exposes per-quote actions (View, Download PDF,
Email, Duplicate, Delete, etc.).

---

## 3. Configuration Surface

All settings below are stored **per shop** alongside the PDF settings, so
each merchant configures their experience once.

### 3.1 Page Header

| Setting                     | Type        | Default              | Notes                                        |
|-----------------------------|-------------|----------------------|----------------------------------------------|
| `page_title`                | String      | `Your Saved Quotes`  | E.g. `My Quotes`, `Saved Quotations`         |
| `page_title_color`          | Color       | inherits theme       |                                              |
| `page_title_size`           | Enum        | `xl`                 | `lg` \| `xl` \| `2xl` \| `3xl`               |
| `show_subtitle`             | Boolean     | `false`              |                                              |
| `subtitle_text`             | String      | (empty)              | E.g. `Review and manage your saved quotes.`  |
| `show_create_quote_button`  | Boolean     | `false`              | CTA for buyers to start a new quote          |
| `create_quote_button_label` | String      | `Request a Quote`    |                                              |
| `create_quote_button_url`   | String      | (empty)              | Link target                                  |

### 3.2 Table Columns

Each column has three settings: **visibility**, **order**, and **label**.
Merchants drag-to-reorder and toggle in the admin UI.

**Available columns** (defaults shown):

| Key                | Default Visible | Default Label    | Source                                                  |
|--------------------|-----------------|------------------|---------------------------------------------------------|
| `number`           | ✅              | `Quote Number`   | `draftOrder.name` with prefix (e.g. `#D2895`)           |
| `title`            | ✅              | `Quote Title`    | Custom name metafield (`custom.quote_title`)            |
| `date_created`     | ✅              | `Date Created`   | `draftOrder.createdAt`                                  |
| `last_updated`     | ❌              | `Last Updated`   | `draftOrder.updatedAt`                                  |
| `expires_at`       | ❌              | `Expires`        | Expiry metafield or `createdAt + default_expiry_days`   |
| `status`           | ❌              | `Status`         | Pending / Open / Completed / Expired (computed)         |
| `total`            | ❌              | `Total`          | `draftOrder.totalPrice`                                 |
| `item_count`       | ❌              | `Items`          | `draftOrder.lineItems.count`                            |
| `customer`         | ❌ (admin: ✅)  | `Customer`       | `draftOrder.customer.displayName` (admin view only)     |
| `location`         | ❌ (admin: ✅)  | `Location`       | `draftOrder.purchasingEntity.location.name`             |
| `created_by`       | ❌              | `Created By`     | Contact name from `companyContact`                      |
| `actions`          | ✅              | (no header)      | Kebab menu — see 3.4                                    |

### 3.3 Table Styling

| Setting                  | Type    | Default         | Notes                                              |
|--------------------------|---------|-----------------|----------------------------------------------------|
| `header_bg_color`        | Color   | `#5B6770`       | Defaults to `primary_color` from PDF settings      |
| `header_text_color`      | Color   | `#FFFFFF`       |                                                    |
| `row_hover_bg`           | Color   | `#F5F7FA`       |                                                    |
| `zebra_striping`         | Boolean | `false`         |                                                    |
| `row_density`            | Enum    | `comfortable`   | `compact` \| `comfortable` \| `spacious`           |
| `show_grid_lines`        | Boolean | `true`          |                                                    |
| `border_radius`          | Enum    | `md`            | `none` \| `sm` \| `md` \| `lg`                     |
| `mobile_layout`          | Enum    | `card`          | `card` (stacked) \| `scroll` (horizontal scroll)   |

### 3.4 Row Actions (Kebab Menu)

Each action is individually toggleable. The order in the menu mirrors the
order below.

| Action         | Default Visible | Default Label      | Behavior                                                  |
|----------------|-----------------|--------------------|-----------------------------------------------------------|
| `view`         | ✅              | `View`             | Navigate to single-quote detail page                      |
| `download_pdf` | ✅              | `Download PDF`     | Calls `/api/v1/quote/{id}/pdf`                            |
| `email`        | ❌              | `Email Quote`      | Opens email modal, sends via `/api/v1/quote/{id}/email`   |
| `duplicate`    | ❌              | `Duplicate`        | Creates a new draft order from this one's line items      |
| `edit`         | ❌              | `Edit`             | Navigate to edit page                                     |
| `add_to_cart`  | ❌              | `Add to Cart`      | Adds all line items to the storefront cart                |
| `checkout`     | ❌              | `Checkout`         | Uses `draftOrder.invoiceUrl` for B2B checkout             |
| `delete`       | ✅              | `Delete`           | Calls `/api/v1/quote/delete` after confirmation modal     |

| Setting                        | Type    | Default                                            |
|--------------------------------|---------|----------------------------------------------------|
| `confirm_delete`               | Boolean | `true`                                             |
| `delete_confirm_text`          | String  | `Are you sure you want to delete this quote?`      |
| `actions_style`                | Enum    | `kebab` — `kebab` \| `inline_buttons` \| `dropdown` |

### 3.5 Search & Filters

| Setting                  | Type    | Default                | Notes                                          |
|--------------------------|---------|------------------------|------------------------------------------------|
| `show_search`            | Boolean | `false`                | Searches quote number + title                  |
| `search_placeholder`     | String  | `Search quotes…`       |                                                |
| `show_status_filter`     | Boolean | `false`                | Pending / Open / Completed / Expired           |
| `show_date_filter`       | Boolean | `false`                | Date range picker                              |
| `show_customer_filter`   | Boolean | `false`                | Admin view only                                |
| `show_sort_dropdown`     | Boolean | `true`                 |                                                |
| `default_sort`           | Enum    | `date_desc`            | `date_desc` \| `date_asc` \| `number_desc` \| `total_desc` |
| `available_sort_options` | Array   | all                    | Subset of the above                            |

### 3.6 Pagination

| Setting                  | Type    | Default   | Notes                                                       |
|--------------------------|---------|-----------|-------------------------------------------------------------|
| `pagination_style`       | Enum    | `cursor`  | `cursor` (Prev/Next) \| `numbered` \| `load_more` \| `infinite` |
| `page_size`              | Number  | `10`      | Allowed: 10, 25, 50, 100                                    |
| `allow_user_page_size`   | Boolean | `false`   | Lets buyer pick page size                                   |

> The underlying `/api/v1/quotes/list` endpoint already uses cursor-based
> pagination (positive `first` = forward, negative = backward). `numbered` /
> `load_more` / `infinite` styles are UI variants over the same cursor API.

### 3.7 Empty State

| Setting                  | Type    | Default                                                               |
|--------------------------|---------|-----------------------------------------------------------------------|
| `empty_title`            | String  | `No saved quotes yet`                                                 |
| `empty_description`      | String  | `When you save a quote, you'll see it here.`                          |
| `empty_show_cta`         | Boolean | `false`                                                               |
| `empty_cta_label`        | String  | `Browse Products`                                                     |
| `empty_cta_url`          | String  | `/collections/all`                                                    |
| `empty_illustration_url` | String  | (empty)                                                               |

### 3.8 Admin View

When the logged-in customer's `companyLocation.role_name` matches the admin
role (currently `Location admin`), the page automatically switches to admin
mode by calling `/api/v1/quotes/admin/list`.

| Setting                       | Type    | Default                                       |
|-------------------------------|---------|-----------------------------------------------|
| `admin_page_title`            | String  | `All Quotes — {{location_name}}`              |
| `admin_show_customer_column`  | Boolean | `true`                                        |
| `admin_show_location_column`  | Boolean | `true`                                        |
| `admin_role_keyword`          | String  | `Location admin`                              |
| `admin_view_toggle`           | Boolean | `false` — when `true`, admins can switch between "My quotes" and "All quotes" |

### 3.9 Localization

| Setting              | Type    | Default     | Notes                                                    |
|----------------------|---------|-------------|----------------------------------------------------------|
| `language`           | Enum    | `en`        | All labels translatable                                  |
| `date_format`        | Enum    | inherits PDF| Same options as PDF: `MM/DD/YY`, `DD/MM/YY`, etc.        |
| `datetime_format`    | Enum    | `MMM D, YYYY h:mm A` | E.g. `January 22, 2026 11:10 PM`                |
| `currency_format`    | Enum    | inherits PDF|                                                          |
| `relative_dates`     | Boolean | `false`     | When `true`, recent dates show as `2 hours ago`          |

---

## 4. Storefront Integration (Theme App Extension)

The list must ship as a **theme app extension** with an **app block** so
merchants can drop it onto any page in the theme editor — typically:

- `/pages/my-quotes`
- `/account` (as a section)
- A custom page they create

### 4.1 App Block Schema

Settings exposed in the theme editor (subset of section 3 — the rest stay in
the embedded admin):

- `page_title`
- `page_size`
- `pagination_style`
- `show_search`
- `show_status_filter`
- `mobile_layout`
- Color overrides (table header bg/text)

Anything not in the block schema is read from the merchant's app settings,
so merchants get sensible defaults without configuring twice.

### 4.2 Authentication on the Storefront

The block uses **Shopify customer session** (App Proxy or Customer Account
API token) to identify the buyer. The app then issues an API call to the
existing `/api/v1/quotes/list` endpoint with the resolved `customerId` /
`companyLocationId`.

**Gadget setup**:

- Configure App Proxy in Gadget's Shopify connection settings
  (`Settings → Connections → Shopify → App Proxy`) with subpath
  `/apps/b2b-quotes` (or similar).
- The proxied storefront requests arrive at Gadget routes with HMAC and the
  buyer's `logged_in_customer_id` automatically validated by Gadget — no
  manual signature verification needed.
- `lib/auth.js` exposes a `verifyAppProxyRequest(request)` helper that
  rejects any storefront-facing route call lacking a valid proxy context.

Direct browser → `/api/v1/...` calls must reject requests that don't carry
a verified Shopify proxy signature.

---

## 5. Data Sources

| List Field        | Source                                                          |
|-------------------|-----------------------------------------------------------------|
| Quote Number      | `draftOrder.name`, optionally re-prefixed via `quote_number_prefix` |
| Quote Title       | Metafield `custom.quote_title` (set on create/update)           |
| Date Created      | `draftOrder.createdAt`                                          |
| Last Updated      | `draftOrder.updatedAt`                                          |
| Expires           | Metafield `custom.expiry_date` or computed                      |
| Status            | Computed: `completed` if order exists, `expired` if past expiry, else `open` |
| Total             | `draftOrder.totalPrice`                                         |
| Items             | `draftOrder.lineItems.count`                                    |
| Customer (admin)  | `draftOrder.customer.displayName`                               |
| Location (admin)  | `draftOrder.purchasingEntity.location.name`                     |

---

## 6. App Admin UI

Add a **Quote List** tab to the Gadget embedded admin (sibling to PDF
Settings), built with **Shopify App Bridge + Polaris React** and rendered
from `web/routes/admin/list-settings.jsx`:

1. **Page** — title, subtitle, CTA button
2. **Columns** — drag-to-reorder, toggle visibility, edit labels
3. **Style** — colors, density, mobile layout
4. **Actions** — toggle row actions, delete confirmation
5. **Filters & Sort** — search, status filter, date filter, default sort
6. **Empty State** — title, description, CTA
7. **Admin View** — admin column visibility, role keyword
8. **Live Preview** — renders the list with sample data using the same
   `QuoteList` block component that powers the storefront

**Save** calls the `updateAppSettings` Gadget action, persisting the
`listSettings` slice on the shop's `appSettings` record.

---

## 7. API Endpoints (Gadget HTTP routes & actions)

The app is being migrated to **Gadget.dev** (see `MIGRATION_PLAN.md`). Routes
live in `api/routes/` and call Gadget global actions in `api/actions/`. The
existing `quotes/list` and `quotes/admin/list` endpoints are already
scheduled for migration in Phase 3 of the plan; this feature **extends**
them with filters/sort and adds a few new routes.

### 7.1 Existing routes (covered by migration plan)

| Method | Route                       | Gadget File                                  | Status                                           |
|--------|-----------------------------|----------------------------------------------|--------------------------------------------------|
| POST   | `/api/v1/quotes/list`       | `api/routes/api/v1/quotes/list.js`           | Already in migration plan; extend with `filters` & `sort` |
| POST   | `/api/v1/quotes/admin/list` | `api/routes/api/v1/quotes/admin/list.js`     | Already in migration plan; extend with `filters` & `sort` |
| POST   | `/api/v1/quote/delete`      | `api/routes/api/v1/quote/delete.js`          | Already in migration plan; no changes needed     |

### 7.2 New routes for this feature

| Method | Route                              | Gadget File                                          | Action                                                      |
|--------|------------------------------------|------------------------------------------------------|-------------------------------------------------------------|
| POST   | `/api/v1/quote/{id}/duplicate`     | `api/routes/api/v1/quote/[id]/duplicate.js`          | `duplicateQuote` — creates a new draft order from line items |
| POST   | `/api/v1/quote/{id}/add-to-cart`   | `api/routes/api/v1/quote/[id]/add-to-cart.js`        | `addQuoteToCart` — pushes line items to storefront cart     |
| GET    | `/api/v1/list-settings`            | `api/routes/api/v1/list-settings/GET.js`             | `getAppSettings` — returns `listSettings` slice             |
| PUT    | `/api/v1/list-settings`            | `api/routes/api/v1/list-settings/PUT.js`             | `updateAppSettings` — validates and persists                |

> `/api/v1/quote/{id}/pdf` and `/api/v1/quote/{id}/email` are defined in
> `QUOTE_PDF_FEATURE.md`.

### 7.3 Filter / Sort extensions

Extend the existing `listQuotes` / `listAdminQuotes` Gadget actions to accept:

```json
{
  "companyId": "...",
  "customerId": "...",
  "first": 25,
  "after": "cursor",
  "filters": {
    "status": "open",
    "dateFrom": "2026-01-01",
    "dateTo": "2026-01-31",
    "search": "test quote"
  },
  "sort": "date_desc"
}
```

Implementation note: the `getDraftOrdersQuery` builder in
`lib/shopify-queries.js` (per migration plan) is the right place to thread
these through. Shopify's `draftOrders` GraphQL query supports a `query`
parameter (Shopify search syntax) and `sortKey` / `reverse` arguments —
these map cleanly to the new fields.

### 7.4 Gadget Implementation modules

| Gadget File                              | Purpose                                                        |
|------------------------------------------|----------------------------------------------------------------|
| `api/actions/duplicateQuote.js`          | Reuses `mapCreateQuotePayload` to clone an existing quote      |
| `api/actions/addQuoteToCart.js`          | Builds Storefront API cart-add mutation; called via App Proxy  |
| `api/actions/getAppSettings.js`          | Shared with PDF feature — single accessor with defaults        |
| `api/actions/updateAppSettings.js`       | Validates section-3 keys; partial updates allowed              |
| `lib/list-filters.js`                    | Maps `filters` object → Shopify `query` string                 |
| `lib/list-sort.js`                       | Maps `sort` enum → `sortKey` + `reverse` args                  |
| `lib/settings-defaults.js`               | Defaults for every key in Section 3 (shared with PDF feature)  |
| `web/blocks/QuoteList/index.jsx`         | Theme app extension (storefront block)                         |
| `web/routes/admin/list-settings.jsx`     | Embedded admin UI page (App Bridge / Polaris)                  |

---

## 8. Storage Model (Gadget data models)

This feature **shares the `appSettings` model defined in
`QUOTE_PDF_FEATURE.md`** — there is one settings record per shop, holding
both the PDF and list configurations as JSON slices.

| Field             | Gadget Type                          | Notes                                                 |
|-------------------|--------------------------------------|-------------------------------------------------------|
| `shop`            | BelongsTo(`shopifyShop`), unique     | One settings record per connected Shopify shop        |
| `pdfSettings`     | JSON                                 | From `QUOTE_PDF_FEATURE.md` Section 3                 |
| `listSettings`    | JSON                                 | This document, Section 3                              |
| `sharedSettings`  | JSON                                 | See below                                             |
| `logo`            | File                                 | Used by both PDF and (optionally) list page header    |

**Shared settings** (used by both features — single source of truth so the
storefront list and PDF stay visually consistent):

- `primaryColor`, `accentColor`, `fontFamily`
- `dateFormat`, `datetimeFormat`, `currencyFormat`, `language`
- `quoteNumberPrefix`, `defaultExpiryDays`

`api/actions/getAppSettings.js` is the single accessor and is responsible
for merging stored values with `lib/settings-defaults.js` so consumers
never need to handle missing keys.

---

## 9. Open Questions

- [ ] Should buyers be able to **rename** a quote inline from the list, or
      only from the detail page?
- [ ] Bulk actions — does the list need multi-select checkboxes for bulk
      delete / bulk download / bulk email?
- [ ] Should expired quotes be hidden by default, or shown with an
      `Expired` badge?
- [ ] Does the `Add to Cart` action replace the cart, or merge line items
      into the existing cart?
- [ ] For the `Duplicate` action, do we copy the buyer/location/billing
      address as-is, or prompt for confirmation?
- [ ] Customer Account API vs App Proxy — which authentication path are we
      committing to for the storefront block?
- [ ] Do merchants need **multiple list views** (e.g., "Active", "Archive",
      "Drafts") or is a single filterable list sufficient?

---

## 10. Acceptance Criteria

The feature is considered ship-ready when:

- [ ] A merchant can install the app, drop the **Quote List** app block onto
      a theme page via the theme editor, and see a working list with **only
      their colors and labels** changed (no code changes).
- [ ] All section 3 settings are exposed in the embedded admin with a live
      preview.
- [ ] The list correctly resolves admin vs customer view based on
      `companyLocation.role_name`.
- [ ] All row actions in section 3.4 work end-to-end (View, Download PDF,
      Duplicate, Delete, etc.).
- [ ] Search, status filter, date filter, and sort all return correct
      results and respect cursor pagination.
- [ ] Empty state renders correctly when a buyer has no quotes.
- [ ] Mobile layout (`card` mode) renders cleanly on small viewports.
- [ ] Storefront requests are authenticated via App Proxy / Customer
      Account API — no direct unauthenticated browser calls accepted.
- [ ] Settings persist per shop and survive app reinstall.
- [ ] Visual consistency with the PDF: colors, date format, currency, and
      quote number prefix are sourced from a single shared config.
