# Dell XPS 15 9500 (4K Touch) — OpenCore EFI for macOS Sonoma, Sequoia & Tahoe

![alt text](https://github.com/Pewdeadcake/Dell-XPS-9500-Hackintosh/blob/main/Homescreen.png)

An OpenCore configuration for the **Dell XPS 15 9500 (2020), 4K UHD touchscreen**. The main target is **macOS Tahoe 26.x**. Booting has also been tested with macOS Sonoma and Sequoia, but each macOS release requires compatible Wi-Fi components and, on versions before Tahoe, a different audio setup.

## Hardware

| Component | Configuration |
| --- | --- |
| Laptop | Dell XPS 15 9500 (2020) |
| CPU | Intel Core i7-10750H (Comet Lake-H) |
| Integrated graphics | Intel UHD Graphics 630 |
| Discrete graphics | NVIDIA GeForce GTX 1650 Ti — disabled for macOS (Maybe I will try nullmoth's driver but what's the point lol) |
| Internal display | 3840 × 2400 4K UHD touchscreen |
| Audio | Realtek ALC3281 / ALC289 family |
| Wi-Fi | Intel Killer AX1650s|

## Tested status

Status below reflects this specific laptop and the author's testing. It is not a guarantee for every XPS 9500 configuration.

| Feature | macOS Tahoe 26.x | Notes |
| --- | --- | --- |
| Intel UHD 630 graphics acceleration | Working | The NVIDIA dGPU is disabled. |
| Internal 4K touchscreen | Working | Started working after USB ports were mapped. |
| USB 2.0 / USB 3.x | Working ||
| Intel Wi-Fi | Working | Requires the `AirportItlwm` build made for the running macOS version. |
| Built-in speakers / headphone jack | Working | Tahoe uses `SimpleHDA.kext` with AppleALC. |
| Internal microphone | Working | `layout-id = 13` was the preferred working audio layout in this setup. |
| Trackpad | Basic operation works; tuning may be needed | Tap-to-click, secondary-click and click feel depend on macOS settings and the I2C configuration. |
| Webcam | Working |  |
| Bluetooth | Working (Kind of) | Airdrop is broken? |
| iServices | Not Working | Trying to fix this atm |
| SD reader, fingerprint reader | Not Working | Apple being apple ig. |
| Sleep | Disabled | Big thanks to DELL. |

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

## BIOS and firmware notes

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
