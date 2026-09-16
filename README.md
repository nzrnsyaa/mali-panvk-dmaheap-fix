# Mali PanVK dma-heap SELinux Fix

Fix `dma-heap permission denied` untuk PanVK pada Mali GPUs dengan KernelSU root.

## Problem

```
[MESA-PANVK] kbase: dma-heap unavailable at /dev/dma_heap/system: Permission denied
```

Blocks hardware dma-buf present path, forces software fallback.

## Root Cause

Two layers must both be fixed:

| Layer | Problem | Fix |
|-------|---------|-----|
| DAC | cr--r--r-- (444) — no write for Termux user | chmod 666 |
| SELinux | untrusted_app_27 blocked from dmabuf_system_heap_device | sepolicy rule |

Fixing only one fails.

## Install

```bash
su -c 'cp -r mali-panvk-dmaheap-fix /data/adb/modules/dma_heap_selinux_fix'
su -c 'reboot'
```

## Verify

```bash
su -c 'ls -la /dev/dma_heap/system'   # expected: crw-rw-rw-
su -c 'getenforce'                     # expected: Enforcing
vulkaninfo | grep dma-heap             # expected: no output
```

## Tested

- Device: Infinix X6873 (Dimensity 8350)
- GPU: Mali-G615 MC6 (Valhall v11)
- OS: Android 15 + KernelSU
- Result: dma-heap works, SELinux Enforcing, vkcube 114 FPS

## Requires

- KernelSU root (uses ksud sepolicy)
- PanVK driver: https://github.com/wonderkast02/panvk-g720-kbase-csf

## Credit

- PanVK driver: [wonderkast02](https://github.com/wonderkast02)
- dma-heap fix: [nzrnsyaa](https://github.com/nzrnsyaa)

## License

MIT
