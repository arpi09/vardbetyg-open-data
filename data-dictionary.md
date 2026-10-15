# Datakatalog (data dictionary)

Kolumndefinitioner för de exporterade CSV-filerna. Genererad av
`scripts/export-open-data.ts` (via src/lib/etl/open-data.ts) - redigera koden,
inte den här filen för hand.

Talvärden använder punkt som decimaltecken (maskinläsbart), tomma celler
betyder "saknat värde". Mätperiod: Nationell Patientenkät 2025.
Schemaversion: **2.0** (kolumnen `schema_version` i units.csv).

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
- **Vårdbetyg (härlett)** - poäng som beräknas av sajten ur ovanstående
  öppna källor. Metod: <https://vardbetyg.se/metodik>.

Vårdbetyg är oberoende och inte affilierat med 1177, SKR eller Socialstyrelsen.

## Så citerar du

Datan är öppen under CC BY 4.0. Använd den fritt och ange var den kommer
ifrån.

En kort källrad, för en artikel eller ett diagram:

> Källa: Vårdbetyg (vardbetyg.se), CC BY 4.0.

Referensen till datasetet, för en rapport eller en uppsats:

> Vårdbetyg. (2026). Vårdbetyg: öppet dataset för svenska vårdcentraler, Nationell Patientenkät 2025 (version 2025.1) [Dataset]. https://github.com/arpi09/vardbetyg-open-data

Repot har en CITATION.cff, så GitHub visar referensen i flera format under
"Cite this repository" (https://github.com/arpi09/vardbetyg-open-data). Som BibTeX:

```bibtex
@misc{vardbetyg20251,
  author = {{Vårdbetyg}},
  title = {Vårdbetyg: öppet dataset för svenska vårdcentraler, Nationell Patientenkät 2025},
  year = {2026},
  version = {2025.1},
  url = {https://github.com/arpi09/vardbetyg-open-data},
  note = {Dataset, CC BY 4.0}
}
```

## units.csv

En rad per vårdcentral med minst ett redovisat enkätresultat.

| Kolumn | Beskrivning | Källa |
|--------|-------------|-------|
| `slug` | URL-identifierare, matchar vardbetyg.se/vardcentral/&lt;slug&gt;. | Vårdbetyg |
| `name` | Vårdcentralens namn. | NPE |
| `kommun` | Kommun. | NPE |
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

## fragor.csv

Frågenivån: en rad per vårdcentral och betygsatt enkätfråga (2025). Samma
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

- **2.0** (2026-10-14): Fotnotskoder för saknade värden (F01 till F06) i units.csv (kolumnerna vardbetyg_fotnot och tillganglighet_fotnot) och i varje enhets data.json, kodlistan som egen fil (fotnoter.csv) och kolumnen schema_version.
- **1.0** (2026-07-10): units.csv och kommuner.csv på sajten. fragor.csv (frågenivån) lades till 2026-07-30 utan ny version.
