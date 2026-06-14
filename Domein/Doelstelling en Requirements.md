# Doelstelling en High-Level Requirements

#rioolportaal #requirements #doelstelling

Terug naar [[Overzicht]]

> Bron: *Aannemersportaal_Doelstelling_Requirements_v2.docx*

---

## Doelstelling

We wensen een digitaal portaal te bouwen dat de raamcontractaannemer ondersteunt bij het registreren, plannen, valideren en doorgeven van alle informatie rond twee types opdrachten: **rioolaansluitingen** en **herstellingen**.

Het portaal vervangt de huidige werkwijze op basis van Excel, gedeelde OneDrive-mappen en manuele communicatie, en centraliseert de informatiestromen tussen de aannemer, de opdrachtgever en de gemeente.

- Bij **rioolaansluitingen**: de opdrachtgever maakt de opdracht aan én coördineert de klantafspraak
- Bij **herstellingen**: de aannemer plant de uitvoering zelf in — die planning is zichtbaar voor de opdrachtgever
- De opdrachtgever valideert de ingediende dossiers
- Verwerking in SAP blijft buiten het portaal (manueel of via gestructureerde export)

---

## High-Level Requirements

### 1. Dossierbeheer
Het portaal werkt dossiergericht. Elk dossier wordt aangemaakt door de opdrachtgever (incl. foto's en voorbereidende info) en heeft een type: rioolaansluiting of herstelling. De aannemer ziet uitsluitend zijn eigen dossiers.

**Lifecycle:** Aangemaakt → Ingepland → In uitvoering → Ingediend → Gevalideerd → Afgesloten

Zie [[Opdrachten]] voor de volledige lifecycle-implementatie.

---

### 2. Inplanning
- Bij **herstellingen**: aannemer plant zelf in (begin- en einddatum), aanpasbaar zolang niet ingediend
- Bij **rioolaansluitingen**: uitvoeringsdatum gekoppeld aan klantafspraak (vastgelegd door opdrachtgever)
- Planning is real-time zichtbaar voor de opdrachtgever

Zie [[Planning en export]].

---

### 3. Registratie van uitgevoerde werken
De aannemer registreert per dossier welke raamcontractservices werden gebruikt, inclusief eenheden en hoeveelheden. De beschikbare services zijn gebaseerd op een voorgedefinieerde catalogus (consistente eenheden en tarieven). Afwijkingen of opmerkingen kunnen worden toegevoegd.

Zie [[Servicecatalogus]].

---

### 4. Asbuilt en opmeting
Voor rioolaansluitingen laadt de aannemer de opgemeten bemating op als onderdeel van het dossier. Verplichte velden worden afgedwongen vóór indiening — onvolledige dossiers kunnen niet ingediend worden.

Zie [[Secties]].

---

### 5. Bewijslast en fotobeheer
Foto's en documenten worden gestructureerd geüpload en gekoppeld aan het dossier, met categorisatie per fase (voor aanvang / tijdens werken / na uitvoering). Metadata (tijdstip, uploader) worden automatisch opgeslagen. Vervangt de huidige gedeelde OneDrive-map.

---

### 6. Wegenisinformatie
Voor dossiers waarbij wegenis werd opengebroken registreert de aannemer:
- Type verharding
- Oppervlakte
- Locatie
- Hersteldatum

Het portaal genereert een **wegenisrapport** dat aan de gemeente bezorgd kan worden.

Zie [[Secties]].

---

### 7. Planningsoverzicht voor de opdrachtgever
De opdrachtgever beschikt over een overzicht van alle ingeplande herstellingen en rioolaansluitingen, met filtering op type, aannemer, periode en status.

Zie [[Planning en export]].

---

### 8. Validatieworkflow
De opdrachtgever reviewt ingediende dossiers en kan ze:
- **Goedkeuren** → status naar Gevalideerd
- **Terugsturen** met commentaar → aannemer herwerkt

Betrokkenen ontvangen een notificatie bij relevante statuswijzigingen.

Zie [[Secties]] en [[Opdrachten]].

---

### 9. SAP-ondersteuning via export
- Het portaal genereert een gestructureerd exportbestand per dossier of per batch
- Veldmapping wordt eenmalig afgestemd met de SAP-beheerder
- Het portaal houdt bij welke dossiers reeds geëxporteerd zijn (geen dubbele verwerking)
- **Geen directe koppeling met SAP**

Zie [[Planning en export]].

---

### 10. Toegang en beveiliging
- Toegang is **rolgebaseerd**
- Aannemer ziet enkel eigen dossiers
- Opdrachtgever heeft toegang tot alle dossiers en het planningsoverzicht
- Aanmelden via **Microsoft 365** (MSAL)

Zie [[Rollen en accounts]].

---

## Buiten scope — eerste versie

- Directe API-koppeling met SAP
- Facturatieverwerking in het portaal
- Planningsmodule voor klantafspraken

---

## Technologiestack (beoogd productie)

| Component | Technologie |
|---|---|
| Database | Azure SQL |
| Hosting | Azure App Service |
| Bestandsopslag | SharePoint |
| Authenticatie | Microsoft 365 via MSAL |

> De huidige **mockup** gebruikt SQLite + Express + EJS. Zie [[Stack en DB-laag]] voor de mockup-architectuur.

---

## Zie ook

- [[Overzicht]]
- [[Opdrachten]]
- [[Secties]]
- [[Planning en export]]
- [[Servicecatalogus]]
- [[Rollen en accounts]]
- [[Stack en DB-laag]]
