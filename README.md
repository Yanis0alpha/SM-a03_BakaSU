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
- Pinned to commit `d5e9e025b8b49155f8014684d64c91742b465f65`.
- Integrated directly via a `drivers/kernelsu` symbolic link (`drivers/kernelsu -> ../KernelSU/kernel`).
- Full compatibility with modern userspace managers (**BakaSU Manager** and the official **KernelSU Manager**, both supporting UAPI 6).
- Unrestricted app profiling and superuser access control, with no modifications to `/system` or `/vendor`.

### 2. SUSFS v2.3.0 Inline Hook (Non-GKI 4.14 Backport)
- Complete syscall-level inline interception, without relying on GKI 2.0 tracepoints.
- **All 9 SUSFS evasion features active and validated**:
  1. `susfs_sus_su` — Hides the `su` binary and internal root mountpoints.
  2. `susfs_sus_mount` — Real-time hiding of sensitive mounts from `/proc/mounts` and `/proc/self/mountinfo`.
  3. `susfs_sus_kstat` — Spoofs file metadata (inode, dev, mtime) against filesystem integrity checks.
  4. `susfs_sus_overlayfs` — Isolates overlayfs layers used by root modules.
  5. `susfs_try_umount` — Automatically unmounts root-related mounts in untrusted app namespaces.
  6. `susfs_spoof_uname` — Spoofs the kernel release string to prevent detection of custom builds.
  7. `susfs_enable_log` — Configurable logging interface for debugging.
  8. `susfs_hide_sus_su` — Prevents detection of the supervisor subsystem.
  9. `susfs_spoof_proc_cmdline` — Spoofs `/proc/cmdline` to conceal unlocked bootloader flags.

### 3. Custom Samsung 4.14 Compatibility Backports
- **`fs/notify/fdinfo.c` (C90 Compliance & Legacy Fix)**:
  - Custom backport of `inotify_mark_user_mask()`, supporting legacy Linux 4.14 event bitmasks (`IN_ALL_EVENTS | IN_ONESHOT | IN_EXCL_UNLINK`).
  - Strict ANSI C90 variable placement (`u32 mask;` declared at function scope to eliminate compiler syntax errors).
  - Fixed `fanotify_fdinfo` callback signatures.
- **Syscall Hook Corrections**:
  - `fs/exec.c`: stack variables are now passed by address to `ksu_handle_execveat()`, avoiding pointer-casting errors.
  - `fs/read_write.c`: `ksu_handle_sys_read()` updated to match the BakaSU UAPI 6 multi-argument signature (`&buf`, `&count`).

---

## 🛠️ Toolchains & Build Requirements

To compile this kernel cleanly, use Google AOSP prebuilt toolchains:
- **Clang Compiler**: Google AOSP Clang `r383902` (LLVM / Clang 11.0.1) from `android12-release`.
- **GCC Cross-Compiler**: Google AOSP GCC 4.9 (`aarch64-linux-android-4.9`) from `android10-qpr1-b-release`.
- **Host Dependencies** (Debian/Ubuntu):
  ```bash
  sudo apt-get update && sudo apt-get install -y \
      build-essential bc bison flex libssl-dev cpio libelf-dev python3 git
  ```

---

## 📦 Compilation Guide

### Step 1: Export Environment Variables
```bash
export ARCH=arm64
export SUBARCH=arm64
export CLANG_PATH=/path/to/clang-r383902/bin
export GCC_PATH=/path/to/aarch64-gcc49-google/bin
export PATH="${CLANG_PATH}:${GCC_PATH}:${PATH}"

export CC=clang
export CLANG_TRIPLE=aarch64-linux-gnu-
export CROSS_COMPILE=aarch64-linux-android-
```

### Step 2: Configure the Target Defconfig
```bash
make clean && make mrproper
make a03_eur_open_defconfig
```

### Step 3: Build the Kernel Image
```bash
make -j$(nproc --all) Image.gz
```
The compiled kernel binary will be produced at:
`arch/arm64/boot/Image.gz`

---

## 📲 Flashing Guide

1. **Extract the stock `boot.img`**: pull the original `boot.img` from your device firmware (`AP_...tar.md5`).
2. **Unpack**: use [Android Image Kitchen (AIK)](https://forum.xda-developers.com/t/tool-android-image-kitchen-unpack-repack-kernel-ramdisk-win-android-linux-mac.2073775/) or Magisk's boot tools.
3. **Replace the kernel**: replace `split_img/boot.img-zImage` (or `kernel`) with your newly built `arch/arm64/boot/Image.gz`.
4. **Repack**: repack the image to generate `image-new.img`.
5. **Flash**:
   - **Via Odin**: convert `image-new.img` to `boot.img.tar` (`tar -cvf boot.img.tar boot.img`) and flash it in the `AP` slot.
   - **Via Fastboot / TWRP**:
     ```bash
     fastboot flash boot image-new.img
     ```
6. **Install a manager**: reboot to system and install **BakaSU Manager** Apk Get the [UAPI 6-compatible BakaSU Manager](https://github.com/zeroedx/BakaSU_CI/releases/tag/BakaSU_40109566037).

---

## 🌟 Credits & Acknowledgments

Our thanks go to the developers and organizations whose work made this project possible:

| Project / Contributor | Description & Contributions |
| :--- | :--- |
| **[Samsung Open Source](https://opensource.samsung.com/)** | Upstream Linux 4.14.199 source tree and vendor platform drivers for the Galaxy A03. |
| **[KernelSU Team](https://github.com/tiann/KernelSU)** | The pioneering kernel-space root solution for modern Android. |
| **[Baka-SU Team](https://github.com/Baka-SU/BakaSU)** | Downstream KernelSU fork providing modern UAPI 6 backports and legacy kernel adaptations. |
| **[simonpunk (susfs4ksu)](https://gitlab.com/simonpunk/susfs4ksu)** | Creator of SUSFS (Super User Snoop File System), the premier kernel-level root-hiding framework. |
| **[JackA1ltman (NonGKI_Kernel_Build_2nd)](https://github.com/JackA1ltman/NonGKI_Kernel_Build_2nd)** | Non-GKI 4.14 reference patches and inline hook scripts. |
| **[Yanis0alpha](https://github.com/Yanis0alpha)** | Galaxy A03 integration, UAPI 6 syscall hook adaptations, C90 legacy notify compatibility fixes, and hardware testing. |

---

## ⚖️ License & Legal Notices

This repository contains code covered by open-source copyleft licenses:

- **Linux Kernel Source**:
  - The Linux kernel is licensed under the [GNU General Public License Version 2 only (GPL-2.0-only)](COPYING), as specified by Linus Torvalds and the Linux Foundation.
- **BakaSU (Kernel Subdirectory)**:
  - All kernel-space code in `KernelSU/kernel/` is licensed under **GPL-2.0-only** ([KernelSU/kernel/LICENSE](KernelSU/kernel/LICENSE)).
  - Userspace management tools and non-kernel components are licensed under **GPL-3.0-or-later**.
- **SUSFS**:
  - Upstream project developed by simonpunk under GPL licensing.
- **Corresponding Source**:
  - In accordance with Section 3 of the GNU General Public License v2.0, this repository constitutes the complete corresponding source code required to build, configure, and install the modified kernel executable.

---

### ⚠️ Disclaimer
Modifying and flashing device kernels carries inherent risks, including potential bootloops, data loss, or voiding of the device warranty. This software is provided "AS IS", without warranty of any kind, express or implied.
