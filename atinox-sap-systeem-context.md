# SAP-systeem — context Atinox (verkoopzijde)

Referentiedocument, analoog aan `sap-systeem-context.md` (Sadel/inkoopzijde), maar dan voor de
verkoop/omzetanalyse bij Atinox (dochterbedrijf van Sadel, actief in Frankrijk).

## Systeem

- **SAP Business One** (SAP B1), zelfde platform als Sadel maar **eigen company database** voor
  Atinox — tabelschema is identiek qua opbouw, maar moet apart bevraagd worden (andere database/
  connectie dan de Sadel-DB uit `sap-systeem-context.md`).
- Toegang via SQL Server ODBC, rechtstreeks op de onderliggende tabellen (Query Manager in SAP B1
  Client, of rechtstreekse SQL-toegang).

## Kerntabellen (verkoopzijde — omzetanalyse)

| Tabel | Inhoud |
|---|---|
| `OINV` | Header van verkoopfacturen (A/R Invoice) |
| `INV1` | Regels van diezelfde facturen (artikel, hoeveelheid, prijs, winst — per factuur) |

Dit is het verkoop-equivalent van `OPOR/POR1` (inkoop) uit de Sadel-context. Het
"Omzetanalyseverslag per artikel per klant" komt uit deze twee tabellen, maar dat standaardverslag
bevat **geen ordernummer** — voor analyses die het ordernummer nodig hebben (bv. % platen per
order) moet rechtstreeks op `OINV`/`INV1` gequeried worden.

## Kolomnamen — belangrijke valkuil (inconsistente spelling winst)

SAP B1 gebruikt **niet dezelfde spelling** van "GrossProfit" op headerniveau en regelniveau:

| Tabel | Kolomnaam winst | Niveau |
|---|---|---|
| `OINV` | `GrosProfit` (één "s") | totaal van het hele order/factuur |
| `INV1` | `GrssProfit` (geen "o") | per regel/artikel |

Beide zijn typefouten in het SAP-schema zelf (niet in onze bestanden) — dit heeft al meermaals tot
"Invalid column name"-fouten geleid in de Query Manager. **Altijd verifiëren met**
`SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = '<tabel>'` bij twijfel,
in plaats van de Sadel-spelling te hergebruiken.

## Artikelnummer-conventie

- Artikelnummers die beginnen met **`5`** = platen (`ItemCode LIKE '5%'`).
- `ItemCode IS NOT NULL` filtert niet-materiaalregels (vracht, verpakking, e.d.) op `INV1`,
  analoog aan de `DocType = 'I'`-filter bij Sadel's inkoopregels.

## Kruisverwijzing Sadel ↔ Atinox

`OINV` heeft twee user-defined fields die orders tussen de twee bedrijven aan elkaar koppelen:
- `U_OrderAtinox`
- `U_OrderSadel`

Nuttig om een Atinox-factuur terug te linken naar het overeenkomstige Sadel-order (of omgekeerd).

## Belangrijkste velden voor de platen/order-analyse

**Header (`OINV`):** `DocEntry`, `DocNum` (Ordernummer), `DocDate`, `CardCode` (Klantcode),
`CardName` (Klantnaam), `GrosProfit`, `U_OrderAtinox`, `U_OrderSadel`

**Regels (`INV1`):** `DocEntry` (FK naar OINV), `LineNum`, `ItemCode` (Artikelnummer),
`Dscription` (Artikelomschrijving), `Quantity`, `LineTotal`, `GrssProfit`, `WhsCode`, `StockPrice`

## Query Manager — praktische valkuilen (ervaring uit deze sessie)

- **Geen `--`-commentaar gebruiken** in queries die in de SAP B1 Query Manager geplakt worden — de
  parser verwerkt inline comments soms niet correct, wat resulteert in "Incorrect syntax near ','".
- **Geen inspringing/multi-line met tabs** plakken vanuit een extern document — verborgen tekens
  (bv. non-breaking spaces) geven "Incorrect syntax near '<tabelnaam>'". Queries als één regel
  plakken, of eerst via Kladblok/Notepad plakken om opmaak te stripping.
- Bij "Invalid column name": eerst de kolomnaam verifiëren via `INFORMATION_SCHEMA.COLUMNS` in
  plaats van de spelling van de andere tabel/bedrijf te hergebruiken (zie spelling-valkuil hoger).

## Gerelateerde bestanden (huidige analysecontext)

- `ATINOX Omzetanalyseverslag per artikel per klant 2026.xlsx` — standaardverslag per klant/artikel/
  maand, zonder ordernummer (vervangen voor order-analyses door rechtstreekse `OINV`/`INV1`-query)
- `SADEL Hoeveelheid platen order_hoeveelheid winst per order_hoeveelheidwinst op de platen.xlsx` —
  analoge analyse voor Sadel, op klantniveau (het doelformat voor de Atinox-vergelijking)


## Volledige kolomlijsten

### `OINV` — headertabel (492 kolommen: naam + SQL-datatype)

```
DocEntry int
DocNum int
DocType char
CANCELED char
Handwrtten char
Printed char
DocStatus char
InvntSttus char
Transfered char
ObjType nvarchar
DocDate datetime
DocDueDate datetime
CardCode nvarchar
CardName nvarchar
Address nvarchar
NumAtCard nvarchar
VatPercent numeric
VatSum numeric
VatSumFC numeric
DiscPrcnt numeric
DiscSum numeric
DiscSumFC numeric
DocCur nvarchar
DocRate numeric
DocTotal numeric
DocTotalFC numeric
PaidToDate numeric
PaidFC numeric
GrosProfit numeric
GrosProfFC numeric
Ref1 nvarchar
Ref2 nvarchar
Comments nvarchar
JrnlMemo nvarchar
TransId int
ReceiptNum int
GroupNum smallint
DocTime smallint
SlpCode int
TrnspCode smallint
PartSupply char
Confirmed char
GrossBase smallint
ImportEnt int
CreateTran char
SummryType char
UpdInvnt char
UpdCardBal char
Instance smallint
Flags int
InvntDirec char
CntctCode int
ShowSCN char
FatherCard nvarchar
SysRate numeric
CurSource char
VatSumSy numeric
DiscSumSy numeric
DocTotalSy numeric
PaidSys numeric
FatherType char
GrosProfSy numeric
UpdateDate datetime
IsICT char
CreateDate datetime
Volume numeric
VolUnit smallint
Weight numeric
WeightUnit smallint
Series int
TaxDate datetime
Filler nvarchar
DataSource char
StampNum nvarchar
isCrin char
FinncPriod int
UserSign smallint
selfInv char
VatPaid numeric
VatPaidFC numeric
VatPaidSys numeric
UserSign2 smallint
WddStatus char
draftKey int
TotalExpns numeric
TotalExpFC numeric
TotalExpSC numeric
DunnLevel int
Address2 nvarchar
LogInstanc int
Exported char
StationID int
Indicator nvarchar
NetProc char
AqcsTax numeric
AqcsTaxFC numeric
AqcsTaxSC numeric
CashDiscPr numeric
CashDiscnt numeric
CashDiscFC numeric
CashDiscSC numeric
ShipToCode nvarchar
LicTradNum nvarchar
PaymentRef nvarchar
WTSum numeric
WTSumFC numeric
WTSumSC numeric
RoundDif numeric
RoundDifFC numeric
RoundDifSy numeric
CheckDigit char
Form1099 int
Box1099 nvarchar
submitted char
PoPrss char
Rounding char
RevisionPo char
Segment smallint
ReqDate datetime
CancelDate datetime
PickStatus char
Pick char
BlockDunn char
PeyMethod nvarchar
PayBlock char
PayBlckRef int
MaxDscn char
Reserve char
Max1099 numeric
CntrlBnk nvarchar
PickRmrk nvarchar
ISRCodLine nvarchar
ExpAppl numeric
ExpApplFC numeric
ExpApplSC numeric
Project nvarchar
DeferrTax char
LetterNum nvarchar
FromDate datetime
ToDate datetime
WTApplied numeric
WTAppliedF numeric
BoeReserev char
AgentCode nvarchar
WTAppliedS numeric
EquVatSum numeric
EquVatSumF numeric
EquVatSumS numeric
Installmnt smallint
VATFirst char
NnSbAmnt numeric
NnSbAmntSC numeric
NbSbAmntFC numeric
ExepAmnt numeric
ExepAmntSC numeric
ExepAmntFC numeric
VatDate datetime
CorrExt nvarchar
CorrInv int
NCorrInv int
CEECFlag char
BaseAmnt numeric
BaseAmntSC numeric
BaseAmntFC numeric
CtlAccount nvarchar
BPLId int
BPLName nvarchar
VATRegNum nvarchar
TxInvRptNo nvarchar
TxInvRptDt datetime
KVVATCode ntext
WTDetails nvarchar
SumAbsId int
SumRptDate datetime
PIndicator nvarchar
ManualNum nvarchar
UseShpdGd char
BaseVtAt numeric
BaseVtAtSC numeric
BaseVtAtFC numeric
NnSbVAt numeric
NnSbVAtSC numeric
NbSbVAtFC numeric
ExptVAt numeric
ExptVAtSC numeric
ExptVAtFC numeric
LYPmtAt numeric
LYPmtAtSC numeric
LYPmtAtFC numeric
ExpAnSum numeric
ExpAnSys numeric
ExpAnFrgn numeric
DocSubType nvarchar
DpmStatus char
DpmAmnt numeric
DpmAmntSC numeric
DpmAmntFC numeric
DpmDrawn char
DpmPrcnt numeric
PaidSum numeric
PaidSumFc numeric
PaidSumSc numeric
FolioPref nvarchar
FolioNum int
DpmAppl numeric
DpmApplFc numeric
DpmApplSc numeric
LPgFolioN int
Header ntext
Footer ntext
Posted char
OwnerCode int
BPChCode nvarchar
BPChCntc int
PayToCode nvarchar
IsPaytoBnk char
BnkCntry nvarchar
BankCode nvarchar
BnkAccount nvarchar
BnkBranch nvarchar
isIns char
TrackNo nvarchar
VersionNum nvarchar
LangCode int
BPNameOW char
BillToOW char
ShipToOW char
RetInvoice char
ClsDate datetime
MInvNum int
MInvDate datetime
SeqCode smallint
Serial int
SeriesStr nvarchar
SubStr nvarchar
Model nvarchar
TaxOnExp numeric
TaxOnExpFc numeric
TaxOnExpSc numeric
TaxOnExAp numeric
TaxOnExApF numeric
TaxOnExApS numeric
LastPmnTyp char
LndCstNum int
UseCorrVat char
BlkCredMmo char
OpenForLaC char
Excised char
ExcRefDate datetime
ExcRmvTime nvarchar
SrvGpPrcnt numeric
DepositNum int
CertNum nvarchar
DutyStatus char
AutoCrtFlw char
FlwRefDate datetime
FlwRefNum nvarchar
VatJENum int
DpmVat numeric
DpmVatFc numeric
DpmVatSc numeric
DpmAppVat numeric
DpmAppVatF numeric
DpmAppVatS numeric
InsurOp347 char
IgnRelDoc char
BuildDesc nvarchar
ResidenNum char
Checker int
Payee int
CopyNumber int
SSIExmpt char
PQTGrpSer int
PQTGrpNum int
PQTGrpHW char
ReopOriDoc char
ReopManCls char
DocManClsd char
ClosingOpt smallint
SpecDate datetime
Ordered char
NTSApprov char
NTSWebSite smallint
NTSeTaxNo nvarchar
NTSApprNo nvarchar
PayDuMonth char
ExtraMonth smallint
ExtraDays smallint
CdcOffset smallint
SignMsg ntext
SignDigest ntext
CertifNum nvarchar
KeyVersion int
EDocGenTyp char
ESeries smallint
EDocNum nvarchar
EDocExpFrm int
OnlineQuo char
POSEqNum nvarchar
POSManufSN nvarchar
POSCashN int
EDocStatus char
EDocCntnt ntext
EDocProces char
EDocErrCod nvarchar
EDocErrMsg ntext
EDocCancel char
EDocTest char
EDocPrefix nvarchar
CUP int
CIG int
DpmAsDscnt char
Attachment ntext
AtcEntry int
SupplCode nvarchar
GTSRlvnt char
BaseDisc numeric
BaseDiscSc numeric
BaseDiscFc numeric
BaseDiscPr numeric
CreateTS int
UpdateTS int
SrvTaxRule char
AnnInvDecR int
Supplier nvarchar
Releaser int
Receiver int
ToWhsCode nvarchar
AssetDate datetime
Requester nvarchar
ReqName nvarchar
Branch smallint
Department smallint
Email nvarchar
Notify char
ReqType int
OriginType char
IsReuseNum char
IsReuseNFN char
DocDlvry char
PaidDpm numeric
PaidDpmF numeric
PaidDpmS numeric
EnvTypeNFe int
AgrNo int
IsAlt char
AltBaseTyp int
AltBaseEnt int
AuthCode nvarchar
StDlvDate datetime
StDlvTime int
EndDlvDate datetime
EndDlvTime int
VclPlate nvarchar
ElCoStatus nvarchar
AtDocType nvarchar
ElCoMsg nvarchar
PrintSEPA char
FreeChrg numeric
FreeChrgFC numeric
FreeChrgSC numeric
NfeValue numeric
FiscDocNum nvarchar
RelatedTyp int
RelatedEnt int
CCDEntry int
NfePrntFo int
ZrdAbs int
POSRcptNo int
FoCTax numeric
FoCTaxFC numeric
FoCTaxSC numeric
TpCusPres int
ExcDocDate datetime
FoCFrght numeric
FoCFrghtFC numeric
FoCFrghtSC numeric
InterimTyp smallint
PTICode nvarchar
Letter char
FolNumFrom int
FolNumTo int
FolSeries int
SplitTax numeric
SplitTaxFC numeric
SplitTaxSC numeric
ToBinCode nvarchar
PriceMode char
PoDropPrss char
PermitNo nvarchar
MYFtype nvarchar
DocTaxID nvarchar
DateReport datetime
RepSection nvarchar
ExclTaxRep char
PosCashReg int
DmpTransID nvarchar
ECommerBP nvarchar
EComerGSTN nvarchar
Revision char
RevRefNo nvarchar
RevRefDate datetime
RevCreRefN nvarchar
RevCreRefD datetime
TaxInvNo nvarchar
FrmBpDate datetime
GSTTranTyp nvarchar
BaseType int
BaseEntry int
ComTrade char
UseBilAddr char
IssReason smallint
ComTradeRt char
SplitPmnt char
SOIWizId int
SelfPosted char
EnBnkAcct ntext
EncryptIV nvarchar
DPPStatus char
SAPPassprt ntext
EWBGenType char
CtActTax numeric
CtActTaxFC numeric
CtActTaxSC numeric
EDocType char
QRCodeSrc ntext
AggregDoc char
DataVers int
ShipState nvarchar
ShipPlace nvarchar
CustOffice nvarchar
FCI nvarchar
NnSbCuAmnt numeric
NnSbCuSC numeric
NnSbCuFC numeric
ExepCuAmnt numeric
ExepCuSC numeric
ExepCuFC numeric
AddLegIn nvarchar
LegTextF int
IndFinal char
DANFELgTxt ntext
PostPmntWT char
QRCodeSPGn ntext
FCEPmnMean char
ReqCode nvarchar
NotRel4MI char
Rel4PPTax char
ConfrmedBy int
ConfrmedOn datetime
ReqID int
NonDdAmt numeric
NonDdAmtSC numeric
NonDdAmtFC numeric
BookeTdsBP char
AllocNum nvarchar
DPayToAddr nvarchar
DigPayment char
OperProfit numeric
OperProfFC numeric
OperProfSy numeric
NetIncome numeric
NetIncomFC numeric
NetIncomSy numeric
PDueMonEnd char
RShipToCod nvarchar
Address3 nvarchar
RShipToOW char
EnLicTradN ntext
U_Bestelling nvarchar
U_LOCBE_NW ntext
U_LOCBE_IS_INTRREL char
U_LT char
U_Douane char
U_Certificaat char
U_Blokkeren char
U_Klantcode nvarchar
U_LeveringsVws nvarchar
U_LeveringsVwLev nvarchar
U_OrderAtinox nvarchar
U_OrderSadel nvarchar
U_OpmRn nvarchar
U_OpmTN nvarchar
U_NumAtCard nvarchar
U_Eigenaar nvarchar
U_VerzendWijze nvarchar
U_CertVerst char
U_Beslissingsdatum datetime
U_Reden nvarchar
U_SMTPStat ntext
U_COR_BW_FromDate datetime
U_COR_BW_ToDate datetime
```

### `INV1` — regeltabel (375 kolommen)

```
DocEntry
LineNum
TargetType
TrgetEntry
BaseRef
BaseType
BaseEntry
BaseLine
LineStatus
ItemCode
Dscription
Quantity
ShipDate
OpenQty
Price
Currency
Rate
DiscPrcnt
LineTotal
TotalFrgn
OpenSum
OpenSumFC
VendorNum
SerialNum
WhsCode
SlpCode
Commission
TreeType
AcctCode
TaxStatus
GrossBuyPr
PriceBefDi
DocDate
Flags
OpenCreQty
UseBaseUn
SubCatNum
BaseCard
TotalSumSy
OpenSumSys
InvntSttus
OcrCode
Project
CodeBars
VatPrcnt
VatGroup
PriceAfVAT
Height1
Hght1Unit
Height2
Hght2Unit
Width1
Wdth1Unit
Width2
Wdth2Unit
Length1
Len1Unit
length2
Len2Unit
Volume
VolUnit
Weight1
Wght1Unit
Weight2
Wght2Unit
Factor1
Factor2
Factor3
Factor4
PackQty
UpdInvntry
BaseDocNum
BaseAtCard
SWW
VatSum
VatSumFrgn
VatSumSy
FinncPriod
ObjType
LogInstanc
BlockNum
ImportLog
DedVatSum
DedVatSumF
DedVatSumS
IsAqcuistn
DistribSum
DstrbSumFC
DstrbSumSC
GrssProfit
GrssProfSC
GrssProfFC
VisOrder
INMPrice
PoTrgNum
PoTrgEntry
DropShip
PoLineNum
Address
TaxCode
TaxType
OrigItem
BackOrdr
FreeTxt
PickStatus
PickOty
PickIdNo
TrnsCode
VatAppld
VatAppldFC
VatAppldSC
BaseQty
BaseOpnQty
VatDscntPr
WtLiable
DeferrTax
EquVatPer
EquVatSum
EquVatSumF
EquVatSumS
LineVat
LineVatlF
LineVatS
unitMsr
NumPerMsr
CEECFlag
ToStock
ToDiff
ExciseAmt
TaxPerUnit
TotInclTax
CountryOrg
StckDstSum
ReleasQtty
LineType
TranType
Text
OwnerCode
StockPrice
ConsumeFCT
LstByDsSum
StckINMPr
LstBINMPr
StckDstFc
StckDstSc
LstByDsFc
LstByDsSc
StockSum
StockSumFc
StockSumSc
StckSumApp
StckAppFc
StckAppSc
ShipToCode
ShipToDesc
StckAppD
StckAppDFC
StckAppDSC
BasePrice
GTotal
GTotalFC
GTotalSC
DistribExp
DescOW
DetailsOW
GrossBase
VatWoDpm
VatWoDpmFc
VatWoDpmSc
CFOPCode
CSTCode
Usage
TaxOnly
WtCalced
QtyToShip
DelivrdQty
OrderedQty
CogsOcrCod
CiOppLineN
CogsAcct
ChgAsmBoMW
ActDelDate
OcrCode2
OcrCode3
OcrCode4
OcrCode5
TaxDistSum
TaxDistSFC
TaxDistSSC
PostTax
Excisable
AssblValue
RG23APart1
RG23APart2
RG23CPart1
RG23CPart2
CogsOcrCo2
CogsOcrCo3
CogsOcrCo4
CogsOcrCo5
LnExcised
LocCode
StockValue
GPTtlBasPr
unitMsr2
NumPerMsr2
SpecPrice
CSTfIPI
CSTfPIS
CSTfCOFINS
ExLineNo
isSrvCall
PQTReqQty
PQTReqDate
PcDocType
PcQuantity
LinManClsd
VatGrpSrc
NoInvtryMv
ActBaseEnt
ActBaseLn
ActBaseNum
OpenRtnQty
AgrNo
AgrLnNum
CredOrigin
Surpluses
DefBreak
Shortages
UomEntry
UomEntry2
UomCode
UomCode2
FromWhsCod
NeedQty
PartRetire
RetireQty
RetireAPC
RetirAPCFC
RetirAPCSC
InvQty
OpenInvQty
EnSetCost
RetCost
Incoterms
TransMod
LineVendor
DistribIS
ISDistrb
ISDistrbFC
ISDistrbSC
IsByPrdct
ItemType
PriceEdit
PrntLnNum
LinePoPrss
FreeChrgBP
TaxRelev
LegalText
ThirdParty
LicTradNum
InvQtyOnly
UnencReasn
ShipFromCo
ShipFromDe
FisrtBin
AllocBinC
ExpType
ExpUUID
ExpOpType
DIOTNat
MYFtype
GPBefDisc
ReturnRsn
ReturnAct
StgSeqNum
StgEntry
StgDesc
ItmTaxType
SacEntry
NCMCode
HsnEntry
OriBAbsEnt
OriBLinNum
OriBDocTyp
IsPrscGood
IsCstmAct
EncryptIV
ExtTaxRate
ExtTaxSum
TaxAmtSrc
ExtTaxSumF
ExtTaxSumS
StdItemId
CommClass
VatExEntry
VatExLN
NatOfTrans
ISDtCryImp
ISDtRgnImp
ISOrCryExp
ISOrRgnExp
NVECode
PoNum
PoItmNum
IndEscala
CESTCode
CtrSealQty
CNJPMan
UFFiscBene
CUSplit
LegalTIMD
LegalTTCA
LegalTW
LegalTCD
RevCharge
ListNum
RecogAmt
RecogAmtSC
RecogAmtFC
RecogVatGr
RecogVatPr
NonDdAmt
NonDdAmtSC
NonDdAmtFC
PPTaxExRe
PlPaWght
CUP
CIG
OperProfit
OperProfFC
OperProfSy
NetIncome
NetIncomFC
NetIncomSy
UoMNum
UoMDen
UoMNum2
UoMDen2
CSTfIBS
CSTfCBS
CSTfIS
U_Percentage
U_OnHand
U_OnOrder
U_EhPrijs
U_Leverancier
U_LeverancierR
U_Ruw
U_phcTRbtw
U_LOCBE_IS_ORIGCTR
U_LOCBE_IS_TRANSAC
U_LOCBE_IS_TRANSP
U_LOCBE_IS_TERMDEL
U_BECC_AcctName
U_ExPrice
U_ExCurrency
U_OpmAkp
U_DatumOA
U_DatumB2B
U_PrijsB2B
U_OpmPL
U_Conversie
U_EhPrijsAkp
U_Eh
U_OpmRN
U_Volledrager
U_Backorder
U_Doc_Sadel
U_DerectLev
U_Smeltnummers
U_Certificaat
U_Wmsid
U_Verzonden
U_RedenItem
```
