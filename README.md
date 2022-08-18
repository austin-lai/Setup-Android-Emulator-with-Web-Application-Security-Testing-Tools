# Setup Android Emulator with Web Application Security Testing Tools

> Austin Lai | August 17th, 2022

---

<!-- Description -->

Setup Android Emulator (Android Studio/Genymotion) with Web Application Security Testing Tools (BurpSuite/OWASP ZAP/Fiddler Classic) to intercept android web and application traffic.

- [Genymotion](https://www.genymotion.com/)
- [Android Studio](https://developer.android.com/studio?gclid=CjwKCAjwo_KXBhAaEiwA2RZ8hOTRQ1y5o4q-rU3N1tspXohc7Ed1yAzosYIavzkKi4z3QBcBCDA_TxoCWjgQAvD_BwE&gclsrc=aw.ds)
- [PortSwigger BurpSuite Community Edition](https://portswigger.net/burp/communitydownload)
- [OWASP ZAP Proxy](https://www.zaproxy.org/download/)

<!-- /Description -->

## Table of Contents

<!-- TOC -->

- [Setup Android Emulator with Web Application Security Testing Tools](#setup-android-emulator-with-web-application-security-testing-tools)
    - [Table of Contents](#table-of-contents)
    - [Setup Genymotion with Web Application Security Testing Tools BurpSuite/OWASP ZAP/Fiddler Classic](#setup-genymotion-with-web-application-security-testing-tools-burpsuiteowasp-zapfiddler-classic)
    - [Setup Android Studio with Web Application Security Testing Tools BurpSuite/OWASP ZAP/Fiddler Classic](#setup-android-studio-with-web-application-security-testing-tools-burpsuiteowasp-zapfiddler-classic)

<!-- /TOC -->

## Setup Genymotion with Web Application Security Testing Tools (BurpSuite/OWASP ZAP/Fiddler Classic)

**Genymotion is recommended for host with zscaler or proxy installed, as Genymotion allow bridge network access that most proxy services will hinders the network connectivity**

**Possible drawback of Genymotion would be the emulator heavy load after GApps installed especially graphics extensive application**

**Step 1**

Download and install **Genymotion without virtualbox** (as virtualbox we can installed manually)

**Step 2**

Create device with below configuration:

![genymotion-create-device.gif](https://github.com/austin-lai/Setup-Android-Emulator-with-Web-Application-Security-Testing-Tools/blob/master/genymotion-create-device.gif)

- API 29 (which is android 10)
- Samsung Galaxy 10
- **RAM = 18 GB**
- Device's network setting = bridge
- After created, check and modify the device network setting to use **bridge** !!!

**Step 3**

Change Oracle VirtualBox VM Setting:

- General > Advance > Shared Clipboard
- System > Processor > Enabled PEA/NX
- Display > Screen > Max Memory + VMSGVA
- Storage > IDE > SSD ticked
- Ensure one of the network adapter is using **bridge**

**Step 4**

Power on Genymotion device.

Go to <https://www.apkmirror.com/> download apk file with x86 only compatible !!!!

Install `duduckgo` browser as it is very lightweight !

Download application you want to intercept the traffic.

Setup the application (since android 10 allow copy paste clipboard, easy setup).

Once done, you may use Genymotion `GApps` setup Google Play services !!!!

**Step 5**

Once done, reboot the device !

In the meantime, setup Burp Suite Community Edition.

Download `cacert` and setup listeners.

Remember to generate new `cacert` from Burp Suite before export to "`cacert.der`"

`cacert` need to change as below:

```
openssl x509 -inform DER -in cacert.der -out cacert.pem
openssl x509 -inform PEM -subject_hash_old -in cacert.pem |head -1
cp cacert.pem <hash>.0
```

Once ready! Use below command to copy the `cacert` to Android as **System Trusted Root Certificate**.

```
adb remount
adb push 9a5ba575.0 /system/etc/security/cacerts/9a5ba575.0
adb shell
ls -l /system/etc/security/cacerts/9a5ba575.0
```

In android, disable **mobile data** and **data roaming**.

Then go to `WIFI` > setup proxy according to Burp Suite and use command below to reboot the android device.

```
adb reboot
```

By now, you should be able to intercept web traffic open browser such as google chrome or firefox.

If succeed, try to intercept the application traffic !

If unable to intercept the application traffic, re-try generate `cacert` and push to android system folder !!!

## Setup Android Studio with Web Application Security Testing Tools (BurpSuite/OWASP ZAP/Fiddler Classic)

**While Android Studio is much smoother than Genymotion when come to emulator with graphic intensive application, however proxy services enabled on host might encounter network connectivity issue**

**Step 1:**

- Download "Android Studio" from <https://developer.android.com/studio?gclid=CjwKCAjwlqOXBhBqEiwA-hhitMF-rHFjtJo3iyMIDgwMK3vAJ-3yMzyVzrOvWmcdAdobt2SF5ZUhohoCSKAQAvD_BwE&gclsrc=aw.ds>
- Install the "Android Studio" with default configuration.
- When finish installation, DO NOT create project
- Instead, click "more option" and select "Device Emulator Manager" and "SDK Manager"
  
  ![android-studio-more-option.gif](https://github.com/austin-lai/Setup-Android-Emulator-with-Web-Application-Security-Testing-Tools/blob/master/android-studio-more-option.gif)

- Install SDK 31 which is Android 12
- Create emulator with SDK 31

**Step 2:**

Configure JAVA_HOME = "C:\Program Files\Java\jdkXXXXXX" in system environment.

Configure emulator and adb path in system environment

```
C:\Users\USERNAME\AppData\Local\Android\Sdk\emulator
C:\Users\USERNAME\AppData\Local\Android\Sdk\platform-tools
```

**Step 3:**

Power on emulator and do basic setup and android.

Download application, firefox and duduckgo browser.

**Step 4:**

Follow below link to root the ADV and install Magisk.

Setup Magisk app according to the guide or github !!!

<https://avicoder.me/2021/09/02/Root-AVD-and-install-Magisk/>

Then download <https://github.com/NVISOsecurity/MagiskTrustUserCerts> and <https://github.com/ViRb3/magisk-frida>

Copy the zip file to Android Emulator -> Download folder

Open Magisk app -> Select Module tab -> Install from package

The frida-server is always on after installed !

Getting ready Burp Suite `cacert`, export the `cacert` from proxy tab -> options -> import/export CA certificate.

May refer to this <https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/>

Save the certificate as "cacert.der"

Then run the command below (in wsl/linux):

```
openssl x509 -inform DER -in cacert.der -out cacert.pem
openssl x509 -inform PEM -subject_hash_old -in cacert.pem |head -1
mv cacert.pem <hash>.0
```

With Magisk app install, we have root shell !

Open command prompt and enter command below:

```
adb push 9a5ba575.0 /sdcard/
```

Then run below

```
adb shell
su
chmod 777 /system/etc/security/cacerts
mv /sdcard/9a5ba575.0 /system/etc/security/cacerts/
ls -l /system/etc/security/cacerts/9a5ba575.0
chmod 644 /system/etc/security/cacerts/9a5ba575.0
```

**Step 5:**

Reboot the emulator

Setup wifi and proxy under wifi

<br />

---

> Do let me know any command or step can be improve or you have any question you can contact me via THM message or write down comment below or via FB
