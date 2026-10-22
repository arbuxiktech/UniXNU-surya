# UniXNU Surya

This directory contains the UniXNU device configuration for the Xiaomi POCO X3
NFC (codename surya, SM7150). It uses the iPad7,1 iPadOS 17.7.11 (21H461) IPSW
as the Darwin 23 kernel and userland donor. This is a bring-up configuration;
it does not mean iPadOS supports the POCO hardware or that a complete UniXNU
boot has been verified.

## Layout and upstream

The generic boot manager, kernel sources, and common drivers are in the parent
UniXNU repository. This directory supplies the Surya board configuration. The
firmware configuration points at the existing Surya platform in the separate
[uefi-sheng](https://github.com/UniXNU/uefi-sheng) checkout under
Platforms/Xiaomi/suryaPkg and Resources/Configs/surya.toml.

The separate device-specific archive contains stock device-tree references and
experimental Surya drivers/firmware. Proprietary firmware blobs are not copied
into this repository.

## Current status

The board, Darwin 23 source pins, and firmware path are configured. The initial XNU
configuration is for conservative single-core entry work. UFS, APFS root
assembly, display, touch, SMP, and Adreno are not enabled in this baseline.
No UniXNU kernel build or full iOS userspace boot is claimed.

The Darwin 23 build uses UniXNU's default XNU 10063.141.1 source and patch set,
with iOS 17 deployment settings. The donor is iPadOS because iPhone X has no
iOS 17 restore image. The output name matches the kernel expected by the
prelink step.

The payload and userdata sections are intentionally absent. The stored stock
DTB has a zeroed /memory reg property, so it does not verify total DRAM. The
userdata GPT geometry and root volume identifiers also need device-specific
measurements. Do not add guessed values: the ADT and boot payload depend on
them. The previous Surya logs and screenshots describe separate experimental
images, not proof that this UniXNU configuration boots.

## Validate configuration

On Windows, from the UniXNU repository root in PowerShell:

    py .\unixnu defconfig surya_defconfig -D .\ports\surya --out .\out\surya

    py .\unixnu status -D .\ports\surya --out .\out\surya --uefi ..\uefi-sheng

On this Windows host, defconfig, status, and source-cache setup have been validated. UniXNU tree assembly currently stops because Windows denies directory symlink creation (WinError 1314). The kernel build requires macOS and Xcode; --skip-setup also needs hdiutil.

## Build on GitHub Actions

The `Surya macOS kernel build` workflow runs the pinned Darwin 23 kernel build
on a GitHub-hosted macOS Intel runner. It runs on changes under `ports/surya`
or when manually dispatched from the Actions tab. The optional `unixnu_ref`
input selects a branch, tag, or commit from this fork; by default it builds the
same revision as the board configuration. The workflow uploads the built
kernel and build logs for seven days. It does not build a phone-flashable image
or flash the phone.

This is a kernel-build milestone only. The board does not yet contain verified
Surya DRAM geometry, an Apple device tree, or a payload configuration, so a
successful CI build would not mean the result is bootable on the Poco.

## Bring-up order

1. Build/boot the existing Surya UEFI profile and confirm its menu and payload
   media path.
2. Build a Darwin 23 kernel with Surya's verified Apple device tree and reach
   XNU platform initialization.
3. Measure DRAM, establish a safe memory map, and create the Surya Apple device
   tree (ADT).
4. Add read-only UFS discovery, verify GPT/APFS identities, then prepare the
   matching iOS root and Data volumes.
5. Reach launchd with storage and userland before adding touch/display services.
6. Bring up Novatek NT36672C touch, display and Adreno 618 one component at a
   time.

See the official [UniXNU porting guide](https://unixnu.com/porting/),
[device repository guide](https://unixnu.com/porting/device-repository),
[device-tree guide](https://unixnu.com/porting/device-tree), and
[bring-up stages](https://unixnu.com/porting/bring-up).
