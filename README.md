# Fujitsu-Esprimo-P556/2
This repository contains the necessary files and information to successfully boot macOS on this prebuilt PC.

- Bootloader version: **OpenCore 0.8.8**
- SMBIOS: [iMac18,2](https://everymac.com/systems/apple/imac/specs/imac-core-i5-3.0-21-inch-aluminum-retina-4k-mid-2017-specs.html)
- Kexts version: everything up-to-date with the latest version (check the links below)
- macOS version: [Monterey](https://www.apple.com/macos/monterey) - Release channel


## Specs

| Component      | Brand                                     |
|----------------|-------------------------------------------|
| **Mobo**       |  FUJITSU D3400-U1                         |
| **CPU**        |  Intel(R) Core(TM) i5-7400 CPU @ 3.00GHz  |
| **Chipset**    |  Intel H110                               |
| **GPU**        |  ASUS AMD RX560 4GB                       |
| **Storage**    |  WD SN750 512GB                           |
| **Audio**      |  Realtek ALC671 - layout 15               |
| **Ethernet**   |  Realtek RTL8111G                         |
| **SMC Chip**   |  Nuvoton NCT6792                          |
| **OS**         |  macOS Monterey 12.6.2 (21G320)           |
| **BIOS**       |  V5.0.0.12 R1.26.0 for D3400-U1x          |


## Important notes

- In the config.plist, section `PlatformInfo > Generic`, the following fields are currently edited with "CHANGEME" in order to force the user to generate his own serials. Refer to this guide to [know how](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/icelake.html#platforminfo). 
  - `MLB`
  - `ROM`
  - `SystemSerialNumber` 
  - `SystemUUID`


- OpenCanopy is fully configured with the correct theme from acidanthera, ([GoldenGate](https://dortania.github.io/OpenCanopy-Gallery/ocbinary.html#set-1-goldengate)) but if you want to disable this you should edit the `config.plist` and change `PickerMode` from `External` to `Builtin` or disable `ShowPicker` entirely.


## How to get this PC to boot macOS flawlessly

I highly suggest to read the [OpenCore guide](https://dortania.github.io/OpenCore-Install-Guide/)

### Drivers

Must have to boot any macOS version from USB:

* OpenRuntime.efi (bundled in OpenCore package)
* HfsPlus.efi (if you created the USB with [this method](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/mac-install-recovery.html#legacy-macos-online-method) or with `createinstallmedia`) and can be found either in the `OC/Drivers` folder of this repository or in [acidanthera/OcBinaryData](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi)

Additional drivers for cosmetic stuff:

* AudioDxe.efi for Boot Chime support in UEFI environment (already enabled)
* OpenCanopy.efi (bundled in OpenCore package) for Mac-like GUI support in picker

### Kexts

* [AppleALC](https://github.com/acidanthera/AppleALC/releases/latest)
* [CPUFriend](https://github.com/acidanthera/CPUFriend/releases/latest) - SSDT-PLUG is already configured with the frequency data
* [HoRNDIS](https://github.com/jwise/HoRNDIS/releases/latest) - for those of you who like to have USB tethering support
* [Lilu](https://github.com/acidanthera/Lilu/releases/latest)
* [NVMeFix](https://github.com/acidanthera/NVMeFix/releases/latest) - for the SN750, correct power management
* [SMCProcessor](https://github.com/acidanthera/VirtualSMC/releases/latest) - shipped inside **VirtualSMC**
* [SMCSuperIO](https://github.com/acidanthera/VirtualSMC/releases/latest) - shipped inside **VirtualSMC** compatible with Nuvoton NCT6792 
* [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases/latest) 
* [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases/latest)

 ### BIOS offsets (searchable on Section_PE32_image_Setup.txt)
 
 **Note**: The BIOSes present in the directory `Misc/Extracted\ sp132835/` are multiple bin files, and the one made for this laptop is precisely [this one](https://github.com/1alessandro1/HP-laptop-15s-fq1034nl-ice-lake/blob/main/Misc/BIOS/BIOS_F.23_HP_086C9/Extracted%20sp135993/086C8.bin)
 
 Please note that even though I've listed here the offsets with `setup_var` (as `modgrubshell.efi` would require) I had to use `RU.efi` to edit these. A nice guide on how to use `RU.efi` can be found [here](https://www.macos86.it/topic/4523-guida-come-modificare-le-impostazioni-nascoste-del-bios-su-pc-con-firmware-uefi/)
 
 
- **CFG Lock** = `setup_var 0x4ED 0x0` (Disabled) (Section `CpuSetup`)
 
- **DVMT Pre-Allocated** = `setup_var 0x796 0x2` (64MB) (or `0x4` for 128MB) (Section `SaSetup`)
 
- **DVMT Total Gfx Mem** = `setup_var 0x797 0x3` (MAX) (Section `SaSetup`)
 
- **SATA Mode** = `setup_var 0x8D2 0x0` (AHCI) - this should be on zero by default (Section `Setup`)


This way, if you applied these settings correctly: 
- You won't need `framebuffer-fbmem` and `framebuffer-stolenmem` properties under `DeviceProperties` for the graphics patch (not present in the source code, I use a dGPU)
- You won't need `AppleXCPMCfgLock` or similar kernel quirks because CFG Lock is unlocked.
  
### Sleep

Remember that you are recommended to apply these settings once you booted macOS:

```
sudo pmset autopoweroff 0
sudo pmset powernap 0
sudo pmset standby 0
sudo pmset proximitywake 0
sudo pmset tcpkeepalive 0
```

## `MAT Support` is `0`

Hence, the only `Booter > Quirks` required to boot are `AvoidRuntimeDefrag`, `EnableWriteUnprotector`, and `SetupVirtualMap`.

MMIO Devirtualization it is not required.

## USB Mapping

USB Ports are mapped thought the SSDT Method (cleanest way). The SSDT in question is `SSDT-xh_rvp08.aml` which was taken from ACPI dump (SysReport.zip) and edited to match the USB ports in the system.

## Credits

* [Apple](https://apple.com) for macOS
* [Acidanthera](https://github.com/Acidanthera) for OpenCore and Lilu-based kexts 
* [dreamwhite](https://github.com/dreamwhite) for helping me to fix the I2C trackpad and with SSDT/ACPI hotpatching
* [Gengik84](https://www.macos86.it/profile/1-gengik84/) for the `GENG` method used in `SSDT-USB.aml`
* [dortania](https://github.com/dortania) team for its detailed guides
* [Corpnewt](https://github.com/CorpNewt) for SSDTTime and [fewtarius](https://github.com/fewtarius) for CPUFriend fork (now merged into Corp's repo)
