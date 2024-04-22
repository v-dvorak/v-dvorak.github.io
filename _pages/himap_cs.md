---
permalink: projects/himap_cs/
title: "HiMap"
author_profile: true
---
📅 22. 4. 2024

[🇬🇧](/projects/himap_en)

HiMap je konzolová aplikace, díky které lze stahovat Google Mapy s vlastním stylováním, s libovolným podporovaným přiblížením a v libovolné velikosti. Program je napsaný v Pythonu a spoléhá se na Google Maps Static API, skrz kterou si postupně žádá o části mapy a poté je pomocí knihovny `PIL` skládá to jednoho velkého obrázku.

Tento projekt je na GitHubu:<br><br>
[![Readme Card](https://github-readme-stats.vercel.app/api/pin/?username=v-dvorak&repo=HiMap)](https://github.com/v-dvorak/HiMap)
{: .notice--info}

## Motivace

Jednoho krásného pátečního odpoledne si sedím v knihovně a najednou mě můj kamarád Damián začne bombardovat naštvanými zprávami -
snaží se sestavit velkou mapu Prahy s vlastním stylováním. Používá Snazzy Maps, kde si nadefinoval vzhled, a postupně projíždí město posuvným čtverečkem, výřezy si stahuje a pak je v Gimpu skládá dohromady, v gridu větším než 30x30.

<figure>
    <img src="/images/maps_takhle_ne.jpg" alt="" />
    <figcaption>obludný grid v Gimpu</figcaption>
</figure>

Co se nestalo, Snazzy Maps, snad kvůli chybě v zaokrouhlování, neposouvají čtverec o tolik o kolik tvrdí. Kraje mu nenavazují, Damián se hroutí a do toho všeho lze stáhnout jen deset takových obrázků denně. Jednoduchou úvahou dojdeme k tomu, že poskládat celou mapu mu bude trvat víc jak čtvrt roku. Ale pouze v případě, že tenhle task někdo nezautomatizuje!

### Je čas automatizovat!

Každý, kdo si někdy chtěl pomocí jednoduchého skriptu usnadnit práci, pocítil poselství tohoto memu:

<p align="center">
<img src="/images/maps_meme.jpg" alt="" width="300"/>
</p>

Idea toho, že nám bude stačit pouze skládat již stažené obrázky se záhy rozpadla - manuálně jich postahovat stovky, i bez různých omezení, bude trvat opravdu dlouho, tak jsme se museli vydat přímo ke zdroji. Snazzy Maps totiž využívají Google Maps, které zobrazují s uživatelsky definovaným vzhledem.
Plán je jednoduchý: vezmu Google Mapy, vnutím jim Damiánův styl a stáhnu obrázek v požadovaném rozlišení.

Ejhle, k tomu je potřeba Google Maps Static API, nevadí, naučím se s ní pracovat a stáhnu si obrázek v požadovaném rozlišení.
Po asi hodině půl snahy (totiž, Google dokumentace má skvělou SEO, zatímco obsah už tak skvělý a užitečný není), kdy jsem Googlu podepsal souhlas s odebráním obou ledvin a párkrát někam zadal svoji kreditku, jsem konečně měl vlastní API klíč a "fungující" Python skript, který umí volat Maps Static API. No, tak teď už je to jednoduché - pošlu vlastní souřadnice, nadefinuji styl a stáhnu si obrázek v požadovaném rozlišení!

![](/images/maps_max_res.png)

A protože nikdo nechce používat mapu Prahy, která má rozlišení 640x640 pixelů, musíme stáhnout několik obrázků najednou a pak je poslepovat do jednoho celku, kde kraje navazují.

Stahování, cachování a podobná práce s Google Maps porušuje TOS, proto se zříkám jakékoliv odpovědnosti, která plyne z použití tohoto programu. Více na [Google Maps TOS](https://cloud.google.com/maps-platform/terms?_gl=1*1x7oou1*_ga*MjgzMTU4Njg3LjE3MTI5MjQ3ODQ.*_ga_NRWSTWS78N*MTcxMzUxMjEyOS40LjEuMTcxMzUxMjEzNS4wLjAuMA..#3.-license.), konkrétně odstavec `3.2.3 a)`.
{: .notice--danger}

## Features

Uživatel specifikuje souřadnice mapy (nebo jednu souřadnici a celkovou velikost), zoom level, styl mapy a zadá svůj API key. Výsledkem je mapa odpovídající požadavkům uložená v PNG.

Ke stylování lze použít [Snazzy Maps](https://snazzymaps.com/) a poté stáhnout styl jako `JavaScript Style Array` přímo ze SM editoru.

<figure>
    <img src="/images/maps_snazzy_showcase.png" alt="" />
    <figcaption>ukázka možných stylů ze <a href="https://snazzymaps.com/">Snazzy Maps</a></figcaption>
</figure>

Výsledná mapa má velikost `width*640 x height*614` px.

## Limitace

Bohužel Země není placatá a tak se musíme smířit s tím, že lineární aproximace v extrémních případech přestávají fungovat. Tento přístup se rozbije například okolo pólů, ale funguje perfektně pro země v Evropě.

## Instalace a použití

Skript se ovládá z příkazové řádky. Je potřeba mít nainstalovaný Python `3.11` a vyšší. Po naklonování repozitáře z GitHubu nejdříve založíme virtuální prostředí a nainstalujeme potřebné balíčky pomocí příkazů:

```bash
# zalozeni virtualniho prostredi:
python -m venv .venv

# spusteni prostredi:
# linux
source .venv/bin/activate
# windows cmd
.venv/Scripts/activate.bat
# windows PS
.venv/Scripts/Activate.ps1

# instalace balicku:
pip install -r requirements.txt
```

Na začátku nového příkazu by se nyní mělo objevit něco jako `(.venv)`, indikátor toho, že se nacházíme ve virtuálním prostředí.

Skript se používá následovně:

```bash
python3 -m app output_path volitelne_parametry
```

- `output_path`
  - výsledek bude k nalezení zde
- `--start X Y`
  - kde `X` a `Y` jsou zeměpisná šířka a délka ve stupních levého horního rohu mapy
  - `X` a `Y` jsou `float`
- `--end X Y`
  - kde X a Y jsou zeměpisná šířka a délka ve stupních pravého dolního rohu mapy
  - `X` a `Y` jsou `float`
- `--width X`
  - šířka mapy jako počet čtverečků, ze kterých se finální mapa skládá
  - `X` je `uint`
- `--height X`
  - height mapy jako počet čtverečků, ze kterých se finální mapa skládá
  - `X` je `uint`
  - tyto parametry lze použít k debugování (například pro kontrolu návaznosti kousků mapy: `--width 2 --height 2`)
- `-z X, --zoom X`
  - Google Maps zoom level, určuje zoom mapy
  - `X` je `unint`, `0 <= X <= 19`
- `--style path_to_style`
  - cesta k `JSON` se stylováním, které se použije na mapě
- `--save`
  - pokud je tento flag nastaven, tak se všechny části mapy, které v průběhu skript tahá z Google serveru, uloží do stejného adresáře jako finální mapa
- `--key api_key`
  - API key je potřebný pro komunikaci s Google serverem, každý uživatel má vlastní
- `--store`
  - uloží zadaný API key pro další použití
  - při dalším spuštění není potřeba ho zadávat, skript si ho sám načte z vytvořeného `TXT` souboru

Uložený API klíč není nijak šifrovaný, uloží se jako plain text do adresáře, ze kterého se spouští skript. Je tedy viditelný pro všechny, kteří mají přístup ke složce.
{: .notice--danger}

Protože je výrazně jednodušší mapu oříznout než zápasit s posunem souřadnic kvůli pár pixelům, má output padding kolem zadaných souřadnic 320 až 640 px. API klíč lze získat [zde](https://console.cloud.google.com/home/dashboard).

## Jak to funguje?

Pokud byl uživatelem definovám styl, skript si ho načte, a potom v začne postupně po řádcích zleva doprava posílat requesty na Google server, odkud si stahuje jednotlivé obrázky. Ty pak složí do jednoho velkého. Velikost stahovaných obrázků je 640x614 px, v dolní části totiž obrázek obsahuje logo Googlu a nápis `Map data © 2024`. Ani jednu z těchto věcí ve finální mapě nechceme, tak je prostě a jednoduše odřízneme.

### Posílání požadavků

Formát požadavku:

```
"https://maps.googleapis.com/maps/api/staticmap?center={center}&zoom={zoom}&size={size}&style={style_parameters}&key={api_key}"
```

Jelikož se požadavky posílají na server ve stupních zeměpisné šířky a délky a s reálnými mapami pracujeme v metrech/kilometrech, je potřeba nejdříve předpočítat o kolik stupňů se mají dotazy posouvat, když skládáme velkou mapu. V každém kroku se posuneme o nějaké stupně doprava a když seskládáme jednu řádku mapy, musíme se posunout na další - právě výpočet posunu zeměpisné délky je ta nejbolestivější část.

### Změna zeměpisné délky

Všechny konstanty jsou spočítány pro Prahu. Pro výpočet finálního posunu z.š. vezmeme defaultní hodnotu pro Prahu, tu přenásobíme poměrem "kolik kilometrů ujdu, když se posunu o 0.01 z.d. v Praze / v novém místě, pro které vytváříme mapu".

Kdybychom používali "pražskou" konstantu pro celou republiku, tak rychle zjistíme, že nám kusy mapy nenavazují, třeba v Rumburku je poměr vzdáleností asi 1.0166 (na jeden kilometr jsme mimo o víc jak 16 metrů), pro Břeclav na jihu Moravy je ochylka asi 30 metrů.

### Zoom

Změna souřadnic po osách se ještě vynásobí poměrem mezi defaultním zoomem `16` a zadaným. Čísla pro tento výpočet mám z [Stack Exchange](https://gis.stackexchange.com/a/7443).

### Skládání mapy

Pomocí knihovny `PIL` se založí prázdný canvas, do kterého se postupně vkládají stažené obrázky. Pokud je zvolena možnost `--save`, tak se části mapy uloží do adresáře společně s finální mapou, jinak si program postupně žádá o obrázky a rovnou je skládá.

![](/images/map_gen.gif)