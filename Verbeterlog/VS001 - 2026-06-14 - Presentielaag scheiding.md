licht de # VS001 — Presentielaag scheiding

#rioolportaal #verbeterlog #VS001

**Status:** In uitwerking  
**Aangemaakt:** 2026-06-14  
**Laatste update:** 2026-06-14 (React Native prototype klaar)

---

## Focus

Scheiding van business logica en presentatielogica om meerdere clients te ondersteunen: Windows desktopapp, tablet (iPad/Android) en smartphone (iOS/Android).

## Inzichten

- De repository-laag is al presentation-agnostic — goede basis
- EJS-routes mengen momenteel nog presentatie en business logica
- Meerdere presentatielagen vereisen een gedeelde API als contractpunt
- React Native dekt iOS + Android in één codebase en sluit aan bij bestaande JS-stack
- Electron is de logische keuze voor Windows desktop (zelfde API)

## Beslissingen

- **API-first architectuur**: server wordt pure API (`routes/api/*`) — JSON-responses, business logica via repositories, geen rendering
- **EJS wordt niet behouden** — de mockup-fase is het eindpunt van server-side rendering; alle clients worden JS-frontends
- **Auth**: JWT-tokens voor alle clients (browser, React Native, Electron) — geen session-cookies
- **Mobiel**: React Native (één codebase voor iOS + Android)
- **Desktop**: Electron (roept dezelfde API aan)
- Repositories blijven ongewijzigd — die zijn het fundament

## Acties

- [x] API-laag bouwen: alle routes opgeknipt in `api/*.js` (11 modules), `server.js` herleid tot ~65 regels
- [x] JWT-middleware toevoegen — `middleware/jwt.js`, cookie + Bearer-header, backwards compatible met sessie
- [ ] EJS-routes en views verwijderen na API-migratie
- [x] API-contract documenteren → [[API-contract]]
- [x] React Native prototype opzetten — `mobile/` (Expo blank), login + opdrachtenlijst + opdrachtdetail + sectie-acties per rol
- [ ] Electron wrapper opzetten

## Openstaande vragen

_(geen — alle vragen beantwoord)_

## Zie ook

- [[Backlog]]
- [[Stack en DB-laag]]
- [[Presentielaag]]
- [[Rollen en accounts]]
