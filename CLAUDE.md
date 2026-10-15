# euBrief – zberač pre ranný monitoring

Kontext pre Claude Code. Prečítaj celý súbor pred každou úlohou v tomto repozitári.

## Čo budujeme
euBrief je slovenské médium o EÚ (partner SME). Redakcia každé ráno vyberá témy dňa. Tento repozitár je **zberač**: dvakrát denne v pracovné dni stiahne nové položky z verejných zdrojov (otvorené API, RSS, výnimočne HTML) a uloží ich v jednotnom formáte do JSON.

Zber je deterministický, **bez AI**. Výber tém (relevancia, slovenský uhol, vrstva, vlastník) robí neskôr Claude v rannom monitoringu mimo tohto repozitára. Cieľ: redakcia má plnú kontrolu nad vstupnými dátami, vidí, čo prišlo, odkiaľ a čo zlyhalo.

## Architektúra
```
zdroje → adaptéry (1 zdroj = 1 adaptér) → normalizácia + deduplikácia → data/latest.json
                                                                      → data/archiv/RRRR-MM-DD-HHMM.json
                                                                      → data/status.json
```
- Python 3.12, čo najmenej závislostí (napr. feedparser, httpx, parser HTML len kde treba).
- Zoznam zdrojov v `sources.yaml` (názov, typ zdroja, kanál, URL, aktívny áno/nie), nie natvrdo v kóde. Nový zdroj = nový riadok, prípadne malý adaptér.
- Spúšťanie: GitHub Actions, pondelok – piatok o ~6:15 a ~12:30 bratislavského času (ranný monitoring beží o 6:58 a 13:00). Cron v Actions je v UTC a môže meškať o niekoľko minút; od 25. 10. 2026 platí zimný čas (UTC+1). Vyrieš to tak, aby časy sedeli celoročne.
- Výsledok sa commitne do repozitára. Repozitár môže byť verejný, preto v dátach nesmú byť celé texty článkov.

## Jednotný formát položky
```json
{
  "id": "sha1 z normalizovanej URL",
  "source": "Európsky parlament",
  "source_type": "official | media | partner | oversight",
  "channel": "api | rss | html",
  "kind": "press_release | agenda | legislation | article | report | ruling | data | other",
  "institution": "skratka inštitúcie alebo null",
  "title": "...",
  "url": "odkaz na pôvodný zdroj",
  "published_at": "ISO 8601 s časovou zónou",
  "event_date": "ISO 8601, len pre agendu (kedy sa udalosť koná)",
  "fetched_at": "ISO 8601",
  "summary": "popis z feedu/API, max 300 znakov",
  "language": "en | sk | cs | pl | fr | de"
}
```
`status.json`: pre každý zdroj čas behu, OK/chyba, počet položiek, text chyby.

## Pravidlá
1. Novinky: položky publikované za posledných 30 hodín (ranný beh), resp. od ranného behu (obedný beh). Agenda: udalosti na najbližších 7 dní.
2. Ukladať len titulok, odkaz a krátky popis z feedu. Nikdy celé texty článkov (autorské práva, verejné repo).
3. Platené zdroje (Financial Times, Politico Pro, TASR): žiadne obchádzanie paywallu, len verejné titulky z RSS. TASR zatiaľ vynechať, kým sa nepotvrdí prístup cez predplatné.
4. Rešpektovať robots.txt a podmienky použitia. Pri čítaní HTML: identifikovateľný User-Agent, pauzy medzi požiadavkami.
5. Poradie kanálov: oficiálne API → RSS → HTML. HTML len vtedy, keď nič iné nie je.
6. Zlyhanie jedného zdroja nezastaví ostatné. Chyba ide do `status.json`, beh pokračuje.
7. Deduplikácia podľa normalizovanej URL (bez utm a podobných parametrov).
8. Ak zdroj vyžaduje API kľúč alebo registráciu, povedz to. Kľúče patria do GitHub Secrets, nikdy do kódu.
9. Newslettre v e-maile (Rapporteur, digest Komisie) sem nepatria, rieši ich ranný monitoring cez Gmail.
10. README a komunikácia po slovensky; kód a komentáre môžu byť po anglicky.

## Zdroje na overenie
Široký zoznam, postupne budeme osekávať. Pri každom zisti, či má oficiálne API alebo RSS, či funguje a aké čerstvé sú dáta. URL nehádaj, nájdi ich a otestuj.

**Inštitúcie EÚ (primárne zdroje)**
- Európsky parlament: tlačové správy, program pléna a výborov, Open Data Portal, Legislatívne observatórium (OEIL) – fáza kľúčových zákonov
- Európska komisia: Press corner (tlačové správy, denné správy, vyhlásenia), agenda kolégia a komisárov, Zastúpenie EK na Slovensku
- Rada EÚ a Európska rada: tlačové správy, kalendár zasadnutí
- EUR-Lex: nové legislatívne návrhy a prijaté akty, Úradný vestník
- Eurostat: zverejnenia dát (pre dátové texty)

**Kontrolné a ďalšie inštitúcie** (stále témy: právny štát, korupcia)
- Európsky dvor audítorov, OLAF, Európska prokuratúra (EPPO), Súdny dvor EÚ (tlačové správy k rozsudkom), Výbor regiónov, EHSV

**Médiá**
- Politico Europe, Euronews, Financial Times (len titulky), Rapporteur (web), Euractiv (návrh)
- TASR: vynechať do potvrdenia (pravidlo 3)

**Partneri euBrief**
- Update EU (CZ), Focus Europe (PL), EastFlank

## Fázy
- [ ] **1. Audit zdrojov** → `docs/zdroje.md`: tabuľka (zdroj, kanál, URL, funguje áno/nie, dátum poslednej položky, odhad položiek za deň, poznámka). Zberač zatiaľ nestavať. **Po audite zastaviť a čakať na schválenie.**
- [ ] **2. Zberač**: `sources.yaml`, adaptéry, normalizácia, deduplikácia, výstupy podľa formátu vyššie. Lokálny beh, ukázať ukážku `latest.json` a `status.json`.
- [ ] **3. Automatika**: GitHub Actions podľa časov vyššie, commit dát, ručné spustenie (workflow_dispatch) na testovanie, skúšobný beh v Actions.
- [ ] **4. Napojenie** ranného monitoringu na `latest.json` – robí sa v claude.ai; tu len zabezpečiť stabilnú URL súboru.
- [ ] **5. Neskôr**: týždenne aktualizovaný archív vlastných článkov euBrief; dashboard v Lovable číta tie isté dáta.

## Ako pracovať
- Pred každou fázou krátky plán (čo spravíš, ktoré súbory), potom realizácia.
- Testuj na reálnych zdrojoch, nie na vymyslených dátach. Výsledok ukáž.
- Malé commity s popisom po slovensky.
- Keď zdroj nejde, povedz to a navrhni alternatívu. Pravidlá 2 – 4 neobchádzaj.
