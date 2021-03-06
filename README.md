# HP Elitebook x360 1040 G6 Opencore Hackintosh

This is not a guide, nor a complete EFI to drop and start. Your own results actually may be much better than my dir.

## specs

* macOS Big Sur 11.2.2 (20D80) with OpenCore 0.6.7
* Intel i5-8365U / Intel UHD Graphics 620
* 16 GB of soldered Samsung DDR4 memory @ 2400Hz (as reported by macOS)
* replaced PM981 SSD - EVO 860 and 970 EVO Plus work fine
* Intel AX200 soldered wifi&bt combo
* 14" 1920x1080 SureView

## what works
please read [notes](#notes)

* WIFI & Bluetooth

* Speakers - sometimes they won't show up in preferences & there is another speakers option that doesn't work (WIP)
* Headphone combo jack
* Trackpad with gestures
* Touchscreen with pen input
* USB 3 & USB-C Ports
* Fn keys to change volume or brightness
* Battery status
* sleep
* HDMI
* HP Sure View
* FileVault 2


## what doesn't

* **Webcam:** detected by system as USB device (disabled in my USBMapJHL7540), but no image
* **Windows Hello biometric auth (fingerprint & IR camera)**
* **Internal microphone** - handled by Intel Smart Sound Technology
* **Thunderbolt**(?) - don't own any thunderbolt device, so no way to test
* **Volume buttons near USB-C** - need some work to do, they are under Intel HID 5 Button Array

**BIOS Version:** R90 01.08.03 Rev.A


**BIOS Settings**
- Disable SecureBoot, Wake on Lan, Power Control, set DVMT to 64MB and you should be good to go
- unfortunately (at lest in my case) there is no way to turn off Wake on USB


I'm not going to cover how to install macOS - [dortania guide](https://dortania.github.io/OpenCore-Install-Guide/)


## ssdt
* **SSDT-AWAC** - generated by ssdttime
* **SSDT-BAT** - battery status, manually made
* **SSDT-EC** - generated by ssdttime
* **SSDT-HPET** - generated by ssdttime
* **SSDT-PLUG** - allows for better power management
* **SSDT-PLNF** - needed
* **SSDT-TB3** - thunderbolt fixes, more info below
* **SSDT-USBX** - needed for skylake and newer
* **SSDT-XOSI** - for trackpad and touchscreen to work, with patch in config.plsit

## kexts:
* Lilu
* WhateverGreen
* AirportItlwm / itlwm
* AppleALC
* CPUFriend
* IntelBluetoothFirmware
* NoTouchID
* NVMEFix
* RTCMemoryFixup (not sure if needed, but with config.plist patches works so)
* VirtualSMC (with plugins)
* VoodooI2C (with HID)
* VoodooPS2Controller
* IOElectrify might be needed for Thunderbolt
* USBMap

**RECOMMENDED (post install):** https://github.com/xzhih/one-key-hidpi.

## notes
* wifi card is not replaceable, using airportitlwm
* couldn't get PM981 to work so I needed to replace it, 970 EVO Plus with NVMEfix works fine tho
* USB-C port tested with HP USB-C Mini dock, no thunderbolt devices to test thunderbolt functionality
* won't sleep with attached external storage when connected via USB-A port due to locked Wake on USB option in bios, in case you want to put your laptop to sleep with external storage use USB-C ports
* sometimes it boots in SureView mode by default
* if it boots to a black screen turn SureView and use it for a little bit
* brightness keys on keyboard change SureView brightness, system preferences allow to change normal brightness
* CPUPM - needs modded voltageshift with wrmem see [improving perfomance](https://github.com/SeptemberHX/HP-Spectre-X360-13-late-2018-Hackintosh#for-20w-tdp)
  * [normal geekbench score](https://browser.geekbench.com/v5/cpu/7251151) vs [after voltageshift](https://browser.geekbench.com/v5/cpu/12764741)
* sometimes the CPU Power gets limited to 10W when waking up from sleep mode - to fix this you need to connect a charger, put laptop to sleep and then wake

two USBMap kexts are present in this directory:
- TXHC was mapped before SSDT-TB3 - USB-C ports work after reboot, after sleep only at 2.0 bus
- JHL7540 was mapped after SSDT, with disabled camera

## Thunderbolt 3
recently i discovered that USB-C ports didn't work at 3.0 speeds after sleep. this is (my guess) due to USB-C controller being under the Thunderbolt controller. this is where SSDT-TB3 comes to play - with it looks like the Thunderbolt controller is finally being powered on after sleep which results in USB ports working correctly
I DID NOT test Thunderbolt devices as I don't have any at hand.
That's also why my config has both IOElectrify & TbtForcePower.efi
