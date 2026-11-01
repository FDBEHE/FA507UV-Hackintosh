# Wi-Fi and Bluetooth

## Hardware IDs from 1401 Probe 2.2.0

| Device | Chip / ID | Bus |
|---|---|---|
| Wi-Fi | Realtek RTL8852BE PCIe 802.11ax, `10EC:B852` | PCIe |
| Bluetooth | Realtek Bluetooth Radio, `13D3:3571` | USB |
| Ethernet | Realtek RTL8111/8168 family, `10EC:8168` | PCIe |

## Wi-Fi

The EFI `OC/Kexts` directory listing supplied by the owner contains:

- `AirPort_RTW89.kext`
- `IO80211FamilyLegacy.kext`
- `IOSkywalkFamily.kext`
- `Lilu.kext`

The owner says Wi-Fi was essentially working after adding the RTW89-related driver. The latest `config.plist` confirms the following entries are enabled: `AirPort_RTW89.kext` (commented version 0.5.0), its `AirPortRTW89Front` plug-in, `IO80211FamilyLegacy.kext` (1200.12.2b1), its `AirPortBrcmNIC` plug-in (1400.1.1), and `IOSkywalkFamily.kext` (1.0). The owner now confirms ordinary Wi-Fi use and Apple ID sign-in work. The exact original repair sequence and feature matrix remain unknown.

Do not assume every macOS wireless feature works. Before publishing a reproducible guide, record separately whether the following were tested: basic 2.4/5 GHz association, throughput, reconnect after sleep, hotspot, AirDrop, Handoff, Continuity and Location Services.

## Bluetooth

The supplied EFI kext listing contains:

- `RealtekBluetoothFirmware.kext`
- `BlueToolFixup.kext`
- `BTSmpFixup.kext`
- `Lilu.kext`

The Probe report identifies a Realtek Bluetooth USB radio as `13D3:3571`. The latest config confirms all three are enabled: `RealtekBluetoothFirmware.kext` (1.0.2), `BlueToolFixup.kext` (2.7.3) and `BTSmpFixup.kext` (2.7.3). The owner has tested a Bluetooth keyboard successfully, but an iPhone 12 cannot connect. Therefore Bluetooth is only partially working; do not describe it as fully functional.

Before publishing a precise installation recipe, still confirm the actual binary `CFBundleVersion` values, the USB port mapped for Bluetooth and its connector type, cold-boot / restart behavior, Bluetooth audio, and coexistence with Wi-Fi. The current config proves the entries are enabled, but it does not include the USB map itself.

## What is known vs unknown

**Known:** Wi-Fi works; Ethernet works; Apple ID sign-in succeeds. Bluetooth keyboard connects and works; iPhone 12 does not connect. Current enabled kext versions are listed above.

**Unknown / not verified:** AirDrop, Handoff, Continuity, hotspot, iMessage, FaceTime, Bluetooth audio, phone pairing root cause, and the USB port path / connector type used by Bluetooth. Do not fill these gaps by guessing.

## Ethernet

`RealtekRTL8111.kext` (commented version 2.4.2) is enabled in the supplied config. The owner confirms wired Ethernet works. The hardware is Realtek RTL8111/8168 family, PCI ID `10EC:8168`.
