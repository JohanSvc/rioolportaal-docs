# User Stories — Rioolportaal

#rioolportaal #requirements #userstories

Terug naar [[Overzicht]] | Bron: [[Doelstelling en Requirements]]

Bijgehouden als **GitHub Issues** op [github.com/JohanSvc/rioolportaal](https://github.com/JohanSvc/rioolportaal/issues)  
Labels: `HLR-[nr]`, `rol:[rol]`, `status:open/in-progress/done`

---

## HLR 1 — Dossierbeheer

| # | User Story | Rol | Status |
|---|---|---|---|
| [#1](https://github.com/JohanSvc/rioolportaal/issues/1) | Als **Opdrachtgever** wil ik een opdracht aanmaken met type (rioolaansluiting / herstelling), foto's en voorbereidende info zodat de aannemer volledig geïnformeerd kan starten | Opdrachtgever / Planner | open |
| [#2](https://github.com/JohanSvc/rioolportaal/issues/2) | Als **Aannemer** wil ik enkel mijn eigen opdrachten zien zodat ik gefocust kan werken zonder afleiding van andere aannemers | AannemerWerfleider / AannemerMedewerker | open |
| [#3](https://github.com/JohanSvc/rioolportaal/issues/3) | Als **Opdrachtgever** wil ik de volledige lifecycle van een dossier kunnen volgen (aangemaakt → ingepland → in uitvoering → ingediend → gevalideerd → afgesloten) zodat ik altijd het actuele overzicht heb | Planner / Werfleider | open |

---

## HLR 2 — Inplanning

| # | User Story | Rol | Status |
|---|---|---|---|
| [#4](https://github.com/JohanSvc/rioolportaal/issues/4) | Als **AannemerWerfleider** wil ik een herstelling inplannen met begin- en einddatum en die planning later kunnen aanpassen of annuleren zolang het dossier nog niet ingediend is | AannemerWerfleider | open |
| [#5](https://github.com/JohanSvc/rioolportaal/issues/5) | Als **Opdrachtgever** wil ik de ingeplande herstellingen real-time zien zodat ik niet telefonisch hoef op te volgen | Planner | open |

---

## HLR 3 — Registratie van uitgevoerde werken

| # | User Story | Rol | Status |
|---|---|---|---|
| [#6](https://github.com/JohanSvc/rioolportaal/issues/6) | Als **AannemerMedewerker** wil ik per dossier de gebruikte raamcontractservices registreren met eenheden en hoeveelheden uit een vaste catalogus zodat tarieven consistent blijven | AannemerMedewerker | open |
| [#7](https://github.com/JohanSvc/rioolportaal/issues/7) | Als **AannemerMedewerker** wil ik afwijkingen of opmerkingen kunnen toevoegen aan een serviceregistratie zodat de opdrachtgever context heeft bij validatie | AannemerMedewerker | open |

---

## HLR 4 — Asbuilt en opmeting

| # | User Story | Rol | Status |
|---|---|---|---|
| [#8](https://github.com/JohanSvc/rioolportaal/issues/8) | Als **AannemerWerfleider** wil ik de opgemeten bemating uploaden voor een rioolaansluiting zodat het dossier volledig is voor indiening | AannemerWerfleider | open |
| [#9](https://github.com/JohanSvc/rioolportaal/issues/9) | Als **systeem** wil ik verplichte velden afdwingen vóór indiening zodat onvolledige dossiers niet ingediend kunnen worden | — | open |

---

## HLR 5 — Bewijslast en fotobeheer

| # | User Story | Rol | Status |
|---|---|---|---|
| [#10](https://github.com/JohanSvc/rioolportaal/issues/10) | Als **Aannemer** wil ik foto's en documenten gestructureerd uploaden per fase (voor aanvang / tijdens werken / na uitvoering) zodat de bewijslast volledig gedocumenteerd is | AannemerMedewerker / AannemerWerfleider | open |
| [#11](https://github.com/JohanSvc/rioolportaal/issues/11) | Als **systeem** wil ik metadata (tijdstip, uploader) automatisch opslaan bij elke upload zodat traceerbaarheid gegarandeerd is | — | open |

---

## HLR 6 — Wegenisinformatie

| # | User Story | Rol | Status |
|---|---|---|---|
| [#12](https://github.com/JohanSvc/rioolportaal/issues/12) | Als **AannemerWerfleider** wil ik per dossier de opgebroken en herstelde wegenis registreren (type verharding, oppervlakte, locatie, hersteldatum) zodat de gemeente correct geïnformeerd wordt | AannemerWerfleider | open |
| [#13](https://github.com/JohanSvc/rioolportaal/issues/13) | Als **Opdrachtgever** wil ik een wegenisrapport kunnen genereren per dossier zodat ik dit direct aan de gemeente kan bezorgen | Planner / Werfleider | open |

---

## HLR 7 — Planningsoverzicht

| # | User Story | Rol | Status |
|---|---|---|---|
| [#14](https://github.com/JohanSvc/rioolportaal/issues/14) | Als **Opdrachtgever** wil ik een planningsoverzicht zien van alle ingeplande opdrachten met filtering op type, aannemer, periode en status zodat ik het overzicht bewaar zonder afhankelijk te zijn van de aannemer | Planner | open |

---

## HLR 8 — Validatieworkflow

| # | User Story | Rol | Status |
|---|---|---|---|
| [#15](https://github.com/JohanSvc/rioolportaal/issues/15) | Als **Werfleider** wil ik een ingediend dossier goedkeuren zodat het naar de exportwachtrij gaat | Werfleider | open |
| [#16](https://github.com/JohanSvc/rioolportaal/issues/16) | Als **Werfleider** wil ik een ingediend dossier terugsturen met commentaar zodat de aannemer het kan herwerken | Werfleider | open |
| [#17](https://github.com/JohanSvc/rioolportaal/issues/17) | Als **betrokkene** wil ik een notificatie ontvangen bij relevante statuswijzigingen zodat ik niet continu hoef in te loggen om te controleren | Alle rollen | open |

---

## HLR 9 — SAP-export

| # | User Story | Rol | Status |
|---|---|---|---|
| [#18](https://github.com/JohanSvc/rioolportaal/issues/18) | Als **Planner** wil ik een gestructureerd exportbestand genereren per dossier of per batch zodat de SAP-beheerder de verwerking kan doen | Planner / Administrator | open |
| [#19](https://github.com/JohanSvc/rioolportaal/issues/19) | Als **systeem** wil ik bijhouden welke dossiers reeds geëxporteerd zijn zodat dubbele verwerking in SAP vermeden wordt | — | open |

---

## HLR 10 — Toegang en beveiliging

| # | User Story | Rol | Status |
|---|---|---|---|
| [#20](https://github.com/JohanSvc/rioolportaal/issues/20) | Als **Administrator** wil ik gebruikers aanmaken en een rol toewijzen (Aannemer / Opdrachtgever / Werfleider / ...) zodat toegang correct afgebakend is | Administrator | open |
| [#21](https://github.com/JohanSvc/rioolportaal/issues/21) | Als **medewerker** wil ik inloggen via Microsoft 365 zodat ik geen apart wachtwoord nodig heb | Alle rollen | open |

---

## Buiten scope — v1

- Directe API-koppeling met SAP
- Facturatieverwerking in het portaal
- Planningsmodule voor klantafspraken

---

## Zie ook

- [[Doelstelling en Requirements]]
- [[Opdrachten]]
- [[Secties]]
- [[Planning en export]]
- [[Rollen en accounts]]
