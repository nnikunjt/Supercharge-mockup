# Quote PDF Feature — Public Shopify App Configuration

## 1. Overview

This document describes the **Quote PDF** feature for the B2B Quote App as it
prepares to ship as a **public Shopify app on [Gadget.dev](https://gadget.dev)**
(the migration target defined in `MIGRATION_PLAN.md`). Once installed, every
merchant will need to render quotes (Shopify Draft Orders) as a branded PDF
document that can be downloaded by buyers or attached to emails.

Because the same codebase will run for many different merchants, every visual
and behavioral element of the PDF must be **merchant-configurable** from the
embedded app admin UI. This document defines:

1. The structure of the PDF (matching the existing Hovertech sample).
2. The full list of configuration options the app should expose to merchants.
3. The data sources powering each section.
4. Recommended defaults, Gadget data models, Gadget actions/routes, and
   rendering pipeline.

> All implementation references in this document use Gadget's conventions
> (`api/actions/`, `api/routes/`, `api/models/`, `lib/`, `web/`) — not
> Laravel. The current Laravel codebase is being phased out per
> `MIGRATION_PLAN.md`.

---

## 2. Reference Layout (from sample)

The supplied sample PDF (Hovertech) defines the canonical layout. The PDF is a
single-page (multi-page when line items overflow) document with the following
sections, top to bottom:

| # | Section          | Contents                                                                 |
|---|------------------|--------------------------------------------------------------------------|
| 1 | Header           | Document title (left), Merchant logo (right)                             |
| 2 | Sender block     | Merchant name, address, phone, email — left-aligned                      |
| 3 | Quote meta       | Quote #, Date, Expires — right-aligned                                   |
| 4 | Address row      | Three columns: Bill To, Ship To, Shipping Method                         |
| 5 | Line items table | Columns: SKU, Description, MSRP, Your Price, Qty, UOM, Ext. MSRP, Ext. Your Price |
| 6 | Totals           | Subtotal, Estimated Shipping (+ note), Total                             |
| 7 | Footer           | Optional disclaimer / terms / page numbers                               |

---

## 3. Configuration Surface

All settings below are stored **per shop** (keyed on `shop_domain` /
Shopify shop ID) so each merchant sees their own configured PDF.

### 3.1 Branding & Theme

| Setting              | Type         | Default            | Notes                                                  |
|----------------------|--------------|--------------------|--------------------------------------------------------|
| `logo`               | File upload  | —                  | PNG/JPG/SVG, recommended 600×200 px                    |
| `logo_position`      | Enum         | `right`            | `left` \| `center` \| `right`                          |
| `logo_max_height_px` | Number       | `60`               | Cap to keep header tidy                                |
| `primary_color`      | Color (hex)  | `#5B6770`          | Used for table header band, total label                |
| `accent_color`       | Color (hex)  | `#9E1B32`          | Used for logo line, links                              |
| `font_family`        | Enum         | `Inter`            | Whitelisted: Inter, Roboto, Lato, Helvetica, Times     |
| `base_font_size_pt`  | Number       | `10`               | 8–12 range                                             |
| `page_size`          | Enum         | `Letter`           | `Letter` \| `A4` \| `Legal`                            |
| `page_margin_mm`     | Number       | `15`               | Uniform margin                                         |
| `watermark_text`     | String       | (empty)            | Optional, e.g. `DRAFT`, `COPY`, `INTERNAL ONLY`        |

### 3.2 Document Header

| Setting                  | Type    | Default     | Notes                                              |
|--------------------------|---------|-------------|----------------------------------------------------|
| `document_title`         | String  | `Quote`     | e.g. `Quotation`, `Estimate`, `Proposal`           |
| `document_title_color`   | Color   | `#1F3B4D`   |                                                    |
| `show_quote_number`      | Boolean | `true`      |                                                    |
| `quote_number_prefix`    | String  | `D`         | Becomes `#D2895` etc.                              |
| `quote_number_label`     | String  | `Quote #`   | Translatable                                       |
| `show_quote_date`        | Boolean | `true`      |                                                    |
| `quote_date_label`       | String  | `Date`      |                                                    |
| `show_expiry_date`       | Boolean | `true`      |                                                    |
| `expiry_date_label`      | String  | `Expires`   |                                                    |
| `default_expiry_days`    | Number  | `90`        | Used when quote has no explicit expiry             |
| `date_format`            | Enum    | `MM/DD/YY`  | `MM/DD/YY` \| `DD/MM/YY` \| `YYYY-MM-DD` \| `DD MMM YYYY` |

### 3.3 Sender Block (Merchant Info)

Pulled by default from the Shopify shop record but overridable per-merchant.

| Setting                    | Type    | Default                  | Notes                                       |
|----------------------------|---------|--------------------------|---------------------------------------------|
| `sender_name`              | String  | shop name from Shopify   | E.g. `Hovertech International`              |
| `sender_address_lines`     | Array   | shop address from Shopify| Multi-line free text                        |
| `sender_phone`             | String  | —                        |                                             |
| `sender_phone_label`       | String  | `Direct`                 | E.g. `Direct`, `Tel`, `Phone`               |
| `sender_email`             | String  | shop email from Shopify  | Rendered as `mailto:` link                  |
| `sender_website`           | String  | —                        | Optional                                    |
| `sender_tax_id`            | String  | —                        | Optional VAT/EIN                            |
| `show_phone`               | Boolean | `true`                   |                                             |
| `show_email`               | Boolean | `true`                   |                                             |
| `show_website`             | Boolean | `false`                  |                                             |
| `show_tax_id`              | Boolean | `false`                  |                                             |

### 3.4 Address Row

| Setting                  | Type    | Default                         | Notes                                |
|--------------------------|---------|---------------------------------|--------------------------------------|
| `show_bill_to`           | Boolean | `true`                          |                                      |
| `bill_to_label`          | String  | `Bill To:`                      |                                      |
| `show_ship_to`           | Boolean | `true`                          |                                      |
| `ship_to_label`          | String  | `Ship To:`                      |                                      |
| `show_shipping_method`   | Boolean | `true`                          |                                      |
| `shipping_method_label`  | String  | `Shipping Method`               |                                      |
| `default_shipping_text`  | String  | `Per Order: UPS, Freight`       | Fallback when no method on the quote |

### 3.5 Line Items Table

| Setting                       | Type    | Default                        | Notes                                                           |
|-------------------------------|---------|--------------------------------|-----------------------------------------------------------------|
| `table_header_bg`             | Color   | `#5B6770`                      | Defaults to `primary_color`                                     |
| `table_header_text_color`     | Color   | `#FFFFFF`                      |                                                                 |
| `zebra_striping`              | Boolean | `false`                        |                                                                 |
| `column_visibility`           | Object  | all `true` except product image| Per-column toggle (see below)                                   |
| `column_order`                | Array   | default order                  | Drag-to-reorder                                                 |
| `show_product_image`          | Boolean | `false`                        | Adds thumbnail column                                            |
| `description_source`          | Enum    | `product_title`                | `product_title` \| `variant_title` \| `product+variant` \| `metafield` |
| `uom_metafield_namespace_key` | String  | `custom.unit_of_measure`       | Pulled per variant; falls back to `Ea`                          |
| `default_uom`                 | String  | `Ea`                           |                                                                 |
| `currency_format`             | Enum    | `symbol-prefix`                | `symbol-prefix` (`$1.00`) \| `symbol-suffix` (`1.00 USD`) \| `iso-prefix` (`USD 1.00`) |
| `number_format_locale`        | String  | `en-US`                        | Drives thousand/decimal separators                              |
| `empty_state_text`            | String  | `No line items`                |                                                                 |

**Available columns** (each individually toggleable):

`SKU`, `Image`, `Description`, `MSRP`, `Your Price`, `Discount %`, `Qty`, `UOM`,
`Extended MSRP`, `Extended Your Price`, `Tax`.

### 3.6 Totals Block

| Setting                       | Type    | Default                                            |
|-------------------------------|---------|----------------------------------------------------|
| `show_subtotal`               | Boolean | `true`                                             |
| `subtotal_label`              | String  | `Subtotal`                                         |
| `show_total_msrp`             | Boolean | `false`                                            |
| `show_total_savings`          | Boolean | `false`                                            |
| `savings_label`               | String  | `You Save`                                         |
| `show_estimated_shipping`     | Boolean | `true`                                             |
| `estimated_shipping_label`    | String  | `Estimated Shipping`                               |
| `estimated_shipping_note`     | String  | `Shipping costs are subject to change at checkout` |
| `show_tax`                    | Boolean | `false`                                            |
| `tax_label`                   | String  | `Tax`                                              |
| `show_discount_line`          | Boolean | `false`                                            |
| `discount_label`              | String  | `Discount`                                         |
| `total_label`                 | String  | `Total`                                            |
| `total_emphasis`              | Enum    | `bold`                                             |

### 3.7 Footer

| Setting             | Type    | Default                | Notes                                 |
|---------------------|---------|------------------------|---------------------------------------|
| `footer_text`       | Rich text | (empty)              | Terms, return policy, signature block |
| `show_page_numbers` | Boolean | `true`                 | `Page 1 of N`                         |
| `show_generated_at` | Boolean | `false`                | Timestamp of PDF generation           |
| `signature_block`   | Boolean | `false`                | Adds `Authorized by ____` line        |

### 3.8 Email Delivery

| Setting                   | Type      | Default                                              |
|---------------------------|-----------|------------------------------------------------------|
| `auto_email_on_create`    | Boolean   | `false`                                              |
| `email_subject_template`  | String    | `Your Quote {{quote_number}} from {{shop_name}}`     |
| `email_body_template`     | Rich text | Default template with `{{customer_name}}`, etc.      |
| `from_name`               | String    | shop name                                            |
| `reply_to`                | String    | shop email                                           |
| `bcc_merchant`            | Boolean   | `true`                                               |
| `attach_pdf`              | Boolean   | `true`                                               |

Supported template variables: `{{quote_number}}`, `{{quote_date}}`,
`{{expiry_date}}`, `{{customer_name}}`, `{{customer_company}}`, `{{shop_name}}`,
`{{total}}`, `{{quote_url}}`.

### 3.9 Localization

| Setting              | Type    | Default | Notes                                                     |
|----------------------|---------|---------|-----------------------------------------------------------|
| `language`           | Enum    | `en`    | All labels above are translatable per merchant            |
| `currency_override`  | String  | (empty) | Force a currency code; otherwise uses quote's currency    |
| `rtl_support`        | Boolean | `false` | Mirrors layout for Arabic, Hebrew, etc.                   |

### 3.10 Behavior & Storage

| Setting                  | Type    | Default     | Notes                                                          |
|--------------------------|---------|-------------|----------------------------------------------------------------|
| `regenerate_on_update`   | Boolean | `true`      | Re-render PDF when quote is updated                            |
| `pdf_retention_days`     | Number  | `365`       | After this, regenerate on demand instead of caching            |
| `customer_download_link` | Boolean | `true`      | Expose secure tokenized download URL on the storefront         |
| `link_expiry_hours`      | Number  | `168`       | 7 days; download link expiration                               |

---

## 4. Data Sources

| PDF Field            | Source                                                                |
|----------------------|-----------------------------------------------------------------------|
| Quote #              | `draftOrder.name` (Shopify), formatted with `quote_number_prefix`     |
| Date                 | `draftOrder.createdAt`                                                |
| Expires              | `draftOrder.metafield(custom.expiry_date)` or `createdAt + default_expiry_days` |
| Sender block         | Shop record + admin overrides                                         |
| Bill To              | `draftOrder.billingAddress`                                           |
| Ship To              | `draftOrder.shippingAddress`                                          |
| Shipping Method      | `draftOrder.shippingLine.title` or `default_shipping_text`            |
| Line items           | `draftOrder.lineItems` with contextual pricing for `Your Price`       |
| MSRP                 | `variant.compareAtPrice` or `variant.price` (configurable)            |
| UOM                  | Variant metafield (`uom_metafield_namespace_key`) or `default_uom`    |
| Subtotal / Total     | `draftOrder.subtotalPrice` / `totalPrice`                             |
| Estimated Shipping   | `draftOrder.shippingLine.price` if present, else `—`                  |

---

## 5. App Admin UI

A single **PDF Settings** page in the Gadget embedded admin, built with
**Shopify App Bridge + Polaris React** components and rendered from
`web/routes/admin/pdf-settings.jsx`. Organized in tabs that mirror the
sections above:

1. **Branding** — logo, colors, fonts, page setup
2. **Header & Meta** — title, quote number format, dates
3. **Sender** — merchant info overrides
4. **Layout** — bill/ship/shipping toggles, table columns, totals
5. **Email** — delivery automation and templates
6. **Localization** — labels, language, currency, RTL

The page must include a **Live Preview** panel rendering a sample quote with
the current settings applied (uses the same `QuotePdfTemplate` React
component that powers the actual PDF render). The **Save** action calls the
`updateAppSettings` Gadget action, which persists to the `appSettings`
model record keyed by `shop`.

---

## 6. API Endpoints (Gadget HTTP routes)

The app is being migrated to **Gadget.dev** (see `MIGRATION_PLAN.md`). All
endpoints below are implemented as Gadget HTTP route handlers under
`api/routes/`, calling Gadget global actions in `api/actions/`. Existing
quote routes (`/api/v1/quote/...`) continue to be honored for backward
compatibility — these are **additions**.

| Method | Route                            | Gadget File                                     | Purpose                                      |
|--------|----------------------------------|-------------------------------------------------|----------------------------------------------|
| GET    | `/api/v1/quote/{id}/pdf`         | `api/routes/api/v1/quote/[id]/pdf.js`           | Stream rendered PDF for a quote              |
| GET    | `/api/v1/quote/{id}/pdf/preview` | `api/routes/api/v1/quote/[id]/pdf/preview.js`   | Return preview HTML (admin live preview)     |
| POST   | `/api/v1/quote/{id}/email`       | `api/routes/api/v1/quote/[id]/email.js`         | Send quote PDF via email using templates     |
| GET    | `/api/v1/pdf-settings`           | `api/routes/api/v1/pdf-settings/GET.js`         | Fetch merchant's current settings            |
| PUT    | `/api/v1/pdf-settings`           | `api/routes/api/v1/pdf-settings/PUT.js`         | Update merchant's settings                   |
| POST   | `/api/v1/pdf-settings/logo`      | `api/routes/api/v1/pdf-settings/logo.js`        | Upload logo asset (Gadget File field)        |

**Authentication**:

- Storefront / merchant API calls: `lib/auth.js` (dual-token during transition
  — legacy bearer + Gadget API key, per Phase 4 of the migration plan).
- Embedded admin calls: Shopify session token validated via Gadget's built-in
  Shopify connection (App Bridge).
- Webhooks: Gadget's automatic HMAC verification.

---

## 7. Storage Model (Gadget data models)

Two new Gadget models. Schemas live in `api/models/<modelName>/schema.gadget.ts`
and are managed through the Gadget editor — no manual migrations needed.

### 7.1 `appSettings` model

Single source of truth for both PDF and Quote List settings (see
`QUOTE_LIST_FEATURE.md`). One record per shop.

| Field             | Gadget Type                          | Notes                                                       |
|-------------------|--------------------------------------|-------------------------------------------------------------|
| `shop`            | BelongsTo(`shopifyShop`), unique     | One settings record per connected Shopify shop              |
| `pdfSettings`     | JSON                                 | Section 3.x keys/values, defaulted when missing             |
| `listSettings`    | JSON                                 | From `QUOTE_LIST_FEATURE.md` Section 3                      |
| `sharedSettings`  | JSON                                 | `primaryColor`, `dateFormat`, `currencyFormat`, etc.        |
| `logo`            | File (Gadget built-in)               | Stored in Gadget File storage; CDN URL auto-generated       |
| `createdAt`       | DateTime (auto)                      |                                                             |
| `updatedAt`       | DateTime (auto)                      |                                                             |

> Storing settings as JSON blobs is intentional — new settings can be added
> without schema migrations, and `lib/settings.js` provides defaults when keys
> are missing.

### 7.2 `quotePdf` model (cache)

Caches rendered PDFs so we're not re-generating on every download.

| Field                  | Gadget Type                         | Notes                                                  |
|------------------------|-------------------------------------|--------------------------------------------------------|
| `shop`                 | BelongsTo(`shopifyShop`), indexed   |                                                        |
| `shopifyDraftOrderId`  | String, indexed                     | Shopify DraftOrder GID (or stripped numeric)           |
| `pdfFile`              | File                                | Rendered PDF binary in Gadget File storage             |
| `settingsHash`         | String                              | SHA-256 of `appSettings.pdfSettings` — invalidate cache when settings change |
| `expiresAt`            | DateTime, indexed                   | Driven by `pdf_retention_days`                         |
| `generatedAt`          | DateTime (auto via `createdAt`)     |                                                        |

Optionally relate `quotePdf` BelongsTo → `draftOrder` (the existing model
from `MIGRATION_PLAN.md`) for a tighter join.

---

## 8. Rendering Pipeline (Gadget / Node)

The rewrite runs on Gadget's Node runtime, so the rendering pipeline is
JavaScript-only — no Blade, no dompdf.

1. **Template**: React component rendered to HTML via `react-dom/server`,
   located at `web/components/QuotePdfTemplate.jsx`. The same component is
   reused for the admin live-preview iframe — single source of truth.
2. **HTML → PDF**: **Puppeteer** (headless Chromium) via the
   `@sparticuz/chromium` + `puppeteer-core` combo so it runs inside Gadget's
   serverless environment. Better CSS fidelity, web fonts, and SVG support
   than dompdf.
3. **Storage**: Gadget's built-in **File field** on the `quotePdf` model —
   no S3 setup, signed URLs are auto-generated by Gadget.
4. **Delivery**: The `/quote/{id}/pdf` route handler:
   - Looks up `quotePdf` cache by `(shopifyDraftOrderId, settingsHash)`
   - On cache miss → calls `renderQuotePdf` action → stores result
   - Streams the file response with `Content-Type: application/pdf`
   - Adds `Content-Disposition: inline` (view) or `attachment` (download)
     based on a query param.

### 8.1 Implementation modules (Gadget structure)

Following the `lib/` convention from `MIGRATION_PLAN.md`:

| Gadget File                          | Purpose                                                         |
|--------------------------------------|-----------------------------------------------------------------|
| `api/actions/renderQuotePdf.js`      | Global action: takes `draftOrderId` → returns rendered PDF      |
| `api/actions/sendQuoteEmail.js`      | Global action: renders PDF + sends via email provider           |
| `api/actions/getAppSettings.js`      | Fetch (with defaults) the merchant's `appSettings` record       |
| `api/actions/updateAppSettings.js`   | Validate + persist settings updates                             |
| `lib/pdf-renderer.js`                | Puppeteer wrapper, font loading, page-size config               |
| `lib/pdf-template.js`                | Server-side React → HTML rendering helper                       |
| `lib/email-templates.js`             | Mustache-style template variable substitution                   |
| `lib/settings-defaults.js`           | Default values for every key in Section 3.x                     |
| `web/components/QuotePdfTemplate.jsx`| The PDF layout React component (used by both renderer + admin preview) |
| `web/routes/admin/pdf-settings.jsx`  | Embedded admin UI page (App Bridge / Polaris)                   |

---

## 9. Open Questions

Items to confirm with stakeholders before implementation:

- [ ] Should MSRP fall back to `compareAtPrice`, or is there a separate B2B
      catalog metafield?
- [ ] Does the merchant need multiple PDF templates per shop (e.g.,
      retail vs. wholesale layouts)?
- [ ] Is signature/approval workflow in scope, or strictly read-only PDFs?
- [ ] Should the PDF support multi-currency display (e.g. quote in USD,
      side-by-side EUR for reference)?
- [ ] Tax compliance — do any merchant regions require statutory fields
      (tax registration, place of supply)?
- [ ] Accessibility — should we generate tagged PDFs (PDF/UA) for
      screen-reader compatibility?

---

## 10. Acceptance Criteria

The feature is considered ship-ready when:

- [ ] A merchant can install the app and produce a branded PDF that visually
      matches the Hovertech sample with **only their logo, address, and
      colors** changed (no code changes).
- [ ] All section 3.x settings are exposed in the admin UI with a live preview.
- [ ] PDFs render correctly in Letter and A4 sizes.
- [ ] Quotes with 1, 10, and 100+ line items paginate cleanly with repeating
      table headers and footers.
- [ ] Generated PDFs are accessible via a signed download URL.
- [ ] Email delivery uses the configured templates and attaches the PDF.
- [ ] Settings persist per shop and survive app reinstall.
