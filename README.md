# Vårdbetyg – öppna data

Öppna, maskinläsbara CSV-filer med jämförelsedata för svenska vårdcentraler
(primärvård). Det här är samma data som driver [vardbetyg.se](https://vardbetyg.se),
publicerad så att andra kan granska, analysera och bygga vidare på den.

Vårdbetyg är ett oberoende jämförelseprojekt. Det är **inte affilierat med
1177, SKR eller Socialstyrelsen** och representerar ingen myndighet.

## Vad datan är

Fem filer, uppdaterade tillsammans:

- **`units.csv`** – en rad per vårdcentral: namn, kommun, region, HSA-id (när
  det är känt), det beräknade Vårdbetyget, kategoripoängen (patientupplevelse
  och tillgänglighet) och varje enkätdimension med konfidensintervall, samt
  antal svarande och svarsfrekvens.
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
[`data-dictionary.md`](data-dictionary.md).

Talvärden använder punkt som decimaltecken (maskinläsbart). Tomma celler
betyder att värdet saknas, till exempel när en vårdcentral fick för få svar för
att ett betyg ska kunna redovisas.

## Så citerar du

Datan är öppen under [CC BY 4.0](LICENSE.md). Använd den fritt och ange var
den kommer ifrån.

En kort källrad, för en artikel eller ett diagram:

> Källa: Vårdbetyg (vardbetyg.se), CC BY 4.0.

Referensen till datasetet, för en rapport eller en uppsats:

> Vårdbetyg. (2026). Vårdbetyg: öppet dataset för svenska vårdcentraler, Nationell Patientenkät 2025 (version 2025.1) [Dataset]. https://github.com/arpi09/vardbetyg-open-data

Repot har en [`CITATION.cff`](CITATION.cff), så GitHub visar referensen i
APA och BibTeX under **Cite this repository** i högerspalten. Samma referens
står på [vardbetyg.se/om-datan](https://vardbetyg.se/om-datan#citera).

<!--
DOI: platshållare. Zenodo skapar DOI:n vid den första releasen. Den läggs in
här, i CITATION.cff och på sajten först när releasen är publicerad.
-->

## Källor och licens

Datan bygger på två öppna, offentliga källor:

- **Nationell Patientenkät** – patienternas svar om sin vårdcentral
  (upplevelsedimensioner, svarsantal, svarsfrekvens). Öppen publik data per
  vårdcentral.
- **Socialstyrelsen, väntetider i primärvården** – tillgänglighet per
  vårdcentral.

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
