# GitHub Pages publisering

## Bakgrunn

Vi ønsker å presentere porteføljedata visuelt via GitHub Pages, men `investment-oslo` skal forbli et privat repo. Løsningen er et dedikert offentlig repo som kun inneholder det som skal publiseres.

## Arkitektur

```
investment-oslo (privat)          investment-oslo-pages (offentlig)
├── data/                          ├── data/
│   ├── portefolje.json      ──►  │   ├── portefolje.json
│   ├── portefoljeutvikling.json►  │   ├── portefoljeutvikling.json
│   └── posisjon.json        ──►  │   └── posisjon.json
├── gh-pages/                      │
│   └── index.html           ──►  ├── index.html
├── doc/                           └── README.md
├── database/
└── ...
```

## Nytt repo: `investment-oslo-pages`

- **Synlighet:** Offentlig
- **Innhold:** Kun `index.html` + `data/*.json`
- **GitHub Pages:** Aktivert fra `main` branch, rot (`/`)

### Filstruktur i det nye repoet

```
investment-oslo-pages/
├── index.html                  # Dashboard (kopi fra gh-pages/index.html)
├── data/
│   ├── portefolje.json         # Porteføljedefinisjoner
│   ├── portefoljeutvikling.json # Daglig porteføljeverdi
│   └── posisjon.json           # Aksjeposisjoner
└── README.md                   # Kort beskrivelse
```

## Endringer i index.html

Fetch-URLer må endres til relative stier siden data og HTML nå ligger i samme repo:

```javascript
// Nåværende (raw.githubusercontent.com)
fetch('https://raw.githubusercontent.com/slegga/investment-oslo/main/data/portefolje.json')

// Nytt (relativ sti, fungerer direkte på GitHub Pages)
fetch('data/portefolje.json')
```

## Oppdatering av data

Datafilene i `data/` oppdateres daglig. To alternativer for å holde det offentlige repoet oppdatert:

### Alternativ A: Manuelt script (enklest)

Et script i `investment-oslo` som kopierer filer og pusher til det offentlige repoet:

```bash
#!/bin/bash
# publish.sh - kjøres fra investment-oslo/

PAGES_REPO=~/git/investment-oslo-pages

# Kopier datafiler
cp data/portefolje.json "$PAGES_REPO/data/"
cp data/portefoljeutvikling.json "$PAGES_REPO/data/"
cp data/posisjon.json "$PAGES_REPO/data/"

# Kopier dashboard
cp gh-pages/index.html "$PAGES_REPO/index.html"

# Commit og push
cd "$PAGES_REPO"
git add -A
git commit -m "Oppdater data $(date +%Y-%m-%d)"
git push
```

### Alternativ B: GitHub Action (automatisk)

En GitHub Action i `investment-oslo` som trigges ved push til `data/` og pusher til det offentlige repoet via deploy key eller PAT.

```yaml
# .github/workflows/publish-pages.yml
name: Publiser til GitHub Pages

on:
  push:
    paths:
      - 'data/**'
      - 'gh-pages/**'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Kopier filer til pages-repo
        run: |
          mkdir -p publish/data
          cp data/portefolje.json publish/data/
          cp data/portefoljeutvikling.json publish/data/
          cp data/posisjon.json publish/data/
          cp gh-pages/index.html publish/index.html

      - name: Push til offentlig repo
        uses: cpina/github-action-push-to-another-repository@v1.7
        env:
          SSH_DEPLOY_KEY: ${{ secrets.PAGES_DEPLOY_KEY }}
        with:
          source-directory: publish
          destination-github-username: slegga
          destination-repository-name: investment-oslo-pages
          target-branch: main
          commit-message: "Oppdater data ${{ github.event.head_commit.timestamp }}"
```

**For GitHub Action trengs:**
1. Generer SSH deploy key: `ssh-keygen -t ed25519 -f pages-deploy-key`
2. Legg public key som deploy key i `investment-oslo-pages` (med write-tilgang)
3. Legg private key som secret `PAGES_DEPLOY_KEY` i `investment-oslo`

## Aktivere GitHub Pages

I `investment-oslo-pages`:
1. Settings → Pages
2. Source: Deploy from a branch
3. Branch: `main`, folder: `/` (rot)
4. Save

Siden blir tilgjengelig på: `https://slegga.github.io/investment-oslo-pages/`

## Steg for å sette opp

1. Opprett repo `investment-oslo-pages` på GitHub (offentlig)
2. Klon repoet lokalt
3. Kopier `gh-pages/index.html` → `index.html` (oppdater fetch-URLer til relative stier)
4. Kopier `data/*.json` → `data/`
5. Push
6. Aktiver GitHub Pages i repo-settings
7. Velg oppdateringsmetode (script eller GitHub Action)
