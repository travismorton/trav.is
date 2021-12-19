---
title: "ADB over TCP/IP over Zerotier"
date: 2019-11-5
layout: post
subtitle: Because sometimes it's necessary
permalink: /adb-zerotier
---

While working on an Android device in the field, I usually find myself needing to connect via ADB wirelessly. This is where the standard TCP/IP mode of ADB comes in handy, although when testing outside of wifi range this is useless since ADB needs to be on the same LAN. Luckily, Zerotier does just that for us so let's see it in practice.

# Access on Wifi
First, get ADB running in TCP/IP mode on each device you want to test. Plug in the device to USB and connect to wifi. Take note of the private IP, either from your router or from the settings menu. Then, issue the following commands:

```shell
> adb usb
> adb tcpip 5555
```

*unplug your device from usb*
```shell
> adb connect <IP>:5555
```
Now you should see it in your devices list:

```shell
> adb devices
List of devices attached
192.168.50.52:5555    device
```

To get a shell just specify the device and port:
```shell
> adb -s 192.168.50.52:5555 shell
```
Now follow the same steps for each device you want to connect.

# Access on Cell
Using Zerotier we can create a private LAN across multiple physical LANs. Create a Zerotier network at [https://my.zerotier.com](https://my.zerotier.com). In this example, the subnet is 10.242.0.0/16. From a command line join the network (or from a GUI if your OS supports it):

```shell
> sudo zerotier-cli join <network_id>
```

Now, install the zerotier APK. If your device has a screen and Google Play Services, just download this from the Google Play Store. If not, you will need to pull the APK from a device with Google Play Services and use [scrcpy](https://github.com/Genymobile/scrcpy) to open the Zerotier app to join the same network. Make sure to authorize each from the [web portal](https://my.zerotier.com).

```shell
> adb -s 192.168.50.52:5555 install zerotier.apk
> scrcpy -s 192.168.50.52:5555
```

Once the device is authorized on the web portal, connect to the zerotier IP given using the same steps from above. You may need to send traffic over the route to initialize it.

```shell
> ping 10.242.206.180
> adb connect 10.242.206.180:5555
> scrcpy -s 10.242.206.180:5555
```

Now, feel free to turn off wifi, however latency will go up and scrcpy will be hard to use (but you will still have an ADB shell). This is now using ADB over TCP/IP over a cell connection!

