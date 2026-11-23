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

**Location:** a make-believe **open-pit coal mine** set in green tropical hills (Malaysia/Borneo-inspired). Rolling green hills ring the site (outside it), there's a soft blue sky with a few low-poly clouds, and inside the site everything is dusty dark grey.

**How it works (for kids):** the rock on top of the coal ("overburden") is dug away first and taken to the **waste dump**; then the black **coal seam** is dug and taken to the **coal stockpile**. Most truckloads are rock, not coal.

**Layout** (roughly 400 × 400 m, viewed from above at an angle):
- **The Pit:** a stepped, terraced hole with 3 benches (each step ~8 m high and ~15 m wide), getting deeper toward the centre. The pit floor is where digging happens.
- **Soil-layer walls ("cutaway"):** the bench walls show horizontal colour bands, top → bottom: dark topsoil `#5B3A24` (thin) · red-brown clay `#A0522D` · beige sandstone overburden `#C8B58F` (thickest) · **BLACK COAL SEAM** `#1C1C1C` (thick, clearly the "treasure" stripe, the lower half of the bottom bench wall, slight sheen) · grey floor rock `#6E6E6E` below it. This should read like a slice through a cake. The pit floor sits on the coal seam, so the loading area looks black-grey.
- **Haul road:** one wide, two-lane road spiralling or zig-zagging from the pit floor up the benches to the rim. Low dirt banks (berms, ~1.5 m tall) line the outer edge. Lane markings are not needed; keep a consistent "drive on the left" rule.
- **Loading spot:** on the pit floor, next to the digger, marked by a flat pad. A **queue marker** (small sign "WAIT HERE") sits ~40 m back along the road.
- **Two destinations** (the haul road reaches both):
  - **Coal stockpile:** a black cone pile on the rim next to a small grey crusher shed, with a short sloped conveyor running from the crusher to the top of the pile.
  - **Waste dump:** a beige flat-topped dump, further from the pit, with an access ramp and a safety berm along its crest. Trucks tip over the edge from the top.
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
4. `LOADING`: the digger does **4 scoops**. Each scoop: bucket dips into the pit face, curls, lifts, the upper body rotates, the bucket **comes in from the back/side, never over the truck's cab**, and dumps. The truck body fills visibly in 4 steps with a **lumpy heap made of a few low-poly chunks** (like real photos), not a smooth mound.
5. `HAUL_UP` the road (slower when loaded).
6. `DUMP` at its destination (coal → coal stockpile, rock → waste dump): the body tilts up high (the "5-storey" moment) and the load slides onto the pile, then the body lowers.
7. `RETURN` down the road empty. Repeat.
- **Loads:** every trip is either **COAL** (black lumpy heap `#1C1C1C`) or **ROCK** (beige lumpy heap `#C8B58F`). Pattern: 1 coal load after every 2 rock loads (rock, rock, coal, …).
- Only **one** truck at the loading pad at a time (a simple lock). The other waits at the queue marker.
- Trucks keep a safe following gap. A truck stops if anything is within ~25 m ahead on its lane.

**Big Digger:** idles (small arm sway) when no truck is there. Between trucks it scoops the pit face once and drops the soil on a small pile beside it (to show it's always working). It digs the beige rock for rock loads and the black seam for coal loads; the bucket contents match the load colour.

**Flag pickup:** drives a slow loop on the service lane. At the crossing it **always stops and waits until no Mega Hauler is within ~60 m**, then crosses. Its flag light blinks.

**Water truck:** drives up and down the haul road on its own slow loop, spraying white mist particles behind it. It pulls into a side bay to let Mega Haulers pass, and refills at the water tower every few laps.

**Dust:** loaded trucks make small brown dust puffs behind their wheels. Where the road was recently sprayed (a short fading "wet" darker strip), there are no puffs.

**Ground changes over time:**
- The coal stockpile and the waste dump grow separately, each a little with every one of its own loads (scale a cone/heap).
- Every 5 trips, the pit floor near the digger lowers by ~0.5 m and the digging face steps back, so the pit visibly gets deeper. The digger and loading pad move down with it.
- At max depth (~4 m below start), show a friendly banner: "Pit step finished! 🎉 Starting the next one…" (the banner also shows the pit depth reached), fade, and reset the pit and both piles to the start. The loop continues forever.

---

## 3. INTERACTION

- **Camera:** OrbitControls with pan, rotate and zoom (mouse and touch). Clamp it so the camera never goes underground or too far away. Start with a nice angled overview of the whole pit.
- **Click/tap a machine or object** → a friendly fact card (rounded, white, big emoji, 2–4 short lines, close button). Highlight the clicked object with a soft yellow outline or glow. Cards:
  - 🚛 **Mega Hauler**: "I'm one of the biggest trucks in the world! One load weighs about as much as 50 elephants. When I tip, I'm as tall as a 5-storey building. My driver climbs stairs to get in!"
  - 🦾 **Big Digger**: "I dig the rock first, then the black coal underneath. I fill a Mega Hauler in about 4 scoops. I never swing my bucket over a truck's cab — that keeps the driver safe."
  - 🚩 **Flag Pickup**: "Mega Haulers are so big they can't see small cars close to them. That's why I carry a tall flag with a light — so the big truck drivers can spot me. On mine roads, big trucks always go first."
  - 💧 **Water Truck**: "Coal mines are very dusty. I spray water on the roads so dust doesn't fly into the air."
  - 🦺 **Spotter**: "I'm the spotter! I watch the area around the digger and use my flag and hand signals to tell drivers when it's safe."
  - ⚫ **Coal seam (pit walls)**: "This black stripe is coal! It was made from plants that were buried and squashed for millions of years. Above it is rock we must move first."
  - ⚫ **Coal stockpile**: "Trucks tip the coal into a hopper in the ground. The crusher breaks it into small pieces, and a conveyor belt carries it up to the pile. Later it's taken away — much of it is burned in power stations to make electricity."
  - 🪨 **Waste dump**: "This isn't rubbish — it's the rock that was on top of the coal. One day it can be put back and planted with trees."
- **"Secrets" toggle button** (the headline learning moment): when ON,
  - each Mega Hauler shows its **blind spot**: a semi-transparent red shape on the ground around the front-right side and rear, close to the truck;
  - a **danger circle** (red ring) shows around the digger's swing area;
  - the flag pickup's flag gets a pulsing highlight, showing it pokes up where the driver can see it.
- **"Can you spot…?" missions:** a small panel with 3 checkboxes. Clicking the right object ticks it and shows a ⭐:
  1. "Find the spotter with the flag"
  2. "Find the car that waits for big trucks"
  3. "Find the black coal stripe in the ground"
  - When all 3 are found, show "You're a Junior Site Inspector! 🏅".

---

## 4. DASHBOARD (tiny, top-left, semi-transparent card)
Three live numbers, big and readable:
- ⚫ **Coal loads:** completed coal trips
- 🪨 **Rock loads:** completed rock trips
- 🐘 **Elephants moved:** `total trips × 345 / 7`, rounded (one elephant ≈ 7 tonnes; show a tooltip "1 truckload ≈ 50 elephants")
- (Pit depth is shown in the finish banner instead.)

---

## 5. EXTRAS (keep to these)
- **Speed control:** buttons ⏸ / 1× / 2× / 4× (bottom-right). All movement and counters scale with it.
- **Secrets toggle** (described above), next to the speed buttons.
- Nothing else for v1 (no day/night, no rain, no sound unless I ask later).

---

## 6. Visual style
- **Accent:** construction yellow `#F5B400` for the machines and UI highlights.
- **Site (inside):** site surface / pit floor dusty dark grey `#4A4A48`; haul road slightly lighter `#5C5C5A`; service lane `#7A746A`. No reddish laterite colours on the site.
- **Pit wall layers:** topsoil `#5B3A24`, red-brown clay `#A0522D`, sandstone overburden `#C8B58F`, coal seam `#1C1C1C`, floor rock `#6E6E6E`.
- **Loads and piles:** coal `#1C1C1C`, waste rock `#C8B58F`.
- **Hills:** soft greens `#6DAA5C` / `#4E8B45`. Sky gradient from light blue to near-white.
- **Other colours:** water truck tank and site office white `#F4F4F4`; tyres and tracks `#2B2B2B`; spotter vest orange `#FF7A1A`; flag orange.
- **UI:** rounded cards, soft shadows, big friendly type (16–20 px body, 24 px numbers), lots of white space. Works on a 390 px wide phone. Title in the top-centre: **"Big Dig"** with a subtitle "A day inside a giant coal mine".

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
   - **2b. Coal mine:** recolour the site to dusty grey, coal-mine wall layers, and replace the dump area with the coal stockpile (+ crusher shed + conveyor) and the waste dump. All later stages use the coal-mine rules: Stage 5 load colours, Stage 9 two piles, Stage 10 new cards, Stage 12 new dashboard, Stage 13 new mission.
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
- [ ] Coal and rock loads follow the 2 rock : 1 coal pattern and go to the right pile.
- [ ] Both piles grow, the pit deepens, and the world resets cleanly.
- [ ] Every listed object has a fact card, and all 3 missions can be completed.
- [ ] The Secrets toggle shows blind spots and the danger circle.
- [ ] No brand names or logos anywhere.
- [ ] Smooth on a phone browser.

---

## 9. Decisions made during the build
- Title card (top-centre) kept from Stage 1.
- 32 low-poly tropical trees on the hills only (one InstancedMesh, the two hill greens). No trees on the site.
- "3 benches" = 3 steps of 8 m; pit floor at −24 m. Bench tops 16 m wide, walls 4 m across.
- Layer depths: topsoil 0 to −1.5 m, clay to −5 m, sandstone to −20 m, coal seam −20 to −26 m (the lower half of the bottom wall, and under the floor), floor rock below −26 m. The digging face therefore shows a beige band (−16 to −20 m) above a black band (−20 to −24 m): rock scoops dig the beige band, coal scoops the black band. Flat bench tops and the floor are dusted grey; the walls show the clean stripes.
- Haul road: pit → ramp → rim, then splits at a junction: north-east branch to the coal yard, south-east branch to the waste dump. The service lane loops around the coal yard and crosses only the coal branch, once.
- Stage 4b road tweaks: haul road 30 m wide; berms sit on its outer 2.5 m, so 26 m is usable. Trucks keep left, 6.5 m from the centre line (≈3.7 m between two passing trucks). The bend at the foot of the ramp is a 32 m-radius arc; the ramp is 280 m at ~8.6%.
- Waste dump = a real end-tipping dump: flat top (crest 48 × 48 m, 7 m high) with a 60 m access ramp and a 1.5 m safety berm round the crest. Trucks drive up, turn on top, reverse to the south crest berm and tip over the edge; the dump grows outward from that edge.
- Coal yard = a hopper pocket set in the ground beside the crusher shed, with a concrete stop wall (the tipping point's safety berm) and apron. Trucks circle the turning pad, then reverse straight up to the wall and tip into the pocket.
- `TIME_SCALE` (default 4.6): machines move at real speeds, sped up so one rock trip (load, waste dump, back) takes ~60 s at 1×; a coal trip takes ~51 s. All movement and timers use it; the speed buttons multiply it.
- Stage 5: the Big Digger sits at (16, 2) on the pit floor, between the loading pad and the east wall. Rock scoops come from the beige band of that wall, coal scoops from the black seam below it. The truck parks with its cab at the far front-left, so the bucket always swings in over the rear of the body.
- Signs are drawn larger than real life (~3 m boards) so kids can spot them.
- Local preview: `python -m http.server 8123` in this folder → http://127.0.0.1:8123/
- Stage 6 traffic: one global load counter (rock, rock, coal) shared by both trucks. Three one-truck zones, each with a hold point where a truck waits if the zone is taken: the loading pad (hold at "WAIT HERE", released once the leaving truck is 55 m out), the dump top (hold at the foot of the dump ramp) and the coal pad (hold before the service-lane crossing, never on it). Trucks keep 25 m clear to the truck ahead in their own lane; oncoming trucks in the other lane are ignored, so they pass keeping left. The digger's idle sway is left for Stage 14; no spare scoop between trucks.
- Stage 7 flag pickup: drives the service lane at 1.7 m/s, one lap ~42 s at 1x. The lane follows a circle round the coal yard but runs straight across the coal road, turned 18° off square (72° to the road) so its far-side run-in keeps 3.6 m clear of the waste road (crossing ~43 m from stop line to tailgate clear). Each lap it stops for 5 s beside the site office, which now sits just north of the lane loop at (148, -176), and always stops at the stop line (2 m before the road edge) for at least 1.5 s, waiting until no Mega Hauler is within 60 m of the crossing (that includes a truck at the coal-pad hold point), then crosses at up to 8 m/s without stopping until its tailgate is clear. Big trucks never stop for it. The light on its whip pole blinks.
- Stage 8 water truck: drives up and down the ramp at 5 m/s (~18 km/h), keep-left, with the same 25 m following gap as the haulers (haulers keep 25 m behind it too). It turns round off the road at both ends: a loop (R 11 m) in the empty south-west corner of the pit floor, clear of the loading pad, its approach and WAIT HERE, and a loop (R 10 m) on the flat rim beside the ramp top, round the water tower, which moved to (70, -110). Those loops are where haulers get past it; there are no other bays. At each loop it waits at a stop line and re-enters the road only when no hauler is within 60 m of the crossing and none would reach it within 18 s. It refills under the spout every 3rd lap (6 s at 1x). Spray: white mist from the spray bar while it moves on the road, and a darker wet strip (9 m wide) that dries over 30 s at 1x. Dust: moving haulers (over 1.5 m/s) make grey-brown puffs behind their rear wheels, none on wet road. Mist and dust share one pooled particle system capped at 300.
- Stage 9 ground changes: a cycle is 24 loads; a load counts when it is tipped. Coal loads grow the stockpile under the conveyor head (same shape, 5.5 m to 10 m high over the cycle's 8 coal loads, capped to stay clear of the conveyor legs and the service lane). Each rock load adds two low mounds (2-3 m) along the toe of the waste dump's south slope, below the crest berm, so the dump top is untouched. Pit progress: the digger (which never moves) cuts an arc-shaped bite into the east bottom wall, centred on itself, with its foot 11 m from the digger (3 m into the wall straight ahead). Every 3 loads the bite widens sideways; after 8 steps it spans about 20 m along the top of the wall (half-angle 43 degrees). Both soil bands follow. Bucket reach stays 14.27 m of 14.8 m at every step. After the 24th load the finished state holds for 20 s, then the banner shows for 5 s, then the piles and counters reset. The wall is put back as soon as the bucket is away from the face. Loads tipped during the hold or the banner count toward the next cycle. The rock, rock, coal pattern carries on across the reset. Freshly cut wall is 30% darker, like damp rock: fully dark in the step it was cut, half as dark one step later, normal after two. Start camera (changed in Stage 9 at the user's request): from the south-west, looking north-east at the digging face, the digger and the loading pad, with the ramp and the coal pile beyond. It aims at (-10, -20, -40) from azimuth 145° and elevation 28°. Its distance is max(148, 205 / aspect^0.85) m, so portrait phones start further back. The orbit limits are unchanged. This replaces the whole-pit overview from section 3.
- Stage 10 fact cards: a tap is one finger, under 8 px of movement and under 0.5 s; drags, pinches and wheel only move the camera. Machines and the spotter have invisible hit spheres that move with them (hauler 8 m, digger 9 m, water truck 6 m, pickup 3.5 m, spotter 2 m, coal pile ~0.9 x its radius). On screen each is at least 48 px across, so it keeps ~44 px where two touch; where several overlap, the one whose centre is nearest the tap (relative to its size) wins. The coal yard is also picked by its meshes (pile, conveyor, legs, crusher shed with hopper), the waste dump by its ground and mounds, and the coal seam by a steep wall point between -20 and -24 m. Machines win over the ground features. Nothing is picked through the ground (a ray march on groundAt). Highlight: the object's meshes swap to a shared copy of their material with a softly pulsing yellow glow (lights that already glow are left alone); the coal seam glows yellow through the soil shader. The card is white and rounded, with a big emoji, the title and the CLAUDE.md text, plus a 44 px close button. It sits at the right on wide screens and at the bottom on screens up to 600 px wide. Tapping empty ground also closes it. On a pick the camera eases for 1 s to look at the object from 35% closer along the same line (at least 40 m away and 8 m above the ground); any user drag stops the ease, and it never chases a moving vehicle. The simulation keeps running.
