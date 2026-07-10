# Datakatalog (data dictionary)

Kolumndefinitioner för de exporterade CSV-filerna. Genererad av
`scripts/export-open-data.ts` (via src/lib/etl/open-data.ts) - redigera koden,
inte den här filen för hand.

Talvärden använder punkt som decimaltecken (maskinläsbart), tomma celler
betyder "saknat värde". Mätperiod: Nationell Patientenkät 2025.

## Proveniens

- **NPE** - Nationell Patientenkät, öppen publik data per vårdcentral
  (patientupplevelse-dimensioner, svarsantal, svarsfrekvens).
- **Socialstyrelsen** - väntetider/tillgänglighet per vårdcentral.
- **Vårdbetyg (härlett)** - poäng som beräknas av sajten ur ovanstående
  öppna källor. Metod: <https://vardbetyg.se/metodik>.

Vårdbetyg är oberoende och inte affilierat med 1177, SKR eller Socialstyrelsen.

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
| `answered` | Antal svarande patienter (underlaget för betyget). | NPE |
| `response_rate` | Svarsfrekvens i procent. | NPE |
| `for_fa_svar` | `true` om enheten hade för få svar för att redovisa ett Vårdbetyg. | Vårdbetyg (härlett) |
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
| `unit_count` | Antal vårdcentraler i kommunen (med enkätresultat). | NPE |
| `scored_count` | Antal med ett publicerat Vårdbetyg. | Vårdbetyg (härlett) |
| `for_fa_svar_count` | Antal med för få svar för att redovisa betyg. | Vårdbetyg (härlett) |
| `avg_vardbetyg` | Medelvärde av kommunens publicerade Vårdbetyg. | Vårdbetyg (härlett) |
| `best_vardbetyg` | Högsta Vårdbetyg i kommunen. | Vårdbetyg (härlett) |
| `total_answered` | Summa svarande patienter i kommunen. | NPE |
