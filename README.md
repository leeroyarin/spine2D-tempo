# spine2D-tempo

What each part of the [Spine Runtimes](https://github.com/EsotericSoftware/spine-runtimes) repo does — a plain-English map of the whole codebase, runtime by runtime.

> Source repo: https://github.com/EsotericSoftware/spine-runtimes (branch 4.3, checked 2026-11-09).
> This file documents that repo. It is not affiliated with Esoteric Software.

---

## What Spine is

Spine is a 2D skeletal animation tool. Instead of drawing every frame, you rig a character with **bones**, attach images to those bones, and animate the bones. The Spine **editor** (a paid desktop app) is where animations are authored. It supports inverse kinematics (IK), skins, meshes with deformation (FFD), weights, and constraints (IK, transform, path, physics).

## What the spine-runtimes repo is

The repo holds the official **runtime libraries** — the code that loads Spine animation data exported from the editor (`.json` or binary `.skel`, plus `.atlas` and `.png` texture pages) and plays it back inside a game or app.

All runtimes share one architecture:

- **`SkeletonData`** — the shared setup data for a skeleton (bones, slots, skins, animations).
- **`Skeleton`** — a live instance of a skeleton (current pose, skin, attachments).
- **`AnimationStateData`** — mix times between animations.
- **`AnimationState`** — plays animations on tracks: set, add, queue, crossfade/mix, empty animations, and event listeners (`Start`, `Interrupt`, `Event`, `Complete`, `End`).
- **Skins** — swap looks at runtime, including mix-and-match custom skins built from pieces of other skins.
- **Attachments** — region, mesh, clipping, bounding-box, point, and path attachments on slots; slots also control draw order.
- **Constraints** — IK, transform, path, and physics constraints, plus sliders (4.3).
- **Procedural control** — bones can be moved by code at runtime (aiming, looking at the mouse, etc.).
- **Bounding boxes** — for hit detection and physics.

**Version rule (important):** the runtime and the Spine editor must match by major.minor version, and be updated in lock step. The repo's default branch is stable and works with the latest non-beta editor; new development happens on `X.X-beta` branches.

**License (important):** the runtimes are **not open source**. They use the Spine Runtimes License Agreement: you can integrate them free of charge, but users of your software must hold their own Spine Editor license (or you hold one at the time of integration to distribute to non-licensees, provided they don't modify it or build new software with it). See the repo's `LICENSE` and https://esotericsoftware.com/spine-runtimes-license.

---

## The runtimes, part by part

### Core libraries (no rendering — load and animate, you draw the result)

#### `spine-cpp` — C++ core
The shared core everything in the C family is built on. Loads and manipulates skeletons and animations, with **no rendering** — you use the pose/slot/attachment data it produces to draw with your own renderer, or to drive physics. All Spine features. C++11. Key classes: `SkeletonData`, `Skeleton`, `AnimationStateData` (`setDefaultMix` / `setMix`), `AnimationState` (tracks), `TrackEntry`. Base for spine-glfw, spine-sdl, spine-sfml, spine-godot, spine-ue, spine-flutter (via spine-c), and spine-ios (via spine-c).

#### `spine-c` — C wrapper
A C99 wrapper around spine-cpp, auto-generated (see `generate-bindings.sh`). Adds a generic `SkeletonRenderer` that returns render commands for any textured-triangle API, so it can be dropped into almost any C renderer. All features. Base for spine-ios, spine-flutter, spine-sdl, and spine-glfw.

#### `spine-csharp` — C# core
The C# equivalent of spine-cpp: loads and manipulates skeletons and animations, no rendering. All features. Base for spine-unity, spine-monogame, and the older XNA runtime.

### Reference implementation

#### `spine-libgdx` — Java / libGDX
The **reference implementation** — the one the others are measured against. Loads, animates, **and renders**. All features. Available on Maven Central (`com.esotericsoftware.spine:spine-libgdx`). Includes `SkeletonRenderer` / `SkeletonMeshRenderer` and a `SkeletonViewer` tool plus tests. If a feature exists in Spine, it works here first.

### Game engines

#### `spine-unity` — Unity
Built on spine-csharp. All features. Unity 2017.1–6000.4, data 4.3.74+. Install via `.unitypackage` or UPM. Provides components (`SkeletonAnimation`, `SkeletonGraphic`, …), custom asset types (`SkeletonDataAsset`), URP/HDRP shader packages, and Timeline support. Premultiplied alpha with the Spine/Skeleton shaders. One caveat: physics components can't mirror bone-weighted/deformed `BoundingBoxAttachment`s (manual hit detection still works).

#### `spine-godot` — Godot
Built on spine-cpp. Godot 3.5.x / 4.0.x. Two flavors: **GDExtension** (drop the `bin/` files into a project) or a custom C++ engine module (adds C# bindings and AnimationPlayer support). Provides `SpineSprite`, `SpineSprite3D` (native 3D, added Sept 2026), `SpineAnimationTrack`, `SpineSlotNode3D`. 2D renderer covers all features except screen blend mode; two-color tinting needs Godot 4.3+. No premultiplied-alpha atlases.

#### `spine-ue` — Unreal Engine
Built on spine-cpp. UE 4.27–5.4. A plugin with C++ and Blueprints, custom asset types, `SpineSkeletonComponent`, `SpineSkeletonAnimationComponent`, and `SpineWidget`. Needs a C++ code project (not Blueprint-only). Supports all Spine features except multiply/screen blending, premultiplied-alpha atlases, and two-color tinting.

### Mobile and cross-platform apps

#### `spine-android` — Android
Android integration built on spine-libgdx. Min SDK 24 (Android 7.0). `SpineView`, Jetpack Compose support, Kotlin/Java. No premultiplied-alpha atlases. The repo README is a stub (release notes only); full docs live on the Spine site.

#### `spine-ios` — iOS / tvOS / macOS / visionOS
Swift/Obj-C, built on spine-c + spine-cpp, Metal rendering. All features except two-color tinting. Distributed via Swift Package Manager (`SpineC`, `SpineSwift`, `SpineiOS` products). `SpineView` / `SpineUIView`. Xcode 14+, iOS 13+.

#### `spine-flutter` — Flutter
Dart, built on spine-c. Desktop, mobile, and web. All features except two-color tinting and screen blend mode. On pub.dev as `spine_flutter` (^4.3.0).

### Web — `spine-ts`

TypeScript/JavaScript, distributed on npm as `@esotericsoftware/*`. Twelve modules:

| Module | What it does |
|---|---|
| `spine-core` | The shared TypeScript core: loading, animation state, skins, constraints. |
| `spine-webgl` | WebGL renderer. All features. |
| `spine-canvas` | Canvas 2D renderer. No meshes/clipping/two-color tinting (experimental mesh flag exists, slow and with artifacts). |
| `spine-canvaskit` | CanvasKit renderer. All features except two-color tinting. |
| `spine-player` | Drop-in web player widget for embedding Spine animations in a page. All features. |
| `spine-webcomponents` | Web-component wrappers for embedding. |
| `spine-threejs` | three.js renderer. No two-color tinting; `SkeletonMesh.zOffset` for depth sorting. |
| `spine-phaser-v3` / `spine-phaser-v4` | Phaser 3 and Phaser 4 plugins. |
| `spine-pixi-v7` / `spine-pixi-v8` | PixiJS v7 and v8 plugins. |
| `spine-construct3` | Construct 3 addon. No tint black / two-color tinting; supports physics constraints. |

### Desktop frameworks and C/C++ examples

#### `spine-glfw` — GLFW + OpenGL
Built on spine-c + spine-cpp. All features. C and C++ examples included; CMake FetchContent install.

#### `spine-sdl` — SDL
Built on spine-c + spine-cpp. All features except screen blend mode and two-color tinting. Both C (`spine-sdl-c`) and C++ (`spine-sdl-cpp`) APIs; SDL2.

#### `spine-sfml` — SFML
C++ API on spine-cpp, SFML 2.6+. All features. `#include <spine-sfml.h>`. No C API.

#### `spine-monogame` — MonoGame
Built on spine-csharp (via the old spine-xna code) with MonoGame project files. All features. `SpineEffect.fx` shader provides two-color tinting. .NET 6 / MonoGame 3.8.2.

#### `spine-haxe` — Haxe
Haxe + OpenFL/Lime. The core module has zero dependencies; renderers for Starling and HaxeFlixel (min 5.9.0). Targets HTML5/WebGL. All features except premultiplied-alpha atlases and two-color tinting. Not on lib.haxe.org (zip install). Canvas rendering not supported.

### Discontinued

#### `spine-xna` — discontinued stub
Only a README remains: *"spine-xna has been discontinued in favor of spine-monogame. Microsoft no longer offers the old XNA tools for download."* Removed Jan 2022. Use spine-monogame instead.

---

## Other parts of the repo

- **`examples/`** — example projects, including the Super Spineboy example game.
- **`tests/`** — runtime test suites.
- **`CHANGELOG.md`** — what changed per version; the place to check before upgrading.
- **`LICENSE`** — the Spine Runtimes License Agreement (see the license note above).
- **`Package.swift`** — Swift package manifest for the iOS runtime.
- **`SpineConfig.cmake.in` / `flags.cmake`** — CMake configuration for the C/C++ runtimes.
- **`generate-bindings.sh`** — regenerates the spine-c wrapper from spine-cpp.
- **`formatters/`** — code formatting configs for contributors.
- **`corrections-of-fable5.1.md`** — notes on an old Spine data format correction.
- **`todos/`** — the maintainers' backlog notes.

Bugs and feature requests live in the [spine-runtimes](https://github.com/EsotericSoftware/spine-runtimes/issues) and [spine-editor](https://github.com/EsotericSoftware/spine-editor/issues) issue trackers; the [roadmap](http://esotericsoftware.com/spine-roadmap) is a friendlier view of the same tracker.

---

## What you can build with this

- **2D games** of any genre — platformers, RPGs, tower defense, fighting games — on any of the engines above.
- **Mobile and desktop games** (Unity, Godot, Unreal, libGDX, MonoGame, Flutter).
- **Web games and embeds** (spine-ts: web player, Phaser, PixiJS, three.js).
- **Interactive storytelling, apps, and cartoons/film/VFX** where smooth bone-driven character animation beats frame-by-frame sprites.

Best supported: **Unity, Godot, Unreal, libGDX, and web via spine-ts**.

Getting started, in one line per runtime: export from the Spine editor → add the runtime matching your editor version (unitypackage/UPM, GDExtension `bin/`, Gradle/Maven, npm, CMake FetchContent, SPM, pub.dev) → load `SkeletonData` + atlas → create a `Skeleton` + `AnimationState` → set/add animations with mix times.

Games made with Spine include Cult of the Lamb, Return to Monkey Island, Unruly Heroes, and Code: HARDCORE.

---

## Sources

- Repo root README and per-runtime READMEs in https://github.com/EsotericSoftware/spine-runtimes (branch 4.3)
- Spine Runtimes Guide: http://esotericsoftware.com/spine-runtimes-guide
- Spine Runtimes License: http://esotericsoftware.com/spine-runtimes-license
