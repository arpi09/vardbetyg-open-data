# Kodbok

Varje fält i den öppna exporten: vad det betyder, var det kommer ifrån,
vilken period det gäller och hur ett saknat värde anges. Genererad av
`scripts/export-open-data.ts` i vardbetyg-repot
(src/lib/domain/oppen-data/kodbok.ts). Samma innehåll finns maskinläsbart
i kodbok.csv.

## Gäller alla filer

- Teckenkodning UTF-8, kommatecken mellan fälten och CRLF som radslut.
- Decimaltal skrivs med punkt och avrundas till en decimal, halvor uppåt.
- Ett saknat värde är en tom cell, aldrig 0 och aldrig en gissning. Varför
  ett värde saknas står i fältets beskrivning nedan, och för betyget och
  väntetiderna också som en fotnotskod (fotnoter.csv).
- slug är nyckeln mellan filerna och sajten.
- Väntetidernas antal (antal ärenden per månad), Google-data, fritext från
  1177 och uppgifter ur Vården i siffror ingår inte.

## units.csv

En rad per vårdcentral med minst ett redovisat enkätresultat.

| Fält | Typ | Enhet | Definition | Källa | Period | Saknat värde |
|------|-----|-------|------------|-------|--------|--------------|
| `slug` | text |  | Vårdcentralens identifierare på sajten: vardbetyg.se/vardcentral/<slug>. Ändras inte när namnet ändras. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `name` | text |  | Vårdcentralens namn. Har en vårdcentral bytt namn eller driftsbolag efter mätningen står det nuvarande namnet. | Nationell Patientenkät, med Vårdbetygs register över namnbyten | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `kommun` | text |  | Kommunen vårdcentralen ligger i. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `region` | text |  | Regionen vårdcentralen ligger i. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `hsa_id` | text |  | HSA-id, den nationella identifieraren för vårdenheter. Skrivs som i patientenkäten, ibland med siffrorna 16 efter SE (SE16...) och ibland utan. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när källan saknar HSA-id. |
| `measurement_period` | text | år | Mätåret i patientenkäten som raden bygger på. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `vardbetyg` | decimaltal | poäng 0 till 100, en decimal | Det publicerade Vårdbetyget: 0,7 × patientupplevelse_score + 0,3 × tillganglighet_score. Saknas väntetider räknas betyget på patientupplevelsen enbart. Metoden: vardbetyg.se/metodik. | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. Väntetiderna: senaste månaden med ett värde per mått, inom oktober 2025 till augusti 2026. | Tom cell när färre än 10 patienter svarade (for_fa_svar = true, fotnot F01 i vardbetyg_fotnot). Aldrig 0. |
| `vardbetyg_raw_composite` | decimaltal | poäng 0 till 100, en decimal | Patientupplevelsen före krympning: medelvärdet av vårdcentralens poäng i enkätens områden, alla områden väger lika. | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. | Tom cell när vardbetyg saknas. |
| `vardbetyg_prior` | decimaltal | poäng 0 till 100, en decimal | Rikssnittet betyget krymps mot. Rikssnittet för patientupplevelsen är medelvärdet över områdena av alla vårdcentralers medel per område. Har vårdcentralen väntetider är värdet 0,7 × det rikssnittet + 0,3 × rikssnittet för tillgängligheten (medelvärdet av alla vårdcentralers tillgänglighet före krympning). | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. Väntetiderna: senaste månaden med ett värde per mått, inom oktober 2025 till augusti 2026. | Tom cell när vardbetyg saknas. |
| `patientupplevelse_score` | decimaltal | poäng 0 till 100, en decimal | Patientupplevelsen efter krympning mot rikssnittet: vikt × vardbetyg_raw_composite + (1 − vikt) × rikssnittet, där vikt = n / (n + 30) och n är antal svarande (answered). | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. | Tom cell när vardbetyg saknas. |
| `tillganglighet_score` | decimaltal | poäng 0 till 100, en decimal | Tillgängligheten efter krympning mot rikssnittet för tillgänglighet. Vårdcentralens eget värde är medelvärdet av två andelar: medicinsk bedömning inom 3 dagar och besvarade samtal. Krympningen använder antalet ärenden bakom andelarna, som inte ingår i den öppna datan. | Socialstyrelsen, väntetider i primärvården, härlett av Vårdbetyg | Den senaste månaden med ett värde per mått, inom oktober 2025 till augusti 2026. | Tom cell när Socialstyrelsen inte redovisar väntetider för vårdcentralen. Orsaken står i tillganglighet_fotnot (F03 eller F04). |
| `invited` | heltal | antal patienter | Antal patienter som fick enkäten. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när källan inte redovisar antalet. |
| `answered` | heltal | antal patienter | Antal patienter som svarade. Underlaget för betyget och för krympningen. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när källan inte redovisar antalet. |
| `response_rate` | decimaltal | procent | Andel av de tillfrågade som svarade. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när källan inte redovisar svarsfrekvensen. |
| `for_fa_svar` | sant/falskt |  | true när vårdcentralen har enkätresultat men för få svarande (färre än 10) för att ett Vårdbetyg ska redovisas. | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `vardbetyg_fotnot` | kod |  | Fotnot för betyget: F01 (för få svar) eller F06 (betyget bygger på 1 av 2 källor). Koderna står i fotnoter.csv. | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. Väntetiderna: senaste månaden med ett värde per mått, inom oktober 2025 till augusti 2026. | Tom cell när betyget bygger på båda källorna. |
| `tillganglighet_fotnot` | kod |  | Fotnot när väntetider saknas: F03 (regionen byter journalsystem enligt Socialstyrelsen) eller F04 (ingen angiven orsak). | Socialstyrelsen, väntetider i primärvården | Den senaste månaden med ett värde per mått, inom oktober 2025 till augusti 2026. | Tom cell när väntetider finns. |
| `schema_version` | text |  | Exportschemats version. Ändringarna står i data-dictionary.md. | Vårdbetyg | Gäller exporten. | Alltid ifyllt. |
| `dim:Vård och behandling` | decimaltal | poäng 0 till 100, en decimal | Vårdcentralens poäng i området "Vård och behandling" i patientenkäten. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när för få svarade på området. |
| `dim_ci:Vård och behandling` | decimaltal | poäng, plus minus | Felmarginalen (95 % konfidensintervall, ± poäng) för området "Vård och behandling". | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när poängen saknas. |
| `dim:Bemötande` | decimaltal | poäng 0 till 100, en decimal | Vårdcentralens poäng i området "Bemötande" i patientenkäten. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när för få svarade på området. |
| `dim_ci:Bemötande` | decimaltal | poäng, plus minus | Felmarginalen (95 % konfidensintervall, ± poäng) för området "Bemötande". | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när poängen saknas. |
| `dim:Delaktighet` | decimaltal | poäng 0 till 100, en decimal | Vårdcentralens poäng i området "Delaktighet" i patientenkäten. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när för få svarade på området. |
| `dim_ci:Delaktighet` | decimaltal | poäng, plus minus | Felmarginalen (95 % konfidensintervall, ± poäng) för området "Delaktighet". | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när poängen saknas. |
| `dim:Information` | decimaltal | poäng 0 till 100, en decimal | Vårdcentralens poäng i området "Information" i patientenkäten. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när för få svarade på området. |
| `dim_ci:Information` | decimaltal | poäng, plus minus | Felmarginalen (95 % konfidensintervall, ± poäng) för området "Information". | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när poängen saknas. |
| `dim:Kontinuitet` | decimaltal | poäng 0 till 100, en decimal | Vårdcentralens poäng i området "Kontinuitet" i patientenkäten. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när för få svarade på området. |
| `dim_ci:Kontinuitet` | decimaltal | poäng, plus minus | Felmarginalen (95 % konfidensintervall, ± poäng) för området "Kontinuitet". | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när poängen saknas. |
| `dim:Tillgänglighet` | decimaltal | poäng 0 till 100, en decimal | Vårdcentralens poäng i området "Tillgänglighet" i patientenkäten. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när för få svarade på området. |
| `dim_ci:Tillgänglighet` | decimaltal | poäng, plus minus | Felmarginalen (95 % konfidensintervall, ± poäng) för området "Tillgänglighet". | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när poängen saknas. |

## kommuner.csv

En rad per kommun.

| Fält | Typ | Enhet | Definition | Källa | Period | Saknat värde |
|------|-----|-------|------------|-------|--------|--------------|
| `slug` | text |  | Kommunens identifierare på sajten: vardbetyg.se/kommun/<slug>. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `name` | text |  | Kommunens namn. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `region` | text |  | Regionen kommunen hör till. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `unit_count` | heltal | antal vårdcentraler | Antal öppna vårdcentraler i kommunen, även de utan enkätresultat (till exempel i en region som inte deltog i mätningen). | Vårdbetyg | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `scored_count` | heltal | antal vårdcentraler | Antal vårdcentraler i kommunen med ett publicerat Vårdbetyg. | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. Väntetiderna: senaste månaden med ett värde per mått, inom oktober 2025 till augusti 2026. | Alltid ifyllt. |
| `for_fa_svar_count` | heltal | antal vårdcentraler | Antal vårdcentraler i kommunen med för få svar för ett betyg. | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `avg_vardbetyg` | decimaltal | poäng 0 till 100, en decimal | Medelvärdet av kommunens publicerade Vårdbetyg, alla vårdcentraler väger lika. | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. Väntetiderna: senaste månaden med ett värde per mått, inom oktober 2025 till augusti 2026. | Tom cell när ingen vårdcentral i kommunen har ett betyg. |
| `best_vardbetyg` | decimaltal | poäng 0 till 100, en decimal | Det högsta publicerade Vårdbetyget i kommunen. | Vårdbetyg, härlett ur de öppna källorna | Nationell Patientenkät 2025. Väntetiderna: senaste månaden med ett värde per mått, inom oktober 2025 till augusti 2026. | Tom cell när ingen vårdcentral i kommunen har ett betyg. |
| `total_answered` | heltal | antal patienter | Summan av svarande patienter i kommunens vårdcentraler. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt, 0 när ingen vårdcentral har enkätresultat. |

## fragor.csv

En rad per vårdcentral och redovisad enkätfråga.

| Fält | Typ | Enhet | Definition | Källa | Period | Saknat värde |
|------|-----|-------|------------|-------|--------|--------------|
| `slug` | text |  | Vårdcentralens identifierare, samma som slug i units.csv. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `dimension` | text |  | Området frågan hör till, samma namn som i kolumnerna dim:<område> i units.csv. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `question_key` | kod |  | Frågans nyckel. Densamma mellan mätår när frågan är densamma. | Vårdbetyg | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `question` | text |  | Frågan som den ställdes i patientenkäten, på svenska. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt. |
| `score` | decimaltal | poäng 0 till 100, en decimal | Frågans poäng, samma skala som områdena. Ingår inte i Vårdbetyget. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när för få svarade på frågan (status for_fa_svar). |
| `confidence_interval` | decimaltal | poäng, plus minus | Felmarginalen (95 % konfidensintervall, ± poäng) för frågans poäng. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när poängen saknas. |
| `positive_share` | decimaltal | procent | Andel positiva svar på frågan. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när för få svarade på frågan (status for_fa_svar). |
| `respondents` | heltal | antal patienter | Antal som svarade på just den här frågan. | Nationell Patientenkät | Nationell Patientenkät 2025. | Tom cell när källan inte redovisar antalet. |
| `status` | kod |  | ok när frågan redovisas, for_fa_svar när för få svarade på frågan. | Nationell Patientenkät | Nationell Patientenkät 2025. | Alltid ifyllt. |

## fotnoter.csv

En rad per fotnotskod.

| Fält | Typ | Enhet | Definition | Källa | Period | Saknat värde |
|------|-----|-------|------------|-------|--------|--------------|
| `kod` | kod |  | Fotnotskoden, F01 till F06. En kod byter aldrig betydelse. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `tillstand` | kod |  | Vilket slags frånvaro koden beskriver, som maskinläsbar nyckel. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `falt` | kod |  | Fältet koden gäller. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `kalla` | text |  | Källan som avgör om koden gäller. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `etikett_sv` | text |  | Kort etikett på svenska. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `text_sv` | text |  | Förklaringen på svenska. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `etikett_en` | text |  | Kort etikett på engelska. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `text_en` | text |  | Förklaringen på engelska. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `schema_version` | text |  | Exportschemats version när kodlistan gavs ut. | Vårdbetyg | Gäller exporten. | Alltid ifyllt. |

## utmarkelser.csv

En rad per utmärkelse.

| Fält | Typ | Enhet | Definition | Källa | Period | Saknat värde |
|------|-----|-------|------------|-------|--------|--------------|
| `utmarkelsear` | text | år | Utmärkelseåret, mätåret plus ett. | Vårdbetyg | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Alltid ifyllt. |
| `matar` | text | år | Mätåret i patientenkäten som utmärkelsen bygger på. | Nationell Patientenkät | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Alltid ifyllt. |
| `utmarkelse` | text |  | Utmärkelsen som den står på sajten, till exempel "Topp 10 % i Sverige 2026". | Vårdbetyg, härlett ur de öppna källorna | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Alltid ifyllt. |
| `typ` | kod |  | national-top: bland de 10 % högsta betygen i Sverige. kommun-winner: högst betyg i en kommun med minst tre vårdcentraler med betyg. most-improved: Årets klättrare, patienternas svar blev bättre över tid och mer än i riket. Alla kräver minst 30 svarande. Reglerna: vardbetyg.se/metodik. | Vårdbetyg, härlett ur de öppna källorna | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Alltid ifyllt. |
| `slug` | text |  | Vårdcentralens identifierare, samma som slug i units.csv. | Vårdbetyg | Gäller tills vidare. | Alltid ifyllt. |
| `name` | text |  | Vårdcentralens namn när utmärkelserna frystes. | Nationell Patientenkät | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Alltid ifyllt. |
| `hsa_id` | text |  | HSA-id när utmärkelserna frystes. | Nationell Patientenkät | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Tom cell när källan saknar HSA-id. |
| `utmarkelse_kommun` | text |  | Kommunen i titeln för kommun-winner. | Nationell Patientenkät | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Tom cell för de andra typerna. |
| `klattring_forsta_ar` | heltal | år | Årets klättrare: första mätåret i jämförelsen. | Nationell Patientenkät | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Tom cell för de andra typerna. |
| `klattring_sista_ar` | heltal | år | Årets klättrare: sista mätåret i jämförelsen. | Nationell Patientenkät | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Tom cell för de andra typerna. |
| `klattring_perioder` | heltal | antal mätår | Årets klättrare: antal mätår jämförelsen vilar på. | Nationell Patientenkät | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Tom cell för de andra typerna. |
| `klattring_delta` | decimaltal | poäng, en decimal | Årets klättrare: vårdcentralens förändring, sista minus första året. | Vårdbetyg, härlett ur de öppna källorna | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Tom cell för de andra typerna. |
| `klattring_riks_delta` | decimaltal | poäng, en decimal | Årets klättrare: rikets genomsnittliga förändring över samma år. | Vårdbetyg, härlett ur de öppna källorna | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Tom cell för de andra typerna. |
| `frusen` | datum | ÅÅÅÅ-MM-DD | Datumet då utmärkelserna frystes. | Vårdbetyg | Utmärkelseåret 2026, fryst 2026-08-25, byggt på Nationell Patientenkät 2025. | Alltid ifyllt. |
