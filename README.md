# Hackintosh: macOS Catalina on AMD - Lazy Man's Guide

## Introduction
There is no shortage of tools and guides about getting macOS Catalina working on an AMD system. But all these seem to be written by hard-core enthusiasts who live and breathe Hackintosh. So, many "obvious" steps are not covered or not given enough explanations.

For example, I have no idea what is DSDT nor how to patch it. I don't know what is _config.plist_ and how it relates to _patches.plist_. Oh, and then there are kexts and drivers. It's all very interesting and educating but the value of this knowledge is pretty low for me because by the time when I need to build my next Hackintosh system I'll either forget all the important details or my knowldege will become obsolete.

This mini guide assumes you already have Windows 10 installed and want to dual boot.

## Resources and Tools
### References and Guides
1. Amd-osx.com Vanilla Guide: https://vanilla.amd-osx.com/ (good starting point but very basic instructions)
2. Snazzy Labs video: https://www.youtube.com/watch?v=l_QPLl81GrY (the mother of all AMD Catalina Hackintosh videos) - you should watch this before doing anything
3. OpenCore AMD Guide: https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/amd-config.plist/amd-config - read only if you run into problems
4. OpenCore Vanilla Desktop Guide for AMD: https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/blob/master/AMD/AMD-config.md (good in-depth guide) - read only if you run into problems
### Software
1. gibMacOS: https://github.com/corpnewt/gibMacOS
2. SSDT Time: https://github.com/corpnewt/SSDTTime
3. GenSMBIOS: https://github.com/corpnewt/GenSMBIOS 
4. Kexts archive: [https://onedrive.live.com/?authkey=%21APj...](https://onedrive.live.com/?authkey=%21APjCyRpzoAKp4xs&id=FE4038DA929BFB23%21455036&cid=FE4038DA929BFB23)
5. Kext descriptions: https://kb.amd-osx.com/guides/MJ/kexts.html
6. ProperTree: https://github.com/corpnewt/ProperTree
### Hardware
1. Any USB 3.0 Flash Drive

## System Specs
These are the specs for my AMD system. The files in this repo (OpenCore configuration, .kexts etc) work on this hardware. If you have another MB - do not even bother. If you have the same MB but another CPU or video - you can try before spending much more time on your own config.
* Ryzen 2700X CPU, not overclocked
* ASRock B450M Pro4 MB
* G-Skill Ripjaws V 2x16Gb 3200 CL16 RAM
* Sapphire RX 570 4Gb
* A-Data 240Gb SATA3 SSD

## Random Notes

1. Clover vs OpenCore. Don't waste your time on Clover, go with OpenCore ("OC"). Even though Clover is simpler, AMD Hackintoshers lean toward OC. macOS Catalina 10.15.3 won't boot from Clover at all.
2. Catalina 10.15.0-3 are still very buggy. If you find any bugs these may be genuine macOS problems, not problems with your setup. Do some search to confirm.
3. If you plan to dualboot, be careful during the installation. You may kill your Windows boot partition.

## Instructions

If your hardware is close enough to mine (same MB, some Ryzen CPU, Polaris graphics card) you can use my OpenCore files from this repo (just put these files on to USB drive formatted as FAT32) and proceed to Step 2.

### Step 1

Watch that Snazzy Labs video. Pay attention to video description. There are useful links and some corrections. Quick summary of Snazzy Lab's instructions:

1. Run gibMacOS.bat (https://youtu.be/l_QPLl81GrY?t=449). This will download all macOS files to your folder.
2. Run MakeInstall.bat (https://youtu.be/l_QPLl81GrY?t=480). This gets the previously downloaded files along with OC to USB drive.
3. Remove unnecessary drivers (https://youtu.be/l_QPLl81GrY?t=545) from your USB drive.
4. Add ApfsDriverLoader.efi and VBoxHfs.efi to drivers folder on your USB drive. You can get theses *.efis [here](https://github.com/acidanthera/AppleSupportPkg/releases).
5. Copy necessary kexts (https://youtu.be/l_QPLl81GrY?t=610). Some folks insist AppleMCEReporterDisabler.kext should be always added to Catalina configs.
6. Deal with DSDTs and SSDTs (whatever this means) by running SSDTTime (https://youtu.be/l_QPLl81GrY?t=696).
7. Deal with _config.plist_ (https://youtu.be/l_QPLl81GrY?t=807), pay attention to *OC Snapshot* step and applying _patches.plist_. 

### Step 2

Even though Snazzy Labs suggest you to boot from the USB drive right away, you need to do few more things with your _config.plist_ before you can proceed.

1. Relax security settings, otherwise you will not be able to boot:
   * Set **RequireVault** and **RequireSignature** to False in _config.plist_ (use Ctrl+F to lookup by key).
   * Also, set *ScanPolicy* to 0.
2. Run _GenSMBIOS_, select option 3, enter _iMacPro1,1_ you'll get result like this:
   ```
      #######################################################
     #              iMacPro1,1 SMBIOS Info                 #
    #######################################################

    Type:         iMacPro1,1
    Serial:       C02V8QZFHX87
    Board Serial: C02734200J9JG36A8
    SmUUID:       0EAA2FD4-34CD-4588-97B3-4FFAD397F669

    Press [enter] to return...
    ```
3. Set the following keys in **PlatformInfo** -> **Generic**:
   1. **SystemProductName** to `iMacPro1,1`
   2. **SystemSerialNumber** to the value generated by _GenSMBIOS_
   3. **SystemUUID** to the value generated by _GenSMBIOS_
   4. **MLB** to the value generated by _GenSMBIOS_ (**Board Serial**)

4. Set **Quirks** -> **ProvideConsoleGop** to True.
5. Set **NVRAM** -> **UUID** -> **boot-args** to `-v keepsyms=1 debug=0x100 npci=0x2000 agdpmod=pikera`

### Step 3

After you configured _config.plist_ (don't forget to save it, btw) you can boot from your USB drive and install macOS Catalina. If your PC won't boot from this USB drive for some reason you can try resetting to UEFI defaults ("Load UEFI defaults" in Boot menu). 

It goes without saying you need to be very careful when selecting your install parition for macOS. macOS doesn't care if partition contains your current Windows install and can kill it in a heartbeat.

**NB!** Also, make sure your Windows boot partition is not affected by macOS. Your Windows EFI boot partition may be located on another drive (this is how I accidentaly disabled my Windows 10 install, it took a while to restore).

### Step 4

If you came from Windows (or even Linux) this step is not immediately obvious. Usually, after installing Windows you can boot into it without problems, because Windows will create all the boot partitions and loaders automatically. Not the case with Hackintosh. 

Once you made sure your macOS works properly you need to copy your OC boot files from USB drive to some boot partition on your hard drive. All you need is some small partition formatted as FAT32. Just copy all files from USB drive to this partition and you are done!