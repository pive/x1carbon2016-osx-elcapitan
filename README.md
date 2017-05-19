# Thinkpad x1 Carbon 4th gen 2016 (skylake) running Mac OS X 10.11.6

## Description
This is a quick manual for those interested in trying to run OS X natively on the Lenovo Thinkpad Carbon x1. No spoonfeeding, read and experiment yourself reading @rehabman documentation. This guide is aimed at giving my feedback and help others figuring out they actually *CAN* do it, if they can read and understand the guides. If you don't, please do not open issues. If you do, please try and give me some feedback.

#### Why El Capitan?
**TL;DR** Karabiner not compatible with Sierra.

At the time of writing Sierra is out for a little while but as an old PC developer I heavily depend on Karabiner to map my PC keys - keyboard has always been the reason I could not work with a real mac. Anyways, there should be no issue to go with Sierra.

**This is not a perfect setup but it suits my needs as a machine for work.**

## Laptop specs
The laptop is a 4th gen (20FB) model with:
- Samsung NVMe 256GB SSD
- Core i7-6500U (Intel HD Graphics 520)
- 8GB RAM
- 14" WQHD (2560 x 1440) IPS display

## Working
- Booting on NVMe disk
- Ethernet
- ACPI and battery
- Graphics and HDMI output
- Audio
- USB3

## Not working
- Wifi (Intel) - use a wifi dongle (e.g. DWA-131) or **stop reading this**.
- Webcam

## Not used
- HiDPI - fonts are REALLY small on the display, but I actually like it. See https://www.tonymacx86.com/threads/adding-using-hidpi-custom-resolutions.133254/ to give it a try.
- Bluetooth
- SD Card
- Fingerprint sensor
- Touchpad (I always use an external mouse, and optionally the pointing stick)
- Backlight level

## Tools used
- Clover v2.4k r4061 with Clover Configurator
- MaciASL
- Piker-Alpha ssdtPRGen
- DCPI Manager
- Kext Utility
- Rehabman's patch-nvme

## Mandatory Reads
- https://www.tonymacx86.com/threads/guide-booting-the-os-x-installer-on-laptops-with-clover.148093/
- https://www.tonymacx86.com/threads/faq-read-first-laptop-frequent-questions.164990/
- https://www.tonymacx86.com/threads/guide-hackrnvmefamily-co-existence-with-ionvmefamily-using-class-code-spoof.210316/
- https://www.tonymacx86.com/threads/guide-patching-laptop-dsdt-ssdts.152573/

## Initial system
- A working OS X 10.11.6 installation on a Lenovo x230 thanks to @Bizzaro (https://github.com/Bizzaro/x230-osx)
- A 16GB USB stick
- Windows 10 running on the target x1 Carbon (will be erased, use Clonezilla to backup if needed)

## Step 1 - USB stick setup
- Follow Rehabman's guide for installing Clover on USB. 
- config.plist is Rehabman's config_HD520_530_540.plist
- Follow Rehabman's guide for patching NVMe using class-code spoof. I had no issues at all.
- Kexts used in Clover USB EFI:
    * FakeSMC
    * HackrNVMeFamily (do *NOT* forget the --spoof)
    * VoodooPS2Controller
- SSDT patches in USB EFI (NVMe patch)
- drivers64UEFI:
    * HFSPlus.efi
    * EmuVariableUefi-64.efi (may be not mandatory?)
    * OsxAptioFixDrv-64.efi
- Copy useful files and software to the main USB install partition for later use

## Step 2 - USB Installation
Boot from USB, graphics are working (no DVMT-prealloc issue)! Then follow Rehabman's guide:

- Erase disk
- Install step 1 will last 15 minutes
- Reboot (USB EFI) and start from USB stick again
- Install is resumed (I actually had to choose disk and validate license again ...)
- Install step 2 will last 20 minutes
- Reboot (USB EFI) and start from HD, finish installation
- OS X should be installed, apply OS X updates via AppStore (2017-002 at the time of writing)
- Reboot (USB EFI) and start from HD
- You might want to install any useful software tools (e.g. Kext utility, Karabiner)

Now we have a valid, up-to-date El Capitan on HD, ethernet not working yet.

## Step 3 - Enabling HD boot
- Install Clover on HD using same parameters as USB stick, use same config.plist as a base
- Patch NVMe with Rehabman's script again (driver was updated)
- The only mandatory kexts are HackNVMe + fakeSMC
- Do not forget HFSplus.efi and NVMe SSDT
- Install VoodooPS2Controller in HD S/L/E (follow the guide!)
- Install IntelMausiEthernet in HD S/L/E (https://www.tonymacx86.com/resources/intelmausiethernet.326/)
- Install HackNVMeFamily in HD S/L/E (yes, you need *both* EFI and S/L/E)
- Using Clover Configurator you can set SMBIOS to iMac 17,1 from wizard
- Reboot on HD EFI (remove USB stick)

Now we have a valid, up-to-date, bootable El Capitan on HD with ethernet.

## Step 4 - ACPI (P-states and Battery)
- Generate SSDT using Piker-Alpha's ssdtPRGen
- Copy SSDT.aml into HD EFI (along with NVMe, this is causing no issues)
- Generate base DSDT using F4 when on Clover boot menu screen (see Rehabman SSDT/DSDT guide)
- Open DSDT with MaciASL and compile it. Errors may occur, fix them applying patches or manually (I had to).
- Apply base patches:
    * sys HDEF
    * sys IRQ
    * bat x220
- Compile and save DSDT aml to HD EFI
- Install Rehabman's ACPIBatteryManager v7
- Reboot

Now we have a working power and battery management.

## Step 5 - Audio
This is the most delicate part. I use AppleHDA patcher from Mirone but had issues patching for this laptop's audio chip Conexant CX20753/4 (codec 14F15111). After numerous unsuccessful attemps I managed to get an old fully patched AppleHDA kext from this thread: https://www.tonymacx86.com/threads/applehda-patch-problem-for-conexant-cx20753-4-14f1-5111.191289/ (From Mirone, used the v7 version with external mic not working).

- Patch DSDT with audio 3 layout
- Install AppleHDA v7 in HD S/L/E (will have to reinstall after each update but sound is not a critical component)
- Install Rehabman's CodecCommander in HD S/L/E (follow the guide!)
- Reboot

Now we have audio (only speaker tested so far but it's a good start).

## Various hints
- Do not forget to disable sleep (see Rehabman's guides).
- Use Clonezilla when important steps are achieved (typically end of step 3) - it's not very time-consuming and can save your life.
- Read the guides.
- Read the guides.
- Did you actually read the guides?

## Open issues
- Sometimes graphic glitches can happen (rare and not critical, solved after a reboot)
- Webcam not detected (might be an internal USB issue, not deeply investigated so far)
- Bluetooth not used (I don't care)
- AppleHDA with CX20753/4 issue
- Touchpad is working though disabled in BIOS (but touchpad click is actually disabled)
- Backlight level and keyboard leds not functional (not investigated so far)
