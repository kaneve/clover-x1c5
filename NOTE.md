# clover-x1c5
Clover Bootloader settings and fixs for ThinkPad X1 Carbon Gen5(2017) 

###############
BlackLight

参考：https://www.tonymacx86.com/threads/guide-laptop-backlight-control-using-applebacklightinjector-kext.218222/
http://bbs.pcbeta.com/viewthread-1774672-1-7.html
第一，注入核显ID驱动核显（使用hotpatch完成，SSDT-IGPU.aml）；
第二，添加PNLF设备（使用hotpatch完成，SSDT-PNLF.aml）；（不要勾选clover里的addPNLF，这个补丁与RehaMan提供的SSDT-PNLF不一样，也不要在dsdt或ssdt里打亮度补丁）；
第三，使用hotpatch的SSDT-Config.aml（已被更名为SSDT-RMCF），选择实现亮度调节的类型BKL1，设为1，LMAX值和FBTP值根据你的自己的核显来进行选择；关于LMAX，六代，七代核显要设置为0x56c；
第四，安装AppleBacklightInjector.kext驱动（好像安装在clover里不工作，所以要安装在sle或者le里面）；
第五，在clover里添加这个驱动补丁（KextsToPatch）和核显重命名补丁（DSDT patches）：
<key>KextsToPatch</key>
<array>
<dict>
<key>Comment</key>
<string>change F%uT%04x to F%uTxxxx in AppleBacklightInjector.kext (credit RehabMan)</string>
<key>Name</key>
<string>com.apple.driver.AppleBacklight</string>
<key>Find</key>
<data>RiV1VCUwNHgA</data>
<key>Replace</key>
<data>RiV1VHh4eHgA</data>
</dict>
</array>

<dict>
<key>Comment</key>
<string>change GFX0 to IGPU</string>
<key>Find</key>
<data>R0ZYMA==</data>
<key>Replace</key>
<data>SUdQVQ==</data>
</dict>
<dict>
<key>Comment</key>
<string>change PCI0.VID to IGPU #1 (Thinkpad)</string>
<key>Find</key>
<data>UENJMFZJRF8=</data>
<key>Replace</key>
<data>UENJMElHUFU=</data>
</dict>
<dict>
<key>Comment</key>
<string>change PCI0.VID to IGPU #2 (Thinkpad)</string>
<key>Find</key>
<data>VklEXwhfQURSDAAAAgA=</data>
<key>Replace</key>
<data>SUdQVQhfQURSDAAAAgA=</data>
</dict>

第六，如果还有问题，则在你的EDID里注入product-id为0x9c7c
config.plist/Graphics/EDID/Inject=true
config.plist/Graphics/EDID/ProductID=0x9c7c

第七，安装clover的RC脚本。x1c 2017原生支持nvram，不用RC脚本
第八（可选），有些笔记本电脑有DSDT中定义的环境光传感器设备。这将干扰背光恢复，因为它们往往不兼容MacOS/OSX环境光传感器驱动程序(这些驱动程序具有MacSMC依赖项)。使用RehaMan提供的ALS0.aml使_STA返回零来禁用ALS即可实现ALS的禁用。


################
Power Management

Be aware that hibernation (suspend to disk or S4 sleep) is not supported on hackintosh.

You should disable it:
Code (Text):

sudo pmset -a hibernatemode 0
sudo rm /var/vm/sleepimage
sudo mkdir /var/vm/sleepimage

Always check your hibernatemode after updates and disable it. System updates tend to re-enable it, although the trick above (making sleepimage a directory) tends to help.


#################
wifi 换成BCM94352z
参考：http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1703346&highlight=dw1830
1、clover EFI的kext中放AirportBrcmFixup.kext，BrcmPatchRAM2.kext，BrcmFirmwareData.kext，BT4LEContiunityFixup.kext，FakePCIID.kext，FakePCIID_Broadcom_WiFi.kext
2、clover中KextsToPatch打补丁
5G补丁：
Comment Enable 5G for Brcm4360
Find <4183fcff 742c48>
Replace <66c70655 53eb2b>
Name AirPortBrcm4360

Handoff Fix：
Name IOBluetoothFamily
Find <SIXAdFwPt0g=>
Replace <Qb4PAAAA61k=>

10.11+-BT4LE-Handoff-Hotspot-lisai9093：
Name IOBluetoothFamily
Find <4885ff74 47488b07>
Replace <41be0f00 0000eb44 >


#####################
声卡
参考：https://www.jianshu.com/p/29a74f0664f1
AppleALC驱动，hotpatch layoutid 3注入，Clover中DSDT打补丁：HDAStoHDEF
我自己生了codec_dump，发现原版ALC通道貌似不对，但是原版AppleALC又工作正常。
自己生了新ALC，也工作正常，暂时不知道原因～～保留相关信息在tools里，以备后续修改用。


#####################
hotpatch指南
https://www.tonymacx86.com/threads/guide-using-clover-to-hotpatch-acpi.200137/
https://www.tonymacx86.com/threads/guide-hp-probook-elitebook-zbook-using-clover-uefi-hotpatch.232948/
http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1761893&highlight=hotpatch
