# B2B Supercharge Admin — Design Guide

A complete, self-contained reference for designing pages, sections, and components inside the **B2B Supercharge merchant admin app**. Built on `@shopify/polaris@13.9.5` with a thin brand overlay. Drop this whole file into Claude Design (or any model) as the design source of truth.

---

## 1. What this design system is

- The design source of truth for the **B2B Supercharge merchant admin** — the embedded Shopify admin UI Plus merchants see when they install B2B Supercharge from the App Store.
- A composition layer on Polaris-as-it-ships. Zero custom primitives, zero forks.
- Brand expression = (a) overridden semantic tokens, (b) composition patterns, (c) content/voice. Nothing else.

### What it is NOT
- Not the Makro agency brand. Makro red `#d72c0d`, PP Neue Montreal, dark-first, the red dot — agency-only. Merchants never see "Makro" inside the admin.
- Not the buyer portal (that's Customer Account UI Extensions, separate codebase).
- Not the marketing site.
- Not a Polaris fork. We compose, never reach inside.

---

## 2. Tech stack

```
React 18.3
TypeScript 6.x
Vite 8.x
React Router 7.x
@shopify/polaris        13.9.5  (pinned exact)
@shopify/polaris-tokens ^9.4.2
@shopify/polaris-icons  (icons)
JetBrains Mono          (Google Fonts, weights 400, 500)
```

Polaris is loaded as styles + components. Token overlay (`src/tokens/makro-overlay.css`) is loaded **after** `@shopify/polaris/build/esm/styles.css` so `:root` overrides win.

```tsx
// src/main.tsx (load order matters)
import '@shopify/polaris/build/esm/styles.css';
import './tokens/makro-overlay.css';
```

`<AppProvider>` wraps the app with `linkComponent={PolarisLink}` so Polaris `url` props route through React Router (not full page reloads). `<Frame>` wraps everything in `App.tsx` (required for `TopBar`, `Navigation`, `Toast`, `Loading`).

---

## 3. The 6 locked decisions (the brand "forks")

| # | Decision | Choice |
|---|---|---|
| 1 | Brand accent + status overlay | B2B Supercharge medium overlay: primary `#18181b`, status Tailwind-600, borders zinc-200 |
| 2 | Radius | Polaris default (4 / 6 / 8 / 12 / 20 / 9999 px) |
| 3 | Density | Polaris default globally; `condensed` prop on IndexTable per surface |
| 4 | Typography | Polaris Inter sans (no override); JetBrains Mono override on `--p-font-family-mono` |
| 5 | Dark mode | Light-default, dark-optional (Polaris baseline only, no authored dark overrides) |
| 6 | Voice | B2B Supercharge: declarative, verb-forward, no adjectives, no exclamations, no em dashes |

Everything below is the implementation of these six locks.

---

## 4. Color tokens — the overlay

Only **semantic** tokens are overridden. Raw color tokens (`--p-color-blue-500`) are read-only. Component-internal classes are off-limits.

### 4.1 Brand (near-black zinc, from B2B Supercharge marketing)

| Token | Value | Notes |
|---|---|---|
| `--p-color-bg-fill-brand` | `#18181b` | zinc-900. Primary CTA. |
| `--p-color-bg-fill-brand-hover` | `#27272a` | zinc-800 |
| `--p-color-bg-fill-brand-active` | `#09090b` | zinc-950 |
| `--p-color-bg-fill-brand-selected` | `#18181b` | aligned to brand |
| `--p-color-text-brand-on-bg-fill` | `#ffffff` | re-asserted for clarity |
| `--p-color-border-brand` | `#18181b` | brand-context border |

### 4.2 Status fills (Tailwind-600 family)

Status semantics stay Polaris (info/success/warning/critical/attention/new); only hex values shift. Every override was WCAG AA contrast-checked at Stage 2.1.

**Success (green)**
| Token | Value |
|---|---|
| `--p-color-bg-fill-success` | `#16a34a` (green-600) |
| `--p-color-bg-fill-success-hover` | `#15803d` (green-700) |
| `--p-color-bg-fill-success-active` | `#166534` (green-800) |
| `--p-color-bg-fill-success-secondary` | `#dcfce7` (green-100) |
| `--p-color-text-success-on-bg-fill` | `#ffffff` (paired override for AA 5.20:1) |

**Info (blue)**
| Token | Value |
|---|---|
| `--p-color-bg-fill-info` | `#2563eb` (blue-600) |
| `--p-color-bg-fill-info-hover` | `#1d4ed8` (blue-700) |
| `--p-color-bg-fill-info-active` | `#1e40af` (blue-800) |
| `--p-color-bg-fill-info-secondary` | `#dbeafe` (blue-100) |
| `--p-color-text-info-on-bg-fill` | `#ffffff` (paired override for AA 5.17:1) |

**Warning (amber)**
| Token | Value |
|---|---|
| `--p-color-bg-fill-warning` | `#ca8a04` (amber-600) |
| `--p-color-bg-fill-warning-hover` | `#a16207` (amber-700) |
| `--p-color-bg-fill-warning-active` | `#854d0e` (amber-800) |
| `--p-color-bg-fill-warning-secondary` | `#fef3c7` (amber-100) |

Polaris-default near-black text on amber pairs at 5.83:1 — no paired-text override needed.

**Critical (red)**
| Token | Value |
|---|---|
| `--p-color-bg-fill-critical` | `#dc2626` (red-600) |
| `--p-color-bg-fill-critical-hover` | `#b91c1c` (red-700) |
| `--p-color-bg-fill-critical-active` | `#991b1b` (red-800) |
| `--p-color-bg-fill-critical-selected` | `#991b1b` |
| `--p-color-bg-fill-critical-secondary` | `#fee2e2` (red-100) |

Polaris-default near-white text pairs at 4.83:1 — passes AA, monitor on Polaris upgrades.

### 4.3 Border

| Token | Value |
|---|---|
| `--p-color-border` | `#e4e4e7` (zinc-200, default border across Card / Input / Divider / table) |

### 4.4 Color rules

- **Use Polaris status semantics, not hex.** Pass `tone="success"` on Badge/Banner — the overlay handles the hex.
- **Red = errors / destructive only.** Never decorative.
- **Never use Makro red (`#d72c0d`).** Wrong brand. Wrong product.
- **Don't override `--p-space-*`, `--p-breakpoints-*`, `--p-z-index-*`, or raw color tokens.** Those are red-light. See § 14.

---

## 5. Typography

### 5.1 Sans (Polaris-default, NOT overridden)

`--p-font-family-sans` stays Polaris default (Inter + system stack). Marketing also uses Inter, so we're already aligned.

### 5.2 Mono (JetBrains Mono — overridden)

```css
--p-font-family-mono: 'JetBrains Mono', ui-monospace, SFMono-Regular, 'SF Mono',
  Consolas, 'Liberation Mono', Menlo, monospace;
```

Loaded via Google Fonts in `index.html`:
```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500&display=swap" />
```

### 5.3 Where mono goes
- Order IDs (`#1008`)
- Quote numbers (`Q-00421`)
- SKUs (`GM-3000-COIL`)
- Currency amounts (`$2,838.00`)
- Any hex value, code, ID, or terminal-ish content

### 5.4 The mono helper (v13.9.5 gotcha)

Polaris v13.9.5 `<Text>` has **no `fontFamily` prop**. Wrap a `<span>` instead. Two helpers in `src/routes/components/_shared.tsx`:

```tsx
export const monoStyle: CSSProperties = {
  fontFamily: 'var(--p-font-family-mono)',
};

// Convenience wrapper for inline mono content
export function Mono({ variant = 'bodyMd', children }: MonoProps) {
  return (
    <Text as="span" variant={variant}>
      <span style={monoStyle}>{children}</span>
    </Text>
  );
}

// Usage
<Mono>Q-00421</Mono>
<Mono variant="bodyLg">$2,838.00</Mono>
```

### 5.5 Type scale (Polaris-default, do NOT override)

Polaris ships `headingXs`, `headingSm`, `headingMd`, `headingLg`, `headingXl`, `heading2xl`, `heading3xl` and `bodyXs`, `bodySm`, `bodyMd`, `bodyLg`. Use the `<Text variant>` prop. Never override font-size tokens piecemeal.

---

## 6. Spacing, radius, shadow, motion (Polaris-default)

These are **inherited, not authored**. Listed here for designer reference only.

| Token family | Values | Override? |
|---|---|---|
| `--p-space-*` | 050=2px · 100=4px · 200=8px · 300=12px · 400=16px · 500=20px · 600=24px · 800=32px · 1200=48px · 1600=64px · 2000=80px | **NEVER** |
| `--p-border-radius-*` | 100=4px · 150=6px · 200=8px · 300=12px · 500=20px · full=9999px | Possible but locked default |
| `--p-shadow-*` | `card`, `button`, `100`–`600` | Possible but using defaults |
| `--p-motion-duration-*` | 100ms / 150ms / 200ms / 250ms / 500ms | Default; nothing bounces |
| `--p-breakpoints-*` | sm / md / lg / xl | **NEVER** |
| `--p-z-index-*` | 1–12 | **NEVER** |

Density rule (Fork 3): for `IndexTable`, set `condensed` per-surface when row count > ~25. Never reach for `--p-space-*` to fake density.

---

## 7. Layout hierarchy

Every page follows the same nesting:

```
Frame (wraps the whole app — provided by App.tsx)
└── Page (per-route, with title + primaryAction)
    └── Layout
        └── Layout.Section (or Layout.AnnotatedSection)
            └── Card
                └── BlockStack / InlineStack / InlineGrid / Grid / Box
                    └── Text + content primitives
```

### 7.1 Layout primitives — what to use when

| Primitive | Use for | Key props |
|---|---|---|
| `Page` | Page chrome (title, subtitle, primaryAction, secondaryActions, backAction, breadcrumbs, titleMetadata). One per route. | `title`, `primaryAction`, `secondaryActions`, `backAction`, `narrowWidth`, `fullWidth` |
| `Layout` | Section grid. Use `Layout.Section` for primary content; `Layout.Section variant="oneHalf|oneThird|oneFourth"` for splits; `Layout.AnnotatedSection` for settings pages with side annotations. | `Layout.Section`, `Layout.AnnotatedSection` |
| `Card` | Content grouping. Wrap every distinct content cluster. | `padding`, `background`, `roundedAbove` |
| `BlockStack` | Vertical stack of children. | `gap`, `align`, `inlineAlign` |
| `InlineStack` | Horizontal stack of children. | `gap`, `align`, `blockAlign`, `wrap` |
| `InlineGrid` | Equal-column grids. | `columns={{xs:1, sm:2, md:3, lg:4}}`, `gap` |
| `Grid` | CSS-grid with explicit cell positioning. | `columns`, `areas`, `Grid.Cell` |
| `Box` | Low-level layout primitive when stacks/grids don't fit. Backgrounds, borders, padding, overflow. | `padding`, `background`, `borderColor`, `borderWidth`, `borderRadius`, `shadow`, `minWidth`, `maxWidth`, `overflow` |
| `Divider` | Horizontal rule between sections inside a Card. | `borderColor`, `borderWidth` |
| `Bleed` | Negative-space spacer (extends content past padding). | `marginInline`, `marginBlock` |
| `Scrollable` | Scrollable container with shadow affordances. | `shadow`, `horizontal`, `vertical`, `onScrolledToBottom` |

**Rule of thumb:** reach for `BlockStack` / `InlineStack` / `InlineGrid` first. Use `Box` only when those primitives can't express the constraint.

### 7.2 Don'ts

- Don't nest `<Page>` inside `<Page>`.
- Don't nest `<Frame>` inside `<Frame>` (App.tsx already provides one).
- Don't render `<TopBar>` or `<Navigation>` inside a section — they need Frame context. Use `FrameShellDemo` mockup pattern instead.
- Don't replace Cards with raw `<div>` — use `<Card>` or `<Box>`.

---

## 8. Component catalog (60 components, by category)

Polaris v13.9.5. Every non-deprecated, user-facing component is below. Source: `src/routes/components/INVENTORY.md`.

### Category A — Actions + Navigation (14)

| Component | Purpose | Key props / variants |
|---|---|---|
| `Button` | Primary actionable control. | `variant` (primary / secondary / tertiary / plain / monochromePlain), `tone` (critical / success), `size`, `disabled`, `loading`, `icon`, `fullWidth`, `url` |
| `ButtonGroup` | Group of related buttons. | `variant="segmented"`, `gap`, `fullWidth`, `connectedTop` |
| `Link` | Inline navigation link. | `url`, `external`, `monochrome`, `removeUnderline`, `target` |
| `Pagination` | Next / previous / page-count control. | `hasNext`, `hasPrevious`, `label`, `type` (page / table) |
| `Tabs` | Top-level content switcher. | `tabs`, `selected`, `canBeSaved`, `fitted`, `disclosureText` |
| `Breadcrumbs` | Back-up-one-level nav. | `backAction` (single only in v13) |
| `ActionList` | Menu of actions, usually inside Popover. | `items`, `sections`, `actionRole` |
| `ActionMenu` | Page-level action disclosure. | `actions`, `groups`, `rollup` |
| `KeyboardKey` | Render a keyboard shortcut glyph. | `children` |
| `Navigation` | Sidebar nav (admin chrome — needs Frame). | `Navigation.Section`, `items`, `rollup` |
| `TopBar` | Top admin chrome (needs Frame). | `userMenu`, `searchField`, `secondaryMenu` |
| `PageActions` | Footer row of primary + secondary actions. | `primaryAction`, `secondaryActions` |
| `FullscreenBar` | Fullscreen-mode top bar with back action. | `onAction` |
| `SettingToggle` | Setting row with toggle-style action. | `action`, `enabled` (deprecated — use Card + Button instead) |

### Category B — Forms + Inputs (15)

| Component | Purpose | Key props / variants |
|---|---|---|
| `Form` | `<form>` wrapper with Polaris submit semantics. | `onSubmit`, `preventDefault`, `implicitSubmit` |
| `FormLayout` | Spacing + grouping for form fields. | `FormLayout.Group`, `condensed` |
| `TextField` | Text/number/email/url/password/search/tel input. | `type`, `value`, `error`, `multiline`, `autoComplete`, `prefix`, `suffix`, `connectedLeft`, `connectedRight`, `loading`, `disabled`, `clearButton`, `requiredIndicator` |
| `Select` | Dropdown single-select. | `options`, `value`, `placeholder`, `error`, `requiredIndicator` |
| `Checkbox` | Boolean input. | `checked`, `indeterminate`, `disabled`, `error`, `helpText` |
| `RadioButton` | Single-option-in-group input. | `checked`, `disabled`, `helpText`, `name` |
| `ChoiceList` | Group of checkboxes/radios with shared label. | `title`, `choices`, `allowMultiple`, `error` |
| `Combobox` | Accessible search-and-select. | `activator`, `allowMultiple`, `onScrolledToBottom` |
| `Autocomplete` | Typeahead suggestion field. | `options`, `selected`, `textField`, `loading`, `emptyState`, `allowMultiple` |
| `RangeSlider` | Numeric slider, single or dual thumb. | `min`, `max`, `step`, `value`, `output`, `prefix`, `suffix` |
| `Tag` | Removable input chip (inside Combobox). | `onRemove`, `disabled`, `url` |
| `DatePicker` | Calendar-grid date picker. | `month`, `year`, `selected`, `allowRange`, `disableDatesBefore`, `disableDatesAfter`, `multiMonth` |
| `DropZone` | File-drop target. | `accept`, `type` (file/image), `allowMultiple`, `errorOverlayText`, `customValidator` |
| `ColorPicker` | HSB color selector. | `color`, `onChange`, `allowAlpha` |
| `InlineError` | Inline validation message next to a field. | `message`, `fieldID` |

### Category C — Data display + Feedback (15)

| Component | Purpose | Key props / variants |
|---|---|---|
| `Badge` | Small status label. | `tone` (info / success / warning / critical / attention / new / magic / *-strong / read-only / enabled), `size`, `progress`, `icon` |
| `Banner` | Page- or section-level alert. | `tone` (info / success / warning / critical), `title`, `action`, `secondaryAction`, `onDismiss`, `hideIcon` |
| `Toast` | Ephemeral confirmation (rendered via Frame). | `content`, `action`, `duration`, `onDismiss`, `tone` (critical / magic) |
| `Spinner` | Circular loading indicator. | `size`, `accessibilityLabel` |
| `ProgressBar` | Linear progress indicator. | `progress`, `size`, `tone`, `animated` |
| `Avatar` | User/entity portrait. | `source`, `initials`, `size`, `shape`, `name` |
| `Icon` | Single icon. | `source`, `tone` (base / subdued / critical / interactive / success / warning / primary / magic / inherit), `accessibilityLabel` |
| `Thumbnail` | Small preview image. | `source`, `alt`, `size`, `transparent` |
| `DescriptionList` | Term + description pairs. | `items`, `gap` |
| `DataTable` | Simple tabular data. | `columnContentTypes`, `headings`, `rows`, `totals`, `sortable`, `truncate`, `stickyHeader` |
| `IndexTable` | Resource list with selection, bulk actions, sorting. | `resourceName`, `itemCount`, `selectedItemsCount`, `headings`, `condensed`, `sortable`, `hasZebraStriping`, `bulkActions`, `promotedBulkActions` |
| `IndexFilters` | Filter / search / tab / sort bar above IndexTable. | `tabs`, `queryValue`, `filters`, `sortOptions`, `primaryAction`, `cancelAction`, `mode` (DEFAULT / FILTERING / EDITING_COLUMNS) |
| `ResourceList` / `ResourceItem` | Mobile-friendly list alternative to IndexTable. | `resourceName`, `items`, `renderItem`, `selectable`, `bulkActions` |
| `Skeleton*` (BodyText / DisplayText / Thumbnail / Page / Tabs) | Loading placeholders. | `size`, `lines` (BodyText), `narrowTitle` (Page) |
| `EmptyState` | Zero-data hero with illustration + action. | `heading`, `action`, `secondaryAction`, `image`, `footerContent`, `fullWidth` |

### Category D — Layout + Overlays (16)

| Component | Purpose | Key props / variants |
|---|---|---|
| `Page` | Page chrome. | `title`, `subtitle`, `backAction`, `primaryAction`, `secondaryActions`, `titleMetadata`, `narrowWidth`, `fullWidth` |
| `Layout` | Section / annotated-section grid. | `Layout.Section`, `Layout.AnnotatedSection` |
| `Card` | Content grouping surface. | `padding`, `roundedAbove`, `background` |
| `Box` | Low-level layout primitive. | `as`, `padding`, `background`, `borderColor`, `borderWidth`, `borderRadius`, `shadow`, `minWidth`, `maxWidth`, `minHeight`, `overflow` |
| `BlockStack` | Vertical stack. | `gap`, `align`, `inlineAlign`, `as`, `reverseOrder` |
| `InlineStack` | Horizontal stack. | `gap`, `align`, `blockAlign`, `wrap`, `direction` |
| `InlineGrid` | Columnar grid. | `columns` (number / responsive object / array), `gap`, `alignItems` |
| `Grid` | CSS-grid layout with cell positioning. | `columns`, `areas`, `Grid.Cell` (`columnSpan`, `area`) |
| `Divider` | Horizontal rule. | `borderColor`, `borderWidth` |
| `Bleed` | Negative-space spacer. | `marginInline`, `marginBlock` |
| `Scrollable` | Scrollable container with shadow affordances. | `shadow`, `focusable`, `horizontal`, `vertical`, `onScrolledToBottom` |
| `Collapsible` | Animated hide/show container. | `open`, `id`, `transition`, `expandOnPrint` |
| `Text` | Type primitive. | `as` (h1-h6 / p / span / legend / dt / dd), `variant` (headingXs..heading3xl / bodyXs / bodySm / bodyMd / bodyLg), `tone` (base / subdued / critical / caution / success / disabled / magic / inherit), `fontWeight`, `alignment`, `numeric`, `truncate`, `visuallyHidden` |
| `List` | Ordered or bulleted list. | `type`, `gap` |
| `Modal` | In-app dialog overlay. | `open`, `title`, `primaryAction`, `secondaryActions`, `size` (small / large), `loading`, `noScroll`, `Modal.Section` |
| `Popover` / `Tooltip` / `Sheet` | Floating overlays. Sheet is **deprecated** in v13 — use `<Modal size="large">`. | Popover: `activator`, `active`, `preferredAlignment`, `autofocusTarget`. Tooltip: `content`, `preferredPosition`, `dismissOnMouseOut` |

### Deprecated / do not use

- `LegacyCard`, `LegacyStack`, `LegacyTabs`, `LegacyFilters` (use modern equivalents)
- `Sheet` (use `<Modal size="large">`)
- `SettingToggle` (use `Card` + `BlockStack` + `Button`)
- `ContextualSaveBar` from Polaris (use App Bridge in production; mock with sticky `Box` + `InlineStack` + `Button` in this specimen)
- `TextContainer` (use `BlockStack gap="300"` + `<Text>` children)
- `UnstyledButton`, `UnstyledLink` (use styled variants)

---

## 9. v13.9.5 component gotchas

These are real, will-bite-you-today edge cases. Memorize them.

| Component | Gotcha |
|---|---|
| `Text` | **No `fontFamily` prop.** For mono content wrap `<span style={monoStyle}>` or use the `Mono` helper. |
| `Text` `tone` | Accepts `base \| subdued \| disabled \| critical \| caution \| success \| magic \| inherit`. **Does NOT accept `'warning'`** — use `caution` for warning-adjacent copy. |
| `Button` `tone` | Accepts only `'critical' \| 'success'`. No `'warning'`, `'info'`, `'attention'`. |
| `Badge` `tone` | The rich one. All six semantic tones plus `*-strong` variants plus `read-only`, `enabled`, `magic`. |
| `Icon` `tone` | Accepts `base \| subdued \| critical \| interactive \| success \| warning \| primary \| magic \| inherit` — **DOES accept `warning`**, unlike `Text`. |
| `Banner` inside `Card` | Resolves to `--p-color-bg-surface-*` (lighter) instead of `--p-color-bg-fill-*` (overlay-overridden). To show overlay color on banners, render outside a Card. |
| `Breadcrumbs` | Accepts only `backAction` (single) in v13. Multi-level paths render via `InlineStack` of `<Link>`s. |
| `IndexFilters` | `mode` prop drives `DEFAULT \| FILTERING \| EDITING_COLUMNS` state. |
| `IndexTable` | `condensed` is a boolean prop. `bulkActions` accepts `promotedBulkActions` (inline) and `bulkActions` (disclosure). Use `useIndexResourceState` hook for selection. |
| `Modal` | `Modal.Section` wraps body content. Submit from inside `Form` requires `preventDefault`. |
| `TopBar` / `Navigation` / `Loading` / `Toast` | Need `<Frame>` context. App.tsx provides one — don't nest a second Frame. |
| `Sheet` | Deprecated. Use `<Modal size="large">`. |
| `EmptyState` `image` | Needs a real asset URL. |
| `ContextualSaveBar` (Polaris) | Superseded by App Bridge in production. Mock with sticky `<Box>` + `<InlineStack>` + `<Button>`. |

---

## 10. Section library (21 reusable patterns)

A **section** is a composition richer than one component but smaller than a page. Source: `src/routes/sections/INVENTORY.md`.

### Category E — Shell + page scaffolding (7)

| # | Section | What it is |
|---|---|---|
| E1 | App shell mockup | Three-tier shell: outer Shopify chrome band + TopBar + Navigation + content frame. Uses `FrameShellDemo` since App.tsx already wraps in Frame. |
| E2 | Page header variants | Five `<Page>` chrome variants: title-only, title+subtitle, with primary action, with back+breadcrumbs, full (title+subtitle+breadcrumbs+primary+secondary+badge+tabs). |
| E3 | Tab bar + saved views | Tab row with count badges, saved-view rename, segmentation (All / Active / Paused). |
| E4 | Breadcrumbs row | `backAction` + breadcrumbs for drill-down. Single back-action and multi-level path variants. |
| E5 | Empty / zero / filtered-zero / error / permission states | Five-variant state matrix wrapping a surface. Two-sentence empty copy: fact + action. |
| E6 | Loading skeleton states | Skeleton patterns for dashboard (stat cards), list (IndexTable), detail (form). |
| E7 | Dashboard stat cards + metric grid | Four-metric trend-card row (`InlineGrid` 4-col) + 2x2 metric grid variant. Status deltas via `Badge tone="success" \| "critical"` — NOT inline Tailwind hex. |

### Category F — Data sections (7)

| # | Section | What it is |
|---|---|---|
| F1 | IndexTable + filter bar + bulk actions + pagination | Full list surface. `IndexFilters` above `IndexTable` with selection, bulk actions, row-click to detail, pagination footer. |
| F2 | IndexFilters composed | Standalone: tabs + query + filter chips + sort + saved-view save + cancel. Show DEFAULT and FILTERING modes. |
| F3 | ResourceList (mobile-friendly grid) | Card-style alt to IndexTable: avatar + primary text + metadata + per-item actions + selection. |
| F4 | Dense IndexTable (`condensed` prop) | Same fixture data as F1 with `condensed={true}`, ~25 rows, tighter row height. Implements Fork 3. |
| F5 | Bulk actions + saved views | Promoted + secondary actions, selection-count, select-all-across-pages. Paired with saved-view list (All / Active / Paused / Mine). |
| F6 | Detail header above table | Composite: entity summary card (title + status + 3 meta facts) directly above an IndexTable of child items. Two `<Card>`s, not nested sections. |
| F7 | Summary cards row above tables | Four summary cards (Pending / Approved / Rejected / Total volume) above an IndexTable. Filter-aware numbers. |

### Category G — Forms + interaction patterns (7)

| # | Section | What it is |
|---|---|---|
| G1 | Settings form + contextual save bar | Multi-field settings with `FormLayout` groups, helper text, sticky save-bar mock, destructive zone at bottom. |
| G2 | Multi-step wizard | Step indicator (composed from `Badge` per step + `ProgressBar` overall) + per-step form + Back / Continue / Submit. 4-step example: Company / Buyers / Rules / Review. |
| G3 | Inline edit in IndexTable row | Click cell → edit in place. Enter saves, Escape cancels. Row-level validation via `InlineError`. |
| G4 | Form in modal | `Modal` containing 3-5 field form. Primary submits, secondary cancels. Loading state on submit. |
| G5 | Destructive confirm modal | Critical-tone Modal: title, plain explanation, optional typed-confirmation, `<Button tone="critical" variant="primary">`. Two variants: simple confirm, type-to-confirm. |
| G6 | Detail drawer via large Modal | `<Modal size="large">` since Sheet is deprecated. Resource detail with Close + Save and scrollable body. |
| G7 | Banner stack + toast queue | Priority-ordered banner stack (commentary: "use one at a time in production") + toast queue via Frame context (success / critical / magic). |

---

## 11. Page templates (7 — full B2B Supercharge admin pages)

Source: `src/routes/pages/INVENTORY.md`. Each page composes Stage 4 sections with real fixture data.

### Category H — Core workflow (4)

| Page | Path | Composes | Core task |
|---|---|---|---|
| **ApprovalsDashboard** | `/pages/approvals-dashboard` | E2 + E7 + F7 + F1 + E5 | Review pending approvals. Approve/reject in bulk. See cycle-time and volume trends. |
| **ApprovalsRules** | `/pages/approvals-rules` | E2 + (F1 or F3) + G1 + G5 + G7 | List, create, edit, delete approval rules. Configure approver chains. Type-to-confirm delete. |
| **QuotingDashboard** | `/pages/quoting-dashboard` | E2 + E3 + E7 + F1 + F5 | Browse quotes. Filter by status/owner/company. Bulk send/archive. Save filtered views. |
| **QuoteDetail** | `/pages/quote-detail` | E2 + E4 + F6 + G4 + G5 + G7 | Review a quote in full. Edit line items. Send / approve / archive / clone. Activity timeline. |

### Category I — Platform surfaces (3)

| Page | Path | Composes | Core task |
|---|---|---|---|
| **VisibilityOrderSearch** | `/pages/visibility-order-search` | E2 + G1 + F3 + G7 | Configure which order fields are searchable for buyers. Two-column: settings form + live preview. |
| **ModuleMarketplace** | `/pages/module-marketplace` | E2 + E7 + custom InlineGrid + G4 | Browse / install / configure B2B Supercharge modules. States: installed, not-installed, locked (Plus-only). |
| **Onboarding** | `/pages/onboarding` | E2 + G2 + E5 + G7 | First-run setup. 4-step wizard: Company → Buyers → Rules → Review. Local `useState` for step index, no routing between steps. |

---

## 12. Voice & content rules

**Register: B2B Supercharge — declarative, verb-forward, admin-adapted.** Ported from the marketing site, minus marketing-only moves (contrast headlines, loss framing, "your buyers" possessive, CTA clichés).

### Always
- **Declarative, not persuasive.** "Your quote inbox is empty." NOT "You don't have any quotes yet!"
- **Specific verbs in button labels.** "Send quote", "Approve order", "Create rule", "Save changes". NOT "Continue", "Submit", "Done", "OK".
- **Concrete nouns, no adjectives.** "Quote total $2,838.00" — not "Your impressive quote total".
- **Mono font for data content.** Order IDs, SKUs, quote numbers, amounts.
- **Helper text is one line.** Under every form field, say what the field does. No paragraphs.
- **Empty states: state the fact, then offer the action.** Two short sentences. "No quotes yet. Create one to get started."

### Never
- **No exclamation points** in admin UI. Ever.
- **No em dashes.** Use colons, periods, commas, spaced hyphens, or restructure.
- **No "please" / "sorry".**
- **No second-person possessive.** "Team members" — not "Your team members".
- **No "Makro" inside admin copy.** Merchants installed "B2B Supercharge".
- **No emojis** in nav or button labels.
- **No lorem / placeholder.** Use the fixtures (§ 13).
- **No CTA clichés.** "Get started!", "Let's go!", etc.
- **No contrast headlines** ("X, not Y") — marketing-only move.
- **No loss framing** ("Stop losing deals") — the merchant has already bought.

### Voice samples (port verbatim)
- Empty state CTA: "Create quote"
- Field helper: "Visible to buyers on the storefront."
- Confirmation: "Quote sent to Atlas Industries."
- Banner (error): "Approval rule needs at least one approver."
- Section title (Approvals dashboard): "Pending your review"

---

## 13. Fixture content (real-feeling B2B sample data)

All sample data lives in `src/content/fixtures.ts`. Reuse these names everywhere for consistency between marketing touch and admin session.

### Companies (5)
| ID | Name | Location | Primary contact | Lifetime volume |
|---|---|---|---|---|
| `atlas` | Atlas Industries | Houston, TX | James Rodriguez (Admin) | $1.2M |
| `redline` | Redline Supply Co | Phoenix, AZ | Sarah Kim (Manager) | $847K |
| `greenfield` | Greenfield Mfg | Columbus, OH | Tom Nguyen (Buyer) | $2.4M |
| `pacific` | Pacific Wholesale | Long Beach, CA | Lisa Park (Admin) | $3.1M |
| `summit` | Summit Electric | Albuquerque, NM | David Chen (Buyer) | $658K |

### Quotes (6) — use `Q-NNNNN` format, mono font
`Q-00421` · `Q-00420` · `Q-00419` · `Q-00418` · `Q-00417` · `Q-00416`
Statuses: `draft \| sent \| approved \| rejected \| expired \| converted`
Owners: Morgan Ellis · Priya Shah · Jordan Park

### Orders (8) — use `#NNNN` format, mono font
`#1008` · `#1007` · `#1006` · `#1005` · `#1004` · `#1003` · `#1002` · `#1001`
Statuses: `pending \| processing \| fulfilled \| shipped \| cancelled`
Payment terms: `Net 30 \| Net 45 \| Net 60 \| Prepaid`

### SKUs (10) — vendor-prefix-codes, mono font
`GM-3000-COIL` (Industrial Cable 12AWG) · `AT-V8-HARN` (V8 Engine Wire Harness) · `RS-48-BRKT` (Mounting Bracket 48") · `PW-EMT-34` (Conduit EMT 3/4") · `SE-JB-6X6` (Junction Box 6x6) · `GM-PANEL-24C` (24-Circuit Panel) · `AT-TRQ-1-2` (Torque Wrench 1/2") · `RS-CONN-M8` (Connector Kit M8) · `PW-TRAY-18` (Cable Tray 18") · `SE-BUS-400` (Bus Bar 400A)
Categories: `Wire \| Hardware \| Fittings \| Tools \| Panels`

### Approval rules (3)
- "Orders over $25K" — approver chain: VP Operations + Accounting
- "Net 60 terms" — approver: Accounting
- "First order from new company" — approver chain: Branch Manager + VP Operations

### Currency formatting
```ts
formatCurrency(2838) // "$2,838.00"
```
Always two decimal places. Always render in mono.

---

## 14. Polaris constraints — what's overridable

| Token category | Status | Notes |
|---|---|---|
| Semantic color tokens (`--p-color-bg-fill-brand`, status tones) | **GREEN** | Primary override surface. Validate WCAG contrast. |
| Border radius tokens | **GREEN** | All overridable. We chose default. |
| Semantic shadow tokens (`--p-shadow-card`, `--p-shadow-button`) | **GREEN** | Subtle brand expression lever. |
| Font-family tokens (`--p-font-family-sans`, `--p-font-family-mono`) | **GREEN** | We override mono only. |
| Motion duration / easing tokens | **GREEN** (with restraint) | Overriding fights admin familiarity. |
| Font-size and line-height tokens | **YELLOW** | Override as a consistent scale, never piecemeal. |
| Height / width tokens used by components | **YELLOW** | Validate with real content first. |
| Font-weight, letter-spacing | **YELLOW** | Higher risk of breaking hierarchy. |
| Space tokens (`--p-space-*`) | **RED** | Never override. Cascades through every Card / Stack / Page. |
| Breakpoint tokens (`--p-breakpoints-*`) | **RED** | Never override. Polaris components key off these. |
| Z-index tokens (`--p-z-index-*`) | **RED** | Never override. Modal/Popover/Tooltip/Banner/Toast stacking depends on these. |
| Raw / scale color tokens (`--p-color-blue-500`) | **RED** | Override semantic tokens that consume these instead. |
| Component-internal class names (`.Polaris-Button__Inner_abc123`) | **RED** | Will silently break on next Polaris upgrade. |

---

## 15. Shopify embedded admin considerations

The admin runs **embedded inside the Shopify admin iframe**. Outer chrome (store switcher, merchant account menu, global search) is Shopify's. We never render those.

- **Sidebar navigation** is configured via App Bridge `s-app-nav` in production. In this specimen we mock it visually with Polaris `<Navigation>`.
- **Contextual Save Bar** is a mandatory App Bridge pattern in production. Mock with sticky `<Box>` + `<InlineStack>` + `<Button>`.
- **Toast** and **Modal** dispatches go through App Bridge in production. Polaris `<Modal>` is for in-app dialogs.
- **App Store review** rejects: custom navigation, custom save buttons, serif/script fonts, emojis in nav/buttons, red used for non-error purposes, auto-launching modals on app load, pressure tactics, non-responsive layouts.
- **Shopify Plus-only market.** Users are sophisticated. Density and verb-forward voice are appropriate.
- **Data density.** Plan for 10,000+ orders, 5,000+ customers, hundreds of quotes per month. Pagination over infinite scroll. Saved views over ad-hoc filters.
- **Print styles matter.** Merchants print quotes and invoices. Test page templates: no sidebar, clean type, readable tables, monochrome status markers.

---

## 16. Code patterns & helpers

These helpers live in `src/routes/components/_shared.tsx` and are used across the catalog and section files. Adopt the same patterns when designing new pages.

### `Mono` — render IDs / amounts / SKUs in JetBrains Mono
```tsx
<Mono>Q-00421</Mono>
<Mono variant="bodyLg">$2,838.00</Mono>
```

### `monoStyle` — for inline mono content where `Mono` doesn't fit
```tsx
<Text as="span" variant="bodyMd">
  Order <span style={monoStyle}>#1008</span> ready to ship
</Text>
```

### `CatalogSection` — top-level wrapper for a category file
```tsx
<CatalogSection title="Category A · Actions + Navigation" description="Buttons, links, nav, action menus.">
  {/* one or more <ComponentCase> children */}
</CatalogSection>
```

### `ComponentCase` — one Card per Polaris component demo
```tsx
<ComponentCase title="Button" description="Primary actionable control." id="button">
  <VariantRow label="Primary" meta="variant=primary">
    <Button variant="primary">Send quote</Button>
  </VariantRow>
</ComponentCase>
```

### `VariantRow` — labeled row demonstrating one variant
```tsx
<VariantRow label="Primary disabled" meta="disabled={true}" direction="inline">
  <Button variant="primary" disabled>Send quote</Button>
  <StateTag tone="disabled">disabled</StateTag>
</VariantRow>
```

### `StateTag` — Badge wrapper for state annotations
```tsx
<StateTag tone="error">error</StateTag>      // critical badge
<StateTag tone="hover">hover</StateTag>      // new tone
<StateTag tone="focus">focus</StateTag>      // attention tone
<StateTag tone="loading">loading</StateTag>  // info tone
<StateTag tone="disabled">disabled</StateTag>// read-only tone
```

### `SectionCase` — one Card per section composition (richer than ComponentCase, with optional aside)
```tsx
<SectionCase
  title="IndexTable with filter bar + bulk actions"
  description="Full list surface: filters, selection, pagination."
  aside={<SectionNote>Use condensed once row count exceeds ~25.</SectionNote>}
>
  <MyComposition />
</SectionCase>
```

### `FrameShellDemo` — mock the three-tier app shell without nesting Frame
```tsx
<FrameShellDemo
  topBarLeft={<Text as="span" variant="bodySm" fontWeight="medium">B2B Supercharge</Text>}
  topBarRight={<Text as="span" variant="bodySm" tone="subdued">Account</Text>}
  navigation={<Navigation>{/* ... */}</Navigation>}
>
  <YourPageContent />
</FrameShellDemo>
```

### `PolarisLink` (in `main.tsx`) — wires Polaris `url` props through React Router
Already configured globally via `<AppProvider linkComponent={PolarisLink}>`. Internal URLs use React Router; absolute URLs (`http`, `mailto:`, `tel:`) and explicit `external` open in a new tab.

---

## 17. The full token overlay (copy-paste ready)

```css
/* src/tokens/makro-overlay.css
 * Loaded AFTER @shopify/polaris/build/esm/styles.css so :root wins.
 * Only semantic tokens overridden. WCAG AA validated.
 */
:root {
  /* --- Brand (zinc near-black) --- */
  --p-color-bg-fill-brand: #18181b;
  --p-color-bg-fill-brand-hover: #27272a;
  --p-color-bg-fill-brand-active: #09090b;
  --p-color-bg-fill-brand-selected: #18181b;
  --p-color-text-brand-on-bg-fill: #ffffff;
  --p-color-border-brand: #18181b;

  /* --- Status: success (green) --- */
  --p-color-bg-fill-success: #16a34a;
  --p-color-bg-fill-success-hover: #15803d;
  --p-color-bg-fill-success-active: #166534;
  --p-color-bg-fill-success-secondary: #dcfce7;
  --p-color-text-success-on-bg-fill: #ffffff;

  /* --- Status: info (blue) --- */
  --p-color-bg-fill-info: #2563eb;
  --p-color-bg-fill-info-hover: #1d4ed8;
  --p-color-bg-fill-info-active: #1e40af;
  --p-color-bg-fill-info-secondary: #dbeafe;
  --p-color-text-info-on-bg-fill: #ffffff;

  /* --- Status: warning (amber) --- */
  --p-color-bg-fill-warning: #ca8a04;
  --p-color-bg-fill-warning-hover: #a16207;
  --p-color-bg-fill-warning-active: #854d0e;
  --p-color-bg-fill-warning-secondary: #fef3c7;

  /* --- Status: critical (red) --- */
  --p-color-bg-fill-critical: #dc2626;
  --p-color-bg-fill-critical-hover: #b91c1c;
  --p-color-bg-fill-critical-active: #991b1b;
  --p-color-bg-fill-critical-selected: #991b1b;
  --p-color-bg-fill-critical-secondary: #fee2e2;

  /* --- Default border --- */
  --p-color-border: #e4e4e7;

  /* --- Mono font family --- */
  --p-font-family-mono: 'JetBrains Mono', ui-monospace, SFMono-Regular,
    'SF Mono', Consolas, 'Liberation Mono', Menlo, monospace;
}
```

---

## 18. Quick reference: designing a new page

1. **Start with `<Page>`** — title, primary action, optional subtitle / breadcrumbs / back action / titleMetadata.
2. **Wrap content in `<Layout>` + `<Layout.Section>`** (use `oneHalf`, `oneThird` variants for splits, or `Layout.AnnotatedSection` for settings).
3. **Group every content cluster in `<Card>`** — never raw `<div>`.
4. **Inside Cards, compose with `BlockStack` / `InlineStack` / `InlineGrid`** — gap props from the 4px scale (`100`, `200`, `300`, `400`, `500`).
5. **Use real fixture data** from § 13. Reuse the five companies for cross-page continuity.
6. **Render IDs / SKUs / quote numbers / amounts in `<Mono>`.**
7. **Status semantics via Polaris tones** (`tone="success"`, `tone="critical"`, etc.) — the overlay handles hex.
8. **Forms get `<FormLayout>`, helper text on every field, mock save bar at top when dirty.**
9. **Lists get `<IndexTable>` + `<IndexFilters>` with `condensed` when row count > ~25.**
10. **Empty states: two short sentences, fact + verb-first action.** No exclamations.
11. **Destructive actions: critical-tone `<Modal>` with type-to-confirm when stakes are high.** `<Button tone="critical" variant="primary">`.
12. **One Banner per page max.** No auto-dismiss for important info.
13. **Voice check before commit.** No "Makro", no em dashes, no exclamations, no "please" / "sorry", no second-person possessive.

---

## 19. File map (for reference)

```
makro-polaris-admin-system/
├── src/
│   ├── main.tsx                          ← Polaris + overlay + router + linkComponent
│   ├── App.tsx                           ← Frame + Navigation + lazy routes
│   ├── tokens/makro-overlay.css          ← the 13 token overrides (§ 17)
│   ├── content/fixtures.ts               ← companies / quotes / orders / SKUs / rules
│   └── routes/
│       ├── Home.tsx                      ← design-system front door
│       ├── Tokens.tsx                    ← Polaris-default vs overlay specimen
│       ├── Components.tsx                ← 60-component catalog
│       ├── Sections.tsx                  ← 21-section library
│       ├── Pages.tsx                     ← 7-page templates index
│       ├── components/
│       │   ├── _shared.tsx               ← Mono, monoStyle, CatalogSection, ComponentCase, VariantRow, StateTag, SectionCase, FrameShellDemo, SectionNote
│       │   ├── INVENTORY.md              ← full component partition
│       │   └── CategoryA..D_*.tsx
│       ├── sections/
│       │   ├── INVENTORY.md              ← 21 sections across E / F / G
│       │   └── CategoryE..G_*.tsx
│       └── pages/
│           ├── INVENTORY.md              ← 7 pages across H / I
│           ├── ApprovalsDashboard.tsx
│           ├── ApprovalsRules.tsx
│           ├── QuotingDashboard.tsx
│           ├── QuoteDetail.tsx
│           ├── VisibilityOrderSearch.tsx
│           ├── ModuleMarketplace.tsx
│           └── Onboarding.tsx
└── direction/
    ├── PRINCIPLES.md                     ← all 6 locks + voice rules + do/don't
    ├── POLARIS-CONSTRAINTS.md            ← red/yellow/green token matrix
    ├── METHOD.md                         ← 10-phase playbook
    ├── REFERENCES.md                     ← Phase 1 reference research
    └── STATUS.md                         ← session log
```
