# Lenovo-XIAOXIN-14-2019-IWL-Hackintosh
联想小新14-2019 IWL黑苹果

## 电脑配置

| 规格 | 详细信息 |
| :----- | :----- |
| 电脑型号 | Lenovo XiaoXin-14IWL 2019 |
| 处理器 | Intel i5-8265U 1.8GHz | 
| 内存 | 板载4G + 插槽ADATA DDR4 2666 16GB（替换原有三星DDR4 2666 4G）|
| 硬盘 | 三星970 EVO 1TB（替换原有三星 PM981 1TB） | 
| 核显 | Intel UHD Graphics 620 Whiskey Lake - U GT2 |
| 独显 | NVIDIA GeForce MX230(屏蔽) |
| 内置显示器 | LG 14寸 1920x1080 FHD |
| 外接显示器 | BenQ BL2410 1920x1080 FHD （通过笔记本HDMI接口）|
| 声卡 | Realtek ALC257 |
| 网卡 | Dell 1820A CN-096JNT (替换原有Intel Wireless-AC 9560) |


## 教程
参考 [黑色小兵 Lenovo Air 13 IWL Hackintosh](https://github.com/daliansky/Lenovo-Air13-IWL-Hackintosh) 
* 开始用的CLOVER，后来换成OpenCore。不过由于OpenCore会修改Windows下的信息，暂时还是回到CLOVER。
* 系统是直接从原来的T460s TimeMachine备份直接恢复（10.14.6），然后升级到10.15.1

## 正常工作
* 核显
* 内置显示器亮度调节，快捷键也可以使用
* USB端口，包括摄像头
* 声卡
* 电源
* 网卡（冷启动需要先通过OC进入windows 10，然后重启再进入macos）
* 触控板

## 不正常工作
* 蓝牙：更新到[BrcmPatchRAM3](https://github.com/acidanthera/BrcmPatchRAM)，蓝牙可以正常显示，但是设备一连接就断开
* 读卡器： 加载了[Sinetek-rtsx](https://github.com/sinetek/Sinetek-rtsx)，可以看到设备信息，但是无法使用。这个Kext在之前的T460s上是可以正常驱动读卡器的


## 设备补丁
### HDMI接口外接显示器
按照黑色小兵的提供的配置，外接显示器没有反应，用HackinTool检查不到设备连接信息。

由于笔记本只有一个HDMI接口，最后参考[教程：利用Hackintool打开第8代核显HDMI/DVI输出的正确姿势](https://blog.daliansky.net/Tutorial-Using-Hackintool-to-open-the-correct-pose-of-the-8th-generation-core-display-HDMI-or-DVI-output.html) 收尾部分，逐步尝试修改总线ID和类型信息，得到下面配置可以点亮外接显示器:

| 索引 | 总线ID | 管道 | 类型 |
| :-- | :-- | :-- | :-- |
| 0 | 0x00 | 18 | LVDS | 
| 1 | 0x05 | 18 | DP |
| 2 | 0x02 (原为0x04) | 18 | HDMI (原为DP）|

同时由于笔记本HDMI接口不支持4K输出，最后得出的显卡补丁信息如下

```
            <key>PciRoot(0x0)/Pci(0x2,0x0)</key>
            <dict>
                <key>AAPL,ig-platform-id</key>
                <data>AACbPg==</data>
                <key>device-id</key>
                <data>mz4AAA==</data>
                <key>disable-external-gpu</key>
                <data>AQAAAA==</data>
                <key>framebuffer-patch-enable</key>
                <data>AQAAAA==</data>
                <key>framebuffer-con2-enable</key>
                <data>AQAAAA==</data>
                <key>framebuffer-con2-busid</key>
                <data>AgAAAA==</data>
                <key>framebuffer-con2-type</key>
                <data>AAgAAA==</data>
                <key>framebuffer-fbmem</key>
                <data>AACQAA==</data>
                <key>framebuffer-stolenmem</key>
                <data>AAAwAQ==</data>
            </dict>
```

### 声卡
声卡型号和小新Air 13的不一样，HackinTool显示为LayoutID有11和18两种。经过测试，11没有内置麦克风，18有，因此最后选择了18。声卡的补丁信息如下：

```
            <key>PciRoot(0x0)/Pci(0x1f,0x3)</key>
            <dict>
                <key>layout-id</key>
                <data>EgAAAA==</data>
            </dict>
```
### USB端口定制
USB端口和小新Air 13的差不多，唯一不同的就是摄像头是在HS06，因此直接修改USBPorts.kext/Contents/Info.plist，将原有的HS05部分内容替换为如下内容：
```
					<key>HS06</key>
					<dict>
						<key>UsbConnector</key>
						<integer>255</integer>
						<key>port</key>
						<data>BgAAAA==</data>
					</dict>
```


### 无线网卡
同样更换为DW1820a，不过买的不是CN-0VW3T3，然后CN-096JNT（Vendor:0x14E4, Device:0x43A3, Sub Vendor:0x1028, Sub Device:0x0022)。这个卡属于奇葩卡，折腾好好久，目前算是基本可用，但是不完美。
1. 目前没有屏蔽针脚
2. 参考[DW1820A/BCM94350ZAE/BCM94356ZEPA50DX插入的正确姿势](https://blog.daliansky.net/DW1820A_BCM94350ZAE-driver-inserts-the-correct-posture.html)对OC进行配置，包括
   * AirportBrcmFixup 2.0.4放到/EFI/OC/Kexts下面，并在OC中加载
   * 启动参数设置brcmfx-driver=1，brcmfx-country=#a
   * 添加PCI设备信息，模拟4353。注意小新14-2019 IWL网卡是挂在PciRoot(0x0)/Pci(0x1d,0x2)/Pci(0x0,0x0)下面

### 触控板
设备名称为TPAD（MSFT0001)，开始尝试使用[VoodooI2C触摸板驱动教程 | 望海之洲](https://www.penghubingzhou.cn/2019/01/06/VoodooI2C%20DSDT%20Edit/)和 [10-1-OCI2C-TPXX补丁方法](https://github.com/daliansky/OC-little/tree/master/10-1-OCI2C-TPXX%E8%A1%A5%E4%B8%81%E6%96%B9%E6%B3%95)，一直无法工作。

尝试下载代码编译，但是遇到一个比较坑的问题，总是无法从系统日志 (`log show --debug --last boot |grep Voodoo`)里面获取足够多的信息，只能看到VoodooGPIO部分信息，但是从verbose模式却是又可以看到信息。也尝试手工加载，但是由于对于代码不熟悉，总是找不到问题所在。

最后爬了两天[VoodooI2C支持讨论组](https://gitter.im/alexandred/VoodooI2C)，总算得出三个信息：
1. 有时候CLOVER总是加载kext太早，可以放到/L/E里面
2. pci8086,9de8默认的SBFG pin 0x0038是无法正常工作，需要修改为0x0108
3. SSCN/FMCN缺乏，也可能会影响

将VoodooI2C.kext/VoodooI2CHID.kext放到/L/E之后，能够看到日志了，问题得到迅速定位。
#### SSCN/FMCN缺失
日志中首先出现的错误信息就是SSCN/FMCN：
```
VoodooI2CControllerNub::pci8086,9de8 SSCN not implemented in ACPI tables
VoodooI2CControllerNub::pci8086,9de8 FMCN not implemented in ACPI tables
VoodooI2CControllerDriver::pci8086,9de8 Warning: Error getting bus config, using defaults where necessary
VoodooI2CControllerDriver::pci8086,9de8 I2C Transaction error details
VoodooI2CControllerDriver::pci8086,9de8 slave address not acknowledged (7bit mode)
VoodooI2CControllerDriver::pci8086,9de8 I2C Transaction error: 0x08800001 - aborting
Request for HID descriptor failed
Could not get HID descriptor
```

对于这个错误，[VoodooI2C触摸设备驱动教程补充](https://www.penghubingzhou.cn/2019/07/24/VoodooI2C%20DSDT%20Edit%20FAQ/)有明确的修复方法，按照SSDT-I2CxConf修复。注意的一点就是I2C0/I2C1都要修复，要不然还是会有问题，因为本机实际上还有个9de9的设备。

#### GPIO pin问题
修复上面的问题之后，重新启动，出现新问题：
```
VoodooGPIOCannonLakeLP::Registering hardware pin 49 for GPIO IRQ pin 56
VoodooGPIOXXX:: pin 49 cannot be used as IRQ
```
根据之前得到的信息，很简单，直接将原来的pinlist替换为0x0108，再次重新启动，终于顺利驱动

#### 收尾工作
将VoodooI2C相关kext从/L/E放回 C/k/O，合并I2C补丁，同样顺利驱动。


