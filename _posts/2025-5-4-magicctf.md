---
layout: post
pin: false
title: "magic CTF: Özümün hazırladığı CTF"
author: rafosw
categories: [AZ]
tags: [CTF]
media_subpath: /images/ctf_magicctf
image:
  path: https://miro.medium.com/v2/resize:fit:1100/format:webp/1*jm926iNSpSEejktoyYKj1w.png
---

Hal-hazırda sizə özümün hazırladığı CTF-i göstərirəm.

Yükləmə linki: [Download Link](https://mega.nz/file/pukTSD4T#HHmYpC8flwiw2mx81zmiGiCCyIPzZBMfD5h9L3Bx73U)

Aşağıda CTF-in həll yolu, yəni write-up-ı var

Əlimizdə bir ədəd .exe PE faylı var, flaqı buradan çıxarmalıyıq.

![SS](1_0nYw-XQOxMDbVApLpczoIQ.webp){: width="330" height="550"}

Faylı Detect It Easy (DIE) proqramı ilə analiz edərək onun tam növünü və hansı proqramlaşdırma dilində yazıldığını müəyyən edə bilərik

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*9QE8jnDtYJ4-iZEzwrkf9Q.png){: width="700" height="550"}

Amma DIE bizə qəribə bir çıxdı verdi burada faylın MS-DOS formatında olduğu görünür və əlavə olaraq heç bir proqramlaşdırma dili və ya digər məlumat təqdim olunmayıb.

Digər exe fayllarda DIE-nin verdiyi çıxış isə belə olur.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*M5GADfe49mQDL-J-SZowMw.png){: width="700" height="550"}

Nəysə, faylın MS-DOS olduğunu gördük, yəni bu bir PE faylıdır. Gəlin, bunu runlayaq, görək nə olur.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*mYboHOgPHylNNOir9JZRnA.png){: width="700" height="550"}

Belə bir xəta aldım, exe fayli acilmir.

Artıq EXE faylını hex editor-da açıb içini görmək vaxtıdır.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*o2P1RjyYWuQvnZFYuljujg.png){: width="700" height="550"}

Burada görürük ki, əslində bu bir EXE yox, bir PCAP faylıdır.

İndi isə EXE uzantısını PCAP ilə dəyişək və faylı açmağa cəhd edək.

![SS](https://miro.medium.com/v2/resize:fit:450/format:webp/1*odSBxt8A9_o5ew2BZlCJ5w.png){: width="330" height="550"}

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*dHXPFfK4BJrBASNQc-3ZNQ.png){: width="700" height="550"}

Burada bir qəribəlik olduğunu görürük. Faylın bir PCAP faylı olduğunu bildiyimiz halda, wireshark bizə “fayl formatı xətası” verdi. Yenidən hex editor-a qayıdaq.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*0_ifQO7iiNwScbPDJ4qv0A.png){: width="700" height="550"}

Burada görürük ki, faylın headerinde 4D 5A yazilib , yəni MS-DOS kimi qeyd olunub. Bu da Wireshark’ın bu faylı PCAP yox, hələ də EXE kimi başa düşməsinə səbəb olur və buna görə də error alırıq.

Gəlin, indi PCAP faylının header hex codesini öyrənmək üçün fayl signature listinə nəzər salaq.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*3DdEO0wOdCwOJY0IrVpJtQ.png){: width="700" height="550"}

Buradakı hex kodlarnan bizim fayldakı hex kodunu uyğunlaşdırmağa çalışaq.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*jm926iNSpSEejktoyYKj1w.png){: width="700" height="550"}

Burada görürük ki, bizim fayldakı ilk 4 kod dəyişdirilib və yerinə 4D 5A, yəni MS-DOS-un ilk 4 hex kodu yazılıb. Buna görə də proqramlar bizim faylı xətalı və ya bir EXE faylı kimi görür.

indi isə, düzəlişlər edək.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*14oY4EKtDf-prCmSDxo0DA.png){: width="700" height="550"}

Artıq faylın header-i ilə pcapng formatını uyğunlaşdırdığımıza görə, faylımız problemsiz şəkildə Wireshark-da açılacaq. Bu arada görürük ki, faylımız tam olaraq bir PCAP deyil, bir PCAPNG faylıdır. Bu da o deməkdir ki, fayl Wireshark və ya yeni nəsil şəbəkə trafiki analiz proqramı tərəfindən dump olunub.

![SS](https://miro.medium.com/v2/resize:fit:626/format:webp/1*YQQix_XiNhfuWTttmO7ETg.png){: width="330" height="550"}

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*QmfLmr24poXB3QqJVyvanA.png){: width="700" height="550"}

PCAPNG faylımız xətasız formada Wireshark-da açıldı. İndi isə gəlin flaqı tapmağa çalışaq.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*hfz840xzb55M1-BVZ4rcuA.png){: width="700" height="550"}

Şəbəkə trafikinə baxdım, amma maraqlı bir şey yoxdur. Əlimizdə olan yeganə şey HTTP sorğularıdır. Gəlin, onlara baxmaq üçün filterdən istifadə edək.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*Zk1c9P0zrso88eg0hsYlPg.png){: width="700" height="550"}

Burada HTTP-yə aid çoxlu GET və POST metodları var. Bizə lazım olan POST metodudur, çünki istifadəçi sayta nəsə göndərəndə POST metodundan istifadə edir.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*S1rGouZQJE_aqP-AJBTTyg.png){: width="700" height="550"}

Lazım olan metodu tapmaq üçün uyğun filtr tətbiq etdik və burada istifadəçinin sayta ad və kod verdiyini görürük. Verilən kod Base64 ilə şifrələnib. Gəlin, bu şifrəni decode edək.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*9AAjWBUljn_NHYjq2MEXiQ.png){: width="700" height="550"}

Və flaqı tapırıq! ✅