# Realtek ALC256 internal speaker fix

## Hardware

- Codec: Realtek ALC256, `10EC:0256`.
- Subsystem ID: `1043:1C23`.
- AMD HD Audio controller: `1022:15E3`.
- macOS: Tahoe 26.7.2.
- Kext stack observed loaded: Lilu 1.7.3, SimpleHDA 1.0.0, AppleALC 1.9.9.
- Current owner report: built-in speakers, 3.5 mm headphone jack and USB audio headset all work.
- Earlier tests also observed USB Apple EarPods working.
- Microphone is not yet verified: the macOS Sound settings input-level meter moves while speaking, but no actual recording / voice-call test has been completed.

## Important diagnostic findings

The Linux codec dump established the relevant hardware nodes:

| Node | Function | Relevant connection / state |
|---|---|---|
| `0x02` | Audio Output DAC | Speaker DAC; playback amplifier capability `0x00025757` |
| `0x03` | Audio Output DAC | Headphone DAC; playback amplifier capability `0x00025757` |
| `0x14` | Pin Complex | Speaker pin; connected to DAC `0x02`; EAPD capable |
| `0x21` | Pin Complex | Headphone pin; output / headphone detect; connection includes DAC `0x03` |
| `0x20` | Vendor-specific processing widget | Processing coefficient index / data interface used for the final workaround |

On macOS, HDEF and AppleHDAController were registered and active; ALC256 was enumerated and matched to AppleALC. Layout 97 exposed audio engines, but built-in speakers remained silent. Layout 15 was an experiment based on adjusted PinConfig values; with `alcid=15`, built-in audio endpoints disappeared. Returning to Layout 97 restored the internal output / headphone endpoints.

An earlier attempt to set the amplifier on speaker Pin `0x14` was based on an incorrect interpretation. `GET_PARAMETER 0x12` at `0x14` returned `0x80000000`, meaning that the pin does not expose the independent output amplifier that was being adjusted. DAC nodes `0x02` and `0x03` returned `0x00025757`.

## Coefficients and GPIO: final working runtime experiment

The following values were read from the machine before modification:

| Processing coefficient | Original value | Final working value |
|---|---:|---:|
| index `0x10` | `0x0220` | `0x7F20` |
| index `0x16` | `0x0E50` | `0x0C50` |
| index `0x35` | `0x8D6A` | unchanged |
| index `0x37` | `0xFE06` | unchanged |
| index `0x57` | `0x7F7F` | unchanged |

GPIO state before the final experiment:

- GPIO count: `0x40000003` (3 GPIOs reported).
- GPIO0 was enabled / output with data low.
- GPIO1 and GPIO2 were disabled.

The combination that made the built-in speakers produce sound at runtime was:

```text
Codec NID 0x20, coefficient index 0x10 -> 0x7F20
Codec NID 0x20, coefficient index 0x16 -> 0x0C50
GPIO mask      -> 0x05
GPIO direction -> 0x05
GPIO data      -> 0x04
```

This preserved GPIO0's existing enabled/output state and enabled GPIO2 as an output-high bit. The owner reports that the speakers began working immediately after this combination. Because the subsystem ID differs from the reference ASUS model from which the values were considered, this should be described as an empirically verified workaround for this FA507UV (`1043:1C23`), not a universal ALC256 setting.

## Persistent AppleALC integration

The working layout was Layout 97. The following verb byte sequence was appended to the ALC256 / Layout 97 entry in `Resources/PinConfigs.kext/Contents/Info.plist`:

```text
02050010 02047F20
02050016 02040C50
00171605 00171705 00171504
```

The same seven commands were appended to both:

- `ConfigData` — codec configuration on initialization;
- `WakeConfigData` — codec configuration during wake reinitialization.

`WakeVerbReinit` was `true` in the Layout 97 entry. The owner does not use sleep and has not verified wake audio after the final change.

The effective added HDA verb commands are:

| Hex bytes | Meaning |
|---|---|
| `02 05 00 10` | Set processing coefficient index `0x10` on NID `0x20` |
| `02 04 7F 20` | Set processing coefficient data to `0x7F20` |
| `02 05 00 16` | Set processing coefficient index `0x16` |
| `02 04 0C 50` | Set processing coefficient data to `0x0C50` |
| `00 17 16 05` | Set GPIO mask to `0x05` |
| `00 17 17 05` | Set GPIO direction to `0x05` |
| `00 17 15 04` | Set GPIO data to `0x04` |

## Build and verification history

- AppleALC source: local AppleALC 1.9.9 checkout.
- Xcode: 26.6, build `17F113`, on the x86_64 macOS install.
- Lilu SDK: prebuilt SDK 1.7.2 used for build support; the installed runtime Lilu was 1.7.3.
- Release build command:

```bash
xcodebuild \
  -project AppleALC.xcodeproj \
  -scheme AppleALC \
  -configuration Release \
  -derivedDataPath build/DerivedData \
  CODE_SIGNING_ALLOWED=NO \
  build
```

- Build result: `BUILD SUCCEEDED`.
- `lipo -info` showed the resulting binary was `x86_64`.
- Resource verification confirmed custom Layout 15 / Platform 15 resources were compiled into an earlier experimental build; Layout 15 was not used for the final fix.
- Final Release build's merged `AppleALC.kext/Contents/Info.plist` contained exactly one ALC256 (`CodecID 283902550`) Layout 97 entry with the expected ConfigData and WakeConfigData suffix.
- Replacing AppleALC in `EFI/OC/Kexts` and retaining `alcid=97` resulted in built-in speakers working after reboot without manually running `alc-verb`.

## Final audio boot arguments observed

```text
alcid=97 alcverbs=1 alctcsel=1
```

The exact necessity of `alcverbs=1` / `alctcsel=1` after the final permanent integration has not been separately tested. Do not remove them without a controlled test.

## Current end-to-end audio status

| Path | Current status | Evidence boundary |
|---|---|---|
| Built-in speakers | Working | Reboot-tested after custom AppleALC Layout 97 integration, and reconfirmed by the latest owner report |
| 3.5 mm headphone jack | Working | Latest owner report |
| USB audio headset | Working | Latest owner report |
| Internal microphone | Not verified | Input meter moves, but no recording / call test yet |
| Sleep / wake audio | Not tested | Lid-close sleep currently fails, so wake audio is not assessed |

## Notes / limitations

- 3.5 mm output generally worked; a few intermittent failures occurred during macOS 26 troubleshooting. macOS 15 behavior was not systematically tested.
- The final built-in speaker fix is verified on this machine.
- Sleep / wake audio was not tested after the final fix.
- Layout 15 was an unsuccessful experiment for this machine and should not be presented as the final layout.
