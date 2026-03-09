---
layout: post
pin: false
title: "Tetrix: Oyun Tərsinə Mühəndisliyi"
author: rafosw
categories: [AZ]
tags: [Reverse Engineering, CTF]
media_subpath: /images/tetrix_game
image:
  path: Screenshot_4.png
---

Bu CTF Hackfinity Battle 2025 CTF-nin bir hissəsidir və oyun hackingini ehtiva edir

Açıqlama: PE daxilində gizli bir flag var və skor 999999u keçəndə flagı görə bilirik.

## Analiz

Gördüyünüz kimi, oyun çox sadədir və belə görünür.

![SS](Screenshot_1.png){: width="330" height="550"}

Mənim diqqətimi çəkən, oyuna girərkən bir anlıq görünən bu hissə oldu.

![SS](Screenshot_2.png){: width="330" height="550"}

Birazdan bura gələcəm indilik Cheat Enginede bu oyunu açaq, Cheat Engine oyunlarda memory üzərində dəyişiklikləri analiz etmək üçün yaradılmış bir proqramdır. Adətən offline PC oyunlarında istifadə olunur.

Oyunun prosesini Cheat Enginedə seçirik.

![SS](Screenshot_3.png){: width="700" height="550"}

Ardından skor səviyyəsini Value hissəsinə yazırıq və gördüyünüz kimi oyundaki dəyərin adresini görürük, burada dəyişiklik edib skoru 0 edsək flagı əldə edə bilərik amma Cheat Engine bugünün mövzusu deyil

![SS](Screenshot_4.png){: width="700" height="550"}

Başda demişdimki oyuna girərkən bir anlıq görünən bu hissə mənim diqqətimi çəkdi burada ekranda "GODOT Game engine" yazır.

![SS](Screenshot_2.png){: width="330" height="250"}

internetə bunu yazanda isə bizi GODOT adındaki oyun mühərriki qarşılayır və biz bilmiş oluruq ki, bu oyun bu mühərrik ilə hazırlanıb.

![SS](Screenshot_5.png){: width="700" height="550"}

Oyun mühərrikini yüklədim və oyunu burada açmaqa çalışdım.

### Godot RE Tool

![SS](Screenshot_6.png){: width="700" height="550"}

Burada qarşıma error cıxdı məndən fərqli və zip formatında fayl istəyirdi və məndə internetdə araştırma etməyə başladım və diqqətimi GDRE adında proqram çəkdi.

![SS](Screenshot_7.png){: width="700" height="550"}

Bu proqram isə godot ilə yazılmış oyunları dekomplanasiya etmək üçündür

![SS](Screenshot_8.png){: width="700" height="550"}

gəlin bu proqramı yükləyək və oyunumuzu proqram içində açaq

![SS](Screenshot_9.png){: width="700" height="550"}

GDRE bizə proqramın source kodunu cıxartdı və biz indi bu folderi birbaşa mühərrikdə aça bilərik

![SS](Screenshot_10.png){: width="700" height="550"}

Artıq biz oyunumuzun bütün soruce kodunu və dizaynını görmüş oluruq.

indi isə flagı tapmaq üçün skor kodunu source kodda tapmaqa çalışaq

![SS](Screenshot_11.png){: width="700" height="550"}

Burada skorun valuesini yəni dəyərini search edirik.

![SS](Screenshot_12.png){: width="700" height="550"}

və bu dəyərin yerləşdiyi kod blokunu görmüş oluruq.

![SS](Screenshot_13.png){: width="700" height="}

və bu kod blokunda məlum dəyərimizi 0 edib oyunumuzu debug etdikdə flagı əldə etmiş olacayıq.

![SS](Screenshot_14.png){: width="700" height="}

## SON 

Ümid edirəm ki, oyunu necə cracklayıb flagı tapmağı tam izah edə bilmişəmdir. Bütün bunlarla məşğul olmaq istəmirsinizsə, string commandından istifadə edərək birbaşa Linux-da bayrağı tapa bilərsiniz d.
