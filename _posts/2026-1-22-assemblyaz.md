---
layout: post
pin: false
title: "0xFF: Assembly dili ilə öz xüsusi shellcode-umuzu yazaraq kernelə request göndərmək."
author: rafosw
categories: [AZ]
tags: [Reverse Engineering]
media_subpath: /images/assembly
image:
  path: Screenshot_8.png
---

## Assembly

Assembly dili CPU-nun birbaşa başa düşdüyü, yaddaş və registrlərlə aşağı səviyyədə işləyən, çox sürətli amma oxunması-yazılması çətin olan və əsasən OS, driver, və embedded sistemlərdə istifadə edilən proqramlaşdırma dilidir. Bu dil registrlər və RAM üzərində birbaşa işləməyə imkan verir, assembler tərəfindən machine code-a çevrilir və prosessor tərəfindən ardıcıl şəkildə icra olunur.

Assembly-də registrlər, çox sürətli işləyən və əməliyyatların icrası zamanı müvəqqəti məlumatların saxlanılması üçün istifadə olunan kiçik yaddaş sahələridir. Assembly dili bu registrlər üzərində birbaşa əməliyyat aparmağa imkan verir və bu da proqramların çox sürətli işləməsini təmin edir.

### Registry

![SS](Screenshot_1.png){: width="500" height="550"}

Yuxarıda 16, 32 və 64 bitlik arxitekturalara uyğun registrləri görürsünüz. Biz bu registrlərdən istifadə edərək kod yazacağıq. Amma Assembly dilində kod yazmaq bildiyimiz kimi çətindir, çünki birbaşa kernelə system call-lar göndərməli, CPU ilə sıx qarşılıqlı əlaqədə olmalı və hansı vəziyyətdə hansı registrdən nə məqsədlə istifadə edəcəyimizi dəqiq bilməliyik. Bu gün assembler olaraq NASM-dən istifadə edəcəyik və Linux Assembly register cədvəlinə əsaslanaraq kodlarımızı yazacağıq.

![SS](Screenshot_2.png){: width="777" height="650"}

İndi isə Linux-da mövcud olan system call cədvəlindən istifadə edərək sadə bir proqram yazacağıq. Bu proqram sys_time system call-undan istifadə edərək yalnız cari vaxt məlumatını əldə edəcək. ([Table Link](https://gist.github.com/GabriOliv/a9411fa771a1e5d94105cb05cbaebd21))

![SS](Screenshot_3.png){: width="577" height="650"}

1️ section .text

Bu, kod bölməsidir. CPU tərəfindən icra ediləcək əmrlər buradadır.

2️ global _start

Proqramın başlanğıc nöqtəsini təyin edir. Linux executable faylında _start burada başlayacaq.

3️ _start:

Kodun giriş nöqtəsi. Bütün əmrlər buradan başlayır.


![SS](Screenshot_4.png){: width="777" height="650"}

```bash
section .text
global _start

_start:
    mov eax, 13   ; eax = 13  sys_time syscall nömrəsi
    mov ebx, 0    ; ebx = tloc pointer NULL             Blok 1
    int 0x80      ; kernel syscall-u icra edir


    mov eax, 1    ; Linux x86 syscall table-dəki exit kodu
    mov ebx, 0    ;                                     Blok 2
    int 0x80
```
mov eax, 13 → kernel-ə deyirik hansı syscall-u icra etmək istəyirik. Buradakı 13, Linux x86 syscall table-də sys_time nömrəsidir. Yəni "mən cari Unix timestamp-ı istəyirəm".

mov ebx, 0 → sys_time-un arqumentini veririk. Syscall-un prototipinə görə time(time_t *tloc) bir pointer istəyir. Biz 0 (NULL) qoyuruq, yəni kernel nəticəni yalnız EAX register-də qaytarsın, yaddaşa yadakı ekrana yazmasın. Blok 2 kodu da eyni şəkildə proqramı dayandırmaq üçün exit əmri yerinə yetirir və bu əməliyyat Linux syscall cədvəlində 1ci yerdədir.

![SS](Screenshot_5.png){: width="577" height="650"}

```nasm -f elf32 test.asm -o test.o```
Bu command test.asm adlı Assembly faylını 32-bit ELF formatında obyekt fayla (test.o) çevirir. Yəni kod hələ icra olunmur, sadəcə maşın koduna çevrilmiş ara mərhələdir.

```ld -m elf_i386 test.o -o test```
Bu command obyekt faylı link edir və Linux-da icra oluna bilən proqram (test) yaradır. -m elf_i386 32-bit x86 arxitekturası üçün olduğunu bildirir.

./test
Bu command Assembly faylını icra edir, amma nəticədə heç nə görünmür, çünki mən demişdim ki, bu kod kernel-dən alınan nəticəni yalnız EAX register-də saxlasın, yaddaşa və ya ekrana yazmasın. İndi isə nəticəni ekrana çıxaran bir formasını yazacağıq.

```bash
section .data
buf db '0000000000', 10    ; 10 simvol üçün buffer + newline

section .text
global _start

_start:
    
    mov eax, 13       ; eax = 13 → sys_time syscall nömrəsi
    mov ebx, 0        ; ebx = tloc pointer NULL
    int 0x80          ; kernel syscall icra olunur, timestamp EAX-da qalır

    
    mov ecx, buf+9    ; pointer buffer sonuna
    mov ebx, 10       ; bölmə üçün divisor
convert_loop:
    xor edx, edx      ; EDX = 0
    div ebx           ; EAX / 10 → quotient EAX, remainder EDX
    add dl, '0'       ; remainder → ASCII simvol
    mov [ecx], dl     ; buffer-ə yaz
    dec ecx
    test eax, eax
    jnz convert_loop

    
    mov eax, 4        ; eax = 4 → sys_write syscall
    mov ebx, 1        ; ebx = 1 → stdout
    mov ecx, buf      ; buffer pointer
    mov edx, 11       ; uzunluq: 10 simvol + newline
    int 0x80          ; kernel-ə çağırış, ekrana yazılır

    
    mov eax, 1        ; eax = 1 → sys_exit
    mov ebx, 0        ; exit code = 0
    int 0x80
```

Bu kod bir az qarışıq görünə bilər, amma binary exploitlər və buffer overflow zəiflikləri ilə müqayisədə bu, olduqca sadədir.

![SS](Screenshot_6.png){: width="577" height="650"}

Bu kod bizə tarixi Unix timestamp formatında qaytarır və biz daha sonra date komandası ilə onu oxunaqlı tarix formatına çeviririk. İndi isə gəlin objdump istifadə edərək yazdığımız kodu tərsinə mühəndislik edib ekrana çıxaraq və grep regex komandalarının köməyi ilə ondan sadə bir shellcode düzəldək.

![SS](Screenshot_8.png){: width="577" height="650"}

```bash
objdump -d {file} | grep -E '^\s+[0-9a-f]+:' | cut -d$'\t' -f2 | tr -s ' ' | tr ' ' '\n' | grep -E '^[0-9a-f]{2}$' | xargs -I{} echo -n "\x{}"
```

Bu komandadan istifadə edərək shellcode-umuzu əldə etdik, amma bir problem var: bu shellcode işləməyəcək, çünki daxilində çoxlu x00 dəyərləri, yəni NULL byte-lar mövcuddur. Binary exploitation və digər exploit texnikalarında NULL byte (x00) arzuolunmazdır, çünki bir çox sistemlər məlumatı string kimi qəbul edir və string-lər x00 ilə qarşılaşdıqda oxumağı dayandırır. Indi isə NULL byte-dən qaçınmaq üçün bir-iki taktika göstərəcəm.

## Null Byte

1.
```bash
mov eax, 1        ; eax = 1 → sys_exit syscall nömrəsi
xor ebx, ebx      ; "mov ebx, 0" artıq istifadə etmirik NULL byte görə
int 0x80      ; kernel syscall-u icra edir
```

Buradaki XOR eyni bitləri sıfırlayan, fərqli bitləri 1 edən məntiqi əməliyyatdır və Assembly-də register-i sıfırlamaq üçün geniş istifadə olunur. xor reg, reg → register-i sıfırlamaq üçün ən sadə və effektiv üsuldurki burada NULL byte yaranmasın.

2.
```bash
xor eax, eax      ; eax = 0 (tam register sıfırlanır)
mov al, 1         ; al = 1 → eax artıq 1 olur (sys_exit)
xor ebx, ebx      ; ebx = 0 → exit code = 0
int 0x80          ; kernel syscall-u icra edir
```

EAX — x86 arxitekturasında 32-bit ümumi təyinatlı register-dir. Bu register-in içində daha kiçik hissələr var və onlara ayrıca müraciət etmək olur.

EAX → 32 bit (4 bayt)
AX → EAX-in aşağı 16 biti,
AH → AX-in yuxarı 8 biti (bit 8–15),
AL → AX-in aşağı 8 biti (bit 0–7),

![SS](Screenshot_9.png){: width="577" height="650"}

EAX-in alt hissəsi olan AL istifadə edilərək eyni exit(1) funksionallığını belə yuxardaki kimi yazaraqda NULL byte söhbətindən qurtulmaq olar, çünki yalnız 8 bit dəyişdirilir və bu zaman mov eax, 1 əmri zamanı yaranan NULL byte‑lardan qaçınmaq mümkün olur. Bu yanaşma aşağı səviyyəli proqramlaşdırmada və shellcode yazılarkən daha təhlükəsiz və səmərəli hesab olunur. Gəlin example olaraq NULL byte yaratmadan txt faylını oxuyub ekrana yazdıran kod yazaq.


```bash
section .text
    global _start

_start:
    ; ============================================================
    ; ADDIM 1: Fayl adını stack-ə yazmaq
    ; ============================================================
    ; "test.txt" sətirini stack-ə yerləşdiririk (tərsdən)
    ; Stack yuxarıdan aşağıyadır, ona görə tərsdən yazırıq
    
    xor eax, eax           ; EAX registrini sıfırla (null byte almaq üçün)
    push eax               ; Stack-ə 0x00000000 push et (sətrin sonu - null terminator)
    push 0x7478742e        ; Stack-ə ".txt" push et (hex formatda)
    push 0x74736574        ; Stack-ə "test" push et (hex formatda)
    mov ebx, esp           ; EBX-ə stack pointer-i köçür (indi EBX fayl adının ünvanını saxlayır)
    
    ; ============================================================
    ; ADDIM 2: Stack-də buffer üçün yer ayırmaq (1024 bayt)
    ; ============================================================
    ; Null byte olmadan 1024 ədədini yaratmalıyıq
    
    xor edx, edx           ; EDX-i sıfırla
    mov dl, 0xff           ; DL-ə 255 yaz (0xff = 255 decimal)
    inc edx                ; EDX-i artır: 255 + 1 = 256
    shl edx, 2             ; EDX-i 2 dəfə sola shift et: 256 * 4 = 1024
    sub esp, edx           ; Stack pointer-dən 1024 çıxar (stack-də yer ayır)
    mov edi, esp           ; EDI-yə stack pointer-i köçür (indi EDI buffer-in başlanğıc ünvanıdır)
    
    ; ============================================================
    ; ADDIM 3: Faylı açmaq - open() system call
    ; ============================================================
    ; System call nömrəsi: 5 (sys_open)
    ; Parametrlər: EBX = fayl adı, ECX = flag (0 = O_RDONLY - yalnız oxumaq üçün)
    
    xor eax, eax           ; EAX-i sıfırla
    mov al, 5              ; AL-ə 5 yaz (sys_open system call nömrəsi)
    xor ecx, ecx           ; ECX-i sıfırla (O_RDONLY = 0, yalnız oxumaq rejimi)
    int 0x80               ; Kernel-ə müraciət et (system call çağır)
    
    mov esi, eax           ; Qaytarılan file descriptor-u ESI-də saxla (sonra istifadə üçün)
    
    ; ============================================================
    ; ADDIM 4: Fayldan oxumaq - read() system call
    ; ============================================================
    ; System call nömrəsi: 3 (sys_read)
    ; Parametrlər: EBX = file descriptor, ECX = buffer ünvanı, EDX = oxunacaq bayt sayı
    
    xor eax, eax           ; EAX-i sıfırla
    mov al, 3              ; AL-ə 3 yaz (sys_read system call nömrəsi)
    mov ebx, esi           ; EBX-ə file descriptor-u köçür (hansı fayldan oxuyuruq)
    mov ecx, edi           ; ECX-ə buffer-in ünvanını köçür (oxunan məlumat hara yazılacaq)
    xor edx, edx           ; EDX-i sıfırla
    mov dl, 0xff           ; DL-ə 255 yaz
    inc edx                ; EDX-i artır: 255 + 1 = 256
    shl edx, 2             ; EDX-i 2 dəfə sola shift et: 256 * 4 = 1024 (oxunacaq bayt sayı)
    int 0x80               ; Kernel-ə müraciət et (fayldan oxu)
    
    mov edx, eax           ; Oxunan bayt sayını EDX-də saxla (write üçün lazım olacaq)
    
    ; ============================================================
    ; ADDIM 5: Ekrana yazmaq - write() system call
    ; ============================================================
    ; System call nömrəsi: 4 (sys_write)
    ; Parametrlər: EBX = file descriptor (1 = STDOUT), ECX = buffer, EDX = yazılacaq bayt sayı
    
    xor eax, eax           ; EAX-i sıfırla
    mov al, 4              ; AL-ə 4 yaz (sys_write system call nömrəsi)
    xor ebx, ebx           ; EBX-i sıfırla
    inc ebx                ; EBX-i artır: 0 + 1 = 1 (STDOUT - standart çıxış, ekran)
    mov ecx, edi           ; ECX-ə buffer-in ünvanını köçür (nəyi yazacağıq)
    int 0x80               ; Kernel-ə müraciət et (ekrana yaz)
    
    ; ============================================================
    ; ADDIM 6: Faylı bağlamaq - close() system call
    ; ============================================================
    ; System call nömrəsi: 6 (sys_close)
    ; Parametr: EBX = file descriptor
    
    xor eax, eax           ; EAX-i sıfırla
    mov al, 6              ; AL-ə 6 yaz (sys_close system call nömrəsi)
    mov ebx, esi           ; EBX-ə file descriptor-u köçür (hansı faylı bağlayırıq)
    int 0x80               ; Kernel-ə müraciət et (faylı bağla)
    
    ; ============================================================
    ; ADDIM 7: Proqramı bitirmək - exit() system call
    ; ============================================================
    ; System call nömrəsi: 1 (sys_exit)
    ; Parametr: EBX = exit code (0 = uğurla bitdi)
    
    xor eax, eax           ; EAX-i sıfırla
    inc eax                ; EAX-i artır: 0 + 1 = 1 (sys_exit system call nömrəsi)
    xor ebx, ebx           ; EBX-i sıfırla (exit code = 0, proqram uğurla başa çatdı)
    int 0x80               ; Kernel-ə müraciət et (proqramı bitir)
```

![SS](Screenshot_7.png){: width="577" height="650"}

Gördüyünüz kimi, bu kodda heç bir x00 dəyəri, yəni NULL byte yaranmadı. Bu shellcode-u artıq problemsiz şəkildə istifadə etmək mümkündür. Gəlin bunun üçün sadə bir C proqramı yazaq və shellcode-u işə salaq, sonra isə text.txt faylının oxunub ekrana yazdırılıb-yazdırılmadığını yoxlayaq.

```c
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>
#include <unistd.h>

unsigned char shellcode[] = 
"\x31\xc0\x50\x68\x2e\x74\x78\x74\x68\x74\x65\x73\x74\x89\xe3"
"\x31\xd2\xb2\xff\x42\xc1\xe2\x02\x29\xd4\x89\xe7\x31\xc0\xb0"
"\x05\x31\xc9\xcd\x80\x89\xc6\x31\xc0\xb0\x03\x89\xf3\x89\xf9"
"\x31\xd2\xb2\xff\x42\xc1\xe2\x02\xcd\x80\x89\xc2\x31\xc0\xb0"
"\x04\x31\xdb\x43\x89\xf9\xcd\x80\x31\xc0\xb0\x06\x89\xf3\xcd"
"\x80\x31\xc0\x40\x31\xdb\xcd\x80";

int main() {
    void *exec = mmap(0, 4096, PROT_READ | PROT_WRITE | PROT_EXEC,
                      MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    
    memcpy(exec, shellcode, sizeof(shellcode));
    
    ((void(*)())exec)();
    
    return 0;
}
```

![SS](Screenshot_10.png){: width="577" height="650"}

Bəli, artıq shellcode-umuz NULL byte-lar olmadan uğurla işləyir və txt faylının içini oxuyaraq bizə qaytarır. İndi isə sual yarana bilər: bəs necə olur ki, bəzi exploitlərdə və shellcode-larda x00 dəyərləri mövcud olur və bu zaman NULL byte problem yaratmır? Əslində, problem yaranmır, çünki həmin shellcode-lar yüksək səviyyəli proqramlaşdırma dillərində yazılır və sonradan assembly-yə kompilyasiya olunur. Bu mərhələdə yaranan NULL byte-lar exploit üçün kritik olmur. Amma biz bu kodu birbaşa Assembly dilində yazdığımız üçün NULL byte-lar bizim üçün çox kritik rol oynayır və shellcode-un işləməsinə mane ola bilər.

## Stack

Əlavə olaraq kodda bir məqama da toxunmaq istəyirəm. Diqqət etsəniz, biz test.txt fayl adını birbaşa Assembly kodunda string kimi yazmadıq

```bash
push 0x7478742e        
push 0x74736574 
```
onun əvəzinə tərsinə çevrilmiş hex dəyərlərindən istifadə etdik. Bunun əsas səbəbi Stack-dır. Birincisi, string-ləri birbaşa yazdıqda daxilində NULL byte (x00) və ya digər “bad character”-lər yarana bilər. İkincisi isə CPU-nun little-endian arxitekturaya malik olmasıdır, yəni çoxbaytlı dəyərlər yaddaşda tərsinə saxlanılır. Biz fayl adını hex formatında və tərsinə yazmaqla həm NULL byte probleminin qarşısını alırıq, həm də string-in yaddaşda düzgün şəkildə formalaşmasını təmin edirik. 

![SS](Screenshot_11.png){: width="577" height="650"}

Stack proqram işləyərkən müvəqqəti məlumatların saxlandığı yaddaş sahəsidir, LIFO prinsipi ilə işləyir yəni son daxil olan ilk çıxır, stack pointer (ESP/RSP) stack-in hazırkı üst hissəsini göstərir, stack adətən yuxarı ünvanlardan aşağıya doğru böyüyür, push dəyəri stack-ə əlavə edir, pop isə stack-dən çıxarır, funksiya çağırışlarında geri dönmə ünvanı və lokal dəyişənlər stack-də saxlanılır, stack sürətlidir amma ölçüsü məhduddur və əsasən qısaömürlü məlumatlar üçün istifadə olunur. Kodu cyberchefdə yenidən yazaq

![SS](Screenshot_12.png){: width="577" height="650"}

Və CyberChef-dən istifadə edərək stack-ə məlumatı necə düzgün şəkildə push edəcəyimizi də öyrənmiş olduq.

## SON

Bu məqalədə Assembly dili ilə birbaşa kernel sistem çağırışlarını (system calls) necə idarə edəcəyimizi, sadə shellcode-ların necə yazıldığını və ən əsası NULL byte probleminin necə həll olunduğunu öyrəndik. Aşağı səviyyəli proqramlaşdırma və exploit development sahəsində bu təməl biliklər olduqca vacibdir. Ümid edirəm hər şey aydın oldu.