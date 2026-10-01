
---

## 6. Managementdashboard Satinox (nieuw model, 24-09-2026)

Apart Power BI-project naast de platenanalyse: `C:\Users\xander\Documents\Visuals\Managementdashboard-Satinox.pbip`
(eigen `.SemanticModel` en `.Report`; raakt de platenanalyse-projecten niet).

**Doel:** management ziet snel omzet, winst en winst % per productgroep (platen, staf, buizen, fittingen), per verkoper en
wat er vandaag verkocht is — Sadel, Atinox of beide.

| Tabel | Bron | Inhoud |
|---|---|---|
| `Verkoopregels` | `OINV`/`INV1` (+), `ORIN`/`RIN1` (−) | Omzet/winst; zelfde kostenuitsluitingen en kostprijsfout-regel als hierboven |
| `Orderregels` | `ORDR`/`RDR1` | Verkooporders; kolom `IsVandaag` (datum = vernieuwingsdatum) |
| `Artikel` | `OITM` + `OITB`, Sadel eerst, Atinox-only aangevuld | `Hoofdgroep` = Platen / Staf / Buizen / Fittingen (incl. flenzen, zuivel) / Overig |
| `Verkoper` | `OSLP` via `H.SlpCode` | `VerkoperKey` = `Bedrijf|SlpCode` |
| `Klant` | `OCRD` | `Intercompany` = Ja voor Sadel `KA00262` en Atinox-klanten `SADEL%` → rapportfilter Nee |
| `Datum`, `Vernieuwing` | Power Query | `Vernieuwing` = tijdstip laatste refresh in Belgische tijd; "vandaag" = die datum |

Hoofdgroep-logica: op SAP-groepsnaam (plaat → Platen, staf → Staf, buis/buizen/pijp → Buizen, fit/zuivel/flens → Fittingen);
groep "Niet bepaald" of leeg → op eerste cijfer artikelnummer (5 platen, 6 staf, 1 buizen, 2/3/4/7/8 fittingen).
Let op: platen met een P-artikelcode tellen hier mee als Platen, in de platenanalyse niet (daar enkel `5%`).

Vorig jaar wordt vergeleken over dezelfde periode (t/m vernieuwingsdatum − 12 maanden).

Rapport: pagina **Managementoverzicht** (KPI's, vandaag-blok, per productgroep, per maand, per verkoper) en **Vandaag verkocht**
(paginafilter `IsVandaag = Ja`). Slicers Bedrijf en Verkoper gesynchroniseerd.

Voor delen met management: Power BI Pro per gebruiker (of capaciteit), werkruimte in Power BI Service, on-premises data gateway
op een pc/server die altijd aanstaat, SQL-login met leesrechten (fout 18456 eerst oplossen), refresh max. 8×/dag met Pro.

### Update 25-09-2026

- Nieuwe tabel `Offerteregels` (`OQUT`/`QUT1`, Sadel + Atinox, zelfde kostenuitsluitingen) met relaties naar Datum, Klant, Artikel.
  Omzetting: regel omgezet als `QUT1.TrgetEntry` ≠ 0; offerte omgezet als minstens één regel omgezet is.
  Metingen: Aantal offertes, Offertebedrag, Omgezette offertes, Offerte naar order %, Omgezet offertebedrag, Omgezet bedrag %, Omgezet (Ja/Nee).
- Nieuwe pagina **Offertes** (kaarten, per productgroep, offertes per klant); op Managementoverzicht tabel *Offertes per productgroep*.
- Verkoper verwijderd uit het rapport (tabellen en slicers verborgen, kolom uit orderregels); verkopertabel blijft in het model.
- Kaart "Titel vandaag" verborgen; slicers Bedrijf/Jaar/Maand gesynchroniseerd over de pagina's.
- Mobiele indeling (324 px breed) op alle pagina's: kaarten 16 pt, tabellen 8 pt, Bedrijf-filter volle breedte, balken per productgroep lichtblauw `#9FC5E8`.
- Het rapport is vooral bedoeld voor gebruik op de gsm (Power BI-app).
- Testklant Sadel **K001 "SADEL TEST KLANT"** uitgesloten in de offertequery (`NOT ('{B}' = 'Sadel' AND H.CardCode = 'K001')`); 2026: 76 offertes, € 7,0 mln.
  Andere testachtige klant in Sadel-offertes: PP12323 "PASCAL/TEST" (4 offertes, € 0) — niet uitgesloten.
- Offertes: lijngrafiek offertebedrag per maand 2025 vs 2026 (metingen `Offertebedrag 2025/2026`); Managementoverzicht: lijngrafiek omzet per productgroep per maand, excl. Overig.
- Vorig jaar: `Omzet/Winst vorig jaar` via TREATAS op dezelfde datums −12 maanden (t/m vernieuwingsdatum); kaarten "Omzet t.o.v. VJ" en "Winst t.o.v. VJ".
