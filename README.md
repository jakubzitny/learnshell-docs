# LearnShell v2.0

## Další části dokumentace

- [generator úloh](./generator.md)
- [vyhodnocovač úloh](./evaluator.md)
- [entity](./docs.pdf)

## Instalace
- Naklonovat repozitář, nainstalovat pythoní requirements
- Nainstalovat `redis`
- Nainstalovat, spustit a vytvořit databázi, je potřeba PostgreSQL
- Spustit Django aplikaci, pokud bude použit Django development server, je třeba parametr `--noreload`
- Spustit celery workery: `celery worker --app=settings.celery_setup`, možno nastavit loglevel pomocí `--loglevel=INFO`
- Pro správnou funkčnost KOSapi je třeba na [https://auth.fit.cvut.cz/manager](https://auth.fit.cvut.cz/manager) vyrobit KOSapi service application a údaje napsat do `settings/secrets.py`


## Vylepšení oproti původní verzi
- Zrušena pevná vazba mezi webem a opravovacím subsystémem, přidán generátor zadání, který umožňuje každému studentovi vytvořit unikátní zadání
- Web je nyní vyřešen pomocí GraphQL API, které umožňuje frontendovým klientům získávat přesně taková data, o která si řeknou
- Data o uživatelích je možno importovat přímo z KOSu
- Systém je použitelný i pro jiné věci než potřeby PS1

## Co systém umí
- Naimportovat data o uživatelích a předmětech z KOSu
- Přidávat a validovat externí služby pro generování zadání a opravu
- Vytvářet šablony úloh, zadávat úlohy, odevzdávat úlohy, ukládat data o výsledku opravy

## Co systém zatím neumí
- Režim testování ("fialový progtest"), třeba vyřešit zabezpečení
- Neexistuje frontendový klient (čeká na backend, do konce roku bude existovat)

## Přidání předmětu do systému

Pro přidání dalšího předmětu do systému je třeba naprogramovat externí službu, která bude opravovat odevzdané úlohy. Tato služba komunikuje přes HTTP a je třeba, aby uměla reagovat na následující požadavky:

- `GET {url}/ping`: vrací status code 200 nebo 204 a slouží k indikaci, zda služba žije a je nakonfigurována
- `GET {url}/schema`: vrací JSONschema dat, která služba očekává na vstupu při requestu popsaném v následující odrážce; systém následně bude podle tohoto schématu validovat data zadávaná uživateli
- `POST {url}`: přijímá JSON s daty splňující popsané schéma a vrátí JSON, který je výsledkem opravy odevzdání úlohy

Naprosto stejné schéma mají i služby pro generování zadání, akorát na `POST {url}` vrací JSON s vygenerovanými daty.

Je tedy úplně jedno, jaké technologie jsou použity při tvorbě služeb a co služby interně dělají, důležité je pouze splnit zadané rozhraní. Díky tomu je systém naprosto univerzální a použitelný (teoreticky) na cokoli.

Po vytvoření služby se do systému zadá URL služby, ten následně ověří, zda služba splňuje rozhraní uvedené výše, a pokud ano, službu přidá a označí jako použitelnou.

## Vytvoření šablony úlohy

Před vytvořením konkrétní úlohy je třeba vytvořit šablonu. Ta specifikuje, jaká služba slouží ke generování zadání a jaká služba bude úlohy opravovat. Ve službách jsou zároveň uloženy formáty dat, takže vlastně šablona zároveň specifikuje, jaká data bude muset zadat učitel při tvorbě nové úlohy a jaká data bude zadávat student při odevzdání.

## Vytvoření úlohy

Součástí úlohy je vždy zadání. Dále učitel zadá data pro generátor dat, ten potom vrátí proměnné, které jsou použitelné v zadání úlohy, a data pro opravovač (například popis testcasů, které budou použity pro ověření správnosti odevzdání).

## Odevzdání úlohy

Pro každého studenta se na základě dat zadaných učitelem vygeneruje zadání. Poté může úlohu odevzdat - zadá data ke kontrole splňující očekávaný formát (například bashový skript), toto se dá dohromady s daty od učitele (popis testcasů) a tento balík se pošle do opravovače.

## Vyhodnocení odevzdání

Opravovač úloh vrátí JSON, který se uloží do systému. Kromě bodového hodnocení může například obsahovat nápovědy, tedy co konkrétně bylo špatně apod.

## Správa uživatelů a předmětů

Systém si drží informace o předmětech, paralelkách, jejich studentech a učitelích. Tato data lze jak importovat z KOSu, tak spravovat ručně.
