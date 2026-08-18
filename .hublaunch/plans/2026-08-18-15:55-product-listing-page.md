# Add a Static Product Listing Page (vanilla HTML + JavaScript, mock data)

## Plan Summary

- **What/why**: Adds a self-contained ecommerce product listing page (grid of product cards, text search, category filter) to this repository. There is currently no frontend of any kind. This is a frontend-only mock — no backend, no API calls, no build step.
- **Key decision**: Hardcoded product array in a plain JS file loaded via classic `<script src>` tags, instead of `fetch()` + JSON or ES modules. Reason: the page must open by double-clicking `public/index.html` (a `file://` URL), where both `fetch()` and `<script type="module">` are blocked by browser CORS rules. Classic scripts are the only option that works with zero tooling.
- **Most important files**: `public/index.html`, `public/app.js`.
- **Priority/complexity**: Medium priority, Simple complexity.

## Problem Statement

This repository (`test-mobile-app`) has no user interface. It contains a single Node entry point, [`src/index.js`](src/index.js), which logs a string. The goal is a minimal but complete ecommerce **product listing page**: a browsable grid of products with a name, price, image, and category, plus basic search and filtering — implemented entirely in HTML, CSS, and browser JavaScript with mock data.

### Planning Context

**Key Requirements (stated by the requester):**

- Product listing page for ecommerce.
- Keep information **minimal** — the simplest, easiest set of fields.
- **Frontend only**, in HTML and JavaScript. No backend, no server-side code.
- Best-guess defaults were to be chosen for every open decision rather than clarified. All such decisions are recorded below under "Decisions Made" so the implementing agent has no ambiguity.

**Decisions Made (all defaults, chosen during planning):**

1. **Product fields are exactly five: `id`, `name`, `price`, `image`, `category`.**
   Rationale: `name` + `price` + `image` is the minimum that reads as a product card; `id` is needed as a stable key for DOM rendering; `category` is needed for the filter control. No description, rating, stock, SKU, variants, or currency field — those were deliberately excluded to honour the "minimal info" requirement.

2. **Mock data lives in a hardcoded JavaScript array in `public/products.js`, not a `.json` file.**
   Rationale: loading a `.json` file requires `fetch()`, which browsers block under the `file://` protocol with a CORS error (`Cross origin requests are only supported for protocol schemes: http, https...`). A hardcoded array loaded by a `<script>` tag has no such restriction, so the page works by double-clicking the HTML file with no web server running.

3. **Classic `<script src="...">` tags, NOT `<script type="module">`, and NOT ES `import`/`export` syntax in the browser files.**
   Rationale: ES modules are also subject to CORS under `file://` and fail to load. Files therefore communicate through global variables on `window`.

4. **Product images are inline SVG data URIs, not remote URLs and not binary image files.**
   Rationale: remote image URLs (e.g. `via.placeholder.com`, Unsplash) require network access and break when offline or when the host disappears; committing binary files bloats a throwaway test repository. A data URI is plain text, works offline forever, and needs no assets directory.

5. **Features are limited to: render the grid, filter by search text, filter by category, and show an empty state.**
   Rationale: matches "minimal". Explicitly no cart, no product detail page, no pagination, no sorting, no routing, no persistence.

6. **No dependencies, no bundler, no framework, no CSS framework.**
   Rationale: [`package.json`](package.json) currently has zero dependencies and zero build tooling. Adding any would contradict "simplest and easiest" and would require a build step before the page could be viewed.

7. **Pure logic functions (`filterProducts`, `formatPrice`) are dual-exported** — attached to `window` for the browser and to `module.exports` for Node — so they can be unit-tested with the repository's existing `node --test` runner without adding a test framework or a DOM emulator.

**Out of Scope (do NOT implement):**

- Any backend, API endpoint, database, or `fetch()` call.
- Shopping cart, checkout, wishlist, or any purchase flow.
- Product detail pages, routing, or navigation between pages.
- Pagination, infinite scroll, sorting controls.
- User accounts, authentication, or the login module described in [`.hublaunch/plans/2026-08-10-18:00-login.md`](.hublaunch/plans/2026-08-10-18:00-login.md) (that is a separate, unrelated plan).
- Changing the existing behaviour of [`src/index.js`](src/index.js).
- TypeScript, JSX, npm dependencies, or any build/bundle step.

### Background & Context

**Current Behavior**: The repository contains only `src/index.js`, `package.json`, and `README.md`. Running `npm start` prints `test-mobile-app running`. There is no `public/` directory, no HTML file, and no test file anywhere in the repository.

**Desired Behavior**: Opening `public/index.html` in any modern browser — with no server and no install step — displays a responsive grid of mock product cards. Typing in a search box narrows the grid by product name as you type. Selecting a category from a dropdown narrows it by category. The two filters combine (both apply at once). When no product matches, a friendly empty-state message replaces the grid. `npm test` passes and covers the filtering and price-formatting logic.

## Detailed Requirements

### Functional Requirements

1. **Product data**
   - Exactly 8 mock products, defined as a JavaScript array of objects in `public/products.js`.
   - Each object has exactly these five keys: `id` (number), `name` (string), `price` (number, in whole US dollars and cents, e.g. `24.99`), `image` (string, an inline SVG data URI), `category` (string).
   - Categories used across the 8 products: `"Apparel"`, `"Accessories"`, `"Home"`. At least two products per category so filtering is visibly meaningful.

2. **Grid rendering**
   - On page load, all 8 products render as cards in a responsive CSS Grid.
   - Each card shows, top to bottom: the image, the category (small, muted text), the product name, and the formatted price.
   - Prices display as US dollars with two decimals, e.g. `$24.99`, produced by `Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' })`.

3. **Search filter**
   - A single text input filters products by `name` using a case-insensitive substring match.
   - Filtering happens on every `input` event (live, as the user types). No submit button, no debounce needed at this data size.
   - Leading and trailing whitespace in the query is ignored (trim before matching).

4. **Category filter**
   - A `<select>` dropdown lists `All` plus each distinct category, derived programmatically from the product array (do not hardcode the option list — it must stay correct if a product is added).
   - `All` is selected by default and matches every product.
   - Filtering happens on the `change` event.

5. **Combined filtering and result count**
   - The search filter and the category filter apply together (logical AND).
   - A line above the grid reads `Showing N of 8 products`, where `N` is the current match count and `8` is the total. It updates on every filter change.

6. **Empty state**
   - Edge case: when zero products match, the grid area is replaced by the message `No products match your search.` — the result-count line still displays `Showing 0 of 8 products`.

### Technical Requirements

- **Technology**: Plain HTML5, CSS3, and ES2015+ JavaScript that runs directly in the browser. Node.js 18+ only for running the tests.
- **Location**: A new `public/` directory at the repository root. The existing `src/` directory is for Node code and is not touched except as noted below.
- **Dependencies**: None. `package.json` gains no `dependencies` or `devDependencies` entries.
- **Constraints**:
  - The page must work when opened as a `file://` URL. This forbids `fetch()`, `XMLHttpRequest`, `<script type="module">`, and any `import`/`export` statement in the `public/` JavaScript files.
  - No network requests of any kind at runtime — no CDN scripts, no external stylesheets, no remote fonts, no remote images.

### Non-Functional Requirements

- **Verified environment facts** (confirmed against this repository, do not re-derive): Node's `node --test` runner discovers `public/app.test.js` recursively from the repository root, so the existing `"test": "node --test"` script needs no change; and `new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' })` formats `24.99` as exactly `$24.99` and `10` as exactly `$10.00` (a plain ASCII `$`, no space).

- **Performance**: Not a concern at 8 products. Re-render the full list on each filter change; do not implement virtualisation or diffing.
- **Security**: Product names are inserted with `textContent`, never `innerHTML`, so that data can never be interpreted as markup. See "Security Considerations" below.
- **Backwards Compatibility**: Purely additive. `npm start` must continue to print `test-mobile-app running` exactly as it does today.
- **Error Handling**: There is no I/O and no async work, so there are no runtime failure modes to handle. The only defensive behaviour required is the empty state (requirement 6).

## Proposed Solution

**High-level approach**: Create a `public/` directory holding four files — an HTML shell, a stylesheet, a mock-data script, and an application script. Load them with classic `<script>` tags in dependency order (`products.js` before `app.js`) so `app.js` can read the global that `products.js` defines. Keep all pure logic (filtering, price formatting) in standalone functions that are exported to both `window` and `module.exports`, so the same functions are unit-testable under Node without a DOM.

```
 index.html
     │  loads (in order, classic <script> tags)
     ├──────────────▶ styles.css
     ├──────────────▶ products.js ──▶ window.PRODUCTS  (8 mock objects)
     └──────────────▶ app.js
                        │  reads window.PRODUCTS
                        │  defines filterProducts(), formatPrice()
                        │  wires #search (input) + #category (change)
                        ▼
                     re-renders #grid + #count on every change

 app.js also does:  module.exports = { filterProducts, formatPrice }
                        │
                        ▼
                  public/app.test.js  (node --test)
```

### Key Components

1. **`public/index.html`** — the static shell. Holds the page header, the two filter controls, the result-count line, and an empty `<div id="grid">` that JavaScript fills. Contains no inline JavaScript and no inline styles.

2. **`public/products.js`** — the mock dataset. Assigns the array to `window.PRODUCTS` and also to `module.exports` so the test file can import the same data the page uses.

3. **`public/app.js`** — all behaviour. Two pure functions plus DOM wiring, guarded so that it does nothing DOM-related when loaded under Node.

4. **`public/styles.css`** — a small stylesheet: responsive grid, card styling, and the filter bar.

5. **`public/app.test.js`** — `node:test` unit tests for the two pure functions.

### Files Likely to Change

- `public/index.html` — **new**. Page shell and script tags.
- `public/styles.css` — **new**. Grid and card styling.
- `public/products.js` — **new**. The 8 mock products.
- `public/app.js` — **new**. Filtering, formatting, rendering, event wiring.
- `public/app.test.js` — **new**. Unit tests.
- [`README.md`](README.md) — **modified**. Add a short "Product listing page" section explaining how to open the page.
- [`package.json`](package.json) — **not modified**. The existing `"test": "node --test"` script already discovers `public/app.test.js` because Node's test runner matches any `*.test.js` file under the working directory.
- [`src/index.js`](src/index.js) — **not modified**. The listing page is browser-only and has no relationship to the Node entry point.

### Code Patterns to Follow

**Pattern references:**

- **For the dual browser/Node export idiom**: this repository has no existing example, so use exactly this shape at the bottom of both `public/products.js` and `public/app.js`:

  ```js
  // Dual export: the browser reads the globals above; `node --test` reads module.exports.
  if (typeof module !== 'undefined' && module.exports) {
    module.exports = { filterProducts, formatPrice };
  }
  ```

  The `typeof module !== 'undefined'` guard is required — without it the browser throws `ReferenceError: module is not defined` and the whole script stops executing.

- **For the DOM-wiring guard in `public/app.js`**: the same file is loaded by Node during tests, where `document` does not exist. Guard the wiring so it only runs in a browser:

  ```js
  if (typeof document !== 'undefined') {
    document.addEventListener('DOMContentLoaded', init);
  }
  ```

- **For test style**: follow [`.hublaunch/plans/2026-08-10-18:00-login.md`](.hublaunch/plans/2026-08-10-18:00-login.md) lines 25-27, which specifies `node:test` + `node:assert` with small, focused assertions and no mocking library. Use the same import style:

  ```js
  const { test } = require('node:test');
  const assert = require('node:assert');
  ```

- **For module syntax**: follow [`src/index.js`](src/index.js) line 9 (`module.exports = { main };`) — this repository is CommonJS. [`package.json`](package.json) has no `"type": "module"` field, so `.js` files are CommonJS and `require`/`module.exports` is correct in Node.

**Anti-patterns to avoid:**

- ❌ **Do not use `innerHTML` with product data.** It is an XSS vector and unnecessary here. Build card elements with `document.createElement()` and set text via `textContent`. Setting `innerHTML = ''` on the grid container to clear it is acceptable — it involves no data.
- ❌ **Do not use `fetch()` or load a `.json` file.** It fails under `file://`. The data is a hardcoded array by design.
- ❌ **Do not use `<script type="module">` or `import`/`export` in `public/`.** Same `file://` failure.
- ❌ **Do not add a bundler, framework, CSS framework, or any npm dependency.**
- ❌ **Do not reference remote URLs** for images, fonts, or stylesheets. Everything ships in-repo.
- ❌ **Do not hardcode the category dropdown options** in the HTML. Derive them from the data at runtime so the control stays correct if the product array changes.

## Implementation Steps

#### Phase 1: Mock data

- [ ] Create `public/products.js`.
- [ ] Define `const PRODUCTS = [...]` with exactly 8 objects, each having the keys `id`, `name`, `price`, `image`, `category`, and nothing else.
- [ ] Use these categories, so each has at least two members: `Apparel`, `Accessories`, `Home`.
- [ ] For every `image` value, use an inline SVG data URI placeholder. Vary only the fill colour and the label per product so cards are visually distinguishable. Use this exact template, substituting the colour and the label:

  ```js
  image: 'data:image/svg+xml;utf8,' + encodeURIComponent(
    '<svg xmlns="http://www.w3.org/2000/svg" width="400" height="300">' +
      '<rect width="400" height="300" fill="#e2e8f0"/>' +
      '<text x="200" y="160" font-family="sans-serif" font-size="28" fill="#64748b" text-anchor="middle">Tee</text>' +
    '</svg>'
  ),
  ```

  `encodeURIComponent` is required because a raw SVG contains `#` and `<` characters that break a data URI when unescaped.
- [ ] At the end of the file, expose the array to both environments:

  ```js
  if (typeof window !== 'undefined') { window.PRODUCTS = PRODUCTS; }
  if (typeof module !== 'undefined' && module.exports) { module.exports = PRODUCTS; }
  ```

#### Phase 2: Application logic

- [ ] Create `public/app.js`.
- [ ] At the top of the file, resolve the product array through a guarded lookup so the file is safe to `require()` under Node, where neither `window` nor `PRODUCTS` exists. An unguarded `const PRODUCTS = window.PRODUCTS;` throws `ReferenceError: window is not defined` under `node --test` and breaks the whole test run:

  ```js
  // In the browser this is the global set by products.js; under `node --test` it is [].
  var PRODUCTS = (typeof window !== 'undefined' && window.PRODUCTS) || [];
  ```

  The pure functions (`filterProducts`, `formatPrice`) must never read this variable — they take everything they need as arguments, which is what makes them testable. Only `render`, `populateCategories`, and `init` touch it.
- [ ] Implement `formatPrice(value)`:
  - Returns a US-dollar string with two decimals, using `new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(value)`.
  - `formatPrice(24.99)` must return the string `"$24.99"`; `formatPrice(10)` must return `"$10.00"`.
- [ ] Implement `filterProducts(products, query, category)`:
  - `products` is an array of product objects; `query` is a search string; `category` is a category name or the literal string `'All'`.
  - Returns a **new** array containing every product where **both** conditions hold:
    - `product.name.toLowerCase()` contains `query.trim().toLowerCase()` as a substring. An empty or whitespace-only query matches everything (`''` is a substring of every string, so this falls out naturally).
    - `category === 'All'` **or** `product.category === category`.
  - Must not mutate the input array.
- [ ] Implement `render(list)`:
  - Clears `#grid` (`grid.innerHTML = ''` — safe, no data involved).
  - If `list.length === 0`, appends a single `<p class="empty">` element whose `textContent` is `No products match your search.` and returns.
  - Otherwise, for each product, builds a card with `document.createElement` and appends it:
    - `<article class="card">` containing
    - `<img class="card-image">` with `src = product.image` and `alt = product.name`,
    - `<p class="card-category">` with `textContent = product.category`,
    - `<h2 class="card-name">` with `textContent = product.name`,
    - `<p class="card-price">` with `textContent = formatPrice(product.price)`.
  - Always sets `#count`'s `textContent` to `` `Showing ${list.length} of ${PRODUCTS.length} products` `` — this is the only place the total is read, and it runs only in the browser where `PRODUCTS` is populated.
- [ ] Implement `populateCategories()`:
  - Builds the `#category` dropdown options as `All` followed by each distinct `product.category` value, in first-seen order. Derive the distinct list with `[...new Set(PRODUCTS.map(p => p.category))]`.
- [ ] Implement `init()`:
  - Calls `populateCategories()`, then `render(PRODUCTS)`.
  - Adds an `input` listener on `#search` and a `change` listener on `#category`; both call a shared `applyFilters()` helper that reads the current values of both controls and calls `render(filterProducts(PRODUCTS, searchValue, categoryValue))`.
- [ ] Guard the DOM wiring so the file is safe to `require()` from Node:

  ```js
  if (typeof document !== 'undefined') {
    document.addEventListener('DOMContentLoaded', init);
  }
  ```
- [ ] Export the pure functions for tests:

  ```js
  if (typeof module !== 'undefined' && module.exports) {
    module.exports = { filterProducts, formatPrice };
  }
  ```

#### Phase 3: Markup and styling

- [ ] Create `public/index.html` with `<!DOCTYPE html>`, `<html lang="en">`, `<meta charset="utf-8">`, `<meta name="viewport" content="width=device-width, initial-scale=1">`, and `<title>Products</title>`.
- [ ] Link the stylesheet: `<link rel="stylesheet" href="styles.css">`.
- [ ] Body structure, in order:
  - `<header>` with an `<h1>Products</h1>`.
  - A filter bar containing `<input id="search" type="search" placeholder="Search products…">` and `<select id="category"></select>` (left empty — filled by `populateCategories()`), each with an associated visually-labelled or `aria-label`-ed control.
  - `<p id="count"></p>` for the result count.
  - `<div id="grid"></div>` — left empty; JavaScript fills it.
- [ ] Add the scripts at the end of `<body>`, in this exact order (`products.js` must come first because `app.js` reads the global it defines):

  ```html
  <script src="products.js"></script>
  <script src="app.js"></script>
  ```
- [ ] Create `public/styles.css`:
  - A neutral system font stack, a light background, and a centred content column with `max-width: 1100px; margin: 0 auto; padding: 1.5rem;`.
  - `#grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 1rem; }` — this gives responsive columns with no media queries.
  - `.card` — white background, `border-radius: 8px`, subtle border or shadow, `overflow: hidden`.
  - `.card-image { width: 100%; aspect-ratio: 4 / 3; object-fit: cover; display: block; }`.
  - `.card-category` — small, uppercase, muted grey.
  - `.card-price` — bold.
  - `.empty` — centred, muted text.

#### Phase 4: Tests and documentation

- [ ] Create `public/app.test.js` using `node:test` and `node:assert` (see Testing Requirements below for the exact cases).
- [ ] Run `npm test` and confirm it passes.
- [ ] Open `public/index.html` in a browser and confirm the manual checklist below.
- [ ] Update [`README.md`](README.md) with a `## Product listing page` section stating: the page lives at `public/index.html`; open it directly in a browser (no server or install needed); it uses mock data from `public/products.js`; it supports search-by-name and category filtering.

<details>
<summary><b>Implementation Detail</b></summary>

### 6. Edge Cases & Considerations

#### Edge Cases to Handle

1. **No products match the active filters** — replace the grid content with a single `<p class="empty">No products match your search.</p>`. The count line must still render, showing `Showing 0 of 8 products`.
2. **Empty or whitespace-only search query** — must match all products. Trimming the query before comparison makes `''` the search term, and `''` is a substring of every string, so no special-casing is needed.
3. **Mixed-case search input** — lowercase both the query and the product name before comparing, so `TEE`, `tee`, and `Tee` all match a product named `Classic Tee`.
4. **`category` set to `All`** — the category condition short-circuits to `true`; only the text query applies.
5. **Page opened from `file://`** — the primary supported way to view the page. Everything must work with zero network access: no `fetch`, no ES modules, no remote assets.
6. **`app.js` loaded under Node during tests** — `document` and `window` are undefined. The `typeof document !== 'undefined'` guard around the wiring, and the `typeof window !== 'undefined'` guard around the global assignment, prevent a `ReferenceError` from breaking the test run.

#### Potential Challenges

- ⚠️ **`file://` blocks `fetch` and ES modules.** This is the single constraint that shapes the whole design. Any deviation toward `fetch('products.json')` or `<script type="module">` produces a page that is blank when opened directly, with a CORS error in the console. Hardcoded array + classic scripts is the mitigation.
- ⚠️ **Script order matters.** `app.js` reads `window.PRODUCTS`, which `products.js` defines. If the tags are reversed, `PRODUCTS` is `undefined` at `init()` time. Keep `products.js` first.
- ⚠️ **Raw SVG in a data URI breaks without escaping.** An unescaped `#` in a fill colour terminates the URI as a fragment identifier and the image fails to render. Always wrap the SVG string in `encodeURIComponent`.
- ⚠️ **`node --test` file discovery.** Node's test runner matches files named `*.test.js` recursively from the working directory, so `public/app.test.js` is picked up by the existing `"test": "node --test"` script with no change to `package.json`.

#### Security Considerations

- All product text is written with `textContent`, never `innerHTML`. Even though the data is trusted and local, this makes the rendering path immune to markup injection and is the correct habit for a page that would later be fed real data.
- Image `src` values are local data URIs. No third-party origin is contacted, so there is no tracking, no CDN supply-chain surface, and no mixed-content warning.
- No user input is stored, transmitted, or persisted. There is no authentication, no cookies, and no `localStorage` usage.

### 7. Technical Considerations

#### Dependencies

None. No package is added to [`package.json`](package.json). The page runs on browser built-ins (`document`, `Intl`, `Set`) and the tests run on Node built-ins (`node:test`, `node:assert`).

#### Configuration Changes

None. [`.hublaunch/hublaunch.config.js`](.hublaunch/hublaunch.config.js) is unrelated to this work and must not be edited.

#### Environment Variables

None.

#### API Rate Limiting

Not applicable — the page makes no network requests.

#### Error Handling Strategies

There is no I/O, no async work, and no external input, so there are no failure paths requiring `try`/`catch`. The only defensive branch in the codebase is the zero-results empty state.

### 8. Testing Requirements

#### Unit Tests

In `public/app.test.js`, using `node:test` and `node:assert`:

- [ ] `filterProducts` with an empty query and category `'All'` returns all 8 products.
- [ ] `filterProducts` with a query matching one product's name returns exactly that product.
- [ ] `filterProducts` is case-insensitive: an uppercase query returns the same result as the equivalent lowercase query.
- [ ] `filterProducts` trims the query: `'  tee  '` returns the same result as `'tee'`.
- [ ] `filterProducts` with a category other than `'All'` returns only products in that category.
- [ ] `filterProducts` combines both filters: a query plus a category that do not overlap returns an empty array.
- [ ] `filterProducts` does not mutate its input: the source array's `length` is unchanged after a filtering call.
- [ ] `formatPrice(24.99)` returns `'$24.99'`.
- [ ] `formatPrice(10)` returns `'$10.00'` (verifies two-decimal padding).

Load the data under test with `const PRODUCTS = require('./products.js');` and the functions with `const { filterProducts, formatPrice } = require('./app.js');`.

#### Integration Tests

None. The DOM rendering path is not integration-tested — doing so would require adding a DOM emulator dependency, which contradicts the zero-dependency decision. Rendering is covered by the manual checklist instead.

#### Manual Testing Checklist

1. **Setup**: none required — no install, no server.
2. **Open the page**: double-click `public/index.html`, or run `open public/index.html` on macOS.
   - Expected: a grid of 8 product cards, each showing an image, a category, a name, and a price like `$24.99`. The line above the grid reads `Showing 8 of 8 products`.
3. **Search**: type a few letters of one product's name into the search box.
   - Expected: the grid narrows as you type; the count line updates to match the number of visible cards.
4. **Category filter**: choose a category from the dropdown.
   - Expected: only products in that category remain; the count updates. The dropdown's first option is `All`.
5. **Combined filters**: set a category, then type a query that matches no product in that category.
   - Expected: `No products match your search.` is shown and the count reads `Showing 0 of 8 products`.
6. **Clear filters**: empty the search box and set the dropdown back to `All`.
   - Expected: all 8 cards return and the count reads `Showing 8 of 8 products`.
7. **Responsive layout**: narrow the browser window to roughly phone width.
   - Expected: the grid reflows to fewer columns with no horizontal scrollbar.
8. **Console check**: open the browser devtools console.
   - Expected: zero errors and zero failed network requests.
9. **Regression**: run `npm start`.
   - Expected: prints `test-mobile-app running`, unchanged.

#### Test Data Requirements

The 8 mock products in `public/products.js` are the only test data. No fixtures, mocks, or test accounts are needed.

### 9. Documentation Updates

#### User-Facing Documentation

- [ ] Add a `## Product listing page` section to [`README.md`](README.md) covering:
  - Where the page lives (`public/index.html`).
  - How to view it: open the file directly in a browser — no build step, no server, no dependencies.
  - Where the mock data lives (`public/products.js`) and that it is hardcoded on purpose so the page works over `file://`.
  - What it does: search by product name, filter by category.

#### Code Documentation

- [ ] Add a JSDoc block to `filterProducts` documenting its three parameters, the `'All'` sentinel for the category argument, and the returned new array.
- [ ] Add a JSDoc block to `formatPrice` documenting that it takes a number and returns a USD-formatted string.
- [ ] Add a one-line comment above each dual-export guard explaining that it exists so the file loads in both the browser and Node.
- [ ] Add a comment above the two `<script>` tags in `index.html` noting that the order matters (`products.js` defines the global that `app.js` reads).

#### Examples to Include

```bash
# View the page (macOS)
open public/index.html

# Run the unit tests
npm test
```

### 10. Acceptance Criteria

- [ ] **AC1**: `public/index.html`, `public/styles.css`, `public/products.js`, `public/app.js`, and `public/app.test.js` all exist.
- [ ] **AC2**: Opening `public/index.html` directly from the filesystem (a `file://` URL) renders 8 product cards with no browser console errors and no failed network requests.
- [ ] **AC3**: Each card displays an image, a category, a product name, and a price formatted as US currency with two decimals (e.g. `$24.99`).
- [ ] **AC4**: Typing in the search box filters the grid live by product name, case-insensitively.
- [ ] **AC5**: The category dropdown is populated from the product data (not hardcoded in HTML), has `All` as its first and default option, and filters the grid on change.
- [ ] **AC6**: Search and category filters apply together; when nothing matches, `No products match your search.` is displayed.
- [ ] **AC7**: The result-count line reads `Showing N of 8 products` and updates on every filter change.
- [ ] **AC8**: `npm test` passes and includes all the unit tests listed in section 8.
- [ ] **AC9**: `package.json` has zero dependencies and zero devDependencies; no bundler or framework was added.
- [ ] **AC10**: No `fetch`, no `XMLHttpRequest`, no `import`/`export`, no `<script type="module">`, and no remote URL appears anywhere in `public/`.
- [ ] **AC11**: `npm start` still prints `test-mobile-app running`; `src/index.js` is unchanged.
- [ ] **AC12**: `README.md` documents how to open the page.

#### Definition of Done

- All acceptance criteria met.
- `npm test` passes.
- The manual testing checklist in section 8 has been walked through in a real browser.
- `README.md` updated.
- No breaking changes to existing behaviour.

### 11. Dependencies & Related Work

#### Dependencies

- [ ] Depends on: nothing. This work is entirely self-contained and additive.
- [ ] Required external setup: none.

#### Blockers

- [ ] None.

#### Related Issues/PRs

- Unrelated to [`.hublaunch/plans/2026-08-10-18:00-login.md`](.hublaunch/plans/2026-08-10-18:00-login.md) (the login module plan). That plan touches `src/`; this one touches only `public/`. They can proceed independently with no merge conflict.

</details>
