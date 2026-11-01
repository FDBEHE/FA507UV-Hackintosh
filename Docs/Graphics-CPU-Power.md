# Graphics, CPU, battery and power management observations

## Graphics hardware from 1401 Probe 2.2.0

| GPU | PCI ID | Probe notes |
|---|---|---|
| NVIDIA GeForce RTX 4060 Max-Q / Mobile (AD107M) | `10DE:28E0` | Probe marked it as boot display; VBIOS 147 KB; UEFI GOP present |
| AMD HawkPoint1 (Radeon 780M integrated graphics) | `1002:1900` | VBIOS not readable; UEFI GOP reported absent |

The owner reports using a `1401` graphics driver, version `2.6.5`, with:

- dual-monitor output working;
- one display at 2560×1440 / 165 Hz;
- another display at 3840×2160 / 160 Hz;
- no observed desktop stutter in that setup;
- video playback / decoding working normally.

This is an owner-reported functional result. The exact connector topology, which GPU drives each display, framebuffer / device properties, driver package source and version hashes should be included before this is treated as a reproducible graphics guide. Do not infer generic NVIDIA Ada support from this one machine-specific report.

### macOS 15 vs Tahoe observation

On the earlier macOS 15 installation, the owner observed a text-rendering issue where text in an unselected window could become black / appear as a dark blob. The owner says this symptom did not occur on macOS Tahoe 26.7.2.

## CPU and AMD power management

The EFI kext directory contains:

- `AMDRyzenCPUPowerManagement.kext`
- `SMCAMDProcessor.kext`

The current config enables `AMDRyzenCPUPowerManagement.kext` and `SMCAMDProcessor.kext`, both commented version 3.34.0. The latest owner report says CPU boost works, with desktop idle / light-load power around 6–10 W (sometimes about 6 W) and an all-core stress test around 48 W. Earlier informal observations also recorded up to about 5.0 GHz peak boost, around 4.8 GHz in a 3-minute all-core test, and around 4.9 GHz in another all-core observation. These are owner readings, not standardized benchmarks.

Earlier approximate observations included:

| Measurement | Owner-reported observation |
|---|---:|
| Peak boost | up to about 5.0 GHz |
| All-core frequency under load | about 4.9 GHz at one point |
| 3-minute all-core test | about 4.8 GHz |
| All-core power | about 48 W |
| Single-core power | about 40 W (owner explicitly unsure) |

These are informal readings rather than standardized benchmark results. Temperature, benchmark tool, power sensor definition, ambient conditions and exact test version were not retained. Do not present them as guaranteed specifications.

## Battery / SMC

The EFI kext folder includes `SMCBatteryManager.kext`, `SMCAMDProcessor.kext`, `SMCLightSensor.kext` and `VirtualSMC.kext`.

The owner recalls an earlier battery reporting issue:

- the computer could discharge normally;
- current changes / charge-current direction could be detected;
- the charging state was not displayed correctly at first;
- a custom patch was created to address a configuration step, but the exact patch name and values are not remembered.

The final state should be rechecked before documenting battery status as completely fixed. Record whether macOS shows charging, battery percentage, cycle count, health, power adapter wattage and whether values update correctly on AC insertion/removal.

## Sleep, lid close, fan and shutdown

- Sleep was not systematically tested.
- An earlier lid-close test did not enter sleep correctly.
- Fan noise was mentioned in connection with the earlier sleep / lid-close behavior; exact fan control state is unknown.
- Shutdown previously could hang at a verbose code, but later seemed to work; cause unknown.
- The latest owner report states that closing the lid does not put the laptop to sleep and the fan continues running. Other sleep scenarios remain untested. The owner does not use sleep and chose not to continue sleep/wake debugging. The keyboard power button is also not tested.

These features must be labeled **untested / unresolved**, not fixed.
