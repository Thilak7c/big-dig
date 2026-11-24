# Big Dig

A small 3D world of a make-believe open-pit coal mine, for kids and curious grown-ups. Two giant
Mega Haulers, a Big Digger, a water truck, a flag pickup and a spotter work in a loop that runs forever.
You watch, tap things to learn about them, and find hidden safety secrets.

Everything is in one file, `index.html`: Three.js from a CDN, every shape built in code, and no other libraries.

## Run it

```sh
npx serve .
```

Then open the address it prints, for example http://localhost:3000. Any static file server works,
such as `python -m http.server`. The page needs an internet connection to load Three.js.

## Controls

| Do this | What happens |
|---|---|
| Drag, pinch, scroll | Turn, zoom and move the camera |
| Tap or click a machine, the coal seam or a pile | A fact card opens and the thing glows |
| 🏠 Overview | Glides back to the start view |
| ⏸ 1× 2× 4× | Pause or speed up the mine (you can still look around and tap while paused) |
| 🔍 Secrets | Shows the haulers' blind spots, the digger's danger circle and the pickup's flag |
| 🎯 Can you spot…? | Three things to find; find all three to become a Junior Site Inspector 🏅 |

The dashboard counts coal loads, rock loads and how many elephants' weight has been moved
(one load ≈ 50 elephants).

## What kids can learn

- **Most of a coal mine is moving rock.** The rock on top ("overburden") goes to the waste dump first:
  2 rock loads for every 1 coal load.
- **The coal is a black stripe in the ground**, made from plants squashed for millions of years.
- **The machines are huge.** A Mega Hauler is as tall as a 5-storey building when it tips, and its
  driver climbs stairs to get in.
- **Big machines can't see everything.** The red blind spots show where a hauler driver can't see. That's
  why the pickup has a tall flag, and why big trucks always go first.
- **Safety rules keep people safe:** the digger never swings over a truck's cab, people stay out of the
  digger's swing area, and the spotter signals when it's safe.
- **Mines change the land, and can give it back.** The waste rock can be put back and planted with trees,
  and the water truck keeps the dust down.

The machines, the place and the numbers are simplified, and no real brands are shown.
