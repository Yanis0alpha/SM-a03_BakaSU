# Samsung Galaxy A03 (SM-A035F/M/G) Kernel Source
### Linux 4.14.199 with BakaSU (UAPI 6) & SUSFS v2.3.0 Inline Hook

[![Kernel](https://img.shields.io/badge/Linux-4.14.199-blue.svg)](https://kernel.org)
[![BakaSU](https://img.shields.io/badge/BakaSU-UAPI%206-green.svg)](https://github.com/Baka-SU/BakaSU)
[![SUSFS](https://img.shields.io/badge/SUSFS-v2.3.0%20Inline-orange.svg)](https://gitlab.com/simonpunk/susfs4ksu)
[![Device](https://img.shields.io/badge/Device-Samsung%20Galaxy%20A03-red.svg)](#)

A fully integrated and compile-verified custom kernel tree for the **Samsung Galaxy A03** (`SM-A035F`, `SM-A035M`, `SM-A035G`), featuring **BakaSU** (KernelSU fork) with modern UAPI v6 support and **SUSFS v2.3.0 Inline Hook** (all 9 features verified and active).

---

## 🚀 Features

- **Base Kernel**: Samsung OEM Android Source (Linux 4.14.199).
- **BakaSU UAPI 6**:
  - Pinned commit `d5e9e025b8b49155f8014684d64c91742b465f65`.
  - Integrated via `drivers/kernelsu` tree.
  - Compatible with modern root managers (BakaSU Manager, KernelSU Manager).
- **SUSFS v2.3.0 Inline Hook**:
  - Non-GKI 4.14 backport with inline syscall interception.
  - Full support for stealth root isolation and anti-detection.
  - **9/9 SUSFS functions active**:
    - `susfs_sus_su`
    - `susfs_sus_mount`
    - `susfs_sus_kstat`
    - `susfs_sus_overlayfs`
    - `susfs_try_umount`
    - `susfs_spoof_uname`
    - `susfs_enable_log`
    - `susfs_hide_sus_su`
    - `susfs_spoof_proc_cmdline`
- **Linux 4.14 Compatibility Fixes**:
  - Custom C90 compliant backport for `fs/notify/fdinfo.c` fixing `inotify_mark_user_mask` and `fanotify_fdinfo` callbacks.
  - Adapted `ksu_handle_execveat` and `sys_read` inline hooks to match BakaSU UAPI 6 signatures.

---

## 🛠️ Build Requirements

To compile this kernel, the following AOSP toolchains are recommended:
1. **Clang**: Google AOSP Clang `r383902` (LLVM 11.0.1) from `android12-release`.
2. **GCC (Cross-Compiler)**: Google AOSP GCC 4.9 (`aarch64-linux-android-`) from `android10-qpr1-b-release`.
3. **Host Tools**: `build-essential`, `bc`, `bison`, `flex`, `libssl-dev`, `cpio`.

---

## 📦 How to Build

### 1. Set Environment Variables
```bash
export ARCH=arm64
export SUBARCH=arm64
export CLANG_PATH=/path/to/clang-r383902/bin
export GCC_PATH=/path/to/aarch64-gcc49-google/bin
export PATH="${CLANG_PATH}:${GCC_PATH}:${PATH}"

export CC=clang
export CLANG_TRIPLE=aarch64-linux-gnu-
export CROSS_COMPILE=aarch64-linux-android-
