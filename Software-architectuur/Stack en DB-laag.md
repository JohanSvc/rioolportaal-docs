# Stack en DB-laag

#rioolportaal #architectuur

Terug naar [[Overzicht]]

---

## Mockup-stack (huidig)

| Laag | Technologie |
|---|---|
| Server | Node.js + Express |
| Templates | EJS (`views/*.ejs`) via `layout.ejs` — wordt niet behouden in productie |
| Database | SQL Server (via `db-mssql.js`) |
| Auth | express-session (mockup); wordt JWT in productie — zie [[Presentielaag]] |
| Upload | multer (memoryStorage) |
| Excel | ExcelJS (import + export) |
| EA-model | better-sqlite3 (read-only op `.qea`) |

## Productie-stack (gepland)

| Laag | Technologie |
|---|---|
| Server | Next.js 16 (App Router) |
| Database | SQL Server via Prisma 5 |
| Auth | NextAuth v5 |

---

## db-mssql.js

Dunne wrapper die een MSSQL-achtige interface biedt:

| Methode | Retourneert | Gebruik |
|---|---|---|
| `mdb.get(sql, params)` | Één rij of `null` | SELECT TOP 1 / by PK |
| `mdb.all(sql, params)` | Array van rijen | SELECT meerdere rijen |
| `mdb.run(sql, params)` | `{ lastInsertId, changes }` | INSERT / UPDATE / DELETE |

Parameters via `?`-placeholders (positional).

---

## API-laag (VS001)

`server.js` is herschreven van een 2286-regels monoliet naar een slanke orchestrator (~65 regels). Alle routes zitten in afzonderlijke modules:

| Module | Pad | Verantwoordelijkheid |
|---|---|---|
| `auth` | `api/auth.js` | login (userId-dropdown), logout |
| `opdrachten` | `api/opdrachten.js` | lijst, CRUD, import, secties, foto's, statustransities |
| `planning` | `api/planning.js` | planningsoverzicht, toewijzen, statuswijziging |
| `validaties` | `api/validaties.js` | werfleider-validatielijst + oordeel |
| `ter-validatie` | `api/ter-validatie.js` | aannemer ter-validatie overzicht |
| `uitvoering` | `api/uitvoering.js` | uitvoeringsoverzicht aannemer |
| `exports` | `api/exports.js` | Excel-export + afhandelen |
| `admin` | `api/admin.js` | accounts, types, catalogi, instellingen |
| `raamcontracten` | `api/raamcontracten.js` | contractbeheer |
| `kaart` | `api/kaart.js` | kaartpagina + `/kaart/data` JSON-endpoint |
| `rapportage` | `api/rapportage.js` | samenvatting + `/rapportage/data` JSON-endpoint |

**Ondersteunende modules:**

| Module | Pad | Inhoud |
|---|---|---|
| Auth-helpers | `middleware/auth.js` | `auth`, `rol`, `hasRol` — ondersteunt sessie én JWT |
| JWT | `middleware/jwt.js` | `signToken(user)`, `verifyToken(token)`, `verifyJwt` middleware, `jwtRol` middleware |
| Navigatie | `nav.js` | `NAV`-object per rol + `mergeNav(rollen)` |
| Seed | `seed.js` | `seedIfEmpty(mdb, repo)` + `makePdf()` |

Elke route-module is een factory-functie: `module.exports = function(mdb, repo) { return router; }`.

Zie [[Presentielaag]] voor de architectuurbeslissingen achter deze opsplitsing.

---

## Auth-patroon

```js
const auth = (req, res, next) =>
  req.session.user ? next() : res.redirect('/login');

const rol = (...rollen) => (req, res, next) =>
  (req.session.user?.rollen || [req.session.user?.rol]).some(r => rollen.includes(r))
    ? next() : res.status(403).send('Geen toegang');
```

Gedefinieerd in `middleware/auth.js`. Checkt eerst sessie, daarna JWT-cookie (`jwt`), daarna `Authorization: Bearer`-header — backwards compatible.

Login in de mockup werkt via een userId-dropdown (geen wachtwoord). De `Accounts`-tabel heeft kolommen `id`, `naam`, `email`, `rol`, `status`, `aannemer_id`.

JWT wordt uitgegeven bij login via `middleware/jwt.js` (`signToken`). Token-geldigheid: 8 uur. Browser ontvangt een httpOnly-cookie; API-clients (React Native, Electron) lezen het token uit de JSON-response. Zie [[API-contract]] voor het login-endpoint.

---

## Repository-laag

Tussen routes en `db-mssql.js` zit een repository-laag (`mockup/repositories/`):

| Repository | Verantwoordelijkheid |
|---|---|
| `OpdrachtenRepository` | CRUD + statuswijzigingen voor opdrachten |
| `SectiesRepository` | Secties, bijlagen, versies |
| `AdminRepository` | Accounts, aannemers, raamcontracten, service-items |
| `ExportRepository` | Exportdata (planning, meetstaat) |
| `RapportageRepository` | KPI's, statistieken |

**Factory:** `repositories/index.js` exporteert `createRepositories(db)` → geeft een object terug met `opdrachten`, `secties`, `admin`, `exports`, `rapportage`.

**Gebruik in routes:**
```js
const repo = createRepositories(mdb);
const opdracht = await repo.opdrachten.findOne(id);
```

`db-mssql.js` blijft de dunne SQL-wrapper (`get`, `all`, `run`, `exec`, `transaction`). Repositories bevatten alle domein-specifieke SQL — routes roepen geen `mdb`-methodes meer rechtstreeks aan.

---

## Zie ook

- [[Presentielaag]]
- [[EA-navigatie]]
- [[Overzicht]]
