# React Native (mobiele client)

#rioolportaal #architectuur #mobile

Terug naar [[Overzicht]]

---

## Locatie

`apps/rioolportaal/mobile/` — Expo blank template (SDK 56)

---

## Mapstructuur

```
mobile/
├── App.js                        ← navigatie + AuthProvider
└── src/
    ├── api/
    │   ├── config.js             ← BASE_URL (lokaal IP instellen)
    │   └── client.js             ← axios-instantie + alle API-calls
    ├── context/
    │   └── AuthContext.js        ← login/logout + JWT in AsyncStorage
    └── screens/
        ├── LoginScreen.js        ← account-dropdown per rol
        ├── OpdrachtenScreen.js   ← lijst + zoeken + pull-to-refresh
        └── OpdrachtScreen.js     ← detail + sectie-acties per rol
```

---

## Dependencies

| Package | Doel |
|---|---|
| `@react-navigation/native` + `native-stack` | Schermnavigatie |
| `react-native-screens` + `safe-area-context` | Navigatie-vereisten |
| `axios` | HTTP-client naar de mockup-API |
| `@react-native-async-storage/async-storage` | JWT-opslag op het apparaat |

---

## Auth-flow

1. App laadt → `AuthContext` leest JWT uit `AsyncStorage`
2. Geen token → `LoginScreen` (account-dropdown, geen wachtwoord in mockup)
3. POST `/login` met `Accept: application/json` → server geeft `{ token, user }`
4. Token opgeslagen in `AsyncStorage`; alle volgende requests krijgen `Authorization: Bearer <token>`
5. Logout → token verwijderd, terug naar `LoginScreen`

---

## Geïmplementeerde schermen

### LoginScreen
Haalt accounts op via GET `/login` (JSON). Groepeert per rol. Tik op een account om in te loggen.

### OpdrachtenScreen
- Lijst van alle opdrachten (gefilterd op eigen aannemer indien van toepassing)
- Zoekbalk op code, titel of gemeente
- Pull-to-refresh
- Tik → `OpdrachtScreen`

### OpdrachtScreen
Toont opdrachtdetail met 4 secties (wegenis, uitvoering, as-built, meetstaat). Acties zijn rol-afhankelijk:

| Rol | Actie |
|---|---|
| AannemerMedewerker | Sectie afsluiten als compleet |
| AannemerWerfleider | Sectie ter validatie indienen, opdracht ter validatie indienen |
| Werfleider / Administrator | Sectie goedkeuren of afkeuren |

---

## Starten (ontwikkeling)

1. Zoek lokaal IP: `ipconfig` → IPv4-adres van je machine
2. Zet IP in `src/api/config.js`
3. Zorg dat de mockup-server draait: `node server.js` (poort 3003)
4. `cd mobile && npx expo start`
5. Scan QR-code met **Expo Go** op tablet of telefoon

---

## Zie ook

- [[Presentielaag]] — architectuurbeslissingen
- [[API-contract]] — alle endpoints
- [[Stack en DB-laag]] — server + middleware
