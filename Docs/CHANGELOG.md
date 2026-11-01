# Troubleshooting changelog

This timeline is reconstructed from the owner's notes. Dates are included only where retained; some intermediate changes are not dated precisely.

## Initial EFI and boot

- First 1401 Probe / EFI iteration: boot stopped around IIC / PCI-related verbose output.
- Second EFI iteration: added the CaseySJ `IOPCIIsHotplugPort` AM5 PCI enumeration patch and other platform quirks; the system could reach the desktop.
- After reaching desktop, internal laptop keyboard and touchpad worked, but USB devices were very delayed / stuttered. The custom USB map alone did not solve the symptom.

## USB

- XHCILagFix was introduced and resolved broad lag for USB flash drives, keyboard and mouse.
- USB audio was still problematic: no sound / video stutter; later XHCILagFix 0.1.1 plus ItlXhciKick returned audio but left periodic stutter and popping.
- Toggling ItlXhciKick did not solve the remaining issue.
- After moving to macOS Tahoe 26.7.2, USB audio reportedly became normal without a clearly isolated configuration change. Current owner report: USB setup works.

## Wireless

- Realtek RTL8852BE Wi-Fi (`10EC:B852`) works after adding the RTW89-related driver stack (`AirPort_RTW89`, `IO80211FamilyLegacy`, `IOSkywalkFamily`). Exact original sequence is not retained.
- Realtek Bluetooth radio (`13D3:3571`) was repaired. Current EFI contains `RealtekBluetoothFirmware`, `BlueToolFixup` and `BTSmpFixup`; exact original repair sequence is not retained.

## Graphics and macOS versions

- Owner reports 1401 graphics driver 2.6.5, dual display output at 2K 165 Hz and 4K 160 Hz, smooth operation and normal video decoding.
- On macOS 15, an unselected-window black-text rendering issue was observed; the owner says it did not appear on Tahoe 26.7.2.

## Audio — ALC256

- Built-in ALC256 speakers were silent while 3.5 mm output generally worked.
- Tested existing layouts including 66 and 97; neither initially solved speakers.
- Created an experimental custom Layout 15 based on adjusted pin defaults. With `alcid=15`, built-in audio endpoints disappeared; returning to `alcid=97` restored them.
- HDA stream / codec inspection confirmed that audio streams were assigned to DAC `0x02` and DAC `0x03` during playback; simple Pin amp / EAPD changes alone did not restore speakers.
- Read processing coefficients on NID `0x20`; changed index `0x10` from `0x0220` to `0x7F20`, index `0x16` from `0x0E50` to `0x0C50`, and enabled GPIO2 while preserving GPIO0: mask `0x05`, direction `0x05`, data `0x04`.
- This combination produced sound from the built-in speakers immediately.
- Embedded the seven initialization verbs in Layout 97 `ConfigData` and `WakeConfigData`, rebuilt AppleALC Release with Xcode 26.6, validated the merged plist, replaced AppleALC in EFI, retained `alcid=97`, and confirmed sound after reboot without manual verb commands.

## Power and miscellaneous

- CPU power management appears functional with AMDRyzenCPUPowerManagement and SMCAMDProcessor present; informal boost / power readings are recorded in `Graphics-CPU-Power.md`.
- Battery status was initially not displayed correctly despite current changes being detectable; a custom patch was made, but exact details are unknown.
- Sleep not tested; lid close had not slept correctly in an earlier test. Shutdown hung at a verbose code once and later seemed to work; cause unknown.

## Current status (owner report)

- Tahoe 26.7.2.
- Internal ALC256 speakers work after permanent Layout 97 coefficient / GPIO initialization was embedded in AppleALC.
- Owner does not use sleep and has chosen not to pursue sleep/wake debugging.
- Owner reports Wi-Fi, Bluetooth, ordinary USB and external display functionality working; precise regression tests and exact versions are not all retained.

## 2026-11-01 — final functional-status update and config audit

- Confirmed by owner: external USB keyboard, mouse, flash drive and USB headset audio work; internal laptop keyboard works; built-in speakers and 3.5 mm audio output work.
- Touchpad pointer / gestures are supported, but user experience is poor. Current config includes `-vi2c-force-polling`; this is recorded as a possible investigation point, not a proven cause.
- Wi-Fi, wired Ethernet and Apple ID sign-in work.
- Bluetooth keyboard connects and works; iPhone 12 cannot connect, so Bluetooth remains partially functional.
- Microphone input meter responds in Sound settings, but actual recording / call test is pending.
- Lid-close sleep remains broken (fan continues running); keyboard power button not tested.
- Latest CPU observation: desktop light-load power about 6–10 W, all-core load about 48 W. GPU video decoding works normally.
- Parsed the supplied `config.plist` and added `Config-Audit.md`. Confirmed enabled kext / ACPI / patch entries and flagged live SMBIOS identifiers, `csr-active-config`, SimpleHDA load order, and a leading tab in one `MinKernel` string for review.
