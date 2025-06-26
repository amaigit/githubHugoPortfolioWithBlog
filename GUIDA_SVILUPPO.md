# Portfolio e Blog con Hugo: La Guida all'Approccio 100% Browser con GitHub Pages ed Actions

Questa guida fornisce un percorso specialistico e pratico per creare un sito portfolio professionale e un blog tecnico utilizzando il generatore di siti statici Hugo e il tema Toha. L'intero processo, dalla configurazione al deploy e alla gestione continua, viene eseguito esclusivamente tramite l'interfaccia web di GitHub, senza la necessità di installare software o utilizzare la riga di comando in locale.

L'approccio "browser-based" offre vantaggi significativi:
- **Costi Zero**: L'hosting su GitHub Pages è gratuito per i repository pubblici.
- **Efficienza**: L'automazione tramite GitHub Actions elimina i passaggi manuali di build e deploy.
- **Accessibilità**: Puoi aggiornare il tuo sito da qualsiasi dispositivo con un browser, senza dipendere dal tuo ambiente di sviluppo locale.

A differenza di altri temi Hugo, il tema Toha utilizza una struttura di configurazione centralizzata in un unico file principale, `config/_default/config.yaml`, che analizzeremo in dettaglio.

## Fase 1: Creazione del Repository dal Template Toha

Il modo più rapido per iniziare è utilizzare il repository template ufficiale di Toha, che include già la struttura di base e il workflow di GitHub Actions.

1. **Vai al Template Starter**: Naviga su https://github.com/hugo-toha/starter.
2. **Usa il Template**: Clicca sul pulsante verde "Use this template" e seleziona "Create a new repository".
3. **Nome del Repository**: Per un sito utente principale su GitHub Pages, è fondamentale nominare il repository in questo modo: `TUO_NOME_UTENTE.github.io`. Sostituisci `TUO_NOME_UTENTE` con il tuo effettivo nome utente di GitHub.
4. **Crea il Repository**: Assicurati che il repository sia Public e clicca su "Create repository".

## Fase 2: Abilitazione di GitHub Pages

Ora devi dire a GitHub dove trovare i file del sito una volta che sono stati compilati.

1. Nel tuo nuovo repository, vai alla tab "Settings".
2. Nel menu a sinistra, seleziona "Pages".
3. Sotto "Build and deployment", alla voce "Source", seleziona "GitHub Actions". Questa è l'opzione moderna e raccomandata, che si integra perfettamente con il workflow.

## Fase 3: Analisi del Workflow di GitHub Actions (.github/workflows/deploy.yml)

Questa è l'automazione al lavoro. Il file, già presente nel template, gestisce la compilazione e la pubblicazione del tuo sito.

### Codice del Workflow con Spiegazioni:

```yaml
# Nome del workflow, visibile nella tab "Actions"
name: Deploy to GitHub Pages

on:
  # Si attiva ad ogni push (commit) sul branch 'main'
  push:
    branches:
      - main
  # Permette di avviare il workflow manualmente
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write # Autorizzazione per scrivere sul branch gh-pages
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      # Step 1: Scarica il codice del tuo repository e i sottomoduli (il tema Toha)
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true  # FONDAMENTALE per scaricare il tema
          fetch-depth: 0    # Scarica l'intera history di Git

      # Step 2: Configura l'ambiente Hugo
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true # Installa la versione Extended, necessaria per Toha

      # Step 3: Compila il sito ottimizzando i file
      - name: Build
        run: hugo --minify

      # Step 4: Pubblica i file compilati su GitHub Pages
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: gh-pages # Branch di destinazione per il deploy
```

## Fase 4: Configurazione Avanzata di Toha (config/_default/config.yaml)

Questo è il cuore della personalizzazione. Modifica il file `config/_default/config.yaml` direttamente nel browser.

### Informazioni di Base e Autore:

```yaml
baseURL: "https://TUO_NOME_UTENTE.github.io/"
languageCode: "it-it"
title: "Portfolio di Mario Rossi"
theme: "toha"

params:
  author:
    name: "Mario Rossi"
    headline: "Sviluppatore Software Full-Stack"
    image: "img/author.jpg" # Carica un'immagine in static/img/

  social:
    - name: github
      icon: "fab fa-github"
      link: "https://github.com/TUO_NOME_UTENTE"
    - name: linkedin
      icon: "fab fa-linkedin"
      link: "https://linkedin.com/in/TUO_PROFILO"
```

### Gestione Sezioni della Homepage:

Toha permette di abilitare diverse sezioni. Attiviamo quelle più utili per un portfolio.

```yaml
params:
  # ... altre configurazioni ...
  # Definisce le sezioni, il loro ordine (weight) e la visibilità
  sections:
    - id: "about"
      name: Chi Sono
      template: "sections/about.html"
      enable: true
      weight: 1
    - id: "skills"
      name: Competenze
      template: "sections/skills.html"
      enable: true # <-- Abilitiamo la sezione Skills
      weight: 2
    - id: "experiences"
      name: Esperienze
      template: "sections/experiences.html"
      enable: true # <-- Abilitiamo la sezione Esperienze
      weight: 3
    - id: "projects"
      name: Progetti
      template: "sections/projects.html"
      enable: true
      weight: 4
    - id: "posts"
      name: Blog
      template: "sections/posts.html"
      enable: true
      weight: 5
```

## Fase 5: Aggiungere Contenuti Strutturati

Creiamo i contenuti per le sezioni che abbiamo appena abilitato.

### A. Struttura per le Competenze (Skills)

Per ogni competenza, crea un file in `content/skills/`.

**Percorso Esempio**: `content/skills/go/index.md`

```yaml
---
name: "Go (Golang)"
logo: "/img/skills/go.svg" # Carica l'SVG o PNG in static/img/skills/
# Link opzionale alla documentazione ufficiale
url: "https://golang.org/"
---
```

### B. Struttura per le Esperienze Lavorative

Per ogni esperienza, crea una cartella in `content/experiences/`.

**Percorso Esempio**: `content/experiences/azienda-tech/index.md`

```yaml
---
company: "Azienda Tech S.p.A."
logo: "/img/experiences/azienda-tech-logo.png" # Carica il logo in static/img/experiences
position: "Senior Software Engineer"
period: "Gennaio 2022 - Presente"
---

- Progettazione e sviluppo di microservizi in ambiente cloud (GCP, AWS).
- Ottimizzazione di query su database PostgreSQL e implementazione di strategie di caching con Redis.
- Mentoring di sviluppatori junior e conduzione di code review.
```

### C. Struttura per i Progetti (con pulsanti)

Per ogni progetto, crea una cartella in `content/projects/`.

**Percorso Esempio**: `content/projects/mio-tool-cli/index.md`

```yaml
---
title: "Mio Tool a Riga di Comando"
name: "Mio Tool CLI"
image: "img/projects/tool-cli-preview.png"
weight: 1
tech: ["Go", "Cobra", "Viper"]

# Pulsanti personalizzati per ogni progetto
buttons:
  - name: GitHub
    icon: "fab fa-github"
    url: "https://github.com/TUO_NOME_UTENTE/mio-tool-cli"
  - name: Live Demo
    icon: "fas fa-desktop"
    url: "https://example.com/demo-tool"
---

Descrizione dettagliata del mio tool a riga di comando. Supporta la formattazione Markdown per evidenziare le **funzionalità principali**.
```

### D. Struttura per il Blog

Per ogni articolo, crea un file in `content/posts/`.

**Percorso Esempio**: `content/posts/primo-dev-log.md`

```yaml
---
title: "Il Mio Primo Dev Log: Avvio del Progetto X"
date: 2025-06-26T10:00:00+02:00
author: "Mario Rossi"
tags: ["sviluppo", "go", "progetto-x"]
---

Benvenuti nel primo articolo di questo diario di sviluppo! Oggi ho iniziato a lavorare sul **Progetto X**.
```

## Fase 6: Deploy e Validazione

Ogni commit sul branch main attiverà il workflow.

1. **Monitora il Workflow**: Vai alla tab "Actions".
2. **Attendi il Successo**: Attendi l'icona con la spunta verde (richiede 2-3 minuti).
3. **Valida il Sito**: Visita `https://TUO_NOME_UTENTE.github.io/`. L'URL esatto si trova in "Settings" -> "Pages".

## Fase 7: Gestione Continua del Sito

La manutenzione è semplice e interamente browser-based:

- **Modificare Contenuti**: Naviga al file Markdown desiderato, modificalo e committa.
- **Modificare Configurazione**: Modifica `config/_default/config.yaml` per cambiare sezioni, titoli o dati personali.
- **Aggiungere Risorse**: Usa "Add file" -> "Upload files" nella cartella `static/img/` (o sottocartelle) per caricare immagini, loghi e SVG.

Ogni commit aggiornerà automaticamente il tuo sito live. 