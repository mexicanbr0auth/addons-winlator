# Winlator Addons - Build Pipeline

GitHub Actions workflows para build de addons do Winlator, customizados para **Mali G57 MC2** e **Helio G99**.

## Components

| Component | Workflow | Description |
|-----------|----------|-------------|
| **DXVK** | `build-dxvk.yml` | DirectX 9/10/11 → Vulkan (GPLALL fork com async) |
| **DXVK-Sarek** | `build-dxvk-sarek.yml` | DXVK para Vulkan 1.1/1.2 (com dyasync) |
| **FEXCore** | `build-fexcore.yml` | Emulação x86 → ARM64 (FEX-Emu) |
| **Proton** | `build-proton.yml` | Wine + DXVK + VKD3D (Proton compat layer) |
| **Build All** | `build-all.yml` | Triggera todos os builds de uma vez |

## How to Use

### Via GitHub UI (Recommended)

1. Go to **Actions** tab in your repo
2. Select the desired workflow
3. Click **Run workflow**
4. Configure options (async, 32/64-bit, etc.)
5. Download artifacts from the completed run

### Via Git Tag (Auto Release)

```bash
# DXVK
git tag dxvk-v2.6.1-custom
git push origin dxvk-v2.6.1-custom

# DXVK-Sarek
git tag dxvk-sarek-v1.12.0-custom
git push origin dxvk-sarek-v1.12.0-custom

# FEXCore
git tag fex-2608
git push origin fex-2608

# Proton
git tag proton-9.0-custom
git push origin proton-9.0-custom
```

## Mali G57 MC2 / Helio G99 Optimizations

All builds include performance optimizations for the target hardware:

### GPU: Mali G57 MC2
- **Async pipeline compilation** - Reduces shader stutter
- **GPL disabled** - Graphics Pipeline Library causes issues on mobile GPUs
- **Tiler batch mode** - Optimized for Mali's tile-based architecture
- **Low-latency frame pacing** - Smoother gameplay

### CPU: Helio G99 (Cortex-A76/A55)
- **FEXCore block size 500** - Optimal for big.LITTLE
- **LTO enabled** - Link-Time Optimization for smaller/faster binaries
- **All cores for shader compilation** - Uses both A76 and A55 cores

### Vulkan
- **DXVK-Sarek**: Targets Vulkan 1.1/1.2 (Mali G57 compatibility)
- **DXVK upstream**: Requires Vulkan 1.3+ (may not work on all Mali drivers)

## Installation in Winlator

### DXVK / DXVK-Sarek
```
winlator/
├── system32/          ← x64 DLLs (d3d9.dll, d3d10core.dll, d3d11.dll, dxgi.dll)
├── syswow64/          ← x32 DLLs
└── dxvk.conf          ← Mali configuration
```

### FEXCore
```
usr/bin/FEXLoader      ← Main loader binary
usr/lib/libFEXCore.*   ← Core library
etc/FEXCore/           ← Configuration files
```

### Proton (full Wine stack)
Replace Wine DLLs in Winlator's rootfs with the built components.

## Build Matrix

| Build | Runner | Compiler | Target |
|-------|--------|----------|--------|
| DXVK | ubuntu-24.04 | mingw-w64 (gcc) | x86_64 Windows PE |
| DXVK-Sarek | ubuntu-24.04 | mingw-w64 (gcc) | x86_64 Windows PE |
| FEXCore | ubuntu-24.04-arm | clang | ARM64 Linux |
| Proton | ubuntu-24.04 | Docker SDK | x86_64 + ARM64 |

## Environment Variables (Runtime)

For DXVK/Proton in Winlator, set these in `dxvk.conf`:

```ini
# Performance
dxvk.enableAsync = True
dxvk.enableGraphicsPipelineLibrary = False
dxvk.enableStateCache = True
dxvk.numCompilerThreads = 0
dxvk.framePace = low-latency
dxvk.maxFrameLatency = 1
dxvk.tilerMode = batch
```

For FEXCore:
```bash
export FEX_BLOCKSIZE=500
export FEX_JIT=1
export FEX_THUNKS=1
```
