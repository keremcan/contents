---
title: "STM32'de 'startup.s' Dosyasının Önemi ve Çalışma Prensibi"
date: "2025-02-18"
category: "Gömülü Yazılım"
tags: ["STM32", "Gömülü", "Yazılım", "Embedded"]
sum: "STM32 mikrodenetleyicileri ile çalışırken, donanımın ilk aşamada nasıl çalıştığını anlamak büyük önem taşır. Sistemin açılışında devreye giren ilk kodlardan biri startup.s dosyasıdır. Bu dosya, mikrodenetleyicinin başlangıç sürecini yönetir, kesme vektör tablosunu hazırlar ve sonunda main() fonksiyonuna geçiş yapar.Bu yazıda, startup.s dosyasının rolünü, içeriğini ve önemini detaylı şekilde ele alacağız."
author: "Kerem Can"
---

# STM32'de  `startup.s`  Dosyasının Önemi ve Çalışma Prensibi

STM32 mikrodenetleyicileri ile çalışırken, donanımın ilk aşamada nasıl çalıştığını anlamak büyük önem taşır. Sistemin açılışında devreye giren ilk kodlardan biri  **`startup.s`**  dosyasıdır. Bu dosya, mikrodenetleyicinin başlangıç sürecini yönetir, kesme vektör tablosunu hazırlar ve sonunda  **main() fonksiyonuna**  geçiş yapar.

Bu yazıda,  **`startup.s`**  dosyasının rolünü, içeriğini ve önemini detaylı şekilde ele alacağız.



## 1.  `startup.s`  Nedir ve Neden Önemlidir?

`startup.s`,  **STM32 mikrodenetleyicisinin başlatılmasını yöneten assembly dilinde yazılmış bir dosyadır**.

Bu dosyanın temel görevleri şunlardır:

-   **Yığın işaretçisini (Stack Pointer - SP) ayarlamak**
-   **Program sayacını (Program Counter - PC) Reset Handler’a yönlendirmek**
-   **Kesme vektör tablosunu oluşturmak**
-   **BSS ve DATA bölümlerini başlatmak**
-   **Ana program olan  `main()`  fonksiyonunu çalıştırmak**

Bu işlemler olmadan, STM32 mikrodenetleyiciniz herhangi bir işlem yapamaz. Yani  `startup.s`, işlemcinin ilk komutlarını çalıştırarak sistemi başlatan temel bileşendir.


## 2.  `startup.s`  Dosyasının İçeriği

### **2.1. Genel Tanımlamalar**

Başlangıçta kullanılan temel ayarlar şu şekildedir:

```assembly
.syntax unified
.cpu cortex-m7
.fpu softvfp
.thumb
```

-   **`.cpu cortex-m7`**  → STM32H743'te kullanılan  **Cortex-M7**  işlemcisi için tanımlanmıştır.
-   **`.fpu softvfp`**  → Yazılım tabanlı kayan nokta birimi kullanılır.
-   **`.thumb`**  → Thumb komut setinin kullanıldığını belirtir.

----------
### **2.2. Vektör Tablosu**

```assembly
.section .isr_vector,"a",%progbits
g_pfnVectors:
  .word  _estack
  .word  Reset_Handler
  .word  NMI_Handler
  .word  HardFault_Handler
  .word  MemManage_Handler
  .word  BusFault_Handler
  .word  UsageFault_Handler
```

-   **Vektör tablosu**, kesme işlemleri için ISR (**Interrupt Service Routine**) adreslerini içerir.
-   İlk giriş  **Stack Pointer (_estack)**  değerini tutar.
-   Reset durumunda, işlemci  **`Reset_Handler`**  fonksiyonunu çağırır.
-   NMI, HardFault, BusFault gibi sistem hataları için ilgili fonksiyonlar atanır.

----------

### **2.3. Reset Handler (Başlatma İşlemi)**

```assembly
.section .text.Reset_Handler
.weak  Reset_Handler
.type  Reset_Handler, %function

Reset_Handler:
  ldr   sp, =_estack      /* Stack Pointer'ı yükle */
  bl  SystemInit          /* Sistem saatini başlat */
  bl  main               /* Ana programı çalıştır */
  bx  lr                 /* Dönüş */

```

-   **Reset gerçekleştiğinde çalışan ilk kod burasıdır**.
-   **Stack Pointer (SP)**  `_estack`  değerine ayarlanır.
-   **Saat yapılandırması (`SystemInit`) başlatılır**.
-   **`main()`  fonksiyonu çağrılarak kullanıcı kodu çalıştırılır**.

----------

### **2.4.  `.data`  ve  `.bss`  Bölümlerinin Başlatılması**

```assembly
  ldr r0, =_sdata
  ldr r1, =_edata
  ldr r2, =_sidata
  movs r3, #0
  b LoopCopyDataInit

CopyDataInit:
  ldr r4, [r2, r3]
  str r4, [r0, r3]
  adds r3, r3, #4

```

-   **.data segmenti (RAM’de değiştirilebilir veri) Flash’tan RAM’e kopyalanır**.
-   **.bss segmenti (başlatılmamış değişkenler) sıfırlanır**.

----------

### **2.5. Varsayılan Kesme İşleyicisi (`Default_Handler`)**

```assembly
.section .text.Default_Handler,"ax",%progbits
Default_Handler:
Infinite_Loop:
  b  Infinite_Loop

```

-   Eğer bir kesme tanımlanmamışsa,  **sonsuz döngüye girerek sistemin çökmesini önler**.
-   **Debugging**  için faydalıdır, böylece beklenmedik kesmeler sistemin davranışını bozmaz.



## 3.  `startup.s`  Dosyasının Önemi

`startup.s`, mikrodenetleyicinin açılış sürecinde kritik bir rol oynar:

✅  **Donanımı başlatır**  → Bellek bölgelerini ayarlar.

✅  **Kesme yönetimini sağlar**  → ISR tablolarını düzenler.

✅  **Kullanıcı kodunu çalıştırır**  →  `main()`  fonksiyonuna geçiş yapar.

STM32 mikrodenetleyicileri ile çalışırken  **boot sürecini anlamak**, sistemin daha iyi optimize edilmesine yardımcı olur. Özellikle  `startup.s`  üzerinde değişiklik yapmak gerekirse, yukarıdaki yapıların bilinmesi büyük avantaj sağlar.



## 4. Sonuç

STM32 projelerinde  **`startup.s`**, donanımın ilk açılış işlemlerini yönetir ve  **kesintisiz bir sistem başlangıcı sağlar**. Vektör tablosunun yönetimi, bellek yapılandırması ve başlangıç kodlarının çalıştırılması gibi görevleri üstlenir.

Eğer  **STM32'nin nasıl çalıştığını anlamak istiyorsanız**,  `startup.s`  kodlarını analiz etmek ve işleyişini öğrenmek kritik bir adımdır. Böylece  **boot sürecini optimize edebilir, hataları daha hızlı tespit edebilir ve mikrodenetleyicinin başlatılmasını daha iyi kontrol edebilirsiniz**.

