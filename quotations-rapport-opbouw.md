# Quotations-rapport — lay-out en opbouw

**Rapport:** Quotations (Power BI Service)
**Pagina's:** 1 (dashboardpagina)
**Data bijgewerkt:** 29/09/2026
**Standaardweergave:** periode 1/01/2026 – 31/12/2026, alle verkopers, alle klanten

> Deze beschrijving is gemaakt op basis van de gepubliceerde weergave. Het onderliggende datamodel (tabellen, relaties, DAX-measures) was niet toegankelijk. Measures onder "Vermoedelijke logica" zijn afgeleid uit wat het rapport toont, niet uit de definities zelf.

---

## 1. Rasterindeling

De pagina is opgebouwd in twee horizontale banden met vier kolommen.

```
┌──────────────┬──────────────┬────────────────────────┬─────────────┬─────────────┐
│ Logo Sadel   │ Datumslicer  │                        │ KPI         │ KPI         │
│              │ (bereik)     │  Conversie per         │ Totaal      │ Offertes    │
├──────────────┼──────────────┤  verkoper              │ offertes    │ >30d        │
│ Slicer       │ Slicer       │  (tabel)               ├─────────────┼─────────────┤
│ Verkoper     │ Klant        │                        │ Open        │ Conversie   │
├──────────────┴──────────────┤                        │             │ aantal %    │
│ Funnel — Offerte naar order │                        ├─────────────┼─────────────┤
│ (trechter, 2 stappen)       │                        │ Omgezet     │ Conversie   │
│                             │                        │             │ waarde %    │
│                             │                        ├─────────────┼─────────────┤
│                             │                        │ Verloren    │ Gem. offerte│
│                             │                        │             │ waarde      │
├─────────────────────────────┴────────────────────────┼─────────────┴─────────────┤
│ Offerte details (matrix)                              │ Verloren offertes per     │
│                                                       │ reden (waarde)            │
│                                                       │ (liggend staafdiagram)    │
└───────────────────────────────────────────────────────┴───────────────────────────┘
```

| Zone | Positie | Inhoud |
|---|---|---|
| Filterzone | Linksboven | Logo, datumslicer, slicers Verkoper en Klant |
| Funnel | Links midden | Trechter offerte → order |
| Verkopersanalyse | Midden boven | Tabel conversie per verkoper |
| KPI-blok | Rechtsboven | 8 kaarten in een raster van 2 × 4 |
| Detail | Linksonder (breed) | Matrix met offertelijnen |
| Verliesanalyse | Rechtsonder | Staafdiagram verlorenwaarde per reden |

---

## 2. Visuals in detail

### 2.1 Logo
- **Type:** afbeelding
- **Inhoud:** logo Sadel Stainless Steel

### 2.2 Datumslicer
- **Type:** slicer, stijl *tussen* (begin- en einddatum met schuifbalk)
- **Veld:** `Date`
- **Bereik:** 1/01/2026 – 31/12/2026

### 2.3 Slicer Verkoper
- **Type:** slicer, stijl *vervolgkeuzelijst*
- **Veld:** Verkoper
- **Standaard:** Alle

### 2.4 Slicer Klant
- **Type:** slicer, stijl *vervolgkeuzelijst*
- **Veld:** Klant
- **Standaard:** Alle

### 2.5 Funnel — Offerte naar order
- **Type:** trechterdiagram, 2 stappen
- **Stap 1:** Aantal offertes (groen)
- **Stap 2:** Naar order, met aantal en percentage t.o.v. stap 1 (blauw)
- **Voorbeeldwaarden:** 17.133 → 7.567 (44,17 %)

### 2.6 Conversie per verkoper
- **Type:** tabel met rijselectie (werkt als kruisfilter)
- **Kolommen:**

| Kolom | Inhoud |
|---|---|
| Verkoper | Naam verkoper, incl. categorie "Ontbrekende verkoper" |
| Totaal | Aantal offertes |
| In order | Aantal offertes omgezet naar order |
| Conversie % | In order / Totaal |

- **Sortering:** aflopend op Conversie %
- **Opmaak:** afwisselende rijkleuren (zebra)

### 2.7 KPI-kaarten (8 stuks)

| Kaart | Voorbeeldwaarde | Betekenis |
|---|---|---|
| Totaal offertes | 17.133 | Aantal offertes in de selectie |
| Offertes >30d | 3.545 | Offertes ouder dan 30 dagen |
| Open | 4.647 | Offertes zonder eindstatus |
| Conversie aantal % | 44,17 | Omgezet / Totaal (aantal) |
| Omgezet | 7567 | Offertes omgezet naar order |
| Conversie waarde % | 19,74 | Omgezette waarde / totale offertewaarde |
| Verloren | 514 | Offertes met status verloren |
| Gem. offerte waarde | € 5.336 | Gemiddelde nettowaarde per offerte |

> Opmaakverschil: "Omgezet" toont `7567` zonder duizendtalscheiding, de andere kaarten wel (bv. `17.133`). De notatie van deze measure moet worden aangepast.

### 2.8 Offerte details
- **Type:** matrix met uitklapbare rijen (per offerte uit te klappen naar onderliggende lijnen)
- **Kolommen:**

| Kolom | Inhoud |
|---|---|
| Offerte | Offertenummer (bv. 267017193) |
| Datum | Offertedatum |
| Klant | Klantnaam |
| Verkoper | Naam verkoper |
| Netto | Nettobedrag (€) |
| Tonnage (KG) | Gewicht in kg |

- **Sortering:** op Datum
- **Voorwaardelijke opmaak:** lichtgroene rijachtergrond, vermoedelijk voor offertes die naar order zijn omgezet

### 2.9 Verloren offertes per reden (waarde)
- **Type:** liggend staafdiagram (gegroepeerde balk)
- **As:** Reden
- **Waarde:** som van de offertewaarde van verloren offertes (€, weergave in miljoen)
- **Sortering:** aflopend op waarde
- **Categorieën:** Budget aanvr..., Te Duur, Alternatief, Project uitges..., Project, Te Duur Platen, Onbekend, Stock, Levertermijn, Te Duur TN

---

## 3. Interactie

- Alle slicers (Datum, Verkoper, Klant) filteren de volledige pagina.
- Een rij selecteren in *Conversie per verkoper* filtert de overige visuals.
- In *Offerte details* kan elke offerte worden uitgeklapt.
- Het filtervenster staat rechts (standaard ingeklapt).
- Het rapport opent in de standaardweergave van de auteur. Er zijn geen persoonlijke bladwijzers actief.

---

## 4. Vermoedelijke logica van de measures

Afgeleid uit de weergegeven waarden. Te verifiëren in het model.

```text
Totaal offertes      = aantal unieke offertes
Omgezet              = aantal offertes met status "order"
Open                 = aantal offertes zonder eindstatus
Verloren             = aantal offertes met status "verloren"
Offertes >30d        = aantal offertes waarvan de offertedatum meer dan 30 dagen voor vandaag ligt (vermoedelijk enkel open offertes)
Conversie aantal %   = Omgezet / Totaal offertes
Conversie waarde %   = som Netto omgezet / som Netto alle offertes
Gem. offerte waarde  = som Netto / Totaal offertes
```

Controle: 7.567 / 17.133 = 44,17 %, dat klopt met de kaart.

---

## 5. Opmaak

| Element | Stijl |
|---|---|
| Achtergrond | Wit, visuals met lichtgrijze rand en afgeronde hoeken |
| Primaire kleur | Blauw (staafdiagram, funnel stap 2), in lijn met het Sadel-logo |
| Accentkleur | Groen (funnel stap 1, markering omgezette offertes) |
| KPI-kaarten | Groot getal bovenaan, label eronder, gecentreerd |
| Getalnotatie | Belgische notatie: punt als duizendtal, komma als decimaal |
