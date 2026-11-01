# Kext inventory from the supplied EFI/OC/Kexts listing

This inventory is cross-checked against the `Kernel -> Add` entries in the supplied `config.plist` (2026-11-01). The version values below are the config comments, not independently verified binary `CFBundleVersion` values. A few nested plug-ins are listed separately in the Config audit.

| Kext folder | Likely role / project area | Notes from this machine |
|---|---|---|
| `AirPort_RTW89.kext` | Realtek RTW89 Wi-Fi | Enabled; config comment 0.5.0; Wi-Fi works |
| `AMDRyzenCPUPowerManagement.kext` | AMD CPU power management | Enabled; config comment 3.34.0 |
| `AMFIPass.kext` | AMFI-related compatibility | Enabled; config comment 1.4.1; exact requirement not isolated |
| `AppleALC.kext` | HDA audio layout / codec support | Enabled; config comment 1.9.9; custom Layout 97 includes ALC256 speaker fix |
| `AppleMCEReporterDisabler.kext` | Apple MCE reporter compatibility | Present |
| `BlueToolFixup.kext` | Bluetooth support / compatibility | Enabled; config comment 2.7.3 |
| `BrightnessKeys.kext` | Brightness key handling | Present |
| `BTSmpFixup.kext` | Bluetooth SMP compatibility | Enabled; config comment 2.7.3 |
| `IO80211FamilyLegacy.kext` | Legacy Wi-Fi framework | Present with RTW89-related stack |
| `IOSkywalkFamily.kext` | Skywalk networking framework | Present with RTW89-related stack |
| `ItlXhciKick.kext` | XHCI-related workaround | Enabled; version comment is only `V`, so exact version is unknown |
| `Lilu.kext` | Lilu plugin framework | Runtime version 1.7.3 was observed loaded during audio debugging |
| `NVMeFix.kext` | NVMe compatibility / power behavior | Present |
| `RealtekBluetoothFirmware.kext` | Realtek Bluetooth firmware | Enabled; config comment 1.0.2; keyboard works, iPhone 12 cannot connect |
| `RealtekRTL8111.kext` | Realtek Ethernet | Enabled; config comment 2.4.2; wired Ethernet works |
| `RestrictEvents.kext` | Event / platform compatibility | Present |
| `SimpleHDA.kext` | Restores / supports the AppleHDA stack on Tahoe | Enabled; config comment 1.0.0; see load-order review item in Config audit |
| `SMCAMDProcessor.kext` | AMD SMC processor monitoring | Enabled; config comment 3.34.0 |
| `SMCBatteryManager.kext` | Battery reporting | Present; exact final reporting state needs verification |
| `SMCLightSensor.kext` | Ambient light sensor support | Present |
| `USBToolBox.kext` | USB mapping support | Present |
| `UTBMap.kext` | USB port map | Present; inspect actual map contents before publishing |
| `VirtualSMC.kext` | SMC emulation | Present |
| `VoodooI2C.kext` | I2C input support | Present |
| `VoodooI2CHID.kext` | I2C HID device support | Present |
| `VoodooPS2Controller.kext` | PS/2 input support | Present |
| `WhateverGreen.kext` | Graphics property / framebuffer patch support | Present; exact role alongside the 1401 graphics driver needs confirmation |
| `XHCILagFix.kext` | USB XHCI latency workaround | Enabled; current config comment 0.1.2; 0.1.1 was tested during earlier USB audio troubleshooting |

## Cleanup before publication

The supplied listing also included three timestamped `UTBMap.kext.nullmoth-*` folders. These appear to be working copies / backups. Do not add them as active kexts. Keep one final `UTBMap.kext` and make sure `Kernel -> Add` points to that one only.

Before release, add exact versions, bundle IDs and source URLs for every kext. The folder names above are not a substitute for a pinned release manifest.

## Additional active entries in the supplied config

The current `Kernel -> Add` also enables nested `AirPortBrcmNIC.kext` (under `IO80211FamilyLegacy`, config comment 1400.1.1), `AirPortRTW89Front.kext` (under `AirPort_RTW89`, 0.5.0), `VoodooI2CServices.kext`, `VoodooGPIO.kext`, `VoodooInput.kext` (under VoodooI2C), and `VoodooPS2Keyboard.kext`, `VoodooPS2Mouse.kext`, `VoodooPS2Trackpad.kext`. The nested `VoodooPS2Controller` `VoodooInput.kext` entry is disabled, avoiding two active VoodooInput instances in this config.

`SimpleHDA` appears earlier than `AppleALC` in the supplied `Kernel -> Add` array. The SimpleHDA project README specifies the order Lilu → AppleALC → SimpleHDA. This is a load-order discrepancy to review on a cloned EFI; because the current audio setup works, do not reorder the live config without a backup and controlled test. See [SimpleHDA upstream README](https://github.com/laobamac/SimpleHDA#installation).
