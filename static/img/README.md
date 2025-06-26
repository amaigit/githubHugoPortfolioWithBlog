# Immagini del Portfolio

Questa cartella contiene tutte le immagini utilizzate nel portfolio. I file attuali sono placeholder che devono essere sostituiti con immagini reali.

## Struttura delle Cartelle

```
static/img/
├── author/
│   └── author.jpg          # Immagine profilo autore (300x300px)
├── skills/
│   ├── go.svg             # Logo Go (già creato)
│   ├── react.svg          # Logo React (già creato)
│   └── docker.svg         # Logo Docker (già creato)
├── experiences/
│   ├── techcorp-logo.png  # Logo TechCorp (200x100px)
│   └── innovate-logo.png  # Logo InnovateStartup (200x100px)
└── projects/
    ├── cli-tool-preview.png    # Anteprima progetto CLI (400x300px)
    └── ecommerce-preview.png   # Anteprima progetto e-commerce (400x300px)
```

## Come Sostituire le Immagini

### 1. Immagine Autore
- **File**: `author/author.jpg`
- **Dimensioni**: 300x300px (quadrata)
- **Formato**: JPG o PNG
- **Contenuto**: Foto professionale dell'autore

### 2. Loghi Aziendali
- **Cartella**: `experiences/`
- **Dimensioni**: 200x100px
- **Formato**: PNG con sfondo trasparente
- **Contenuto**: Loghi ufficiali delle aziende

### 3. Anteprime Progetti
- **Cartella**: `projects/`
- **Dimensioni**: 400x300px
- **Formato**: PNG o JPG
- **Contenuto**: Screenshot o mockup dei progetti

### 4. Loghi Tecnologie
- **Cartella**: `skills/`
- **Formato**: SVG (già creati)
- **Contenuto**: Loghi ufficiali delle tecnologie

## Note Importanti

- Mantieni le stesse dimensioni consigliate per una visualizzazione ottimale
- Usa formati ottimizzati per il web (JPG per foto, PNG per loghi, SVG per icone)
- Assicurati di avere i diritti per utilizzare le immagini
- Testa la visualizzazione dopo aver sostituito le immagini

## Comando per Sostituire

```bash
# Esempio per sostituire l'immagine autore
cp /path/to/your/photo.jpg static/img/author/author.jpg

# Esempio per sostituire un logo aziendale
cp /path/to/company-logo.png static/img/experiences/techcorp-logo.png
``` 