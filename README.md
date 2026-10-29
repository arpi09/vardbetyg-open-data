# Vårdbetyg – öppna data

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23399625.svg)](https://doi.org/10.5281/zenodo.23399625)

Öppna, maskinläsbara CSV-filer med jämförelsedata för svenska vårdcentraler
(primärvård). Det här är samma data som driver [vardbetyg.se](https://vardbetyg.se),
publicerad så att andra kan granska, analysera och bygga vidare på den.

Vårdbetyg är ett oberoende jämförelseprojekt. Det är **inte affilierat med
1177, SKR eller Socialstyrelsen** och representerar ingen myndighet.

## Vad datan är

Sex filer, uppdaterade tillsammans:

- **`units.csv`** – en rad per vårdcentral: namn, kommun, region, HSA-id (när
  det är känt), det beräknade Vårdbetyget, kategoripoängen (patientupplevelse
  och tillgänglighet) och varje enkätdimension med konfidensintervall, samt
  antal svarande och svarsfrekvens. Sedan schemaversion 3.0 också
  huvudman (region eller privat, ur Nationell Patientenkät), kedjan i
  vårdcentralens namn och, där en öppen källa anger det, bolaget som driver
  vårdcentralen med organisationsnummer. Sedan 3.1 har units.csv,
  kommuner.csv och historik.csv också `kommun_kod` (SCB:s fyrsiffriga
  kommunkod som text, till exempel `0117`) och `lan_kod` (tvåsiffrig).
- **`historik.csv`** – en rad per vårdcentral och mätår i Nationell
  Patientenkät från 2015: områdenas poäng med konfidensintervall, antal
  svarande, huvudman och kedja det året. Enkäten byttes 2025, så kolumnen
  `serie` säger vilken enkät en rad kommer ur. Inget Vårdbetyg bakåt i tiden.
  Kommunnamnen följer SCB:s stavning (290 kommuner). Region Sörmland deltog
  inte 2025, så Sörmlands vårdcentraler har rader bara till och med 2023.
  På sajten heter filen
  [vardcentraler-historik.csv](https://vardbetyg.se/data/vardcentraler-historik.csv).
- **`kommuner.csv`** – aggregat per kommun: antal vårdcentraler, hur många som
  har ett publicerat betyg, medel- och toppbetyg, och summa svarande.
- **`fragor.csv`** – enkäten på frågenivå: en rad per vårdcentral och
  redovisad fråga, med poäng, konfidensintervall, andel positiva svar och
  antal svarande.
- **`fotnoter.csv`** – koderna F01 till F06 som säger varför ett värde saknas.
  Koderna beskriver källan, aldrig vårdcentralen.
- **`utmarkelser.csv`** – de utmärkta vårdcentralerna: en rad per utmärkelse
  (Topp 10 % i Sverige, Topprankad i kommunen och Årets klättrare), samma
  utmärkelser som sajten visar.

Fullständiga kolumndefinitioner finns i
[`data-dictionary.md`](data-dictionary.md) och, med period och saknade värden
för varje fält, i kodboken (se nedan).

Får du förskjutna kolumner i Excel eller Power Query, eller tappar
`kommun_kod` sin inledande nolla? Se
[Så läser du filerna](data-dictionary.md#så-läser-du-filerna), med recept för
Power Query, Excel, R och Python.

Talvärden använder punkt som decimaltecken (maskinläsbart). Tomma celler
betyder att värdet saknas, till exempel när en vårdcentral fick för få svar för
att ett betyg ska kunna redovisas.

## Kodbok

[`kodbok.md`](kodbok.md) beskriver varje fält i varje fil: definition, källa,
period och hur ett saknat värde anges. Samma innehåll finns maskinläsbart i
[`kodbok.csv`](kodbok.csv), en rad per fält, för den som vill läsa in
kodboken i R, Stata eller Python.

## Räkna om betyget själv

[`rakna-om-betyget.ipynb`](rakna-om-betyget.ipynb) hämtar Nationell
Patientenkät och Socialstyrelsens väntetider direkt från källorna, räknar om
Vårdbetyget med metoden på [vardbetyg.se/metodik](https://vardbetyg.se/metodik)
och jämför med `units.csv`. Den kräver Python 3.10 eller senare och bara
standardbiblioteket. Öppna den i Jupyter, VS Code eller Google Colab och kör
cellerna uppifrån och ned.

- **Patientupplevelsen** (70 % av betyget) räknas om helt: sammanvägningen av
  enkätens områden, rikssnittet och krympningen mot rikssnittet.
- **Tillgängligheten** (30 %) räknas fram ur Socialstyrelsens publicerade
  andelar. Krympningen av den använder antalet ärenden per månad, och det
  ingår inte i den öppna datan. Notebooken hämtar det inte, utan kontrollerar
  att den publicerade poängen ligger mellan vårdcentralens eget värde och
  rikssnittet.

För Nationell Patientenkät 2025 återskapar notebooken alla 1 192 publicerade
betyg inom avrundning, 1 105 av dem exakt. Resten skiljer en tiondel, eftersom
tillgänglighetspoängen i `units.csv` är avrundad. Hela resultatet står i
notebookens sista celler. Hittar du en avvikelse, öppna en issue i det här
repot. Ett betyg ändras aldrig i efterhand i en släppt export: avvikelser
rättas i nästa metodikrelease.

## Så citerar du

Datan är öppen under [CC BY 4.0](LICENSE.md). Använd den fritt och ange var
den kommer ifrån.

En kort källrad, för en artikel eller ett diagram:

> Källa: Vårdbetyg (vardbetyg.se), CC BY 4.0.

Referensen till datasetet, för en rapport eller en uppsats:

> Vårdbetyg. (2026). Vårdbetyg: öppet dataset för svenska vårdcentraler, Nationell Patientenkät 2025 (version 2025.1) [Dataset]. Zenodo. https://doi.org/10.5281/zenodo.23399626

Zenodo arkiverar varje release och ger den en DOI:

- **Version 2025.1** (release `v2025.1`):
  [10.5281/zenodo.23399626](https://doi.org/10.5281/zenodo.23399626). Citera
  den när någon annan ska kunna hitta exakt samma filer.
- **Alla versioner**:
  [10.5281/zenodo.23399625](https://doi.org/10.5281/zenodo.23399625). Pekar
  alltid på den senaste versionen.

Repot har en [`CITATION.cff`](CITATION.cff), så GitHub visar referensen i
APA och BibTeX under **Cite this repository** i högerspalten. Samma referens
står på [vardbetyg.se/om-datan](https://vardbetyg.se/om-datan#citera).

## Källor och licens

Datan bygger på öppna, offentliga källor:

- **Nationell Patientenkät** – patienternas svar om sin vårdcentral
  (upplevelsedimensioner, svarsantal, svarsfrekvens). Öppen publik data per
  vårdcentral.
- **Socialstyrelsen, väntetider i primärvården** – tillgänglighet per
  vårdcentral.
- **Västra Götalandsregionen, vårdval vårdcentral** (CC0) – vilket bolag som
  driver en privat vårdcentral i regionen.
- **Bolagsverket, värdefulla datamängder** – bolagets registrerade namn och
  organisationsnummer.

**Vårdbetyget är härlett.** Själva betyget finns inte i källorna, utan beräknas
av sajten ur den öppna datan (en viktad sammanvägning av patientupplevelse och
tillgänglighet, med bayesiansk shrinkage mot rikssnittet så att enheter med få
svar inte toppar listan på statistiskt brus). Hela metoden är dokumenterad på
[vardbetyg.se/metodik](https://vardbetyg.se/metodik). Utmärkelserna i
`utmarkelser.csv` räknas fram ur betyget och ur patientenkätens historik. De
kan inte köpas och påverkar aldrig betyget.

Käll-datan tillhör sina respektive avsändare och lyder under deras villkor. Den
här sammanställningen (urvalet, strukturen och de härledda Vårdbetygs-
kolumnerna) är licensierad under **CC BY 4.0** (se [LICENSE](LICENSE.md)):
använd gärna datan, ange "Vårdbetyg (vardbetyg.se)" som källa.

Datan går också att ladda ner direkt från sajten:
[vardbetyg.se/riket](https://vardbetyg.se/riket).

## Hur kolumnerna hänger ihop med sajten

Varje rad går att slå upp på sajten via sin `slug`:

- vårdcentral: `https://vardbetyg.se/vardcentral/<slug>`
- kommun: `https://vardbetyg.se/kommun/<slug>`

Poängen i CSV:erna är desamma som visas på respektive sida. På sajten skrivs de
med svenskt decimalkomma; i CSV:erna med punkt, för att vara enklare att läsa in
i kalkylprogram och skript.

## Uppdateringstakt och releaser

Datan följer källorna. Nationell Patientenkät för primärvården publiceras
årsvis, väntetiderna oftare. Filerna här uppdateras när en ny mätning har
lästs in i tjänsten. Varje fil anger sin mätperiod (`measurement_period` i
`units.csv`), så det framgår vilken omgång en rad hör till.

Varje årsexport släpps som en release i det här repot, med taggen
`v<mätår>.<utgåva>`: `v2025.1` är första utgåvan för Nationell Patientenkät
2025. Zenodo arkiverar releasen och ger den en DOI, så att exakt samma filer
går att hitta och citera senare. Citera releasen, inte filerna i `main`, om
någon annan ska kunna räkna om dina siffror.

### Så görs en release

1. Kör `USE_PGLITE=true npm run export:open-data` i vardbetyg-repot. Skriptet
   skriver alla filer ovan plus `CITATION.cff` och `.zenodo.json`.
2. Kopiera filerna hit och öppna en pull request.
3. Releasen (som skapar DOI:n) publiceras först efter ägarens uttryckliga ja.
4. När DOI:n finns: sätt den i `src/lib/domain/oppen-data/citering.ts` i
   vardbetyg-repot och kör exporten igen, så får sajten, `CITATION.cff` och den
   här filen samma DOI.

## Om siffrorna

- Ett tomt `vardbetyg` (och `for_fa_svar = true`) betyder att vårdcentralen hade
  för få svar för att ett betyg ska redovisas med statistisk säkerhet. Det säger
  ingenting om vårdens kvalitet, bara att underlaget är för tunt.
- Konfidensintervallen kommer från enkäten och visar hur säkert varje
  dimensionsvärde är. Ett smalt intervall betyder fler svar och en säkrare siffra.
- Betyget är ett beslutsunderlag, inte en fullständig bild av en vårdcentral.
  Använd det tillsammans med annat du vet.
- `huvudman`, `kedja` och `vardgivare` beskriver vem som driver
  vårdcentralen. Patienterna skiljer sig mellan regiondrivna och privata
  vårdcentraler, så en skillnad i snittbetyg mellan driftformerna säger inte
  att den ena vården är bättre. Vårdbetyg jämför inte betyg mellan
  driftformer. Ett tomt `kedja` betyder att namnet inte bär någon kedja i
  tabellen, inte att vårdcentralen är fristående.
