# Dell XPS 15 9500 (4K Touch) — OpenCore EFI for macOS Sonoma, Sequoia & Tahoe

A personal OpenCore configuration for the **Dell XPS 15 9500 (2020), 4K UHD touchscreen**. The main target is **macOS Tahoe 26.x**. Booting has also been tested with macOS Sonoma and Sequoia, but each macOS release requires compatible Wi-Fi components and, on versions before Tahoe, a different audio setup.

> **Important:** This is a hardware-specific Hackintosh project, not a universal EFI. Do not copy this EFI to another model and expect it to work. Back up your working EFI before making changes. Generate your own SMBIOS values before using or publishing a config.

## Hardware

| Component | Configuration |
| --- | --- |
| Laptop | Dell XPS 15 9500 (2020) |
| CPU | Intel Core i7-10750H (Comet Lake-H) |
| Integrated graphics | Intel UHD Graphics 630 |
| Discrete graphics | NVIDIA GeForce GTX 1650 Ti — disabled for macOS |
| Internal display | 3840 × 2400 4K UHD touchscreen |
| Audio | Realtek ALC3281 / ALC289 family |
| Wi-Fi | Intel Wi-Fi using `AirportItlwm` (use a build matching the target macOS) |
| Bootloader | OpenCore |

Your hardware revision, BIOS version, Wi-Fi card, storage device and other components may differ. Verify them before applying any hardware-specific settings.

## Tested status

Status below reflects this specific laptop and the author's testing. It is not a guarantee for every XPS 9500 configuration.

| Feature | macOS Tahoe 26.x | Notes |
| --- | --- | --- |
| Intel UHD 630 graphics acceleration | Working | Confirmed on the author's 4K model; the NVIDIA dGPU is disabled. |
| Internal 4K touchscreen | Working | Started working after USB ports were mapped. |
| USB 2.0 / USB 3.x | Working | Uses a final `USBMap.kext` generated with CorpNewt/USBMap. |
| Intel Wi-Fi | Working | Requires the `AirportItlwm` build made for the running macOS version. |
| Built-in speakers / headphone jack | Working | Tahoe uses `SimpleHDA.kext` with AppleALC. |
| Internal microphone | Working | `layout-id = 13` was the preferred working audio layout in this setup. |
| Trackpad | Basic operation works; tuning may be needed | Tap-to-click, secondary-click and click feel depend on macOS settings and the I2C configuration. |
| CPU power management / Turbo Boost | Not yet fully benchmarked | Verify idle and loaded frequency with `powermetrics`; do not infer status from one frequency reading. |
| Bluetooth, webcam, SD reader, fingerprint reader, sleep/wake | Not fully verified | Treat as untested unless confirmed on your own machine. |

## macOS version notes

### Tahoe 26.x — primary configuration

- Use a Tahoe-compatible `AirportItlwm.kext` build. Tahoe uses a separate build target from Sequoia.
- Analog audio is restored with [`SimpleHDA`](https://github.com/laobamac/SimpleHDA), along with Lilu and AppleALC.
- Keep the working audio layout and ACPI configuration. On this system, `layout-id = 13` produced the preferred sound and the internal microphone worked.
- The SimpleHDA entry should load after Lilu and AppleALC in `Kernel -> Add`.
- Keep `keepsyms=1` in `boot-args` when using SimpleHDA, as required by its project documentation.

### Sequoia 15.x

- Replace the Tahoe Wi-Fi kext with an `AirportItlwm` build compatible with the installed Sequoia release. Do not assume a Tahoe build also works on Sequoia.
- Remove `SimpleHDA.kext` from `EFI/OC/Kexts` **and** its entry in `Kernel -> Add`. Sequoia includes AppleHDA; use the working AppleALC/layout configuration for this machine instead.
- If the optional SimpleHDA HAL package was installed in Tahoe, uninstall it using the `Tools/uninstall_hal.zsh` script from the SimpleHDA package/repository, then reboot. See the [SimpleHDA documentation](https://github.com/laobamac/SimpleHDA/blob/main/README.en.md).

### Sonoma 14.x

- Use an `AirportItlwm` release built for the exact macOS version. AirportItlwm releases are OS-specific; check the upstream compatibility notes rather than reusing the Tahoe or Sequoia kext.
- Remove `SimpleHDA.kext` and its OpenCore entry. The Tahoe-only SimpleHDA workaround is not needed for the normal AppleHDA + AppleALC audio path on Sonoma.
- Re-test audio, Bluetooth, sleep, USB and all input devices after changing the OS version.

**Version rule:** keep a separate EFI backup for each macOS version where practical. If sharing one EFI between macOS versions, configure kernel-version ranges carefully and make sure only the intended kext is injected on each OS. A bootable EFI does not mean every device is functional on every macOS version.

Useful version references:
- [Intel Wi-Fi driver project and OS-specific AirportItlwm builds](https://github.com/laobamac/itlwm)
- [SimpleHDA — Tahoe audio restoration](https://github.com/laobamac/SimpleHDA)

## USB mapping

The final USB mapping in this setup uses [CorpNewt/USBMap](https://github.com/corpnewt/USBMap).

- Keep the generated `USBMap.kext` and its enabled `Kernel -> Add` entry.
- Remove/disable temporary `USBMapDummy.kext` after discovery is complete.
- Do not mix mapping methods. When using CorpNewt/USBMap, remove `USBToolBox.kext`, `UTBDefault.kext`, and `UTBMap.kext` if they are left over from a USBToolBox setup.
- Keep `XhciPortLimit` disabled. A proper map should respect macOS's 15-port limit per controller.
- Re-check the map after BIOS changes, changing USB-C docks/hubs, or major hardware changes.

On this laptop the touchscreen began working after the USB ports were mapped. That does not mean every touchscreen uses USB; I2C devices still depend on the appropriate ACPI and VoodooI2C configuration.

## OpenCore picker and display scaling

- A graphical OpenCore picker requires `OpenCanopy.efi` from the **same OpenCore release** as the rest of the bootloader and a matching `EFI/OC/Resources` directory from [Acidanthera/OcBinaryData](https://github.com/acidanthera/OcBinaryData).
- In `Misc -> Boot`, a common graphical setup is `PickerMode = External`, `PickerVariant = Auto`, `ShowPicker = true`, and a short positive `Timeout`.
- HiDPI/scaled display modes are a macOS display-scaling feature, not a kext requirement. Tools such as [BetterDisplay](https://betterdisplay.pro/) can expose additional modes, but not every custom mode is guaranteed to be available as HiDPI.

## BIOS and firmware notes

Use these as checks, not as a blind universal recipe; menu names and behavior can differ by BIOS revision and EFI configuration.

- **SATA Operation:** AHCI is commonly used for macOS compatibility.
- **Secure Boot:** usually disabled for this OpenCore setup.
- **VT-d / DisableIoMapper:** make sure the BIOS setting agrees with the current OpenCore configuration. Do not toggle it without understanding your config.
- **CFG Lock:** CPU power management can depend on CFG Lock status and OpenCore quirks. Check the status with an appropriate tool before changing firmware variables.
- BIOS variable offsets and `setup_var` commands are firmware-version-specific. Do not run commands copied from another guide unless you have verified they match your exact BIOS; incorrect firmware-variable writes can prevent the machine from booting.
- After a BIOS update, re-check relevant settings and re-test boot, graphics, Wi-Fi, audio and sleep.

For the Comet Lake laptop starting point, consult the [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html#starting-point).

## Known limitations and troubleshooting

### Sleep / wake

Sleep and resume have been reported as problematic on some XPS 9500 configurations. This is not conclusively tested for every BIOS/macOS combination here. If sleep causes a failed wake, troubleshoot logs and firmware behavior before applying a workaround. The command below disables macOS sleep entirely and should **not** be used unless that trade-off is intended:

```sh
sudo pmset -a disablesleep 1
```

### Brightness hotkeys

The earlier Monterey guide documented `Fn+S` and `Fn+B` for brightness down/up. Hotkeys can vary by EFI and keyboard configuration; test the bindings on your own setup rather than assuming they match every release.

### CPU frequency / Turbo Boost

`SSDT-PLUG.aml` is commonly used to expose native CPU power management on this platform. To check actual behavior, compare idle and loaded readings with:

```sh
sudo powermetrics --samplers cpu_power -i 1000 -n 10
```

A single low or high frequency does not by itself prove that Turbo Boost is broken. AC power, temperature, power limits, workload and BIOS configuration all affect the observed frequency.

### Audio on Tahoe

macOS Tahoe removes the AppleHDA component that AppleALC normally relies on for analog audio. See the [Dortania Tahoe notes](https://dortania.github.io/OpenCore-Install-Guide/extras/tahoe.html) and [SimpleHDA's README](https://github.com/laobamac/SimpleHDA/blob/main/README.en.md). Keep the known-working layout rather than copying a layout from another laptop.

## Before using or publishing this EFI

1. Read the [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/).
2. Back up your current EFI and keep a known-good recovery USB.
3. Run `ocvalidate` from the same OpenCore release used by the EFI.
4. Generate your own SMBIOS using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS). Update `SystemSerialNumber`, `MLB`, `SystemUUID` and `ROM` consistently.
5. **Never publish personal SMBIOS values.** Before making this repository public, redact your serial number, MLB, SystemUUID and ROM from any shared `config.plist`; do not upload logs or screenshots that expose them.
6. Never include a macOS installer, Apple system volume, personal data, or other files you are not permitted to redistribute.
7. Do not install an OpenCore/kext update and a macOS major update at the same time. Change one thing, then re-test.

This configuration is supplied as-is, without warranty. You are responsible for verifying it against your hardware and understanding every firmware or bootloader change before applying it.

## Credits and references

This repository is an independently maintained configuration inspired by earlier Dell XPS 15 9500 OpenCore work. The source below is credited for its hardware-specific observations and troubleshooting notes; this README is rewritten for the current Tahoe-based configuration rather than a verbatim copy.

- [zspherez/Dell-XPS-9500-4K-UHD-Monterey](https://github.com/zspherez/Dell-XPS-9500-4K-UHD-Monterey) — original 4K XPS 9500 Monterey reference; documents graphics, CPU power management, USB, touchscreen, hotkeys, CFG Lock and sleep caveats.
- [jkbuha/XPS-9500-CometLake-OpenCore](https://github.com/jkbuha/XPS-9500-CometLake-OpenCore) — newer XPS 9500 / Comet Lake reference including Tahoe notes.
- [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
- [Acidanthera OpenCorePkg](https://github.com/acidanthera/OpenCorePkg), [Lilu](https://github.com/acidanthera/Lilu), [WhateverGreen](https://github.com/acidanthera/WhateverGreen), and [AppleALC](https://github.com/acidanthera/AppleALC)
- [AirportItlwm / itlwm](https://github.com/laobamac/itlwm), [SimpleHDA](https://github.com/laobamac/SimpleHDA), and [CorpNewt/USBMap](https://github.com/corpnewt/USBMap)

Please read and follow the license terms of each upstream project and kext you use.
