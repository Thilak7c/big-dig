# Build prompt — "Big Dig" (minimal 3D open-pit mine world for kids)

You are building a small, single-file 3D "simulation world" for kids (and curious parents). The goal is learning by watching a living world, not a realistic industrial tool. Keep everything VERY minimal: a handful of machines and one clear loop.

**Build incrementally and sequentially, never in one shot.** Follow the stages in section 7 exactly, one stage per turn. After each stage, STOP and wait for my confirmation before starting the next one (see section 7). Ask me before adding anything not listed here.

---

## 0. Setup and hard rules

- **Output:** one file, `index.html`. HTML, CSS and JS all inline. Open it by double-clicking or with `npx serve .`.
- **3D:** Three.js loaded from a CDN with an import map, pinned to one version (e.g. `three@0.160.0`), plus `OrbitControls` from the same version's `examples/jsm`. No other libraries, no build step, no npm packages.
- **Everything built from simple shapes in code** (boxes, cylinders, cones, extruded shapes). No external models, textures, images or fonts (use the system font stack).
- **Low-poly, flat-shaded look** (`MeshLambertMaterial` or `MeshStandardMaterial` with `flatShading: true`). One directional "sun" light with soft shadows, plus a hemisphere light.
- **No real brand names, logos, model numbers or brand-style stripes anywhere.** The giant truck is called "Mega Hauler". Use plain construction yellow, with no black or red logo bands.
- **Units:** 1 world unit = 1 metre. Keep the machines at their real proportions (sizes below) so scale is part of the lesson.
- **Performance:** steady 60 fps on a mid-range laptop and a phone. Merge static geometry where easy, and keep shadow maps to one light at 2048 max.
- **Code style:** small and readable. Plain functions and objects, one simple state machine per vehicle, constants at the top. No frameworks, no classes unless they make it clearly simpler.
- **Facts:** use only the facts listed in section 3. Don't invent new numbers.

---

## 1. WORLD — what exists

**Location:** a make-believe open-pit mine set in green tropical hills (inspired by Malaysia). Rolling green hills ring the site, there's a soft blue sky with a few low-poly clouds, and the soil is warm reddish-orange (tropical laterite).

**Layout** (roughly 400 × 400 m, viewed from above at an angle):
- **The Pit:** a stepped, terraced hole with 3 benches (each step ~8 m high and ~15 m wide), getting deeper toward the centre. The pit floor is where digging happens.
- **Soil-layer walls ("cutaway"):** the bench walls show horizontal colour bands: dark brown topsoil on top, then orange-red clay, sand-beige, and grey rock at the bottom. This should read like a slice through a cake.
- **Haul road:** one wide, two-lane road spiralling or zig-zagging from the pit floor up the benches to the rim. Low dirt banks (berms, ~1.5 m tall) line the outer edge. Lane markings are not needed; keep a consistent "drive on the left" rule.
- **Loading spot:** on the pit floor, next to the digger, marked by a flat pad. A **queue marker** (small sign "WAIT HERE") sits ~40 m back along the road.
- **Dump area:** on the rim, a flat zone where a soil pile grows.
- **Service lane + crossing:** a narrow side track for the small pickup that crosses the haul road once near the rim, at a "STOP — big trucks first" sign.
- **Site office:** one small white cabin on the rim, with a flagpole.
- **Water point:** a small tank tower on the rim where the water truck refills.

**Machines** (low-poly, construction yellow `#F5B400` unless stated):
| Machine | Count | Real size (approx.) | Look |
|---|---|---|---|
| Big Digger (hydraulic excavator) | 1 | ~20 m long with arm, cab ~8 m up | Tracks (dark grey), rotating upper body, boom + stick + bucket, small cab, counterweight at the back |
| Mega Hauler (ultra-class haul truck) | 2 | 14.5 m long, 9.2 m wide, ~6.5 m to top of cab, 15.3 m tall with body raised | Huge dark tyres (taller than a person), canopy over cab, diagonal staircase on the front, big tipping body |
| Water truck | 1 | ~12 m long | Yellow cab + white tank, spray bar at the back |
| Flag pickup | 1 | ~5 m long | White pickup with a tall thin whip pole (~4 m) topped by an orange flag + small flashing light |

**People:** one spotter in an orange hi-vis vest standing near the loading spot holding a flag (static, idle sway only). No crowds.

---

## 2. LIFE — what moves and loops forever

All movement follows fixed paths (splines or waypoint lists). **Nothing ever overlaps or drives through anything.** Vehicles slow down and stop to give way.

**Mega Hauler loop** (each truck is a state machine):
1. `DRIVE_DOWN` the haul road to the queue marker.
2. `WAIT` at the queue marker if the loading spot is taken.
3. `REVERSE_IN` to the loading pad (beep icon above the truck; the spotter raises the flag).
4. `LOADING`: the digger does **4 scoops**. Each scoop: bucket dips into the pit face, curls, lifts, the upper body rotates, the bucket **comes in from the back/side, never over the truck's cab**, and dumps. The truck body fills visibly with a soil mound in 4 steps.
5. `HAUL_UP` the road (slower when loaded).
6. `DUMP` at the dump area: the body tilts up high (the "5-storey" moment) and soil slides onto the pile, then the body lowers.
7. `RETURN` down the road empty. Repeat.
- Only **one** truck at the loading pad at a time (a simple lock). The other waits at the queue marker.
- Trucks keep a safe following gap. A truck stops if anything is within ~25 m ahead on its lane.

**Big Digger:** idles (small arm sway) when no truck is there. Between trucks it scoops the pit face once and drops the soil on a small pile beside it (to show it's always working).

**Flag pickup:** drives a slow loop on the service lane. At the crossing it **always stops and waits until no Mega Hauler is within ~60 m**, then crosses. Its flag light blinks.

**Water truck:** drives up and down the haul road on its own slow loop, spraying white mist particles behind it. It pulls into a side bay to let Mega Haulers pass, and refills at the water tower every few laps.

**Dust:** loaded trucks make small brown dust puffs behind their wheels. Where the road was recently sprayed (a short fading "wet" darker strip), there are no puffs.

**Ground changes over time:**
- The dump pile grows a little with every trip (scale a cone/heap).
- Every 5 trips, the pit floor near the digger lowers by ~0.5 m and the digging face steps back, so the pit visibly gets deeper. The digger and loading pad move down with it.
- At max depth (~4 m below start), show a friendly banner: "Pit step finished! 🎉 Starting the next one…", fade, and reset the pit and pile to the start. The loop continues forever.

---

## 3. INTERACTION

- **Camera:** OrbitControls with pan, rotate and zoom (mouse and touch). Clamp it so the camera never goes underground or too far away. Start with a nice angled overview of the whole pit.
- **Click/tap a machine or object** → a friendly fact card (rounded, white, big emoji, 2–4 short lines, close button). Highlight the clicked object with a soft yellow outline or glow. Cards:
  - 🚛 **Mega Hauler**: "I'm one of the biggest trucks in the world! I carry about 345 tonnes in one trip — about as heavy as 50 elephants. When I tip my load, I'm about 15 metres tall — like a 5-storey building! My driver climbs a staircase to reach the seat."
  - 🦾 **Big Digger**: "I dig and load the trucks. My top half can spin all the way round on my tracks. I fill a Mega Hauler in about 4 scoops. I never swing my bucket over a truck's cab — that keeps the driver safe."
  - 🚩 **Flag Pickup**: "Mega Haulers are so big they can't see small cars close to them. That's why I carry a tall flag with a light — so the big truck drivers can spot me. On mine roads, big trucks always go first."
  - 💧 **Water Truck**: "I spray water on the roads so dust doesn't fly into the air, into people's eyes, or onto the trees nearby."
  - 🦺 **Spotter**: "I'm the spotter! I watch the area around the digger and use my flag and hand signals to tell drivers when it's safe."
  - 🪨 **Pit walls (soil layers)**: "See the stripes? That's the ground sliced open! Top: soft topsoil. Then clay, sand, and hard rock at the bottom."
  - 🏔️ **Dump pile**: "Every truckload ends up here. Watch it grow!"
- **"Secrets" toggle button** (the headline learning moment): when ON,
  - each Mega Hauler shows its **blind spot**: a semi-transparent red shape on the ground around the front-right side and rear, close to the truck;
  - a **danger circle** (red ring) shows around the digger's swing area;
  - the flag pickup's flag gets a pulsing highlight, showing it pokes up where the driver can see it.
- **"Can you spot…?" missions:** a small panel with 3 checkboxes. Clicking the right object ticks it and shows a ⭐:
  1. "Find the spotter with the flag"
  2. "Find the car that waits for big trucks"
  3. "Find the stripes in the ground"
  - When all 3 are found, show "You're a Junior Site Inspector! 🏅".

---

## 4. DASHBOARD (tiny, top-left, semi-transparent card)
Three live numbers, big and readable:
- 🚛 **Trips:** total completed dump trips
- ⬇️ **Pit depth:** current depth in metres, 1 decimal place
- 🐘 **Elephants moved:** `trips × 345 / 7`, rounded (one elephant ≈ 7 tonnes; show a tooltip "1 truckload ≈ 50 elephants")

---

## 5. EXTRAS (keep to these)
- **Speed control:** buttons ⏸ / 1× / 2× / 4× (bottom-right). All movement and counters scale with it.
- **Secrets toggle** (described above), next to the speed buttons.
- Nothing else for v1 (no day/night, no rain, no sound unless I ask later).

---

## 6. Visual style
- **Accent:** construction yellow `#F5B400` for the machines and UI highlights.
- **Soil:** topsoil `#5B3A24`, laterite clay `#C1592B`, sand `#E2C08D`, rock `#8A8A8A`, haul road `#B5653A`, site surface (dusty laterite) `#CC7144`, service lane `#D48B5C`.
- **Hills:** soft greens `#6DAA5C` / `#4E8B45`. Sky gradient from light blue to near-white.
- **Other colours:** water truck tank and site office white `#F4F4F4`; tyres and tracks `#2B2B2B`; spotter vest orange `#FF7A1A`; flag orange.
- **UI:** rounded cards, soft shadows, big friendly type (16–20 px body, 24 px numbers), lots of white space. Works on a 390 px wide phone. Title in the top-centre: **"Big Dig"** with a subtitle "A day inside a giant mine".

---

## 7. Build order — incremental, one stage at a time, with my approval

**Rules for every stage:**
- Build **only** that stage. Don't start or stub later stages early.
- Keep everything from earlier stages working. Don't rewrite what already works unless it's needed.
- When the stage is done, **STOP** and reply with:
  1. what you added (2–4 bullets),
  2. how to check it in the browser (what I should see),
  3. any problem or choice you need me to decide,
  4. the question: **"Stage X done. Shall I proceed to Stage Y: <name>?"**
- Wait for me to say yes / "next" / "proceed". If I ask for changes, make them and ask again. Never continue on your own.
- Suggest a git commit after each approved stage (e.g. `stage 1: land`).

**Stages:**
1. **Land only (nothing moves):** scene, camera, lights, OrbitControls, sky, clouds, green hills, and the empty site ground with reddish soil. No pit yet, no machines, no animation.
2. **The pit and site (still nothing moves):** the 3-bench pit with soil-layer walls, haul road, berms, loading pad, queue sign, dump area, service lane, crossing sign, site office, water tower.
3. **Machines placed, standing still:** Big Digger, 2 Mega Haulers, water truck, flag pickup, spotter, all built at the correct sizes and parked in their start spots. Still no animation.
4. **First movement — one Mega Hauler:** a single truck drives its full path (down → loading pad → up → dump area → back) on a simple loop. No loading or tipping yet.
5. **Digger + loading + tipping:** digger scoop animation (4 scoops, never over the cab), soil filling the truck body, body tipping at the dump area, reversing in, the spotter's flag raising.
6. **Second Mega Hauler:** add truck 2 with the loading-pad lock, the queue at "WAIT HERE", and the safe following gap.
7. **Flag pickup:** its service-lane loop plus the crossing give-way rule (always waits for big trucks), with the blinking flag light.
8. **Water truck + dust:** its road loop, mist spray, pulling into the side bay to let trucks pass, refills, dust puffs and the wet-road strip.
9. **Ground changes over time:** pile growth, the pit lowering every 5 trips, the finish banner and the reset.
10. **Click → fact cards:** raycast picking, highlight, all fact cards from section 3.
11. **Secrets toggle:** blind spots, danger circle, flag highlight.
12. **Dashboard + speed control:** the 3 live numbers and ⏸ / 1× / 2× / 4×.
13. **Missions:** "Can you spot…?" panel, stars, final badge.
14. **Polish:** phone layout, performance pass, then a 5-minute run at 4× to confirm no overlaps or stuck vehicles.

## 8. Done when
- [ ] Opens from one `index.html` with no console errors.
- [ ] Two Mega Haulers loop forever, take turns at the loading pad, and never overlap.
- [ ] The digger never swings its bucket over a truck cab.
- [ ] The pickup always waits for big trucks at the crossing.
- [ ] The pile grows, the pit deepens, and the world resets cleanly.
- [ ] Every listed object has a fact card, and all 3 missions can be completed.
- [ ] The Secrets toggle shows blind spots and the danger circle.
- [ ] No brand names or logos anywhere.
- [ ] Smooth on a phone browser.

---

## 9. Decisions made during the build
- Title card (top-centre) kept from Stage 1.
- 32 low-poly tropical trees on the hills only (one InstancedMesh, the two hill greens). No trees on the site.
- "3 benches" = 3 steps of 8 m; pit floor at −24 m. Bench tops 16 m wide, walls 4 m across.
- Signs are drawn larger than real life (~3 m boards) so kids can spot them.
- Local preview: `python -m http.server 8123` in this folder → http://127.0.0.1:8123/
