---
name: machin-game-demo-anim
description: Build, run, and modify machin-game-demo-anim — a 2D procedural flow-field animation in machin (MFL) via raylib. Use when working on this repo, or as the worked example of native math (sin/cos/atan2/hypot/pi) driving procedural motion, particle state in parallel slices, and HSV color via a struct-returning FFI call.
---

# machin-game-demo-anim

A 2D procedural flow-field animation: 1200 particles swirling around three drifting vortex centers, written in [machin](https://github.com/javimosch/machin) (MFL) and drawn with raylib. It is the reference example for **procedural animation / native math** in machin.

> Shared game-dev setup, build-and-verify workflow, and cross-cutting gotchas live in the canonical **[machin-gamedev skill](https://github.com/javimosch/machin/blob/main/skills/machin-gamedev/SKILL.md)**. This file is the specifics.

## Build & run

```bash
./build.sh                 # machin encode anim.src -> anim.mfl, then machin build -> ./machin-game-demo-anim
./machin-game-demo-anim
```

Needs `machin` **v0.46.0+** (native math), a C compiler, **raylib**, and a display. `build.sh` prefers a system raylib, else vendors the prebuilt static release into `vendor/` (no root).

## What it exercises

- **Native math** (v0.46.0): `sin cos atan2 hypot fmod pi` — the whole field is trig over time. No `extern "m"` needed.
- **Struct-return FFI:** `ColorFromHSV(f32,f32,f32) Color` returns a by-value `Color`; hue comes from each particle's flow direction.
- **No new machin feature** — pure composition on the math suite + raylib 2D + slices. That's the intended outcome for this one.

## Patterns worth copying

- **Tangential pull:** to make particles *circle* a point rather than fall into it, take the bearing to the center with `atan2(dy, dx)` and add `pi/2`; velocity is `cos`/`sin` of that, weighted by `1/distance` (`hypot` + a small epsilon so it never blows up).
- **Drifting centers:** animate attractors on Lissajous paths — `cx = wc + A*sin(t*f)`, `cy = hc + B*cos(t*g)` with different freqs/phases per center.
- **Speed cap:** `sp := hypot(vx, vy); if sp > MAX { vx = vx/sp*MAX; vy = vy/sp*MAX }` keeps the field smooth.
- **Direction → hue:** `fmod((atan2(vy,vx) + pi()) / TAU * 360.0 + t*40.0, 360.0)` maps flow angle to the color wheel and slowly cycles it.
- **Particle state in parallel `[]float`s** (`px`, `py`); `px[i] = x` writes back. Seed with `rndf(max)` = `float((byte_at(rand_bytes(2),0) << 8 | byte_at(...,1)) % max)`.
- **int/float discipline:** world coords are `float`; geometry constants return `float` (`W() { n = 900.0 }`); cross to `int` only at the FFI (`DrawCircle(int(x), int(y), 1.6, c)`, `InitWindow(int(W()), ...)`). See the gamedev skill for the full rule.

## Modifying

- **Look:** particle count `N()`, `DrawCircle` radius, the `0.65`/`1.0` HSV saturation/value, the speed cap `3.2`.
- **Field:** number of centers and their amplitudes/frequencies/phases; the `70.0 / d` pull strength; flip `+ pi/2` to `- pi/2` to reverse the swirl.
- **Trails:** for accumulation trails, render into a `RenderTexture2D` (a nested cstruct of `Texture2D`s) with `BeginTextureMode` and blit it each frame with a translucent fade — more FFI surface; this demo clears each frame.
- After any edit to `anim.src`, re-run `./build.sh` (never hand-edit `anim.mfl` — it is generated).
