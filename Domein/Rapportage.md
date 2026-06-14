# Rapportage

#rioolportaal #domein

Terug naar [[Overzicht]]

Het rapportage-scherm (`/rapportage`) toont KPI's en trends over de volledige opdracht-lifecycle. Beschikbaar voor Werfleider, Administrator, AannemerWerfleider en AannemerAdministrator (aannemer ziet enkel eigen data).

---

## Databron

Alle rapportage-data komt uit `OpdrachtStatusLog` (tijdstip per statusovergang per opdracht) en `SectieStatusLog` (tijdstip per sectie-overgang). Via een CTE (`Fasen`) worden sleuteltijdstippen per opdracht berekend:

| Veld | Betekenis |
|---|---|
| `ts_dg` | Tijdstip Doorgegeven |
| `ts_tw` | Tijdstip Toegewezen |
| `ts_ui` | Tijdstip Uitgevoerd |
| `ts_tv` | Tijdstip Ter validatie |
| `ts_gk` | Tijdstip Te exporteren (= goedgekeurd) |
| `ts_ak` | Tijdstip Afgekeurd |

---

## KPI's (globaal)

| KPI | Berekening |
|---|---|
| Totaal opdrachten | COUNT in periode |
| Goedgekeurd | COUNT waar `ts_gk IS NOT NULL` |
| Afgekeurd | COUNT waar `ts_ak IS NOT NULL` |
| Gem. doorlooptijd | AVG(dagen `ts_dg` → `ts_gk`) |

---

## Rapporten

| Rapport | Dimensie | Metrics |
|---|---|---|
| **Per aannemer** | Aannemer | Totaal, goedgekeurd, afgekeurd, gem. toewijzing, uitvoering, validatie, totaal |
| **Per opdrachttype** | Opdrachttype | Totaal, goedgekeurd, afgekeurd, gem. uitvoering, validatie, totaal |
| **Per maand** | Maand (yyyy-MM) | Totaal, goedgekeurd, afgekeurd, gem. doorlooptijd |
| **Backlog-trend** | Maand | Backlog (open bij start maand) + doorvoer + gem. doorlooptijd DG→UI |
| **Per validator** | Werfleider | Goedgekeurd, afgekeurd, totaal validaties |
| **Sectie-trend** | Maand + versie + uitkomst | Aantal secties per OK/NOK per versiegroep |
| **UV→GK doorlooptijd** | Maand + versiegroep | Gem. dagen uitvoering→goedkeuring, per versietraject |

---

## Filters

| Filter | Toepasbaar op |
|---|---|
| `maanden` (terug) | Alle opdrachtrapporten |
| `aannemer_id` | Opdrachtrapporten (geforceerd voor aannemer-rollen) |
| `type_code` | Opdrachtrapporten |
| `adm_sectietype` | Sectie-trend |
| `adm_indiener_id` | Sectie-trend (AannemerWerfleider die indiende) |
| `adm_goedkeurder_id` | Sectie-trend (Werfleider die valideerde) |

---

## Versiegroepen

Secties met versie ≥ 3 worden gegroepeerd als `3+` (te veel herindieningen = problematisch).

## Zie ook

- [[Opdrachten]]
- [[Secties]]
- [[Planning en export]]
