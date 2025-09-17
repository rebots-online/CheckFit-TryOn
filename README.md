<div align="center">
  <img width="1200" height="475" alt="GHBanner" src="https://github.com/user-attachments/assets/0aa67016-6eaf-458a-adb2-6e31a0763ed6" />
</div>

# CheckFit Try-On

CheckFit Try-On is a React + Vite experience that lets shoppers upload a reference photo, transform it into a personal model, and layer garments by calling Google Gemini image generation endpoints. The current codebase focuses on a single-page dressing room with animated transitions and wardrobe layering.

## Quick Start

**Prerequisites:** Node.js 18+

1. Install dependencies: `npm install`
2. Export your Gemini key (matches the `services/geminiService.ts` expectation): `export API_KEY=your_gemini_key`
3. Run the app locally: `npm run dev`
4. Open the provided localhost URL in your browser.

> _For hosted AI Studio builds, ensure the same environment variable is configured server-side._

## Architecture Overview

The application is built around a single React root (`App.tsx`) that switches between the onboarding flow and the dressing room. The dressing room composes a set of focused components that share state held inside `App.tsx`. Gemini image generation lives in `services/geminiService.ts`, and wardrobe data is typed via `types.ts` and `wardrobe.ts`.

```
App.tsx
├─ StartScreen (photo upload → generate model)
├─ Canvas (active pose display & controls)
├─ WardrobePanel (default + uploaded garments)
├─ OutfitStack (layer history + remove)
├─ Footer (rotating remix prompts)
└─ Gemini Service (model/try-on/pose API calls)
```

Full architecture notes plus diagrams are stored under [`docs/ARCHITECTURES/`](docs/ARCHITECTURES). The Mermaid and PlantUML files capture both current flows and the proposed billing extension modules.

## Codebase Commentary

**What works well**
- The UI composes cleanly from modular components with Tailwind utility classes.
- Gemini service wrappers centralize file conversion, API invocation, and error normalisation via `getFriendlyErrorMessage`.
- Outfit layering logic in `App.tsx` handles redo/undo and pose caching efficiently for a single session.

**Key improvement areas**
- _State management_: All session data lives in `App.tsx`. Moving wardrobe, billing, and entitlement state into a dedicated store (Zustand/Redux) will prevent prop drilling when collaboration features arrive.
- _Observability_: No analytics, error boundaries, or logging hooks. Adding telemetry providers and instrumentation middleware is critical for paid experiences.
- _Persistence_: Generated results are ephemeral. Storing wardrobe uploads and outfits (e.g., IndexedDB + remote sync) unlocks return visits.
- _Testing_: There are no automated tests or component stories; regression coverage is necessary before adding billing logic.

## Product Roadmap

1. **Foundations & Documentation** – Maintain architecture/checklist docs, clarify environment requirements, and set up linting/testing baselines.
2. **Design System & Visual Polish** – Introduce a `design-system/` token library, motion-enhanced hero sections, glassmorphism cards, and dynamic gradients for the canvas background.
3. **Billing Integration** – Implement a provider-agnostic billing layer (see below) with SKU packs, entitlement caching, and UI gates.
4. **Experience Expansion** – Add wardrobe management, export/share actions, social lookbooks, and saved outfit collections.
5. **Observability & QA** – Layer in analytics, crash reporting, integration tests, and Storybook-driven component validation.
6. **Deployment & Scaling** – Automate builds, secrets management, and rollout policies for new billing partners or AI models.

## Billing Integration Requirements

To monetize as "microsaas packs," introduce a universal billing connector that can address both centralized (RevenueCat) and decentralized (Alby Hub / BTCPay Lightning) channels.

- **Domain Models**
  - `MicrosaasPack`: `{ id, name, description, priceUSD, priceSats, entitlements: EntitlementKey[], recurring?: boolean }`
  - `EntitlementKey`: enumerations such as `try_on_slots`, `pose_variations`, `hi_res_exports`, `priority_queue`.
  - `EntitlementState`: user-specific limits and expirations.
- **Service Contracts** (new folder `services/billing/`)
  - `BillingProvider` interface with `fetchPacks()`, `purchasePack(packId)`, `syncEntitlements()`, and `restorePurchases()`.
  - `revenueCatClient.ts`: wraps RevenueCat REST APIs, maps packs to offerings, consumes webhook events for entitlement sync.
  - `albyLightningClient.ts`: issues Lightning invoices via Alby Hub or BTCPay, validates payment callbacks, and mints JWT-based entitlements.
  - `billingOrchestrator.ts`: runtime selector that caches the active provider, merges webhook events, and exposes entitlement snapshots to React.
- **Client Integration**
  - `hooks/useBilling.ts`: React hook returning `{ packs, entitlements, purchase, restore, isLoading, error }`.
  - UI elements such as `components/billing/BillingPortalButton.tsx` and `components/billing/EntitlementGate.tsx` to surface packs and protect premium flows.
  - Webhook entry points (server or edge) to ingest RevenueCat/Lightning updates.
- **Operational Requirements**
  - Centralized configuration to toggle providers per brand.
  - Audit logs and retries for entitlement synchronization.
  - Shared SKU definition file to prevent drift between providers.

## UX Enhancement Vision

- **Immersive Canvas**: Add parallax backgrounds (`components/SceneBackground.tsx`) and contextual lighting overlays responsive to pose changes.
- **Motion Design**: Wrap wardrobe and pose controls with Framer Motion variants that respond to hover/drag states; introduce hero transitions when moving from onboarding to dressing room.
- **Design System Tokens**: Centralize typography, color palettes, and spacing in a `design-system/tokens.ts` module to support white-labeled microsaas brands.
- **Personalized Wardrobe**: Persist wardrobe uploads, allow tagging, and preview outfits via a carousel or "style stack" timeline.
- **Performance Considerations**: Lazy-load heavy assets, prefetch garment thumbnails, and provide offline fallbacks for wardrobe browsing.

## Documentation & Knowledge Graph Alignment

- Architecture narrative, PlantUML, and Mermaid diagrams reside in [`docs/ARCHITECTURES/`](docs/ARCHITECTURES).
- Execution checklists live in [`docs/CHECKLISTS/`](docs/CHECKLISTS) for future agents; update progress markers in-place when tasks advance.
- Hybrid knowledge graph and Neo4j synchronization must be completed externally; record ingestion status alongside the UUID `cfit-tryon-2025a` when the graph is updated.

## Contributing

1. Follow the checklist in `docs/CHECKLISTS/20250917T222345Z-documentation-refresh.md` for this documentation cycle.
2. Run `npm run lint` and `npm run test` (when available) before committing future code changes.
3. Keep sensitive keys out of the repo; rely on environment variables and deployment secrets.
