# Wuwa Mobile Config

Performance and graphics configuration presets for **Wuthering Waves Mobile (Version 3.5)** designed to help optimize gameplay based on device capability.

These configs aim to improve FPS stability, visual quality, or provide a balanced gaming experience depending on your hardware.

Built using Kuro Game's official **3.5 engine configuration structure**, AlteriaX's command references, and extensive testing across multiple Android GPU architectures.

---

# 🛠️ Patch 3.5 Optimization Summary

Version **3.5** continues Kuro's streaming and memory management improvements introduced in previous versions while expanding support for newer mobile hardware.

The following optimizations are applied throughout the config collection:

* **Updated Kuro Streaming System**

  * Uses Kuro's latest streaming architecture where applicable.
  * Optimized texture streaming pools based on device tier.

* **Improved Memory Management**

  * Streaming and garbage collection values adjusted for better stability.
  * Reduced texture pop-in on higher-end presets.
  * Lower RAM pressure on low-end presets.

* **Device Tier Separation**

  * Presets are categorized by actual hardware capability rather than a one-size-fits-all approach.
  * Separate Snapdragon and All Devices profiles.

* **GPU-Specific Optimization**

  * Snapdragon presets include Adreno-focused tuning.
  * All Devices presets are designed for Mali, PowerVR, Xclipse, and other Android GPUs.

* **Streaming & World Loading Improvements**

  * Optimized World Partition loading ranges.
  * Reduced stutters during traversal and fast movement.

---

# 🔧 Config Rebuild Notes (v3.5)

This update carries over the same 8-tier structure (4 tiers × All Devices / Snapdragon) but every preset was rebuilt from the ground up after an audit turned up a structural bug affecting every prior draft, plus a batch of forbidden/ineffective CVars. Details below.

### The structural bug (fixed in every tier)

Every prior draft defined its profile like this:

```ini
[Android_SomeTierName DeviceProfile]
DeviceType=Android
BaseProfileName=Android
```

The engine only assigns a device to a profile when it detects a **real GPU family name** (e.g. `Android_Mali_G57`, `Android_Adreno730`). `Android_SomeTierName` isn't a name the engine ever detects, so none of these profiles were actually applied to any real device — every tweak inside them was dead code.

**Fix:** every tier now uses a two-layer structure — one base profile holding the actual CVars, and a list of real detected GPU family names that point to it via `BaseProfileName=`, matching the pattern this repo's own configs use.

### Forbidden CVars removed

Per this repo's own FAQ, Kuro's `ConfigMonitor` silently strips certain CVars at runtime — setting them does nothing.

| CVar | Status | What we did |
|---|---|---|
| `r.MobileContentScaleFactor` | Forbidden, no bypass | Removed. Resolution scale must be set in-game via Developer Options. |
| `r.Streaming.PoolSize` | Forbidden, no bypass | Removed. Texture memory footprint now controlled through `sg.TextureQuality`, which the engine does honor. |
| `r.ViewDistanceScale` | Forbidden, **but has a bypass** | Converted to `sg.ViewDistanceQuality` in every tier — same intent, actually takes effect. |
| `r.FEstimation.Option`, `r.SecondaryScreenPercentage.GameViewport` | Forbidden, no bypass | Never added — even the maintainer's own High-Visual and A High-End files still carry these as dead weight. |

### Recurring fixes applied across multiple tiers

- **`sg.EffectsQuality` hard-capped at 2 everywhere**, regardless of visual tier — a documented crash threshold; going higher crashes the game outright.
- **KuroFI (Frame Interpolation)** needs two companion CVars to avoid a known crash (the "bike" frame-gen bug per this repo's own changelog): `r.KuroFI.EnableSingleOcclusionBloomReplace=1` and `r.KuroFI.InternalResolutionScale=0.5`. Both Low-End tiers disable FI entirely instead — pure overhead with no upside on weak hardware.
- **Engine.ini vs. DeviceProfiles.ini conflicts**: DeviceProfiles.ini loads after Engine.ini and can silently override it. A few drafts had DeviceProfile CVars quietly *downgrading* a value Engine.ini had already set higher (e.g. `r.SSR.Quality`, `r.ShadowQuality`). Removed rather than left to fight the baseline.
- **Skin cache reduced (128MB → 64MB)** on both Low-End tiers — a fixed memory allocation for character mesh deformation; halving it helps 3-4GB RAM devices without a visible model quality hit.

### Unverified / flagged CVars

Kept because they're plausible and not proven harmful, but not confirmed against this repo's testing — watch these closely if you install:

- **`r.GSR.Enabled`** (Snapdragon tiers) — Qualcomm's GSR upscaler plugin exists in the engine, but no `r.GSR.*` cvar appears anywhere in this repo.
- **`r.PSO.LRUCapacity`** — plausible shader-cache cvar, not seen in any reference file.
- **`fx.Niagara.QualityLevel`** (Balanced Visual tier and up) — not used in any current-gen tier here.

**VRS is disabled by default** on Balanced Visual and High End (both folders). The requested `r.VRS.EnableMaterial` / `r.VRS.EnableMesh` don't appear anywhere in this repo — the only place VRS shows up at all is a `Deprecated Configs` folder from an old experimental RT test, and even there the cvars used were different (`r.VRS.EnableImage`, `r.VRS.ContrastAdaptiveShading`, etc). Left in as commented-out lines with the real deprecated names in case you want to test separately.

### Device scoring

| Tier | All Devices `DeviceScore` | Snapdragon `DeviceScore` |
|---|---|---|
| Low End | 3000 | 3500 |
| Balanced Performance | 4000 | 6000 |
| Balanced Visual | 7000 | 9000 |
| High End | 10000 | 12000 |

Snapdragon scores sit above their All Devices counterparts at every tier — Adreno chips generally have more mature drivers and better sustained performance than budget Mali/Unisoc/PowerVR silicon at an equivalent nominal tier.

### What wasn't touched

Every Engine.ini in this update was checked byte-for-byte against this repo's own corresponding file. Where they matched exactly, they were left untouched (only the skin-cache tweak above was applied, and only to the two Low-End tiers). No Engine.ini in this update contains invented or unverified CVars.

---

# 📁 Overview

This repository contains two main configuration folders.

## All Devices

General presets designed to work across most Android devices including:

* Mali
* PowerVR
* Xclipse
* Tegra
* Adreno

## Snapdragon

Presets specifically optimized for Qualcomm Snapdragon chipsets.

These presets include additional Adreno-focused optimizations and tuning designed for Snapdragon devices.

---

# ⚙️ Performance Presets & Recommended Chipsets

| Tier                    | Snapdragon Target         | All Devices Target         | Scale Factor         |
| ----------------------- | ------------------------- | -------------------------- | -------------------- |
| **Low End**             | Adreno 5xx / 6xx Low      | Mali T/G5x, PowerVR GE8xxx | 1.0                  |
| **Balance Performance** | Adreno 6xx / 7xx Midrange | Mali G57/G68/G78           | 1.0                  |
| **Balance Visual**      | Adreno 730 / 740          | Mali G715/G78, Xclipse 9xx | 1.5 (SD) / 1.0 (All) |
| **High End**            | Adreno 750 / 830          | Mali G925, Xclipse 9xx     | 2.0 (SD) / 1.5 (All) |

---

# Low End

Best for devices that struggle with default graphics or experience FPS drops.

### Typical Compatible Chipsets

* Snapdragon 660
* Snapdragon 665
* Snapdragon 680
* Helio G85
* Helio G88
* Helio G99
* Dimensity 700
* Dimensity 810

### What's Applied

* Reduced texture quality
* Reduced shadow quality
* Reduced view distance
* Reduced foliage density
* Reduced visual effects
* Optimized streaming pools

### Goal

*Maximum smoothness over visuals.*

---

# Balanced Performance

Focuses on stable FPS while maintaining acceptable graphics quality.

### Typical Compatible Chipsets

* Snapdragon 720G
* Snapdragon 730G
* Snapdragon 778G
* Snapdragon 845
* Snapdragon 855
* Dimensity 900
* Dimensity 920
* Dimensity 1080

### What's Applied

* Balanced texture quality
* Medium shadows
* Moderate foliage density
* Improved streaming quality
* Optimized world loading

### Goal

*Smooth gameplay with decent visuals.*

---

# Balanced Visual

Improved graphics while maintaining stable performance.

### Typical Compatible Chipsets

* Snapdragon 870
* Snapdragon 888
* Snapdragon 7+ Gen 2
* Dimensity 1200
* Dimensity 1300
* Dimensity 8020

### What's Applied

* Higher shadow quality
* Increased texture quality
* Improved post-processing
* Better foliage rendering
* Increased view distance

### Goal

*Improved visuals without sacrificing stability.*

---

# High End

Maximum visuals for flagship-level devices.

### Typical Compatible Chipsets

* Snapdragon 8 Gen 1
* Snapdragon 8+ Gen 1
* Snapdragon 8 Gen 2
* Snapdragon 8 Gen 3
* Snapdragon 8 Elite
* Dimensity 9000
* Dimensity 9200
* Dimensity 9300
* Dimensity 9400

### What's Applied

* Maximum texture quality
* Extended view distance
* Increased shadow quality
* Enhanced effects
* Increased world streaming range

### Goal

*Highest visual quality with stable high FPS.*

---

# 🎮 Frame Generation

Some presets include KuroFI-related settings where supported by the game.

> Actual frame generation availability depends on the game version, device, graphics API, and Kuro's implementation.

Not all Android devices will benefit from these settings.

---

# 📌 Installation

1. Choose either the **All Devices** or **Snapdragon** folder.
2. Select the preset suitable for your device.
3. Copy the configuration files into:

```text
Internal Storage/Android/data/
com.kurogame.wutheringwaves.global/files/
UE4Game/Client/Client/Saved/Config/Android/
```

4. Overwrite existing files when prompted.
5. Launch the game normally.

> On newer Android versions you may need Shizuku, ZArchiver, MT Manager, or another Android data folder access solution.

---

# 📖 Command Reference

https://alteriax.github.io/WuWa-Config-Info/

---

# 📖 Video Tutorial

https://youtu.be/MvgWzF41Pnc

---

# ☕ Support

If you find these configs helpful, consider supporting the project:

https://ko-fi.com/geilan63

---

# ⚠️ Important Warning

> [!WARNING]
>
> These configs are provided as-is.
>
> Choosing the wrong preset may cause:
>
> * Reduced performance
> * Overheating
> * Increased battery drain
> * Visual issues
> * Game instability
>
> Always back up your original configuration files before replacing anything.

---

# 📌 Notes

* Configs are designed specifically for Wuthering Waves 3.5.
* Snapdragon presets include Adreno-specific tuning.
* All Devices presets are designed to work across multiple Android GPU architectures.
* Future game updates may change or remove supported commands.
* Always verify functionality after major game updates.

---

# 🙏 Credits

* **AlteriaX** — Command reference, documentation, and research.
* **Arglax** — Mobile configuration references and testing methodology.
* **Kuro Game** — Wuthering Waves.

---

# 💻 Similar Repository for WuWa PC

https://github.com/AlteriaX/WuWa-Configs