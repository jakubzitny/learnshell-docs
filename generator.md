# PS1 generátor zadání pro LearnShell

## Instalace
- Naklonovat repozitář, nainstalovat pythoní requirements
- V `cfg.py` nastavit absolutní cestu:
  - `LIDL_ROOT`, defaultně `root tohoto repozitáře/lidl_compiler`
- Spustit aplikaci, která běží na Flasku.

## Formát dat
- Popsáno schématem v `app.py`
- Generování je založeno na jazyce [LI-DL](https://li-dl.readthedocs.io/en/latest/), v kódu je možno používat [Jinja2](https://jinja.palletsprojects.com/en/2.10.x/), kde je přístupné uživatelské jméno uživatele, pro kterého se úloha generuje

## Použití
- Slouží k vygenerování unikátního zadání pro každého studenta.
- Po proběhnutí programu se vezmou z tabulky symbolů všechny záznamy, které mají klíč splňující pythoní metodu `.islower()`, a spolu s hodnotami se vrátí jako vygenerovaná data

## Příklad

Tento kód

```
a = [1:4, LOWER_ASCII]
b = 10:50
C = "ahoj"
```

vrátí JSON, kde bude klíč `"a"` s hodnotou 1 až 4 malé znaky anglické abecedy a klíč `"b"` s hodnotou číslo od 10 do 50. `C` není `.islower()`, tedy nebude nijak zaznamenáno.

## Vylepšení oproti verzi ve starém LearnShellu
- V původním LearnShellu byla všechna zadání statická, a tedy tento generátor vůbec neexistoval.