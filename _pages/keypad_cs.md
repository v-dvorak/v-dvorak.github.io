---
permalink: projects/keypad/cs
title: "Keypad"
author_profile: true
---
📅 29. 2. 2024

[🇬🇧](/projects/keypad/en)

Propojení maticové klávesnice s přijímačem. Vypsané znaky jsou zobrazovány na LCD a odesílány přes USB. Při psaní lze přepínat mezi velkými a malými písmeny a čísly, zařízení podporuje mazání znaků.

Tento projekt je na GitHubu:<br><br>
[![Readme Card](https://github-readme-stats.vercel.app/api/pin/?username=v-dvorak&repo=arduino-keypad)](https://github.com/v-dvorak/arduino-keypad)
{: .notice--info}

## Motivace

Nejspíš jsme už všichni, alespoň v americkém filmu, někdy viděli stenografovací stroj - zařízení používáné soudními zapisovači pro maximální výkon. Uživatelé moderních počítačů jsou připoutáni na 60 až 100% klávesnice, kde se prsty dost nalétají.

Nebylo to tak ale vždycky. Kdysi dávno existovaly tlačítkové telefony a na pokrytí všech jejich tlačítek skoro stačilo deset prstů na rukou.
Když se zamyslíme více: klávesnice má rozměry 4x3, levé tlačítko s pravým na spodní řádce se používají jen zřídka, máme tedy 10 tlačítek, která opravdu potřebujeme. A ta zvládneme pokrýt prsty na rukou! A to neumí ani stenografovací stroj!

Cílem tohoto projektu je vrátit se o pár let nazpět a nastartovat revoluci v oblasti rychlého psaní za pomoci součástek za několika málo piv.

## Features

- zobrazení napsaných znaků na LCD a posílání přes USB
  - na LCD se zobrazují poslední dvě napsané řádky
  - znaky přes USB se odesílají po jednom
- při opakovaném mačkání se na místě kurzoru zobrazuje aktuálně vybrané písmenko
- nastavitelné mapování kláves (ve zdrojovém kódu)
- nastavitelné funkce makro tlačítek `A`, `B`, `C`, `D`, defaultně jsou nastavené následovně:
- přepínání mezi velkými, malými písmeny a čísly
  - upper/lower case pomocí `A`
  - čísla pomocí `B`
  - při přechodu z velkých písmen na čísla a z nich zpět na písmena zachovává velká písmena
- scrollování na LCD
  - pomocí `C` lze scollovat na displeji
  - scrollování se neprojevuje v odeslaných datech, není ekvivalentem `\n`, je čistě vizuální
- mazání znaků pomocí `D`
- LED svítí po dobu stisku libovolné klávesy

## Technikálie

Seznam komponent:

- deska Arduino UNO s Atmega328P - jádro zařízení, komunikuje s vnějšími zařízeními přes USB
- maticová klávesnice - slouží k ovládání zařízení
- LCD HD44780 16x2 - zobrazuje uživatelem napsaný text
- I2C převodník pro LCD - zajišťuje převod komunikace mezi deskou a LCD

## Schéma zapojení

LCD s I2C převodníkem jsou propojeni dle nákresu. Převodník je pak na Arduino UNO připojen skrz piny `A4`, `A5`, `5V`, `GND`. Klávesnice je připojena na piny `4-11`, a to hlavně kvůli jednodušší práci s načtenými výsledky: piny `4-7` odpovídají sloupcům, `8-11` řádkům.

![](/images/keypad_arduino_schematic.png)

## Klávesnice

`keypad.h`

Zajištuje čtení z klávesnice a dekóvání 8bitového čísla na souřadnice matice, data ukládá to struktury `Key{row, column}`.
Poskytuje ostatním komponentám informace o tom, která tlačítka byla stlačena a kolikrát.
Pamatuje si, ve kterém modu se zrovna nachází `lowerCase`, `upperCase`, `numbers` a podle toho vrací hodnoty znaků, je-li tázána.

### Nepříjemnosti

**Zapojení klávesnice**: je možné i do jiných pinů, než do těch v projektu. Ovšem je potřeba dávat si pozor: piny 0, 1 (`RX`, `TX`) nejsou standarní a nelze je v tomto případě použít. Při případné změně zapojení stačí provést drobné změny ve funkci `Scan()`. Orientace pinů vůči sobě je důležitá, jestliže je output ze skenu podivně pootočení, pak jsou otočené i piny.

**Čekání u skenování**: ve funkci skenu je vložený delay. Registry zřejmě potřebují trochu času, aby se do nich zapsané změny projevily. Vkládání funkce `nop` nepomohlo, nakonec jsem se smířil s delay 10\\(\mu \text{s}\\).

## LCD

`lcd.h`, `HD44780_PCF8574.h`

Zobrazuje znaky, které uživatel vypsal. Zvládá základní funkce jako je scrollování a mazání. Pokud se první řádek naplní, automaticky se text vypisuje do druhého, po zaplnění druhého řádku se druhý řádek přesune na místo prvního , vamže se a uvolní tak místo pro další psaní.

### Scrollování

Se děje buď automaticky po odeslání `\n` nebo přetečení řádku, nebo na žádost uživatele pomocí makro tlačítka. Posuny na LCD jsou do přijímače odesílány pouze v případě, že jde o znak nového řádku, všechno ostatní scrollovaní je pouze vizuální.

Pokud uživatel scrolluje tak, že by byl první řádek prázdný, kurzor se přesune do něj, aby se využil celý protor LCD. Vše komunikuje s klávesnicí.

Využití prázdného prostoru:

```
+---------------------------+  y
| ABCDEF...                 |  0
| GHIJKL... _               |  1
+---------------------------+

*scroll*, *scroll*

+---------------------------+  y
| _                         |  0
|                           |  1
+---------------------------+
```

### Maximální délka mazání

Kvůli technickému řešení LCD lze mazat jen znaky v právě aktivním řádku - jakmile je jednou znak "odscrollován", už se z LCD nedá smazat. Ovšem přes USB lze příkaz ke smazání poslat kolikrát jen budeme chtít. Pokud `CAP_DEL_DISPLAY_SIZE == true` v souboru `config.h`, pak je počet mazání přes USB omezen počtem smazatelných znaků na LCD.

```
+---------------------------+  y
| ...                       |  0
| ABCDEF _                  |  1
+---------------------------+
```
Na obrázku je právě 6 smazatelných znaků.

### Komunikace s LCD

O komunikaci mezi mikrontrolerem a LCD přes I2C se stará knihovna [**HD44780_PCF8574**](https://github.com/Matiasus/HD44780_PCF8574) od [**Matiasus**](https://github.com/Matiasus), na ni je postavena moje vlastní implementace potřebných funkcí ve tříde `LCD`.

## Timer

`timer.h`

Časovač v milisenkundách, ze kterého hlavní program odečítá čas mezi jednotlivými stisky. Běží na `TIMER0`, není těžké ho adaptovat na jiné timery mikrokontroleru.

Knihovna timeru je dostupná samostatně na [**GitHubu**](https://github.com/v-dvorak/timer).
{: .notice--info}

## Hlavní program

`main.cpp`, `config.h`

### 1) Načtení nového stisku

Na začátku hlavního loopu načte program stisklou klávesu. Poté zkontroluje, jestli je stisk validní (nějaká klávesa byla stisknuta) a zaznamená, že je klávesnice aktivní. V případě, že nic není stisknuto, opakuje pokus o načtení.

### 2) Vyhodnocení stisku

Program zkontroluje, jestli stisknutá klávesa odpovídá nějaké klávese akce. Pokud ano, upraví nastavení jednotlivých tříd tak, aby odpovídalo novým požadavkům. Pokud ne, pokračuje dále ve vyhodnocování. Veškeré informace o klávesách získává hlavní program pomocí funkcí `LastKeyValue()` a `CurrentKeyValue()`, ty volá z klávesnice.

Klávesnice zodpovídá za správné zpracování identifikátoru tlačítka, ať už se nachází v jakémkoliv módu (`lowerCase`, `upperCase`, `numbers`).

### 3) Vypsání znaku

Vypsání znaku může proběhnout z několika důvodů:

- uběhla doba `PRESS_DELAY` nastavená v `config.h`, po které zařízení už nečeká a znak odešle
-  stisklá klávesa obsahuje pouze jeden symbol (v defaultním nastavení je to třeba klávesa `#`, na kterou je namapovaný znak `\n`), není potřeba čekat, jestli uživatel z klávesy vybere jiný znak, protože na ní žádný jiný znak není  
-  byla stisknuta jiná klávesa, je potřeba zaobírat se novou klávesou a poslední vybraný znak se vypíše
-  bylo spuštěno makro, je potřeba přejít na nové nastavení a poslední vybraný znak se vypíše

Takováte přechody mezi stavy zajišťuje funkce `Flush()`.

Po tomto kroku se celý cyklus opakuje.

### Objektový návrh / objektový nákres

![](/images/keypad_program_schematic.png)

## Možná vylepšení

- Více módů klávesnice a zlepšení jejich indikace (například pomocí více různobarevných LED).
- Nastavení mapování kláves přímo za běhu zařízení. Vstup do menu, kde by uživatel mohl naklikat, jaká písmenka na klávesách chce.
- Tlačítko On/Off.
- Umístění zařízení do kompaktního obalu, krabičky cca 9x12 cm.
- Bluetooth, Wi-Fi připojení k počítači, pak bychom mohli rychle textovat opravdu odkudkoliv.
- Propojení s PC tak, aby bylo možné zařízení používat jako plnohodnotnou klávesnici. V současnou chvíli si její výstup pouze zobrazuji terminál, který moniituruje sériovou linku.
- Mód ovládání kurzoru myši. Klávesnice na to má tlačítek dost, dokonce bychom mohli podporovat pohyb po diagonále.

## Licence

**Nejsem právník**, zříkávám se jakékoliv odpovědnosti, ale!
{: .notice--danger}

Celý projekt je pod [GPL3](https://www.gnu.org/licenses/gpl-3.0.html) kvůli použité knihovně na ovládání LCD. Zbytek projektu berme jako knihovny, které jsem si napsal sám, a nechť jsou k použití pod licencí [MIT](https://choosealicense.com/licenses/mit/). Kdybyste s tím někdo náhodou udělal něco cool, napište mi, dík.