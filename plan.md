# Budgetly implementation plan

## Product scope

Budgetly is a mobile-first personal budgeting app that runs in Expo Web and native Expo surfaces. It keeps the user’s financial data in the browser/device through AsyncStorage, with no account or server persistence required for the requested workflow.

Required outcomes:

- Log income and expenses with amount, date, category, and optional note.
- Maintain reusable income and expense categories.
- Set a monthly budget for each expense category and see spent, budget, and remaining amounts.
- Provide a dashboard with income, expenses, total budget, remaining cash flow, spending by category, a month-over-month spending trend chart, and plain-English insights.
- Support CSV import with validation plus an error summary and CSV export.
- Keep all user-created data local to the browser/device.

## Implementation approach

- Use the fixed Expo Router / React Native / TypeScript / NativeWind starter. Keep the existing server/auth scaffold untouched because this product does not require login or server persistence.
- Add a typed local domain model in `lib/budget-store.ts` for transactions, monthly budgets, categories, derived metrics, CSV serialization, and CSV parsing.
- Add `hooks/use-budget-data.ts` as the single state boundary. It hydrates from `@react-native-async-storage/async-storage`, persists after changes, and exposes transaction, budget, category, import, and reset actions.
- Build four portrait-friendly routes under `app/(tabs)`: Overview, Activity, Budgets, and Data. Each route shares the same local store and uses compact one-handed controls.
- Use browser APIs only behind `Platform.OS === "web"` guards for CSV file selection and downloads. Native export falls back to the platform Share API; the core data model remains portable.
- Use deterministic bar charts built from React Native `View` elements instead of an additional chart dependency, keeping web and native rendering consistent.
- Use seeded demo data on first launch so the dashboard is immediately legible; first-launch data is saved locally and can be replaced through Data.
- Add `public/manus-routes.json` with the four page routes before starting Metro, as required by the managed webdev workflow.

## Project structure

- `app/(tabs)/_layout.tsx`: bottom-tab navigation and app-level navigation styling.
- `app/(tabs)/index.tsx`: selected-month dashboard, summary cards, category spending bars, trend bars, and insights.
- `app/(tabs)/activity.tsx`: add transaction form, filters, and recent transaction list.
- `app/(tabs)/budgets.tsx`: monthly budget editor, category progress rows, and category management.
- `app/(tabs)/data.tsx`: CSV import/export, format guidance, validation feedback, and local-data reset.
- `components/budget-ui.tsx`: reusable visual primitives for cards, progress bars, section headings, and icon badges.
- `lib/budget-store.ts`: types, calculations, seed data, CSV helpers, month utilities, and import validation.
- `hooks/use-budget-data.ts`: persisted state and mutation actions.
- `assets/images/*`: Budgetly launcher, splash, adaptive, and favicon artwork.
- `plan.md`, `TODO.md`: implementation decisions and outcome tracking.

## Design direction

### Design movement

Quietly premium editorial finance: a composed, paper-like workspace that treats budgeting as a weekly ritual rather than a noisy spreadsheet.

### Core principles

1. **Clarity before control** — show the one number that matters first, then expose detail progressively.
2. **Warm accountability** — use coral for attention and mint for progress without shame-driven red screens.
3. **Tactile structure** — cards, ruled dividers, compact pills, and strong type create a physical journal feel.
4. **One-handed rhythm** — every high-frequency action is reachable in the lower half of the viewport and uses large tap targets.

### Color philosophy

The base is a warm oat paper (`#F7F5EE`) rather than clinical white. Ink navy (`#10212B`) gives financial numbers authority, sea-glass teal (`#2D9C8A`) signals healthy progress, coral (`#F26B5E`) flags attention, and muted lilac (`#8C83A6`) separates secondary analytics. The ownable brand color is **Budgetly Sea Glass** (`#2D9C8A`), a calm green-teal that reads as optimistic but not childish.

### Layout paradigm

A vertically editorial dashboard: an oversized month-and-balance header, a single horizontal health rail, then stacked sections that read like a personal money journal. Avoid dense desktop grids; use full-width cards and horizontal scroll chips so the design stays natural on a phone.

### Signature elements

- A rounded “money mood” rail that combines the remaining amount and a thin progress line.
- Small uppercase section labels with a coral rule, echoing annotated paper notes.
- Category rows with capsule icons and bold right-aligned amounts, designed to scan in one glance.

### Interaction and animation

Interactions should feel deliberate and light: pressed cards reduce opacity briefly, transaction creation updates the dashboard immediately, and progress bars interpolate through React Native layout changes without decorative motion. No auto-advancing carousels or attention-grabbing animations. Native tabs use the starter haptic button behavior.

### Typography system

Use the platform system sans for reliable native rendering. Large balances are bold and tightly spaced; section labels use 11px uppercase tracking; body copy is 14–16px with muted navy-gray for supporting context. Money is always right aligned in lists and uses tabular-looking weight through `fontVariant` where available.

### Brand essence and voice

**Budgetly is the calm money check-in for people who want a clearer month without becoming spreadsheet people.** Personality: grounded, observant, encouraging.

- Headlines: “Your month, in focus.”
- Microcopy: “A little context goes a long way.”
- CTA voice: “Log it”, “Set the guardrail”, and “Bring in a CSV”.

### Wordmark and icon

The wordmark is `Budgetly` in bold ink with a small sea-glass square “ledger tab” mark. The launcher icon uses a sea-glass square with a cream ledger line and coral dot, with no baked-in rounded corners.

## Constraints and trade-offs

- The app is local-first by design; CSV is the portability and backup path.
- Native CSV import is represented by the same import model and web file picker; browser Preview is the fully interactive import surface because the request explicitly calls for browser-local data.
- No external images, APIs, login, or database are needed, which keeps the app private and functional offline after the first load.
