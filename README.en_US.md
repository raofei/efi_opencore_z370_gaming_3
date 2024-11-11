#### OpenCore  Ventura|Sonoma  EFI For  Z370 AORUS Gaming 3 (rev. 1.0)  

##### Summary

This EFI SSDT comes from a version 0.8.8 of OLARILA, but it is also possible to directly use the SSDT of OpenCore's official Coffee Lake guidance configuration, this version does not unlock CFG Lock, by configuring Kernel -> Quirks -> AppleXcpmCfgLock to Boolean True, you can install Ventura normally 13.7, but upgrading to Sonoma 14.7 may result in a black screen or repeated reboots. My version is mainly a fix for this issue and an upgrade to OpenCore 1.0.2, which makes it possible to upgrade to Sonoma 14.7 without a hitch
![System Information](info.png)

##### Current status

Ventura :white_check_mark:，Sonoma :white_check_mark:，Sequoia :information_source:

Sound Card				 :white_check_mark: \
Graphics Card				 :white_check_mark: \
NIC				 :white_check_mark: \
USB Status		 :white_check_mark: \
Wake up from sleep		 :white_check_mark: 

##### Hardware configuration

CPU：Intel 8th i5-8400
Motherboard: GIGABYTE Z370 AORUS Gaming 3 (rev. 1.0) (BIOS Version F15)
Memory: Glory DDR4 3200 32G
Graphics card: UHD630
Sound card: Onboard ALC1220
Hard disk: ADATA 256G SSD
Power supply: GreatWall 750 watts

##### Procedure

1. BIOS configuration

Use the official @BIOS tool to flash the BIOS version to the latest F15 version under win, and it will restart 3 times in the middle, prohibiting intervention and restarting, wait patiently.

+ Load Optimized Defaults

+ X.M.P is not required for memory, and this option will not appear when memory of different specifications is mixed 

+ Disable Secure Boot

+ Turn off Fast Boot

+ Turn off CSM Support

+ Turn off VT-d

+ Turn off Intel Platform Trust Technology, use the PE installer to install Win11, it is only verified during the standard installation

+ Turn on XHCI Hand Off for USB configuration

+ Enable 4G EnCoding for integrated display

+ If you select igfx for display output, you will use a built-in graphics card, if you want to use a discrete graphics card, you will choose Slot 1, and the discrete graphics card will not be considered here

+ DVMT Pre Allocated changed to 128M

+ Power on RC6 (Render Standby)

1. Manually unlock the CFG Lock

+ If you don't unlock it, you can also install Ventura 13.7 by configuring config.plist, you can't install Sonoma, not sure if it's related to this

+ Download set_dump GUI tool, download the BIOS file of the latest F15 of this motherboard, use the set_dump GUI to open the BIOS file, enter CFG Lock in the search box, after a few seconds of waiting, you can see the configuration address of CFG Lock such as 0x5A4, remember this address, this value will be different for different versions of BIOS

+ Download modGRUBShell.efi, put it in the Tools directory in the OC directory of OpenCore, and use OCAT or ProperTree to import it into the configuration and save it

+ Restart the computer to enter the OpenCore menu, select modGRUBShell, and run the following command:

```
     # Note that this address is capitalized, the specific address is filled in according to what you obtained earlier, and the 0x01 returned by this command indicates that CFG Lock is locked
     setup_var 0x5A4
     # Perform unlocking after confirming the lock, and then execute the above query, and return the 0x00 is successful, and exit is executed
     setup_var 0x5A4 0x00
     ```

+ After unlocking, both AppleCpuPmCfgLock and AppleXcpmCfgLock in config.plist will be disabled
     

3. Configure config.plist, the EFI file has been configured as follows, modify it according to your own situation

+ **Important**,PlatformInfo machine information, please select**iMac 19,2** or iMac pro 1,1 as the model to generate your own three codes,**Don't use the three codes in my configuration**,It should be noted that the iMac 19,2 model does not have T2 chip detection, you can receive the upgrade push after installation, the black apple of iMac pro 1,1 cannot receive the upgrade, You need to use the RestrictEvents.kext driver and the startup parameter revpatch=sbvmm (virtual machine mode) to implement the upgrade push, which can be removed after the upgrade.

+ In order to improve the compatibility and stability of the boot memory in UEFI mode, the Booter has made the following changes:

Booter -> Quirks: AvoidRuntimeDefrag, DevirtualiseMmio, ProtectUefiServices, ProvideCustomSlide, RebuildAppleMemoryMap, SetupVirtualMap and SyncRuntimePermissions = True

+ DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x1F,0x3) The device-id must be the device-id of the sound card of your motherboard, this can be viewed in win through aida64, if the device ID is 8086 A2F0, then you need to fill in the F0A20000 here, pay attention to the reverse side, otherwise the sound card ALC1220 can not be recognized, Please fill in the information based on the value of the aida64 query.

+ DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0) -> AAPL,ig-platform-id This value Coffee lake desktop optional values **'07009B3E' and **'00009B3E'**, both of which represent a device interface rule (there is a special comparison table in the WhatEverGreen documentation), It means that there are 3 output ports, the latter is a desktop 3 output, the latter value is more compatible, and this value is recommended for installing Sonoma; device-id is the device ID of the core graphics card, which can be queried through gpu-z, and the same is written backwards, and 923E0000 is filled in when you see 3E92

+ **Here may be the main reason for the repeated black screen reboot of the installation of Sonoma**, the video is not output during the installation process, it is not that the installation is stuck, because there is system sound feedback when you hit the keyboard. Therefore, the following repairs need to be made to ensure that the configuration of the three output ports 0, 1, and 2 is the same, so that the HDMI output port can work normally:

Add the key DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0) ->force-online value Data type 01000000

Add the key DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0) ->framebuffer-con0-enable value Data type 01000000

Add the key DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0) ->framebuffer-con0-type The value of data type 00080000 indicates HDMI
   
Add the key DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0) ->enable-hdmi-dividers-fix Boolean type True
   
Add the key NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 ->boot-args boot parameter added:
     
-igfxhdmidivs igfxonln=1 **Recommended this way**, the highest priority, the alcid=3 of the sound card (this Layout-id to try by yourself, each sound card has this layout-id table query) can also be set here, the two options can be configured at the same time, and the startup command takes effect first
     
   

##### Pay attention to avoid pits

1. The motherboard model is different but the CPU is the same, it is possible to succeed with my EFI, but it is not recommended to use it directly, the wrong ACPI may cause the BIOS to reset
2. Using the general SSDT-USBX.AML (USB Power Management), SSDT-AWAC.aml (Clock Repair), SSDT-EC.aml (Emulating EC Embedded Controller to load power driver), SSDT-PLUG.AML (CPU Power Management) can usually boot the installation normally, if OpenCore uses the general acpi mode, it is recommended to install the system after Use ssdttime to generate a more complete SSDT to fix potential incompatibilities
3. If there is an endless loop of restarting the above operations, you can temporarily change the SecureBootModel to Disable and the DisableSecurityPolicy to True


##### 参考


> [EliteMacx86 Forum](https://elitemacx86.com/threads/gigabyte-z370-aorus-gaming-3-intel-core-i5-8400-32gb-ram-amd-rx-580-opencore.718/)
>
> [Z370 AORUS Gaming 3 (rev. 1.0) 支持与下载 ](https://www.gigabyte.cn/Motherboard/Z370-AORUS-Gaming-3-rev-10/support#support-dl-driver)
>
> [Desktop Coffee Lake | OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#deviceproperties)
>
> [Release Final Ventura OC EFI · kinoute/Hack-Z370-HD3P-i5-8400](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.cn.md)
>
> [声卡Layout表](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs)
>
> [acidanthera/RestrictEvents](https://github.com/acidanthera/RestrictEvents)