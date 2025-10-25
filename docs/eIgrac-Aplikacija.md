# е-Играч: Mobilna Aplikacija za Odgovornu Igru

## Pregled Građanske Inicijative

> **Napomena**: е-Играč је predlog **grupe građana sa iskustvom u oblasti igara na sreću**, NE zvanična inicijativa Ministarstva finansija.

**е-Играč** je predložena mobilna aplikacija koju bi razvilo Ministarstvo finansija Republike Srbije, a koja omogućava:

1. **Vezu sa Centralnim Registrom Igrača** - jedinstvena evidencija svih igrača i transakcija
2. **Obaveznu autentifikaciju** - samo punoletni građani i nerezidenti mogu aktivirati aplikaciju
3. **Implementaciju alata odgovorne igre** - mesečni limiti, samoisključenje, zabrana marketinga
4. **Transparentan pregled potrošnje** - uvid u sve transakcije kod svih priređivača
5. **Sprečavanje kockanja maloletnika** - tehnička kontrola starosti

---

## Problem Zavisnosti od Igara na Sreću

Prema postojećim istraživanjima, Zakon o igrama na sreću u tekućem obliku, nažalost, ne sprečava učestvovanje maloletnika u igrama (iako to zabranjuje), ni pojavu zavisnosti od igara na sreću kod velikog broja građana Srbije, uključujući i maloletnike.

Predlog grupe građana sa iskustvom u ovoj oblasti je da se uvede kombinovano pravno-tehničko rešenje koje bi, daljim otežavanjem igre kod stranih priređivača, izbegavanjem igre iz bankarske pozajmice, uvođenjem zakonom propisanih alata odgovorne igre, Centralnog Registra Igrača i opšteg i ličnog maksimuma mesečnog ograničenja, smanjilo negativne efekte igara na sreću - i istovremeno povećalo sprovodivost buduće regulative.

Jedan od elemenata rešenja je i obavezna mobilna aplikacija za igrače, koju opisuje ovaj dokument.

---

## Osnovne Funkcije е-Играč Aplikacije

### 1. Aktivacija Aplikacije

#### Za građane Srbije:

**Instalacija i aktivacija:**
- Igrač instalira aplikaciju na svoj mobilni telefon (Android/iOS)
- Aplikacija za aktivaciju traži email adresu registrovanu u eID.gov.rs
- Aplikacija zahteva **ConsentID prijavu visoke pouzdanosti**
  - Isto kao prijava na eUprava portal (eid.gov.rs)
  - Autentifikacija preko državnog identiteta
  - **Maloletna lica ne mogu aktivirati aplikaciju**

**Tehnički detalji:**
- Nakon prijave, aplikacija dobija "refresh token" koji omogućava dugotrajnu sesiju bez ponovne prijave
- Koristi se OAuth2 protokol
- Procedura kao kod većine mobilnih aplikacija koje koriste OAuth2

#### Za nerezidente:

**Registracija preko kasino blagajnika:**
- Nerezidenti registruju aplikaciju skeniranjem QR koda koji dobiju od blagajnika u kazinu
- Blagajnik ih registruje na osnovu podataka iz pasoša
- Jednom registrovana aplikacija važi za sve priređivače

**Proces aktivacije:**

```
┌──────────────────────────────────────────┐
│  Aktivacija е-Играч (Građani Srbije)     │
├──────────────────────────────────────────┤
│                                          │
│  1. Igrač otvara е-Играч aplikaciju      │
│  2. Unosi email registrovan u eID.gov.rs │
│  3. Preusmerava se na ConsentID prijavu  │
│  4. Autentifikuje se (kao na eUprava)    │
│  5. Aplikacija dobija token i aktivira se│
│                                          │
│  Provera starosti: Automatska kroz       │
│  ConsentID - maloletnici ne mogu         │
└──────────────────────────────────────────┘
```

---

### 2. Mesečno Ograničenje Potrošnje (OBAVEZNO)

**Igrač mora podesiti mesečno ograničenje potrošnje pre početka igre.**

#### Pravila podešavanja limita:

**Smanjenje ograničenja:**
- Primenjuje se **odmah**

**Povećanje ograničenja:**
- Primenjuje se nakon **"perioda hlađenja glave"** (predlog: 72 sata)
- Sprečava igrače da prave ishitrene odluke u trenutku kada su "obuzeti igrom"

#### Opšti i lični maksimum:

**Opšta maksimalna vrednost ograničenja:**
- Minimalna zarada na nivou Republike × zakonski utvrđeni koeficijent

**Lična maksimalna vrednost ograničenja:**
- Prijavljeni prihod u prethodnoj godini × zakonski utvrđeni koeficijent
- Primenjuje se ako je veća od opšte maksimalne vrednosti

**Za limite veće od ovih vrednosti:**
Igrač mora da:
- Kompletira zakonom propisanu proceduru
- Priloži dokaze o imovinskom stanju
- Uplati odgovarajuću administrativnu taksu

---

### 3. Uplata Tiketa u Radnjama (QR Kod Sistem)

**Proces uplate:**

1. **Priprema:** Igrač pripremi tiket kao i obično i preda ga blagajniku sa novcem
2. **Unos:** Blagajnik primi tiket i novac i unese detalje u sistem
3. **Zahtev za QR kod:** Kada blagajnik potvrdi unos, sistem traži da skenira QR kod sa igrčevog mobilnog telefona
4. **Generisanje QR koda:** Igrač otvara е-Играч aplikaciju i klikne "Prikaži QR kod"
5. **Skeniranje:** Blagajnik skenira QR kod
6. **Provera:** Sistem za igre na sreću traži odobrenje od Centralnog Registra Igrača:
   - Provera da li je igrač samoisključen
   - Provera da li je dostignut mesečni limit
   - Provera da li je transakcija unutar pravila
7. **Rezultat:**
   - U slučaju odobrenja: Transakcija se potvrđuje
   - U slučaju odbijanja: Blagajnik vraća tiket i novac igraču

#### Bezbednosne karakteristike:

- **QR kod ističe nakon 60 sekundi**
  - Nemoguće je "pozajmiti ga od prijatelja"
- **Svaka transakcija dobija jedinstveni broj** u Centralnom Registru Igrača
  - Priređivač je dužan da prikaže ovaj broj na priznanici

**Ova funkcija je nedostupna igračima u periodu samoisključenja.**

```
┌───────────────────────────────────────────────┐
│  Tok Uplate Tiketa                            │
├───────────────────────────────────────────────┤
│                                               │
│  Igrač              Blagajnik       Centralni │
│    │                   │            Registar  │
│    │                   │                │     │
│    │─── Pripremi tiket ──>            │     │
│    │                   │                │     │
│    │                   │─── Unesi u sistem    │
│    │                   │                │     │
│    │<── Traži QR kod ──┤                │     │
│    │                   │                │     │
│    │─── Prikaže QR ────>                │     │
│    │                   │                │     │
│    │                   │─── Skeniraj QR ────> │
│    │                   │                │     │
│    │                   │<── Proveri limit ────┤
│    │                   │<── Proveri isključen─┤
│    │                   │<── ODOBRENO/ODBIJENO │
│    │                   │                │     │
│    │<── Potvrda/Odbijanje ──┤          │     │
│                                               │
└───────────────────────────────────────────────┘
```

---

### 4. Registracija Online Računa (Token za Jednokratnu Upotrebu)

Igrač povezuje račun za online igre sa Centralnim Registrom Igrača **samo jednom**:
- U trenutku registracije novog računa
- Ili reaktivacijom postojećih računa (nakon stupanja na snagu nove regulative)

#### Proces registracije:

1. **Generisanje tokena:** Igrač otvara е-Играč aplikaciju i bira "Registracija online računa"
2. **Dobijanje koda:** Aplikacija generiše token za jednokratnu upotrebu (npr. `7f9-248-680-423`)
3. **Unos kod priređivača:** Igrač unosi ovaj token na sajtu priređivača pri registraciji
4. **Verifikacija:** Priređivač šalje token ka Centralnom Registru Igrača za verifikaciju
5. **Povratak podataka:** Centralni Registar vraća:
   - **Jedinstveni identifikator igrača** (ovo NIJE JMBG!)
   - Verifikovane lične podatke: ime, prezime, adresu prebivališta

#### Zaštita privatnosti:

**Šta priređivač dobija:**
- Ime, prezime
- Adresa prebivališta
- Jedinstveni identifikator igrača za transakcije (pseudonimizovano)

**Šta priređivač NE SME držati:**
Regulativa može propisati da priređivač mora uništiti:
- JMBG
- Broj lične karte
- Broj pasoša
- Druge "jake identifikatore" opšteg tipa

**Benefiti:**
- Igrač ne deli JMBG sa mnogim privatnim kompanijama
- Smanjenje rizika od curenja osetljivih podataka
- Priređivač dobija samo minimum potrebnih informacija
- Automatska verifikacija identiteta bez potrebe za dodatnom proveru

---

### 5. Pregled Potrošnje

Igrač može u bilo kom trenutku videti:

**Ukupna potrošnja:**
- Ukupna potrošnja u tekućem mesecu
- Preostali limit do kraja meseca

**Pregled po priređivaču:**
- Potrošnja kod svakog priređivača pojedinačno
- Ukupan broj uplata kod svakog priređivača

**Pojedinačne transakcije:**
Za svaku transakciju igrač vidi:
- Datum i vreme
- Iznos uplate
- Ime priređivača
- **Jedinstveni broj transakcije u Centralnom Registru**

**Obaveza priređivača:**
Priređivač je dužan da na priznanici (radnje) ili online pregledu uplate (online) prikaže jedinstveni broj transakcije iz Centralnog Registra Igrača.

---

### 6. Samoisključenje

Igrač koji zaključi da ima problem zavisnosti može sebi zabraniti igru na određeni period:

**Dostupni periodi:**
- 3 meseca
- 6 meseci
- 12 meseci
- 5 godina
- 10 godina
- Trajno samoisključenje

#### Tokom perioda samoisključenja:

**Sprovođenje zabrane:**
- Sistemi za igre na sreću ovlašćenih priređivača **odbijaju sve uplate**
- QR kod u radnjama vraća odbijanje transakcije
- Online nalozi ne dozvoljavaju depozite

**Pristup računima:**
- Igrač i dalje može da se prijavi na online sisteme
- Cilj prijave: **Samo da povuče preostali depozit ili dobitke**
- Nove uplate nisu moguće

**Zabrana marketinga:**
- Priređivačima zakon zabranjuje da šalju marketinške poruke igračima u samoisključenju
- Kršenje ove zabrane podleže kaznama

#### Ukidanje samoisključenja pre isteka perioda:

**Strogi uslovi:**
- Moguće **samo po odobrenju lekara specijaliste** za bolesti zavisnosti
- Cilj: Sprečavanje ishitrenih odluka tokom krize
- Zaštita igrača od impulsivnih reakcija

**Procedura:**
1. Igrač podnosi zahtev
2. Potreban je pregled kod lekara specijaliste
3. Lekar daje ili ne daje saglasnost
4. Samo uz saglasnost lekara moguće je poništenje pre kraja perioda

---

### 7. Zabrana Marketinških Poruka

**Mogućnost nezavisna od samoisključenja:**
Igrač može zabraniti priređivačima da mu šalju marketinške poruke čak i ako nije u periodu samoisključenja.

**Pravna snaga:**
Priređivačima je ovaj zahtev **iste zakonske snage** kao i zabrana marketinških poruka u periodu samoisključenja.

**Šta obuhvata:**
- Email poruke o ponudama
- SMS notifikacije
- Push notifikacije
- Pozivi sa promocijama
- Bilo koja druga direktna marketinška komunikacija

**Sprovođenje:**
- Priređivač je dužan da poštuje ovaj zahtev
- Kršenje podleže kaznama kao i kršenje zabrane u samoisključenju

---

## Tehnički Stack

е-Играč aplikacija koristi sledeće tehnologije:

### Autentifikacija i Identitet

**ConsentID OAuth2:**
- Državna autentifikaciona platforma
- Visoka pouzdanost identifikacije
- Integracija sa eID.gov.rs sistemom

**Refresh Token Mehanizam:**
- Dugotrajne sesije bez ponovne prijave
- OAuth2 standard
- Bezbedno čuvanje tokena

### Mobilne Platforme

**Android aplikacija:**
- Nativna Android aplikacija
- Podrška za najnovije verzije Android OS-a

**iOS aplikacija:**
- Nativna iOS aplikacija
- Podrška za iPhone i iPad uređaje

### Komunikacija

**REST API:**
- Standardizovana API komunikacija
- Siguran HTTPS prenos podataka
- Rate limiting i zaštita od zloupotrebe

**QR Kod Tehnologija:**
- Dinamički QR kodovi
- Vremensko ograničenje (60 sekundi validnost)
- Enkripcija podataka u QR kodu

### Bezbednost

**Šifrovanje podataka:**
- End-to-end enkripcija komunikacije
- Sigurno čuvanje osetljivih podataka na uređaju
- TLS/SSL za sve mrežne komunikacije

**Biometrijska autentifikacija:**
- Touch ID / Face ID (iOS)
- Fingerprint / Face Unlock (Android)
- Dodatni sloj zaštite za osetljive operacije

---

## Glavne Prednosti е-Играч Sistema

### Centralizovana Kontrola

**Jedinstveni prolaz za sve priređivače:**
- Jedna aplikacija za sva kockanja
- Automatska integracija sa svim licenciranim priređivačima
- Nema potrebe za instalacijom više aplikacija

**Centralni Registar Igrača:**
- Svi igrači registrovani na jednom mestu
- Sve transakcije logirane centralno
- Real-time praćenje limita između priređivača

### Zaštita Igrača

**Automatska provera starosti:**
- Tehnička nemogućnost aktivacije za maloletnike
- Integracija sa državnim registrima
- Bez mogućnosti zaobilaženja

**Efikasno samoisključenje:**
- Važi kod svih licenciranih priređivača automatski
- Trenutno sprovođenje
- Nema potrebe da igrač traži isključenje kod svakog priređivača pojedinačno

**Transparentnost potrošnje:**
- Igrač vidi tačno koliko troši
- Pregled svih transakcija na jednom mestu
- Upozorenja kada se približava limitu

### Olakšanje za Priređivače

**Automatska verifikacija:**
- Priređivač ne mora implementirati sopstveni KYC
- Automatska provera starosti
- Smanjenje regulatornih rizika

**Smanjenje troškova:**
- Nema potrebe za sopstvenim sistemima verifikacije
- Jednostavna integracija preko API-ja
- Država preuzima odgovornost za proveru identiteta

**Usklađenost sa zakonom:**
- Automatska usklađenost sa svim zakonskim zahtevima
- Centralno praćenje samoisključenja
- Automatsko logovanje transakcija

---

## Razlika е-Играč vs. Mini Aplikacije Operatora

### Trenutni Model (Bez е-Играч):

**Fragmentisano okruženje:**
- Svaki operator ima svoju aplikaciju
- Igrač mora skinuti 5-10 aplikacija za različite operatore
- Nema centralne kontrole
- Limiti važe samo kod jednog operatora
- Samoisključenje mora se tražiti kod svakog operatora posebno

**Problemi:**
- Igrači zaobilaze limite otvaranjem naloga kod više operatora
- Nema centralnog pregleda potrošnje
- Samoisključenje je neefikasno
- Provera starosti nekonsistentna

### Novi Model (Sa е-Играč):

**Centralizovano okruženje:**
- **Jedna aplikacija za sve priređivače**
- Centralna kontrola preko Centralnog Registra Igrača
- Operatori mogu nuditi "mini aplikacije" unutar е-Играч platforme
  - Ali sve transakcije prolaze kroz centralni sistem
  - Limiti važe kod svih priređivača

**Prednosti:**
- Nemoguće zaobići limite preko više naloga
- Centralni pregled cele potrošnje
- Samoisključenje automatski važi kod svih
- Konzistentna provera starosti

**Arhitektura:**

```
┌──────────────────────────────────────────────────────┐
│           Stari Model (Trenutni)                     │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Igrač → Aplikacija Operatora 1 (Direktan pristup)  │
│  Igrač → Aplikacija Operatora 2 (Direktan pristup)  │
│  Igrač → Sajt Operatora (Direktan pristup)          │
│  Igrač → Fizički Kazino (Ulaz)                      │
│                                                      │
│  Problem: Nema centralne tačke sprovođenja          │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│           Novi Model (Obavezna Aplikacija)           │
├──────────────────────────────────────────────────────┤
│                                                      │
│              ┌─────────────────┐                    │
│              │  Državna App    │                    │
│              │  "е-Играч"      │                    │
│              │                 │                    │
│              │  - Vezan uređaj │                    │
│              │  - Biometrija   │                    │
│              │  - Centralna    │                    │
│              │    autent.      │                    │
│              └────────┬────────┘                    │
│                       │                              │
│         ┌─────────────┼─────────────┐               │
│         │             │             │               │
│    ┌────▼───┐   ┌────▼───┐   ┌────▼────┐          │
│    │ Op. 1  │   │ Op. 2  │   │ Casino  │          │
│    │(mini)  │   │(mini)  │   │ (QR kod)│          │
│    └────────┘   └────────┘   └─────────┘          │
│                                                      │
│  Operatori nemaju samostalne aplikacije - samo      │
│  "mini aplikacije" unutar е-Играč platforme         │
└──────────────────────────────────────────────────────┘
```

---

## Korisničko Iskustvo - Tipični Scenariji

### Scenario 1: Prvi Put Korisnik

**Marko (25 godina) želi da se registruje za online kockanje:**

1. **Preuzimanje:** Preuzima е-Играч aplikaciju sa App Store / Google Play
2. **Aktivacija:** Otvara aplikaciju, unosi svoj email sa eID.gov.rs
3. **Autentifikacija:** Preusmerava se na ConsentID prijavu (kao eUprava)
4. **Potvrda identiteta:** Prijavljuje se sa svojim državnim identitetom
5. **Postavljanje limita:** Aplikacija traži da postavi mesečni limit (OBAVEZNO)
   - Marko bira €200/mesečno
6. **Registracija kod operatora:** Ide na sajt Mozzart-a, klikne "Registruj se"
7. **Unos tokena:** Mozzart traži token iz е-Играч aplikacije
8. **Generisanje:** Marko otvara aplikaciju, generiše token (npr. `7f9-248-680-423`)
9. **Završetak:** Unosi token na Mozzart sajtu, račun automatski verifikovan
10. **Igranje:** Marko može da igra, svi depoziti prolaze kroz е-Играč kontrolu

### Scenario 2: Uplata Tiketa u Radnji

**Ana (32 godine) želi da uplati tiket u kladionici:**

1. **Priprema:** Ana popuni tiket i donese 1.000 dinara
2. **Predaja:** Preda tiket i novac blagajniku
3. **Unos:** Blagajnik unese tiket u sistem
4. **Zahtev:** Sistem traži QR kod
5. **Otvaranje aplikacije:** Ana otvara е-Играч na telefonu
6. **Generisanje QR:** Klikne "Prikaži QR kod"
7. **Skeniranje:** Blagajnik skenira QR sa Aninog telefona
8. **Provera:** Centralni Registar proverava:
   - Ana nije samoisključena ✓
   - Još nije dostigla mesečni limit ✓
   - Transakcija odobrena ✓
9. **Potvrda:** Ana dobija priznanicu sa jedinstvenim brojem transakcije
10. **Pregled:** Ana može u aplikaciji videti ovu transakciju i preostali limit

### Scenario 3: Samoisključenje

**Milan (28 godina) shvata da ima problem sa kockanjem:**

1. **Odluka:** Odlučuje da se samoisključi na 6 meseci
2. **Aktivacija:** Otvara е-Играч aplikaciju
3. **Izbor:** Bira "Samoisključenje" iz menija
4. **Period:** Bira period: 6 meseci
5. **Potvrda:** Aplikacija traži potvrdu (objašnjava posledice)
6. **Aktiviranje:** Milan potvrđuje, samoisključenje aktivno
7. **Automatsko sprovođenje:**
   - Svi online nalozi blokirani za depozite
   - QR kod u radnjama ne radi (vraća odbijanje)
   - Priređivači ne mogu slati marketinške poruke
8. **Dan 30:** Milan pokušava da igra, ali sve aplikacije odbijaju
9. **Kontakt podrške:** Poziva podršku, ali mu kažu da mora čekati 6 meseci
10. **Dan 180:** Samoisključenje ističe, Milan dobija notifikaciju
11. **Period hlađenja:** Mora čekati još 7 dana pre nego što može ponovo igrati
12. **Resetovanje limita:** Limiti se resetuju na podrazumevane vrednosti

---

## Implementaciona Strategija

### Faza 1: Pilot Program (Meseci 1-6)

**Obim:**
- 1.000-5.000 dobrovoljnih korisnika (beta testeri)
- 3-5 licenciranih operatora (dobrovoljci)
- Samo online kockanje (još nema fizičkih kazina)

**Cilj:**
- Testiranje korisničkog iskustva
- Otkrivanje tehničkih problema
- Prikupljanje povratnih informacija
- Optimizacija procesa

**Kriterijumi uspeha:**
- Manje od 5% stopa tehničkih neuspeha
- Više od 80% zadovoljstvo korisnika
- 0 bezbednosnih incidenata
- Regulatorno odobrenje za sledeću fazu

### Faza 2: Obavezno Online (Meseci 7-12)

**Obim:**
- SVI online operatori obavezni da integrišu е-Играč
- Procenjeno 50.000-100.000 igrača
- Aktivno sprovođenje limita između operatora

**Ključne aktivnosti:**
- Migracija postojećih igrača (period prilagođavanja od 6 meseci)
- Stare aplikacije operatora zastarele (preusmerenje ka е-Играč)
- Edukativna kampanja za korisnike

**Rok za usklađenost:**
- Operatori moraju integrisati sistem u roku od 6 meseci
- Kazne za neusklađenost nakon krajnjeg roka

### Faza 3: Integracija Fizičkih Kazina (Meseci 13-18)

**Obim:**
- Prijava QR kodom u svim kazinima
- Integracija sa klađionicama
- Praćenje gotovine između objekata

**Izazovi:**
- Starija demografija (potrebni kioski + pomoć osoblja)
- Kultura bazirana na gotovini
- Manjim objektima može nedostajati infrastruktura

**Rešenja:**
- Kioski na ulazu kazina za korisnike bez pametnih telefona
- Program obuke osoblja (1 nedelja)
- Subvencije za hardver za male operatore

---

## Zaključak

е-Играч mobilna aplikacija predstavlja sveobuhvatno rešenje za implementaciju mera odgovorne igre u Srbiji. Sistem kombinuje:

- **Državnu autentifikaciju** (ConsentID) za pouzdanu proveru identiteta
- **Centralni Registar Igrača** za efikasno sprovođenje limita i samoisključenja
- **Moderne tehnologije** (QR kodovi, biometrija, OAuth2) za bezbednost
- **Jednostavno korisničko iskustvo** sa jednom aplikacijom za sve priređivače

Ključne prednosti sistema:
- Automatska zaštita maloletnika
- Efikasno samoisključenje kod svih priređivača
- Transparentan pregled potrošnje
- Obavezni mesečni limiti sa "periodom hlađenja"
- Smanjenje fragmentacije trenutnog sistema

**Napomena:** Ovaj dokument opisuje osnovne funkcionalnosti sistema. Za napredna unapređenja, tehničke specifikacije arhitekture, analizu troškova i koristi, kao i detaljnu implementacionu strategiju, konsultujte dokument "eIgrac-Improvements.md".

---

**Za više informacija:**
- Napredna unapređenja: `eIgrac-Improvements.md`
- Analiza trenutnog stanja u Srbiji: `srbija-analiza-stanja-2025.md`
- Originalni dokument: `Obavezna-Mobilna-Aplikacija.md`
