# Audit zdrojov – fáza 1

Stav k 15. 10. 2026. Zberač zatiaľ nie je postavený; tento dokument slúži na rozhodnutie, ktoré zdroje a kanály pôjdu do `sources.yaml`.

## Ako čítať tabuľky

**Funguje:**
- **áno** – feed alebo stránka načítaná naživo, dáta aktuálne.
- **áno (stará kópia)** – kanál funguje a má správny formát, ale nástroj na testovanie vrátil staršiu kópiu, takže dátum poslednej položky nie je dnešný stav. Čerstvosť treba potvrdiť.
- **čiastočne** – niečo funguje, niečo chýba (napr. dokumentácia overená, dáta nie).
- **neoverené** – URL pochádza z oficiálnej dokumentácie inštitúcie alebo z katalógov feedov, ale priamo sa ju nepodarilo načítať. Nie je to verdikt, že nefunguje.
- **nie** – nefunguje alebo kanál neexistuje.

**Položiek/deň** = odhad za pracovný deň z dátumov položiek vo feede. Pri zdrojoch, kde vyšiel údaj len z katalógov feedov, ide o rádový odhad.

**Prečo je toľko „neoverené“:** pracovné prostredie, v ktorom audit bežal, má zablokovaný priamy prístup na tieto weby a nástroj na čítanie webu vyžaduje pri adresách, ktoré sa neobjavia vo výsledkoch vyhľadávania, potvrdenie od používateľa. Tie zostali bez odpovede. Niektoré weby (EUR-Lex SK, Eurostat Atom, katalóg RSS Parlamentu) navyše nástroj blokujú v robots.txt. Ako to dokončiť, je v časti [Ako dokončiť overenie](#ako-dokončiť-overenie).

---

## Inštitúcie EÚ

### Európsky parlament

| Zdroj | Kanál | URL | Funguje | Posledná položka | Položiek/deň | Poznámka |
|---|---|---|---|---|---|---|
| Tlačové správy (EN) | rss | `https://www.europarl.europa.eu/rss/doc/press-releases/en.xml` | neoverené | ~13.–14. 10. 2026 (katalóg feedov) | ~1, v plenárnom týždni 4–5 | Hlavný kandidát. Feed drží ~12 položiek. |
| Tlačové správy (SK) | rss | `https://www.europarl.europa.eu/rss/doc/press-releases/sk.xml` | neoverené | – | – | URL odvodená od existujúcich DE/FR verzií, nikde nezverejnená. Aj DE/FR feedy majú veľa položiek po anglicky. |
| Správy (tlačové + týždenný výhľad) | rss | `https://www.europarl.europa.eu/rss/doc/press-news/en.xml` | neoverené | ~15. 10. 2026 | 1–2 | Obsahuje aj „The Week Ahead“. |
| Program pléna | rss | `https://www.europarl.europa.eu/rss/doc/agendas-plenary/en.xml` | neoverené | ~9. 10. 2026 | ~8 položiek na zasadnutie | Dátum konania je len v titulku („Draft agenda – Wednesday, 21 October 2026 – Strasbourg“), treba ho vyparsovať. Najbližšie plénum 19.–22. 10. |
| Týždenný výhľad | rss | `https://www.europarl.europa.eu/rss/doc/agendas-news/en.xml` | neoverené | 9. 10. 2026 („Week Ahead 12–18 October“) | 1 týždenne | Najlepší prehľad na 7 dní dopredu. V lete pauza. |
| Výbory – všetky | rss | `https://www.europarl.europa.eu/rss/doc/last-news-committees/en.xml` | neoverené | ~15. 10. 2026 | ~2 | Položky o schôdzach výborov, dátum konania len v titulku. Katalóg feedov hlásil občasné výpadky. |
| Jednotlivý výbor (vzor: ENVI) | rss | `https://www.europarl.europa.eu/rss/committee/envi/en.xml` | neoverené | ~13. 10. 2026 | ~1 | Kód výboru v URL. Obsahuje návrhy programu schôdzí („Draft agenda for Monday, 19 October 2026“). |
| Open Data Portal (API) | api | `https://data.europarl.europa.eu/api/v2` | čiastočne | – | – | Dokumentácia overená: bez kľúča, limit 500 požiadaviek za 5 min, robots.txt povoľuje všetko. Kandidáti: `/events/feed?timeframe=one-week`, `/procedures/feed`, `/adopted-texts/feed` (Atom). Schôdze výborov v API zrejme nie sú. Tretia strana hlásila v apríli 2026 výpadky týchto feedov. |
| Legislatívne observatórium (OEIL) | html | `https://oeil.europarl.europa.eu/oeil/en/procedure-file?reference=…` | čiastočne | 14. 10. 2026 | – | Web po redizajne funguje, stránky procedúr majú fázu a kľúčové udalosti s dátumami. Menu „Export“ ponúka RSS/XML, no URL sa nepodarilo zistiť. Alternatíva: `/api/v2/procedures`. robots.txt neoverený. |

### Európska komisia

| Zdroj | Kanál | URL | Funguje | Posledná položka | Položiek/deň | Poznámka |
|---|---|---|---|---|---|---|
| Press corner – všetky typy | rss | `https://ec.europa.eu/commission/presscorner/api/rss` | áno (stará kópia) | 18. 9. 2026 (kópia) | ~5–7 (zo vzorky) | RSS 2.0, popis ~200 znakov. Typ dokumentu je v URL (`ip_`, `statement_`, `speech_`, `mex_`). Predvolene len 10 položiek – pri tomto objeme hrozí strata medzi behmi. Parametre `language`, `documenttype`, `policyarea`, `pagesize` existujú, ich účinok (aj `language=sk`, viac než 10 položiek) sa nepodarilo overiť. Denné správy (MEX) sa vo vzorke neobjavili. |
| Kalendár predsedníčky a komisárov | rss | `https://commission.europa.eu/node/33084/rss_en` | áno (stará kópia) | 3. 9. 2026 (kópia) | 10–15 | 30 položiek, bez stránkovania. Dátum konania len ako predpona titulku („2026-09-07 – …“), `pubDate` je čas zadania (nočné dávky), popis = miesto. |
| Agenda kolégia | – | – | nie | – | – | Žiadne RSS, iCal ani API. Register dokumentov je JavaScriptová aplikácia, PDF „Possible items for College agendas“ je dlhodobý plán. Zasadnutie kolégia sa dá odvodiť z kalendára komisárov (streda, v plenárnom týždni utorok). |
| Midday briefing | – | – | nie | – | – | Len video po udalosti, žiadny feed ani program. |
| Zastúpenie EK na Slovensku – aktuality | rss | `https://slovakia.representation.ec.europa.eu/node/2/rss_sk` | **áno** | 15. 10. 2026 14:14 | ~1 | Po slovensky, 30 položiek (~3,5 mesiaca), celý krátky popis. Položky nie sú zoradené a `pubDate` sa správa ako dátum úpravy – deduplikovať podľa URL. robots.txt povoľuje. |
| Zastúpenie EK – podujatia | rss | `https://slovakia.representation.ec.europa.eu/node/4/rss_en?prefLang=sk` | čiastočne | – (0 položiek) | ~0 | Feed funguje, ale je prázdny; podujatia sú zriedkavé. |

### Rada EÚ a Európska rada

| Zdroj | Kanál | URL | Funguje | Posledná položka | Položiek/deň | Poznámka |
|---|---|---|---|---|---|---|
| Tlačové správy (EN) | rss | `https://www.consilium.europa.eu/en/rss/pressreleases.ashx` | neoverené | – | – | URL z oficiálnej stránky RSS Rady, nepodarilo sa načítať. |
| Tlačové správy (SK) | rss | `https://www.consilium.europa.eu/sk/rss/pressreleases.ashx` | neoverené | – | – | Slovenská stránka RSS existuje, URL feedu nepotvrdená. |
| Zasadnutia – Európska rada | rss | `https://www.consilium.europa.eu/en/rss/meetings.ashx?cat=euco` | **áno** | najbližšie 15.–16. 10., 13. 11. (neformálna), 17. 12. | agenda | Dátum konania v `a10:updated` (len deň začiatku), koniec viacdňových zasadnutí len v URL. 50 položiek vrátane minulých, nezoradené. |
| Zasadnutia – Coreper | rss | `https://www.consilium.europa.eu/en/rss/meetings.ashx?cat=coreper` | **áno** | 21. 10. (Coreper I a II), 23. 10. | ~3 týždenne | Prázdne popisy. |
| Zasadnutia – EPSCO | rss | `https://www.consilium.europa.eu/en/rss/meetings.ashx?cat=epsco` | **áno** | 19. 10. 2026 | agenda | – |
| Zasadnutia – Rada pre zahraničné veci | rss | `https://www.consilium.europa.eu/en/rss/meetings.ashx?cat=fac` | **áno** | ďalšie 16. 11. 2026 | agenda | – |
| Zasadnutia – ostatné formácie | rss | `…/meetings.ashx?cat=` agri, compet, ecofin, eycs, envi, gac, jha, tte, committees… | neoverené | – | – | Z oficiálneho zoznamu, rovnaký formát ako overené feedy. |
| Kalendár zasadnutí | html | `https://www.consilium.europa.eu/en/meetings/` | **áno** | 15.–22. 10.: Európska rada, neformálne rady (vzdelávanie, obchod), EPSCO | – | Záloha a kontrola. Jediný zdroj s celými rozsahmi dátumov a so samitmi s tretími krajinami (EÚ – Kanada 29.–30. 10.). iCal neexistuje. |

### EUR-Lex

| Zdroj | Kanál | URL | Funguje | Posledná položka | Položiek/deň | Poznámka |
|---|---|---|---|---|---|---|
| Návrhy Komisie (COM) | rss | `https://eur-lex.europa.eu/EN/display-feed.rss?rssId=161` | čiastočne | položky bez dátumu (kanál 17. 8., kópia) | neznáme | 100 položiek, titulky sú len čísla CELEX, takmer bez popisu. Ako dátum treba brať čas prvého zachytenia. |
| Legislatíva EP a Rady | rss | `https://eur-lex.europa.eu/EN/display-feed.rss?rssId=162` | áno (stará kópia) | 26. 8. 2026 (kópia) | ~4 | 100 položiek, väčšinou korigendá, prázdne popisy. |
| Úradný vestník, séria L | rss | `https://eur-lex.europa.eu/EN/display-feed.rss?rssId=222` | áno (stará kópia) | 21. 9. 2026 (kópia) | ~14 | 100 položiek ≈ 7 pracovných dní. Titulky často len identifikátor (`OJ:L_…`). |
| Úradný vestník, séria C | rss | `https://eur-lex.europa.eu/EN/display-feed.rss?rssId=221` | áno (stará kópia) | 21. 9. 2026 (kópia) | ~16 | Veľa šumu (oznámenia Súdneho dvora, fúzie, štátna pomoc, kurzy). |
| Slovenské verzie feedov | rss | `https://eur-lex.europa.eu/SK/display-feed.rss?rssId=222` | nie | – | – | Cesta `/SK/` je pre automatické čítanie zakázaná v robots.txt (podľa testovacieho nástroja). |
| Cellar SPARQL | api | `https://publications.europa.eu/webapi/rdf/sparql` | neoverené | – | – | Podľa Úradu pre publikácie verejné, bez kľúča. Dá sa z neho získať aj slovenský titulok a dátum. Živý dopyt sa nepodarilo spustiť. |
| Webová služba EUR-Lex (SOAP) | api | – | neoverené | – | – | Vyžaduje registráciu a schválenie, limit 1 000 volaní denne. Len ak by zlyhal SPARQL. |

### Eurostat

| Zdroj | Kanál | URL | Funguje | Posledná položka | Položiek/deň | Poznámka |
|---|---|---|---|---|---|---|
| Kalendár zverejnení | ics | `https://ec.europa.eu/eurostat/cache/RELEASE_CALENDAR/calendar_EN.ics` | neoverené | – | – | Podľa Eurostatu sa aktualizuje 2× denne, obsahuje dátum a čas zverejnenia. Načítanie zlyhalo. Najvhodnejší kandidát na „čo vyjde tento týždeň“. |
| Euroindikátory (tlačové správy) | html | `https://ec.europa.eu/eurostat/en/news/euro-indicators` | áno (stará kópia) | 31. 7. 2026 (kópia) | ~0,7 | Atom feedy Eurostatu (News releases) sú zakázané v robots.txt. HTML výpis má dátum, titulok aj perex. |
| Aktualizácie dát | rss | `https://ec.europa.eu/eurostat/api/dissemination/catalogue/rss/en/statistics-update.rss` | áno (stará kópia) | 24. 9. 2026 (kópia) | ~250 | Každá aktualizácia datasetu, obnova o 11:00 a 23:00. Neštandardný formát dátumu, duplicity. Bez filtra príliš veľa šumu. |
| Statistics API | api | `https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/…` | neoverené | – | – | Bez kľúča. Na konkrétne dátové texty, nie na zisťovanie noviniek. |

---

## Kontrolné a ďalšie inštitúcie

| Zdroj | Kanál | URL | Funguje | Posledná položka | Položiek/deň | Poznámka |
|---|---|---|---|---|---|---|
| Európsky dvor audítorov | – | `https://www.eca.europa.eu/en/all-news` | nie | – | 2–3 osobitné správy mesačne | Žiadne RSS, web je SharePoint vykresľovaný JavaScriptom. Statické fragmenty tlačových správ (`/ECAHTMLNews/NEWS-SR-2026-NN/en/body.html`) nemajú dátum. Orientačný plán publikácií v PDF existuje. robots.txt sa nepodarilo načítať. |
| OLAF | rss | `https://anti-fraud.ec.europa.eu/node/2/rss_en` | **áno** | 14. 10. 2026 | ~0,7 týždenne | Testovaný variant s `?prefLang=pt`, jazyk kanála je aj tak EN. 30 položiek (~10 mesiacov). `pubDate` býva neskorší ako zobrazený dátum – dátum brať z URL. robots.txt povoľuje. |
| Európska prokuratúra (EPPO) | rss | `https://www.eppo.europa.eu/node/2/rss_en` | čiastočne | feed 22. 9., web 14. 10. 2026 | ~0,6, v aktívnom týždni ~1,4 | Testovaný variant s `?prefLang=es`. 30 položiek. Feed zaostáva za webom, alebo nástroj vrátil starú kópiu. Slovenské prípady majú v titulku predponu „Slovakia:“. robots.txt povoľuje. |
| Súdny dvor EÚ – tlačové správy | rss | `https://curia.europa.eu/site/rss.jsp?lang=sk&secondLang=en` | neoverené | – | – | Oficiálne zdokumentované v 24 jazykoch vrátane SK (alternatíva `lang=en&secondLang=fr`). Nepodarilo sa načítať. Starý feed `/feeds/rssfren.xml` je mŕtvy. |
| Súdny dvor EÚ – súdny kalendár | html | `https://curia.europa.eu/site/jcms/qua2_6301/en/judicial-calendar` | **áno** | kalendár 13. 10. – 3. 11. 2026 | ~3,6 rozsudku, 1,4 návrhu GA, 1,8 pojednávania | Pri každej udalosti dátum, čas, typ, číslo veci a jazyk konania – slovenské veci sa dajú odfiltrovať (napr. C-853/25, Najvyšší súd, 15. 10.). robots.txt povoľuje. |
| Výbor regiónov | html | `https://www.cor.europa.eu/en/news/all-news-and-press-releases` | **áno** | 15. 10. 2026 | 12 za 14.–15. 10. (plenárny týždeň), inak menej | Žiadne RSS ani API. Prvá strana má 12 položiek s dátumom, typom (News / Press release) a perexom. robots.txt povoľuje. Stránkovanie neoverené. |
| EHSV – tlačové správy | rss | `https://www.eesc.europa.eu/en/news-media/press-releases.rss` | **áno** | 24. 9. 2026 | ~0,3 (nárazovo v plenárnom týždni) | 20 položiek, popis obsahuje HTML. |
| EHSV – správy | rss | `https://www.eesc.europa.eu/en/news-media/news.rss` | **áno** | 7. 10. 2026 | ~0,4 | 20 položiek. |

---

## Médiá

| Zdroj | Kanál | URL | Funguje | Posledná položka | Položiek/deň | Poznámka |
|---|---|---|---|---|---|---|
| Politico Europe | rss | `https://www.politico.eu/feed/` | neoverené | – | – | Uvádzajú ho katalógy feedov. Existujú aj sekčné feedy `/section/{politics, foreign-affairs, energy, defense…}/feed/`. Podmienky použitia neoverené. |
| Euronews – My Europe | rss | `https://www.euronews.com/rss?format=mrss&level=vertical&name=my-europe` | neoverené | podľa katalógu feedov aktívny | – | Vzor URL z oficiálnej stránky widgetov Euronews; odber znamená súhlas s ich podmienkami. |
| Financial Times – EÚ | rss | `https://www.ft.com/european-union?format=rss` (aj `/europe?format=rss`) | neoverené | podľa katalógu feedov aktívny | – | Len titulky, články sú za paywallom. Podmienky RSS neoverené. |
| Euractiv | rss | `https://www.euractiv.com/feed/` | neoverené | – | – | Podľa Semaforu (28. 9. 2026) Euractiv prechádza na značku Rapporteur – treba overiť, či feed ešte žije a kam smeruje. |
| Rapporteur (web) | – | `https://www.rapporteur.com/` | nie | – | – | Každé načítanie zlyhalo (možno blokuje roboty), RSS sa nenašlo. Obsah je zrejme za predplatným. Newsletter rieši ranný monitoring cez Gmail. |
| TASR | – | – | vynechané | – | – | Podľa pravidla 3 do potvrdenia prístupu. |

## Partneri euBrief

| Zdroj | Kanál | URL | Funguje | Posledná položka | Položiek/deň | Poznámka |
|---|---|---|---|---|---|---|
| Update EU (CZ) | rss | `https://update-eu.cz/rss` | **áno** | 15. 10. 2026 12:49 | ~5 | Česky, 10 položiek ≈ 46 hodín, bez plného textu. robots.txt povoľuje. |
| Focus Europe (PL) | rss | `https://focuseurope.pl/feed/` | **áno** | 15. 10. 2026 12:50 | ~10–12 | Poľsky. Feed obsahuje plný text (`content:encoded`) – ukladať len popis. Hĺbka feedu neoverená (WordPress má štandardne 10), pri tomto objeme je to tesné. Záloha: WordPress REST API. robots.txt povoľuje. |
| EastFlank | rss | `https://eastflank.eu/feed/` | **áno** | 13. 10. 2026 | ~1 | Anglicky, plný text vo feede (neukladať). robots.txt obsahuje výhradu proti vyťažovaniu textov a dát a tréningu AI podľa smernice 2019/790; zber titulkov bez AI to neblokuje, ale od partnera je vhodný súhlas. |

---

## Zhrnutie

- **Overené naživo, pripravené pre zberač:** Zastúpenie EK na Slovensku, kalendár Rady (4 feedy + HTML záloha), OLAF, EHSV (2 feedy), súdny kalendár SDEÚ, Výbor regiónov, Update EU, Focus Europe, EastFlank.
- **Funkčné, čerstvosť treba potvrdiť:** Press corner, kalendár komisárov, EUR-Lex (4 feedy), Eurostat, EPPO.
- **Neoverené, URL z oficiálnych zdrojov:** všetky RSS Parlamentu a jeho API, tlačové správy Rady, RSS Súdneho dvora, Politico, Euronews, FT, Euractiv, kalendár Eurostatu, Cellar SPARQL.
- **Bez použiteľného kanála:** agenda kolégia, midday briefing, Dvor audítorov, Rapporteur, RSS export OEIL.

## Ako dokončiť overenie

Pripravený je krátky testovací skript (len štandardná knižnica Pythonu), ktorý všetky URL z tabuliek načíta naraz a pri každej vypíše HTTP stav, počet položiek, najnovší a najstarší dátum, počet položiek za 7 dní a 30 hodín, prítomnosť popisu a plného textu a súlad s robots.txt.

1. **Teraz:** spustiť ho z počítača redakcie (cez aplikáciu Claude na počítači) – overí existenciu a čerstvosť všetkých „neoverených“ a „starých kópií“ za minútu.
2. **Vo fáze 3:** spustiť ho ešte raz v GitHub Actions. Až to ukáže, či niektoré weby (Politico, FT, Euractiv a pod.) neblokujú IP adresy dátových centier, z ktorých zberač pobeží.

## Na rozhodnutie pred fázou 2

1. **Dvor audítorov** nemá RSS a web vyžaduje JavaScript. Návrh: zatiaľ vynechať (prehliadač v zberači by bol ťažká závislosť), termíny sledovať z orientačného plánu publikácií.
2. **Agenda kolégia** nemá feed. Návrh: odvodiť z kalendára komisárov.
3. **EUR-Lex** má v RSS slabé titulky. Návrh: Úradný vestník L (`rssId=222`) a návrhy Komisie (`rssId=161`), titulky a slovenské názvy doplniť cez Cellar SPARQL, ak sa ho podarí overiť. Sériu C vynechať (šum).
4. **Eurostat:** aktualizácie dát sú šum. Návrh: kalendár zverejnení (iCal), ak funguje, a HTML výpis euroindikátorov.
5. **Rapporteur / Euractiv:** web Rapporteur vynechať (paywall, newsletter ide cez Gmail). Euractiv feed overiť po rebrandingu.
6. **HTML zdroje** (Výbor regiónov, súdny kalendár SDEÚ, záloha kalendára Rady) – súhlas s ich zaradením podľa pravidla 5?
7. **Parlament po slovensky:** ak `sk.xml` neexistuje, ide sa z anglického feedu.
8. **EastFlank:** stačí ústny súhlas partnera kvôli výhrade v robots.txt, alebo ho netreba?

## Technické poznámky pre fázu 2

- **Dátum konania udalostí** je často len v titulku alebo v URL: program pléna EP, výbory EP, kalendár komisárov (predpona `RRRR-MM-DD`), koniec zasadnutí Rady (`/15-16/` v URL).
- **Rada:** feedy začínajú znakom BOM a namiesto `pubDate` majú `a10:updated`.
- **`pubDate` nie je vždy dátum zverejnenia:** Zastúpenie EK a OLAF ho menia pri úprave – deduplikovať podľa URL a dátum brať z URL.
- **Eurostat:** dátumy v tvare `2026-09-24 23:00:00.0`, nie RFC 822.
- **EUR-Lex:** v titulkoch dvojito escapované `&amp;`, položky len s dátumom bez času.
- **WordPress feedy partnerov:** ignorovať `content:encoded`, ukladať len popis skrátený na 300 znakov.
- **Hĺbka feedov:** Press corner (10 položiek predvolene) a kalendár komisárov (30 bez stránkovania) môžu pri vyššom objeme strácať položky medzi behmi – zberač by mal hlásiť do `status.json`, keď sú všetky položky feedu nové.

## Vyradení kandidáti

- EP – feed dokumentov pléna (`/rss/doc/plenary/en.xml`): ~17 pozmeňujúcich návrhov a uznesení denne, šum.
- EP – „Coming up in committees“: len statické stránky.
- Register Rady (`LPD.xml`, `AGNDCONS.xml`): dokumenty bez redakčnej hodnoty, jeden zastaraný.
- EHSV – „recent content“: mix bez poradia, zaostáva za ostatnými feedmi.
- EUR-Lex `rssId=165/166`: vracajú obsah série C, nie L.
- Eurostat Atom feedy: zakázané v robots.txt.
