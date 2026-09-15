---
layout: post
title: "Add Boot Driver for DualBoot Ubuntu and MacOS with OpenCore"
date: 2026-07-27 15:08:17 +0000
tags: ["boot", "Broadcom", "dualboot", "MacBookAir", "macOS", "OCLP", "Sonoma", "Ubuntu", "UEFI", "Wi-Fi"]
blogger_orig_link: https://lexsysko.blogspot.com/2026/07/add-boot-driver-for-dualboot-ubuntu-and_0474436225.html
---

### Передісторія

У доньки є старенький MacBook Air моделі 7.2 2017 року, який
вона використовує для простих завдань.   
З часом я вже ремонтував його,
замінивши SSD на Samsung M.2 EVO 970Plus 500 ГБ через   
перехідну плату. Це остання модель MacBook Air котра дозволяла це робити.   
Тож тепер місця
стало значно більше, ніж на рідному диску обсягом 120 ГБ.  

[![](/assets/images/blog/f83ca6f4751b0bd0-dac241530bd3e4f4.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjnS3ReOVMtwoccb2wJGJG_RYIiDh9lHOlopP6Y1DhB5GjhS0tpxxhgN1ociRu3QzpTakB1C5Mm8OCfHPBIDq4CR0ON76RKR9tiftfWtHa5fu34On9JNckIgStxC_9GCed-ghG9iwhD56ZIBzgyQySeeio8fJjFpFmPXEo9KS6F1A1UjR3mEq41wwW9kvk/s4000/20231016_173056.jpg)  
*SSD Samsung M.2 черезперехідну плату*

Модель MacBook
Air 7.2 2017 року це:

* процесор: Intel i5
* оперативної пам'яті: 8 ГБ
* відеоадаптер: Intel HD 6000
* Wi-Fi адаптер:  Broadcom BCM4360 802.11ac Dual Band Wireless
  [14e4:43a0] (rev 03)

Під час заміни диска я також встановив за допомогою "OpenCore Legacy Patcher" версію macOS "Sonoma" з доданими усіма відсутніми драйверами. Але забула вона, що оновлювати її більше не можна, і
нове оновлення macOS стерло драйвери Wi-Fi адаптера, тож система залишилася
без Wi-Fi, адже він, Broadcom BCM4360, не підтримується у macOS "Sonoma".   
І вона вимушена була відновила стару
версію macOS "Mojave" що була при виробництві через Recovery tools.

І ось, коли донька приїхала в гості, мені знову довелося вирішувати ту ж задачу з "OpenCore Legacy Patcher" та macOS "Sonoma", але цього разу я вирішив встановити ще й альтернативну операційну систему - Ubuntu 26.04 LTS.

### Встановлення операційних систем

Встановив macOS "Sonoma" з нуля через USB-інсталяцію, створену за допомогою [OpenCore Legacy Patcher GUI tools](https://dortania.github.io/OpenCore-Legacy-Patcher).

Потім розділив диск 512 ГБ на 60 ГБ у форматі FAT для Ext4 та 8 ГБ FAT для Swap. На 60 ГБ розділ встановив Ubuntu 26.04 LTS як "/", а на 8 ГБ розділ - "Swap".

Проте Wi-Fi (BCM4360) не запрацював, тож я роздав інтернет через Bluetooth зі свого мобільного телефону, щоб оновити систему та встановити потрібне.

[![](/assets/images/blog/abb9dd2c747843d3-714187b445a1f555.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj4Ya3WWWEXAWBq3b2ZdjwXAK3DNWpDT1Z0jnROxpZDL1XWKM_9XCrJhNiAv4byn9SwFDTmdDfFaO28cN4_iRyWNDdWKk1iEk8EqBut4C-PfKcXgHPV0Dmm4GFyRT0-TSd2hdx-NvpL5ggHEd7WuTUsG1aHYBJ2Y6IZfAZm4jSb5CVBHmvKNPqV4AKbVzo/s4000/20260725_214243.jpg)  
*Bluetooth з мобільного телефону в Ubuntu*

  

Для відновлення роботи Wi-Fi встановлюю *broadcom-sta-dkms*:

```
sudo apt install broadcom-sta-dkms
```

Все запрацювало після перезавантаження.

### Подвійне завантаження систем macOS та Ubuntu.

EFI boot розділ один і основний для OCLP. Тому для того щоб не затиралася інформація про Ubuntu Grub EFI loader я змінив файл "*config.plist*" з завантажувача OCLP додавши драйвер файлової системи Ext4 - "[OpenCore-1.0.7-RELEASE/X64/EFI/OC/Drivers/Ext4Dxe.efi](https://github.com/acidanthera/OpenCorePkg/releases)".   
  
Я створив простий BASH-скрипт для macOS під назвою: [mount\_and\_fix\_oc.sh](https://gist.github.com/lexsysko/acc2e1d891e5869c485fae82a51fa4f9) , який монтує EFI Volume, копіює файл "*Ext4Dxe.efi*" у теку "*/EFI/EFI/OC/Drivers*" та за допомогою "*/usr/libexec/PlistBuddy*" додає його в розділ "*:UEFI:Drivers:0*" файлу "*config.plist*".  
  
Результат завантаження системи Ubuntu and MacOS:

* [Video: Dual boot MacBook Air 7.2 (2017) Core Legacy (Sonoma) and Ubuntu 26.04 LTS](https://youtu.be/jOcvtdfqfYw)
