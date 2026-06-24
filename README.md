# machin-game-demo-anim

A **2D procedural animation** — a swirling **flow field** of 1200 particles pulled into vortices around three centers that drift on sine paths. Every pixel of motion is math over time: no assets, no physics engine. Written in **[machin](https://github.com/javimosch/machin)** (MFL), drawn with raylib through the C FFI.

Part of [**awesome-machin**](https://github.com/javimosch/awesome-machin) — the machin ecosystem.

> **Agents:** [`SKILL.md`](SKILL.md) covers the build and the math/FFI specifics.

```
  . ·  ╲ ╲ ╲  · .        machin — procedural flow field
 ·  ╲ ╲ (∘) ╱ ╱ ·        particles circle three drifting vortex
   ╲ ╲ ╱   ╲ ╱ ╱         centers; hue follows the flow direction
 ·  ╱ ╱ (∘) ╲ ╲ ·
  · ·  ╱ ╱ ╱  · ·
```

## Why it exists

The machin north star is "build real things, let usage drive features." This is the **showcase for machin's native math builtins** (added in **v0.46.0**) — procedural motion is exactly what they unlock. The flow field is pure trigonometry:

- each of three **vortex centers** drifts on a Lissajous path (`sin`/`cos` of time);
- at every particle, each center contributes a **tangential** velocity — `hypot` for the distance, `atan2` for the bearing, `+ pi/2` to turn it into a circling pull;
- the contributions sum, the speed is capped with `hypot`, and the particle's **hue** comes from its flow direction (`atan2` → HSV).

It **composes the existing FFI** (scalars + by-value `Color`, and a struct-returning `ColorFromHSV`) — it drove **no new machin feature**, and that's the point: once the math suite landed, a real procedural piece is just MFL. (It closes the arc [machin-game-demo-3d](https://github.com/javimosch/machin-game-demo-3d) opened: 3D drove nested cstructs → its libm orbit drove native math → this spends that math.)

## Build

Needs the `machin` compiler (**v0.46.0+**), a C compiler, **raylib**, and a display. A GUI binary links the system graphics stack, so it is **not** a no-dependency binary.

```bash
./build.sh            # → ./machin-game-demo-anim
./machin-game-demo-anim
```

`build.sh` uses a **system raylib** if installed (`sudo apt-get install libraylib-dev`, `brew install raylib`, …); otherwise it **vendors raylib's prebuilt static release** into `vendor/` automatically — no root required.

## How it works

- **State:** two parallel `[]float`s (`px`, `py`) for 1200 particles, seeded at random with `rand_bytes`.
- **Per frame:** recompute the three centers from `t`; for each particle sum the three `swirl` contributions, cap the speed, advance, wrap at the edges, and draw a small `DrawCircle` tinted by `ColorFromHSV(direction, …)`.
- **Frame-timed** (`SetTargetFPS(60)`, time advances `t` per frame) — no `sleep`.

See [`anim.src`](anim.src). `build.sh` runs `machin encode` to produce the canonical `anim.mfl`, then `machin build`.

## License

MIT
