# Samsung Galaxy A03 (SM-A035F / SM-A035M / SM-A035G) Kernel Source
### Linux 4.14.199 with BakaSU (UAPI 6) & SUSFS v2.3.0 Inline Hook

[![Kernel Version](https://img.shields.io/badge/Kernel-Linux%204.14.199-blue.svg)](https://kernel.org)
[![BakaSU](https://img.shields.io/badge/BakaSU-UAPI%206-brightgreen.svg)](https://github.com/Baka-SU/BakaSU)
[![SUSFS](https://img.shields.io/badge/SUSFS-v2.3.0%20Inline-orange.svg)](https://gitlab.com/simonpunk/susfs4ksu)
[![Target Architecture](https://img.shields.io/badge/Arch-arm64--v8a-purple.svg)](#)
[![Device](https://img.shields.io/badge/Device-Samsung%20Galaxy%20A03-red.svg)](#)
[![License](https://img.shields.io/badge/License-GPL--2.0--only-yellow.svg)](COPYING)

A fully integrated, compile-verified, and hardware-tested custom kernel tree for the **Samsung Galaxy A03** (`SM-A035F`, `SM-A035M`, `SM-A035G`), featuring **BakaSU** (KernelSU fork) with modern UAPI v6 support and **SUSFS v2.3.0 Inline Hook** with all 9 core evasion features active.

---

## 📱 Device Specifications & Compatibility

| Attribute | Specification |
| :--- | :--- |
| **Device Model** | Samsung Galaxy A03 (`SM-A035F`, `SM-A035M`, `SM-A035G`) |
| **Chipset / Platform** | Unisoc T606 / UMS9230 (Sharkl3 / sp9863a tree) |
| **Architecture** | ARM64 (`aarch64` / `arm64-v8a`) |
| **Kernel Base** | Samsung OEM Stock Linux `4.14.199` |
| **Root Solution** | BakaSU (KernelSU fork) pinned to commit `d5e9e025b8b49155f8014684d64c91742b465f65` |
| **Stealth Engine** | SUSFS (Super User Snoop File System) v2.3.0 Inline Hook (Non-GKI 4.14) |
| **Defconfig Target** | `arch/arm64/configs/a03_eur_open_defconfig` |

---

## 🚀 Key Features & Kernel Modifications

### 1. BakaSU Root Integration (UAPI 6)
- Pinned commit `d5e9e025b8b49155f8014684d64c91742b465f65`.
- Integrated directly via `drivers/kernelsu` symbolic link (`drivers/kernelsu -> ../KernelSU/kernel`).
- Full compatibility with modern userspace managers (**BakaSU Manager** and official **KernelSU Manager** supporting UAPI 6).
- Unrestricted app profiling and superuser access control without modifying `/system` or `/vendor`.

### 2. SUSFS v2.3.0 Inline Hook (Non-GKI 4.14 Backport)
- Complete syscall-level inline interception without relying on GKI 2.0 tracepoints.
- **All 9 SUSFS evasion features active and validated**:
  1. `susfs_sus_su` — Hides `su` binary and internal root mountpoints.
  2. `susfs_sus_mount` — Real-time hiding of sensitive mounts from `/proc/mounts` and `/proc/self/mountinfo`.
  3. `susfs_sus_kstat` — Spoofs file metadata (inode, dev, mtime) against filesystem integrity checks.
  4. `susfs_sus_overlayfs` — Isolates overlayfs layers used by root modules.
  5. `susfs_try_umount` — Automatically unmounts root-related mounts in untrusted app namespaces.
  6. `susfs_spoof_uname` — Spoofs kernel release string to prevent detection of custom builds.
  7. `susfs_enable_log` — Configurable logging interface for debugging.
  8. `susfs_hide_sus_su` — Prevents detection of the supervisor subsystem.
  9. `susfs_spoof_proc_cmdline` — Spoofs `/proc/cmdline` to conceal unlocked bootloader flags.

### 3. Custom Samsung 4.14 Compatibility Backports
- **`fs/notify/fdinfo.c` (C90 Compliance & Legacy Fix)**:
  - Custom backport of `inotify_mark_user_mask()` supporting legacy Linux 4.14 event bitmasks (`IN_ALL_EVENTS | IN_ONESHOT | IN_EXCL_UNLINK`).
  - Strict ANSI C90 variable placement (`u32 mask;` declared at function scope to eliminate compiler syntax errors).
  - Fixed `fanotify_fdinfo` callback signatures.
- **Syscall Hook Corrections**:
  - `fs/exec.c`: Passed stack variables by address to `ksu_handle_execveat()` avoiding pointer casting errors.
  - `fs/read_write.c`: Updated `ksu_handle_sys_read()` to match BakaSU UAPI 6 multi-argument signature (`&buf`, `&count`).

---

## 🛠️ Toolchains & Build Requirements

To compile this kernel cleanly, use Google AOSP prebuilt toolchains:
- **Clang Compiler**: Google AOSP Clang `r383902` (LLVM / Clang 11.0.1) from `android12-release`.
- **GCC Cross-Assembler**: Google AOSP GCC 4.9 (`aarch64-linux-android-4.9`) from `android10-qpr1-b-release`.
- **Host Dependencies** (Debian/Ubuntu):
  ```bash
  sudo apt-get update && sudo apt-get install -y \
      build-essential bc bison flex libssl-dev cpio libelf-dev python3 git
