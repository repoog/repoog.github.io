---
title: 使用Arduino UNO制作BadUSB
date: 2021-08-02 19:40:01
author: repoog
summary: BadUSB攻击已经是多年前的技术了，攻击成本只需要十来块人民币，网络上有很多基于诸如Rubber Ducky的制作方法。笔者用手头的Arduino设备想制作BadUSB的攻击设备，本文即是介绍基于Arduino UNO的BadUSB攻击方法（当然，其他类型的Arduino设备也可以使用）。
comments: true
tags:
  - Arduino
  - Arduino UNO
  - BadUSB
  - Flip
categories:
  - DIY
  - 硬件安全
---

2014年Black Hat会议上，由Jakob Lell和Karsten Nohl展示，通过USB接口让USB设备能够成为攻击设备攻击具有USB端口的电脑。

### **BadUSB原理**

BadUSB攻击主要是利用HID（Human Interface Device）的交互性，将攻击载荷植入到HID中，从而通过键盘鼠标控制设备，而键盘能够实现鼠标所有的操作，因此BadUSB主要是利用虚拟键盘进行攻击。由于是作为虚拟键盘，攻击载荷存放在USB设备的固件区域，因此杀毒软件无法扫描到BadUSB攻击。

![BadUSB原理](images/2021/08/badusb.png 'BadUSB原理')

BadUSB利用了USB设计上的漏洞，处于对USB设备的兼容性及免驱的要求，USB设计上没有要求对应的设备特征，而是拥有多种USB设备的特征，这样通过重写固件，可以将USB设备伪装为USB键盘，从而通过键盘输入固件中的指令。

从攻击原理看，只要能够作为键盘的HID设备都可以实现BadUSB效果，比如Teensy、Arduino UNO/Leonardo、树莓派。

### **BadUSB防御**

* 采用只充电无数据传输的USB转接头；
* 组策略-计算机配置-管理模版-系统-设备安装-设备安装限制；

### **使用Arduino UNO做BadUSB**

ATmega16U2芯片是Arduino开发板中作为电脑USB接口和主处理器串口的桥梁，之前版本中的UNO板中使用的是ATmega8U2芯片。ATmega16U2芯片运行的固件可以通过特殊的USB协议（即DFU，设备固件升级）来升级。

![Arduino UNO](images/2021/08/arduino_uno.jpg 'Arduino UNO')

Arduino UNO默认使用最新版本的ATmega16U2固件，检查固件是否需要升级可以检查设备管理器中Arduino串口的驱动信息。

根据上文BadUSB原理，要将Arduino UNO作为BadUSB的设备，需要将其改造为能够被电脑识别为HID设备，因此刷ATmega16U2固件为键盘固件程序（文末有UNO和Keyboard固件程序下载）。刷芯片固件需要将Arduino UNO设置到DFU模式下，并使用Flip工具。

* Flip下载地址：http://www.microchip.com/developmenttools/productdetails.aspx?partno=flip
* Arduino固件下载地址：https://github.com/arduino/ArduinoCore-avr/tree/master/firmwares/atmegaxxu2

设置Arduino UNO处于DFU模式，首先使用USB线连接Arduino与电脑，让Arduino中的重置针脚短路，实现重置8u2或16u2芯片。如果重置成功，在Arduino IDE中将不会出现串口信息。

![DFU模式下的重置](images/2021/08/DFU_reset.jpg 'DFU模式下的重置')

要基于Arduino UNO制作BadUSB，首先需要在UNO默认固件（Arduino-usbserial-uno.hex）下编写、编译和上传代码，由于BadUSB的攻击基于模拟键盘输入，因此代码中需要使用HIDKeyboard库（https://github.com/SFE-Chris/UNO-HIDKeyboard-Library），Arduino官方的Keyboard库无法在默认固件下编译和上传，会报error: 'Keyboard' not found. Does your sketch include the line '#include '，且不同于Arduino Leonardo，一旦将固件刷为HID，将无法上传代码。

示例代码如下，该代码将通过run命令在浏览器中访问本站：

``` C
#include <HIDKeyboard.h>

HIDKeyboard keyboard;

void setup() {
  keyboard.begin();
  delay(200);
}

void loop() {
  delay(200);
  keyboard.pressSpecialKey(GUI);
  keyboard.releaseKey();
  delay(200);
  
  keyboard.println("run");
  keyboard.pressSpecialKey(ENTER);
  keyboard.releaseKey();
  delay(1000);
  
  keyboard.println("https://peirs.net");
  keyboard.pressSpecialKey(ENTER);
  keyboard.releaseKey();
  delay(10);
  
  while(1);
}
```

代码上传后，将Arduino UNO至于DFU模式，使用Arduino-keyboard-0.3.hex固件程序刷新固件，从而将Arduino UNO伪装为键盘。将Arduino设备置于DFU模式后，运行Flip，选择目标设备是ATmega16U2，再选择通讯媒介是USB，USB协议传输打开后，左侧的Operation勾选框变为可选，最后通过Load HEX File，选择Arduino-keyboard-0.3.hex固件程序，点击Run刷入固件程序。

固件写入成功后，重新插入Arduino的USB连接线，即可看到Arduino UNO作为键盘被电脑识别，之前写入的代码也会运行。

![Flip截图](images/2021/08/Flip_1.png 'Flip截图')

![Flip截图](images/2021/08/Flip_2.png 'Flip截图')

![Flip截图](images/2021/08/Flip_3.png 'Flip截图')

操作Flip的过程中，若出现AtLibUsbDfu.dll not found的错误，则可以在设备管理器中，更新未知设备（unknown device）的驱动，选择驱动是Flip安装路径下的USB目录，完成后，在设备管理器中可见ATmega16U2的设备。

![ATmega16u2设备](images/2021/08/device.png 'ATmega16u2设备')

如果需要重新编辑代码，则需要重新设置Arduino的DFU模式，刷Arduino-usbserial-uno固件，刷新固件后，需要重新插入USB线，让电脑识别Arduino UNO串口，再编写、编译、上传代码。

[Arduino-firmware下载](/images/2021/08/Arduino-firmware.zip)