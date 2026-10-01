# Werkmap `Prijsupdate Pascal-methode.xlsx` (01-10-2026)

Geautomatiseerde versie van Pascals prijsupdate (zie `claude/prijsmotor-pascal.md` voor de methode).
Locatie: `Documents\Pascal\Prijsupdate Pascal-methode.xlsx`. Geen macro's.

## Keuzes Xander
1. Toeslagen centraal in de werkmap (tabblad Artikelen), niet meer in Pascals opbouwbestanden.
2. Werkwijze: Gegevens → Alles vernieuwen, daarna tabblad Upload kopiëren naar Blad1 van een importbestand.
3. Bij een nieuwe stockorder wordt de **hele basis** geüpload (ook als de basis gelijk blijft).
4. Basis: referentieartikel van Pascal op de order → anders ander artikel met toeslag 0 → anders mediaan €/kg (prijs/kg − toeslag) van de order.

## Opbouw
| Tabblad | Inhoud |
|---|---|
| Start | Stappen, status (aantal te uploaden, kopieerbereik, meldingen), instellingen (peildatum, venster 7 d, landingsdeler Azië 0,95, drempels 8 % / 3 %) |
| Controle | Per basis: basis stond, laatste stockorder, methode, gebruikt artikel, basis nieuw, status, nakijken, upload; nieuwe artikelen op stockorder; blok "Nieuwe regels voor Historiek" |
| Upload | Artikel + prijs, Blad1-formaat, geen kopregel |
| Artikelen | 1.183 artikelen: BasisID, eh, kg/eh, toeslag, CBAM-deler (uit Pascals laatste tabbladen); prijs stond/nieuw; toeslag-voorstel voor nieuwe artikelen |
| Basissen | 25 basissen: herkomst, referentieartikel Pascal, bronbestand |
| Leveranciers | Code → Europa/Azië; trefwoorden Azië als terugval |
| Historiek | Basis die in SAP staat; laatste rij per basis telt (gestart vanuit Pascals tabbladen) |
| Stocklijnen | Rekenblad (15.000 rijen) |
| Bestellingen | Power Query `Query2` (zelfde SQL als `SAP bestellingen.xlsx`: OPOR/POR1, 180 dagen) |

## Regels in de formules
- Enkel stockorders (cijfer 3–4 van het ordernummer = 61), van dezelfde herkomst als de basis, binnen het venster en ná de laatst verwerkte order in Historiek.
- Twee stockorders op dezelfde dag → die met het referentieartikel. Meerdere artikelen met toeslag 0 → zwaarste lijn (kg).
- kg/eh = Pascals waarde (niet de stocklijst).
- Na de upload: blok uit Controle als waarden in Historiek plakken; daarna tellen enkel orders vanaf de dag ná de gebruikte order.

## Test
Peildatum 29/09 + venster 28 d tegen de run van 30/09: zelfde orders en basissen, behalve:
- constructie 316L: 100x100x5 i.p.v. 80x80x5 → 4,3824 i.p.v. 4,4189;
- zuivel 304L en leiding 304L Europa: kleine verschillen door kg/eh van Pascal;
- plat 316L: nu mee in de upload (keuze 3).

Peildatum 01/10, venster 7: 8 basissen, 538 artikelen; nieuwe stockorder 266100402 (30/09) → platen 304L 2B 2,71.

## Open
- Is de upload van 30/09 al in SAP gezet? Zo ja: de 11 regels van die run in Historiek zetten.
- Keuze zwaarste lijn vs andere regel bij meerdere toeslag-0-artikelen bevestigen met Pascal.
- Werkmap één keer openen in Excel en vernieuwen (verbinding, eventueel "Uitvoeren" goedkeuren).
