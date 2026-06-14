# GitHub-setup — Rioolportaal

#rioolportaal #github #samenwerking

Terug naar [[Overzicht]]

---

## Twee repositories

| Repo | URL | Inhoud |
|---|---|---|
| `rioolportaal` | https://github.com/JohanSvc/rioolportaal | Broncode (mockup, mobile, ea-scripts) |
| `rioolportaal-docs` | https://github.com/JohanSvc/rioolportaal-docs | Obsidian-documentatie (deze map) |

Beide repos zijn **private** en staan op GitHub (geen Microsoft-tenant vereist).

---

## Lokale paden

| Wat | Pad |
|---|---|
| Code | `C:\Users\johan\AppDev\apps\rioolportaal\` |
| Docs (Obsidian) | `C:\Users\johan\OneDrive\Documenten\Obsidian Vault\AppDev\Rioolportaal\` |

---

## Wat zit in git — en wat niet

### Code-repo (`rioolportaal`)

**Wel:**
- `mockup/` — server, api/, middleware/, repositories/, views/, seed.js, nav.js
- `mobile/` — Expo React Native app
- `ea/` — QEA-scripts (`.js`)
- `react-app/` — experimentele React-client
- `app.md`, `.gitignore`

**Niet (via `.gitignore`):**
- `node_modules/`
- `*.db`, `*.db-shm`, `*.db-wal` — SQLite-databases (lokaal gegenereerd via seed)
- `*.qea`, `*.qea-shm`, `*.qea-wal` — EA-model (binair, te groot voor git)
- `.env`-bestanden

> De `.qea`-database wordt **niet** gedeeld via git. Teamleden die het EA-model nodig hebben, krijgen het bestand apart (bv. via OneDrive of Teams-bestandsdeling).

### Docs-repo (`rioolportaal-docs`)

**Wel:** alle `.md`-pagina's in deze map

**Niet:**
- `.obsidian/workspace.json` — lokale Obsidian-staat
- `.obsidian/cache`

---

## Dagelijkse workflow (code)

```bash
# Starten
cd C:\Users\johan\AppDev\apps\rioolportaal
git pull

# Na een wijziging
git add .
git commit -m "feat: korte beschrijving"
git push
```

---

## Dagelijkse workflow (docs)

Docs worden automatisch bijgewerkt via de `#U`-skill in Claude Code — die doet na elke Obsidian-update een `git commit && git push` naar `rioolportaal-docs`.

Handmatig kan ook:
```bash
cd "C:\Users\johan\OneDrive\Documenten\Obsidian Vault\AppDev\Rioolportaal"
git add .
git commit -m "docs: beschrijving"
git push
```

Of via de **Obsidian Git plugin** (auto-commit elke 10 min, auto-pull elke 5 min).

---

## Obsidian Git plugin instellen

1. Obsidian → Instellingen → Community plugins → Browse → **Obsidian Git** installeren
2. Plugin-instellingen:
   - **Custom base path**: `AppDev/Rioolportaal`
   - **Auto commit interval**: `10` (minuten)
   - **Auto pull interval**: `5` (minuten)
   - **Commit message**: `docs: auto-sync {{date}}`

---

## Teamleden toevoegen

1. Ga naar de repo op GitHub → **Settings → Collaborators**
2. Klik **"Invite a collaborator"** → zoek op GitHub-gebruikersnaam of e-mailadres
3. Kies rol: **Write** voor ontwikkelaars, **Read** voor lezers

Voor de docs-repo hetzelfde proces.

---

## Git-identiteit (eenmalig per machine)

```bash
git config --global user.email "jouw@email.com"
git config --global user.name "Voornaam Achternaam"
```

---

## Zie ook

- [[Overzicht]]
- [[Presentielaag]] — architectuurbeslissingen
- [[Stack en DB-laag]] — server + middleware
