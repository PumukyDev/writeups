# Android UnCrackeable Level 3

![alt text](image-13.png)

The crack will be performed on a Kali Linux machine.

The installation process for each tool will not be shown, but every step to create the environment will be documented.

To install the `.apk` an emulator is required. Create an emulator using the following command:

```bash
avdmanager create avd -n Android26 -k "system-images;android-26;default;x86_64" -c 10M
```

Verify that the device has been installed correctly:

```bash
avdmanager list avd
```

![alt text](image.png)

Start the emulator as follows:

```bash
emulator -avd Android26
```

![alt text](image-1.png)

Use the following command to list connected devices. Note the device name.

```bash
adb devices
```

![alt text](image-2.png)

Download [UnCrackable-Level3.apk](https://github.com/OWASP/mastg/raw/master/Crackmes/Android/Level_03/UnCrackable-Level3.apk) and install it on the emulator. Then open the app.

```bash
adb -s emulator-5554 install UnCrackable-Level3.apk
```

![alt text](image-3.png)

![alt text](image-4.png)

The application detects that the device is rooted and immediately closes.

![alt text](image-5.png)

To bypass this, we need to investigate the app for an anti-root check. Android APK Studio will be used for static analysis, although other tools like `jadx` or `apktool` can do the same job.

Open Android APK Studio and click "File" -> "Open" -> "APK", then select UnCrackable-Level2.apk.

![alt text](image-6.png)

Select the "Decompile java?" checkbox and click "Decompile".

![alt text](image-7.png)

![alt text](image-8.png)

![alt text](image-9.png)

![alt text](image-10.png)

![alt text](image-11.png)

![alt text](image-12.png)