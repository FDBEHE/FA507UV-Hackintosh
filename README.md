# ASUS TUF Gaming A15 FA507UV — Hackintosh Project

> **本仓库是个人实机记录，不是通用 EFI，也不是对所有 FA507UV 批次的兼容性承诺。** 内容区分了已验证结果、用户回忆、尚未定位的配置细节和未测试项目。复用前请核对硬件 ID、BIOS、OpenCore、kext 版本和 USB 映射。

**English summary:** OpenCore/macOS notes for the ASUS TUF Gaming A15 FA507UV with AMD Ryzen 7 8845H. Includes firmware notes, AM5 PCI enumeration workaround, Realtek RTL8852BE Wi-Fi, Realtek Bluetooth, AMD USB/XHCI troubleshooting, graphics observations, power management notes and a custom AppleALC ALC256 internal-speaker fix.

## At a glance

| Item | This machine / observed value |
|---|---|
| Laptop | ASUS TUF Gaming A15 FA507UV |
| CPU | AMD Ryzen 7 8845H, Phoenix/Hawk Point, 8C/16T |
| BIOS | FA507UV.312, report date string `01/06/2025` |
| macOS | Tahoe 26.7.2 (owner-reported current working system) |
| SMBIOS | Earlier IORegistry observation: `MacBookPro16,2`; regenerate your own identity before publishing an EFI |
| Wi-Fi | Realtek RTL8852BE PCIe, `10EC:B852` |
| Bluetooth | Realtek Bluetooth Radio, USB ID `13D3:3571` |
| Ethernet | Realtek RTL8111/8168 family, `10EC:8168` |
| Audio | Realtek ALC256, `10EC:0256`, subsystem `1043:1C23` |
| GPU (Probe report) | NVIDIA AD107M RTX 4060 Max-Q `10DE:28E0`; AMD HawkPoint1 `1002:1900` |
| Storage | Samsung `MZVL8512HELU-00BTW`; WD Blue SN5000 1TB |
| Probe | 1401 Probe 2.2.0 report dated 2026-10-30 |

The hardware inventory above is summarized from the owner's 1401 Probe output and troubleshooting notes. The exact OpenCore version, ACPI binary contents, BIOS hidden-menu navigation, USB port map and several kext source revisions are not yet documented.

## Current status summary (owner testing, 2026-11-01)

| Area | Current result | Status / caveat |
|---|---|---|
| Boot / PCI enumeration | Boots to desktop with CaseySJ `IOPCIIsHotplugPort` patch enabled | Owner reports disabling the patch causes a boot hang |
| Internal laptop keyboard | Works | Tested by owner |
| Touchpad | Pointer and gestures work | Gesture support exists, but the feel / usability is poor; currently uses `-vi2c-force-polling` (see Config audit) |
| External USB | Keyboard, mouse, USB flash drive and USB audio headset work | Current `config.plist` enables USBToolBox, UTBMap, XHCILagFix and ItlXhciKick |
| Internal audio | Built-in speakers work | Custom ALC256 Layout 97 coefficient + GPIO verbs are embedded in AppleALC |
| 3.5 mm audio | Headphone jack works | Owner now reports normal use |
| Microphone | Input meter in macOS Sound settings moves when speaking | Not yet tested by recording / call; do not mark microphone as verified working |
| Wi-Fi | Works | RTL8852BE / RTW89 driver stack; Apple ID sign-in also succeeds |
| Ethernet | Works | Realtek RTL8111/8168 family |
| Bluetooth | Keyboard pairs and works | iPhone 12 cannot connect; Bluetooth is therefore only partially verified |
| Apple services | Owner can sign in to Apple ID | This does not establish that AirDrop, Handoff, iMessage, FaceTime or all Continuity features work |
| GPU / video | Graphics driver and video decoding work normally | Owner also reports dual displays at 2K 165 Hz + 4K 160 Hz with 1401 driver 2.6.5 |
| CPU power | Boost works; desktop idle/light load about 6–10 W; all-core load about 48 W | Owner-observed values, not standardized benchmarks |
| Sleep / lid close | Lid close does not enter sleep correctly; fans continue running | Unresolved; owner does not rely on sleep |
| Keyboard power button | Not tested | Explicitly not tested yet |
| Battery / shutdown | Earlier observations only | Full final-state verification is still missing |

The latest status supersedes earlier broad statements such as “Bluetooth working” or “USB audio working”: Bluetooth keyboard works but iPhone 12 pairing does not; USB audio is now reported working; microphone and sleep remain unverified / unresolved.

## Current config snapshot

The attached `config.plist` was parsed for this documentation update. It confirms the enabled kext entries, ACPI tables, boot arguments, kernel patches and Layout 97 device property. A separate [Config audit](Docs/Config-Audit.md) records the relevant values and review items. The file also contained populated SMBIOS identity fields, so those values are intentionally not reproduced in this repository.

## Critical OpenCore patch: AM5 PCI enumeration

The following entry is required by the owner's working configuration:

```text
CaseySJ | IOPCIIsHotplugPort | Fix PCI bus enumeration on AM5 | 13.0+
```

- OpenCore path: `Kernel -> Patch`.
- Owner's required setting: **Enabled = true**.
- Owner reports that with this patch unchecked, startup hangs; enabling it was part of the second EFI iteration that reached the desktop.
- The patch description refers to AM5 PCI bus enumeration. The FA507UV itself is a mobile Phoenix/Hawk Point Ryzen laptop; do not describe it as a desktop AM5-socket motherboard.
- Record the exact patch Find/Replace/Mask/Comment/MinKernel values from the actual sanitized `config.plist` before publishing a reusable EFI. This repository currently records the name and observed requirement, not the full binary patch dictionary.

## Main troubleshooting chapters

- [BIOS, boot flow and OpenCore](Docs/BIOS-OpenCore.md)
- [Wi-Fi and Bluetooth](Docs/WiFi-Bluetooth.md)
- [USB / XHCI and USB audio](Docs/USB.md)
- [ALC256 internal-speaker repair](Docs/Audio-ALC256.md)
- [GPU, CPU, battery and power management](Docs/Graphics-CPU-Power.md)
- [EFI kext inventory](Docs/Kext-Inventory.md)
- [Current config.plist audit](Docs/Config-Audit.md)
- [Chronological changelog](Docs/CHANGELOG.md)

## Public EFI safety checklist

**Do not upload the live EFI folder or the supplied live `config.plist` without review.** The supplied config contains populated SMBIOS identifiers. Make a separate `EFI-Public` copy first and regenerate / replace those identifiers before publishing.

1. Replace or remove personal SMBIOS values: `SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM`.
2. Search all files for Wi-Fi passwords, API tokens, private keys, certificates, account names, personal paths and network identifiers.
3. Do not include FileVault recovery keys or personal Preboot / APFS data.
4. Keep only one intended `UTBMap.kext`; do not upload `UTBMap.kext.nullmoth-*` working backups as active kexts.
5. Remove build outputs, Xcode DerivedData, local source checkouts, temporary logs and duplicate kexts unless intentionally archived as a documented experiment.
6. Review every third-party kext's license and source; retain attribution and license files.
7. Publish a sanitized `config.sample.plist`, not an unreviewed live `config.plist`.
8. Add a clear note that hardware IDs and BIOS options differ between revisions; users must make their own backups and understand their own hardware.

See [EFI publication notes](EFI/README.md) and `.gitignore`.

## Evidence labels used in this repository

- **Verified on this machine:** a command output or successful real-world test was observed in this troubleshooting history.
- **Owner-reported:** the owner reports the result, but no raw test log was retained here.
- **Uncertain / needs confirmation:** the owner does not remember the exact setting, cause, or final state.
- **Not tested:** no claim is made.

## Contributing / issue reports

When reporting a similar FA507UV issue, include the exact CPU model, BIOS version, macOS version/build, PCI/USB hardware IDs, OpenCore version, kext versions, and a sanitized config excerpt. Do not post serial numbers, MLB, UUID, MAC addresses, passwords or FileVault recovery keys.
