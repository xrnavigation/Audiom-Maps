# Audiom Maps

**A fully accessible Google Maps clone, built on the Audiom audio map surface.**

Audiom Maps recreates the Google Maps end-user experience — search, place details, menus, directions, walking turn-by-turn — on top of an audio map that is fully usable from the keyboard with a screen reader. It is backed by OpenStreetMap data and Valhalla routing, and ships to **Web (GitHub Pages), iOS, and Android** from a single web codebase wrapped in Capacitor shells.

> **Status:** Draft v2 · **Last updated:** 2026-05-05

---

## What this is (and isn't)

Audiom Maps is a **new product in a new repository**, not a feature of [`Audiom-Front-End`](https://github.com/Coughlan-Lab/Audiom-Front-End). It embeds Audiom via the documented iframe + `postMessage` API ([`documentation/embedding.md`](https://github.com/Coughlan-Lab/Audiom-Front-End/blob/master/documentation/embedding.md), [`documentation/map-parameters.md`](https://github.com/Coughlan-Lab/Audiom-Front-End/blob/master/documentation/map-parameters.md)) and reuses a small set of published `@coughlan-lab/*` libraries. It does **not** copy Audiom source, components, or stores.

### Goals

- One-product Google-Maps-style discovery flow (search → result list → detail → menu → walking directions) that is fully usable from the keyboard with a screen reader.
- **Three keystrokes** from app open to "navigating to a place": one to search a category, one to open a result, one (`D`) to start directions.
- Walking turn-by-turn that stays coherent with what Audiom is sonifying — same OSM graph for rendering and routing.
- Web, iOS, and Android from a single web codebase. No platform-only features.

### Non-Goals

- No Google Maps base map (`google.maps.Map`), markers, info windows, or polyline overlays.
- No reimplementation of Audiom features (layers, road-following, "where am I", sonar). Use the embed.
- No Street View.
- No "read aloud" buttons that duplicate what the user's screen reader already does.

---

## Design Principles — "No Junk on the Screen"

A hard constraint, enforced by `docs/checklists/ui.md`.

1. Every panel has one job. Search lists results. Detail shows one place. Directions shows one route.
2. No skip-past chrome. First focusable element is the most useful action.
3. Results before decoration — no hero photos, ad slots, or "trending" lists. Sponsored results are dropped server-side.
4. One primary keystroke per common task: `/` search, `Enter` open, `D` directions, `M` menu, `O` order, `C` call, `Esc` back.
5. Linear DOM order = reading order. Plain semantic HTML.
6. Default 10 nearest results. "Show more" is one keystroke; never auto-loads.
7. Loading and empty states are a single short polite live-region sentence.
8. Settings live in a settings screen, not inline.
9. Photos default **OFF** for the primary audience; sighted users opt in.
10. Trust the screen reader. Real `<h1>`/`<button>`/`<ul>`/`<dialog>`. ARIA is a last resort.

---

## Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                        Web app (this repo)                    │
│  ┌─────────────────┐  ┌──────────────────┐  ┌──────────────┐ │
│  │  Search /       │  │  Place / Route   │  │  Settings    │ │
│  │  Category UI    │  │  Detail UI       │  │              │ │
│  └────────┬────────┘  └────────┬─────────┘  └──────────────┘ │
│           │                    │                              │
│           ▼                    ▼                              │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │ Service layer (TS modules)                                │ │
│  │   - GooglePlacesClient (autocomplete + Places (New) REST) │ │
│  │   - ValhallaClient (OSM walking routing)                  │ │
│  │   - OsmEnrichmentService (Google ↔ OSM matching)          │ │
│  │   - DeliveryLinkResolver (DoorDash, UberEats)             │ │
│  │   - SavedPlacesClient (Audiom maps API)                   │ │
│  │   - GeolocationService (Web Geolocation API)              │ │
│  └─────────────────────────┬────────────────────────────────┘ │
│                            ▼                                   │
│                Audiom Embed (iframe + postMessage)             │
└───────────────────────────────────────────────────────────────┘
```

The Audiom Embed is the audio map surface; this repo is the discovery and directions UI that drives it.

---

## APIs and Engines

| Concern | Engine |
|---|---|
| Place search & details | Google Places (New) — autocomplete widget + REST for details and menus |
| Walking routes | Valhalla on the same OpenStreetMap graph Audiom renders |
| OSM matching | Internal `OsmEnrichmentService` keyed on Google place IDs |
| Delivery deep links | `DeliveryLinkResolver` (DoorDash, UberEats) |
| Saved places | Audiom maps API |
| Audio map | Audiom Embed (iframe + `postMessage`) |
| Geolocation | Web Geolocation API |

---

## Keyboard shortcuts

| Key | Action |
|---|---|
| `/` | Focus search |
| `Enter` | Open the focused result |
| `D` | Start walking directions to the focused place |
| `M` | Open menu (for restaurants and similar) |
| `O` | Order (delivery deep link) |
| `C` | Call |
| `Esc` | Back |

Common tasks should always be one keystroke. Common flows should always be three.

---

## Cross-platform strategy

A single web codebase ships to:

- **Web** — GitHub Pages
- **iOS** — Capacitor shell
- **Android** — Capacitor shell

No platform-only features in v1.

---

## Phased rollout

Phase issues will be created and tracked from the umbrella spec ([#1](../../issues/1)).

---

## Repository layout (planned)

```
.
├── src/                  # Web app (TypeScript, MobX state)
├── docs/
│   └── checklists/
│       └── ui.md         # "No junk on the screen" UI checklist
└── ios/, android/        # Capacitor shells
```

---

## License

TBD.

## References

- [`Coughlan-Lab/Audiom-Front-End`](https://github.com/Coughlan-Lab/Audiom-Front-End) — the Audiom platform this product embeds.
- [Audiom Embed API — `documentation/embedding.md`](https://github.com/Coughlan-Lab/Audiom-Front-End/blob/master/documentation/embedding.md)
- [Audiom map parameters — `documentation/map-parameters.md`](https://github.com/Coughlan-Lab/Audiom-Front-End/blob/master/documentation/map-parameters.md)

## Acknowledgements

- [Audiom](https://github.com/Coughlan-Lab/Audiom-Front-End) — accessible audio mapping platform from the Coughlan Lab.
- [OpenStreetMap](https://www.openstreetmap.org) contributors — base geographic data.
- [Valhalla](https://valhalla.github.io/valhalla/) — open-source routing engine.
