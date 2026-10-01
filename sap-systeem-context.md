# SAP-systeem — context Sadel NV

Referentiedocument om in een ander project/chat te plakken, zodat direct duidelijk is met welk
ERP-systeem en welke datastructuur wordt gewerkt bij Sadel NV (roestvast staal import/export).

## Systeem

- **SAP Business One** (SAP B1), met rechtstreekse SQL-toegang op de onderliggende tabellen.
- Gebruikt voor: goederenontvangst/-registratie, inkoopbestellingen, facturatie, en als bron voor
  inkoopanalyses en prijszetting.
- Sinds 24-09-2026 ook rechtstreeks gekoppeld aan **Power BI** (zie onderaan).

## Kerntabellen (inkoopzijde)

| Tabel | Inhoud |
|---|---|
| `OPOR` | Header van inkoopbestellingen (purchase order) |
| `POR1` | Regels van inkoopbestellingen (artikel, hoeveelheid, prijs, per bestelling) |
| `OPCH` | Header van inkoopfacturen **zonder onderliggende bestelling** |
| `PCH1` | Regels van diezelfde facturen |

Een volledige inkoophistoriek combineert dus `OPOR/POR1` (bestelde en ontvangen goederen) met
`OPCH/PCH1` (rechtstreeks gefactureerde goederen die geen apart bestelnummer hadden).

## Kerntabellen (verkoopzijde)

| Tabel | Inhoud |
|---|---|
| `OINV` / `INV1` | Verkoopfacturen (header / regels) — winst: `OINV.GrosProfit`, `INV1.GrssProfit` |
| `ORIN` / `RIN1` | Creditnota's (header / regels) |
| `OQUT` / `QUT1` | Offertes (header / regels); `QUT1.TrgetEntry` ≠ 0 = omgezet in een order |
| `OCRD` | Klanten en leveranciers |
| `OITM` / `OITB` | Artikelen / artikelgroepen |
| `OITW` | Stock per magazijn |
| `ITM1` / `OPLN` | Prijzen per prijslijst / prijslijstnamen |
| `NNM1` | Reeksnamen (`SeriesName`) per `Series` |

## Documentnummer — coderingslogica

Documentnummers coderen **jaar + reeks**, in een vast patroon:

- **Eerste 2 cijfers** = jaar (bv. `25` = 2025, `26` = 2026)
- **Volgende 2 cijfers** = nummerreeks/serie, die het type document/bestelling aanduidt

### Gekende series

| Reeks | Betekenis | Kenmerk |
|---|---|---|
| **61** (bv. 2561, 2661) | Stockorder | Geplande, reguliere bestelling — **geen TR** |
| **67** | 2e OPOR-reeks | Duidelijk spoedorder-patroon (TR) — mediaan gelijk aan reeks 61, maar p90 +37%, p95 +55%, 20% van de regels >20% duurder dan artikel-mediaan |
| **64** | Factuur zonder bestelling (OPCH/PCH1) | Premiepatroon tussen 61 en 67 in |
| **65** | Factuur zonder bestelling (OPCH/PCH1) | Premiepatroon tussen 61 en 67 in, lichter dan 64 |
| **70** (2670) | Offerte Sadel (OQUT) | Uit de offertebestanden |
| **71** (2671) | Verkooporder Sadel | Doeldocument van omgezette offertes |
| **74** (2674) | Verkoopfactuur Sadel (OINV) | "Ordernummer" in de platenanalyse |

**Belangrijk:** enkel reeks 61 is bevestigd als stockorder/niet-TR. De precieze betekenis van 64,
65 en 67 is een werkhypothese afgeleid uit prijspatronen (niet uit een officiële SAP/IT-legende) —
nog te bevestigen met SAP/IT (via `NNM1.SeriesName`). Alle reeksen behalve 61 worden in de huidige
inkoopanalyse behandeld als **TR (spoedbestelling)**. De verkoopreeksen 70/71/74 zijn afgeleid uit
de nummers in de bestanden.

## Niet-materiaalregels

`DocType = 'I'` filtert regels die geen fysiek materiaal zijn, o.a.:
- `TRANSPORT`
- `VERPAKKING`
- `CERTFLEX`
- `ZAAGSNEDE`
- `INVOERRECHTEN`

## Overige velden/eigenaardigheden om rekening mee te houden

- Hoeveelheden en bedragen komen soms in tekstformaat binnen (bv. `"EUR 1.428,00"`); bij parsing
  van gebroken hoeveelheden (bv. `223,7`) mag je **niet** dezelfde bedrag-parsingfunctie
  hergebruiken — die verwijdert punten/komma's op een manier die bedoeld is voor bedragen, en
  vermenigvuldigt kleine hoeveelheden per ongeluk met 10. Correcte aanpak: `str(x).replace(',', '.')`.
- `Actuele_Prijs` (in de eigen Excel-tools) is een **afgeleide** van de inkoopprijs (mediaan ≈1,03x
  gemiddelde kostprijs) — niet gebruiken als onafhankelijke marktreferentie in modellen die de
  inkoopprijs zelf proberen te verklaren (circulariteit).
- Origine/land van herkomst ontbreekt vaak op artikelniveau; wordt afgeleid uit de leveranciersnaam
  in de aankoophistoriek (relevant voor CBAM-tarief: 8% of 12%).
- Eigen velden (`U_…`) en tabellen (`@…`) van de SAP-partner: omschrijving via
  `SELECT TableID, AliasID, Descr FROM CUFD` en `SELECT TableName, Descr FROM OUTB`.
  In de SAP-client toont *Beeld → Systeeminformatie* de tabel- en veldnaam onder de muis.

## Kolomhoofdingen van de werkbestanden

### SAP-export op orderniveau — `Bestelling op artikelniveau` / `... inclusief Documentnummer`
(dit zijn de bruikbare regels uit `OPOR/POR1` + `OPCH/PCH1` samen)

`Ordernummer` (of `Documentnummer` in de versie met documentserie), `Datum`, `Leverancierscode`,
`Leverancier`, `Artikelnummer`, `Omschrijving`, `Hoeveelheid`, `Eenheid`, `PrijsPerEenheid`,
`RegeltotaalLokaal`, `RegeltotaalVreemd`, `Munt`, `Koers`

### `Sym_Stockinfolijst` (stocklijst)

`Artikelnummer`, `Artikelomschrijving`, `In magazijn`, `Magazijncode`, `Eenheid`,
`BeschikbaarSAP`, `Besteld`, `Gereserveerd`, `GewichtSAP`, `Gemiddelde_kostprijs`,
`Actuele_Prijs`, `VervangPrijs`, `Voorlooptijd`, `Minimumorderhoeveelheid`,
`Minimale voorraad`, `StockARtikel`, `DefaultLeverancier`, `Gew_per_Eenheid`, `Theoretisch`,
`Groepsnaam`, `ZoekCOde`, `Origine`, `DatumAP`, `LastOrderDatePurchase`, `LastOrderDateSales`,
`LastDeliveryDate`, `LastReceivedDate`, `StockY-1`, `AantalVerkochtLY`,
`GemiddeldeVoorraadwaarde`, `KostprijsOmzet`, `VerkochtMaand0` t/m `VerkochtMaand-6`,
`Omloopsnelheid`

Vermoedelijke bron in SAP: de partnertabellen `@SYM_STATISTIEK_STOCK_01` / `_02` (nog te bevestigen).

### `Inkoopanalyseverslag op artikelen Sinds 2025` (vervangen door de orderdata hierboven)

`Artikelnummer`, `Artikelomschrijving`, `Leveranciernaam`, `Jaartotaal`, en per maand
(jan 2025 t/m sep 2026) telkens een kolompaar `<Maand> (<jaar>) - Hoeveelheid` /
`<Maand> (<jaar>) - Verworven bedrag`

### `Pricing overzicht - alle productgroepen`

Eén tabblad per productgroep (bv. `Plat 304 kwaliteit 1`, `gelaste buizen iso e~ule 316L 1`,
`hoek 304L 1`, `vol rond 304L 1`, …). Geen vaste kolomstructuur over de tabbladen heen — het is
een verzameling losstaande rekenbladen, telkens met een combinatie van: artikelnummer/omschrijving,
`toeslag/kg` (of `toeslag Europa`/`toeslag Azië`), `basisprijs`/`basisprijs kg` (soms apart voor
Europa en Azië), `Cebam`/`Cbam` (8% of 12%), en op sommige tabbladen `Simulatie dagprijs` of
`offerte`. Enkel het tabblad `vol rond 304L 1` heeft nette kolomkoppen: `Artikelnummer`,
`Artikelomschrijving`, `Eh`, `Conversie`, `Toeslag`, `basis kg/prijs`, `Cebam`.

### `Namen van alle querys` — ruwe SAP-kolommetadata

Volledige kolomlijst (naam + SQL-datatype) rechtstreeks uit SAP Business One, 510 kolommen,
formaat `# | COLUMN_NAME | DATA_TYPE` (bv. `DocEntry int`, `DocNum int`, `DocType char`,
`CardCode nvarchar`, `DocTotal numeric`, …). Dit is het volledige veldenoverzicht van de
onderliggende SAP-tabellen (o.a. `OPOR`/header-velden); gebruik dit bestand als je een veld nodig
hebt dat niet in de bewerkte exports hierboven zit.

## Automatiseringstool (context)

- Automatische Excel-prijstool voor platte RVS-producten, kruist SAP-orderdata tegen historische
  goederenlijsten (304/304L en 316/316L kwaliteiten).
- Features: automatische categoriegebaseerde prijsupdate, kleurcodering, en een historieklog.
- SAP Business One SQL-queries voor het koppelen van artikelnummers aan factuurnummers.

## Gerelateerde Excel-bestanden (huidige analysecontext)

- `Sym_Stockinfolijst` — stocklijst
- `Pricing overzicht - alle productgroepen` — huidige prijszetting per productgroep
- SAP-export op orderniveau (`OPOR/POR1` + `OPCH/PCH1`, sinds 01-01-2025)
- Ouder, vervangen bestand: `Inkoopanalyseverslag op artikelen Sinds 2025` (maandelijks
  samengevat — nu vervangen door de orderdata rechtstreeks uit SAP)
- `Platenanalyse per Klant Sadel Atinox V8.xlsx` — laatste Excel-versie van de platenanalyse,
  nu vervangen door het Power BI-model hieronder

## Power BI — live verbinding met SAP (platenanalyse, sinds 24-09-2026)

De platenanalyse (V8 in Excel) is overgezet naar **Power BI Desktop**, met een rechtstreekse
verbinding op de SAP-database. Er zijn geen manuele exports uit Query Manager meer nodig.

### Verbinding

| Parameter | Waarde |
|---|---|
| Server | `192.168.1.13` |
| Database Sadel | `SBO_SADEL_PROD` (naam zichtbaar op het SAP-inlogscherm) |
| Database Atinox | **geen toegang** — enkel de Sadel-database is beschikbaar |
| Startdatum data | parameter `StartDatum` = `2025-01-01` |

Modus: **Import** (niet DirectQuery). Native SQL-query's per tabel, enkel de nodige kolommen.

### Datamodel (sterschema, TMDL-script)

| Tabel | Bron in SAP | Inhoud |
|---|---|---|
| `Verkoopregels` | `OINV`/`INV1` (+) en `ORIN`/`RIN1` (−), `NNM1` | Factuur- en creditnotaregels; `IsPlaat` (ItemCode begint met `5`), `OrderBevatPlaat` (window-functie per DocEntry), `Kostprijsfout` |
| `Offerteregels` | `OQUT`/`QUT1` | Offerteregels; `RegelOmgezet` (`TrgetEntry` ≠ 0), `OfferteBevatPlaat`, `OfferteOmgezet` |
| `Klant` | `OCRD` (CardType C en L) | Sleutel `KlantKey` = `Bedrijf|CardCode` |
| `Artikel` | `OITM` + `OITB` | Artikelgroep uit SAP, `ArtikelType` Plaat/Overig |
| `Datum` | door Power BI gegenereerd | Jaar, Kwartaal, Maand, JaarMaand |
| `Vergelijking` | tabel in het script | Segmenten: Platen / Bijverkoop in plaatorders / Orders zonder platen / Totaal alle orders |

Relaties: beide feitentabellen → `Klant`, `Artikel`, `Datum` (veel-op-één).

Metingen (± 58) in mappen: orders met platen, heel assortiment, datakwaliteit, offertes, offertes
per artikel, algemeen, orders zonder platen, aandelen, plaatklanten en segmentvergelijking
(`Segment omzet/winst/winst %` via `SWITCH` op `Vergelijking[Segment]`).

### Bedrijfsregels in het model

- **Orders = verkoopfacturen** (`OINV`). Nummers Sadel beginnen met `2674`; offertes `2670`,
  verkooporders (doel van offertes) `2671`. Atinox: facturen `2666`, offertes `2661`, orders `2662`.
- **Omzet** = `INV1.LineTotal`, **winst** = `INV1.GrssProfit` (spelling!). Creditnota's tellen
  negatief mee in "heel assortiment", niet in de analyses op orderniveau.
- **Kostprijsfout-regel:** winst op een regel wordt 0 gezet als `LineTotal > 0` en
  `GrssProfit < −2 × LineTotal`. Vangt o.a. Atinox-order 266603936 en 266602775 (klant KL00036).
- **Intercompany Atinox uitsluiten:** Sadel verkoopt aan Atinox als klant `KA00262`
  (± € 13,3 mln = 43 % van de Sadel-omzet jan–sep 2026). Rapportfilter: `Klant[Klantcode]` is niet
  `KA00262`. Zonder die filter is het aandeel platen zwaar onderschat.

### Kerncijfers Sadel jan–sep 2026 (excl. Atinox, uit V8-data)

| Segment | Omzet | Winst % |
|---|---|---|
| Platen | € 3,18 mln (17,8 %) | 9,6 % |
| Bijverkoop in plaatorders | € 1,45 mln (8,1 %) | 18,5 % |
| Orders zonder platen | € 13,21 mln (74,1 %) | 17,8 % |
| Totaal | € 17,84 mln | 16,4 % |

540 plaatklanten (van 1.458) = 61 % van de omzet. Plaatorder gemiddeld € 3.175 tegenover € 1.403
voor andere orders van dezelfde klanten. Offerteconversie: 44,2 % alle offertes, 41,7 % met plaat.

Controlecijfers orders met platen (Sadel 2026): 1.457 orders, 540 klanten, omzet € 4.625.377,
platen € 3.178.150, winst platen € 305.506.

### Bestanden

| Bestand | Doel |
|---|---|
| `Platenanalyse_model_Sadel.tmdl` | Volledig model (enkel Sadel) — plakken in TMDL-weergave |
| `Platenanalyse_model.tmdl` | Zelfde model voor Sadel + Atinox (parameter `DB_Atinox`, nog niet bruikbaar) |
| `Stap1_tabellen_Sadel.tmdl` | Enkel tabellen + relaties (fallback) |
| `Stap3_vergelijking.tmdl` | Tabel `Vergelijking` + vergelijkingsmetingen |
| `Sadel_thema.json` | Power BI-thema in Sadel-huisstijl (platen `#B4690E`, bijverkoop `#0D5CAB`, grijs `#6B7686`, navy `#0E3A63`) |
| `C:\Users\xander\Documents\Visuals\Platenanalyse-DashboardV1.pbip` | Power BI-project (PBIR) met 3 dashboardpagina's: Managementoverzicht, Plaatklanten, Bijverkoop & offertes (origineel op `Z:\Xander\Visuals`) |

Voorbeeldontwerp van het managementdashboard: Claude-artifact "Platenanalyse Sadel –
managementdashboard" (3 pagina's van 1280 × 720).

### Valkuilen (ervaring uit deze sessie)

- **Geen ruwe SAP-tabellen volledig laden** (`OPOR`, `INV1`, `ITM1`, …): honderden kolommen en
  alle historiek → "not enough memory". Enkel via query's met geselecteerde kolommen.
- In de TMDL-weergave geven metingen "Cannot find name" zolang de tabellen nog niet bestaan; dat
  zijn enkel waarschuwingen. **Toepassen** en daarna **Vernieuwen**.
- Een meting mag niet dezelfde naam hebben als een kolom in dezelfde tabel (daarom
  `OmzetRegel`/`WinstRegel`/`BedragRegel` als kolomnamen).
- Native query's vragen eenmalig goedkeuring ("Uitvoeren"); uit te zetten in
  *Opties → Beveiliging*.
- Visuals kunnen enkel via code via een **.pbip-project met PBIR** (preview-functies aanzetten);
  rapportversie in deze installatie: visualContainer 2.12.0, page 2.1.0, report 3.3.0.
- Netwerkschijf `Z:` kan niet rechtstreeks gekoppeld worden aan Claude; project eerst naar
  `Documenten` kopiëren.
- **Fout in V8 (Excel):** tabblad *Overzicht* verwijst voor Sadel naar rij 545 (klant ABUTRIEK)
  in plaats van de totaalrij 546 — de Sadel-cijfers in dat overzicht zijn fout.
