# WWE SmackDown vs. Raw 2011 - Xbox Series S/X (Dev Mode)

[![Platform](https://img.shields.io/badge/Platform-Xbox%20Series%20S%20%7C%20X%20(Dev%20Mode)-107C10?style=for-the-badge&logo=xbox&logoColor=white)](https://learn.microsoft.com/en-us/windows/uwp/xbox-apps/devkit-activation)
[![Architecture](https://img.shields.io/badge/Architecture-x86__64%20UWP-0078D7?style=for-the-badge&logo=windows&logoColor=white)](https://learn.microsoft.com/en-us/windows/uwp/)
[![Graphics](https://img.shields.io/badge/Graphics-Direct3D%2012-orange?style=for-the-badge&logo=directx&logoColor=white)](#why-these-settings-are-needed)
[![Status](https://img.shields.io/badge/Status-Playable%20%2F%2060%20FPS-brightgreen?style=for-the-badge)](#current-status)

Configuration and deployment guide to run **WWE SmackDown vs. Raw 2011** on **Xbox Series S and Xbox Series X in Developer Mode** at a smooth **60 FPS** using Direct3D 12.

> [!NOTE]
> **No copyrighted game files are included.** You must provide your own legally acquired copy of *WWE SmackDown vs. Raw 2011* (Xbox 360 `.iso` or extracted folder with `default.xex`).

---

## Current Status

- [x] **Boots natively on Xbox Dev Mode** (no startup crashes when set to 1080p).
- [x] **Full 60 FPS in-ring gameplay** in 1-on-1 and standard exhibition matches.
- [x] **Native wireless Xbox controller support** with trigger rumble and vibration.
- [x] **Fixed in-match white screen lockup** (via D3D12 RTV pipeline).
- [ ] ⚠️ **Multi-man Match Slowdowns:** The internal game engine and Havok physics are tied directly to 60 FPS. If framerates dip in heavy matches (Royal Rumble, 6-Man Tag), the game runs in slow-motion. Community optimization for Zen 2 CPU scheduling is welcomed!
- [ ] ⚠️ **Minor Graphical Glitches:** Occasional shadow flickering on certain arenas and custom wrestler attires.
- [ ] ⚠️ **Entrance Movies:** Titantron Bink video decoding can stutter during initial cache compilation. Setting **`ENTRANCES: OFF`** in match options is recommended.

---

## Simplified Setup Guide

> [!WARNING]
> ### 🛑 CRITICAL PREREQUISITE: Console Resolution MUST be set to 1080p!
> On Xbox Series S and Series X, **Xenia Canary UWP will crash immediately upon opening (instant black screen back to home) if your console output resolution is set to 4K.**
> 
> Before launching the app, you **MUST** change your Xbox display resolution to 1080p:
> 1. Go to Xbox **Settings** $\rightarrow$ **General** $\rightarrow$ **TV & display options**.
> 2. Set **Resolution** to **`1080p`** (do not leave it on 4K).

---

### Step 1: Install via Xbox Device Portal
1. Open the **Xbox Device Portal** in your PC browser (`https://<XBOX_IP>:11443`).
2. Go to **My games & apps** $\rightarrow$ click **Add**.
3. In **App package**, choose:
   `xenia-canary-uwp_1.1.7.1_x64.appxbundle`
4. In **Dependency**, choose:
   `Microsoft.VCLibs.x64.14.00.33519.appx`
5. Click **Deploy** and wait for installation to finish.

---

### Step 2: Set App to Game Mode & Restart Console
1. On your Xbox Dev Mode home screen, highlight the installed Xenia app.
2. Press the **View** button (two overlapping squares) on your controller.
3. Change **App Type** from `App` to **`Game`** (this unlocks full console RAM and CPU cores).
4. On the right-side panel of the Dev Mode home screen, select **Restart console**.

---

### Step 3: Insert the `xenia-canary.config.toml` (MANDATORY)
> [!IMPORTANT]
> **You MUST upload the provided `xenia-canary.config.toml` into Xenia.** Without this file, the emulator uses default settings that trigger an immediate black-screen crash on boot and a blinding white-screen freeze when entering matches.

1. Back in the **Xbox Device Portal** on your PC, click **File explorer** on the left menu.
2. Navigate to:
   `LocalAppData` $\rightarrow$ open the Xenia folder (starts with `27d6ba69...`) $\rightarrow$ open **`LocalState`**.
3. Click **Upload** at the top and upload the provided **`xenia-canary.config.toml`** into this folder.
*(Note: If loading games from an external USB drive, placing a copy of `xenia-canary.config.toml` in the same folder as your game will also work).*

---

### Step 4: Launch and Play
1. Confirm that your Xbox display is set to **1080p**, then launch Xenia.
2. Select your copy of *WWE SmackDown vs. Raw 2011* (`.iso` or extracted folder).
3. In match options, set **`ENTRANCES: OFF`** for the smoothest loading and performance.
4. Enjoy the match at 60 FPS!

---

## Why These Settings Are Needed

The pre-configured `xenia-canary.config.toml` includes critical fixes discovered through reverse engineering:

```toml
[Display]
fullscreen = true
internal_display_resolution = 8  # 1280x720 base (prevents D3D12 swapchain exhaustion)
vsync = true
framerate_limit = 60

[GPU]
gpu = "d3d12"
render_target_path_d3d12 = "rtv"  # FIXES WHITE SCREEN: uses direct RTV rasterization instead of broken ROV bloom
gpu_allow_invalid_fetch_constants = true  # FIXES BOOT CRASH: parses wrestler vertex buffers properly
d3d12_readback_resolve = false  # Avoids heavy CPU-GPU sync stalls
draw_resolution_scale_x = 1
draw_resolution_scale_y = 1
query_occlusion_fake_sample_count = 1000

[Memory]
ignore_offset_for_ranged_allocations = true

[Storage]
max_queued_frames = 4  # Eliminates pipeline latency and sluggish frame pacing
mount_cache = true
mount_scratch = true
```

---

## Community Help & Contributing

Pull requests, testing reports, and feedback are very welcome!
* **Performance optimization:** Help profile and optimize CPU worker thread dispatching for 6-man matches and Royal Rumbles.
* **Shaders & Lighting:** Fine-tune shadow map depth resolve under RTV.
* **Entrance Movies:** Assist in tuning Bink YUV video playback through Media Foundation.

To contribute:
1. Fork the repo and create your branch (`feature/perf-improvements`).
2. Commit your changes and open a Pull Request.

---

## Credits & Acknowledgements

* **ReXGlue SDK & wwe-svr2011-pc** — Original static decompilation research.
* **Xenia Canary Project** — Direct3D 12 Xbox 360 emulation layer.
* **Emulation Revival Community** — UWP packaging for Xbox Developer Mode.
* Reverse engineering diagnostics conducted with **REA (Reverse Engineer Anything)** and **Ghidra**.
