---
layout: post
pin: false
title: "APKdevastate: A program for analyzing APK payloads"
author: rafosw
categories: [EN]
tags: [Software]
media_subpath: /images/software_apkdevastate
image:
  path: 11.jpg
---
**Translated from Azerbaijani. Translation errors may exist!**

## APKdevastate

APKdevastate was developed using the C# programming language and identifies which RAT created the .apk payload.

## Download

APKdevastate is completely free. To download it, go to the link below, select the latest version from the Releases section, and click on the file with the APKdevastate.rar extension.

[APKdevastate Github repository](https://github.com/rafosw/APKdevastate)

[APKdevastate latest version](https://github.com/rafosw/APKdevastate/releases)

## Usage

When we open the program folder, we see:

**Resources** - A folder containing the software required for the program to function.

**temp** - A temporary folder used during analysis.

**Guna.UI2.dll** - Dynamic-link library for UI.

**APKdevastate.exe** - The program we will analyze with.

*NOTE*: If all programs and folders are not in the same directory, the program will not work.

When we launch the program, we get this warning, which indicates that the person who created this file is anonymous.

![SS](Screenshot_2.png){: width="500" height="550"}

We continue by clicking Run anyway and click the *Analyze!* button. We are asked to select the apk file to be analyzed. After selecting the apk file, this screen greets us.

![SS](Screenshot_3.png){: width="700" height="550"}

**Run!** - Starts the analysis process

**Another APK** - To select another apk file

**Guide** - Some information about how to use the program

**About** - About the program (Authors, Language)

After clicking the Run! button, we wait a bit and a series of information appears. Note that the speed of the apk file analysis process depends on the file size.

![SS](Screenshot_4.png){: width="700" height="550"}

Here we see information such as the manifest file of the apk file, permissions, hash values, whether it is malicious or not, whether it is encrypted or not, whether it has a trusted signature or not. Additionally, when we click on the Native button, we also get information about Native libraries. Here, the apk file loaded into the program is harmless and was not created by any RAT, so the Alert box tells us that the apk file is clean. But what if the APK file was created by a RAT?

![SS](Screenshot_5.png){: width="700" height="550"}

As you can see, APKdevastate tells us that the apk file is a payload created by *Metasploit* and the apk file requests these permissions from us:
```xml
android.permission.INTERNET
android.permission.ACCESS_WIFI_STATE
android.permission.CHANGE_WIFI_STATE
android.permission.ACCESS_NETWORK_STATE
android.permission.ACCESS_COARSE_LOCATION
android.permission.ACCESS_FINE_LOCATION
android.permission.READ_PHONE_STATE
android.permission.SEND_SMS
android.permission.RECEIVE_SMS
android.permission.RECORD_AUDIO
android.permission.CALL_PHONE
android.permission.READ_CONTACTS
android.permission.WRITE_CONTACTS
android.permission.WRITE_SETTINGS
android.permission.CAMERA
android.permission.READ_SMS
android.permission.WRITE_EXTERNAL_STORAGE
android.permission.RECEIVE_BOOT_COMPLETED
android.permission.SET_WALLPAPER
android.permission.READ_CALL_LOG
android.permission.WRITE_CALL_LOG
android.permission.WAKE_LOCK
android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS
android.permission.READ_EXTERNAL_STORAGE
```

These permissions can also be found in a normal apk file. How does APKdevastate know that this apk file was created by *Metasploit(msfvenom)*? In the code shown below, some RAT names are listed and each RAT creates a different apk payload. This created payload always contains code written by the RAT.
```c#
string[] ratadlariEncoded = new string[] { 
    "c3B5bm90ZQ==",           // spynote
    "c3B5bWF4",               // spymax
    "Y3JheHNyYXQ=",           // craxsrat
    "Y2VsbGlrcmF0",           // cellikrat
    "aW5zb21uaWFzcHk=",       // insomniaspy
    "Y3lwaGVycmF0",           // cypherrat
    "ZWFnbGVzcHk=",           // eaglespy
    "Zy03MDByYXQ=",           // g-700rat
    "bWV0YXNwbG9pdA==",       // metasploit
    "YnJhdGFyYXQ=",           // bratarat
    "ZXZlcnNweQ==",           // everspy
    "YmxhY2tzcHk=",           // blackspy
    "Ymlnc2hhcmtyYXQ=",       // bigsharkrat
    "ZHJvaWRqYWNr",           // droidjack
    "YW5kcm9yYXQ=",           // androrat
};
```

When the apk file is analyzed, it is checked by special code written separately for each RAT above. In this apk file, we know that the apk file is a payload according to the code below:
```c#
if (!ratFound)
{
    var smaliFilesForPayload = Directory.GetFiles(tempPath, "*.smali", SearchOption.AllDirectories);
    string payloadSmaliName = DecodeBase64("UGF5bG9hZC5zbWFsaQ=="); // Payload.smali
    
    foreach (var smaliFile in smaliFilesForPayload)
    {
        string smaliFileName = Path.GetFileName(smaliFile);
        if (smaliFileName.Equals(payloadSmaliName, StringComparison.OrdinalIgnoreCase))
        {
            ratFound = true;
            foundRatName = metasploitName;
            break;
        }
    }
}
```

This code snippet does the following:

It checks the names of all .smali files one by one.

If the file name is Payload.smali:

ratFound = true; → Indicates that a RAT was found.

foundRatName = metasploitName; → Saves the RAT name as "Metasploit".

*NOTE*: Of course, whether the apk is malicious or not does not depend solely on these codes. This is just an example.

Now let's add the facebook.apk file to the program and see what response APKdevastate gives us this time.

![SS](Screenshot_6.png){: width="700" height="550"}

APKdevastate tells us that the apk file is clean, and it tells us this based on apk signatures. When we press the Native button, we see the native libraries used by Facebook.
```console
Native Libraries Found: 2
Architectures: 1

[arm64-v8a] - 2 files
   - libbreakpad_cpp_helper.so (7.3 KB)
   - libsuperpack-jni.so (189.2 KB)
```

The programs running in the background during analysis are as follows:

aapt.exe

APKEditor.jar

apksigner.jar

apktool.jar

keytool.exe


## CONCLUSION

In this article, I showed you what the APKdevastate program does and to some extent how it works. I hope you like the program and don't forget to star our repository on Github to support us.


*Disclaimer: APKdevastate does not guarantee 100% accuracy in all detections or results. Use at your own discretion.*