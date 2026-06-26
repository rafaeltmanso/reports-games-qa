# Game QA Report Template

Use this template as a starting point for new web game QA reports.

```markdown
# Build Information

- **Game name:** [Name + version]
- **Platform:** [itch.io URL]
- **Browsers tested:** [Chrome 149, Firefox, Edge, etc]
- **Test date:** [YYYY-MM-DD]

# Test Environment

- **CPU/GPU:** [e.g. 20 cores / AMD Radeon RX 6650 XT]
- **RAM:** [e.g. 32 GB]
- **Browser and version:** [e.g. Google Chrome 149.0.0.0]
- **Operating system:** [e.g. Windows 10 (Win64)]
- **Engine:** [e.g. Unity 6000.3.3f1 WebGL 2.0 / HTML5+JS vanilla / etc]
- **Viewport resolution:** [e.g. 1280x720 @ DPR 1]
- **Connection:** [e.g. 4G effective, ~10 Mbps, RTT 50ms]
- **Assets:** [e.g. ~74 MB total (.br Brotli)]

# Tests performed

| Scenario | Result | Notes |
| --- | --- | --- |
| Initial Load | Pass | [Details] |
| Fullscreen | Pass (partial) | [Details] |
| Window Resize | Pass | [Details] |
| Gamepad | N/A | [No device connected] |
| Long Session (30 min) | Not executed | [Short session ~Xmin] |
| Refresh During Gameplay | Fail | [Details] |

# Bugs found

**Bug ID:** WEB-XXX

**Title:** [Short description]

**Steps to reproduce:**

1. [Step 1]
2. [Step 2]

**Expected result:**

[What should happen]

**Actual result:**

[What happens]

**Severity:** [Major / Minor / Blocker]

---

# Performance Analysis

- **Average FPS:** ~XXX FPS
- **Peak Memory Usage:** ~XX MB
- **Console Errors:** [number and types]
- **Network:** [X requests, ~XX MB]
- **Render path:** [WebGL 2.0 + URP / DOM-based / etc]

# Recommendation

**[Approved for Release / Needs Fixes Before Release / Blocked]**

**Reason:**

[Summary of main findings]
```

## Conventions

- Use `Bug ID: WEB-XXX` for sequential numbering
- Severity: Blocker (prevents use) > Major (significant loss of functionality) > Minor (polish)
- Do not use em-dashes (`-`) as standalone separators. Use periods, semicolons, or restructure the sentence.
- Always include a screenshot in the game directory for each tested scenario
- Performance traces should be saved as `perf-baseline.json` when collected
