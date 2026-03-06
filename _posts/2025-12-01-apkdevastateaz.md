---
layout: post
pin: false
title: "APKdevastate: APK payloadlarını analiz etmək üçün proqram"
author: rafok2v9c
categories: [AZ]
tags: [Software]
media_subpath: /images/software_apkdevastate
image:
  path: 11.jpg
--- 

## APKdevastate

APKdevastate C# Proqramlaşdırma dilindən istifadə edilərək hazırlanmışdır və .apk payloadını hansı RAT tərəfindən yaradıldığnı bildirir. 

## Yükləmə

APKdevastate tamamilə pulsuzdur yükləmək üçün aşağıdakı linkə keçid edib Relases bölməsindən axrıncı versianı seçin APKdevastate.rar uzantılı fayla klik edin

[APKdevastate Github reposu](https://github.com/rafok2v9c/APKdevastate)

[APKdevastate axrıncı versiya](https://github.com/rafok2v9c/APKdevastate/releases)

## Istifadə

Proqram qovluqunu açanda qarşımıza çıxır

**Resources**- Proqramın işləməsi üçün tələb olunan proqram təminatları olan qovluq.

**temp**- Analiz zamanı istifadə olunan müvəqqəti qovluq.

**Guna.UI2.dll**- UI üçün Dynamic-link library.

**APKdevastate.exe**- Analiz edəcəyimiz proqram.

*NOT*: Bütün proqramlar və qovluqlar eyni kataloqda deyilsə, proqram işləməyəcək.

Proqramı işə saldığımız zaman bu xəbərdarlığı alırıq, bu da bizə bu faylı yaradan şəxsin anonim olduğunu bildirir.

![SS](Screenshot_2.png){: width="500" height="550"}

Run anyway deyərək dəvam edirik və *Analyze!* düyməsinə klik edirik bizdən üzərində analiz aparılacaq apk faylını seçməyimiz tələb olunur, apk faylını seçəndən sonra bizi bu ekran qarşılaylır


![SS](Screenshot_3.png){: width="700" height="550"}

**Run!**- Analiz prosesini başladır

**Another APK**- Başqa bir apk faylı seçmək üçün

**Guide**- Proqramın necə istifadə olunacaqı haqqında bir sıra məlumatlar

**About**- Proqram haqqında (Yazarlar, Dil)


Run! düyməsinə klik etdikdən sonra bir qədər gözəyirik və qarşımıza bir sıra məlumatlar çıxır nəzərə alın ki, apk faylının analiz prosesinin sürəti faylın ölçüsündən asılıdır.

![SS](Screenshot_4.png){: width="700" height="550"}

Burada apk faylının manifest faylı, icazələri, hash dəyərləri, zərərli olub-olmaması, şifrəli olub-olmaması, etibarlı imzası varmı yoxmu kimi məlumatları görürük. Əlavə olaraqda Native düyməsinə klik etdiyimizdə Native kitabxanalar haqqindada məlumat almış oluruq. Burada proqrama yüklənilən apk faylı zərərsizdir və heçbir RAT tərəfindən yaradılmayıb ona görədə Alert box bizə apk faylının təmiz olduğunu bildir. Yaxşı bəs APK faylı RAT tərəfindən yaradılmış olarsa?

![SS](Screenshot_5.png){: width="700" height="550"}

Gördüyünüz kimi APKdevastate bizə apk faylının *Metasploit* tərəfindən yaradılan bir payload olduğunu bildirir və apk faylı bizdən bu icazələri istəyir:

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
Bu icazələr normal bir apk faylındada ola bilər necə olurki APKdevastate bu apk faylının *Metasploit(msfvenom)* tərəfindən yaradıldığını bilir? Aşağda göstərdiyim kodda bəzi RAT adları qeyd olunubdur və hər bir RAT fərqli apk payloadı yaradır. Bu yaradılan payload həmişə RAT tərəfindən yazılmış kodları ehtiva edir.

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

Apk faylı analiz olunan zaman yuxardakı hər bir RAT üçün ayrıca yazılan xüsusi kodlar tərəfindən yoxlanılır. Bu apk faylında isə aşağdaki koda görə apk faylının payload olduğunu bilmiş oluruq

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

Bu kod parçası aşağıdakıları edir:

Bütün .smali fayllarının adlarını bir-bir yoxlayır.

Əgər fayl adı Payload.smali-dirsə:

ratFound = doğrudur; → RAT-ın tapıldığını göstərir.

foundRatName = metasploitName; → RAT-ın adını "Metasploit" kimi saxlayır.

*NOT*: Təbii ki, apk-nin zərərli olub-olmaması təkcə bu kodlardan asılı deyil. Bu sadəcə bir nümunədir.

Indi isə gəlin birdə facebook.apk faylını proqrama daxil edək, görək bu səfər APKdevastate bizə nə cavab qaytarır.

![SS](Screenshot_6.png){: width="700" height="550"}

APKdevastate bizə apk faylının təmiz olduğunu bildirir və bunu bizə apk imzalarına əsasən deyir. Native düyməsinə basanda isə Facebookun istifadə etdiyi native kitabxanalarını görürük.

```console
Native Libraries Found: 2
Architectures: 1

[arm64-v8a] - 2 files
   - libbreakpad_cpp_helper.so (7.3 KB)
   - libsuperpack-jni.so (189.2 KB)
```

Təhlil zamanı arxa panda işləyən proqramlar aşağıdakılardır:

aapt.exe

APKEditor.jar

apksigner.jar

apktool.jar

keytool.exe


## SON

Bu yazıda APKdevastate proqramının nə etdiyini və birazda olsa necə işlədiyini sizə göstərdim. Ümid edirəm ki, proqramı bəyənirsiniz və bizə dəstək olmaq üçün Github-da repomuzu ulduzlamağı unutmayın.


*İmtina: APKdevastate bütün aşkarlamalarda və ya nəticələrdə 100% dəqiqliyə zəmanət vermir. Öz mülahizənizlə istifadə edin.*