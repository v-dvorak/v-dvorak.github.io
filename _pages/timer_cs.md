---
permalink: projects/timer_cs/
title: "Timer"
author_profile: true
---
📅 27. 2. 2024

[🇬🇧](/projects/timer_en)

Jednoduchá knihovna pro časování v milisekundách, funguje na Atmega328P / Arduino UNO, s drobnými modifikacemi ji lze použít na různých hodinách mikrokontroleru a pro různá podobná AVR (úpravy lze provést podle dokumentace, link dole). K updatování času používá interrupty, může běžet bez přetečení skoro 50 dní.

Kód je dostupný na GitHubu:<br><br>
[![Readme Card](https://github-readme-stats.vercel.app/api/pin/?username=v-dvorak&repo=arduino-timer)](https://github.com/v-dvorak/arduino-timer)
{: .notice--info}

## Features

### Init

```cpp
void Init(void);
```

Inicializace, tuto funkci je potřeba zavolat jako úplně první.

### Get

```cpp
unsigned long Get(void);
```

Vrací čas v ms od inicializace.

### Reset

```cpp
void Reset(void);
```

Resetuje naměřený čas na nulu.

## Ukázka použití

```cpp
#include "timer.h"
#define DELAY 200 // v ms

Timer timer;
unsigned long lastReading;

int main(void)
{
    // setup timeru
    timer.Init();
    unsigned long lastReading = 0;
    unsigned long currentTime = 0;
    // povoleni interruptu
	sei();

	// hlavni loop
	while (1)
	{
        // cas prave ted
        currentTime = timer.Get();

        // pokud ubehlo dost casu
        if (currentTime - lastReading > DELAY) {
            /* 
            delej neco (treba blikej LED)
            */
           // reset
           lastReading = currentTime;
        }
    }
}

```

## Sources

- [Atmega328P Dokumentace](https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf)