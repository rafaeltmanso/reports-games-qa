# Build Information

- **Game name:** FEED THE AI (Demo 0.40v)
- **Platform:** itch.io (HTML5 / Unity WebGL)
- **Browsers tested:** Google Chrome 149.0.0.0
- **Test date:** 2026-06-26

# Test Environment

- **CPU/GPU:** 20 logical cores / AMD Radeon RX 6650 XT (ANGLE/D3D11)
- **RAM:** 32 GB (Chrome allocates 4 GB JS heap limit)
- **Browser and version:** Google Chrome 149.0.0.0
- **Operating system:** Windows 10 (Win64)
- **Engine:** Unity 6000.3.3f1 (WebGL 2.0, OpenGL ES 3.0)
- **Viewport resolution:** 1280x720 @ DPR 1
- **Connection:** 4G effective, ~10 Mbps, RTT 50ms
- **WebGL assets:** ~74 MB (`.br` Brotli: framework, wasm, data)

# Tests performed

| Scenario | Result | Notes |
| --- | --- | --- |
| Initial Load | Pass | Unity WebGL loads in ~3s. Initial download via Brotli. "Booting language core" dialog screen displayed correctly |
| Fullscreen | Pass (partial) | Fullscreen button present in the lower right bar (`fullscreen-button.png`). RequestFullscreen API works. UX after fullscreen could not be validated without gamepad |
| Window Resize | Pass | Canvas resizes without losing state. In-game timer kept running (10:40 to 16:72) during the 1280x720 to 1024x600 to 1280x720 transition |
| Gamepad | N/A | Gamepad API available (`navigator.getGamepads` returns ok). No device connected in this environment. Unity Input System initialized ("Input System module state changed to: Initialized") |
| Long Session (30 min) | Not executed | Short session (~5min) executed. FPS stable around 182. JS heap 5.3 MB. No visible degradation. The full 30-minute cycle was not awaited |
| Refresh During Gameplay | Fail | F5/reload of the iframe wipes state. Page returns to itch.io landing page (Run game button) and requires a new Unity boot. Local Unity saves (IndexedDB) are also lost because the iframe context is recreated |

# Bugs found

**Bug ID:** WEB-001

**Title:** Refresh during gameplay discards the entire game state.

**Steps to reproduce:**

1. Open `https://weirdkidgames.itch.io/feed-the-ai`.
2. Click **Run game**.
3. Wait for Unity WebGL boot and click **Start!**.
4. Play for a few minutes (accumulate Data/timer).
5. Press F5 or reload the tab.

**Expected result:**

Local Unity saves (IndexedDB via `UnityCache`) should persist between reloads, returning to the "Continue" state.

**Actual result:**

The page returns to the itch.io landing page with the **Run game** button visible. Even though the asset cache (`FEED%20THE%20AI%20Web.data.br`) is reused, manual `boot.config` sync is deprecated (`JS_FileSystem_Sync()` warning) and `autoSyncPersistentDataPath` is not enabled (`config.autoSyncPersistentDataPath = true`). Console shows `refresh` plus `2 FS.syncfs operations in flight at once, probably just doing extra work`, indicating that save writes are not being confirmed before reload.

**Severity:** Major. Breaks the "close and come back" UX and can cause progress loss for casual players.

---

**Bug ID:** WEB-002

**Title:** URP shaders not supported on this GPU (warning).

**Steps to reproduce:**

1. Boot the game in Chrome.
2. Open DevTools to Console.

**Expected result:**

No shader errors.

**Actual result:**

Two `ERROR: Shader` appear in the console:
- `Hidden/CoreSRP/CoreCopy shader is not supported on this GPU (none of subshaders/fallbacks are suitable)`
- `Hidden/Universal/HDRDebugView shader is not supported on this GPU (none of subshaders/fallbacks are suitable)`

**Severity:** Minor. Does not block gameplay (the game runs normally), but indicates that the WebGL build was not fully validated for the DX11/ANGLE fallback path. May cause artifacts on AMD cards or older drivers.

---

**Bug ID:** WEB-003

**Title:** Repeated audio warnings during boot and gameplay.

**Steps to reproduce:**

1. Boot the game.
2. Watch DevTools Console during loading and the first seconds of gameplay.

**Expected result:**

No audio warnings.

**Actual result:**

The message `Trying to get length of sound which is not loaded yet.` appears **14 times** during boot plus 2 times in later interactions. Indicates that SFX referenced in code had not finished decoding when the game tried to play them.

**Severity:** Minor. UX affected (some sound effects may be missing in the first seconds).

---

**Bug ID:** WEB-004

**Title:** Unity deprecation warnings in production.

**Steps to reproduce:**

1. Boot the game.

**Expected result:**

Release build without deprecated API warnings.

**Actual result:**

`Manual synchronization of Unity Application.persistentDataPath via JS_FileSystem_Sync() is deprecated and will be later removed in a future Unity version. ... Pass config.autoSyncPersistentDataPath = true; to configuration in createUnityInstance() to opt in to the new behavior.`

**Severity:** Minor. Deprecation warning, but directly related to WEB-001 (manual sync is what is currently trying to persist saves).

# Performance Analysis

- **Average FPS:** ~182 FPS (estimated via `requestAnimationFrame` in the iframe. Measured on the main thread of the container, not inside WASM)
- **Peak Memory Usage:** ~5.3 MB of JS heap in the container. WASM/Unity memory could not be measured directly (Chrome DevTools exposes `performance.memory` only for the JS context, not the Unity WebAssembly heap). Estimate based on asset size: 74 MB uncompressed plus runtime around 250 to 400 MB
- **Console Errors:** 1 real error (`warning: 2 FS.syncfs operations in flight at once`) plus 4 `ERROR: Shader` (counted as errors by the Unity logger but functionally warnings); 7 warnings (`monetization`, `xr`, `allowfullscreen` precedence, syncfs, NavMesh agent x2, persistentDataPath deprecation)
- **CLS:** 0.00
- **Network:** 93 requests on initial load. Brotli enabled on Unity assets (`.br`). Browser cache reuses `.data.br` on subsequent loads
- **Render path:** WebGL 2.0 plus Unity URP. Extensions `EXT_color_buffer_float`, `WEBGL_compressed_texture_s3tc`, etc. available

# Recommendation

**Needs Fixes Before Release** (with caveats)

**Reason:**

The game is technically solid. Unity 6000 WebGL2 works well, FPS is high, input and render are OK. There are no critical blocking bugs. The issues found are mostly polish/regression:

1. **WEB-001 (Major):** Local save does not survive F5. This is severe for an incremental where players leave the tab open for hours/days. The fix is trivial. Enable `autoSyncPersistentDataPath` in `createUnityInstance()`.
2. **WEB-002 (Minor):** URP shaders without fallback. Recommend testing on hardware representative of the audience (Intel/AMD/NVIDIA integrated and dedicated).
3. **WEB-003 (Minor):** Audio playing before decode finishes. Add `audioClip.loadAudioData` promise check before `Play()`.
4. **WEB-004 (Minor):** Clean up deprecation warnings before release.

None of the bugs prevent the current demo from running, but **WEB-001 should be fixed before the Steam release** to avoid support tickets about "I lost my save".
