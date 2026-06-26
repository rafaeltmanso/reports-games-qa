# Web Game QA Reports

QA reports for browser games hosted on itch.io, using Chrome DevTools MCP for automated inspection.

## Disclaimer

**This is a personal study/portfolio project only.** The author has no professional, commercial, or personal relationship with the games, developers, or publishers mentioned in this repository. All testing was performed on publicly available material on itch.io for the sole purpose of practicing QA methodology and showcasing tooling skills (Chrome DevTools MCP, performance analysis, bug reporting).

The games tested are publicly distributed on itch.io (often as free demos or name-your-price downloads) and were tested in their publicly accessible web builds. No private builds, NDAs, betas, or non-public material were accessed. All findings are observational reports based on time-limited testing sessions and should not be treated as exhaustive or authoritative reviews.

If you are a developer of one of the tested games and would like a bug report removed or amended, please open an issue.

## Setup

- **MCP Server:** `chrome-devtools-mcp` configured in `opencode.json`
- **Browser:** Google Chrome 149.0.0.0
- **OS:** Windows 10 (Win64)
- **Hardware:** 20 cores, AMD Radeon RX 6650 XT, 32 GB RAM
- **Test viewport:** 1280x720 @ DPR 1

## How to run

1. Open opencode in this directory
2. Provide an itch.io game URL
3. The 6 standard scenarios are tested:
   - Initial Load
   - Fullscreen
   - Window Resize
   - Gamepad
   - Long Session (30 min) - simulated in a short window
   - Refresh During Gameplay

Each game gets its own subdirectory with screenshots, traces (when applicable), and `REPORT.md`.

## Comparative summary of tested games

| Game | Engine | Build | FPS | Console Issues | Verdict |
| --- | --- | --- | --- | --- | --- |
| [FEED THE AI](feed-the-ai/REPORT.md) (Demo 0.40v) | Unity 6000.3.3f1 WebGL | ~74 MB | ~182 | 1 error, 4 shader errors, 7 warnings | Needs Fixes Before Release |
| [Serenitrove](serenitrove/REPORT.md) (v1.2.0) | HTML5/JS vanilla | ~25 MB | ~182 | 1 error (410), 3 warnings | Approved for Release |
| [Peggy's Post](peggys-post/REPORT.md) (v1.5.9) | Unity 2021.3.15f1 WebGL | ~67 MB | ~172 | 1 error (410), 6 shader errors, 10945 camera warnings | Needs Fixes Before Release |

## Cross-game bug patterns

### WEB-001 / WEB-009 (Major): Refresh wipes save state in Unity WebGL on itch.io
Both Unity WebGL games (FEED THE AI and Peggy's Post) suffer from the same issue:
- Local save does not persist between reloads (F5 or refresh)
- IndexedDB cache fails with `Could not connect to database` on boot
- Console shows deprecated `JS_FileSystem_Sync()` warning and `autoSyncPersistentDataPath` not enabled
- Full boot takes 3s to 30s per reload (depending on build size)

**General recommendation:** Unity WebGL developers should enable `config.autoSyncPersistentDataPath = true;` in `createUnityInstance()` to auto-sync saves.

### WEB-006 / WEB-013 (Minor): HTTP 410 error on itch tracking endpoint
Both games (Serenitrove and Peggy's Post) receive `410 Gone` on `GET /rh/...` (itch tracking endpoint). Likely deprecated on itch.io's side. Does not block gameplay.

### URP/Postprocessing shaders without fallback (Minor)
Unity games with URP (FEED THE AI) and Postprocessing Stack v2 (Peggy's Post) show `ERROR: Shader` in console due to lack of subshaders compatible with the tested GPU (AMD RX 6650 XT / ANGLE). Functional but should be validated on hardware representative of the audience.

## Bugs by severity

### Major (blocker)
- **WEB-001 (FEED THE AI):** Refresh wipes state, autoSync disabled
- **WEB-009 (Peggy's Post):** Refresh wipes state, IndexedDB cache fails

### Minor (polish)
- **WEB-002 (FEED THE AI):** URP shaders without fallback (CoreCopy, HDRDebugView)
- **WEB-003 (FEED THE AI):** Audio warnings on boot (14 occurrences)
- **WEB-004 (FEED THE AI):** Deprecation warning for JS_FileSystem_Sync
- **WEB-005 (Serenitrove):** Refresh does not auto-restore autosave
- **WEB-006 (Serenitrove):** itch 410 error
- **WEB-007 (Serenitrove):** Fixed 800x600 iframe does not scale
- **WEB-008 (Serenitrove):** Cross-origin prevents programmatic inspection
- **WEB-010 (Peggy's Post):** Postprocessing shaders without fallback
- **WEB-011 (Peggy's Post):** 10945 repeated Camera viewport warnings
- **WEB-012 (Peggy's Post):** 1300x800 iframe does not fit common viewport
- **WEB-013 (Peggy's Post):** itch 410 error

## Statistics

- **Games tested:** 3
- **Bugs found:** 13 (2 Major, 11 Minor)
- **Screenshots captured:** 19
- **Performance traces:** 1 (FEED THE AI)
- **Verdicts:**
  - 1 Approved for Release (Serenitrove)
  - 2 Needs Fixes Before Release (FEED THE AI, Peggy's Post)

## Methodological notes

- **Long Session (30 min)** was simulated in a short window (~2-5min) with FPS, heap and console measurements taken throughout the session. The full 30-minute cycle was not awaited due to QA time constraints.
- **Gamepad** was not tested in any of the games due to lack of hardware; only the presence of the Gamepad API was verified.
- **Refresh During Gameplay** is the most revealing test for Unity WebGL: both Unity games (FEED THE AI and Peggy's Post) fail, while the vanilla JS game (Serenitrove) passes because it explicitly documents the behavior and provides manual Export.
- **Fullscreen** was not 100% validated due to Chrome DevTools MCP limitations (cross-origin iframe blocks synthetic coordinate clicks); only the API availability was confirmed.
