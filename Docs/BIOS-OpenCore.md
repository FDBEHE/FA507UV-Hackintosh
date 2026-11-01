# BIOS, OpenCore and boot flow

## Firmware facts from 1401 Probe

| Field | Reported value |
|---|---|
| Manufacturer / model | ASUSTeK COMPUTER INC. ASUS TUF Gaming A15 FA507UV |
| Board | FA507UV |
| BIOS | FA507UV.312 |
| BIOS date string | `01/06/2025` (kept exactly as Probe printed it; locale interpretation not confirmed) |
| Firmware | 64-bit UEFI |
| Secure Boot | Off |
| CFG Lock | n/a in Probe output |
| VT-d / IOMMU | IOMMU table present in Probe output |
| Firmware GOP | NVIDIA AD107M GOP present; AMD HawkPoint1 GOP not readable / reported absent |

## Hidden AMD firmware menu

The owner reports that the visible BIOS setup does **not** expose the usual `Above 4G Decoding` and `IOMMU` controls. These options were accessed through an AMD hidden settings menu exposed using a third-party project/tool (the project name and exact menu path are no longer remembered). Both `Above 4G Decoding` and `IOMMU` were explicitly enabled there.

Do not invent the path or claim the visible stock BIOS has these options. The exact hidden-menu method needs to be rediscovered and documented from the machine before others follow it. Opening hidden firmware menus and changing undocumented options carries risk; record original values and make recovery preparations first.

## OpenCore / AM5 PCI enumeration patch

The working setup requires this entry to be enabled according to owner testing:

```text
CaseySJ | IOPCIIsHotplugPort | Fix PCI bus enumeration on AM5 | 13.0+
```

- Location: `Kernel -> Patch`.
- Required owner-reported state: `Enabled = true`.
- The first 1401 EFI version hung around IIC / PCI startup stages.
- The second iteration added the AM5 PCI patch and other quirks and could reach the desktop.
- With the second EFI, laptop keyboard and touchpad worked, while USB devices still had severe problems; USB was addressed separately with XHCILagFix.

The current supplied `config.plist` confirms that the patch is enabled for Darwin kernel versions 22.0.0–25.99.99. It also contains two CaseySJ `probeBusGated` patches (for Darwin 21–24.99.99 and 25.0.0–25.99.99). The precise set of “other quirks” from the second iteration is not available in the earlier notes. Do not claim the AM5 patch alone fixed every initial boot issue.

## SMBIOS and NVRAM

An earlier IORegistry inspection showed SMBIOS model `MacBookPro16,2`. Verify the currently active SMBIOS before publishing.

The supplied config uses `MacBookPro16,2` as `PlatformInfo -> Generic -> SystemProductName`. The serial number, MLB, SystemUUID and ROM are populated in the live file; they are intentionally omitted from this documentation. Apple ID sign-in reportedly works with this identity, but a public EFI needs its own properly generated identity.

Audio boot arguments used after the final fix:

```text
alcid=97 alcverbs=1 alctcsel=1
```

The owner also asked about `npci=0x3000`, but the available notes do not establish whether it is currently present in the final boot arguments. Do not list it as a confirmed required argument without checking the live `config.plist`.

## FileVault and unusual boot order

`fdesetup status` returned `FileVault is On.` The owner observed this sequence:

1. Power on and enter OpenCore.
2. Select macOS.
3. A login / password screen appears before verbose boot.
4. After password entry, verbose boot messages appear.
5. The Apple logo and progress bar appear after the verbose messages.
6. Desktop loads.

This is consistent with FileVault Preboot authentication: the startup volume is unlocked before the installed macOS system continues booting. It is not evidence by itself of an OpenCore boot-order failure.

## Startup / sleep / shutdown observations

- Initial EFI revision 1 reportedly hung around IIC / PCI-related boot text.
- Revision 2, after adding the AM5 patch and other quirks, reached desktop.
- Sleep was not systematically tested.
- Closing the lid did not sleep correctly in an earlier test; fan noise was also mentioned.
- Shutdown previously hung at a verbose code, then later appeared to work; cause unknown.
- Current owner does not rely on sleep and has chosen not to pursue sleep/wake fixes.

The current config also sets `Kernel -> Quirks -> DisableIoMapper = true` and deletes all ACPI `IVRS` tables. This is separate from the firmware menu state: although Above 4G Decoding and IOMMU were enabled in the hidden BIOS menu, the current macOS boot config is explicitly configured not to use the normal I/O-mapper path / IVRS table. The reason for this combination has not been recorded; preserve it in the known-working copy and do not toggle it casually.

## Before publishing a config

Extract and document the exact current values for:

- OpenCore version and build date.
- ACPI `Add`, `Delete`, `Patch` contents.
- Booter quirks and Kernel quirks.
- Exact CaseySJ patch dictionary and enabled state.
- `Kernel -> Add` entries and their `Enabled` flags.
- DeviceProperties and NVRAM entries.
- SMBIOS model (identity values must be regenerated / redacted).
- Audio boot arguments.
- Whether the IOMMU / IVRS suppression is still required on the current macOS build.

Do not publish serial numbers, MLB, SystemUUID or ROM/MAC values.
