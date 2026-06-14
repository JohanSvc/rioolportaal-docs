# Navigatiestructuur

#rioolportaal #architectuur

Terug naar [[Overzicht]]

De navigatiestructuur is hardcoded in `server.js` als `app.locals.NAV` — één object per rol met gegroepeerde nav-items. Er is geen afhankelijkheid meer van het QEA-bestand of `better-sqlite3`.

---

## Structuur

```js
app.locals.NAV = {
  Administrator:         [ { cat, icon, items: [{ label, route }] }, ... ],
  Planner:               [ ... ],
  Werfleider:            [ ... ],
  AannemerAdministrator: [ ... ],
  AannemerWerfleider:    [ ... ],
  AannemerMedewerker:    [ ... ],
};
```

Categorieën per rol:

| Rol | Categorieën |
|---|---|
| `Administrator` | Opdrachten · Exports · Aannemers en servicecatalogi · Rapportage · Instellingen |
| `Planner` | Planning · Opdrachten |
| `Werfleider` | Validaties · Exports · Opdrachten · Rapportage |
| `AannemerAdministrator` | Aannemers en servicecatalogi · Opdrachten · Rapportage |
| `AannemerWerfleider` | Planning · Uitvoering · Opdrachten · Rapportage |
| `AannemerMedewerker` | Uitvoering · Opdrachten |

---

## `mergeNav(rollen)`

Wanneer een gebruiker meerdere rollen heeft, combineert `mergeNav()` de nav-items van alle rollen (deduplicatie op `label|route`). Sorteer op `NAV_ORDER`:

```js
const NAV_ORDER = [
  'Planning','Uitvoering','Validaties','Exports',
  'Opdrachten','Aannemers en servicecatalogi','Rapportage','Instellingen'
];
```

Gebruik:
```js
const nav = mergeNav(u.rollen || [u.rol]);
// Doorgegeven aan elke render() als `nav`
```

---

## Aanpassen

Om een nav-item toe te voegen of te verwijderen: pas `app.locals.NAV` direct aan in `server.js`. Geen QEA-tagged values meer nodig.

## Zie ook

- [[Stack en DB-laag]]
- [[Rollen en accounts]]
- [[Overzicht]]
