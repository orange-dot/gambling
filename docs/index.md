---
layout: default
title: KockaID - Početna
---

# KockaID - Sistem Zaštite od Kockanja

Nacionalni sistem za odgovorno kockanje i zaštitu građana Republike Srbije

## O Projektu

KockaID je sveobuhvatna platforma dizajnirana za promociju odgovornog kockanja i zaštitu ranjivih korisnika kroz monitoring, intervenciju i self-management alate. Sistem je osmišljen specifično za potrebe Republike Srbije i rešava jedinstvene izazove sa kojima se suočava naša zemlja.

## Trenutno Stanje u Srbiji

- **74%** tržišta u sivoj zoni
- Samo **6 inspektora** za kontrolu 95,000+ lokacija
- **300,000** građana sa problemom zavisnosti
- **42%** mladih kocka
- **-79.5M EUR** gubitaka godišnje

## Rešenje: Obavezna Mobilna Aplikacija

KockaID aplikacija koristi device binding, biometrijsku autentifikaciju i real-time nadzor da automatski enforcuje postojeće politike i eliminiše crno tržište.

---

## Dokumentacija

### Ključni Dokumenti

#### [Koncept Sistema](Koncept-Sistema)
Detaljan koncept centralnog državnog sistema za monitoring i zaštitu. Pokriva:
- Centralna državna platforma sa real-time prikupljanjem podataka
- Jedinstveni registar igrača
- Modul odgovornog kockanja (inspirisan UK GAMSTOP, švedskim Spelpaus)
- Analytics layer sa ML-based detekcijom rizika
- Šifrovana razmena podataka (GDPR compliant)

#### [Obavezna Mobilna Aplikacija - Analiza i Implementacija](Obavezna-Mobilna-Aplikacija)
Sveobuhvatna analiza kako obavezna mobilna aplikacija rešava probleme enforcementa postojećih politika u EU i Srbiji. Uključuje:
- Top 10 problema enforcementa u EU sistemima
- Kako aplikacija rešava svaki problem (višestruki nalozi, crno tržište, self-exclusion breaches)
- Tehnička arhitektura (C#/.NET, MongoDB, native iOS/Android)
- **Specifični kontekst Srbije** - kako KockaID rešava jedinstvene izazove
- Implementaciona strategija u 5 faza
- Cost-benefit analiza (260x ROI)

#### [Analiza Stanja u Srbiji 2025](srbija-analiza-stanja-2025)
Detaljna analiza trenutnog stanja regulacije kockanja u Srbiji:
- Legislativa i institucije
- Tržišni podaci i statistike
- Analiza crnog tržišta (60,000 nelegalnih aparata)
- Zdravstveni uticaj i zavisnost
- Fiskalni gubici i korupcija

### Referentni Dokumenti

#### [Analiza Srbije - Gambling Regulation](serbia-gambling)
Pregled regulatornog okvira kockanja u Srbiji (engleski)

#### [EU Strategije za Odgovorno Kockanje](eu-startegies)
Analiza najboljih praksi iz EU zemalja: UK GAMSTOP, švedski Spelpaus, italijanski GAD, holandski CRUKS

---

## Ključne Karakteristike Sistema

### Self-Exclusion
Korisnici mogu dobrovoljno ograničiti pristup kockanju na određeni period

### Limiti Potrošnje
Konfigurisanje dnevnih, nedeljnih i mesečnih limita depozita/gubitaka

### Monitoring Aktivnosti
Real-time praćenje obrazaca kockanja i analiza ponašanja

### Detekcija Rizika
Automatski alarmi za indikatore problematičnog kockanja

### Cool-off Periodi
Obavezne pauze aktivirane na osnovu specifičnih ponašanja

### Resursi za Podršku
Direktan pristup helpline-ovima za zavisnost i savetovanju

---

## Tehnički Stack

- **Backend**: C# / .NET 8
- **Database**: MongoDB
- **Frontend**: React
- **Real-time**: SignalR
- **Mobile**: Native iOS (Swift) / Android (Kotlin)
- **ML/AI**: XGBoost, TensorFlow
- **Security**: Certificate Pinning, Biometric Auth, Device Binding

---

## Implementacija

### Faza 0: Priprema (3 meseca)
- Pravni okvir i legislativa
- Ugovaranje developmenta
- Brendiranje i marketing kampanja

### Faza 1: Pilot (6 meseci)
- 5,000 korisnika
- 5 online operatora
- Beta testiranje

### Faza 2: Obavezno za Online (9 meseci)
- 200-300K igrača
- 21 licencirana online operatora
- Potpuna nacionalna pokrivenost

### Faza 3: Fizički Objekti (12 meseci)
- 2,900 kladionica
- 33,000+ aparata
- QR check-in sistem

### Faza 4: Eliminacija Crnog Tržišta (18 meseci)
- Nacionalne racije
- 1,000 inspekcija mesečno
- DNS-level blokiranje nelegalnih sajtova

### Faza 5: Regionalna Ekspanzija (24+ meseci)
- Bilateralni sporazumi sa Hrvatskom, Slovenijom, BiH
- Cross-border digitalni identitet

---

## Očekivani Rezultati

| Metrika | Pre | Posle |
|---------|-----|-------|
| Legalno tržište | 26% | 90% |
| Maloletni korisnici | 42% | <5% |
| Pokrivenost zavisnika | 0.02% | 78% |
| Fiskalni bilans | -79.5M EUR | +100M EUR |
| Inspektori potrebni | 600+ | 100 |

---

## Kontakt i Resursi

- **Uprava za Igre na Sreću**: [www.uzrs.gov.rs](https://www.uzrs.gov.rs)
- **Nacionalna helpline za zavisnost**: 011-xxxx-xxx
- **GitHub**: [Repository](https://github.com/yourusername/gambling)

---

## Licenca

Ovaj projekat je inicijativa fokusirana na smanjenje štete, zaštitu igrača i prakse odgovornog kockanja.

**Napomena**: Dokumentacija u ovom repozitorijumu je kreirana za potrebe analize i dizajna sistema zaštite od kockanja za Republiku Srbiju.
