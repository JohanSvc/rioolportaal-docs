# Presentielaag

#rioolportaal #architectuur #VS001

Terug naar [[Overzicht]]

---

## Doelstelling

De mockup-server moet meerdere client-types kunnen bedienen zonder de business logica te dupliceren:

- **Windows desktopapp** → Electron
- **Tablet (iPad/Android)** → React Native
- **Smartphone (iOS/Android)** → React Native (zelfde codebase als tablet)

Dit vereist een gedeelde API als contractpunt tussen server en alle clients.

---

## Beslissingen (VS001 — 2026-06-14)

| Beslissing | Keuze | Reden |
|---|---|---|
| Architectuur | API-first | Server wordt pure JSON-API; geen rendering meer |
| EJS | Niet behouden | Mockup-fase is eindpunt van server-side rendering |
| Auth | JWT (productie) | Werkt voor browser, React Native en Electron |
| Mobiel | React Native | Één codebase voor iOS + Android, aansluitend bij JS-stack |
| Desktop | Electron | Roept dezelfde API aan als mobile |

---

## Huidige staat (mockup)

De mockup gebruikt nog **EJS** voor visuele validatie — dit wordt niet meegenomen naar productie.

`server.js` is gereorganiseerd als slim orchestrator. Alle routes zitten in `api/`-modules (zie [[Stack en DB-laag]]). Auth werkt via **JWT** (`middleware/jwt.js`) — browser krijgt een httpOnly-cookie, API-clients (React Native, Electron) gebruiken `Authorization: Bearer`. Express-session blijft als fallback voor de browser-mockup.

---

## Roadmap

- [x] API-laag bouwen: alle routes opgeknipt in `api/*.js` (11 modules), `server.js` ~65 regels
- [x] JWT-middleware toevoegen — `middleware/jwt.js`, cookie + Bearer-header, backwards compatible met sessie
- [x] API-contract documenteren → [[API-contract]]
- [ ] EJS-views en -routes verwijderen (wacht op productie-beslissing)
- [x] React Native prototype — `mobile/` (Expo), AannemerWerfleider-flow: login → opdrachtenlijst → detail → secties → ter validatie
- [ ] Electron wrapper opzetten

---

## Zie ook

- [[Stack en DB-laag]]
- [[Rollen en accounts]]
