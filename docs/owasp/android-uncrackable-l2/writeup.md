# Android UnCrackable Level 2

![alt text](./images/image-40.png)

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

![alt text](./images/image-3.png)

Start the emulator as follows:

```bash
emulator -avd Android26
```

![alt text](./images/image-4.png)

Use the following command to list connected devices. Note the device name.

```bash
adb devices
```

![alt text](./images/image-5.png)

Download [UnCrackable-Level2.apk](https://github.com/OWASP/mastg/raw/master/Crackmes/Android/Level_02/UnCrackable-Level2.apk) and install it on the emulator. Then open the app.

```bash
adb -s emulator-5554 install UnCrackable-Level2.apk
```

![alt text](./images/image-6.png)

![alt text](./images/image-7.png)

The application detects that the device is rooted and immediately closes.

![alt text](./images/image-8.png)

To bypass this, we need to investigate the app for an anti-root check. Android APK Studio will be used for static analysis, although other tools like `jadx` or `apktool` can do the same job.

Open Android APK Studio and click "File" -> "Open" -> "APK", then select UnCrackable-Level2.apk.

![alt text](./images/image-41.png)

Select the "Decompile java?" checkbox and click "Decompile".

![alt text](./images/image.png)

Analyzing the code, we can see that in `MainActivity.java` a method `a` is called after detecting a rooted device and the application then exits.

![alt text](./images/image-2.png)

To bypass this we will use Frida. For better visualization I will complement it using RMS (Runtime Mobile Security), which is a web interface that simplifies using Frida.

RMS is written in JavaScript, so we need to install npm and the RMS package.

```bash
sudo apt install -y npm
sudo npm install -g rms-runtime-mobile-security
```

A Frida server needs to be running on the Android device. Go to the Frida releases page and download the latest server version for Android x86_64.

![alt text](./images/image-9.png)

Unpack the downloaded server.

```bash
unxz frida-server-17.8.2-android-x86_64.xz
```

Give the emulator root privileges.

```bash
adb -s emulator-5554 root
```

![alt text](./images/image-10.png)

Copy the server to `/data/local/tmp` on the emulator.

```bash
adb -s emulator-5554 push frida-server-17.8.2-android-x86_64 /data/local/tmp/
```

![alt text](./images/image-11.png)

Make the file executable.

```bash
adb -s emulator-5554 shell "chmod 755 /data/local/tmp/frida-server-17.8.2-android-x86_64"
```

Run it listening on all interfaces at port 27042.

```bash
adb -s emulator-5554 shell "/data/local/tmp/frida-server-17.8.2-android-x86_64 -l 0.0.0.0:27042 &"
```

In another terminal forward the local port 27042 to the device.

```bash
adb -s emulator-5556 forward tcp:27042 tcp:27042
```

![alt text](./images/image-12.png)

Start RMS.

```bash
rms
```

![alt text](./images/image-13.png)

Open a web browser and navigate to http://127.0.0.1:5491/ to see the interface. Then select:

- Mobile OS: Android.
- Package name: owasp.mstg.uncrackable2.
- Spawn or Attach: Spawn.

Click "Load Default Frida Scripts".

![alt text](./images/image-20.png)

Several scripts will appear. For now we only need script number 24, called `system_exit_bypass.js`. Click it.

![alt text](./images/image-21.png)

The script should be loaded. Click "Start RMS" to run the script and bypass the root protection.

![alt text](./images/image-22.png)

Return to the mobile application and check the console logs to confirm the script is loaded. If you click "OK" when root is detected, the message "System.exit() Bypassed!" should appear and the application will remain open.

![alt text](./images/image-23.png)

![alt text](./images/image-24.png)

After bypassing the root protection, a form appears to enter a secret. Enter "secret" and click "Verify" to see what happens.

![alt text](./images/image-25.png)

An error message is displayed. We now need to obtain the actual secret code.

Return to RMS. Navigate to the settings panel, fill in the data as before, and click "Start RMS".

![alt text](./images/image-20.png)

![alt text](./images/image-21.png)

![alt text](./images/image-22.png)

Click "Load classes".

![alt text](./images/image-15.png)

Several classes will be loaded. Click "Insert a Filter" to view only those related to the main activity.

![alt text](./images/image-16.png)

Add a filter to hook classes starting with `sv.vantagepoint` and click "Submit".

![alt text](./images/image-17.png)

Click "Load Methods" to view and analyze them.

![alt text](./images/image-18.png)

![alt text](./images/image-19.png)

Click "Hook all methods".

![alt text](./images/image-27.png)

Return to the Android application and click the "Root detected!" message.

![alt text](./images/image-8.png)

In RMS you will see a console showing which methods have been called.

![alt text](./images/image-28.png)

Enter any secret and click "Verify".

![alt text](./images/image-25.png)

Messages will appear in the console. In this case, we can see that it calls `CodeCheck`.

![alt text](./images/image-29.png)

Searching in `MainActivity` using Android APK Studio, we see that it loads the native library `foo`.

![alt text](./images/image-30.png)

This library can be found under `/lib/<architecture>/libfoo.so`. However, we cannot read it directly, so we need to decompile it.

![alt text](./images/image-31.png)

To do that we will use Ghidra. Rename the `.apk` to `.zip`, extract it, open `libfoo.so` in Ghidra, and double-click it.

![alt text](./images/image-33.png)

Analyze the code and search for the part where the check is performed.

![alt text](./images/image-34.png)

![alt text](./images/image-35.png)

In the Symbol Tree under Exports you can see `CodeCheck_bar`. That is exactly what we need to analyze to find how to obtain the password.

![alt text](./images/image-36.png)

In the "Decompile" tab you will find a `strncpy` containing the password. Ghidra has decompiled it and shows the password in plain text.

![alt text](./images/image-37.png)

Return to the UnCrackable-l2 application, enter the secret string, and click "Verify".

![alt text](./images/image-38.png)

You should see that the secret is correct.

![alt text](./images/image-39.png)