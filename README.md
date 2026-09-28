# VITALIS — Healthcare, Reimagined.

A cinematic, fully-3D concept website for a fictional premium clinic, built as a
demo for **Nexa Web Works**.

> **All content in this project is fictional demonstration content.** The clinic,
> its specialists, facilities, clinical figures and prices are invented. Nothing
> here is real medical data, medical advice, or a description of a real
> healthcare provider.

---

## Running it

```bash
npm install
npm run dev        # http://localhost:5173
```

```bash
npm run build      # typecheck + production build to dist/
npm run preview    # serve the production build
npm run typecheck  # TypeScript only
```

Requires Node 18+.

---

## Stack

| Concern | Choice |
| --- | --- |
| Framework | React 18 + TypeScript (strict) |
| Build | Vite 5 |
| 3D | Three.js + React Three Fiber + Drei |
| Animation | Framer Motion + GSAP |
| Routing | React Router 6 |
| Styling | Hand-written CSS design system (no CSS framework) |

No backend, no database. The booking flow and the dashboard both run on local
mock data.

---

## Architecture

```
src/
  data/          Typed content source — all fictional copy in one place
  lib/           quality tiers, hooks, math, ambience (silent by default)
  three/         R3F canvas, scene graph, camera rig, lighting
  components/
    ui/          Buttons, panels, nav, cursor, preloader, reveals
    booking/     Modular steps for the appointment flow
    dashboard/   Console panels, charts, tables, views
  scenes/        The ten homepage scenes
  routes/        One file per route
  styles/        Design system + page + dashboard stylesheets
```

### Performance strategy

`src/lib/quality.ts` resolves a device tier (`low` / `medium` / `high`) from
core count, memory, pointer type, data-saver preference, reduced-motion and
viewport size. That tier drives a single `TIER_SETTINGS` budget consumed by
`TierCanvas`, so scenes never make their own rendering decisions:

| | low | medium | high |
| --- | --- | --- | --- |
| Particles | 260 | 900 | 2200 |
| DPR cap | 1 | 1.5 | 2 |
| Shadows | off | 1024 | 2048 |
| Antialiasing | off | on | on |

On coarse-pointer devices an FPS probe runs for the first two seconds and
downgrades to `low` if the frame budget is blown. Reduced-motion users get a
static, fully-lit scene rendered on demand rather than an animation loop.

Every 3D element is functionally tied to the clinic experience — the core
sphere is the patient record, nodes are the care network, the orbit ring is the
service directory. There is no decorative geometry.

### The 3D environment

One `<Canvas>` is mounted for the whole homepage. `CameraRig` samples a
keyframe track against global scroll progress and interpolates with
frame-rate-independent damping, so all ten scenes read as one continuous camera
move through the building. Scroll and pointer values are written to refs and
read inside `useFrame`, so scrolling never triggers a React render.

The care network and service explorer are separate canvases, lazily imported so
the 3D bundle never blocks first paint on inner routes.

### Audio

`src/lib/ambience.ts` contains a complete Web Audio architecture (master gain,
mute state, generated drone bed) but **nothing is ever started automatically**.
It is wired for a future opt-in toggle and deliberately silent by default.

---

## Routes

| Route | Contents |
| --- | --- |
| `/` | Ten connected 3D scenes over a persistent environment |
| `/doctors`, `/doctors/:id` | Filterable roster, full profiles |
| `/services`, `/services/:id` | Discipline index, detail with its specialists |
| `/appointments` | Six-step booking flow |
| `/wellness` | Six prevention programmes |
| `/packages` | Five health packages |
| `/about`, `/contact` | Clinic concept, contact routes |
| `/dashboard/*` | Nine-section operational console |

---

## Accessibility notes

- Every 3D interactive scene has a keyboard-operable DOM equivalent (the care
  node list mirrors the 3D graph; the service rail mirrors the orbit ring).
- `prefers-reduced-motion` disables animation loops, the custom cursor and the
  magnetic button transforms.
- The facilities flythrough collapses to a vertical stack below 760px rather
  than forcing horizontal scrolling against vertical page scroll.
- The facilities rail steps, booking flow and dashboard nav are fully
  keyboard-navigable with visible focus rings.

---

## Known limitations

- The custom cursor is suppressed on touch devices by design; the native
  cursor is used instead.
- The facilities flythrough relies on page scroll, so it is disabled on mobile
  in favour of a stacked list.
- Dashboard search filters the patient list only; it is not a global search.
- Charts are hand-built SVG rather than a charting library, to avoid adding a
  dependency for a fixed set of mock datasets.

---

Concept and build by **Nexa Web Works**.
