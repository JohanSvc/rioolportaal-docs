# Product Requirements Document — Rioolportaal

Aannemersportaal voor rioolaansluitingen en herstellingen — Water-Link

| | |
|---|---|
| **Versie** | 1.0 — vrijgegeven voor bespreking met de bouwpartij |
| **Datum** | 2026-09-14 |
| **Opdrachtgever** | Water-Link |
| **Doelpubliek** | Externe bouwpartij (technische realisatie) + de motor-eigenaar |
| **Status** | Vrijgegeven. Vijf open punten uit v0.9 zijn gesloten; wat nog open is, staat in §12 |
| **Bron van waarheid** | Dit document + het bijhorende API-contract en datamodel (zie §13) |

> **Leeswijzer voor de bouwpartij.** Dit document beschrijft *wat* gebouwd moet worden en *waarom*. Het schrijft geen presentatietechnologie voor het portaal voor — die keuze ligt bij de bouwpartij — met één harde uitzondering: het portaal wordt gebouwd **bovenop de bestaande SvcMgt-motor** van Water-Link (§5). Een werkende referentie-implementatie (mockup) bestaat en mag geïnspecteerd worden; ze is illustratief, geen specificatie. Waar dit document en de mockup verschillen, geldt dit document.
>
> **Over de cijfers en regels in v1.0.** De volumes, bedrijfsregels en prioriteiten in deze versie komen uit het antwoorddocument van Water-Link op de vragenlijst van de SAP-partner (7 september 2026, §13). Dat document is een **momentopname**: het geeft weer hoe het proces vandaag verloopt en hoe Water-Link het portaal voor zich ziet. Wordt in een voorstel of raming vertrokken van uitgangspunten die hier niet uitdrukkelijk bevestigd zijn, vermeld die dan expliciet, zodat ze samen afgetoetst kunnen worden.

---

## 0. Wat er wijzigde van v0.9 naar v1.0

v0.9 (2 september) was een concept, opgebouwd uit de bestaande projectdocumentatie. v1.0 verwerkt de antwoorden die Water-Link op 7 september aan de SAP-partner gaf, plus twee architectuurbeslissingen die sindsdien vastliggen.

**Gesloten open punten**

| v0.9 § 12 | Punt | Nu |
|---|---|---|
| 2 | Opdrachttype-mapping HA/KH/IP | **Gesloten** (§7): HA = huisaansluiting, KH = kolkherstel, IP = inspectieput. Types dienen om te filteren en te rapporteren, **niet** om toe te wijzen; er is geen specialisatie per type |
| 3 | Verplichte velden per sectietype | **Gesloten** (§6.6): er wordt niet met verplichte velden gewerkt maar met **voorwaarden op de statusovergangen** |
| 6 | Notificatiemechanisme | **Gesloten** (§6.11): e-mail naar de werfleiders, uitsluitend bij afkeuring en bij annulering of dringende wijziging; voor de rest een werkbak met teller |
| 7 | Identity-configuratie | **Gesloten** (§5.3): twee issuers, beide uitsluitend authenticatie; rechten blijven in de motor |
| 8 | Bijlagecategorieën als vaste dataset | **Gesloten** (§6.8): negen vaste fotocontexten over vier secties |

**Gewijzigde scope — één punt vraagt aandacht**

- **De doorstroom van goedgekeurde services naar SAP wordt een automatische koppeling, geen bestand.** v0.9 hield de SAP-export bestandsgebaseerd en zette een directe koppeling buiten scope. Water-Link heeft dat op 7 september uitdrukkelijk bijgesteld: de services moeten bij het bereiken van *Te exporteren* automatisch als **entrysheet per opdracht** naar SAP stromen. Dit is de **grootste tijdswinst van het project** — vandaag worden die services handmatig per gemeente geboekt. Zie §3.2, §6.12 en §10.

**Nieuwe hoofdstukken en requirements**

- **Toewijzing op postcode** (§6.3) — postcode → raamcontract → aannemer, als regel, niet als beoordeling. De postcodegebieden per raamcontract moeten beheerbaar zijn.
- **Instroom en vrijgave vanuit SAP** (§6.2) — vrijgave op gebruikersstatus, direct doorsturen, intrekken in overleg, en het gedrag bij een SAP-statuswijziging op een lopend dossier.
- **Annulering** (§6.13) — met verplichte bevestiging door de aannemer.
- **Kaartweergave** (§6.14) — interactief, met handmatige correctie van de locatie.
- **Historiek en onveranderlijkheid** (§6.15) — afgesloten dossiers volledig read-only; correctie enkel via de versiemechaniek.
- **Overgang vanuit de huidige werkwijze** (§6.16) — knipdatum, geen volledige migratie, go-live per aannemer.
- **Volumes en dimensionering** (§1.1) — de cijfers waarop een raming kan steunen.
- **Prioriteiten voor go-live** (§3.4) — zes must-haves, vijf punten voor een volgende fase.

**Gecorrigeerd**

- **§5.2 datalaag.** v0.9 stelde dat de motor-datalaag op PostgreSQL draait. Dat is de *beslissing*, niet de *huidige stand*: de motor-code is vandaag T-SQL, en het hertalen van de repository-laag is bewust uitgesteld ("eerst functionaliteit"). Zie §5.2 en §12.

---

## 1. Context en probleem

Water-Link laat rioolaansluitingen en herstellingen uitvoeren door raamcontractaannemers. De informatiestroom tussen aannemer, opdrachtgever (Water-Link) en gemeente verloopt vandaag via **Excel-bestanden, gedeelde SharePoint-lijsten per aannemer en manuele communicatie**. De werfleider controleert aangeleverde meetstaten via een Excel-tool met macro's, legt hoeveelheden naast foto's, en keurt goed of af. De goedgekeurde services worden daarna **handmatig per gemeente in SAP geboekt**. Dit is arbeidsintensief, foutgevoelig en biedt geen centraal overzicht of audittrail.

Het **Rioolportaal** vervangt die werkwijze door één digitaal portaal dat het volledige dossier centraliseert: van instroom uit SAP en planning, over registratie op het terrein (meetstaat, foto's, wegenis, opmeting), tot validatie door de werfleider en doorstroom richting SAP. De SharePoint-lijsten en Excel-bestanden van de huidige werkwijze **verdwijnen**.

**Betrokken partijen**

- **Water-Link (opdrachtgever):** ontvangt de werkorders uit SAP, plant rioolaansluitingen, valideert ingediende dossiers, laat de services doorstromen naar SAP.
- **Raamcontractaannemer:** twee partijen, **DENYS** en **SOGA**, beide in raamcontract. Plannen hun eigen opdrachten in, voeren uit, registreren werken en bewijslast, dienen in ter validatie.
- **Gemeente:** ontvangt wegenisinformatie (rapport per dossier).

### 1.1 Volumes en dimensionering

Cijfers van Water-Link (7 september 2026), bedoeld om de omvang van de oplossing te kunnen inschatten.

| Grootheid | Waarde |
|---|---|
| Aannemers | 2 (DENYS, SOGA) |
| Gebruikers totaal | ± 37 — ± 7 intern (2 planners, 2 à 3 werfleiders, 1 à 2 administrators) en ± 15 per aannemer (1 administrator, 1 à 2 werfleiders, 10 à 15 terreinmedewerkers) |
| Mobiel gebruik | 20 à 30 terreinmedewerkers op de werf |
| Werkorders | ± 170 per maand · ± 40 per week · ± 2.000 per jaar |
| Gelijktijdig in beeld | ± 120 opdrachten; bij piek 150 à 200 |
| Piekperiodes | Na het kerstverlof en na het bouwverlof — de instroom loopt door terwijl de uitvoering stilligt |
| Foto's | 10 à 20 per opdracht → **20.000 à 40.000 per jaar** |
| Documenten | Beduidend minder; de bewijslast is overwegend fotografisch |
| Doorlooptijd | ± 20 dagen, van doorgave tot afhandeling |

---

## 2. Doelstelling

Een digitaal portaal dat de raamcontractaannemer en de opdrachtgever ondersteunt bij het **registreren, plannen, valideren en doorgeven** van alle informatie rond twee types opdrachten: **rioolaansluitingen** en **herstellingen**.

Concreet vervangt het portaal de huidige Excel/SharePoint-werkwijze en centraliseert het de informatiestromen, met:

- één gedeeld dossier per opdracht als bron van waarheid;
- rolgebaseerde toegang met strikte aannemer-isolatie;
- afdwinging van volledigheid via voorwaarden op de statusovergangen;
- een validatieworkflow met correctie per meetstaatpost en versionering per sectie;
- automatische doorstroom van goedgekeurde services naar SAP.

---

## 3. Scope

### 3.1 In scope (v1)

1. Dossierbeheer met lifecycle (instroom → planning → uitvoering → indiening → validatie → afsluiting).
2. **Instroom uit SAP** met de aannemer al toegewezen, plus batch-import (Excel) en handmatige aanmaak, met asynchrone geocoding.
3. **Toewijzing op postcode** via het raamcontract, met beheerbare postcodegebieden.
4. Planning: HL-week-planning en detailplanning door de aannemer; read-only zichtbaar voor de opdrachtgever.
5. Registratie van uitgevoerde werken via een meetstaat op basis van de raamcontractcatalogus.
6. Bemating (as-built) voor rioolaansluitingen.
7. Gestructureerd fotobeheer in negen vaste contexten, met GPS en afstandscontrole.
8. Wegenisregistratie en een wegenisrapport per dossier voor de gemeente.
9. Validatieworkflow met beoordeling **per sectie**, versionering bij afkeuring, en het werfleider-controlescherm dat meetstaat + afmetingen + foto's samenbrengt met correctie per post.
10. **Automatische doorstroom van goedgekeurde services naar SAP** als entrysheet per opdracht, plus terugmelding van de statussen die een feit vaststellen.
11. Rapportage/KPI-dashboards over de lifecycle.
12. Rolgebaseerde toegang met authenticatie via de identity-laag van de motor.

### 3.2 Buiten scope (v1)

- Facturatieverwerking binnen het portaal.
- Planningsmodule voor klantafspraken — de afspraak met de bewoner wordt buiten het portaal vastgelegd en enkel als datum en tekst meegegeven.
- Een pool of marktplaats waaruit aannemers werk kiezen. Het werk is contractueel toegewezen, niet aangeboden ter aanvaarding.
- Een weigerknop voor de aannemer (§6.6).
- Een afzonderlijke managersrol (§4).

> **Gewijzigd t.o.v. v0.9.** De directe koppeling met SAP stond in v0.9 buiten scope; de export was bestandsgebaseerd. Dat is bijgesteld: de doorstroom van services is nu een **automatische koppeling** en staat in scope (§6.12).

### 3.3 Scope-grens web / mobiel / motor — **open beslissing (§12)**

De volledige oplossing bestaat conceptueel uit drie lagen: de **motor** (business- en datalaag, bestaand — §5), het **web-portaal** (opdrachtgever + aannemer-backoffice) en een **mobiele/terreinclient** (uitvoerder). Dit document beschrijft de functionaliteit van alle drie, maar **welke lagen in deze opdracht aan de externe bouwpartij worden toegewezen is nog niet vastgelegd.** Onderdelen die typisch tot de terreinclient behoren (foto-capture met GPS, invoer bij zwakke verbinding) zijn als zodanig gemarkeerd. Water-Link bevestigt de exacte leveringsomvang vóór aanvang.

### 3.4 Prioriteiten voor go-live

Leidraad van Water-Link: **een go-live moet de huidige Excel-werkwijze effectief vervangen.** Wat dat niet doet, mag later.

**Must-have voor go-live**

| # | Functionaliteit | Waarom onmisbaar |
|---|---|---|
| 1 | Rolmodel met scheiding per aannemer | Vertrouwelijkheidsgrens tussen DENYS en SOGA |
| 2 | Werkorderinstroom uit SAP, met toewijzing op postcode | Einde aan de dubbele ingave |
| 3 | Meetstaat op de servicecatalogus | Vervangt de Excel-meetstaat |
| 4 | Validatieflow met goedkeuring per sectie en versionering | Kwaliteitsborging en bruikbaar bewijsdossier |
| 5 | Foto's met GPS, gekoppeld aan het dossier | Brengt de bewijslast samen in één dossier |
| 6 | Automatische doorstroom van goedgekeurde services naar SAP | Vervangt het manuele boeken per gemeente; de grootste tijdswinst |

**Volgende fase**

| # | Functionaliteit | Toelichting |
|---|---|---|
| 7 | Kaartweergave en navigatie (§6.14) | Functioneel waardevol, maar het proces draait ook zonder |
| 8 | E-mailnotificaties bij afkeuring en annulering (§6.11) | Werkbakken volstaan bij de start |
| 9 | Annuleringsflow met bevestiging (§6.13) | Tot dan verlopen annulaties via overleg en telefoon |
| 10 | Rapportage voor de aannemer (§6.10) | Elke aannemer ziet zijn eigen doorlooptijden en afkeurpercentage |
| 11 | Vergelijkende rapportage voor Water-Link (§6.10) | Vergt eerst enkele maanden reële gegevens |

---

## 4. Actoren en rollen

Zes rollen, verdeeld over twee zijden. Een account kan **meerdere** rollen dragen; de hoofdrol is de eerste in de lijst. Aannemer-rollen zijn gebonden aan één aannemer (`aannemer_id`) en zien uitsluitend de eigen dossiers.

| Rol | Zijde | Kernbevoegdheden |
|---|---|---|
| `Administrator` | Opdrachtgever | Volledig beheer: aannemers, accounts, catalogi, raamcontracten, postcodegebieden, systeeminstellingen, exports |
| `Planner` | Opdrachtgever | HL- en detailplanning (read-only op het planbord van de aannemer), import, planning-export |
| `Werfleider` | Opdrachtgever | Valideren per sectie (goed-/afkeuren), meetstaat-controle, exports meetstaat/wegenis/bemating |
| `AannemerAdministrator` | Aannemer | Beheer eigen aannemer-accounts en eigen tariefstelsel |
| `AannemerWerfleider` | Aannemer | Eigen opdrachten plannen, secties aanbieden ter validatie, afgekeurde secties herwerken |
| `AannemerMedewerker` | Aannemer | Secties invullen op het terrein (wegenis, uitvoering, bemating, meetstaat, foto's) |

Een **afzonderlijke managersrol is niet nodig**; wie op het proces stuurt, gebruikt de rapportage van de interne rollen.

### 4.1 Autorisatieregels (hard)

- **Authenticatie vereist** voor elke functionele route.
- **Rolcheck** per actie: de gebruiker moet minstens één van de toegelaten rollen bezitten.
- **Aannemer-isolatie:** voor elke dossiergebonden actie geldt aanvullend dat de `aannemer_id` van de gebruiker moet overeenkomen met de aannemer van het dossier. Deze check mag niet omzeilbaar zijn via directe URL-toegang. Het gaat om een **vertrouwelijkheidsgrens tussen twee concurrenten** die beide voor Water-Link werken.
- **Eén account, één aannemer.** Er zijn geen gebruikers die voor meerdere aannemers werken. Interne rollen kennen deze beperking niet.
- **Persoonlijke accounts verplicht.** Gedeelde ploegaccounts zijn uitgesloten: elke ingave moet herleidbaar blijven tot een persoon. De aannemer beheert zijn eigen gebruikers zelf — bij terreinmedewerkers is het verloop hoog door ploegwerk met wisselende bezetting.
- **Tarieven zijn niet zichtbaar voor `AannemerMedewerker`.** Die ziet de servicecatalogus met de posten en eenheden die hij mag boeken, maar **niet de eenheidsprijzen**. Prijsinformatie blijft voorbehouden aan `AannemerWerfleider` en `AannemerAdministrator`.
- **De Water-Link-werfleider bewerkt geen sectiegegevens** — hij beoordeelt ze.
- **Bewerkbaarheid volgt de status.** Sectiegegevens zijn enkel bewerkbaar door de aannemer zolang de sectie open staat en de opdracht onderhanden of afgekeurd is.

### 4.2 Schermtoegang

| Scherm | Admin | Planner | Werfleider | Aann. admin | Aann. werfl. | Aann. medew. |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| Dashboard | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Opdrachten / Mijn dossiers | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Planning (planbord) | ✓ | ✓ | — | — | ✓ | — |
| Validatie (goedkeuren/afkeuren) | ✓ | — | ✓ | — | — | — |
| Aanbieden ter validatie | — | — | — | ✓ | ✓ | ✓ |
| Uitvoering | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Kaart | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Rapportage | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Exports | ✓ | ✓ | ✓ | — | — | — |
| Raamcontract | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Beheer (accounts, types, catalogus) | ✓ | ✓ | ✓ | — | — | — |

Op de gedeelde schermen ziet een aannemer **uitsluitend eigen gegevens**. Bij de rapportage: de aannemer ziet zijn eigen doorlooptijden en afkeurpercentage, Water-Link ziet alles en kan beide aannemers vergelijken.

De volledige route-/rolmatrix staat in het **API-contract** (§13) en is bindend voor de motor-interface.

---

## 5. Bouw-fundament en architectuur-constraints

> Dit hoofdstuk bevat de enige harde architectuurvoorschriften. Buiten deze constraints is de bouwpartij vrij in de technische invulling van het portaal.

### 5.1 Bouwen op de SvcMgt-motor (verplicht)

Het portaal wordt gebouwd **bovenop de bestaande SvcMgt-motor** van Water-Link — de status-gedreven business- en datalaag die dossiers, secties, statustransities en autorisatie beheert. Het portaal is een **presentatie- en interactielaag** op die motor, geen zelfstandige applicatie met een eigen datamodel.

Kernprincipes:

- **Eén bron van waarheid.** Status, hoeveelheden, secties en bijlagen leven in de motor. Het portaal leest en schrijft die tabellen via de motor-interface; er wordt **geen tweede datastore** aangelegd voor dossierdata (zie ADR-0001, §13).
- **Afdwinging hoort in de motor.** Businessregels zoals de voorwaarden op statusovergangen, de factuurplicht per post en de statustransitie-bewaking horen thuis in de motor-businesslaag, niet enkel in de UI.
- **Interface = het API-contract.** De endpoints, rollen en statustransities in het API-contract (§13) definiëren het contract tussen portaal en motor.

### 5.2 Datalaag

- **Doelstand:** de motor-datalaag draait op **PostgreSQL**. Die keuze is beslist.
- **Huidige stand — belangrijk voor de planning:** de motor-code is vandaag **T-SQL** (SQL Server). Het hertalen van de repository-laag naar PostgreSQL is bewust **uitgesteld** ten voordele van functionaliteit. De motor-op-PostgreSQL is daarmee een **latere mijlpaal**, afhankelijk van die refactor. Een bouwpartij die op de motor bouwt, moet weten op welke dialect-stand ze aansluit — dit punt af te stemmen met de motor-eigenaar vóór aanvang (§12).
- Het datamodel in §11 is leidend, ongeacht het dialect.
- Alle statuswijzigingen worden **geaudit** (`OpdrachtStatusLog`, `SectieStatusLog`, `AuditLog`): wie, welke overgang, wanneer.

### 5.3 Identity en authenticatie — vastgelegd

De configuratie die in v0.9 nog open stond, is beslist:

- **Twee issuers, beide uitsluitend authenticatie.** Interne Water-Link-gebruikers authenticeren rechtstreeks via de eigen Entra-tenant; externe gebruikers (de aannemers) en machine-consumenten via een **aparte Keycloak-issuer**.
- **AuthN-only.** Geen van beide issuers draagt rechten. De **rechten blijven in de motor** (rollen, aannemer-binding) — dat is wat de aannemer-isolatie afdwingbaar houdt.
- **Deny-by-default** met binding op het identiteits-subject: een token zonder gekende binding in de motor krijgt geen toegang.
- De oorspronkelijke brondocumenten vermeldden nog *Azure SQL + NextAuth/MSAL* als beoogde stack. Die aanname is **achterhaald**. Voor de bouwpartij geldt: reken op OIDC-integratie met twee issuers, niet op een eigen accountsysteem.
- Voor machine-consumenten bestaat in de motor al een werkend patroon: een confidential client per consument, met rechten per EntitySet en verb.

### 5.4 Bestandsopslag

- Foto's en documenten worden als bijlage bij de sectie bewaard. In de referentie-implementatie gebeurt dat als base64 in de databank; dit is **expliciet tijdelijk**. De doelarchitectuur verplaatst binaire bijlagen naar **objectopslag** met enkel een referentie in de databank. De bouwpartij levert de objectopslag-variant, niet base64-in-DB.
- **Eén bewaarplaats per bestand.** Dit is een harde eis: elk bestand staat op precies één plek, met verwijzingen vanuit de andere systemen. Twee kopieën van dezelfde foto lopen na verloop van tijd uiteen.
- **Niet alles hoeft in SAP terecht te komen.** Met 20.000 à 40.000 foto's per jaar (§1.1) vraagt dit een afweging tussen SAP, een documentopslag en het portaal zelf. De keuze wordt met de IT-dienst van Water-Link bepaald (§12).
- Bestaande documenten in SharePoint worden **gekoppeld, niet gemigreerd** (§6.16).

### 5.5 Integraties (bestaand in de motor/referentie)

- **GIPOD** — melding van grondwerken aan de Vlaamse overheid (velden `gipod_*` op het dossier). Zie §6.17 en de aparte GIPOD-API-aanvraag (§13).
- **EA-model** — het datamodel is afgeleid van een Enterprise Architect-model; wijzigingen aan het datamodel volgen dat model.

---

## 6. Functionele requirements

Elke capability verwijst naar de high-level requirement (HLR 1–10) en de user stories (US #1–#21). Acceptatiecriteria zijn concreet waar de bron dat toelaat; waar de bron een gat laat, staat **[te bevestigen]** met een verwijzing naar §12.

### 6.1 Dossierbeheer (HLR 1 — US #1, #2, #3)

Het portaal is dossiergericht: één dossier per rioolwerkitem, met adres, aannemer, type en status.

- **US #1** — De opdrachtgever (Planner/Administrator) maakt een dossier aan met **type** (rioolaansluiting of herstelling), foto's en voorbereidende info, zodat de aannemer volledig geïnformeerd kan starten.
  - *Acceptatie:* aanmaak vereist type, adresvelden en aannemer; voorbereidende bijlagen kunnen worden toegevoegd; het dossier start in status `Doorgegeven`.
- **US #2** — Een aannemer ziet **uitsluitend** de eigen dossiers (aannemer-isolatie, §4.1).
- **US #3** — De opdrachtgever volgt de volledige lifecycle van elk dossier (§8).

### 6.2 Instroom en vrijgave vanuit SAP

De normale weg waarlangs een dossier ontstaat.

- **FR-1.** Een werkorder komt uit SAP **met de aannemer er al bij**: de toewijzing gebeurt in SAP vóór het doorsturen (§6.3).
- **FR-2.** De vrijgave gebeurt op basis van een **gebruikersstatus in SAP**. Draagt de werkorder die status, dan wordt hij **direct doorgestuurd** — er is géén bijkomende interne goedkeuringsstap. *Welke gebruikersstatus precies, is met de SAP-partner vast te leggen* **[te bevestigen §12]**.
- **FR-3.** Uit SAP komen mee: kopgegevens (code, opdrachttype, titel, omschrijving, toegewezen aannemer, status), locatie (gemeente, postcode, straat, huisnummer), data en planning (doorgavedatum, plandatum, uitvoeringsdatum, exportdatum) en — optioneel — de **afspraak met de bewoner** (datum en tekst). Niet elke werkorder heeft er een.
- **FR-4.** De **analysefoto's** uit SAP komen mee en horen bij de sectie Meetstaat. Het samenbrengen van analyse- en uitvoeringsfoto's in één dossier is een van de kerndoelen van het portaal.
- **FR-5.** Een vrijgegeven werkorder kan **opnieuw ingetrokken** worden, maar altijd **in overleg met de aannemer** — bij een vergissing of een annulatie. Herverdelen naar een andere aannemer is niet mogelijk: de postcode bepaalt de aannemer.
- **FR-6.** Gedrag bij een SAP-statuswijziging op een bestaand dossier:

| Situatie in het portaal | Gedrag |
|---|---|
| Nog niet in uitvoering | Het portaal volgt automatisch |
| Al onderhanden of verder | **Niet overschrijven.** De wijziging wordt getoond als signaal aan de werfleider, die beslist |
| Annulering vanuit SAP | Volgt de annuleringsprocedure (§6.13) |

  *Grond voor deze regel:* vanaf `OnderhandenGenomen` bevat het dossier metingen en foto's die nergens anders bestaan. Die mogen niet verloren gaan door een wijziging van buitenaf.

- **FR-7.** Naast de SAP-instroom blijft **batch-import** via Excel (`.xlsx`) bestaan voor de Planner/Administrator, met verplichte kolommen `OpdrachtCode`, `OpdrachtType`, `Titel`, `Beschrijving`, `Gemeente`, `Postcode`, `Straat`, `Huisnr`, `AannemerCode`, en handmatige aanmaak.

### 6.3 Toewijzing op postcode

- **FR-8.** Elke aannemer heeft in zijn raamcontract een **aantal postcodes onder contract**. De locatie van de werken bepaalt daarmee rechtstreeks welke aannemer de opdracht krijgt: **postcode → raamcontract → aannemer**.
- **FR-9.** Dat is een **regel, geen beoordeling**: er komt geen planner aan te pas.
- **FR-10.** De **postcodegebieden per raamcontract moeten beheerbaar zijn** (rol Administrator) — contracten worden hernieuwd en werkgebieden verschuiven.
- **FR-11.** Het **raamcontract is het enige bepalende criterium**, en daarbinnen het postcodegebied. Er is **geen specialisatie per opdrachttype**: beide aannemers voeren alle types uit binnen hun eigen gebied.
- **FR-12.** Het raamcontract stuurt daarnaast de afrekening: het bepaalt welke servicecatalogus en welke eenheidsprijzen gelden bij het invullen van de meetstaat (§6.9).

> **Gat in de referentie-implementatie.** Het huidige datamodel kent **geen postcodegebied**. Dit is nieuw te bouwen — zie §11.

### 6.4 Planning (HLR 2, 7 — US #4, #5, #14)

Twee lagen: **HL-planning** (weekniveau) en **detailplanning** (concrete datum), gevolgd door toewijzing voor uitvoering.

| Stap | Wie | Transitie |
|---|---|---|
| HL-week instellen (bulk) | AannemerWerfleider; Planner read-only | `Doorgegeven` → `HLGepland` |
| Plandatum instellen (bulk) | AannemerWerfleider | → `Gepland` |
| Toewijzen | AannemerWerfleider | `Gepland` → `Toegewezen` |
| Intrekken | Planner, AannemerWerfleider | `Toegewezen` → `Gepland` |

- **FR-13.** **De aannemer plant zelf.** Het planbord is een taak van de `AannemerWerfleider`, in twee trappen, met alle acties als **bulkactie** op een selectie opdrachten. De Water-Link-planner ziet hetzelfde planbord **read-only**, met import- en exportmogelijkheden.
- **FR-14.** Er is **geen uiterste uitvoeringstermijn** per werkorder. Het enige bindende gegeven is de **afspraak met de bewoner**, wanneer die er is. De doorgavedatum is het startpunt van de doorlooptijdmeting.
- **FR-15.** Een ingeplande datum is **vrij te wijzigen** zolang de opdracht niet onderhanden genomen is; er geldt geen termijn vóór de startdatum. **Uitzondering:** ligt er een afspraak met een bewoner vast, dan raakt een verschuiving een derde partij en moet de wijziging zichtbaar zijn voor Water-Link. Elke wijziging wordt gelogd.
- **FR-16.** Naar SAP gaat **enkel de bevestigde uitvoeringsdatum**. De weekplanning en de vooraf geplande datum blijven in het portaal: dat is werkorganisatie van de aannemer die geregeld opschuift.
- **Planning-export (CSV):** kolommen `OpdrachtCode, Adres, Gemeente, OpdrachtType, Aannemer, PlanningsWeek, PlanningsDatum, Status`; BOM-prefix voor correcte Excel-weergave.
- **US #5 / #14** — De opdrachtgever ziet de planning **real-time**, met filtering op type, aannemer, periode en status.

### 6.5 Registratie uitgevoerde werken — de vier secties (HLR 3, 4, 6 — US #6, #7, #8, #12)

De aannemer vult **vier secties** aan:

| Sectie | Wat de aannemer registreert |
|---|---|
| **Meetstaat** | Geboekte serviceposten met aantallen, uit de catalogus van het geldende raamcontract. Vervangt de manuele Excel-meetstaat |
| **Wegenis** | Wegenisherstelposten per plaats met aantallen, plus toelichting. Wordt automatisch doorgerekend |
| **Uitvoering** | Uitvoeringsverslag in vrije tekst |
| **Bemating (as-built)** | Maatvoering plus toelichting |

- **FR-17.** **Meetstaat** — regels met een catalogus-item (prijs + eenheid) en een aantal. `DefaultSets` bieden per opdrachttype een **suggestie**, niet een automatische vulling. Afwijkingen en opmerkingen kunnen per registratie worden toegevoegd (US #7).
- **FR-18.** De gebruikte **materialen zitten in de meetstaatposten**: geen vrije materiaallijst, maar gestructureerde posten uit een beheerde catalogus met contractuele eenheidsprijzen.
- **FR-19. Algemene post met factuurplicht.** Wordt materiaal gebruikt waarvoor geen meetstaatpost bestaat, dan mag de aannemer de **algemene post** gebruiken. Die moet **gestaafd worden met een kopie van de factuur in pdf** als bijlage bij de opdracht. **Zonder die kopie is de post niet aanvaardbaar bij de validatie.** Deze regel wordt afgedwongen in de motor (`factuur_verplicht` op het service-item).
- **FR-20.** **Wegenis** — per plaats: type verharding, oppervlakte (m²), plaats (rijweg, parkeerstrook, fietspad, voetpad, verharde berm), laag (toplaag/onderlaag/grondwerk), afmetingen en hersteldatum. Het datamodel ondersteunt zowel eenvoudige m²-regels als een gedetailleerde per-plaats-registratie.
- **FR-21.** **Bemating** — een **kopblad** (type schets TYPE1–4, grondplan, grenzen, uitvoerder, datum) plus **één rij per putje** met maatvoering (`maat_1..10`). Bij gescheiden stelsel (TYPE1–3) twee putjes, bij TYPE4 één.
  > *Noot bij de brondocumenten:* het antwoorddocument spreekt van "vijf maten"; het leidende schema werkt met putjes met `maat_1..10`. **Het schema is leidend** — de vijf-maten-notatie is een oudere beschrijving.
- **FR-22.** Verder registreert de aannemer de **uitvoeringsdatum** en de **foto's** (§6.8).
- **FR-23.** De aannemer kan een werkorder **niet weigeren**: er komt geen weigerknop. Loopt er iets mis op het terrein, dan neemt hij contact op en beslist Water-Link of de toewijzing wordt ingetrokken (FR-5). Vaststellingen horen in het uitvoeringsverslag en de toelichtingsvelden.

### 6.6 Volledigheid: voorwaarden op de statusovergangen

**Dit vervangt open punt 3 van v0.9.** Er wordt **niet** met verplichte velden gewerkt, maar met voorwaarden op de overgangen:

| Overgang | Voorwaarde |
|---|---|
| **Klaar melden** | Alle vier secties zijn minstens `Compleet` |
| **Aanbieden ter validatie** | Alle secties staan `Ter validatie` of zijn goedgekeurd. Bij een eerder afgekeurde sectie is **een antwoord op de opmerking verplicht** |
| **Goedkeuren** | Alle secties zijn beoordeeld en goedgekeurd |
| **Terugsturen** | Alle secties zijn beoordeeld, minstens één is afgekeurd, **met verplichte opmerking** |

- **FR-24.** Deze voorwaarden worden afgedwongen in de **motor-businesslaag**, niet enkel in de UI (§5.1).
- **FR-25.** Afkeuren zonder reden wordt geweigerd, zowel aan de client- als aan de serverzijde.

### 6.7 Validatieworkflow en werfleider-controle (HLR 8 — US #15, #16, #17)

- **FR-26.** De werfleider beoordeelt **per sectie**, niet met één oordeel over het hele dossier. Elke sectie krijgt een eigen oordeel (goedkeuren/afkeuren) met een eigen opmerkingveld.
- **FR-27.** De **opdrachtstatus wordt afgeleid**: pas als elke sectie beoordeeld is → alle goedgekeurd geeft `Te exporteren`, minstens één afgekeurd geeft `Afgekeurd`, met de afgekeurde secties in het logboek. Bewust niet meteen afkeuren bij de eerste afkeuring — anders wordt het scherm read-only vóór de werfleider de rest kan beoordelen.
- **FR-28. Afkeuren maakt een nieuwe versie.** Elke afgekeurde sectie krijgt automatisch een nieuwe versie met een **kopie van de ingevulde gegevens** en de opmerking erbij. De oude versie blijft als historiek bestaan, zodat de volledige beoordelingsgeschiedenis bewaard blijft.
- **Werfleider-controlescherm (ADR-0001, §13):** één scherm dat de doorgegeven services + hoeveelheden + afmetingen naast de **foto's per sectie** legt, met **correctie per bestekpost** (hoeveelheid aannemer → correctie Water-Link → definitief) en de mogelijkheid een ontbrekende post toe te voegen (`toegevoegd_door_wl`). Dit scherm werkt op **dezelfde tabellen** als de rest — geen tweede bron van waarheid.
  - *Noot:* in de referentie bestaan tijdelijk twee validatie-ingangen (`/validaties` en `/controle`); die moeten **geconvergeerd** worden. De bouwpartij levert één validatieflow.

### 6.8 Bewijslast en fotobeheer (HLR 5 — US #10, #11)

**De negen fotocontexten liggen vast.** Dit vervangt open punt 8 van v0.9.

| Sectie | Fotocontexten |
|---|---|
| Meetstaat | Analyse |
| Wegenis | Overzicht wegenis · Detail wegenis |
| Uitvoering | Sleuf · Buislegging · Aansluiting · Afwerking |
| Bemating | Bematingsschets · Maatvoering |

- **FR-29.** Elke foto draagt een **omschrijving, een tijdstip en een GPS-positie**. Het portaal toont de **afstand tussen de foto en het opdrachtadres**, zodat bij de beoordeling zichtbaar is of het beeld van de juiste werf komt. *Terreinclient-functionaliteit.*
- **FR-30.** Daarnaast documenten in pdf, waaronder de **factuurkopie** die een algemene post moet staven (FR-19).
- **FR-31.** Formaten en groottes:

| Type | Formaten | Grootte |
|---|---|---|
| Foto's uit het veld | JPEG, PNG, **HEIC** | tot ± 10 MB per foto |
| Documenten | PDF | tot ± 10 MB |
| Bulk-import (intern) | Excel, CSV | 10 MB |

- **FR-32.** Het gaat om smartphonefoto's, genomen op een werf over een mobiele verbinding. **Compressie bij het uploaden en werken bij een zwakke verbinding wegen zwaarder dan een hoge maximumgrootte.**
- **FR-33.** De aannemer kan **na afsluiting** het dossier en de bijlagen blijven **lezen**. Bij een discussie over facturatie of een latere klacht is het dossier het bewijsmateriaal van beide partijen.

### 6.9 Servicecatalogus en raamcontracten (ondersteunend — HLR 3)

- Aannemers kunnen **enkel** service-items aanrekenen die in hun **raamcontract** zitten.
- **Structuur:** `ServiceCatalogi` (versioned) → hiërarchische `ServiceNodes` (`1 → 1.1 → 1.1.1`) → `ServiceItems` (bladknopen met prijs, eenheid, factuurplicht, gebonden aan één raamcontract). `Raamcontracten` koppelen een aannemer aan prijsafspraken voor een geldigheidsperiode, **en aan een postcodegebied** (FR-10). `DefaultSets` bieden per opdrachttype suggesties.
- Bij openen van een dossier wordt het **meest recente geldige** raamcontract van de aannemer geselecteerd.
- Beheer (catalogi, raamcontracten, postcodegebieden, opdrachttypes): rol Administrator.
- De aannemer kan zijn **eigen raamcontract** met de bijbehorende catalogus inzien; de **tarieven niet voor `AannemerMedewerker`** (§4.1).

### 6.10 Rapportage (volgende fase — §3.4)

KPI's en trends over de lifecycle, op basis van de statuslogs. Aannemers zien enkel eigen data.

- **Globale KPI's:** totaal opdrachten, goedgekeurd, afgekeurd, gemiddelde doorlooptijd (doorgave → goedkeuring).
- **Rapporten:** per aannemer, per opdrachttype, per maand, backlog-trend, per validator, sectie-trend (goed/afgekeurd per versiegroep), doorlooptijd uitvoering → goedkeuring.
- **Filters:** periode, aannemer, type, sectietype, indiener, goedkeurder.
- **Versiegroepen:** secties met versie ≥ 3 worden gegroepeerd als `3+` — te veel herindieningen is een signaal.
- **FR-34.** Vergelijkende rapportage voor Water-Link vergt eerst enkele maanden reële gegevens en komt daarom na de rapportage per aannemer.

### 6.11 Notificaties (volgende fase — §3.4)

**Dit vervangt open punt 6 van v0.9.** Notificaties gaan uitsluitend uit wanneer een **reactie vereist** is, en zijn gericht op de **werfleiders** — dat is de rol die moet reageren.

| Gebeurtenis | Notificatie | Kanaal |
|---|---|---|
| Nieuwe opdracht beschikbaar | **nee** | Werkbak met teller |
| Opdracht afgekeurd | ja | E-mail naar de `AannemerWerfleider` |
| Annulering of dringende wijziging | ja | E-mail, plus telefoon bij korte termijn |
| Werk aangeboden ter validatie | **nee** | Werkbak-teller bij de werfleider |

- **FR-35.** De terreinmedewerker krijgt geen notificaties; hij ziet zijn werk in de mobiele weergave.
- **FR-36.** Voor de go-live volstaan de **werkbakken met teller** (§3.4, punt 8).

### 6.12 Doorstroom naar SAP (HLR 9 — US #18, #19) — **gewijzigd t.o.v. v0.9**

- **FR-37.** Bij het bereiken van `Te exporteren` stromen de **goedgekeurde meetstaat- en wegenisposten** door naar SAP als **entrysheet per opdracht**. Dit moet een **automatische koppeling** worden en **geen uit te wisselen bestand**: vandaag worden die services handmatig per gemeente geboekt, en daar zit de grootste tijdswinst van het project.
- **FR-38.** Enkel **goedgekeurde** secties komen in de doorstroom, zodat er geen niet-goedgekeurd werk in de facturatiestroom terechtkomt.
- **FR-39.** Naar SAP worden **enkel de statussen teruggemeld die een feit vaststellen**:

| Portaalstatus | Naar SAP | Reden |
|---|:-:|---|
| `HLGepland` · `Gepland` | nee | Werkorganisatie van de aannemer |
| `OnderhandenGenomen` | **ja** | Werk is gestart |
| `Uitgevoerd` | **ja** | Werk is uitgevoerd op het terrein |
| `Ter validatie` · `Afgekeurd` | nee | Interne kwaliteitsstap tussen Water-Link en de aannemer |
| `Te exporteren` | **ja** | Goedgekeurd; de services mogen door |
| `Afgehandeld` | **ja** | Definitief afgesloten |

- **FR-40.** Het portaal houdt bij welke dossiers reeds doorgestroomd zijn, zodat **dubbele verwerking** in SAP vermeden wordt (`export_datum` + de overgang `Te exporteren` → `Afgehandeld`).
- **FR-41.** De bestaande bestandsexports blijven bestaan als hulpmiddel (meetstaat, wegenis, bemating in Excel; planning in CSV), met filters op periode, aannemer en type. Ze zijn niet langer het hoofdmechanisme.
- **[te bevestigen §12]** De **veldmapping** van de entrysheet en het exact-eenmalig-mechanisme worden met de SAP-partner vastgelegd.

### 6.13 Annulering en dringende wijzigingen (volgende fase — §3.4)

- **FR-42.** Een annulering **ontstaat in SAP** en stroomt van daaruit door naar het portaal, waar ze zichtbaar wordt voor de aannemer. Verloop:

| Stap | Wat er gebeurt |
|---|---|
| 1. Annulering | De werkorder wordt in SAP geannuleerd |
| 2. Doorstroom | Het portaal ontvangt de annulering en past de status aan |
| 3. Melding | De `AannemerWerfleider` krijgt een notificatie, bij korte termijn aangevuld met telefonisch contact |
| 4. Bevestiging | **De aannemer bevestigt de annulering in het portaal** |

- **FR-43.** De **bevestiging is verplicht**. Zonder bevestiging weet Water-Link niet of de boodschap is aangekomen, met het risico dat een ploeg alsnog uitrukt naar een werf die niet doorgaat.
- **FR-44.** Een geannuleerd werk **blijft zichtbaar** in het portaal, met de **annuleringsreden en het tijdstip**. Er wordt in het portaal niets verwijderd. Hoeveel opdrachten geannuleerd worden en in welke fase is bovendien bruikbare stuurinformatie over de kwaliteit van de voorbereiding.
- **FR-45.** Dringende wijzigingen aan datum, locatie of werkzaamheden:

| Moment van de wijziging | Werkwijze |
|---|---|
| Meer dan 2 werkdagen vóór uitvoering | Wijziging in het portaal, met notificatie naar de werfleider |
| Binnen 2 werkdagen vóór uitvoering | Wijziging in het portaal **én** telefonisch contact |

  *De drempel van twee werkdagen wordt nog afgetoetst bij DENYS en SOGA* **[te bevestigen §12]**.

> **Gat in de referentie-implementatie.** Het huidige datamodel kent **geen annuleringsstatus**. Nieuw te bouwen — zie §8 en §11.

### 6.14 Kaartweergave en geografische informatie (volgende fase — §3.4)

- **FR-46.** De kaart is **functioneel belangrijk maar niet nodig voor de go-live**: het proces draait zonder kaart, de meetstaat en de validatieflow niet.
- **FR-47.** De kaart is **interactief**: filteren op status, opdrachttype, aannemer en nauwkeurigheid, met een detailpaneel per opdracht en doorklik naar het dossier.
- **FR-48.** Is de locatie **onnauwkeurig** bepaald, dan kan een gebruiker de opdracht **handmatig op de juiste plaats zetten**.
- **FR-49.** De mobiele weergave heeft een **eigen kaart met de opdrachten van de dag**, en een **doorverwijzing naar de navigatie-app** van het toestel.
- **FR-50. Geocoding.** Adresgegevens komen gestructureerd uit SAP (straat en huisnummer als aparte velden, naast postcode en gemeente); **GPS-coördinaten niet** — die worden afgeleid uit het adres. Daarbij wordt **bijgehouden hoe nauwkeurig** dat gelukt is: adres, straat, postcode of gemeente. Ontbreekt het huisnummer, dan wordt **het midden van de straat** genomen (nauwkeurigheid op straatniveau).
- **FR-51.** De kaart maakt ook zichtbaar **waar de adresgegevens tekortschieten** — wat extra weegt nu de postcode de toewijzing bepaalt (§6.3).

### 6.15 Historiek en onveranderlijkheid

- **FR-52.** Afgesloten werkorders blijven zichtbaar voor de aannemer, **in lezen** (FR-33).
- **FR-53.** Afgesloten werkorders zijn **volledig read-only**. Bij goedkeuring gaan alle secties naar de eindstatus, en die kent **geen weg terug** naar bewerkbaar.
- **FR-54.** Een correctie na afsluiting gebeurt **niet door te wijzigen**, maar via de **versiemechaniek**: de sectie wordt afgekeurd en krijgt een nieuwe versie met een kopie van de gegevens, terwijl de oude versie bewaard blijft (FR-28).
- **FR-55.** Bewaarbeleid: alles online houden voor de **looptijd van het raamcontract plus de verjaringstermijn** op betwistingen over uitgevoerde werken, daarna archiveren. De exacte termijn wordt vastgelegd samen met het archiveringsbeleid van Water-Link **[te bevestigen §12]**.

### 6.16 Overgang vanuit de huidige werkwijze

- **FR-56. Knipdatum.** Lopende en al doorgestuurde werkorders worden **afgewerkt volgens de bestaande procedure, niet gemigreerd**. Wat nadien uit SAP komt, gaat door het portaal. Met een doorlooptijd van ± 20 dagen is de oude stroom **na ongeveer een maand leeg**.
- **FR-57. Geen volledige historische migratie.** De afgesloten opdrachten uit de huidige opvolging bevatten niet de gestructureerde gegevens die het portaal verwacht — geen sectieversies, geen meetstaatposten uit de catalogus, geen gegeotagde foto's — waardoor migratie lege dossiers zou opleveren. In plaats daarvan wordt een **beperkte set** ingelezen (werkhypothese: het laatste jaar), **uitsluitend om te raadplegen** en duidelijk herkenbaar als voor-portaal.
- **FR-58. SharePoint gekoppeld, niet gemigreerd.** Bestaande documenten blijven staan waar ze staan; het dossier verwijst ernaar, zodat een oud dossier bereikbaar is zonder de bestanden te kopiëren.
- **FR-59. Go-live per aannemer, te beginnen met SOGA.** Omdat een aannemer uitsluitend zijn eigen opdrachten ziet, kunnen beide werkwijzen tijdelijk naast elkaar bestaan zonder elkaar te raken. De eerste overgang levert leerpunten voor de tweede, en het risico blijft beperkt tot de helft van het volume. Na een geslaagde overgang volgt DENYS.
- **FR-60.** De **overgangsperiode is kort** — beperkt tot het uitlopen van de reeds doorgestuurde opdrachten, dus ongeveer een maand.

### 6.17 GIPOD-melding (ondersteunend)

Grondwerken worden gemeld aan GIPOD (Vlaamse overheid); de meldingsstatus wordt op het dossier bijgehouden (`gipod_*`).

- **FR-61.** Twee sporen, niet te verwarren: **lezen** gebeurt via de open-data-dienst van de Vlaamse overheid (geen authenticatie); **melden** gebeurt via de REST-API van Athumi met OAuth2 client-credentials en een **JWT client-assertion**.
- **FR-62.** Een rioolaansluiting is een grondwerk **categorie 3**; de locatie gaat mee als **adres-id uit het Adressenregister**. De melding gebeurt met een handmatige knop per opdracht.
- **FR-63.** De registratie is in de referentie-implementatie gebouwd en werkt als **dry-run**; ze wacht op toegang tot de test-omgeving van de Vlaamse overheid. De productie-omgeving is publieke data waarop wegbeheerders plannen — **er mag daar niet getest worden** (de client weigert productie zonder expliciete vlag).

---

## 7. Opdrachttypes — vastgelegd

**Dit vervangt open punt 2 van v0.9.**

Er bestaan vandaag **drie opdrachttypes**: **HA** huisaansluiting, **KH** kolkherstel en **IP** inspectieput. Functioneel vallen ze onder twee noemers — rioolaansluiting en herstelling — die doorwerken in planning en secties:

| Aspect | Rioolaansluiting | Herstelling |
|---|---|---|
| Aanmaak & datum | Instroom uit SAP; uitvoeringsdatum gekoppeld aan de klantafspraak wanneer die er is | Aannemer plant zelf in |
| Bemating | Vereist | Doorgaans niet |
| Standaard-services | Via `DefaultSet` per opdrachttype | Via `DefaultSet` per opdrachttype |

- **FR-64.** De opdrachttypes dienen om te **filteren en te rapporteren**, **niet om toe te wijzen**. Toewijzen gebeurt op postcode (§6.3).
- **FR-65.** Er is **geen specialisatie per type**: beide aannemers voeren alle opdrachttypes uit binnen hun eigen postcodegebied.
- **[te bevestigen §12]** De per-type **sectievereisten** (welke secties verplicht zijn per type) zijn nog niet hard vastgelegd; de volledigheid wordt vandaag op de statusovergangen geregeld (§6.6).

---

## 8. Statuslifecycles

### 8.1 Opdracht (dossier) — tien statussen

```
Doorgegeven → HLGepland → Gepland → Toegewezen → OnderhandenGenomen
           → Uitgevoerd → Ter validatie → (Afgekeurd ⇄ herwerking)
           → Te exporteren → Afgehandeld
```

| Status | Betekenis | Voor de aannemer |
|---|---|---|
| `Doorgegeven` | Net binnengekomen uit SAP | zichtbaar, niet bewerkbaar |
| `HLGepland` | Grof ingepland op een planningsweek | zichtbaar, planbaar |
| `Gepland` | Concrete plandatum gezet | zichtbaar, planbaar |
| `Toegewezen` | **Vrijgegeven voor uitvoering** | uitvoerbaar |
| `OnderhandenGenomen` | Aannemer is gestart; de secties zijn bewerkbaar | bewerkbaar |
| `Uitgevoerd` | Werk klaar gemeld | — |
| `Ter validatie` | Aangeboden aan de werfleider | read-only |
| `Afgekeurd` | Teruggestuurd voor herwerking | bewerkbaar |
| `Te exporteren` | Goedgekeurd; klaar voor doorstroom naar SAP | read-only |
| `Afgehandeld` | Definitief afgesloten | read-only |

- **FR-66.** Een binnenkomende opdracht is **meteen zichtbaar**, maar pas **uitvoerbaar vanaf `Toegewezen`**.
- **FR-67.** `OnderhandenGenomen` is een **expliciete handeling** van de aannemer en functioneel noodzakelijk: pas daarna worden de secties bewerkbaar. Dat levert twee meetpunten op — start en afronding — en daarmee een echte doorlooptijd.
- **FR-68.** Er wordt **toegewezen aan een aannemer, niet aan een ploeg**. De uitvoeringsdatum volstaat; wie het werk uitvoerde blijkt uit het logboek, waar elke ingave en statuswijziging de gebruiker en het tijdstip draagt.
- **Nieuw te bouwen:** een **annuleringsstatus** met reden, tijdstip en bevestiging (§6.13). Die ontbreekt in de referentie-implementatie.
- Elke overgang wordt gelogd (`OpdrachtStatusLog`, `AuditLog`).

### 8.2 Sectie — zes statussen

```
Draft → Compleet → Ter validatie → OK → Afgehandeld
                                 → NOK → (aannemer past aan) → nieuwe versie (versie+1)
```

De concrete statuscodes zijn configureerbaar (`SectieStatussen`). `Secties` zijn uniek per `(opdracht, sectietype, versie)`. Bij afkeuring gaat een opmerking terug naar de aannemer; herindiening creëert een **nieuwe sectieversie met een kopie van de gegevens** (FR-28). De opdracht schuift pas op als de secties dat samen toelaten (§6.6).

---

## 9. Niet-functionele requirements

| Domein | Requirement |
|---|---|
| **Security** | Rolgebaseerde toegang; aannemer-isolatie niet omzeilbaar; AuthN via twee issuers, rechten in de motor (§5.3); geen dossierdata buiten de motor; persoonlijke accounts verplicht |
| **Privacy / compliance** | AVG-conform (persoonsgegevens van medewerkers, adresgegevens); NIS2-bewust (Water-Link is nutsbedrijf). Geen persoonsgegevens in URL's of logs. Bewaartermijn = raamcontract + verjaringstermijn, exact vast te leggen (FR-55) |
| **Auditeerbaarheid** | Volledige audittrail van statuswijzigingen en dossieracties (wie/wat/wanneer). Elke ingave herleidbaar tot een persoon |
| **Data-integriteit** | Eén bron van waarheid in de motor; afdwinging van de statusovergangvoorwaarden en de factuurplicht in de businesslaag; één bewaarplaats per bestand (§5.4) |
| **Dimensionering** | ± 37 gebruikers, waarvan 20 à 30 mobiel; ± 170 werkorders per maand; 120 tot 200 gelijktijdig; 20.000 à 40.000 foto's per jaar (§1.1). Rekening houden met de piek ná verlofperiodes |
| **Terreingebruik** | Foto-capture met GPS; **compressie bij upload en werken bij een zwakke mobiele verbinding wegen zwaarder dan een hoge maximumbestandsgrootte** (FR-32). Offline-gedrag **[te bevestigen §12]**, afhankelijk van de scope-beslissing (§3.3) |
| **Beschikbaarheid / hosting** | Draait op de Water-Link-tenant/motor-infrastructuur; concrete SLA **[te bevestigen §12]** |
| **Bestandsopslag** | Binaire bijlagen in objectopslag, niet base64-in-DB (§5.4) |
| **Taal** | Nederlandstalige UI |

---

## 10. Integraties (overzicht)

| Integratie | Richting | Vorm | Status |
|---|---|---|---|
| **SvcMgt-motor** | Portaal ↔ motor | Bindend API-contract (§13) | Bestaand |
| **SAP — instroom werkorders** | SAP → portaal | Vrijgave op gebruikersstatus, direct doorsturen | **Te bouwen** |
| **SAP — doorstroom services** | Portaal → SAP | **Automatische koppeling**, entrysheet per opdracht | **Te bouwen — must-have** |
| **SAP — statusterugmelding** | Portaal → SAP | Enkel de feit-vaststellende statussen (FR-39) | **Te bouwen** |
| **SAP — analysefoto's** | SAP → portaal | Bijlagen bij de sectie Meetstaat | **Te bouwen** |
| **Identity** | AuthN | Twee OIDC-issuers, AuthN-only (§5.3) | Vastgelegd |
| **Objectopslag** | Bijlagen | Eén bewaarplaats per bestand | Keuze met IT (§12) |
| **SharePoint (bestaand)** | Verwijzing | Gekoppeld, niet gemigreerd (FR-58) | Bestaand |
| **GIPOD** | Portaal → Vlaamse overheid | REST + OAuth2 JWT-assertion (§6.17) | Gebouwd als dry-run; wacht op toegang |
| **EA-model** | Bron datamodel | — | Bestaand |

---

## 11. Datamodel (samenvatting)

Het volledige, leidende schema staat in `mockup/schema-pg.sql`. Kern-entiteiten, logisch gegroepeerd:

**Accounts & rollen** — `Aannemers`; `Accounts` (hoofdrol + `aannemer_id`); `Account_Rollen` (n rollen per account).

**Opdracht & referenties** — `Opdrachten` (code, type, adres, datum-velden, status, `planningsweek_id`, `aannemer_id`, `geotag_id`, `gipod_*`); `OpdrachtTypes`; `Planningsweken`; `Eenheden`; `Wegenis_Items`; `Geotag` (gedeeld door dossier én foto); `OpdrachtStatusLog`; `AuditLog`; `Systeeminstellingen`.

**Secties (supertype + subtypes)** — `SectieType`; `SectieStatussen`; `Secties` (opdracht, type, versie, status; uniek per opdracht+type+versie); `SectieStatusLog`; subtypes:
- `Sectie_Wegenis` → `Sectie_Wegenis_Items` (m²-regels) + `Sectie_Wegenis_Regel` (per-plaats-detail);
- `Sectie_Uitvoeringsverslag`;
- `Sectie_AsBuilt` (kopblad) → `Sectie_AsBuilt_Putje` (`maat_1..10` per putje);
- `Sectie_ServiceItems` → `Sectie_ServiceItems_Items` (item, aantal, `correctie_aantal`, `toegevoegd_door_wl`);
- `Sectie_Voorbereiding`.

**Servicecatalogus** — `ServiceCatalogi`; `ServiceNodes` (hiërarchisch); `Raamcontracten` (per aannemer); `ServiceItems` (prijs, eenheid, factuurplicht, per raamcontract); `DefaultSets` → `DefaultSet_Items`.

**Bijlagen** — `Sectie_Bijlagen` (aan sectie + categorie; `data` tijdelijk base64 → objectopslag; eigen `geotag_id`; `sp_item_id`); `BijlageCategorie` ↔ `SectieType_BijlageCategorie` (m:n, volgorde); `Sharepoint_ListItem`.

### 11.1 Nieuw te bouwen — gaten in het huidige model

Bij het opstellen van v1.0 zijn vier gaten vastgesteld tussen de bedrijfsregels en het bestaande model. Ze zijn hier expliciet gemaakt zodat ze niet in de bouw als verrassing opduiken.

| # | Gat | Nodig voor |
|---|---|---|
| 1 | **Geen postcodegebied** per raamcontract | De toewijzingsregel postcode → raamcontract → aannemer, en het beheer daarvan (FR-8 t.e.m. FR-11) |
| 2 | **Geen annuleringsstatus** met reden, tijdstip en bevestiging | De annuleringsflow (FR-42 t.e.m. FR-44) |
| 3 | **Geen nauwkeurigheidsniveau** op de geocoding | Het onderscheid adres / straat / postcode / gemeente en de handmatige correctie (FR-48, FR-50) |
| 4 | **E-mail als verplichte unieke accountsleutel** | Terreinmedewerkers hebben niet altijd een werk-e-mailadres; met hoog verloop en persoonlijke accounts is dit een praktische grens **[te bevestigen §12]** |

> **Docs-vs-schema-drift:** enkele oudere domeindocumenten lopen achter op het schema (o.a. bemating "5 maten" → putjes met `maat_1..10`; SQL Server-notatie; extra tabellen `Sectie_Wegenis_Regel`, `Sectie_AsBuilt_Putje`, `AuditLog`, GIPOD-velden). **Het schema is leidend, niet de oudere domeindocs.**

---

## 12. Aannames, afhankelijkheden en open punten

Vijf punten uit v0.9 zijn gesloten (§0). Wat hieronder staat, moet met Water-Link, de motor-eigenaar of de SAP-partner bevestigd worden vóór of vroeg in de bouw.

### Bij de SAP-partner

| # | Open punt | Impact |
|---|---|---|
| 1 | **Welke SAP-gebruikersstatus** de vrijgave naar het portaal triggert — FR-2 | Instroom-implementatie |
| 2 | **Veldmapping van de entrysheet** en het exact-eenmalig-mechanisme — FR-37, FR-40 | De must-have-koppeling |
| 3 | **Timing.** De go-live hangt af van de implementatie van **SAP S/4HANA** bij Water-Link; het portaal steunt op de SAP-omgeving zoals die er ná die implementatie uitziet. **Er is geen streefdatum** | Planning van het volledige traject |

### Bij Water-Link

| # | Open punt | Impact |
|---|---|---|
| 4 | **Leveringsomvang** (web / mobiel / motor) — §3.3 | Bepaalt de scope van de opdracht |
| 5 | **Documentopslag-keuze** met de IT-dienst: SAP, een documentopslag of het portaal — §5.4. Eén eis staat los van die keuze: één bewaarplaats per bestand | Bijlage-architectuur; 20.000 à 40.000 foto's per jaar |
| 6 | **Bewaartermijn en archiveringsbeleid** — FR-55 | Compliance en opslagkosten |
| 7 | **Wegenisrapport** voor de gemeente (opmaak, inhoud, formaat) — §6.5 | Nieuwe rapportfunctie |
| 8 | **Per-type sectievereisten** — §7 | Validatieregels |
| 9 | **SLA en offline-gedrag** — §9 | Infrastructuur en terreinclient |
| 10 | **E-mail als accountsleutel** — §11.1 punt 4 | Accountbeheer bij hoog verloop |

### Bij de aannemers

| # | Open punt | Impact |
|---|---|---|
| 11 | **De drempel van twee werkdagen** voor dringende wijzigingen, af te toetsen bij DENYS en SOGA op basis van het moment waarop een ploeg praktisch niet meer omgeboekt kan worden — FR-45 | Notificatieregel |

### Bij de motor-eigenaar

| # | Open punt | Impact |
|---|---|---|
| 12 | **Dialect-stand van de motor.** PostgreSQL is beslist als doelstand, maar de motor-code is vandaag T-SQL en de refactor is uitgesteld — §5.2 | Op welke stand de bouwpartij aansluit |
| 13 | **Convergentie van de twee validatie-ingangen** (`/validaties` en `/controle`) naar één flow in de businesslaag — §6.7 | Eén validatieflow, geen tweede bron van waarheid |

---

## 13. Bijlagen en referenties

| Document | Inhoud |
|---|---|
| `Requirements/Antwoorden vragenlijst SAP-partner (2026-09-07)` | **Bron van de volumes, bedrijfsregels en prioriteiten in v1.0.** Antwoorden van Water-Link op de vragenlijst RioPortaal van 1 september 2026 |
| `Domein/Doelstelling en Requirements.md` | Doelstelling + 10 HLR's (brondocument) |
| `Requirements/User Stories.md` | 21 user stories, gekoppeld aan HLR's + GitHub Issues |
| `Domein/Opdrachten.md` | Dossier-lifecycle, import |
| `Domein/Secties.md` | Versioned sectie-patroon |
| `Domein/Servicecatalogus.md` | Raamcontracten, catalogi, service-items |
| `Domein/Planning en export.md` | Planning + exports |
| `Domein/Rapportage.md` | KPI-dashboards |
| `Domein/Rollen en accounts.md` | Rollenmodel + autorisatie |
| `Software-architectuur/API-contract.md` | **Bindend** endpoint-/rol-/transitiecontract |
| `Software-architectuur/Stack en DB-laag.md` | Architectuur referentie-implementatie |
| `rioolportaal/docs/adr/0001-werfleider-controle-parallel-scherm.md` | ADR werfleider-controlescherm |
| `rioolportaal/docs/gipod/AANVRAAG-GIPOD-API-CLIENT.md` | GIPOD-integratie |
| `mockup/schema-pg.sql` | **Leidend** datamodel |
| `mockup/` (referentie-implementatie) | Werkende demo — illustratief, geen specificatie |

---

*Einde document. v1.0 verwerkt de antwoorden van Water-Link van 7 september 2026 en de architectuurbeslissingen rond identity en datastore. De open punten in §12 worden afgestemd vóór of vroeg in de bouw.*
