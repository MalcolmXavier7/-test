# CLAUDE.md — Shopify Frontend Template (React)

This file provides guidance for AI assistants working in this repository.

## Repository Overview

This is a **Shopify app frontend template** built with React and Vite. It provides a ready-to-use starting point for Shopify embedded apps, integrating Shopify's App Bridge, Polaris UI library, and Shopify Admin API patterns. It is intended to be used via the Shopify CLI rather than cloned directly.

## Tech Stack

| Layer | Technology |
|---|---|
| UI framework | React 18 |
| Build tool | Vite 4 |
| Routing | React Router DOM 6 (file-based) |
| Data fetching | React Query 3 |
| UI components | Shopify Polaris 10 |
| Shopify integration | App Bridge 3 |
| Internationalization | i18next + react-i18next |
| Styling lint | Stylelint + @shopify/stylelint-polaris |
| Formatting | Prettier |
| Module format | ES Modules (`"type": "module"`) |

## Directory Structure

```
/
├── .github/
│   ├── workflows/
│   │   ├── cla.yml                   # CLA enforcement on PRs
│   │   └── merge_patch_dependencies.yml  # Auto-merge Dependabot patch updates
│   ├── dependabot.yml                # Daily npm dependency updates
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── ISSUE_TEMPLATE.md
│   └── CODE_OF_CONDUCT.md
├── assets/
│   ├── home-trophy.png               # Trophy image for home page
│   ├── empty-state.svg               # 404 state icon
│   └── index.js                      # Re-exports all assets
├── components/
│   ├── ProductsCard.jsx              # Example card: fetches/creates products
│   └── providers/
│       ├── AppBridgeProvider.jsx     # Shopify App Bridge context/config
│       ├── PolarisProvider.jsx       # Polaris UI library setup
│       └── QueryProvider.jsx         # React Query client setup
├── hooks/
│   ├── useAppQuery.js                # Authenticated React Query wrapper
│   └── useAuthenticatedFetch.js     # Shopify-authenticated fetch hook
├── locales/
│   ├── en.json                       # English translations (source)
│   ├── de.json                       # German translations
│   └── fr.json                       # French translations
├── pages/
│   ├── index.jsx                     # Home page → route "/"
│   ├── pagename.jsx                  # Example page → route "/pagename"
│   ├── NotFound.jsx                  # 404 fallback page
│   └── ExitIframe.jsx                # Shopify iframe exit handler
├── utils/
│   └── i18nUtils.js                  # i18next initialization and locale setup
├── App.jsx                           # Root component with provider tree
├── Routes.jsx                        # File-based routing logic
├── index.jsx                         # App entry point
├── index.html                        # HTML template
├── vite.config.js                    # Vite build/dev configuration
├── shopify.web.toml                  # Shopify CLI frontend config
└── translation.yml                   # Translation service config (en→de,fr)
```

## Development Workflow

### Scripts

```bash
npm run dev      # Start Vite dev server (typically via Shopify CLI)
npm run build    # Production build (outputs to dist/)
npm run coverage # Run tests with coverage via Vitest
```

> **Note:** This frontend is typically started by Shopify CLI rather than directly. The `shopify.web.toml` declares `dev` and `build` commands for CLI integration.

### Environment Variables

| Variable | Description |
|---|---|
| `SHOPIFY_API_KEY` | Required for App Bridge. Injected by Vite at build time. |
| `BACKEND_PORT` | Port the backend API server listens on (used for dev proxy). |
| `FRONTEND_PORT` | Port the Vite dev server uses. |

Vite exposes `SHOPIFY_API_KEY` as `process.env.SHOPIFY_API_KEY` (via `define` in `vite.config.js`).

### Dev Server & Proxy

The Vite dev server proxies all `/api/*` requests (and the root path) to `http://127.0.0.1:${BACKEND_PORT}`. This allows the frontend to call backend API endpoints seamlessly during development.

HMR is configured for both local (`localhost`) and remote (tunneled) environments using WebSocket or secure WebSocket respectively.

## Routing Conventions (File-based)

Routes are derived automatically from files in the `pages/` directory by `Routes.jsx`:

| File pattern | Route path |
|---|---|
| `pages/index.jsx` | `/` |
| `pages/pagename.jsx` | `/pagename` |
| `pages/[id].jsx` | `/:id` |
| `pages/[...catchAll].jsx` | `/*` |

Rules:
- All routes are lowercased.
- Dynamic segments use `[paramName]` syntax.
- Catch-all routes use `[...name]` syntax.
- Files ending in `.test.jsx` or `.test.js` are excluded from routing.
- `NotFound.jsx` is the fallback for unmatched routes.

When adding new pages, simply create a `.jsx` file in `pages/` — no route config edits needed.

## Provider Architecture

The app wraps everything in a chain of providers (outermost first):

```
PolarisProvider
  └─ BrowserRouter
       └─ AppBridgeProvider
            └─ QueryProvider
                 └─ Routes (pages)
```

- **PolarisProvider** — Supplies Polaris UI tokens and translations. Provides a custom `AppBridgeLink` for in-app navigation that integrates with Polaris's link handling.
- **AppBridgeProvider** — Initializes Shopify App Bridge using `SHOPIFY_API_KEY` and the `host` query param. Shows an error banner if either is missing. Caches config to avoid re-initialization.
- **QueryProvider** — Creates and provides a `QueryClient` instance for React Query.

## Custom Hooks

### `useAuthenticatedFetch(options?)`
Returns a `fetch`-compatible function that automatically:
- Attaches the Shopify session token as `X-Shopify-Access-Token`.
- Monitors responses for `X-Shopify-API-Request-Failure-Reauthorize` headers and redirects to the reauth URL if present.

### `useAppQuery({ url, fetchInit?, reactQueryOptions? })`
A wrapper around React Query's `useQuery` that uses `useAuthenticatedFetch` internally. Automatically disables `refetchOnWindowFocus`. Use this for all GET requests to `/api/*` endpoints.

**Example:**
```jsx
const { data, isLoading } = useAppQuery({ url: "/api/products/count" });
```

## Internationalization (i18n)

- Initialized in `utils/i18nUtils.js` before the app renders (called in `index.jsx`).
- Uses i18next with `react-i18next`. Locale JSON files are lazy-loaded from `locales/`.
- Supported locales: `en` (default), `de`, `fr`.
- Polaris component translations are also loaded per-locale.
- Polyfills `Intl.Locale` and `Intl.PluralRules` for browser compatibility.

### Adding Translations

1. Add keys to `locales/en.json` (source language).
2. The translation service (`translation.yml`) will auto-generate `de.json` and `fr.json` via PR.
3. Use `useTranslation()` hook from `react-i18next` in components.

Translation keys are organized by component/page:
```json
{
  "HomePage": { "title": "...", "body": "..." },
  "ProductsCard": { "title": "...", "createProducts": { "one": "...", "other": "..." } }
}
```

## Code Conventions

### Imports

Order imports as:
1. External libraries (React, Shopify, third-party)
2. Internal components, hooks, utilities
3. Assets (images, SVGs)

### Components

- Functional components only (React 18 hooks).
- Default exports for pages and components.
- No PropTypes validation in this template — add as needed.
- JSX files use `.jsx` extension.

### Styling

- No custom CSS files — styling is done exclusively through Shopify Polaris components and their props.
- Stylelint is configured with `@shopify/stylelint-polaris` rules if CSS is needed.
- Run `npx stylelint "**/*.css"` to lint CSS files.

### Formatting

Prettier is configured. Run:
```bash
npx prettier --write .
```

## Testing

Test runner is Vitest (referenced in the `coverage` script). DOM environment uses `jsdom`. Fetch mocking uses `vi-fetch`.

- Test files: `*.test.jsx` or `*.test.js` (these are excluded from routing).
- Run tests with coverage: `npm run coverage`.
- No example test files are included in the template — add tests alongside components.

## CI/CD

| Workflow | Trigger | Action |
|---|---|---|
| `cla.yml` | PR open/sync, issue comments | Enforces Shopify CLA signing |
| `merge_patch_dependencies.yml` | Dependabot PRs (patch) | Auto-approves and squash-merges |

Dependabot checks npm dependencies daily.

## Security Notes

- **ExitIframe page** validates redirect URIs — only Shopify domains (`*.myshopify.com`, etc.) are trusted. Untrusted URIs display a warning banner rather than redirecting.
- **useAuthenticatedFetch** handles token-based authentication transparently.
- Never expose `SHOPIFY_API_KEY` or session tokens in client-side code beyond what the Shopify integration requires.

## Key Files for Common Tasks

| Task | File(s) to edit |
|---|---|
| Add a new page/route | Create `pages/yourpage.jsx` |
| Add a shared component | Create `components/YourComponent.jsx` |
| Add a custom hook | Create `hooks/useYourHook.js` |
| Add translations | `locales/en.json` (others auto-generated) |
| Change API proxy target | `vite.config.js` |
| Add/modify navigation | `App.jsx` (NavMenu section) |
| Modify Polaris link behavior | `components/providers/PolarisProvider.jsx` |
| Change App Bridge config | `components/providers/AppBridgeProvider.jsx` |
