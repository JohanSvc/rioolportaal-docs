# Rioolportaal — Overzicht

#rioolportaal #overzicht

Aannemersportaal voor rioolwerken — opdrachtbeheer, planning en meetstaten.

**Status:** mockup-fase (Express + EJS + SQLite)  
**Productie-stack:** Next.js 16, Prisma 5, SQL Server, NextAuth v5  
**Mockup:** `C:\Users\johan\AppDev\apps\rioolportaal\mockup\` → `node server.js` → http://localhost:3003  
**EA-model:** `mockup/Rioolportaal.qea` (SQLite, read-only — drijft de navigatie)

---

## Domeinen

| Domein | Inhoud |
|---|---|
| [[Doelstelling en Requirements]] | Projectdoelstelling + 10 high-level requirements (brondocument v2) |
| [[Opdrachten]] | Kern van de app — lifecycle, import, status, secties |
| [[Secties]] | Versioned sectie-patroon — wegenis, uitvoering, as-built, meetstaat |
| [[Servicecatalogus]] | Raamcontracten, catalogi, servicenodes, service-items |
| [[Rollen en accounts]] | Actoren, rolhiërarchie, auth-patroon |

## Software-architectuur

| Pagina | Inhoud |
|---|---|
| [[Stack en DB-laag]] | Express + EJS, db-mssql.js wrapper, SQLite mockup vs. SQL Server prod |
| [[EA-navigatie]] | QEA-model drijft nav — hoe `buildNav()` werkt |
| [[Presentielaag]] | API-first beslissingen, JWT, roadmap clients |
| [[API-contract]] | Alle endpoints gedocumenteerd |
| [[React Native]] | Mobiele client (Expo SDK 56) |

## Samenwerking

| Pagina | Inhoud |
|---|---|
| [[GitHub-setup]] | Git-repos, workflow, teamleden toevoegen, Obsidian Git plugin |

---

## Actoren

| Actor | Rol in de app |
|---|---|
| **Administrator** | Beheert accounts, instellingen, raamcontracten |
| **Planner** | Plant opdrachten (HL-planning, weekplanning), importeert, exporteert |
| **Werfleider** | Valideert secties (wegenis, uitvoering, as-built, meetstaat) |
| **AannemerAdministrator** | Beheert eigen aannemeraccounts |
| **AannemerWerfleider** | Stuurt aannemer-medewerkers aan, keurt secties goed voor indiening |
| **AannemerMedewerker** | Vult secties in op het terrein |

---

## Opdrachtstatus-lifecycle

```
Doorgegeven → HLGepland → Gepland → OnderhandenGenomen → Uitgevoerd
           → Ter validatie → Te exporteren → Afgehandeld
```

Zie [[Opdrachten]] voor de volledige transitielogica.

---

## Sectiestatus-lifecycle

```
Draft → Compleet → Ter validatie → OK / NOK → Afgehandeld
                                ↗ reply → nieuwe versie
```

Zie [[Secties]] voor versiegedrag en subtabelstructuur.
