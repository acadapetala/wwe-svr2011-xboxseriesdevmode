# WWE SmackDown vs. Raw 2011 - Xbox Series S/X (Dev Mode Port)

[![Platform](https://img.shields.io/badge/Platform-Xbox%20Series%20S%20%7C%20X%20(Dev%20Mode)-107C10?style=for-the-badge&logo=xbox&logoColor=white)](https://learn.microsoft.com/en-us/windows/uwp/xbox-apps/devkit-activation)
[![Architecture](https://img.shields.io/badge/Architecture-x86__64%20UWP-0078D7?style=for-the-badge&logo=windows&logoColor=white)](https://learn.microsoft.com/en-us/windows/uwp/)
[![Graphics](https://img.shields.io/badge/Graphics-Direct3D%2012-orange?style=for-the-badge&logo=directx&logoColor=white)](#technical-details)
[![Status](https://img.shields.io/badge/Status-Playable%20%2F%20In%20Development-yellow?style=for-the-badge)](#current-status)

Native **UWP (Universal Windows Platform)** deployment and compatibility runtime for **WWE SmackDown vs. Raw 2011** on **Xbox Series S and Xbox Series X in Developer Mode**.

This project provides the configuration, runtime layer, shaders, and packaging recipes to execute *WWE SmackDown vs. Raw 2011* natively on modern Xbox Series hardware using Direct3D 12.

> [!NOTE]
> **No copyrighted game assets or disc images are distributed in this repository.** You must provide your own legally dumped Xbox 360 game files (`default.xex` / ISO / extracted game disc).

---

## Current Status

- [x] **Boots natively on Xbox Dev Mode** (without immediate exit or sandbox crash).
- [x] **Full 60 FPS in-ring gameplay** in standard 1-on-1 and exhibition matches.
- [x] **Native Xbox wireless gamepad support** with vibration and original controller mapping.
- [x] **USB & Internal SSD storage routing** (reads game data from NTFS external drives or `LocalState`).
- [ ] ⚠️ **Occasional Framerate Slowdowns:** The game logic and Havok physics are tied directly to a locked 60 FPS. In certain multi-man matches (e.g., Royal Rumble, 6-man tags) or crowded arena views, framerate dips can trigger internal slow-motion.
- [ ] ⚠️ **Minor Graphical Glitches:** Render Target View (RTV) vs. Rasterizer-Ordered View (ROV) discrepancies may cause occasional shadow flickering, texture misalignments, or blown-out lighting on certain custom wrestler attires.
- [ ] ⚠️ **Entrance Video Stutter:** Entrance titantron video decoding (`.bik` Bink streams) can stutter or hitch during initial cache population. Disabling entrances (`ENTRANCES: OFF`) is currently recommended for optimal match stability.

---

## Quick Start (Xbox Dev Mode)

### 1. Requirements
* An Xbox Series S or Xbox Series X console activated in **Developer Mode**.
* PC with access to the **Xbox Device Portal** (`https://<XBOX_IP>:11443`).
* A USB flash drive or external hard drive formatted in **NTFS** (recommended).
* Your own extracted copy of *WWE SmackDown vs. Raw 2011* (Xbox 360).

### 2. Package Deployment
1. Open the **Xbox Device Portal** in your PC browser.
2. Navigate to **My games & apps** $\rightarrow$ **Add**.
3. Set **App package** to `xenia-canary-uwp_1.1.7.1_x64.appxbundle`.
4. Set **Dependency** to `Microsoft.VCLibs.x64.14.00.33519.appx`.
5. Click **Deploy**.
6. On the Xbox Dev Mode home screen:
   * Highlight the installed app, press the **View** button (two overlapping squares), and change **App Type** from `App` to **`Game`**.
   * Select **Restart console** from the right-hand panel to apply full RAM allocation.

### 3. Storage Setup (USB Drive)
1. Format your USB drive in **NTFS**.
2. Run the included NTFS permission script in PowerShell (Run as Administrator) to grant UWP container access:
   ```powershell
   powershell -ExecutionPolicy Bypass -File .\configurar_pendrive.ps1 -Drive E:
   ```
   *(Replace `E:` with your USB drive letter).*
3. Copy the contents of `out/USB_PENDRIVE_ESTRUTURA/` to the root of your USB drive.
4. Place your game files (`default.xex` or `.iso`) inside the `WWE2011/` folder.
5. Plug the USB drive into your Xbox and launch the app.

---

## Technical Details & Configuration

### Critical `xenia-canary.config.toml` Flags

To avoid startup black screens and in-match white-screen lockups on Xbox D3D12 hardware, the following configuration flags are mandatory:

```toml
[Display]
fullscreen = true
internal_display_resolution = 8  # 1280x720 native base (prevents D3D12 swapchain exhaustion)
vsync = true
framerate_limit = 60

[GPU]
gpu = "d3d12"
render_target_path_d3d12 = "rtv"  # Fixes post-processing bloom blowout (white screen freeze)
gpu_allow_invalid_fetch_constants = true  # Required for wrestler vertex buffer parsing
d3d12_readback_resolve = false  # Prevents severe CPU-GPU synchronization stalls
draw_resolution_scale_x = 1
draw_resolution_scale_y = 1
query_occlusion_fake_sample_count = 1000

[Memory]
ignore_offset_for_ranged_allocations = true

[Storage]
max_queued_frames = 4  # Drastically cuts input latency and frame pacing jitter
mount_cache = true
mount_scratch = true
```

---

## Roadmap & Areas Needing Community Help

We welcome contributions, pull requests, and testing feedback! The primary goals for upcoming revisions include:

1. **Eliminate Performance Slowdowns (Framerate Drops):**
   - Optimize multi-threaded job dispatching for the 8-core AMD Zen 2 CPU.
   - Pre-compile Pipeline State Objects (PSO) to eliminate in-ring shader compilation hitching.
   - Profile draw calls during heavy 6-man matches to maintain a locked 60 FPS.
2. **Fix Minor Graphical Glitches:**
   - Refine depth resolve and shadow map generation under RTV to eliminate minor character shadow flickering.
   - Correct custom attire texture wrapping in Superstar Threads.
3. **Smooth Entrance Playback:**
   - Optimize Bink hardware YUV video decoding through the UWP Media Foundation pipeline to allow flawless entrance movies without hitching.
4. **Automated Sideloading & UI:**
   - Build a lightweight launcher UI tailored for Dev Mode gamepad navigation.

---

## Contributing

1. Fork the repository.
2. Create your feature branch (`git checkout -b feature/performance-optimization`).
3. Commit your changes (`git commit -m 'Tune D3D12 PSO caching for multi-man matches'`).
4. Push to the branch (`git push origin feature/performance-optimization`).
5. Open a Pull Request detailing your console model (Series S vs. Series X), Dev Mode OS build, and performance logs.

---

## Credits & Acknowledgements

* **ReXGlue SDK & wwe-svr2011-pc** — Original static decompilation research and PC port infrastructure.
* **Xenia Canary Project** — Direct3D 12 emulation and guest translation layer.
* **Emulation Revival Community** — UWP adaptation and packaging for Xbox Developer Mode.
* Reverse engineering diagnostics conducted with **REA (Reverse Engineer Anything)** and **Ghidra**.
