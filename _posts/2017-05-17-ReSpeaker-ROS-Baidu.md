---
author: Sam.Lu
date: 2017-05-17 17:15:10
layout: post
title: ReSpeaker声场定位与语音识别
description: 定位与百度语音TTS、STT
tags: [Hardware]
categories: [hands-on]
image:
    feature: seeedstudio_respeaker_mic_array_front.jpg
    credit: 果动云
    creditlink: 
---
# ReSpeaker在Ubuntu下的使用

## 驱动
买来一大套ReSpeaker开发板，一块是麦克风阵列板，一块是OpenWRT板，一块是Groovy传感器扩展板。其实有用的就是那块麦克风阵列板。

这个板子默认在ubuntu下是免驱的，我的ubuntu用的是mbp的虚拟机，插上ReSpeaker麦克风阵列后，直接去右上角小喇叭处，就可以选择输入设备了。选择后，我们还可以设置收音的增益，感觉音柱不是很灵敏啊，真的有10米收音范围么？还是我所在的工作环境太乱...

## 百度语音加持
方向决定达成成果的速度。
我一上来还是去了讯飞官网，然后找了半天，说实话网站好乱啊，找了半天，放弃了。虽然我在树莓派2上面成功测试过讯飞的TTS和STT，但是确实够折腾。

第二个选择就是百度语音，搜来搜去发现百度语音竟然有ROS的package，网址：https://github.com/DinnerHowe/baidu_speech，如何运行请看github上面对应的wiki，简单到想哭。

然后就是测试啦，先测试了一下文字转语音，你会打开两个ROS node，一个等着，往另一个输入文字，然后回车，等着的那个node就会把你输入的文字说出来了；然后测试了语音识别，按下回车，然后6秒内必须说啊说，最长识别设置的是60s，实测识别率还是可以的，这里用到的硬件就是ReSpeaker的麦克风阵列了，距离超过1米感觉识别率下降挺多的

## ReSpeaker声场定位
声场定位，是耳机一个重要参数（闭嘴，不许再谈耳机！）
我们对着ReSpeaker说话时，它的指示灯就显示了音源方位，难道加一个摄像头，通过机器视觉来识别语音方位吗...当然不是，去ReSpeaker官方Github的issue里面骂街，立刻有技术人员出来平事，我们得到了一个python代码，如下

```
import usb.core
import usb.util


class PyUSB:
    """
    This class provides basic functions to access
    a USB HID device using pyusb to write an endpoint
    """

    def __init__(self):
        self.dev = usb.core.find(idVendor=0x2886)

        # get active config
        config = self.dev.get_active_configuration()

        # iterate on all interfaces:
        #    - if we found a HID interface
        for interface in config:
            if interface.bInterfaceClass == 0x03:
                interface_number = interface.bInterfaceNumber
                break

        try:
            if self.dev.is_kernel_driver_active(interface_number):
                self.dev.detach_kernel_driver(interface_number)
        except Exception as e:
            print(e)

        ep_in, ep_out = None, None
        for ep in interface:
            if ep.bEndpointAddress & 0x80:
                ep_in = ep
            else:
                ep_out = ep

        self.ep_in = ep_in
        self.ep_out = ep_out
        self.interface_number = interface_number

    def write(self, data):
        """
        write data on the OUT endpoint associated to the HID interface
        """

        # report_size = 64
        # if self.ep_out:
        #     report_size = self.ep_out.wMaxPacketSize
        #
        # for _ in range(report_size - len(data)):
        #    data.append(0)

        if not self.ep_out:
            bmRequestType = 0x21              # Host to device request of type Class of Recipient Interface
            bmRequest = 0x09              # Set_REPORT (HID class-specific request for transferring data over EP0)
            wValue = 0x200             # Issuing an OUT report
            wIndex = self.interface_number  # SeeedStudio ReSpeaker interface number for HID
            self.dev.ctrl_transfer(bmRequestType, bmRequest, wValue, wIndex, data)

        self.ep_out.write(data)


    def read(self):
        return self.ep_in.read(self.ep_in.wMaxPacketSize, -1)


    def close(self):
        """
        close the interface
        """
        usb.util.dispose_resources(self.dev)



if __name__ == '__main__':
    import signal
    import time

    is_quit = False
    def handler(signum, frame):
        global is_quit
        is_quit = True
        print 'quit'

    signal.signal(signal.SIGINT, handler)

    hid = PyUSB()

    data = None
    while not is_quit:
        new_data =  hid.read()
        if len(new_data) == 64 and data != new_data:
            data = new_data
            print [int(c) for c in data]
            vad = data[4]
            direction = int(data[5]) + (int(data[6]) << 8)
            print 'vad: {}, direction: {}'.format(vad, direction)
           
```

当然，这个代码不是ROS的package，只需要python run一下就行，实测它可以给出音源的角度，0~330，30度一个，一共12个，正好一圈，这样你就可以识别音源方位，然后让你的机器人听取召唤啦~