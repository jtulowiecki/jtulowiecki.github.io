---
title: Setup Android Studio for Pentesting
layout: post
date: '2022-03-20 13:25:00 -0400'
categories: Setup Android
tags: android android_studio frida objection
---

### Summary
This guide will describe how to set up a test device and install the tools necessary to perform Android Security Assessments using Android Studio. 

## Install and Configure Android Studio
Download and Install [Android Studio](https://developer.android.com/studio)
<br>Add the following to your **$PATH**: 
* `$HOME/Library/Android/sdk/platform-tools`
* `$HOME/Library/Android/sdk/emulator`
* `$HOME/Library/Android/sdk/build-tools/INSERT_VERSION`

<br>

> Test that the installation and configuration is successful by running `adb` and `emulator`
{: .prompt-info }


```bash
➜  adb version
Android Debug Bridge version 1.0.41
Version 31.0.3-7562133

➜  emulator
INFO    | Android emulator version 31.2.8.0 (build_id 8143646) (CL:N/A)
ERROR   | No AVD specified. Use '@foo' or '-avd foo' to launch a virtual device named 'foo'
```

## Create VM
- Select the menu bar and choose `Virtual Device Manager`. 
![image](/assets/images/android_setup/VirtDevice.png)
> If the menu bar isn't visible, then there should be a link that says `More Actions`. Clicking on this should reveal the `Virtual Device Manager`.
{: .prompt-info }

- Select `Create device` and then choose `Pixel XL`, and `Next`.
![image](/assets/images/android_setup/CreateDevice.png)

- Select `Pie` and then choose `Next` and `Finish`. If this is the first time using the System Image, you will have to click `Download` first next to the release name.
![image](/assets/images/android_setup/Hardware.png)

> These settings can be changed according to your needs, but I have had the most success with this setup.
Avoid using any phone or tablet that has a check in the Playstore column.
{: .prompt }

## Install Burp Certificate
1. Download the Burp Suite certificate and rename it from `cacert.der` to `cacert.cer`.
2. Power on the emulator and drag the certificate to it.
3. Swipe up from the bottom and choose `Settings`.
4. Type `cert` in the search box and select `Install from SD card`.
5. Click on the menu in the top left corner.
6. Select `Downloads`.
7. Double click on `cacert.cer`, name the certificate `Burp`, and select `OK`
8. A screen should now pop up notifying you that a lock screen needs to be set. Select `SET LOCK`.
9. On the next screen, select `Continue without fingerprint`, and then `PIN`.
10. Select `NO` on the Secure start-up screen.
11. Set your pin, confirm it, and then select `Done`.
12. This will bring you back to the `Encryption & credentials` screen. Select `Install from SD card`.
13. Double click on `cacert.cer`.
14. Enter the PIN that was just created.
15. Name the certificate `Burp`, and select `OK`

## Add Certificate to System CA Store
* Power off the emulator and then select the `x` to completely close it.
* Open a new terminal.
* Using the emulator command, list all Android virtual devices. This should display the name of the device that was just created. 
```bash
➜  emulator -list-avds
Pixel_XL_API_28
```
* If all went well, enter the following, replacing `Pixel_XL_API_28` with your virtual device name.
```bash
➜  emulator -avd Pixel_XL_API_28 -writable-system
```
This will power on the device with a writable system image. 

* Open a **new** terminal and enter the following:
```bash
➜  adb disable-verity
➜  adb root
➜  adb remount
➜  adb shell "cp /data/misc/user/0/cacerts-added/9a5ba575.0 /system/etc/security/cacerts"
```

## Configure Proxy
* Select `Menu -> Settings -> Proxy`
* Set the proxy Host name and Port as shown below.
![image](/assets/images/android_setup/Proxy.png)
* Click `Apply` and exit from the menu.

## Install Frida and Objection on Host
```bash
➜ python3 -m pip install frida-tools
➜ python3 -m pip install objection
```

<br>

> Test that the installation and configuration is successful by running `frida` and `objection`
{: .prompt-info }

```bash
➜  frida --version
15.1.17
➜  objection version
objection: 1.11.0
```


## Install Frida Server on the Emulator.
* Download the latest release of [Frida Server](https://github.com/frida/frida/releases)
* Make sure to choose the build that is compatible with your emulator. For the example above, we chose `Pie` which is `x86` so I've downloaded `frida-server-15.1.17-android-x86`
* Once it is downloaded, push it to the virtual device.
```bash
➜  adb push frida-server-15.1.17-android-x86 /data/local/tmp/frida
```
* Make it executable and then run it.
```bash
➜  adb shell "chmod 755 /data/local/tmp/frida"
➜  adb shell "./data/local/tmp/frida &"
```

* Open a new terminal and use `frida-ps -Ua` to ensure that the server is running.
![image](/assets/images/android_setup/frida_install.png)
