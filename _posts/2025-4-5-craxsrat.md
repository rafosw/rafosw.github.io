---
layout: post
pin: false
title: "CraxsRat: Android RAT proqramına texniki baxiş"
author: rafosw
categories: [AZ]
tags: [RAT, Malware, Reverse Engineering ]
media_subpath: /images/trojan_craxsrat
image:
  path: craxsratlogo.webp
---

## Analiz

Hal-hazırda ən çox istifadə olunan Android RAT-lar suriyalı təhdid aktoru olan EvLFDevZ-ə aiddir və CraxsRat da onun proqramlaşdırdığı bir RAT-dır.
Sayt bağlı olsa da, Wayback Machine vasitəsilə sayta baxmaq mümkündür. RAT bu saytda reklam olunur və Telegram üzərindən satılır. Sol tərəfdə RAT-ın funksiyaları görünür. 


![Web 80 Index](1.webp){: width="800" height="550"}

Nəysə, gəlin RAT-ı zip faylından çıxaraq içindəki fayalara baxaq.
                                                                                                                                                                        
![Web 80 Index](2.webp){: width="800" height="550"}

Burada gördüyünüz res qovluğuna bir azdan qayıdacağıq. Qısa olaraq deyim ki, içərisində zərərli Android proqramını yaratmaq üçün lazım olan fayllar var və digər .dll fayllari isə proqramın bir hissəsidir.

İndi isə proqramı çalışdıraq (nəzərə alın ki, bunların hamısı virtual maşın daxilində edilir)

![Web 80 Index](3.webp){: width="800" height="550"}

**Dashboard**:Bu, proqramın əsas ekranıdır və istifadəçiyə ümumi bir baxış təqdim edir. İstifadəçi burada mövcud vəziyyət, aktiv cihazlar və əlaqələr barədə məlumat ala bilər.

**Clients**:Bu bölmə, əlaqə qurulan bütün cihazların siyahısını təqdim edir. Yəni, proqramın hədəflədiyi cihazlar burada görünür.

**Servers**:CraxsRat bir növ C2 (Command and Control) olduğu üçün burada CraxsRat-ın işləyəcəyi portu təyin edirik. IP ünvanını isə “Builder” hissəsində yazacağıq.

**Connections**:Bu bölmə, aktiv əlaqələri və bu əlaqələrlə bağlı məlumatları göstərir. Hansı cihazlarla məlumat mübadiləsi aparıldığına dair detalları bu sekmədə görə bilərsiniz.

**Builder**:Bu bölmə, istifadəçiyə zərərli proqram və ya RAT yaratma vasitəsi təqdim edir. İstifadəçi burada hədəf cihazlara yükləmək üçün proqram hazırlayır.

Əsas olan bölmələr bunlardı, digər bölmələri yazmadım, çünki o qədər də lazım deyil. İndi isə bir .apk yaradaq və onu analiz etməyə başlayaq.

Aşağıda APK-nın sorğu göndərəcəyi IP və portu yazdıq.

![Web 80 Index](4.webp){: width="800" height="550"}

APK “ready.apk” adı ilə yaradıldı, amma əvvəlcə APK-nı analiz etmədən öncə gəlin birinci proqramın APK faylını necə hazırladığını, proqramı dekompilyasiya edərək öyrənək.

![Web 80 Index](5.webp){: width="800" height="550"}

dnSpy istifadə edərək CraxsRat-ı dekompilyasiya etdik, Assembly Explorer hissəsində bizə lazım olan build namespaces-i tapdıq və artıq build hissəsinin source kodunu görürük (əvvəlcə proqramı deobfuscate etmək lazımdır, yoxsa source kodunu oxumaq mümkün deyil).

![Web 80 Index](6.webp){: width="800" height="550"}

Bu kod parçacığı zərərli icazələri APK-ya yazır.

![Web 80 Index](7.webp){: width="800" height="550"}

Başda demişdim ki, res qovluğuna birazdan baxacağıq və hal-hazırda res qovluğunda olan apktool-u görürsünüz. Qısa olaraq gəlin, apktool vəcertificate.pem-in nə etdiyinə baxaq.

**apktool**: Bu alət, APK faylını dekompilyasiya etmək və yenidən qurmaq üçün istifadə olunur. apktool həmçinin APK-nın içindəki resursları və manifest faylını idarə etməyə imkan verir. Bu, zərərli proqramların qurulmasında və özəlləşdirilməsində istifadə olunur.

**certificate.pem**: Bu fayl, proqramın imzalanmasında istifadə edilən sertifikatdır. APK fayllarını imzalamaq üçün istifadə olunur və təhlükəsizlik məqsədilə də tətbiqin doğruluğunu təmin edir.

![Web 80 Index](8.webp){: width="800" height="550"}
 
## APK İmzalama və Fayl Yazma

Koddakı bu sətir, imzalama əməliyyatı ilə əlaqədardır ilk olaraq, certificate.pem faylını silir
Sonra, res -dən yeni bir certificate.pem faylı yazılır.:

```c#
File.Delete(this.folder_apktool + "\\certificate.pem");
File.WriteAllBytes(this.folder_apktool + "\\certificate.pem", Resources.c2);
```
## APK Faylını İmzalama Komandaları

Burada, apktool ilə bağlı imzalama komandaları belədir:
apktool.bat faylını işə salaraq APK faylını imzalamaq üçün lazım olan komandaları yaradır. Bu komandalar, apktool vasitəsilə APK faylını imzalayır və certificate.pem sertifikatını istifadə edir.


```c#
string.Concat(new string[]
{
    this.folder_apktool + "\\apktool.bat",
    " sign --key ",
    this.folder_apktool + "\\certificate.pem"
});
```

Ümumi Xülasə:

Bu kod, bir APK faylını almaq, lazımi imza əməliyyatları üçün sertifikat faylı (certificate.pem) istifadə etmək və APK-ni apktool ilə imzalamaq üçün bir sıra əməliyyatlar yerinə yetirir.

Lazım olan APK üzərində işləməyə və APK-nın əlaqə qurduğu C2 serverinin IP və portunu tapmağa başlayaq.

ApkTool-dan istifadə edərkən Resurs Cədvəlini yükləyərkən xətanın baş verdiyini gördüm. məndə *aapt* (Android Asset Packaging Tool) istifadə edərək icazələri və xidmətləri araşdıraraq təhlili davam etdirdim.

![Web 80 Index](9.webp){: width="800" height="550"}

Burada zərərli icazələri görürsünüz, bunları əlavə edən kod parçasını yuxarıda göstərdim.

![Web 80 Index](10.webp){: width="800" height="550"}

Accessibility Services-ə baxaq və görək zərərli APK Accessibility-dən istifadə edir?

![Web 80 Index](11.webp){: width="800" height="550"}

**AccessibilityService** istinadları zərərli proqram təminatının güclü göstəriciləridir.

![Web 80 Index](12.webp){: width="800" height="550"}

Gördüm ki, aapt ilə bir az vəziyyət çətinləşdi, mən də bu dəfə yenidən apktool-dan istifadə etmə qərarına gəldim, amma resursların deşifrə edilməsinin qarşısını almaq üçün -r bayrağı istifadə edərək.

Və apktool xətasız formada işlədikdən sonra, indi dekompilyasiya edilmiş folderi Windows-a ataq və oradan davam edək.


Android Studio istifadə edərək folderi açdım və C2-nin IP və portunu tapmaq üçün araşdırmağa başladım

![Web 80 Index](13.webp){: width="800" height="550"}

Və uzun bir araşdırmadan sonra IP və portun olduğu .smali faylını tapdım və səndemə IP və port base64 ilə şifrələnib. Gəlin indi onları deşifrə edək.

![Web 80 Index](14.webp){: width="800" height="550"}

Mən IP və portu buradan tapdım:

```xml
C:\Users\user\Desktop\ready\smali\kentucky\robert\icuheevgdgtlglnmkorsowcururfkkmpxnwvahibtwjulnfhep2\zbfncnpqdbiphruvjmmxueamnjcyu.smali
```
və mənə maraqlı gəldi ki, nəyə görə .smali faylı bu formada idi? Və araşdırmağa başladım.
 Döndük yenə başa, bu dəfə diqqətlə baxaq görək res folderinin içində apktooldan başqa lazımlı nələr var.

Və **apkeditor** ilə qarşılaşdım, Apk Editor güclü Android APK resursları redaktorudur, və mən də düşündüm ki, bu .smali faylının bu formada olmasının tək səbəbi varsa, o da budur.

![Web 80 Index](15.webp){: width="800" height="550"}

Yenə CraxsRatı decompile etdim və build namespacesinə gedib ApkEditoru axtarmağa başladım. Və bu kod parçası ilə qarşılaşdım, bu kod parçası bizim IP və portu tapmağımızı çətinləşdirdi.

![Web 80 Index](16.webp){: width="800" height="550"}


Gəlin, bu kod parçasına bir də GitHub-da apkeditorun rəsmi repositoriesinde baxaq.

![Web 80 Index](17.webp){: width="800" height="550"}

Qısacası, bu kod bizim işimizi çətinləşdirdi, amma biz yenə də IP və portu tapdıq.

## SON

Bu məqalədə CraxsRatın və onun yaratdığı zərərli APK faylını analiz edərək C2 serverinin IP və portunu tapmağa çalışdıq.

Oxuduğunuz üçün təşəkkür edirəm. Əgər hər hansı bir fikriniz olsa, və ya haradasa səhvlik etdiyimi düşünsəniz Instagram üzərindən mənimlə əlaqə keçə bilərsiniz.
