# PS1 opravovač úloh pro LearnShell

## Instalace
- Naklonovat repozitář, nainstalovat pythoní requirements
- Vytvořit v systému uživatele, kteří budou spouštět opravovací příkazy:
  - Uživatelské jméno musí mít formát `<prefix>stud<id>` resp. `<prefix>ref<id>`, kde `<prefix>` je defaultně `user`, prostřední část určuje, zda slouží ke spouštění studentského či referenčního řešení, a `id` je od 0 do počtu subsystémů (defaultně jsou 4 subsystémy, tedy od 0 do 3). Defaultně bude tedy 8 uživatelů: `userstud0` až `userref3`.
  - Na UID ani GID nezáleží, může však být výhodné mít všechny tyto uživatele v jedné skupině, viz dále.
  - Aby byl systém chráněný proti fork bombě a přílišné spotřebě paměti, je dobré do `/etc/secutiry/limits.conf` přidat těmto uživatelům (či jejich skupině) hard limit na `nproc` a na `as`.
  - **Ochranu proti přílišné spotřebě diskového prostoru je třeba vyřešit**. Naivně se nabízí udělat diskové oddíly, ale takové řešení je nedockerizovatelné.
- Nainstalovat `redis`.
- V `cfg.py` nastavit absolutní cesty:
  - `LIDL_COMPILER_PATH`, defaultně `root tohoto repozitáře/lidl_compiler`
  - `SYSTEM_TAR_PATH`, defaultně `root tohoto repozitáře/fs.tar`
  - `BASE_WORKDIR_PATH`, adresář, ve kterém se bude provádět oprava. Musí patřit stejnému uživateli, který provozuje webovou aplikaci a celery, a musí existovat právo na čtení a zápis.
  - `REDIS_*`, možná bude potřeba upravit port redisu či číslo databáze. Obě databáze musejí být rozdílné.
  - Dále zkontrolovat, zda jsou všechna UID v zadaném rozsahu volná.
- Spustit aplikaci, která běží na Flasku.
- Spustit celery workery: `celery worker --app=tasks`, možno nastavit loglevel pomocí `--loglevel=INFO`


## Formát dat
- Popsáno schématem v `app.py`
- `solution` a v každém testcase `name`, `description` a `lidl` jsou před zpracováním renderovány jako [Jinja2 templates](https://jinja.palletsprojects.com/en/2.10.x/) a jsou plněny daty, které přijdou v requestu.
- Dokumentace jazyka `LI-DL` je [zde](https://li-dl.readthedocs.io/en/latest/).


## Technické řešení
- Chroot bez přimountovaných `/proc` a `/dev`. 
- Zařízení `/dev/stdout`, `/dev/stderr` a `/dev/null` jsou všechny přítomné a všechny fungují jako standardní `/dev/null`.
- Adresář `/tmp` má oprávnění 777 a jeho obsah není kontrolován.
- Ostatní systémové soubory mají standardní vlastníky i oprávnění.
- Kontroluje se obsah domovského adresáře, standardní výstupy a návratový kód.


## Pasti
- Přestože je kladen důraz na to, aby prostředí pro studentovo a referenční řešení bylo co nejpodobnější, jsou zde následující omezení, na které je potřeba dávat pozor:
  - Uživatelé spouštějící skripty mají různá UID. To je naprosto zásadní, protože jinak by se navzájem přetahovali o procesový limit a studentova fork bomba by tak mohla shodit referenční řešení. Studentovo UID je přístupné pod Jinja2 proměnnou `{{ uid }}`.
  - Nevypisovat soubory, kde se vyskytuje UID aktuálního uživatele. Důvod viz předchozí odrážka.
  - Nepoužívat čas: je systémový a nevíme, kdy co zrovna poběží.
  - Nepoužívat čísla inodů: ty jsou v linuxovém systému unikátní a učitelský a studentský soubor tak určitě nebudou mít stejné číslo inodu.
  - Nepoužívat další nedeterministické věci jako například `$RANDOM` apod.


## Vylepšení oproti verzi ve starém LearnShellu
- Možnost parametrizace testovacích dat (přístup k vygenerovaným datům úlohy, uživatelskému jménu testovaného studenta a další)
- Náhodně generované uživatelské jméno a UID (předtím bylo obojí statické)
- Opravena chyba, kdy se služba mohla při paralelní opravě více skriptů dostat do deadlocku


## Známé chyby a TODOs
- Symlinky se kontrolují na přesnou shodu (toto bylo i ve starém LearnShellu), nikoli zda ukazují na stejný soubor
- Nelze generovat adresářové struktury (problém jazyka LI-DL)
- Není vyřešena ochrana proti nadměrnému užívání diskového prostoru.