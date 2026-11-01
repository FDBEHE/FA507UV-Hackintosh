# Current `config.plist` audit (2026-11-01)

This page summarizes the `config.plist` text supplied by the owner. It is a config-only review: the actual EFI folder, `.aml` binaries, kext binaries, USB map contents and OpenCore executable were not supplied. Therefore this is not an `ocvalidate` result and does not prove that every referenced file exists or matches the version comment.

## What the config confirms

- `DeviceProperties` sets `layout-id = 97` for the audio device path. This agrees with the final ALC256 fix.
- NVRAM boot args include `alcid=97 alcverbs=1 alctcsel=1`, `keepsyms=1`, `-vi2c-force-polling`, several NVIDIA/1401-related arguments, and Bluetooth compatibility arguments. There is no `npci=0x3000` in this supplied boot-args string.
- `PlatformInfo -> Generic -> SystemProductName` is `MacBookPro16,2`; SMBIOS serial, MLB, SystemUUID and ROM are populated in the supplied live file. Those identity values are intentionally not repeated here.
- The `Kernel -> Patch` array contains enabled CaseySJ `IOPCIIsHotplugPort` and two `probeBusGated` entries, plus AMD CPUID / PAT and timekeeping patches.
- The config sets `Kernel -> Quirks -> DisableIoMapper = true` and deletes all ACPI `IVRS` tables. This is separate from the BIOS hidden-menu setting: BIOS IOMMU was enabled, but the macOS config suppresses the IVRS table and disables the I/O mapper path. The reason for this exact combination has not been established; keep it as a known-working setting until it can be tested separately.
- The config enables the current USB stack: USBToolBox 1.2.0, UTBMap 1.1, XHCILagFix 0.1.2 and ItlXhciKick (version comment incomplete: `V`).

## ACPI configuration

All of these entries are enabled in `ACPI -> Add`:

| SSDT | Enabled |
|---|---|
| `SSDT-Disable_Network_GPP6.aml` | Yes |
| `SSDT-EC.aml` | Yes |
| `SSDT-PLUG-ALT.aml` | Yes |
| `SSDT-USB-Reset.aml` | Yes |
| `SSDT-USBX.aml` | Yes |
| `SSDT-XOSI.aml` | Yes |
| `SSDT-ALIBFix.aml` | Yes |

`ACPI -> Delete` deletes all `IVRS` tables. `ACPI -> Patch` enables `_OSI` → `XOSI` (requires `SSDT-XOSI.aml`) and AMD `ALIB` → `XLIB` rename. ACPI binary files were not provided, so their AML contents and target paths cannot be confirmed.

## Key kernel entries

All entries below are enabled unless stated otherwise. The version values are the `Comment` strings in the config and may not equal the actual kext binary versions.

| Entry | Comment / version |
|---|---:|
| `Lilu.kext` | 1.7.3 |
| `VirtualSMC.kext` | 1.3.9 |
| `IOSkywalkFamily.kext` | 1.0 |
| `IO80211FamilyLegacy.kext` | 1200.12.2b1 |
| `AirPortBrcmNIC.kext` (nested) | 1400.1.1 |
| `AirPort_RTW89.kext` + `AirPortRTW89Front.kext` | 0.5.0 |
| `AMFIPass.kext` | 1.4.1 |
| `AMDRyzenCPUPowerManagement.kext` / `SMCAMDProcessor.kext` | 3.34.0 |
| `SimpleHDA.kext` | 1.0.0 |
| `RealtekBluetoothFirmware.kext` | 1.0.2 |
| `BlueToolFixup.kext` / `BTSmpFixup.kext` | 2.7.3 |
| `BrightnessKeys.kext` | 1.0.4 |
| `NVMeFix.kext` | 1.1.4 |
| `RealtekRTL8111.kext` | 2.4.2 |
| `RestrictEvents.kext` | 1.1.7 |
| `SMCBatteryManager.kext` / `SMCLightSensor.kext` | 1.3.9 |
| `XHCILagFix.kext` | 0.1.2 |
| `USBToolBox.kext` / `UTBMap.kext` | 1.2.0 / 1.1 |
| `VoodooI2C.kext` | 2.9.1 |
| `VoodooI2CHID.kext` | comment `V1` |
| `VoodooPS2Controller.kext` | 2.3.8 |
| `WhateverGreen.kext` | 1.7.2 |
| `AppleALC.kext` | 1.9.9 |

The config also enables nested `VoodooI2CServices`, `VoodooGPIO`, `VoodooInput`, PS/2 Keyboard, PS/2 Mouse and PS/2 Trackpad plug-ins. The `VoodooPS2Controller`-nested `VoodooInput` entry is disabled.

## OpenCore patches and compatibility ranges

The current plist contains 27 `Kernel -> Patch` entries: 25 enabled and 2 disabled. Notable active entries include:

- CaseySJ `probeBusGated` patch variants for Darwin 21.0.0–24.99.99 and 25.0.0–25.99.99.
- CaseySJ `IOPCIIsHotplugPort` patch for Darwin 22.0.0–25.99.99.
- Visual / laobamac timekeeping patches, including a split for Darwin 25.4.0+ (matching the Tahoe-era kernel family).
- Algrey / Zormeister PAT patch for Darwin 24.0.0–25.99.99.

These version windows are copied from the config. Do not import the whole patch set into another AMD laptop: hardware, kernel and OpenCore versions must match.

## Boot arguments

The supplied `boot-args` string is:

```text
debug=0x100 keepsyms=1 -vi2c-force-polling nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth amfi_get_out_of_my_way=0x1 amfi=0x80 ipc_control_port_options=0 gfxrst=4 nvrmacpibl=1 nvfbhidpi=1 -amdcppcactive -btlfxallowanyaddr -btlfxboardid -btsmpforce alcid=97 alcverbs=1 alctcsel=1
```

Important observations:

- `-vi2c-force-polling` is enabled. It is a useful compatibility fallback for some I2C trackpads, but polling can feel less responsive / use more CPU than a working interrupt mode. This is a plausible item to investigate for the owner's “gestures work but feel poor” report, not a proven cause. Compare only on a cloned EFI.
- `amfi_get_out_of_my_way=0x1`, `amfi=0x80` and `AMFIPass.kext` are security-sensitive compatibility settings. Do not copy them into a general-purpose macOS configuration without understanding why they are needed.
- `alcid=97` matches `DeviceProperties -> layout-id = 97`. Keep the current known-working audio setup unchanged until testing a backup.
- No `npci=0x3000` appears in this config.

For VoodooI2C polling versus interrupt-mode context, see the [VoodooI2C project](https://github.com/VoodooI2C/VoodooI2C) and relevant issue reports; individual trackpad behavior varies by machine.

## Security and publication notes

The supplied config contains live SMBIOS identity values (`SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM`). Never commit this original file publicly. Create a separate public copy and regenerate all values for the intended release. OpenCore documentation maps these fields under `PlatformInfo -> Generic`; see the [OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/clover-conversion/Clover-config.html#smbios).

The encoded `csr-active-config` data is `43 0A 00 00` (little-endian numeric value `0x00000A43`), not the fully enabled SIP value `0x00000000`. It includes permissive flags such as unapproved-kext and unauthenticated-root allowance. The OpenCore Install Guide warns that disabling SIP protections can affect OS functionality and OTA updates, especially with unauthenticated-root enabled; see [Post-Install Issues: SIP](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/post-issues.html).

`Misc -> Security` also has `SecureBootModel = Disabled`, `Vault = Optional`, `ScanPolicy = 0`, and `ExposeSensitiveData = 7`. `Misc -> Debug` has AppleDebug / ApplePanic enabled. This is a working/debug-oriented configuration, not a hardened public release template. `ScanPolicy = 0` intentionally allows OpenCore to scan all boot devices; it is not the same thing as disabling SIP.

## Review items (do not change blindly)

1. **SimpleHDA load order:** in the supplied array `SimpleHDA.kext` appears before `AppleALC.kext`. The upstream [SimpleHDA README](https://github.com/laobamac/SimpleHDA#installation) specifies Lilu → AppleALC → SimpleHDA. Since internal and external audio currently work, preserve a backup and test any reorder on a cloned EFI rather than editing the live boot disk.
2. **Leading tab in MinKernel:** the `Kernel -> Block` entry for `com.apple.iokit.IOSkywalkFamily` has a `MinKernel` string beginning with a tab before `23.0.0`. Clean this to `23.0.0` in a test copy and run the matching OpenCore `ocvalidate`; do not assume the current file passes validation. OpenCore compares `MinKernel` / `MaxKernel` against Darwin kernel versions (see the [OpenCore Configuration reference](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf)).
3. **ItlXhciKick version:** its config comment is only `V`; extract actual `CFBundleVersion` / source commit before documenting a release.
4. **OpenCore version:** not encoded in this plist. Obtain `OpenCore.efi` / boot log version before publishing.
5. **UTBMap:** the `UTBMap.kext` bundle was not supplied; actual USB port mapping, Bluetooth USB port and connector types remain unknown.
6. **SSDT files:** names and enabled flags were supplied, but no AML binaries were included for disassembly / dependency checks.

This audit is not a recommendation to alter a stable daily-use EFI. Keep a known-good backup and make one change at a time.
