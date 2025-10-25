# Setup GitHub Pages - Brzi Vodič

## Pre Push-a na GitHub

**VAŽNO**: Pre nego što pushneš kod na GitHub, **MORAŠ** da promeniš sledeće vrednosti u [`_config.yml`](_config.yml):

### Otvori _config.yml i izmeni:

```yaml
# PROMENI OVE VREDNOSTI:
baseurl: "/gambling"                          # → "/ime-tvog-repo"
url: "https://yourusername.github.io"         # → "https://tvoje-username.github.io"
```

**Primer**: Ako je tvoj GitHub username `bojan123` i repository se zove `kockaid`:

```yaml
baseurl: "/kockaid"
url: "https://bojan123.github.io"
```

## Korak-po-Korak Deployment

### 1. Proveri Promene

```bash
# Proveri šta si promenio
git status
```

### 2. Ažuriraj _config.yml

```bash
# Otvori _config.yml i promeni baseurl i url
notepad docs/_config.yml
```

### 3. Commituj i Pushuj

```bash
git add .
git commit -m "Setup Jekyll GitHub Pages site"
git push origin main
```

### 4. Omogući GitHub Pages

1. Idi na GitHub repository
2. Klikni **Settings** tab
3. U levom meniju klikni **Pages**
4. U **Source** dropdown izaberi **GitHub Actions**
5. Workflow će se automatski pokrenuti

### 5. Čekaj Build

- Idi na **Actions** tab u svom repository
- Vidi da se "Deploy Jekyll site to Pages" workflow izvršava
- Traje obično 2-3 minuta

### 6. Pristup Sajtu

Kada se build završi, tvoj sajt će biti dostupan na:

```
https://tvoje-username.github.io/ime-repository-ja/
```

## Testiranje Lokalno (Opcionalno)

Ako želiš da vidiš sajt pre deploya:

```bash
cd docs

# Instalirati Ruby gems (samo prvi put)
bundle install

# Pokrenuti lokalno
bundle exec jekyll serve

# Otvori browser na:
# http://localhost:4000/gambling/
```

## Troubleshooting

### Build Failed?

1. Proveri da li si promenio `baseurl` i `url` u `_config.yml`
2. Proveri da li ima grešaka u markdown sintaksi
3. Pogledaj detalje greške u Actions tab → klikni na failed workflow

### Sajt ne učitava stilove?

Proveri da li je `baseurl` u `_config.yml` tačan i odgovara imenu repository-ja.

### 404 greška?

- Proveri da li je GitHub Pages omogućen u Settings → Pages
- Proveri da li je Source postavljen na **GitHub Actions**
- Čekaj 2-3 minuta nakon pusha

## Dodatni Resursi

- [GitHub Pages Dokumentacija](https://docs.github.com/en/pages)
- [Jekyll Dokumentacija](https://jekyllrb.com/docs/)
- [Jekyll Themes](https://jekyllrb.com/docs/themes/)

---

**Napomena**: Nakon što se sajt deployuje, svaki push na `main` branch će automatski triggervati rebuild i deployment.
