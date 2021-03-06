
# AirDrop/Continuity for those unsupported Mac device 

## use Air Drop with ethernet

```bash
defaults write com.Apple.NetworkBrowser BrowseAllInterfaces 1
```



## Catalina Continuity Activation 

For earlier mac device which has replaced with newer wifi-bluetooth moudle:

https://github.com/dokterdok/Continuity-Activation-Tool/issues/463


```bash
// 重要
sudo mount -uw /
```

```bash
// 另一个，不确定是否需要
$ ioreg -l | grep "board-id" | awk -F\" '{print $4}'
Mac-*YourBoardID*


sudo /usr/libexec/PlistBuddy -c "Set:Mac-*YourBoardID*:ContinuitySupport true" "/System/Library/Frameworks/IOBluetooth.framework/Versions/A/Resources/SystemParameters.plist"


sudo -E perl -pi -e "s/\Mac-00BE6ED71E35EB86/\Mac-*YourBoardID*/" /System/Library/Extensions/IO80211Family.kext/Contents/PlugIns/AirPortBrcm4360.kext/Contents/MacOS/AirPortBrcm4360

sudo -E perl -pi -e "s/\Mac-00BE6ED71E35EB86/\Mac-*YourBoardID*/" /System/Library/Extensions/IO80211Family.kext/Contents/PlugIns/AirPortBrcmNIC.kext/Contents/MacOS/AirPortBrcmNIC
```

# NUC8i5BEK Hackintosh

## NUC8i5BEK BISO setting

- « Intel VT for directed I/VO (VT-d) » ： disabled
- « Secure Boot » ： disabled
- « Fast Boot » ： disabled
- UEFI boot enalbe /   Legacy Boot disable
- SATA mode ： AHCI
- Boot->Boot Configuration-> « Boot Network Devices Last » ： disabled
- Power->Secondary Power Settings, « Wake on LAN from S4/S5 », set to « Stay Off »
- Devices->Video, « IGD Minimum Memory » set to 64mb
- Devices->Video, « IGD Aperture Size » set to 256mb
- disable reader-card

## Clover USB Creation

- [post on tonymac86](https://www.tonymacx86.com/threads/guide-booting-the-os-x-installer-on-laptops-with-clover.148093/#post-917900)

- Recommend: MBR with a FAT32 partition for Clover and a separate HFS+J partition for the OS X installer.
    - most computers can boot from it (even with UEFI), and it is convenient that the EFI partition will mount automatically when the USB is inserted.

- step 1,find your USB disk, in this case its /dev/disk1
    - 
    ```bash
    diskutil list
    /dev/disk0 (internal, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:      GUID_partition_scheme                        *500.1 GB   disk0
       1:                        EFI EFI                     209.7 MB   disk0s1
       2:                  Apple_HFS 10.10.x                 80.0 GB    disk0s2
       3:                  Apple_HFS 10.11.gm1               80.0 GB    disk0s3
       4:       Microsoft Basic Data Win10_TP                79.4 GB    disk0s4
       5:                  Apple_HFS 10.10.test              80.0 GB    disk0s5
    /dev/disk1 (external, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:                                                   *8.0 GB     disk1[/B]
    ```
- step 2, partion , in this example, it create 3 partions 
    - 
    ```bash
    diskutil partitionDisk /dev/disk1 3 MBR FAT32 "CLOVER EFI" 200Mi HFS+J "install_osx" 12000Mi ExFat "ExFat" R
    Finished partitioning on disk1
    /dev/disk1 (external, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:     FDisk_partition_scheme                        *31.0 GB    disk1
       1:                 DOS_FAT_32 CLOVER EFI              209.7 MB   disk1s1
       2:                  Apple_HFS install_osx             12.6 GB    disk1s2
       3:               Windows_NTFS ExFat                   18.2 GB    disk1s3
    ```
    - Note: If you're using Clover legacy, the USB should must be MBR.
    - Note: Some BIOS implementations require GPT, some require MBR (many allow both). If you can't get BIOS to recognize your USB for booting, try GPT instead of MBR.
- Clover
    1. download [Clover installer](http://sourceforge.net/projects/cloverefiboot) 
        - or [CloverHackyColor](https://github.com/CloverHackyColor/CloverBootloader/releases)
    2. run Clover installer
        - if using MBR, select the target of the install to "CLOVER EFI" using "Change Install Location"
        - select "Customize" (the default is a legacy install -- we need to change it)
            - check "Install for UEFI booting only", "Install Clover in the ESP" will automatically select
            - 
- OSX installation
    - 
    ```bash
    sudo /Applications/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia --volume /Volumes/install_osx/ --nointeraction --downloadassets
    ```

## Misc

```
DataHubDxe-64.efi = DataHub protocol and mandatory for macOS.
FSInject-64.efi = Driver responsible for Clover kext injection into kernel cache.
SMCHelper-64.efi = Restore SMC keys left in NVRAM by FakeSMC (VirtualSMC.efi incompatible)
VBoxHfs-64.efi = Open source EFI file-system driver for HFS+ file system.
+
ApfsDriverLoader-64.efi = Recognize and boot from APFS volumes needed by Mojave.
AptioMemoryFix-64.efi = Needed for Afps driver; includes NVRAM fixes and better memory management.
EmuVariableUefi-64.efi = Support for NVRAM variables if hardware NVRAM is not supported.
NvmExpressDxe-64.efi = Driver support for NVM Express devices.
```


## 改3码

0. Hackintool 查看 System ID,  Gq3489ugfi 等一堆信息...
1. 挂载硬盘的EFI分区用Clover Configurator打开config.plist..
2. System Parameters /  从系统中获取
3. RT Variable  / 依次点击 from SMBIOS 获取 然后 from system /  hack 获取。command+s  保存

> 有同学说不是三码吗，怎么才改了两个地方？其实还有一个最重要的序列号没改，这就是为什么安装后序列号搜是一样的原因。主要是因为我们是黑苹果，只需要iMessage info就够了，尽量避免污染其他的序列号。改动方式是，选择SMBIOS，然后点击生成新的（generate）。

> 需要 EmuVariableUefi-64.efi


## Clover 自动启动

use volume name of your OSX volume.

## Clover 目录结构

- BOOT   clover引导文件
- CLOVER 
    1. ACPI/
        - origin  提取/储存 DSDT/SSDT 的问题， 如果你需要修改你的 DSDT/SSDT, 可以在4叶草引导界面下 直接按F4进行提取，提取的DSDT/SSDT 就会存放到这里。
        - patched  提取的DSDT/SSDT 修改好后，放到这个文件夹中，它会被自动加载。
    2. CLOVERX64.efi  引导clover最主要的文件之一
    3. config.plist   clover引导配置文件. 通过它的配置，可以驱动你的声卡 显卡 睡眠，等等.
    4. doc  clover 帮助文档
    5. driver   驱动 legacy boot
    6. driver64-UEFI  驱动 for UEFI boot
    7. kerts   可以配置各个OS版本的驱动，一般只留下Other 一个文件夹
        - Other
            - Lilu.kext  补丁驱动，多用于配合其他驱动使用
            - AppleALC.kext  Realtek onboard audio  需要配合lilu.kext使用，主要的作用就是加载原生AppleHDA声卡驱动。
            - VoodooHDA-2.9.1.kext黑苹果万能声卡驱动
            - CodecCommander.kext   解决耳机有杂音和睡眠唤醒无法自动切换或无声的问题
            - CPUFriend.kextCPU  变频动态注入 CPU 电源管理数据
            - Fake-PCI-ID.kext  牛逼
            - USBInjectAll.kext  In 10.11+ Apple has changed significantly the way the USB drivers work.
            - WhateverGreen.kext  Lilu plugin providing patches to select GPUs on macOS. 显卡驱动补丁集
            - XHCI-unsupported.kext： 英特X99系列主板驱动
    8. misc  主要存放日志，用于排错
    9. OEM   存放不同主板电脑型号的一个配置文件. OEM可以包含整个clover文件夹下面的所有文件(除了OEM). 除了你希望用一个引导去引导多台电脑，否则没什么用
    10. ROM  一般用于存放显卡提取
    11. themes 开机引导界面主题
    12. tools  没什么用, 一般可以忽略


## Clover 引导界面快捷键

- F4 如上
- F5 保存初步修复的 DSDT
- F6 保存显卡 BIOS 文件


## active native bluetooth

- brew install libusb
- firmware source:  https://github.com/wulf7/iwmbt-firmware
- firmware download: https://www.tonymacx86.com/attachments/bluetooth_fix-zip.439550/
- thread: https://www.tonymacx86.com/threads/guide-intel-nuc7-nuc8-using-clover-uefi-nuc7i7bxx-nuc8i7bxx-etc.261711/page-207

```bash
sudo launchctl load -w /Library/LaunchDaemons/iwmbtfw.plist

iwmbtfw.plist
---------------------------------------------------------------------------------
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.iwmbtfw.firmwares</string>
	<key>ProgramArguments</key>
	<array>
		<string>/etc/rc.boot.d/30.fix_bluetooth.local</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>KeepAlive</key>
	<true/>
</dict>
</plist>
```

Does it need specifiy root user ?

```xml
<key>UserName</key>
<string>root</string>
```

# Misc 


## Centos bootable USB installer on OSX

https://cloudwrk.com/create-centos-7-bootable-usb-on-osx/

1. Format a USB drive as eFAT with Disk Utility
2. Download CentOS 7 Full or Minimal iso file
3. Open terminal and find USB drive partition name with command “diskutil list“
4. In terminal, use dd command to copy CentOS iso to usb
5. Test usb drive in Parallels virtual machine

```bash
sudo dd if=./Downloads/CentOS-7-x86_64-DVD-1611.iso of=/dev/rdisk2 bs=1m
```

**Note the additional “r” prepended to the usb partition name rdisk2 instead of disk2**.


 



