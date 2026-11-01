# USB / XHCI troubleshooting record

## Observed hardware / EFI components

The machine is an AMD Ryzen 7 8845H Phoenix / Hawk Point platform. The owner supplied an `OC/Kexts` directory listing containing:

- `USBToolBox.kext`
- `UTBMap.kext`
- `XHCILagFix.kext`
- `ItlXhciKick.kext`
- `Lilu.kext`

The listing also contained three timestamped backup folders:

- `UTBMap.kext.nullmoth-20261031-205404`
- `UTBMap.kext.nullmoth-20261031-205423`
- `UTBMap.kext.nullmoth-20261031-205529`

These are backups / working copies, not three separate mappings to enable. Keep only the intended active mapping in a public EFI and do not place duplicate `.kext` bundles with timestamp suffixes in the active `Kernel -> Add` list.

## Symptoms and timeline

### First USB failure after EFI reached desktop

After the second EFI revision (which could reach the desktop after changes to PCI enumeration / AM5 patching), the laptop keyboard and touchpad worked, but USB devices were severely affected.

The initial symptom was not simply missing enumeration:

- device insertion took a long time to respond;
- the whole system felt heavily delayed / stuttered around USB activity;
- USB flash drives, keyboard and mouse all showed latency;
- USB enumeration / initial checks were mostly normal.

A custom USB map alone did not solve the severe lag.

### XHCILagFix

Adding `XHCILagFix` resolved the ordinary USB peripheral problem according to the owner: USB storage, keyboard and mouse became usable / normal. This is the reported practical fix for the broad USB delay issue.

The earlier USB-audio experiment used `XHCILagFix` 0.1.1. The supplied current config comments `XHCILagFix` as version 0.1.2 and sets it enabled.

### USB audio was a separate unresolved issue

USB audio initially had a different and more persistent failure pattern:

- USB headset produced no sound, or audio/video playback was badly choppy;
- after changing `XHCILagFix` to 0.1.1 and adding `ItlXhciKick`, audio could return, but there was periodic stutter / popping and the result was not usable;
- enabling or disabling `ItlXhciKick` did not reliably eliminate the issue;
- the owner also observed a video-acceleration / playback side effect during that phase, but its exact relationship was not established.

After upgrading to macOS Tahoe 26.7.2 (the owner also referred to a 15.8.2 system update in the same context; exact build/update relationship should be verified), the USB audio issue reportedly resolved by itself. The owner says the relevant drivers remain enabled and the USB setup now works. The latest config confirms `USBToolBox.kext` (1.2.0), `UTBMap.kext` (1.1), `XHCILagFix.kext` (0.1.2) and `ItlXhciKick.kext` are all enabled. The `ItlXhciKick` version comment is only `V`, so its actual version / build is not established.

## Current known state

The latest owner report confirms USB keyboard, mouse, flash drive and USB audio headset all work. The supplied config confirms the four entries above are enabled. The actual `UTBMap.kext` port map and its connector types were not included, so the USB mapping cannot yet be reproduced from this document.

Do not describe the ItlXhciKick toggle as the proven cause of the final USB audio repair. The owner explicitly reported that toggling it did not resolve the issue; the improvement appeared after the macOS update.

## Data still needed for a fully reproducible USB guide

- Exact AMD USB controller PCI IDs and controller topology.
- Actual `UTBMap.kext` content and port list (connector types, HS/SS pairing, internal Bluetooth port).
- USBToolBox and UTBMap versions / build configuration.
- XHCILagFix 0.1.1 source or release identifier, exact kext version and plist settings.
- ItlXhciKick release / commit (the current config comment is only `V`).
- Before/after logs demonstrating latency and final stability.
- Separate tests for USB 2.0, USB 3.x storage, keyboard/mouse, USB audio, sleep and hot-plug.

Until those are added, this chapter is a troubleshooting history, not a copy-and-paste USB port-mapping guide.
