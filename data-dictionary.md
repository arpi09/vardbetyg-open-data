# Datakatalog (data dictionary)

Kolumndefinitioner för de exporterade CSV-filerna. Genererad av
`scripts/export-open-data.ts` (via src/lib/etl/open-data.ts) - redigera koden,
inte den här filen för hand.

Talvärden använder punkt som decimaltecken (maskinläsbart), tomma celler
betyder "saknat värde". Mätperiod: Nationell Patientenkät 2025
(kolumnen `measurement_period` säger radens mätår).
Schemaversion: **3.1** (kolumnen `schema_version` i units.csv).

Vi fyller aldrig i saknade värden med gissningar. När ett värde saknas säger
en fotnotskod varför (se Fotnoter nedan). Koderna beskriver källan, aldrig
vårdcentralen.

Kodboken (kodbok.md, maskinläsbart kodbok.csv) beskriver varje fält med typ,
källa, period och hur ett saknat värde anges. Notebooken
rakna-om-betyget.ipynb räknar om betyget ur de öppna källorna.

## Proveniens

- **NPE** - Nationell Patientenkät, öppen publik data per vårdcentral
  (patientupplevelse-dimensioner, svarsantal, svarsfrekvens).
- **Socialstyrelsen** - väntetider/tillgänglighet per vårdcentral.
- **VGR** - Västra Götalandsregionens öppna data om vårdval vårdcentral
  (CC0): vilket bolag som driver en privat vårdcentral i regionen.
- **Bolagsverket** - värdefulla datamängder (avgiftsfria, EU-förordning
  2023/138): bolagets registrerade namn och organisationsnummer.
- **Vårdbetyg (härlett)** - poäng som beräknas av sajten ur ovanstående
  öppna källor. Metod: <https://vardbetyg.se/metodik>.

Vårdbetyg är oberoende och inte affilierat med 1177, SKR eller Socialstyrelsen.

## Så citerar du

Datan är öppen under CC BY 4.0. Använd den fritt och ange var den kommer
ifrån.

En kort källrad, för en artikel eller ett diagram:

> Källa: Vårdbetyg (vardbetyg.se), CC BY 4.0.

Referensen till datasetet, för en rapport eller en uppsats:

> Vårdbetyg. (2026). Vårdbetyg: öppet dataset för svenska vårdcentraler, Nationell Patientenkät 2025 (version 2025.1) [Dataset]. Zenodo. https://doi.org/10.5281/zenodo.23399626

Repot har en CITATION.cff, så GitHub visar referensen i flera format under
"Cite this repository" (https://github.com/arpi09/vardbetyg-open-data). Som BibTeX:

```bibtex
@misc{vardbetyg20251,
  author = {{Vårdbetyg}},
  title = {Vårdbetyg: öppet dataset för svenska vårdcentraler, Nationell Patientenkät 2025},
  year = {2026},
  version = {2025.1},
  publisher = {Zenodo},
  doi = {10.5281/zenodo.23399626},
  url = {https://doi.org/10.5281/zenodo.23399626},
  note = {Dataset, CC BY 4.0}
}
```

## Så läser du filerna

Filerna är vanlig CSV enligt RFC 4180: kommatecken mellan kolumnerna,
rubrikrad överst, radslut CRLF. Fyra saker avgör om de läses in rätt.

- **Kommatecken i namn.** Text med kommatecken står inom citationstecken, till
  exempel `"Berga Läkarhus, Helsingborg"`. Det gäller över tusen rader i
  historik.csv. Läs in med citeringen på. Gör du inte det delas namnet i två
  kolumner och resten av raden hamnar ett steg fel.
- **UTF-8 utan BOM.** Filerna har ingen byte order mark. Program som gissar
  teckenkodning (Excel, äldre verktyg) väljer ofta fel och visar å, ä och ö
  som skräptecken. Ange UTF-8 själv.
- **Koderna är text.** `kommun_kod` och `lan_kod` har inledande nolla
  (`0117`, `01`). Läs dem som text, annars blir de `117` och `1`.
- **Punkt som decimaltecken, tom cell = saknas.** Ett program med svensk
  inställning väntar sig decimalkomma och läser `85.2` som text eller datum.
  Ange engelsk inställning för talen när du räknar på dem.

Recepten nedan läser historik.csv från sajten, där filen heter
`vardcentraler-historik.csv`. Byt adress för de andra filerna
(`vardcentraler.csv` är units.csv). Kontroll efter inläsning: alla rader har
lika många kolumner som rubrikraden, och `Berga Läkarhus, Helsingborg` ligger
i en cell i kolumnen `name`.

### Power Query (Excel och Power BI)

Guiden under Hämta data kan skapa steget med `QuoteStyle.None`, som delar på
varje kommatecken. Öppna Avancerad redigerare och ersätt frågan med:

```powerquery
let
    Kalla = Csv.Document(
        Web.Contents("https://vardbetyg.se/data/vardcentraler-historik.csv"),
        [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
    ),
    Rubriker = Table.PromoteHeaders(Kalla, [PromoteAllScalars = true]),
    Koder = Table.TransformColumnTypes(
        Rubriker,
        {{"kommun_kod", type text}, {"lan_kod", type text}}
    ),
    Tal = Table.TransformColumnTypes(
        Koder,
        {{"ar", Int64.Type}, {"answered", Int64.Type}, {"response_rate", type number}},
        "en-US"
    )
in
    Tal
```

`65001` är UTF-8 och `QuoteStyle.Csv` läser citerade fält som en cell. Lägg
fler talkolumner i sista steget på samma sätt, med kulturen `en-US`. Låt inte
Power Query lägga till ett eget steg "Ändrad typ" före `Koder`: det gör koderna
till heltal och nollan försvinner.

### Excel utan Power Query-kod

Dubbelklicka inte på filen. Excel gissar då teckenkodningen och, med svenskt
språk, semikolon som avgränsare, så allt hamnar i en kolumn.

1. Data > Från text/CSV, och välj den nedladdade filen.
2. Filursprung: `65001: Unicode (UTF-8)`. Avgränsare: Komma.
3. Identifiering av datatyp: `Identifiera inte datatyper`. Då behåller
   `kommun_kod` och `lan_kod` nollan.
4. Läs in. Vill du räkna på talen väljer du Transformera data i stället och
   ställer in typen på talkolumnerna med Använd språkinställning > English
   (United States), eller använder receptet ovan.

### R

```r
library(readr)

historik <- read_csv(
  "https://vardbetyg.se/data/vardcentraler-historik.csv",
  col_types = cols(kommun_kod = col_character(), lan_kod = col_character())
)
problems(historik)  # tom om alla rader lästes rätt
```

`read_csv` läser UTF-8, punkt som decimaltecken och citerade fält som standard.
Kolumner du inte nämner i `col_types` gissas.

### Python

```python
import pandas as pd

historik = pd.read_csv(
    "https://vardbetyg.se/data/vardcentraler-historik.csv",
    dtype={"kommun_kod": str, "lan_kod": str},
    encoding="utf-8",
)
```

`pandas` läser citerade fält och punkt som decimaltecken som standard. Tomma
celler blir `NaN`.

## units.csv

En rad per vårdcentral med minst ett redovisat enkätresultat.

| Kolumn | Beskrivning | Källa |
|--------|-------------|-------|
| `slug` | URL-identifierare, matchar vardbetyg.se/vardcentral/&lt;slug&gt;. | Vårdbetyg |
| `name` | Vårdcentralens namn. | NPE |
| `kommun` | Kommun, med SCB:s stavning. | NPE |
| `region` | Region. | NPE |
| `hsa_id` | HSA-id, om känt (nationell enhetsidentifierare). | NPE |
| `measurement_period` | Mätperiod för enkäten. | NPE |
| `vardbetyg` | Publicerat Vårdbetyg 0-100 (composite efter shrinkage). Tomt = för få svar. | Vårdbetyg (härlett) |
| `vardbetyg_raw_composite` | Patientupplevelse-composite före shrinkage. | Vårdbetyg (härlett) |
| `vardbetyg_prior` | Nationell referens (rikssnittet) som betyget krymps mot. | Vårdbetyg (härlett) |
| `patientupplevelse_score` | Kategoripoäng patientupplevelse (NPE-composite efter shrinkage). | Vårdbetyg (härlett) |
| `tillganglighet_score` | Kategoripoäng tillgänglighet (väntetider). Tomt = ingen väntetidsdata. | Vårdbetyg (härlett) |
| `invited` | Antal patienter som fick enkäten (tillfrågade). | NPE |
| `answered` | Antal svarande patienter (underlaget för betyget). | NPE |
| `response_rate` | Svarsfrekvens i procent. | NPE |
| `for_fa_svar` | `true` om enheten hade för få svar för att redovisa ett Vårdbetyg. | Vårdbetyg (härlett) |
| `vardbetyg_fotnot` | Fotnotskod för betyget: F01 (för få svar) eller F06 (betyget bygger på 1 av 2 källor). Tomt = betyget bygger på båda källorna. | Vårdbetyg (härlett) |
| `tillganglighet_fotnot` | Fotnotskod när väntetider saknas: F03 (regionen byter journalsystem enligt Socialstyrelsen) eller F04 (ingen angiven orsak). Tomt = väntetider finns. | Socialstyrelsen |
| `schema_version` | Exportschemats version, se Ändringslogg. | Vårdbetyg |
| `huvudman` | `region`, `privat` eller `okänd`: vem som driver vårdcentralen enligt NPE, samma mätår som raden. Se Om huvudman nedan. | NPE |
| `kedja` | Kedjan eller varumärket i vårdcentralens namn (Capio, Kry, Närhälsan med flera). Tomt = namnet bär ingen kedja i tabellen, inte att enheten är fristående. | Vårdbetyg (härlett) |
| `vardgivare` | Bolaget som driver vårdcentralen, med namnet hos Bolagsverket. Tomt = ingen öppen källa anger bolaget (alltid tomt för regiondrivna). | VGR/Vårdbetyg + Bolagsverket |
| `vardgivare_orgnr` | Bolagets organisationsnummer (NNNNNN-NNNN). | Bolagsverket |
| `vardgivare_kalla` | `vgr` (regionens öppna vårdvalsdata) eller `vardbetyg` (vår egen kontroll mot två oberoende uppgifter). | Vårdbetyg |
| `kommun_kod` | SCB:s fyrsiffriga kommunkod som text med inledande nolla, t.ex. `0117`. Läs kolumnen som text. Se Om kommunkod nedan. | SCB |
| `lan_kod` | SCB:s tvåsiffriga länskod, de två första siffrorna i kommunkoden (`01` = Stockholms län). | SCB |
| `dim:Vård och behandling` | Dimensionspoäng 0-100 för "Vård och behandling" (Nationell Patientenkät, 2025). | NPE |
| `dim_ci:Vård och behandling` | Konfidensintervall (± enheter) för "Vård och behandling". | NPE |
| `dim:Bemötande` | Dimensionspoäng 0-100 för "Bemötande" (Nationell Patientenkät, 2025). | NPE |
| `dim_ci:Bemötande` | Konfidensintervall (± enheter) för "Bemötande". | NPE |
| `dim:Delaktighet` | Dimensionspoäng 0-100 för "Delaktighet" (Nationell Patientenkät, 2025). | NPE |
| `dim_ci:Delaktighet` | Konfidensintervall (± enheter) för "Delaktighet". | NPE |
| `dim:Information` | Dimensionspoäng 0-100 för "Information" (Nationell Patientenkät, 2025). | NPE |
| `dim_ci:Information` | Konfidensintervall (± enheter) för "Information". | NPE |
| `dim:Kontinuitet` | Dimensionspoäng 0-100 för "Kontinuitet" (Nationell Patientenkät, 2025). | NPE |
| `dim_ci:Kontinuitet` | Konfidensintervall (± enheter) för "Kontinuitet". | NPE |
| `dim:Tillgänglighet` | Dimensionspoäng 0-100 för "Tillgänglighet" (Nationell Patientenkät, 2025). | NPE |
| `dim_ci:Tillgänglighet` | Konfidensintervall (± enheter) för "Tillgänglighet". | NPE |

### Om huvudman, kedja och vårdgivare

Fälten är beskrivande. De säger vem som driver vårdcentralen, inget om
vårdens kvalitet. Patienterna skiljer sig mellan regiondrivna och privata
vårdcentraler (ålder, sjuklighet, var i landet de ligger), så en skillnad i
snittbetyg mellan driftformerna säger inte att den ena vården är bättre.
Vårdbetyg jämför därför aldrig betyg mellan driftformer och publicerar inga
sådana fördelningar.

`huvudman` kommer ur NPE, som skrev "Landsting/Region" till och med 2021 och
"Region" därefter. Regionägda bolag räknas som det NPE anger. `kedja` läses ur
namnet med en granskad tabell; Praktikertjänst och andra som sällan har
bolagsnamnet i mottagningens namn får därför tomt på de flesta rader.
`vardgivare` finns bara där en öppen källa anger bolaget. Övriga regioners
listor saknar publicerad licens och används inte. Koncern (moderbolag) finns
inte i någon öppen källa och ingår inte.

## kommuner.csv

En rad per kommun, aggregat över kommunens vårdcentraler.

| Kolumn | Beskrivning | Källa |
|--------|-------------|-------|
| `slug` | URL-identifierare, matchar vardbetyg.se/kommun/&lt;slug&gt;. | Vårdbetyg |
| `name` | Kommunens namn. | NPE |
| `region` | Region kommunen tillhör. | NPE |
| `unit_count` | Antal öppna vårdcentraler i kommunen (katalogen; även enheter utan enkätresultat, t.ex. Sörmland vars region inte deltog i mätningen). | Vårdbetyg |
| `scored_count` | Antal med ett publicerat Vårdbetyg. | Vårdbetyg (härlett) |
| `for_fa_svar_count` | Antal med för få svar för att redovisa betyg. | Vårdbetyg (härlett) |
| `avg_vardbetyg` | Medelvärde av kommunens publicerade Vårdbetyg. | Vårdbetyg (härlett) |
| `best_vardbetyg` | Högsta Vårdbetyg i kommunen. | Vårdbetyg (härlett) |
| `total_answered` | Summa svarande patienter i kommunen. | NPE |
| `kommun_kod` | SCB:s fyrsiffriga kommunkod som text med inledande nolla, t.ex. `0117`. Läs kolumnen som text. Se Om kommunkod nedan. | SCB |
| `lan_kod` | SCB:s tvåsiffriga länskod, de två första siffrorna i kommunkoden (`01` = Stockholms län). | SCB |

## fragor.csv

Frågenivån: en rad per vårdcentral och betygsatt enkätfråga (2025,
mätåret per rad står i units.csv). Samma
öppna data som visas fråga för fråga på enhetssidan. Fördjupning, aldrig en
betygskomponent, så frågepoängen räknas inte in i `vardbetyg` i units.csv.

| Kolumn | Beskrivning | Källa |
|--------|-------------|-------|
| `slug` | URL-identifierare, matchar vardbetyg.se/vardcentral/&lt;slug&gt;. | Vårdbetyg |
| `dimension` | Dimensionen frågan hör till, matchar `dim:&lt;namn&gt;` i units.csv. | NPE |
| `question_key` | Stabil nyckel för frågan (oförändrad mellan mätår). | Vårdbetyg |
| `question` | Frågans fullständiga lydelse (svenska), enligt patientenkäten. | NPE |
| `score` | Frågepoäng 0-100 (samma skala som dimensionerna). Tomt = för få svar. | NPE |
| `confidence_interval` | Konfidensintervall (± enheter) för poängen. | NPE |
| `positive_share` | Andel positiva svar 0-100. Tomt = för få svar. | NPE |
| `respondents` | Antal som svarade på just den här frågan. | NPE |
| `status` | `ok` eller `for_fa_svar` (för få svar på frågan). | NPE |

## utmarkelser.csv

De utmärkta vårdcentralerna: en rad per utmärkelse, samma utmärkelser som
enhetssidan och topplistorna visar. Utmärkelserna fryses när mätningen
släpps och gäller hela utmärkelseåret. De är gratis, kan inte köpas och
påverkar aldrig betyget. En nedlagd vårdcentral finns inte med.

| Kolumn | Beskrivning | Källa |
|--------|-------------|-------|
| `utmarkelsear` | Utmärkelseåret (mätåret plus ett). | Vårdbetyg |
| `matar` | Mätåret i Nationell Patientenkät som utmärkelsen bygger på. | NPE |
| `utmarkelse` | Utmärkelsen som den står på sajten, t.ex. "Topp 10 % i Sverige 2026". | Vårdbetyg (härlett) |
| `typ` | `national-top` (topp 10 % i Sverige), `kommun-winner` (topprankad i kommunen) eller `most-improved` (Årets klättrare). | Vårdbetyg (härlett) |
| `slug` | URL-identifierare, matchar vardbetyg.se/vardcentral/&lt;slug&gt; och `slug` i units.csv. | Vårdbetyg |
| `name` | Vårdcentralens namn när utmärkelserna frystes. | NPE |
| `hsa_id` | HSA-id, om känt. | NPE |
| `utmarkelse_kommun` | Kommunen i titeln, bara för `kommun-winner`. | NPE |
| `klattring_forsta_ar` | Årets klättrare: första mätåret i jämförelsen. | NPE |
| `klattring_sista_ar` | Årets klättrare: sista mätåret i jämförelsen. | NPE |
| `klattring_perioder` | Årets klättrare: antal mätår jämförelsen vilar på. | NPE |
| `klattring_delta` | Årets klättrare: vårdcentralens förändring i poäng, sista minus första året. | Vårdbetyg (härlett) |
| `klattring_riks_delta` | Årets klättrare: rikets genomsnittliga förändring i poäng över samma år. | Vårdbetyg (härlett) |
| `frusen` | Datum då utmärkelserna frystes. | Vårdbetyg |

## historik.csv

En rad per vårdcentral och mätår i Nationell Patientenkät, från 2015.
Sajtens adress: vardbetyg.se/data/vardcentraler-historik.csv.

**Seriebrottet.** NPE bytte enkät 2025. Raderna 2015-2024 kommer ur serien
"Primärvård (2015-2024)" och raderna från 2025 ur "Vårdcentraler och
hälsocentraler", med andra områden och frågor. Kolumnen `serie` säger vilken.
Jämför bara poäng inom samma serie; området "Tillgänglighet" finns i båda men
bygger på olika frågor. Mätningen görs vartannat år per region, så alla
regioner finns inte med varje år.

**Inget Vårdbetyg.** Dagens metod väger patientupplevelsen mot
tillgängligheten ur Socialstyrelsens väntetider, och de finns inte bakåt till
2015 i vår data. Betyget för den senaste mätningen står i units.csv.

**Nyckeln** är HSA-id så som NPE angav det det året. När en vårdcentral har
lagts ner, blivit filial eller slagits ihop står det i `nedlagd_fran` och
`efterfoljare_hsa_id` (Vårdbetygs register, med källa på enhetssidan). Byter
en vårdcentral HSA-id utan att det står i registret följer raderna inte med.

| Kolumn | Beskrivning | Källa |
|--------|-------------|-------|
| `ar` | Mätåret. Region Sörmland deltog inte 2025, se Sörmland nedan. | NPE |
| `serie` | Enkätserien raden kommer ur. | NPE |
| `hsa_id` | HSA-id det året. | NPE |
| `slug` | Samma slug som i units.csv när vårdcentralen är öppen i dag. | Vårdbetyg |
| `name` | Namnet det året. | NPE |
| `kommun` | Kommunen det året, med SCB:s stavning (se Om kommunkod nedan). | NPE |
| `region` | Regionen det året. | NPE |
| `huvudman` | `region`, `privat` eller `okänd` det året (2015 saknar NPE uppgiften för många enheter). | NPE |
| `kedja` | Kedjan i namnet det året, samma tabell som units.csv. | Vårdbetyg (härlett) |
| `status` | `ok` eller `for_fa_svar`. | NPE |
| `invited` | Antal som fick enkäten. | NPE |
| `answered` | Antal som svarade. | NPE |
| `response_rate` | Svarsfrekvens i procent. | NPE |
| `nedlagd_fran` | Första månaden som egen vårdcentral upphörde (ÅÅÅÅ-MM). | Vårdbetyg |
| `efterfoljare_hsa_id` | HSA-id för vårdcentralen verksamheten gick upp i. | Vårdbetyg |
| `kommun_kod` | SCB:s fyrsiffriga kommunkod som text med inledande nolla, t.ex. `0117`. Läs kolumnen som text. Se Om kommunkod nedan. | SCB |
| `lan_kod` | SCB:s tvåsiffriga länskod, de två första siffrorna i kommunkoden (`01` = Stockholms län). | SCB |
| `dim:Helhetsintryck` | Poäng 0-100 för "Helhetsintryck" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Helhetsintryck` | Konfidensintervall (± enheter) för "Helhetsintryck". | NPE |
| `dim:Emotionellt stöd` | Poäng 0-100 för "Emotionellt stöd" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Emotionellt stöd` | Konfidensintervall (± enheter) för "Emotionellt stöd". | NPE |
| `dim:Delaktighet och involvering` | Poäng 0-100 för "Delaktighet och involvering" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Delaktighet och involvering` | Konfidensintervall (± enheter) för "Delaktighet och involvering". | NPE |
| `dim:Respekt och bemötande` | Poäng 0-100 för "Respekt och bemötande" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Respekt och bemötande` | Konfidensintervall (± enheter) för "Respekt och bemötande". | NPE |
| `dim:Kontinuitet och koordinering` | Poäng 0-100 för "Kontinuitet och koordinering" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Kontinuitet och koordinering` | Konfidensintervall (± enheter) för "Kontinuitet och koordinering". | NPE |
| `dim:Information och kunskap` | Poäng 0-100 för "Information och kunskap" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Information och kunskap` | Konfidensintervall (± enheter) för "Information och kunskap". | NPE |
| `dim:Tillgänglighet` | Poäng 0-100 för "Tillgänglighet" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Tillgänglighet` | Konfidensintervall (± enheter) för "Tillgänglighet". | NPE |
| `dim:Vård och behandling` | Poäng 0-100 för "Vård och behandling" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Vård och behandling` | Konfidensintervall (± enheter) för "Vård och behandling". | NPE |
| `dim:Delaktighet` | Poäng 0-100 för "Delaktighet" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Delaktighet` | Konfidensintervall (± enheter) för "Delaktighet". | NPE |
| `dim:Bemötande` | Poäng 0-100 för "Bemötande" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Bemötande` | Konfidensintervall (± enheter) för "Bemötande". | NPE |
| `dim:Kontinuitet` | Poäng 0-100 för "Kontinuitet" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Kontinuitet` | Konfidensintervall (± enheter) för "Kontinuitet". | NPE |
| `dim:Information` | Poäng 0-100 för "Information" det året. Tomt de år serien saknar området. | NPE |
| `dim_ci:Information` | Konfidensintervall (± enheter) för "Information". | NPE |

### Om kommunkod och kommunnamn

`kommun_kod` och `lan_kod` är SCB:s koder. De är text, inte tal: kommunerna
i Stockholms län börjar på 0 (`0117`), och ett kalkylprogram som tolkar kolumnen
som tal tappar nollan. Länskoden är de två första siffrorna i kommunkoden.
Listan över kommuner är SCB:s (290 kommuner, tabell BE0101A i statistikdatabasen,
CC0), kopplad till kommunen på namnet. Varje rad har en kod; exporten och ett test
stoppar om ett namn inte finns i SCB:s lista.

NPE har stavat fyra kommuner fel i äldre mätår: `Upplands Bro` (SCB:
`Upplands-Bro`), `Upplands-Väsby` (`Upplands Väsby`), `Malung-sälen`
(`Malung-Sälen`) och `Solentuna` (`Sollentuna`). I historik.csv står SCB:s
stavning på alla rader. Före schemaversion 3.1 gav de fyra felstavningarna 294
olika kommunnamn i filen i stället för 290.

### Sörmland

Region Sörmland deltog inte i Nationell Patientenkät 2025. Sörmlands
vårdcentraler har därför rader i historik.csv bara till och med 2023 (Sörmland
mättes 2015, 2017, 2019, 2021 och 2023) och saknas i units.csv, som tar med
vårdcentraler med enkätresultat i den senaste mätningen. De finns med i
kommuner.csv (`unit_count`). Det är en uppgift om källan, inget fel i filerna.

## Fotnoter

Kodlistan finns också som egen fil, fotnoter.csv (vardbetyg.se/data/fotnoter.csv).
En kod byter aldrig betydelse; en ny orsak får en ny kod och en ny schemaversion.
F02 och F05 förekommer i enheternas data.json men inte i units.csv, som bara tar
med öppna vårdcentraler med enkätresultat.

| Kod | Etikett | Betydelse | Källa |
|-----|---------|-----------|-------|
| `F01` | För få svar | Mottagningen ingick i Nationell Patientenkät, men för få patienter svarade för att ett betyg ska kunna visas. | Nationell Patientenkät |
| `F02` | Ej rapporterat, ingick inte i mätningen | Mottagningen, eller hela regionen, fanns inte med i Nationell Patientenkätens urval för mätåret. | Nationell Patientenkät |
| `F03` | Ej rapporterat, regionen byter journalsystem | Socialstyrelsens väntetider har inga uppgifter för mottagningen, och Socialstyrelsen anger att regionen byter journalsystem, vilket kan påverka rapporteringen. | Socialstyrelsen |
| `F04` | Ej rapporterat | Socialstyrelsens väntetider har inga uppgifter för mottagningen, och källan anger ingen orsak. | Socialstyrelsen |
| `F05` | Ej tillämpligt | Mottagningen är nedlagd, en filial eller sammanslagen med en annan, så aktuella värden gäller inte. | Vårdbetygs register över nedlagda mottagningar |
| `F06` | Betyget bygger på 1 av 2 källor | Väntetider saknas, så betyget räknas på patienternas svar enbart. Vikterna fördelas om över den källa som har data. Inget saknat värde fylls i med en gissning. | Vårdbetyg |

## Ändringslogg

- **3.1** (2026-10-27): Kommunkod och länskod: nya kolumnerna kommun_kod (SCB:s fyrsiffriga kommunkod som text med inledande nolla, t.ex. 0117) och lan_kod (SCB:s tvåsiffriga länskod) sist bland de fasta kolumnerna i units.csv, kommuner.csv och historik.csv. Kommunnamnen i historik.csv följer nu SCB:s stavning: fyra felstavningar från äldre mätår i NPE (Upplands Bro, Upplands-Väsby, Malung-sälen, Solentuna) är rättade, så filen har 290 kommuner och inte 294. Region Sörmland deltog inte i NPE 2025, så Sörmlands vårdcentraler har rader i historik.csv bara till och med 2023.
- **3.0** (2026-10-27): Vem som driver vårdcentralen i units.csv: huvudman (region, privat eller okänd, ur Nationell Patientenkät), kedja (varumärket i namnet) samt vardgivare, vardgivare_orgnr och vardgivare_kalla där en öppen källa anger bolaget. Ny fil historik.csv med en rad per vårdcentral och mätår 2015 och framåt. Fälten är beskrivande, Vårdbetyg jämför inte driftformer.
- **2.0** (2026-10-14): Fotnotskoder för saknade värden (F01 till F06) i units.csv (kolumnerna vardbetyg_fotnot och tillganglighet_fotnot) och i varje enhets data.json, kodlistan som egen fil (fotnoter.csv) och kolumnen schema_version.
- **1.0** (2026-07-10): units.csv och kommuner.csv på sajten. fragor.csv (frågenivån) lades till 2026-07-30 utan ny version.
