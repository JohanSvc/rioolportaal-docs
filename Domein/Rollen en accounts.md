# Rollen en accounts

#rioolportaal #domein

Terug naar [[Overzicht]]

---

## Rollen

| Rol | Type | Bevoegdheden |
|---|---|---|
| `Administrator` | Opdrachtgever | Volledig beheer: aannemers, accounts, catalogi, instellingen, exports |
| `Planner` | Opdrachtgever | Planning (HL + weekplanning), import, export planning |
| `Werfleider` | Opdrachtgever | Validaties goedkeuren/afkeuren, exports meetstaten/wegenis/bemating |
| `AannemerAdministrator` | Aannemer | Beheert eigen aannemer-accounts |
| `AannemerWerfleider` | Aannemer | Plant eigen opdrachten, dient meetstaten in ter validatie |
| `AannemerMedewerker` | Aannemer | Vult secties in op het terrein (wegenis, uitvoering, as-built, meetstaat) |

Een account kan meerdere rollen hebben (`Account_Rollen`-tabel). De hoofd-rol (`Accounts.rol`) is de eerste in de lijst.

---

## DB-structuur

```
Accounts
  ├── Account_Rollen (n rollen per account)
  └── Aannemers (enkel voor aannemer-rollen: aannemer_id)
```

---

## Auth-patroon (mockup)

```js
// Authenticatie: sessie aanwezig?
const auth = (req, res, next) =>
  req.session.user ? next() : res.redirect('/login');

// Autorisatie: heeft de gebruiker één van deze rollen?
const rol = (...rollen) => (req, res, next) =>
  (req.session.user?.rollen || [req.session.user?.rol]).some(r => rollen.includes(r))
    ? next() : res.status(403).send('Geen toegang');
```

Gebruik: `app.get('/route', auth, rol('Administrator', 'Planner'), handler)`

**Aannemer-isolatie:** routes controleren aanvullend of `u.aannemer_id` overeenkomt met de opdracht-aannemer:
```js
if (u.aannemer_id && op.aannemer_id !== u.aannemer_id) return res.status(403).send('Geen toegang');
```

---

## Navigatie per rol

De navigatie wordt bij opstart gebouwd vanuit het QEA-model (`buildNav()`). De `mergeNav(rollen)`-functie combineert de navigatie-items van alle rollen van de ingelogde gebruiker.

Zie [[EA-navigatie]] voor het details van `buildNav()`.

---

## Systeeminstellingen

`Systeeminstellingen`-tabel: sleutel/waarde-paren voor SMTP, SharePoint URL, login-beleid (max pogingen, blokkering, sessie-timeout, wachtwoordlengte).

---

## Login (mockup)

In de mockup is er geen wachtwoordcontrole — de gebruiker kiest een account uit een dropdown. In productie: NextAuth v5 met e-mail/wachtwoord.

## Zie ook

- [[EA-navigatie]]
- [[Overzicht]]
