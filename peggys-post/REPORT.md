# Build Information

- **Game name:** Peggy's Post (v1.5.9)
- **Platform:** itch.io (HTML5 / Unity WebGL)
- **Browsers tested:** Google Chrome 149.0.0.0
- **Test date:** 2026-06-26

# Test Environment

- **CPU/GPU:** 20 logical cores / AMD Radeon RX 6650 XT (ANGLE/D3D11)
- **RAM:** 32 GB (Chrome allocates 4 GB JS heap limit)
- **Browser and version:** Google Chrome 149.0.0.0
- **Operating system:** Windows 10 (Win64)
- **Engine:** Unity 2021.3.15f1 (WebGL 2.0, OpenGL ES 3.0, Built-in Render Pipeline plus Postprocessing Stack v2)
- **Viewport resolution:** 1280x720 @ DPR 1
- **Connection:** 4G effective, ~10 Mbps, RTT 50ms
- **WebGL assets:** ~67 MB Windows build (~67 MB total WebGL. itch.io page measures ~25 MB in the DOM, Unity game weighs more after iframe)

# Tests performed

| Scenario | Result | Notes |
| --- | --- | --- |
| Initial Load | Pass | itch.io page in ~2s. Click on Run game opens the Unity iframe. **Full Unity boot takes ~30s** (WebGL 2021 with heavy Postprocessing). After boot, displays an open manual menu showing shipping rules and starter stamp pack |
| Fullscreen | Pass (partial) | Fullscreen API available. Fullscreen button present on the Unity template's bottom bar. UX after fullscreen could not be validated via direct click on the iframe (cross-origin blocks synthetic clicks, requires manual interaction) |
| Window Resize | Pass | Game keeps rendering correctly after resize from 1280x720 to 1024x768. Layout stays intact, parcel on the desk and customer visible |
| Gamepad | N/A | Gamepad API available, no device connected in this environment. Game is mouse + keyboard only per the doc |
| Long Session (30 min) | Not executed | Short session (~2min) executed. FPS stable around 172. JS heap 6.7 MB. No visible degradation. The full 30-minute cycle was not awaited |
| Refresh During Gameplay | Fail | F5/reload wipes the Unity iframe (reloads from scratch). "Run game" button reappears. The player must wait the ~30s boot again. Local save (IndexedDB) does not auto-restore. **Same issue already known from FEED THE AI**, both Unity WebGL on itch.io |

# Bugs found

**Bug ID:** WEB-009

**Title:** Refresh during gameplay discards the entire game state.

**Steps to reproduce:**

1. Open `https://digitarium.itch.io/peggys-post`.
2. Click **Run game**.
3. Wait the ~30s Unity WebGL boot.
4. Click **Next** (or press D) to start receiving customers.
5. Press F5 or reload the tab.

**Expected result:**

Local save (IndexedDB) should persist game state between reloads.

**Actual result:**

The page returns to the itch.io landing page with the **Run game** button visible. The Unity iframe is completely recreated, losing any state in memory. Console shows `[UnityCache] Failed to load '.../Peggy's%20Post.data.unityweb' from indexedDB cache due to the error: Error: Could not connect to database.` on first load, indicating that even the asset cache cannot initialize correctly. Full boot takes ~30s per refresh.

**Severity:** Major. Similar to WEB-001 (FEED THE AI). Both Unity WebGL on itch.io suffer from the same problem: `autoSyncPersistentDataPath` is likely not enabled, and IndexedDB is not being used for state saves. The dev has a "save slots" feature in the game (v1.5.2), but it only works within the current session.

---

**Bug ID:** WEB-010

**Title:** Postprocessing shaders without fallback for this GPU.

**Steps to reproduce:**

1. Open `https://digitarium.itch.io/peggys-post`.
2. Click Run game.
3. Open DevTools to Console.

**Expected result:**

No `ERROR: Shader` in console.

**Actual result:**

Four Postprocessing Stack v2 shaders (compatible with Unity 2021.3 builtin pipeline) do not run on the GPU:
- `Hidden/PostProcessing/Debug/Histogram`
- `Hidden/PostProcessing/Debug/LightMeter`
- `Hidden/PostProcessing/Debug/Vectorscope`
- `Hidden/PostProcessing/Debug/Waveform`
- `Hidden/PostProcessing/MultiScaleVO`
- `Hidden/PostProcessing/ScreenSpaceReflections`

Message: `none of subshaders/fallbacks are suitable` on each.

**Severity:** Minor. Debug shaders do not affect normal gameplay (used for dev tools). MultiScaleVO and ScreenSpaceReflections are production, but since the game is on builtin pipeline (not URP/HDRP), these effects are probably not active in normal gameplay. Does not block release.

---

**Bug ID:** WEB-011

**Title:** Warning repeated 10945 times about Postprocessing package.

**Steps to reproduce:**

1. Boot the game.
2. Observe console.

**Expected result:**

No repeated warnings.

**Actual result:**

`When used with builtin render pipeline, Postprocessing package expects to be used on a fullscreen Camera. Please note that using Camera viewport may result in visual artefacts or some things not working.` appears **1774 times** plus **9171 times** = **10945 times** total. Indicates that the game is using the Postprocessing package with Camera viewport instead of fullscreen, and the engine fires the warning every frame for each affected camera.

**Severity:** Minor. Functional (game runs), but pollutes the console and may indicate subtle performance degradation (each warning = log call). Recommend the dev adjust the Postprocessing Camera to use fullscreen or disable the deprecation warning.

---

**Bug ID:** WEB-012

**Title:** Game iframe is larger than the viewport (1300x800 in a 1280x720 window).

**Steps to reproduce:**

1. Open `https://digitarium.itch.io/peggys-post` in a 1280x720 window.
2. Observe the game iframe.

**Expected result:**

The iframe should fit in the viewport with some margin, or scrolling should be natural.

**Actual result:**

The Unity iframe is fixed at 1300x800 pixels, creating horizontal and vertical scroll bars in smaller windows. The itch.io page is also huge (19724px of total height) because the iframe is inline in the middle of the layout, forcing the user to scroll to interact with parts of the game. Playing in fullscreen helps, but hurts users who play in a window.

**Severity:** Minor. Recommend using responsive CSS (`max-width: 100%; height: auto;`) and/or detecting viewport and adjusting dynamically.

---

**Bug ID:** WEB-013

**Title:** HTTP 410 error on itch tracking endpoint (rh endpoint).

**Steps to reproduce:**

1. Open `https://digitarium.itch.io/peggys-post`.
2. Open DevTools to Network/Console.

**Expected result:**

No network errors on itch resources.

**Actual result:**

Request `GET https://digitarium.itch.io/peggys-post/rh/...` returns **HTTP 410 Gone**. Console shows `Failed to load resource: the server responded with a status of 410`.

**Severity:** Minor. Identical to WEB-006 (Serenitrove). Likely deprecated on itch's side, not the Peggy's Post dev's.

# Performance Analysis

- **Average FPS:** ~172 FPS (estimated via `requestAnimationFrame` in the itch container)
- **Peak Memory Usage:** ~6.7 MB of JS heap in the container (excludes Unity WASM memory which can reach 1 to 2 GB). Heavy Unity game, recommend 4 GB+ RAM for players
- **Console Errors:** 1 real error (410 Gone). ~6 shader errors classified as ERROR by Unity logger (but functionally warnings). **10945 warnings about Postprocessing Camera** (excessive logging issue)
- **Boot Time:** ~30 seconds for Unity WebGL to fully initialize with Postprocessing (may be too long for casual players)
- **Network:** Unity requests (`.data.br`, `.wasm.br`, `.framework.js.br`) with Brotli enabled
- **Render path:** WebGL 2.0 plus Unity Built-in Render Pipeline plus Postprocessing Stack v2
- **Gamepad API:** available but not supported by the game
- **Fullscreen API:** available and supported

# Recommendation

**Needs Fixes Before Release** (with caveats)

**Reason:**

Peggy's Post is a beautiful game with complex mechanics (post office management, stamp system, multiple morality paths), but has clear optimization and UX issues that should be addressed before an official web release:

1. **WEB-009 (Major):** Refresh wipes state. Same problem as FEED THE AI. This is a standard Unity WebGL plus itch.io pattern: the IndexedDB asset cache does not work well (`Could not connect to database`), and state saves do not persist between reloads. Recommendation: implement exportable save (similar to Serenitrove, which has functional Export) or at least enable `autoSyncPersistentDataPath` in Unity. The dev already has save slots in the game (v1.5.2+), so they could extend that mechanic to include export.

2. **WEB-011 (Minor):** 10945 repeated warnings in console. Does not affect gameplay but hurts QA tooling and likely degrades performance marginally. Adjust the Postprocessing Camera.

3. **WEB-010 (Minor):** Postprocessing shaders without fallback. Recommend testing on more GPUs.

4. **WEB-012 (Minor):** 1300x800 iframe does not fit common viewports. Makes the game hard to play without fullscreen.

5. **WEB-013 (Minor):** itch 410 error. Identical to WEB-006.

Peggy's Post has solid mechanics and charming visuals. The dev has an active Discord community and responds quickly to reports. These fixes are all polish (except WEB-009) and can be addressed in a v1.6.x.
