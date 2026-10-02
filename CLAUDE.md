# Werkafspraken voor dit project (Planbord)

Dit document legt vast hoe er in dit project met Claude Code wordt samengewerkt. Dit wordt bij elke sessie automatisch gelezen — ook als een gesprek lang wordt of een nieuwe sessie start, blijven deze afspraken dus gelden.

De eigenaar van dit project (Noortje) is niet zelf een ontwikkelaar. Alles moet in gewone, begrijpelijke taal uitgelegd worden.

## Wat je NIET mag doen, zelfs niet als "verbetering"

- **De Microsoft/SharePoint-koppeling niet aanraken.** `index.html` logt in via Microsoft Entra ID (MSAL) en leest/schrijft data naar een SharePoint-lijst via de Graph API. Dit is bewust zo gebouwd na veel troubleshooten — verwijder, vereenvoudig of "refactor" dit niet, ook niet als het overbodig complex lijkt.
- **Geen automatische opschoning, herstructurering of "moderne herschrijving"** van bestaande, werkende code, tenzij expliciet gevraagd. Dit geldt ook voor dingen die een ontwikkelaar normaal gesproken zou "verbeteren" (bijv. losse `<script>`-blokken samenvoegen, `var` vervangen door `let`/`const`, CSS herstructureren, functies verplaatsen). Als iets lelijk maar werkend is: laat het met rust.
- **Geen kleuren, statuslogica of bedrijfsregels wijzigen zonder dat het gevraagd is.** Sommige kleuren zijn bewust NIET de huisstijlkleuren (bijv. de status-bolletjes op bordjes), en de volgorde-logica in Bouwplanning en Magazijnplanning is zorgvuldig afgestemd op hoe er in het echt gewerkt wordt.
- **Geen bestanden opsplitsen, herstructureren of verplaatsen** zonder dat eerst voor te stellen en op akkoord te wachten. Dit geldt ook voor het verplaatsen van grote stukken HTML/JS binnen hetzelfde bestand (bijv. het bewerk-scherm) — eerst voorstellen, dan pas doen.

## Hoe er gewerkt wil worden

- **Doe alleen wat er gevraagd is.** Lijkt iets anders ook een verbetering, stel dat dan voor en wacht op akkoord — verander het niet automatisch mee. Dit geldt ook voor losse implementatiekeuzes (bijv. een bepaalde UI-aanpak) die niet expliciet zijn afgesproken.
- **Laat altijd zien wat er gaat veranderen** voordat het wordt toegepast, en leg in gewone taal uit wat de wijziging doet.
- **Bij twijfel: vragen, niet aannemen.**
- **Nooit committen of pushen naar GitHub zonder dat dat expliciet gevraagd is** — ook niet in Auto mode. Altijd expliciet vragen voordat `git commit` of `git push` wordt uitgevoerd.
- **Een akkoord om te pushen is altijd beperkt tot wat er op dat moment concreet besproken is** — niet tot alles wat toevallig op dat moment nog los klaarstaat. Staan er meerdere, losse dingen klaar om te pushen (bijv. een los documentatie-bestand én niet-gerelateerde app-wijzigingen van eerder in het gesprek), vraag die dan apart na. Nooit een "ja" die over het ene ding ging, gebruiken als dekking om ook iets anders mee te pushen.
- Geen overbodige disclaimers herhalen (zoals steeds "ik push nog niets" zeggen) — gewoon kort vragen "wil je dit al pushen of nog verder werken?" zodra iets echt klaar is.

## Twee bestanden, altijd samen

- `index.html` = de echte, live versie met MSAL/SharePoint-login. Hier NOOIT de auth-logica aanraken.
- `index-test.html` = mock/test-versie die met localStorage werkt (geen Microsoft-login nodig). Hier worden alle Playwright-tests tegen gedraaid — nooit tegen `index.html`, want die heeft geen testdata-ondersteuning en vereist een echte login.
- Alle gedeelde (niet-MSAL) logica moet in beide bestanden **byte-identiek** blijven. Na elke wijziging: eerst in `index-test.html` bouwen en testen, dan exact dezelfde wijziging overzetten naar `index.html`, en controleren dat de gedeelde stukken identiek zijn.

## Terminologie

- **"Bordje"** betekent altijd de hele kaart/card op het planbord (een order-, levering- of notitie-bordje).
- Als het zonder verdere toevoeging over "de status" van een bordje gaat, wordt daarmee de **orderstatus** bedoeld (het veld `status`), niet de kasten-status, bezorgstatus of een ander statusveld — tenzij expliciet anders gezegd.
- **Bedrijfsregel:** de kasten-status volgt automatisch de orderstatus, behalve wanneer de orderstatus **geel** is — dan is er een keuze uit verschillende kasten-statussen (gepickt/wordt gebouwd/klaar voor controle). Bij elke andere orderstatus staat de kasten-status vast en gelijk aan de orderstatus.

## Status-kleuren (orderstatus en kasten-status)

| Kleur | Betekenis |
|---|---|
| Wit (leeg) | Nieuw |
| Blauw | Goedgekeurd |
| Rood | Aanwezig |
| Geel | In verwerking (orderstatus) / div. tussenstappen (kasten-status) |
| Groen | Klaar voor leveren (orderstatus) / gebouwd, klaar voor controle (kasten-status) |
| Zwart | Uitgeleverd |

Zwart is puur cosmetisch anders dan groen (allebei "klaar"/afgerond) — bijv. bij tellingen van "nog te doen" horen groen én zwart allebei niet mee te tellen.

## Overige praktische afspraken

- De testdata-knop (alleen in `index-test.html`) moet altijd een realistisch, actueel scenario opleveren, ongeacht de kalenderdatum waarop erop gedrukt wordt — "week 39/40/41" in die logica is relatief aan de echte datum van vandaag (39 = deze week), niet aan een vaste kalenderweek.

## Hoe Bouwplanning en Magazijnplanning met elkaar samenwerken (en waarom)

Dit zijn twee gekoppelde simulaties die elke dag opnieuw doorrekenen wie wat doet — geen vaste, handmatig ingestelde planning, maar een dag-voor-dag berekening op basis van de actuele bordjes op het Planbord.

- **Magazijnplanning** plant het picken (spullen klaarzetten) en de controle/inpak (eindcheck, ook voor orders zonder kasten).
- **Bouwplanning** plant het daadwerkelijk bouwen van de kasten.
- **De koppeling is fysiek verplicht, niet alleen praktisch:** Bouwplanning mag een order pas gaan bouwen nadat Magazijnplanning 'm heeft gepickt (je kan niet bouwen met spullen die er nog niet zijn). Zodra Bouwplanning een order heeft afgebouwd, geeft dat een seintje terug aan Magazijnplanning dat de controle/inpak-stap mag worden ingepland.
- **Prioriteit:** een order met een **gele** status (al in de werkplaats, halverwege gebouwd) wordt altijd als eerste afgemaakt — die wordt nooit onderbroken voor iets anders. Daarna geldt: hoe dichter bij de leverdatum, hoe hoger de prioriteit, met extra voorrang voor spoed-orders binnen hun eigen tijdsvenster.
- **Max. 2 weken vooruit (`MZ_LOOKAHEAD_DAYS` = 14 dagen):** zowel Magazijnplanning (picken) als Bouwplanning (bouwen) beginnen nooit aan een order waarvan de leverdatum meer dan 2 weken in de toekomst ligt — zelfs niet om anders onbenutte capaciteit te vullen. Die dag blijft dan bewust leeg. **Uitzondering:** werk dat al loopt (gele status, of al gepickt/klaar voor controle) wordt altijd gewoon afgemaakt, ongeacht de datum — dat is geen "te vroeg beginnen" maar het afronden van iets dat al bezig is.
- **Waarom deze grens er is:** zonder deze grens vult de planner lege capaciteit liever met willekeurig verder-weg-liggend werk dan de dag ongebruikt te laten — maar dat betekent dat er soms al weken van tevoren aan een order begonnen wordt terwijl die nog helemaal niet urgent is. Wil een order toch eerder gebouwd worden, dan schuift de planner het bordje zelf naar voren op het Planbord.

## Hoe wijzigingen getest worden

- Elke wijziging wordt eerst gebouwd en getest in `index-test.html`, via Playwright (geautomatiseerde browsertests) met neptestdata in `localStorage` — nooit tegen `index.html`, die heeft geen testdata-ondersteuning en vereist een echte Microsoft-login.
- Na elke wijziging: een syntax-check (alle `<script>`-blokken moeten foutloos zijn) en, waar relevant, een of meer Playwright-scenario's die het nieuwe gedrag controleren.
- Pas als `index-test.html` goed getest is, wordt dezelfde wijziging overgezet naar `index.html`, met een controle dat de gedeelde (niet-MSAL) stukken byte-identiek zijn in beide bestanden.

## Openstaande wensen (besproken, nog niet gebouwd)

- **Deels-beschikbare orders:** sommige orders hebben een deel van de kasten al klaar om te picken/bouwen, terwijl de rest nog op materiaal wacht (bijv. 20 van de 24 kasten). Afgesproken aanpak: twee velden op het bordje — "nu beschikbaar" en "totaal" — plus een eigen (latere) datum "spullen binnen" voor de resterende kasten. Dit wordt als eerste gebouwd, vóór de Concept/Klad-modus hieronder.
- **Concept/Klad-modus voor Bouwplanning:** een los werkblad waarin de bouwvolgorde vrij (zonder automatische regels) versleept kan worden, met waarschuwingen (niet-blokkerend) in een zijpaneel, en een "exporteren"-knop die de gekozen volgorde als nieuwe, vaste prioriteit voor de live planning vastlegt — tot er weer handmatig aangepast wordt. De enige regel die ook hier hard blijft: een order kan nooit eerder bouwen dan wanneer Magazijnplanning 'm heeft gepickt (fysieke onmogelijkheid, geen beleidskeuze). Magazijnplanning's pick-volgorde volgt na het exporteren automatisch dezelfde lijst. Deze modus is bewust nog even gepauzeerd totdat "deels-beschikbare orders" (hierboven) goed staat, omdat dat de opzet van de Concept-modus nog kan beïnvloeden.

## Huisstijl

Uit `Tom-Lock_huisstijl.pdf`. Dit is de officiële bedrijfshuisstijl (logo's, drukwerk, kleurenpalet) — let op: de **status-kleuren op de bordjes in de app zijn hier bewust los van** (zie de "wat je niet mag doen"-sectie hierboven), die zijn functioneel/semantisch (rood/geel/groen/blauw als workflow-status), geen huisstijlkleuren.

**Logo:** "TOM-LOCK" in zwarte letters op een geel blok, met de ondertitel "BEDRIJFSWAGENINRICHTING" eronder in grijs. Er is ook een icoon-variant (geel blokje met alleen een "T").

**Hoofdkleuren:**
| Kleur | Hex | Gebruikt in de app als |
|---|---|---|
| Zwart | `#000000` | — |
| Geel (huisstijlgeel) | `#d2c000` | `--brand-yellow` |

**Overig paletten:**
| Hex | Omschrijving |
|---|---|
| `#565100` | Donker olijfgeel |
| `#b2a400` | Olijfgeel |
| `#f3f0df` | Crème/ivoor |
| `#404649` | Donkergrijsblauw — dit is `--brand-dark` in de app (headers, knoppen, kaders) |
| `#a9aeb2` | Middengrijs |
| `#e7eaec` | Lichtgrijs |
| `#f4f6f7` | Zeer lichtgrijs |
| `#7ca6bc` | Staalblauw |
| `#acc7d6` | Lichtblauw |
| `#dde8f0` | Zeer lichtblauw |

**Fonts:**
- Koppen: **Alternate Gothic Condensed ATF** (de grote, smalle hoofdletterige koppen, bijv. "ONTWORPEN VOOR HELDEN")
- Lopende tekst: **Roboto** (bold voor nadruk/labels, regular voor platte tekst)

**Toon/stijl:** strakke, zakelijke productfotografie van ingerichte bestelwagens; donkergrijs/zwart met gele accenten; technisch en to-the-point, geen speelse illustraties.
