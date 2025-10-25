# е-Играч - Sistem Zaštite od Kockanja

**Građanska inicijativa** za odgovorno kockanje i zaštitu građana Republike Srbije.

> **⚠️ Važna napomena**: е-Играč je **predlog grupe građana sa iskustvom u oblasti igara na sreću**, NE zvanična inicijativa Ministarstva finansija.

Ovaj repository sadrži tehničku dokumentaciju, analize i predlog implementacije obavezne mobilne aplikacije za odgovorno kockanje u Srbiji.

## Dokumentacija

### GitHub Pages Sajt (Latinica)
Kompletan dokumentacioni sajt je dostupan na GitHub Pages.

### HTML Dokumentacija (Ćirilica)
Svi dokumenti su dostupni i u HTML formatu na **srpskoj ćirilici** u folderu [`html/`](html/):
- [index.html](html/index.html) - Početna stranica
- [eIgrac-Aplikacija.html](html/eIgrac-Aplikacija.html) - Mobilna aplikacija
- [eIgrac-Improvements.html](html/eIgrac-Improvements.html) - Napredna unapređenja i analiza
- [Obavezna-Mobilna-Aplikacija.html](html/Obavezna-Mobilna-Aplikacija.html) - Detaljna analiza
- [srbija-analiza-stanja-2025.html](html/srbija-analiza-stanja-2025.html) - Analiza stanja u Srbiji
- [serbia-gambling.html](html/serbia-gambling.html) - GDPR-compliant registry
- [eu-startegies.html](html/eu-startegies.html) - EU strategije

Originalni fajlovi na latinici dostupni su u [`html/latin/`](html/latin/).

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
├── docs/                          # Jekyll sajt (GitHub Pages source - latinica)
│   ├── _config.yml               # Jekyll konfiguracija
│   ├── _layouts/
│   │   └── default.html          # Custom layout sa navigacijom
│   ├── index.md                  # Početna stranica
│   ├── eIgrac-Aplikacija.md      # Mobilna aplikacija
│   ├── eIgrac-Improvements.md    # Napredna unapređenja i analiza
│   ├── Obavezna-Mobilna-Aplikacija.md  # Detaljna analiza
│   ├── srbija-analiza-stanja-2025.md   # Analiza stanja u Srbiji
│   ├── serbia-gambling.md        # GDPR-compliant registry spec
│   ├── eu-startegies.md          # EU strategije
│   └── Gemfile                   # Ruby dependencies
├── html/                          # HTML dokumentacija (ćirilica)
│   ├── latin/                    # Originalni fajlovi (latinica)
│   ├── index.html
│   ├── eIgrac-Aplikacija.html
│   ├── eIgrac-Improvements.html
│   ├── Obavezna-Mobilna-Aplikacija.html
│   ├── srbija-analiza-stanja-2025.html
│   ├── serbia-gambling.html
│   └── eu-startegies.html
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

- **[Mobilna Aplikacija е-Играč](docs/eIgrac-Aplikacija.md)** - Osnovni koncept mobilne aplikacije
- **[Napredna Unapređenja](docs/eIgrac-Improvements.md)** - Detaljna analiza sa izvorima i metodologijom
- **[Obavezna Mobilna Aplikacija](docs/Obavezna-Mobilna-Aplikacija.md)** - Kako app rešava enforcement probleme
- **[Analiza Stanja u Srbiji](docs/srbija-analiza-stanja-2025.md)** - Trenutno stanje i statistike
- **[EU Strategije](docs/eu-startegies.md)** - Najbolje prakse iz EU
- **[GDPR-Compliant Registry](docs/serbia-gambling.md)** - Specifikacija registra

## O Podacima i Izvorima

### Napomena o Statistikama

Dokumentacija koristi podatke iz različitih izvora:

**Verifikovani podaci:**
- Broj inspektora (6) - javno potvrđeno od strane direktora Uprave za igre na sreću
- Broj legalnih automata (33.000+) i kladionica (2.900+) - zvanični podaci
- Zakonski okvir - Zakon o igrama na sreću i izmene iz 2025. godine

**Industrijske procene:**
- **~74% sivog tržišta** - procena Udruženja priređivača igara na sreću (JAKTA)
  - JAKTA navodi da je sivo tržište između 50-70% legitimnog tržišta
  - Tačan zvaničan izvor za cifru 74% nije javno dostupan
- **60.000 nelegalnih automata** - industrijska procena
- **300.000+ građana sa problemom zavisnosti** - procena Kluba ŠANSA

**Projektovane brojke:**
- ROI projekcije, troškovi razvoja, operativni budžet - bazirano na analogijama sa drugim tržištima
- Tehnička arhitektura i kapaciteti - konceptualne procene

Detaljnije informacije o izvorima dostupne su u [dodatku dokumenta eIgrac-Improvements](docs/eIgrac-Improvements.md#dodatak-izvori-metodologija-i-napomene).

> **⚠️ Važno**: Ovaj projekat je **konceptualna analiza i predlog sistema**, ne finalna studija izvodljivosti. Preporučuje se nezavisna verifikacija svih brojki i pilot program pre potpune implementacije.

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
