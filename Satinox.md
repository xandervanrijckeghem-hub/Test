# Satinox — SAP & Power BI-context Sadel + Atinox

Referentiedocument voor de platenanalyse van de groep: **Sadel NV** (moederbedrijf, België) en
**Atinox** (dochterbedrijf, Frankrijk). Sadel en Atinox verkopen buizen, fittingen, platen en staf.
Dit document vat samen met welk ERP-systeem en welke data gewerkt wordt, hoe het Power BI-model
is opgebouwd, en welke bedrijfsregels en valkuilen gelden. Laatste update: 24-09-2026.

Volledige kolomlijsten van `OINV`/`INV1` staan in `claude/atinox-sap-systeem-context.md`.
De oudere Sadel-context (inkoopzijde, werkbestanden, prijstool) staat in `sap-systeem-context.md`.

---

## 1. Systeem en verbinding

- **SAP Business One** (SAP B1), met rechtstreekse SQL-toegang op de onderliggende tabellen.
- Sadel en Atinox draaien op **dezelfde SQL Server** maar in **aparte company databases**, met een
  identiek tabelschema.

| Parameter | Waarde |
|---|---|
| Server | `192.168.1.13` |
| Database Sadel | `SBO_SADEL_PROD` |
| Database Atinox | `SBO_ATINOX_FR` |
| Login Power BI | SQL Server Authentication |
| Startdatum data in model | parameter `StartDatum` = `2025-01-01` |

### Rechtstreeks queryen (SSMS / Azure Data Studio)

- Server `192.168.1.13`, *SQL Server Authentication*, *Encrypt* = Mandatory **met
  "Trust Server Certificate" aangevinkt** (anders certificaatfout; alternatief: Encrypt = Optional).
- Wisselen tussen bedrijven: `USE SBO_ATINOX_FR;` of volledige naam `SBO_ATINOX_FR.dbo.OINV`.
- **Open punt (24-09-2026):** login met gebruiker `Power BI` gaf fout **18456 (Login failed)**.
  Nakijken in Power BI via *Bestand → Opties en instellingen → Instellingen voor gegevensbron →
  192.168.1.13 → Machtigingen bewerken* welk logintype (Windows/Database) en welke exacte
  gebruikersnaam Power BI gebruikt. Anders IT/SAP-partner vragen met het Connection Id uit de fout.

---

## 2. Kerntabellen (verkoopzijde)

| Tabel | Inhoud |
|---|---|
| `OINV` / `INV1` | Verkoopfacturen (header / regels) |
| `ORIN` / `RIN1` | Creditnota's (header / regels) |
| `OQUT` / `QUT1` | Offertes (header / regels); `QUT1.TrgetEntry` ≠ 0 = omgezet in een order |
| `OCRD` | Klanten en leveranciers |
| `OITM` / `OITB` | Artikelen / artikelgroepen |
| `NNM1` | Reeksnamen (`SeriesName`) per `Series` |

Inkoopzijde (Sadel): `OPOR`/`POR1` (bestellingen) + `OPCH`/`PCH1` (facturen zonder bestelling).

### Spelling winstkolommen (valkuil)

| Tabel | Kolom | Niveau |
|---|---|---|
| `OINV` | `GrosProfit` | factuurtotaal |
| `INV1` | `GrssProfit` | per regel |

Omzet per regel = `INV1.LineTotal`. Bij twijfel altijd verifiëren via
`SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = '<tabel>'`.

### Kruisverwijzing Sadel ↔ Atinox

`OINV.U_OrderAtinox` en `OINV.U_OrderSadel` koppelen orders tussen beide bedrijven.

### Documentnummers (jaar + reeks)

| Bedrijf | Offerte | Verkooporder | Factuur |
|---|---|---|---|
| Sadel | `2670…` | `2671…` | `2674…` |
| Atinox | `2661…` | `2662…` | `2666…` |

Eerste 2 cijfers = jaar, volgende 2 = reeks. Inkoopreeksen Sadel: 61 = stockorder (geen TR);
64/65/67 = werkhypothese spoed/zonder bestelling (te bevestigen via `NNM1.SeriesName`).

---

## 3. Bedrijfsregels voor de platenanalyse

- **Plaat** = artikelnummer begint met `5` (`ItemCode LIKE '5%'`).
- **Order** = verkoopfactuur (`OINV`). Een order "bevat een plaat" als minstens één regel een plaat is.
- **Creditnota's** tellen negatief mee in "heel assortiment", niet in de analyses op orderniveau.
- **Kostprijsfout:** winst van een regel wordt 0 gezet als `LineTotal > 0` en
  `GrssProfit < −2 × LineTotal`. Vangt o.a. Atinox-orders 266603936 en 266602775 (klant KL00036).
- **Intercompany uitsluiten:** Sadel verkoopt aan Atinox als klant **`KA00262`**
  (± € 13,3 mln, 43 % van de Sadel-omzet jan–sep 2026). In het rapport uitgesloten via filter
  `Klant[KlantKey]` niet in `'Sadel|KA00262'`.
- **Enkel artikels, geen kosten:** kostenregels worden uit de analyse gehaald, ook al worden ze
  doorgerekend aan de klant. Uitgesloten artikelcodes (in de SQL van Verkoopregels én Offerteregels):

  | Soort | Uitgesloten codes |
  |---|---|
  | Prefix (`LIKE 'X%'`) | `PALET`, `PORT`, `CERT`, `TRANSPORT`, `VERPAKKING`, `ZAAGSNEDE`, `INVOERRECHT`, `VRACHT`, `EMBAL`, `ATINOX FRAIS`, `ADMIN`, `COMMISSION`, `ESSAI`, `REMISE`, `COMMERCIELE` |
  | Exact (`IN (…)`) | `RLA`, `DIV` |

  Filter in SQL:
  ```
  WHERE H.CANCELED = 'N' AND L.ItemCode IS NOT NULL AND NOT (L.ItemCode LIKE 'PALET%' OR L.ItemCode LIKE 'PORT%' OR L.ItemCode LIKE 'CERT%' OR L.ItemCode LIKE 'TRANSPORT%' OR L.ItemCode LIKE 'VERPAKKING%' OR L.ItemCode LIKE 'ZAAGSNEDE%' OR L.ItemCode LIKE 'INVOERRECHT%' OR L.ItemCode LIKE 'VRACHT%' OR L.ItemCode LIKE 'EMBAL%' OR L.ItemCode LIKE 'ATINOX FRAIS%' OR L.ItemCode LIKE 'ADMIN%' OR L.ItemCode LIKE 'COMMISSION%' OR L.ItemCode LIKE 'ESSAI%' OR L.ItemCode LIKE 'REMISE%' OR L.ItemCode LIKE 'COMMERCIELE%' OR L.ItemCode IN ('RLA','DIV')) AND H.DocDate >= '{Start}'
  ```

### Winstmarge Atinox: 23 % vs 16 % (opgelost 24-09-2026)

Het dashboard toonde voor Atinox **23,3 %** winst, de SAP-verkoopanalyse ± **16,5 %**. Oorzaak:
het artikel **`COMMISSION`** ("COMMISSION SUR LES ACHATS SADEL"), gefactureerd aan **SADEL NV**:
€ 1.176.576 omzet aan **100 % marge** (geen kostprijs). Samen met `RLA` (livraison directe, 100 %),
`DIV`, `ESSAI CRYO`, `REMISE COMMERCIALE` en `COMMERCIELE GESTE` trok dat de marge omhoog.

| Atinox 2026 | Omzet | Winst | Marge |
|---|---|---|---|
| Oude uitsluitingen | 13.979.716 | 3.255.766 | 23,3 % |
| Nieuwe uitsluitingen | 12.786.012 | 2.062.707 | **16,1 %** |

Let op: dit weerlegt "Sadel staat nooit als klant in Atinox" — de commissiefacturen staan op naam
van SADEL NV (intercompany).

---

## 4. Power BI-model

Power BI Desktop, modus **Import**, native SQL per tabel (enkel de nodige kolommen). Elke
feitentabel haalt Sadel en Atinox op met dezelfde query (placeholder `{B}` = bedrijf) en voegt ze
samen met `Table.Combine({Sadel, Atinox})`.

### Parameters (`expressions.tmdl`)

`SAP_Server` = `192.168.1.13`, `DB_Sadel` = `SBO_SADEL_PROD`, `DB_Atinox` = `SBO_ATINOX_FR`,
`StartDatum` = `2025-01-01`.

### Tabellen

| Tabel | Bron | Inhoud |
|---|---|---|
| `Verkoopregels` | `OINV`/`INV1` (+), `ORIN`/`RIN1` (−), `NNM1` | Kolommen: Bedrijf, DocSoort, OrderKey, DocNr, Datum, KlantKey, Reeks, Regel, Artikelnummer, Omschrijving, Magazijn, Hoeveelheid, OmzetRegel, WinstRuw, WinstRegel, Kostprijsfout, IsPlaat, OrderBevatPlaat |
| `Offerteregels` | `OQUT`/`QUT1` | BedragRegel, WinstRegel, RegelOmgezet, OfferteBevatPlaat, OfferteOmgezet |
| `Klant` | `OCRD` | `KlantKey` = `Bedrijf|CardCode` |
| `Artikel` | `OITM` + `OITB` | Artikelgroep, ArtikelType Plaat/Overig |
| `Datum` | gegenereerd | Jaar, Kwartaal, Maand, JaarMaand |
| `Vergelijking` | in het script | Segmenten: Platen / Bijverkoop in plaatorders / Orders zonder platen / Totaal alle orders |

Sleutels: `OrderKey` = `Bedrijf|F|DocEntry` (factuur) of `Bedrijf|C|DocEntry` (creditnota).
Relaties: feitentabellen → `Klant`, `Artikel`, `Datum` (veel-op-één).

Belangrijkste metingen: `Omzet = SUM(Verkoopregels[OmzetRegel])`,
`Winst = SUM(Verkoopregels[WinstRegel])`, `Winst % alle orders` (enkel `DocSoort = "Factuur"`),
segmentmetingen via `SWITCH` op `Vergelijking[Segment]`.

### Rapport (PBIP/PBIR)

Project: `C:\Users\xander\Documents\Visuals\Platenanalyse-DashboardV1.pbip`
(kopie van `Z:\Xander\Visuals`; de Z:-schijf kan niet rechtstreeks aan Claude gekoppeld worden).

- Pagina's: **Managementoverzicht**, **Plaatklanten**, **Bijverkoop & offertes** (+ testpagina).
- Rapportfilters: `KlantKey` niet `Sadel|KA00262`; Jaar = 2026.
- **Bedrijf-slicer** op elke pagina, gesynchroniseerd (syncGroup "Bedrijf"): Sadel / Atinox / beide.
- Kleuren (thema `Sadel_thema.json`): platen `#B4690E`, bijverkoop `#0D5CAB`, grijs `#6B7686`,
  navy `#0E3A63`; navy titelband, gekleurde kaarten, tabelkoppen met databalken.
- Het tekstvak "Kernboodschap" is statische tekst en moet manueel bijgewerkt worden.
- PBIR-versies in deze installatie: visualContainer 2.12.0, page 2.1.0, report 3.3.0.

### Kerncijfers Sadel jan–sep 2026 (excl. Atinox-intercompany, V8-data)

| Segment | Omzet | Winst % |
|---|---|---|
| Platen | € 3,18 mln (17,8 %) | 9,6 % |
| Bijverkoop in plaatorders | € 1,45 mln (8,1 %) | 18,5 % |
| Orders zonder platen | € 13,21 mln (74,1 %) | 17,8 % |
| Totaal | € 17,84 mln | 16,4 % |

540 plaatklanten (van 1.458) = 61 % van de omzet. Plaatorder gemiddeld € 3.175 vs € 1.403.
Offerteconversie 44,2 % (alle), 41,7 % (met plaat).

### Bestanden

| Bestand | Doel |
|---|---|
| `Platenanalyse-DashboardV1.pbip` | Power BI-project (model + 3 dashboardpagina's) |
| `Platenanalyse_model.tmdl` / `_Sadel.tmdl` | Volledig model als TMDL-script |
| `Stap1_tabellen_Sadel.tmdl`, `Stap3_vergelijking.tmdl` | Stapsgewijze fallback |
| `Sadel_thema.json` | Power BI-thema |
| `Atinox_verkooplijst_query.sql` | Volledige verkooplijst Atinox met marges, om zelf na te rekenen |
| `Platenanalyse per Klant Sadel Atinox V8.xlsx` | Vorige Excel-versie (vervangen door Power BI) |

---

## 5. Valkuilen

- **Power BI overschrijft de TMDL-bestanden bij opslaan.** Na een aanpassing door Claude: Power BI
  sluiten **zonder opslaan**, opnieuw openen, **Vernieuwen**, daarna pas opslaan. (De
  `ADMIN%`-uitsluiting ging zo tweemaal verloren.)
- Geen ruwe SAP-tabellen volledig laden (`OPOR`, `INV1`, …) → "not enough memory".
- "Cannot find name" in de TMDL-weergave zijn enkel waarschuwingen: **Toepassen**, dan **Vernieuwen**.
- Meting en kolom mogen niet dezelfde naam hebben (daarom `OmzetRegel`/`WinstRegel`/`BedragRegel`).
- Native query's vragen eenmalig "Uitvoeren" (uit te zetten via *Opties → Beveiliging*).
- SAP Query Manager: geen `--`-commentaar en geen tabs/verborgen tekens plakken; query als één regel.
- Standaardverslagen (Omzetanalyseverslag) bevatten geen ordernummer en tellen niet-artikelregels
  zoals `COMMISSION` anders — altijd dezelfde uitsluitingen toepassen bij vergelijken.
- **Fout in V8 (Excel):** tabblad *Overzicht* verwijst voor Sadel naar rij 545 i.p.v. totaalrij 546.
