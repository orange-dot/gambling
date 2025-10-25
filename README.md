# KockaID - Sistem Zaštite od Kockanja

Nacionalni sistem za odgovorno kockanje i zaštitu građana Republike Srbije.

## Dokumentacija

Kompletan dokumentacioni sajt je dostupan na GitHub Pages.

## Pokretanje Dokumentacionog Sajta Lokalno

```bash
cd docs
bundle install
bundle exec jekyll serve
```

Zatim otvori browser na `http://localhost:4000/gambling/`

## Konfiguracija GitHub Pages

### Korak 1: Ažuriraj _config.yml

Pre nego što pushneš kod, otvori [`docs/_config.yml`](docs/_config.yml) i ažuriraj sledeće vrednosti:

```yaml
baseurl: "/gambling"  # Promeni u /ime-tvog-repository-ja
url: "https://yourusername.github.io"  # Promeni u https://tvoje-github-username.github.io
```

### Korak 2: Omogući GitHub Pages u Repository Settings

1. Idi na GitHub repository Settings
2. U levom meniju klikni na **Pages**
3. U **Source** sekciji izaberi:
   - **Source**: GitHub Actions
4. Workflow će se automatski pokrenuti kada pushneš kod

### Korak 3: Push na GitHub

```bash
git add .
git commit -m "Setup Jekyll GitHub Pages site"
git push origin main
```

### Korak 4: Pristup Sajtu

Nakon što se GitHub Actions workflow završi (obično 2-3 minuta), tvoj sajt će biti dostupan na:

```
https://tvoje-github-username.github.io/ime-repository-ja/
```

## Struktura Projekta

```
gambling/
├── .github/
│   └── workflows/
│       └── jekyll-gh-pages.yml    # GitHub Actions workflow za deployment
├── docs/                          # Jekyll sajt (GitHub Pages source)
│   ├── _config.yml               # Jekyll konfiguracija
│   ├── _layouts/
│   │   └── default.html          # Custom layout sa navigacijom
│   ├── index.md                  # Početna stranica
│   ├── Koncept-Sistema.md        # Koncept centralnog sistema
│   ├── Obavezna-Mobilna-Aplikacija.md  # Analiza obavezne mobilne app
│   ├── srbija-analiza-stanja-2025.md   # Analiza stanja u Srbiji
│   ├── serbia-gambling.md        # GDPR-compliant registry spec
│   ├── eu-startegies.md          # EU strategije
│   └── Gemfile                   # Ruby dependencies
└── README.md                     # Ovaj fajl
```

## Tehnički Stack

- **Backend**: C# / .NET 8
- **Database**: MongoDB
- **Frontend**: React
- **Real-time**: SignalR
- **Mobile**: Native iOS (Swift) / Android (Kotlin)
- **Documentation**: Jekyll (GitHub Pages)

## Dokumenti

### Ključni Dokumenti

- **[Koncept Sistema](docs/Koncept-Sistema.md)** - Detaljan koncept centralnog državnog sistema
- **[Obavezna Mobilna Aplikacija](docs/Obavezna-Mobilna-Aplikacija.md)** - Analiza kako app rešava enforcement probleme
- **[Analiza Stanja u Srbiji](docs/srbija-analiza-stanja-2025.md)** - Trenutno stanje i statistike
- **[EU Strategije](docs/eu-startegies.md)** - Najbolje prakse iz EU

## Getting Started (Development)

```bash
# Restore packages
dotnet restore

# Configure environment
cp appsettings.example.json appsettings.json

# Run development server
dotnet run
```

## Licenca

Ovaj projekat je inicijativa fokusirana na smanjenje štete, zaštitu igrača i prakse odgovornog kockanja.
