# Obavezna Mobilna Aplikacija za Kockanje: Revolucija u Sprovođenju

## Izvršna Sinteza

**Analiza 30+ evropskih jurisdikcija otkriva kritičan nedostatak: kontrolne mehanizme zaobilaze igrači preko više naloga, inostranih operatora, i anonimnog pristupa.** Obavezna državna mobilna aplikacija kao jedini prolaz za sve kockanje rešava 80% problema sprovođenja identifikovanih u trenutnim sistemima.

Ovaj dokument analizira:
1. **Kritične slabosti** u trenutnim sistemima sprovođenja (Srbija + EU)
2. **Kako mobilna aplikacija zatvara svaki nedostatak**
3. **Tehničku arhitekturu** obaveznog aplikacijskog sistema
4. **Pravne i političke implikacije**
5. **Implementacionu strategiju** sa fazama

---

## DEO 1: Analiza Trenutnih Problema sa Sprovođenjem

### Iz Analize EU Strategija: 10 Najvećih Neuspeha u Sprovođenju

#### 1. Zaobilaženje Preko Više Naloga

**Problem identifikovan:**
- Nemačka: Limiti €1,000 mesečno, ali igrači otvaraju 5-10 naloga kod različitih operatora
- Holandija: 90% kanalisanja igrača, ali samo 50% prihoda - igrači visokih uloga koriste više naloga kod inostranih operatora
- Samo 24.5% igrača koristi limite depozita jer znaju da mogu otvoriti drugi nalog

**Glavni uzrok:**
- Identifikacija bazirana na dokumentima - lako falsifikovati
- Nema vezivanja uređaja - isti igrač koristi isti telefon za sve naloge
- Operatori nemaju proveru u realnom vremenu između njih

#### 2. Dominacija Crnog Tržišta

**Podaci:**
- **Nemačka:** 54% prihoda (~€4 milijarde) na crnom tržištu, 15.8M građana (20%) koristi nelicencirane operatore
- **Francuska:** €748M-€1.5B u nelegalnim onlajn kazinima
- **Švedska:** 38-49% samoisključenih igrača nastavlja da igra na inostranim sajtovima

**Glavni uzrok:**
- VPN zaobilazi geografsko blokiranje
- Kriptovalutna plaćanja zaobilaze blokiranje plaćanja
- Inostrani operatori nude bolji korisnički doživljaj bez mera zaštite
- Nema načina da se spreči instalacija inostranih aplikacija

#### 3. Kršenje Samoisključenja

**Problem:**
- Velika Britanija: 250,000 igrača posetilo neregulirane sajtove tokom Svetskog prvenstva 2022
- Nema načina da se blokira pristup jednom kada igrač ode na drugi sajt ili aplikaciju
- Operatori ne mogu fizički sprečiti prijavu - samo mogu da zatvore nalog

#### 4. Praznine u Proveri Identiteta

**Trenutno stanje:**
- 91% operatora koristi biometrijsku verifikaciju, ali **to je na nivou operatora**
- Nema centralne biometrijske baze podataka
- Igrač može da koristi bratove ili prijateljeve identifikacione dokumente
- Onlajn verifikacija lako se zaobilazi sa tehnologijom lažnog lica

**Danska uspešna:** MitID + Kartice Igrača = nema anonimnog kockanja
- Ali, ovo radi samo u Danskoj - čim odeš onlajn na operatora licenciranog na Malti, MitID ne postoji

#### 5. Kašnjenja u Sprovođenju u Realnom Vremenu

**Problem:**
- Holandija: 1 sat maksimalno vreme odgovora kada se detektuje preterano igranje
- Ali, kako operator zna da igrač nije **istovremeno** igrao kod drugog operatora?
- Nemački LUGAS fajl aktivnosti pokušava ovo, ali samo za licencirane nemačke operatore

#### 6. Praznine u Proveri Plaćanja

**Problem:**
- Gotovinske transakcije < €2,000 (prag za sprečavanje pranja novca) ne prate se
- Prepejd kartice omogućavaju anonimne depozite
- Kripto transakcije zaobilaze bankarski sistem
- Elektronski novčanici (Skrill, Neteller) stvaraju sloj anonimnosti

#### 7. Nemogućnost Prekograničnog Sprovođenja

**Problem:**
- **Ne postoji EU sistem samoisključenja**
- 321 različitih dozvola za kockanje kroz 21 zemlju
- GDPR blokira deljenje podataka između jurisdikcija
- Igrač isključen u Francuskoj može slobodno igrati kod operatora sa Malte

#### 8. Zaobilaženje Preko VPN-a

**Problem:**
- Geografsko blokiranje lako se zaobilazi sa VPN-om od $5 mesečno
- Nemački igrač pristupa kazinu sa Kurasaa preko VPN-a
- Operatori ne mogu (legalno) agresivno blokirati VPN jer narušava legitimnu upotrebu privatnosti

#### 9. Niska Upotreba Alata (Dobrovoljni Alati Ne Funkcionišu)

**Podaci:**
- Samo **24.5%** koristi limite depozita
- Samo **8.1%** koristi pauze
- Igrači vide alate kao "za problematične kockare", a ne preventivne

**Glavni uzrok:**
- Alati su zakopani u podešavanjima naloga
- Nema obaveznog angažovanja
- Nema trenja - prebrzo je od instalacije aplikacije do prve opklade

#### 10. Deljenje Uređaja i Provera Godina

**Problem:**
- Maloletnici koriste uređaje ili naloge roditelja
- Biometrijska verifikacija na nivou aplikacije operatora ne sprečava deljenje uređaja
- 16-17 godišnjaci lako zaobilaze provere godina onlajn

---

### Iz Analize Serbia-Gambling.md: Dodatni Izazovi u Sprovođenju

#### Specifični Problemi u Srbiji:

**1. Zahtevi za Oglednom Bazom Podataka (Član 51)**
- Operatori moraju održavati oglednu ili punu bazu podataka u Srbiji
- Ali, kako verifikovati tačnost podataka?
- Nema mogućnosti revizije u realnom vremenu trenutno

**2. Povezivanje Video Nadzora**
- Fizički objekti moraju imati video vezu u realnom vremenu ka Upravi
- Ali, kako sprečiti igrača koji je isključen u fizičkom kazinu da igra onlajn?

**3. Sprovođenje 5-Godišnjeg Čuvanja**
- Zakon zahteva čuvanje 5 godina
- Operatori mogu brisati podatke bez detekcije
- Nema nepromenjivosti kao kod blokčejna

**4. Limit Gotovinskih Transakcija (RSD 1,175,000 / €10K po 30 dana)**
- Kako pratiti gotovinu u fizičkim objektima kada igrač posećuje 5 različitih kazina?
- Trenutno je sistem poverenja

**5. Zahtev za Fizičko Prisustvo pri KYC-u**
- Nemačka zahteva verifikaciju licem u lice sa originalnim dokumentima
- U praksi, operatori outsource-uju na "centre za verifikaciju"
- Problem kontrole kvaliteta

---

## DEO 2: Mobilna Aplikacija kao Prolaz za Sprovođenje

### Koncept: "OAuth za Kockanje" Model

**Centralna Premisa:**
> **Nijedan operator ne sme direktno pružiti pristup uslugama kockanja. Sve transakcije kockanja moraju proći kroz državnu obaveznu aplikaciju koja sprovodi sve mere zaštite pre nego što prosleđuje zahtev ka operatoru.**

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
│              │  "KockaID"      │                    │
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
│  "mini aplikacije" unutar KockaID platforme         │
└──────────────────────────────────────────────────────┘
```

---

## DEO 3: Kako Aplikacija Rešava Svaki Problem

### 1. Sprečavanje Više Naloga (Vezivanje Uređaja + Biometrija)

**Mehanizam Sprovođenja:**

```csharp
public class VezivanjeUredjaja
{
    public string IgracID { get; set; }           // SHA-256(JMBG)
    public string UredjajID { get; set; }         // Hardverski ID (iOS: IdentifierForVendor, Android: Android ID)
    public string BiometrijskiHash { get; set; }  // Heš šablona Face ID / Touch ID
    public DateTime DatumPrvogVezivanja { get; set; }
    public bool JeAktivanUredjaj { get; set; }

    // PRAVILO: Jedan igrač može imati maksimalno 2 aktivna uređaja (telefon + tablet)
    // Promena uređaja dozvoljena samo jednom na 90 dana uz ponovu verifikaciju ličnom kartom
}

public class TokRegistracije
{
    public async Task<RezultatRegistracije> RegistrujIgraca()
    {
        // Korak 1: Skeniranje državne lične karte (čitanje MRZ)
        var podaciLK = await SkenirajDrzavnuLicnuKartu();

        // Korak 2: Detekcija živosti (trepni, okreni glavu) - sprečava lažiranje fotografijom
        var zivoLice = await IzvrsiProveruZivosti();

        // Korak 3: Uporedi fotografiju sa LK sa živim licem (99.9% tačnost)
        if (!await UporedjLica(podaciLK.Fotografija, zivoLice))
            throw new Exception("Verifikacija identiteta neuspešna");

        // Korak 4: Proveri centralni registar za postojeće naloge
        var postojeciIgrac = await ProveriCentralniRegistar(podaciLK.JMBG);
        if (postojeciIgrac != null)
        {
            // Igrač već registrovan
            if (postojeciIgrac.BrojUredjaja >= 2)
                throw new Exception("Maksimalno 2 uređaja po igraču. Kontaktirajte podršku za promenu uređaja.");

            // Dodaj ovo kao drugi uređaj
            await VeziDodatniUredjaj(postojeciIgrac.IgracID, UzmiUredjajID());
        }
        else
        {
            // Novi igrač
            var igracID = SHA256(podaciLK.JMBG + SALT);
            await KreirajIgraca(igracID, podaciLK, UzmiUredjajID());
        }

        // Korak 5: Veži biometriju za uređaj
        await RegistrujBiometriju(igracID, UzmiUredjajID());

        // Korak 6: Postavi obavezne početne limite
        await PostaviPodrazumevaneLimite(igracID); // €50/dan podrazumevano prvih 30 dana

        return new RezultatRegistracije { Uspeh = true, IgracID = igracID };
    }
}
```

**Kako ovo sprečava više naloga:**
- Jedan JMBG = jedan IgracID u centralnom registru
- Vezivanje uređaja = ne može instalirati aplikaciju na 10 telefona
- Vezivanje biometrije = ne može dati svoj telefon bratu da igra
- Limit promene uređaja od 90 dana = ne može kupiti nov telefon svaki dan

**Scenario iz stvarnog života:**
```
Scenario: Milan (25) pokušava da zaobiđe mesečni limit od €700

Pokušaj 1: Instalira aplikaciju na drugi telefon
❌ NE PROLAZI: Pokušava da registruje isti JMBG
        Centralni registar detektuje: "Igrač već postoji"
        Aplikacija kaže: "Već ste registrovani. Prijavite se sa postojećim nalogom."

Pokušaj 2: Koristi bratov JMBG za registraciju
❌ NE PROLAZI: Detekcija živosti pokazuje da lice ne odgovara bratovoj ličnoj karti
        Sistem kaže: "Verifikacija identiteta neuspešna. Molimo posetite kancelariju."

Pokušaj 3: Koristi deepfake video svog brata
❌ NE PROLAZI: Napredna detekcija živosti (3D sensing dubine, analiza teksture)
        Detektuje digitalnu manipulaciju
        Sistem označava za istragu prevare

Pokušaj 4: Uzme bratov telefon koji je već registrovan
❌ NE PROLAZI: Kada pokušava prijavu, aplikacija zahteva biometriju (Face ID)
        Milanovo lice ne odgovara bratovom registrovanom Face ID
        Sistem kaže: "Biometrijska verifikacija neuspešna"

Pokušaj 5: Pokušava inostrani sajt preko pretraživača
❌ NE PROLAZI: Sprečavanje dubokih linkova + blokiranje na DNS nivou
        Kada klikne na oglas za kockanje, telefon otvara KockaID aplikaciju
        Aplikacija kaže: "Neautorizovan pristup kockanju. Koristite samo licencirane operatore."
```

**Rezultat:** Nemoguće kreirati više naloga. Jedan igrač = jedan nalog = limiti koje je moguće sprovesti.

---

### 2. Eliminacija Crnog Tržišta (App Store + DNS Kontrola)

**Višeslojno Sprovođenje:**

#### Sloj 1: Čuvar App Store-a

```yaml
iOS App Store Pravila:
  - Samo KockaID aplikacija odobrena za kategoriju kockanja u srpskom App Store
  - Sve aplikacije operatora odbijene (nisu odobrene za distribuciju)
  - Inostrane aplikacije za kockanje geografski blokirane za srpske Apple ID-ove

Google Play Pravila:
  - Ista politika kao iOS
  - Google Play Protect označava nelicencirane aplikacije za kockanje
  - Device Policy Manager (Android Enterprise) blokira bočno instaliranje aplikacija za kockanje
```

#### Sloj 2: Blokiranje na DNS Nivou (nivo internet provajdera)

```
Srpski internet provajderi obavezni da implementiraju:
1. DNS ponor za poznate inostrane domene za kockanje
   - Cloudflare-ova lista domena (10,000+ sajtova za kockanje)
   - Zahtevi za offshore-gambling.com → preusmerenje na warning.kockaid.rs

2. Dubinska Inspekcija Paketa (DPI) za obrasce saobraćaja kockanja
   - Detektuje WebSocket saobraćaj ka poznatim serverima za kockanje
   - Blokira u realnom vremenu

3. HTTPS SNI (Server Name Indication) inspekcija
   - Detektuje šifrovani saobraćaj ka domenima na crnoj listi
   - Ne otvara šifrovanje, samo čita ime servera
```

#### Sloj 3: Detekcija i Sprečavanje VPN-a

```csharp
public class DetekcijaVPN
{
    public async Task<bool> DetektujVPN()
    {
        var indikatori = new List<bool>();

        // Provera 1: Poznati VPN IP opsezi
        var javniIP = await UzmiJavniIP();
        indikatori.Add(await JePoznatVPNIP(javniIP));

        // Provera 2: Nepodudaranje vremenske zone
        var vrZonaUredjaja = TimeZone.CurrentTimeZone;
        var vrZonaIP = await UzmiVremenskuZonuIzIP(javniIP);
        indikatori.Add(vrZonaUredjaja != vrZonaIP);

        // Provera 3: GPS lokacija vs IP lokacija
        var gpsLokacija = await UzmiGPSLokaciju();
        var ipLokacija = await UzmiIPLokaciju(javniIP);
        var razdaljina = IzracunajRazdaljinu(gpsLokacija, ipLokacija);
        indikatori.Add(razdaljina > 100); // Više od 100km udaljenosti

        // Provera 4: Detekcija mrežnog interfejsa (iOS/Android)
        var mrezniInterfejsi = UzmiAktivneMrezneInterfejse();
        indikatori.Add(mrezniInterfejsi.Any(n => n.Ime.Contains("tun") || n.Ime.Contains("vpn")));

        // Provera 5: Provera DNS servera
        var dnsServeri = UzmiDNSServere();
        indikatori.Add(await SuPoznatVPNDNS(dnsServeri)); // Npr., 1.1.1.1 za WARP, 8.8.8.8 ponekad VPN

        // Ako su 2+ indikatora pozitivna = verovatno VPN
        return indikatori.Count(x => x) >= 2;
    }

    public async Task SprovediZabranuVPN()
    {
        if (await DetektujVPN())
        {
            // Blokiraj sve funkcionalnosti kockanja
            throw new DetektovanVPNIzuzetak("VPN detektovan. Isključite VPN za pristup uslugama kockanja.");

            // Logiraj kršenje
            await LogirajBezbednosnoKrsenje(IgracID, "POKUSAJ_KORISTENJA_VPN", UzmiVPNDetalje());

            // Eskalacija ako se ponavlja (3+ pokušaja = pregled naloga)
            if (await UzmiBrojVPNPokusaja(IgracID) >= 3)
            {
                await OznaciZaManuelniPregled(IgracID, "Ponovljeni pokušaji zaobilaženja putem VPN");
                await PrivremenoSuspendu jNalog(IgracID, sati: 24);
            }
        }
    }
}
```

#### Sloj 4: Prikačinjavanje Sertifikata (Sprečavanje Pristupa Inostranim API-jima)

```csharp
// Aplikacija hardkoduje samo sertifikate licenciranih operatora
public class PrikacinjanjeSertifikata
{
    private static readonly string[] DOZVOLJENI_SERTIFIKATI = {
        "sha256/abc123...",  // Sertifikat Licenciranog Operatora 1
        "sha256/def456...",  // Sertifikat Licenciranog Operatora 2
        "sha256/xyz789..."   // KockaID backend sertifikat
    };

    public async Task<HttpResponseMessage> NapraviZahtev(string url)
    {
        // Validiraj da serverski sertifikat odgovara prikačenim sertifikatima
        var handler = new HttpClientHandler
        {
            ServerCertificateCustomValidationCallback = (poruka, sert, lanac, greske) =>
            {
                var sertHash = IzracunajSHA256(sert.RawData);
                return DOZVOLJENI_SERTIFIKATI.Contains(sertHash);
            }
        };

        var klijent = new HttpClient(handler);
        return await klijent.GetAsync(url);
    }
}
```

**Rezultat:**
- Inostrane aplikacije ne mogu se instalirati (App Store blokira)
- Inostrani sajtovi ne rade (DNS blok)
- VPN ne pomaže (višestruka detekcija sa suspenzijom naloga)
- Igrači **moraju** koristiti licencirane operatore kroz KockaID aplikaciju

---

### 3. Sprovođenje Samoisključenja (Zaključavanje na Nivou Uređaja)

**Problem sa trenutnim sistemima:**
- Samoisključenje je unos u bazi podataka
- Operator proverava bazu prilikom prijave
- **Ali**, igrač može jednostavno otići na sajt drugog operatora

**KockaID Aplikacija Rešenje:**

```csharp
public class SprovodjenjeS amoisključenja
{
    public async Task PrimeniSamoisključenje(string igracID, TipIsključenja tip, int trajanjeUMesecima)
    {
        // 1. Upiši u centralni registar
        await CentralniRegistar.PostaviIsključenje(igracID, tip, trajanjeUMesecima);

        // 2. ZAKLJUČAVANJE NA NIVOU UREĐAJA
        await ZakljucajFunkcionalnostKockanjaUredjaja(igracID);

        // 3. Obavesti sve operatore (trenutno)
        await ObavestiSveOperatore(igracID, StatusIsključenja.Aktivan);

        // 4. Opozovi sve aktivne sesije
        await OpozovjAktivneSesije(igracID);

        // 5. Blokiraj pristup aplikaciji
        await PostaviNivoPristupaAplikaciji(igracID, NivoPristupa.Samoisključen);
    }

    public async Task<bool> MozeIgracIgrati(string igracID)
    {
        // Prvo proveri lokalni keš uređaja (trenutno, radi offline)
        var lokalniStatus = await UzmiLokalniStatusIsključenja(igracID);
        if (lokalniStatus == StatusIsključenja.Aktivan)
        {
            PrikaziPoruSamoisključenju();
            return false;
        }

        // Proveri centralni registar (dnevna sinhronizacija u pozadini)
        var centralniStatus = await CentralniRegistar.UzmiStatusIsključenja(igracID);
        if (centralniStatus == StatusIsključenja.Aktivan)
        {
            // Ažuriraj lokalni keš
            await AzurirajLokalniKes(igracID, centralniStatus);
            PrikaziPoruSamoisključenju();
            return false;
        }

        return true;
    }

    private async Task ZakljucajFunkcionalnostKockanjaUredjaja(string igracID)
    {
        // iOS/Android sigurno skladište - ne može se obrisati bez resetovanja na fabrička podešavanja
        await SigurnoSkladiste.PostaviAsync($"isključenje_{igracID}", "AKTIVAN");

        // Postavi ograničenja na nivou uređaja
        var uredjaji = await UzmiIgraceveUredjaje(igracID);
        foreach (var uredjaj in uredjaji)
        {
            await PosaljiPushNotifikaciju(uredjaj.UredjajID, new
            {
                Tip = "AKTIVIRANO_ISKLJUČENJE",
                Poruka = "Samoisključenje aktivirano. Aplikacija će biti zaključana.",
                JeTiha = false
            });

            // Prinudi aplikaciju da ponovo proveri status pri sledećem otvaranju
            await PonistavanjeSesijeUredjaja(uredjaj.UredjajID);
        }
    }
}
```

**Korisničko Iskustvo kada je samoisključen:**

```
Milan se samoisključio na 6 meseci.

Dan 1 (2 sata kasnije - kajanje):
- Otvara KockaID aplikaciju
- Aplikacija prikazuje: "Samoisključeni ste do [datum]"
- SVI dugmići su onemogućeni
- Samo opcije: "Prikaži Detalje Isključenja" | "Kontaktiraj Podršku" | "Resursi za Lečenje"
- Pokušava da obriše aplikaciju i reinstalira
  → Nakon reinstalacije, prijava sa Face ID
  → Aplikacija ponovo proverava centralni registar
  → Status: I dalje isključen
  → Pojavljuje se isti zaključani ekran

Dan 30 (želi da igra ponovo):
- Otvara aplikaciju → I dalje zaključana
- Poziva podršku → "Ne možete ukloniti isključenje ranije. Čekajte još 5 meseci."
- Pokušava da instalira inostranu aplikaciju
  → App Store kaže "Nije dostupno u vašoj regiji"
- Pokušava inostrani sajt
  → DNS preusmerenje na stranicu upozorenja
- Pokušava VPN + inostrani sajt
  → KockaID aplikacija detektuje VPN
  → Logiraj bezbednosno kršenje
  → Email: "VPN detektovan. Ponovljeni pokušaji mogu rezultirati produženim isključenjem."

Dan 180 (isključenje ističe):
- Aplikacija prikazuje: "Vaše isključenje je isteklo. Potreban je period hlađenja od 7 dana."
- Nakon 7 dana: "Hlađenje završeno. Možete nastaviti sa kockanjem."
- Mora ponovo potvrditi: "Razumem rizike" (ne može preskočiti)
- Limiti resetovani na podrazumevano (€50/dan prvih 30 dana)
```

**Rezultat:** Samoisključenje postaje zaista sprovodivo. Ne može zaobići jer je zaključavanje na nivou aplikacije + vezivanje uređaja.

---

### 4. Sprovođenje Limita između Operatora u Realnom Vremenu

**Problem sa trenutnim:**
- Nemačka ima mesečni limit od €1,000 u LUGAS
- Ali, šta ako igrač deponuje €500 kod Operatora A, zatim brzo pređe kod Operatora B i deponuje još €600?
- LUGAS ažuriranja nisu trenutna (kašnjenje od minuta)

**KockaID Proksi u Realnom Vremenu:**

```csharp
public class SprovodjenjeL imitaURealnom Vremenu
{
    // SVI depoziti moraju proći kroz KockaID aplikaciju proksi
    public async Task<RezultatDepozita> ObbradiDepozit(string igracID, decimal iznos, string operatorID)
    {
        // 1. Pribavi distribuiranu bravu (Redis)
        using (var brava = await PribaviB ravu($"depozit_{igracID}", timeout: 5000))
        {
            if (brava == null)
                return new RezultatDepozita { Uspeh = false, Razlog = "Druga transakcija u toku. Pokušajte ponovo." };

            // 2. Uzmi trenutnu potrošnju (Redis keš + SQL rezerva)
            var trenutniLimiti = await UzmiTrenutneLimite(igracID);

            // 3. Izračunaj novu potrošnju
            var predloženo = new
            {
                Dnevno = trenutniLimiti.DnevniDepoziti + iznos,
                Nedeljno = trenutniLimiti.NedeljniDepoziti + iznos,
                Mesečno = trenutniLimiti.MesečniDepoziti + iznos
            };

            // 4. Proveri sve limite
            var limItigrača = await UzmiLimiteIgrača(igracID);

            if (predloženo.Dnevno > limItigrača.DnevniLimit)
                return new RezultatDepozita
                {
                    Uspeh = false,
                    Razlog = $"Dnevni limit premašen. Limit: {limItigrača.DnevniLimit}, Trenutno: {trenutniLimiti.DnevniDepoziti}, Predloženo: {iznos}",
                    SledeceD ostupno = Sutra()
                };

            if (predloženo.Nedeljno > limItigrača.NedeljniLimit)
                return new RezultatDepozita { Uspeh = false, Razlog = "Nedeljni limit premašen" };

            if (predloženo.Mesečno > limItigrača.MesečniLimit)
                return new RezultatDepozita { Uspeh = false, Razlog = "Mesečni limit premašen" };

            // 5. Proveri bihevioralne zastavice (pokretano AI-em)
            var rezultatRizika = await IzračunajRezultatRizika(igracID);
            if (rezultatRizika > 70) // Visok rizik
            {
                // Pokreni intervenciju
                await PokreniIntervenciju(igracID, rezultatRizika, iznos);
                return new RezultatDepozita { Uspeh = false, Razlog = "Depozit blokiran radi vaše zaštite. Kontaktirajte podršku." };
            }

            // 6. Obradi plaćanje kroz prolaz za plaćanje kontrolisan aplikacijom
            var rezultatPlacanja = await ObradiPlacanje(igracID, iznos);
            if (!rezultatPlacanja.Uspeh)
                return new RezultatDepozita { Uspeh = false, Razlog = rezultatPlacanja.PorukaGreske };

            // 7. Trenutno ažuriraj limite (Redis)
            await AzurirajLimite(igracID, iznos);

            // 8. Prosled novčanik operatora
            await PrebacNaOperatora(operatorID, igracID, iznos, rezultatPlacanja.TransakcijaID);

            // 9. Logiraj transakciju
            await LogirajTransakciju(igracID, operatorID, iznos, TipTransakcije.Depozit);

            return new RezultatDepozita { Uspeh = true, NovoStanje = iznos };
        }
    }
}
```

**Arhitektura:**

```
┌─────────────────────────────────────────────────┐
│  Igrač želi da deponuje €100 kod Operatora A   │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
         ┌─────────────────┐
         │  KockaID App    │
         │  (Uređaj igrača)│
         └────────┬─────────┘
                  │ API Poziv
                  ▼
    ┌──────────────────────────┐
    │  KockaID Backend         │
    │  (Provera limita u       │
    │   realnom vremenu)       │
    │                          │
    │  Redis Keš:              │
    │  igrac_123_dnevno: €250  │  ← Trenutno ažurirano
    │  igrac_123_mesecno: €800 │
    │                          │
    │  Provera: €250 + €100 > €300 (dnevni limit)?
    │  → NE, dozvoli           │
    └──────────┬───────────────┘
               │ Odobreno
               ▼
    ┌──────────────────────┐
    │  Prolaz Plaćanja     │
    │  (Stripe/Adyen)      │
    └──────────┬───────────┘
               │ €100 naplaćeno
               ▼
    ┌──────────────────────┐
    │  Novčanik Operatora A│
    │  +€100               │
    └──────────────────────┘

5 minuta kasnije...

┌─────────────────────────────────────────────────┐
│  Igrač pokušava da deponuje €100 kod Op. B     │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
         ┌─────────────────┐
         │  KockaID App    │
         └────────┬─────────┘
                  │
                  ▼
    ┌──────────────────────────┐
    │  KockaID Backend         │
    │                          │
    │  Redis Keš:              │
    │  igrac_123_dnevno: €350  │  ← Ažurirano od prvog depozita
    │                          │
    │  Provera: €350 + €100 > €300 (dnevni limit)?
    │  → DA, BLOKIRAJ          │
    └──────────┬───────────────┘
               │ Odbijeno
               ▼
    ┌──────────────────────────────────┐
    │  Aplikacija prikazuje:           │
    │  ❌ "Dnevni limit premašen"      │
    │  "Deponovaliiste €350 danas"     │
    │  "Vaš dnevni limit je €300"      │
    │  "Sledeće dostupno: Sutra 00:00" │
    └──────────────────────────────────┘
```

**Rezultat:**
- Sprovođenje u realnom vremenu ispod sekunde
- Nemoguće zaobići preko više operatora (sve ide kroz isti prolaz)
- Distribuirana brava sprečava uslove trke
- Redis keš = trenutna čitanja (ne čeka SQL)

---

### 5. Praćenje Ponašanja sa Obaveznim Intervencijama

**EU Najbolja Praksa:** Holandija zahteva maksimalno vreme odgovora od 1 sat + ljudski kontakt (ne samo iskačući prozor)

**KockaID Implementacija:**

```csharp
public class PracenjePonasanja
{
    // Izvršava se svakih 5 minuta za aktivne igrače
    public async Task PratiAktivneIgrace()
    {
        var aktivniIgraci = await UzmiAktivneIgrace(); // Trenutno u sesiji

        foreach (var igrac in aktivniIgraci)
        {
            var ponasanje = await AnalizirajPonasanje(igrac.IgracID, minovaUnazad: 60);

            // Indikatori rizika
            var indikatori = new
            {
                TrajanjeSesije = ponasanje.MinutaSesije,
                BrziDepoziti = ponasanje.DepozitaZadnjih60Min,
                TrkaZaGubicima = ponasanje.PonavljaniDepoziti NakonGubitka,
                KasnoNocu = JeKasnoNocu(), // 23:00-06:00
                EskalacijaOpkladа = ponasanje.TrendProsecneVeličineOpklade,
                ViseIgara = ponasanje.IstovremeneIgre
            };

            // Izračunaj rezultat rizika
            var rezultat = IzračunajRezultatRizika(indikatori);

            // Pragovi intervencije
            if (rezultat >= 80) // KRITIČNO
            {
                await PrimorujHitnuIntervenciju(igrac.IgracID, rezultat, indikatori);
            }
            else if (rezultat >= 60) // VISOKO
            {
                await ZakaziHitnuIntervenciju(igrac.IgracID, rezultat, uMinutima: 30);
            }
            else if (rezultat >= 40) // UMERENO
            {
                await PosaljiPersonalizovanuPoruku(igrac.IgracID, rezultat);
            }
        }
    }

    private async Task PrimorujHitnuIntervenciju(string igracID, int rezultat, object indikatori)
    {
        // 1. PAUZIRAJ sve aktivne sesije
        await PauzirajSveAktivneSesije(igracID);

        // 2. Zaključaj funkcionalnosti kockanja u aplikaciji (ne može nastaviti)
        await PostaviPrivremenoZ akljucavanje(igracID, minutaTrajanja: 30);

        // 3. Prikaži obaveznu poruku preko celog ekrana
        await PrikaziEkranIntervencije(igracID, new
        {
            Naslov = "OBAVEZAN PREKID",
            Poruka = "Detektovali smo zabrinjavajuće ponašanje u vašoj igri.",
            Detalji = $"Rezultat Rizika: {rezultat}/100",
            Indikatori = indikatori,
            Akcije = new[]
            {
                "Razgovaraj sa savetnikom (obavezno)",
                "Postavi niže limite",
                "Samoisključi se"
            }
        });

        // 4. Obavesti tim podrške (za 5 minuta)
        await ObavestiTimPodrške(new
        {
            IgracID = igracID,
            Prioritet = "HITNO",
            RezultatRizika = rezultat,
            ZahtevanaAkcija = "Pozovite igrača za 5 minuta",
            KontaktInfo = await UzmiIgračevKontakt(igracID)
        });

        // 5. Logiraj intervenciju
        await LogirajIntervenciju(igracID, TipIntervencije.PrisilnaPauza, rezultat, indikatori);

        // 6. Zakaži praćenje (24 sata)
        await ZakaziPracenje(igracID, satiKasnije: 24);
    }

    private int IzračunajRezultatRizika(dynamic indikatori)
    {
        int rezultat = 0;

        // Ocenjivanje trajanja sesije (maksimalno 25 poena)
        if (indikatori.TrajanjeSesije > 180) rezultat += 25; // 3+ sata
        else if (indikatori.TrajanjeSesije > 120) rezultat += 20;
        else if (indikatori.TrajanjeSesije > 60) rezultat += 10;

        // Brzi depoziti (maksimalno 25 poena)
        if (indikatori.BrziDepoziti >= 5) rezultat += 25;
        else if (indikatori.BrziDepoziti >= 3) rezultat += 15;

        // Trka za gubicima (maksimalno 20 poena)
        if (indikatori.TrkaZaGubicima >= 3) rezultat += 20;
        else if (indikatori.TrkaZaGubicima >= 2) rezultat += 10;

        // Kasno noću (maksimalno 15 poena)
        if (indikatori.KasnoNocu) rezultat += 15;

        // Eskalacija opklada (maksimalno 15 poena)
        if (indikatori.EskalacijaOpklada > 100) rezultat += 15; // Udvostručio veličinu opklade
        else if (indikatori.EskalacijaOpklada > 50) rezultat += 10;

        // Više igara (maksimalno 10 poena)
        if (indikatori.ViseIgara > 3) rezultat += 10;

        return Math.Min(rezultat, 100);
    }
}
```

**Korisničko Iskustvo (Scenario Visokog Rizika):**

```
Milan igra slot igre već 2.5 sata. Izgubio €200, deponovao 4 puta.
Trenutno vreme: 01:30 (kasno noću)

Okidač praćenja ponašanja:
- Sesija: 150 minuta (20 poena)
- Brzi depoziti: 4 (20 poena)
- Jurenje: 2 ponovljena depozita nakon gubitka (10 poena)
- Kasno noću: DA (15 poena)
- Eskalacija opklada: 80% povećanje (10 poena)
→ Ukupno: 75 poena (VISOK RIZIK)

Odgovor aplikacije (trenutno):
┌─────────────────────────────────────────┐
│          ⚠ OBAVEZAN PREKID              │
├─────────────────────────────────────────┤
│                                         │
│ Detektovali smo zabrinjavajuće         │
│ ponašanje:                              │
│                                         │
│ ✓ Igrate već 2.5 sata                  │
│ ✓ Deponovali ste 4 puta                │
│ ✓ Trenutno gubite €200                 │
│ ✓ Povećali ste opklade za 80%          │
│                                         │
│ Vaša sesija je pauzirana na 30 minuta. │
│                                         │
│ [Razgovaraj sa savetnikom] (OBAVEZNO)  │
│ [Postavi niže limite]                  │
│ [Samoisključenje]                       │
│                                         │
│ Ne možete nastaviti dok ne izaberete   │
│ jednu od opcija.                        │
└─────────────────────────────────────────┘

Milan klikće "Razgovaraj sa savetnikom":
→ Otvara se video razgovor u aplikaciji (ili telefonski poziv)
→ Savetnik: "Pozdrav Milan, primećujemo da igrate dugo i gubite..."
→ Razgovor od 10 minuta
→ Savetnik preporučuje sniženje limita na €50/dan
→ Milan prihvata
→ Novi limit aktivan trenutno

Alternativa: Milan pokušava da zatvori aplikaciju
→ Aplikacija ne dozvoljava zatvaranje dok ne izabere akciju
→ Pokušava prisilno zatvaranje aplikacije
→ Ponovno otvaranje: Pojavljuje se isti ekran (ne može zaobići)
→ Pokušava deinstalaciju aplikacije
→ Reinstalacija: Nakon prijave, zaključavanje je i dalje aktivno (odbrojavanje 30 minuta)
```

**Rezultat:**
- Nemoguće ignorisati intervencije
- Pravi ljudski kontakt (ne samo iskačući prozor)
- Sprovođenje na nivou uređaja (ne može zatvoriti aplikaciju i zaobići)

---

### 6. Integracija Sistema Plaćanja (Sprečavanje Pranja Novca)

**Problem:**
- Kripto transakcije zaobilaze KYC
- Prepejd kartice = anonimne
- Gotovina u fizičkim objektima nije moguće pratiti između kazina

**KockaID Rešenje: Jedinstveni Prolaz za Plaćanje**

```csharp
public class ObaveziProlazPlaćanja
{
    // Samo dozvoljeni načini plaćanja
    public enum DozvoljeniNaciniPlacanja
    {
        BankovniRacun,        // Direktan bankovni transfer (potpun KYC)
        DebitnaKartica,       // Povezana sa verifikovanim bankovnim računom
        MobilnaBankacija      // Srpske bankarske aplikacije (mBanking)
        // NIJE dozvoljeno: Kreditne kartice, kripto, prepejd, elektronski novčanici
    }

    public async Task<RezultatNacinaPlacanja> DodajNacinPlacanja(string igracID, DetaljiNacinaPlacanja detalji)
    {
        // 1. Verifikuj vlasništvo načina plaćanja
        if (!await VerifikujVlasnistvo(igracID, detalji))
            return new RezultatNacinaPlacanja { Uspeh = false, Razlog = "Način plaćanja mora biti na vaše ime" };

        // 2. Proveri protiv baza podataka za sprečavanje pranja novca
        var amlProvera = await ProveriAML(detalji.BrojRacuna);
        if (amlProvera.JeOznačen)
        {
            await ObavestiVlasti(igracID, amlProvera);
            return new RezultatNacinaPlacanja { Uspeh = false, Razlog = "Verifikacija načina plaćanja neuspešna" };
        }

        // 3. Verifikacija mikro-depozitom (bankovski računi)
        if (detalji.Tip == TipNacinaPlacanja.BankovniRacun)
        {
            await PosaljiMikroDepozit(detalji.IBAN, slucajniIznos: 0.27); // €0.27
            return new RezultatNacinaPlacanja
            {
                Uspeh = false,
                Razlog = "Verifikujte mikro-depozit. Proverite bankovni izvod i potvrdite iznos u aplikaciji."
            };
        }

        // 4. Skladišti šifrovano
        await SkladistiNacinPlacanja(igracID, SifrujDetalјePlacanja(detalji));

        return new RezultatNacinaPlacanja { Uspeh = true };
    }

    public async Task<LimitiTransakcije> SprovediAMLLimite(string igracID, decimal iznos)
    {
        // Srpski zakon: Gotovinski limit RSD 1,175,000 (~€10K) po 30 dana
        var poslednjih30Dana = await UzmiUkupneTransakcije(igracID, dana: 30);

        if (poslednjih30Dana + iznos > 10000)
        {
            // Pokreni pojačanu dubinsku analizu
            await ZahteajVerifikacijuIzvoraS redstava(igracID, iznos);
            return new LimitiTransakcije
            {
                Dozvoljeno = false,
                Razlog = "Iznos premašuje 30-dnevni AML prag. Molimo verifikujte izvor sredstava.",
                ZahtevaniDokumenti = new[] { "Dokaz prihoda", "Bankovni izvodi (3 meseca)" }
            };
        }

        // Detekcija sumnjivog obrasca
        if (await DetektujSumnjivObrasc(igracID, iznos))
        {
            await OznaciZaPregled(igracID, "Sumnjiv obrazac transakcije");
            await ObavestiUpravu(igracID, iznos); // Uprava za finansijsku inteligenciju
        }

        return new LimitiTransakcije { Dozvoljeno = true };
    }
}
```

**Integracija Fizičkog Kazina (Prijava QR Kodom):**

```csharp
public class IntegracijaFizickogKazina
{
    // Igrač mora skenirati QR kod na ulazu u kazino
    public async Task<RezultatPrijave> PrijaviSeUKazino(string qrKod, string igracID)
    {
        var kazino = await UzmiKazinoIzQR(qrKod);

        // 1. Proveri samoisključenje
        if (await JeSamoisključen(igracID))
        {
            await AlarmirajObezbeđenjeKazina(kazino.ID, igracID, "Samoisključeni igrač pokušao ulaz");
            return new RezultatPrijave { Dozvoljeno = false, Razlog = "Samoisključeni ste" };
        }

        // 2. Proveri dnevni gotovinski limit
        var gotovинaDanas = await UzmiGoتوvinskeTrans akciјeDanas(igracID);
        var preostaliLimit = await UzmiPreostaliDnevniLimit(igracID, gotovинaDanas);

        if (preostaliLimit <= 0)
        {
            return new RezultatPrijave { Dozvoljeno = false, Razlog = "Dnevni gotovinski limit dostignut" };
        }

        // 3. Kreiraj virtualni novčanik za sesiju kazina
        var virNovcanik = await KreirajKazinoNovcanik(igracID, kazino.ID);

        // 4. Igrač može deponovati gotovinu kod kаsе kazina
        //    Kasir skenira QR kod aplikacije igrača + unosi iznos gotovine
        //    Iznos upisan u virtualni novčanik (praćen u aplikaciji)

        // 5. Igrač koristi novčanik za kockanje
        //    Sve transakcije logirane u realnom vremenu

        // 6. Pri odlasku, preostalo stanje može biti povučeno

        return new RezultatPrijave
        {
            Dozvoljeno = true,
            IDVirtualnogNovcanika = virNovcanik.ID,
            PreostaliDnevniLimit = preostaliLimit
        };
    }

    public async Task KasirDepozit(string idVirtualnogNovcanika, decimal iznos)
    {
        var novcanik = await UzmiNovcanik(idVirtualnogNovcanika);
        var igracID = novcanik.IgracID;

        // Proveri kumulativni limit između kazina
        var sviKazinoDepozitiDanas = await UzmiSveKazinoDepoziteDanas(igracID);

        if (sviKazinoDepozitiDanas + iznos > UzmiDnevniLimit(igracID))
        {
            throw new Exception($"Dnevni limit između kazina premašen. Deponovali ste €{sviKazinoDepozitiDanas} u svim kazinima danas.");
        }

        // Upisi na novčanik
        novcanik.Stanje += iznos;
        await AzurirajNovcanik(novcanik);

        // Logiraj transakciju
        await LogirajTransakciju(igracID, novcanik.KazinoID, iznos, TipTransakcije.GotovinskiDepozit);

        // Ažuriraj centralne limite
        await AzurirajCentralneLimite(igracID, iznos);
    }
}
```

**Rezultat:**
- Svi načini plaćanja su verifikovani i povezani sa pravim identitetom
- Praćenje gotovine između objekata (fizički kazini)
- AML usklađenost automatizovana
- Nema anonimnih depozita

---

### 7. Prekogranično Sprovođenje (Regionalna Saradnja)

**Problem:**
- Srpski igrač putuje u Hrvatsku, pokušava da igra tamo
- Hrvatska nema pristup srpskom registru samoisključenja
- Igrač zaobilazi zaštitu

**KockaID Rešenje: Prenosivi Digitalni Identitet**

```csharp
public class PrekograničnaZaštita
{
    // Kada igrač putuje u drugu zemlju
    public async Task<DozvolaZaKockanje> UzmiDozvoluZaKockanje(string igracID, string oznakaZemlje)
    {
        // 1. Proveri srpski status isključenja
        var srpskiStatus = await UzmiSrpskiStatusIsključenja(igracID);
        if (srpskiStatus.JeIsključen)
        {
            return new DozvolaZaKockanje
            {
                Dozvoljeno = false,
                Razlog = "Samoisključeni ste u Srbiji",
                VažiUZemljama = new string[] { } // Nema zemalja
            };
        }

        // 2. Proveri bilateralne sporazume
        var partnerZemље = new[] { "HR", "SI", "BA", "MK", "ME" }; // Balkanski region

        if (!partnerZemље.Contains(oznakaZemlje))
        {
            // Zemlja nije u sporazumu - primenjuju se lokalna pravila
            return new DozvolaZaKockanje
            {
                Dozvoljeno = true,
                Razlog = "Nema bilateralnog sporazuma. Primenjuju se lokalna pravila kockanja.",
                SrpskiLimitiAktivni = false
            };
        }

        // 3. Generiši prenosivi sertifikat isključenja
        var sertifikat = new
        {
            IgracID = igracID, // Pseudonimizovano
            StatusIsključenja = srpskiStatus.Status,
            Limiti = await UzmiLimiteIgrača(igracID),
            VažiDo = DateTime.UtcNow.AddHours(24),
            IzdaoOrgan = "Uprava za igre na sreću Srbije",
            Potpis = PotpisiPrivatnimKljučem(srpskiStatus) // Kriptografski dokaz
        };

        // 4. QR kod koji strani regulator može skenirati
        var qrKod = GenerišiQRKod(sertifikat);

        return new DozvolaZaKockanje
        {
            Dozvoljeno = true,
            Sertifikat = sertifikat,
            QRKod = qrKod,
            SrpskiLimitiAktivni = true, // Limiti i dalje važe u inostranstvu
            VažiUZemljama = partnerZemље
        };
    }

    // Strani operator (npr. Hrvatski kazino) skenira QR kod
    public async Task<RezultatVerifikacije> VerifikujStranog Igraca(string podaciQRKoda)
    {
        var sertifikat = ParsirajSertifikat(podaciQRKoda);

        // 1. Verifikuj kriptografski potpis
        if (!await VerifikujPotpis(sertifikat, "Srpski_JavniKlјuc"))
            return new RezultatVerifikacije { Validan = false, Razlog = "Nevažeći sertifikat" };

        // 2. Proveri istek
        if (sertifikat.VažiDo < DateTime.UtcNow)
            return new RezultatVerifikacije { Validan = false, Razlog = "Sertifikat istekao" };

        // 3. Proveri status isključenja
        if (sertifikat.StatusIsključenja == StatusIsključenja.Aktivan)
            return new RezultatVerifikacije
            {
                Validan = false,
                Razlog = "Igrač je samoisključen u matičnoj zemlji (Srbija)",
                PreporučenaAkcija = "Odbij ulaz"
            };

        // 4. Vrati limite Hrvatskoj operatora
        return new RezultatVerifikacije
        {
            Validan = true,
            SprovediLimite = sertifikat.Limiti, // Hrvatski operator treba poštovati srpske limite
            PseudonimIgrača = sertifikat.IgracID
        };
    }
}
```

**Scenario iz stvarnog života:**

```
Milan (samoisključen u Srbiji) putuje u Zagreb.

Pokušaj 1: Pokušava ući u hrvatski kazino
→ Kazino traži ličnu kartu
→ Milan pokazuje srpski pasoš
→ Zaposleni u kazinu: "Molim vas, pokažite KockaID aplikaciju"
→ Milan otvara aplikaciju
→ Aplikacija generiše QR kod sa sertifikatom isključenja
→ Zaposleni skenira QR kod
→ Ekran kaže: "⛔ IGRAČ SAMOISKLJUČEN (Srbija)"
→ Ulaz odbijen

Pokušaj 2: Pokušava onlajn na hrvatskom sajtu za kockanje
→ Sajt zahteva verifikaciju identiteta
→ Milan otprema srpski pasoš
→ API sajta šalje upit ka srpsko-hrvatskom međuregulatornom API-ju
→ API vraća: "Igrač isključen"
→ Kreiranje naloga blokirano

Prednosti Bilateralnog Sporazuma:
- Srbija priznaje hrvatska isključenja
- Hrvatska priznaje srpska isključenja
- Limiti se prenose (ako je Milanov limit €300/mesec u Srbiji, to važi i u Hrvatskoj)
- Integracija API-ja u realnom vremenu između regulatora
```

---

## DEO 4: Tehnička Arhitektura

### Pregled Sistemske Arhitekture

```
┌─────────────────────────────────────────────────────────────────┐
│                    KOCKAID OBAVEZNA APLIKACIJA                   │
│                     (iOS i Android Nativno)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐      │
│  │ Registracija │  │  Biometrijska│  │  Vezivanje      │      │
│  │   i KYC      │  │  Autentif.   │  │  Uređaja        │      │
│  └──────────────┘  └──────────────┘  └─────────────────┘      │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐      │
│  │ Mašina za    │  │ Sprovođenje  │  │  Praćenje       │      │
│  │ Limite       │  │ Samoisklj.   │  │  Rizika (AI/ML) │      │
│  └──────────────┘  └──────────────┘  └─────────────────┘      │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐      │
│  │   Prolaz     │  │   Mini       │  │   QR za         │      │
│  │   Plaćanja   │  │ Aplikacije   │  │   Fizičke       │      │
│  │              │  │  Operatora   │  │   Kazine        │      │
│  └──────────────┘  └──────────────┘  └─────────────────┘      │
│                                                                  │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS + mTLS
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                   API PROLAZ (Azure APIM)                        │
│                                                                  │
│  • Ograničavanje Brzine: 1000 zahteva/min po igraču            │
│  • JWT Validacija                                               │
│  • Sprovođenje Prikačinjavanja Sertifikata                     │
│  • Logiranje Zahteva/Odgovora                                   │
│  • DDoS Zaštita                                                 │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────────┐
         │               │                   │
         ▼               ▼                   ▼
┌─────────────┐  ┌──────────────┐  ┌────────────────┐
│  Servis za  │  │   Servis za  │  │    Servis za   │
│  Identitet  │  │  Isključenje │  │     Limite     │
│   Igrača    │  │              │  │                │
│             │  │ • Trajno     │  │ • Depozit      │
│ • KYC       │  │ • Privremeno │  │ • Gubitak      │
│ • Uređaj    │  │ • Hlađenje   │  │ • Vreme/Sesija │
│ • Biometrija│  │ • Između op. │  │ • Između op.   │
└─────────────┘  └──────────────┘  └────────────────┘

┌──────────────┐  ┌─────────────┐  ┌─────────────────┐
│   Servis za  │  │  AI Ocena   │  │  Servis za      │
│   Plaćanje   │  │   Rizika    │  │  Intervenciju   │
│              │  │             │  │                 │
│ • Prolaz     │  │ • 60+ Indic.│  │ • Poruke        │
│ • AML Prov.  │  │ • Realno vr.│  │ • Ljudski Kont. │
│ • Limiti     │  │             │  │ • Zaključavanje │
└──────────────┘  └─────────────┘  └─────────────────┘

┌──────────────┐  ┌─────────────┐  ┌─────────────────┐
│   Servis za  │  │   Servis za │  │   Servis za     │
│  Integraciju │  │  Fizičke    │  │  Obaveštenja    │
│  Operatora   │  │  Objekte    │  │                 │
│              │  │             │  │ • Push Notif.   │
│ • Mini-app   │  │ • QR Kodovi │  │ • SMS           │
│ • Webhook-i  │  │ • Praćenje  │  │ • Email         │
│              │  │   Gotovine  │  │                 │
└──────────────┘  └─────────────┘  └─────────────────┘
         │               │                   │
         └───────────────┼───────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                        SLOJ PODATAKA                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  MongoDB (Primarna Baza Podataka)                       │   │
│  │  • Kolekcija igrača (identitet, uređaji, biometrija)   │   │
│  │  • Kolekcija isključenja (status, istorija)            │   │
│  │  • Kolekcija limita (trenutni, promene na čekanju)     │   │
│  │  • Kolekcija transakcija (čuvanje 5 godina)            │   │
│  │  Šifrovanje: AES-256 u mirovanju                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Redis Keš (Sloj Performansi)                          │   │
│  │  • Stanje sesije (TTL: 4 sata)                         │   │
│  │  • Trenutni limiti (TTL: 1 sat)                        │   │
│  │  • Status isključenja (TTL: 1 dan, poništava pri prm.)│   │
│  │  • Tokeni uređaja (push notifikacije)                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Elasticsearch (Analitika i Revizija)                  │   │
│  │  • Svi API pozivi logirani                             │   │
│  │  • Čuvanje 7 godina radi usklađenosti                  │   │
│  │  • SIEM integracija u realnom vremenu                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Azure Blob Storage                                     │   │
│  │  • KYC dokumenti (fotografije LK, itd.)                │   │
│  │  • Šifrovano ključevima koje upravlja kupac            │   │
│  │  • Geo-replikacija (Srbija + rezervni region)         │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              INTEGRACIJA OPERATORA I REGULATORA                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────┐         ┌──────────────────────┐       │
│  │   Licencirani      │         │ Uprava za igre       │       │
│  │   Operatori        │         │ na sreću             │       │
│  │                    │         │                      │       │
│  │ • Primaju webhook  │◄────────┤ • Dashboard u        │       │
│  │ • SDK mini-app     │         │   realnom vremenu    │       │
│  │ • Mogu služiti samo│         │ • Mesečni izveštaji  │       │
│  │   verifikovane igr.│         │ • Pristup reviziji   │       │
│  └────────────────────┘         └──────────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

### Tehnička Specifikacija Mobilne Aplikacije

```yaml
Platforma: Nativni iOS i Android (NE hibridni/web)
Razlog: Potreban nativan pristup biometriji, sigurnom skladištu, ID-ovima uređaja

iOS:
  Minimalna Verzija: iOS 15+ (Face ID/Touch ID)
  Ključne Mogućnosti:
    - Keychain za skladištenje osetљivih podataka
    - Secure Enclave za biometrijske podatke
    - IdentifierForVendor za vezivanje uređaja
    - Prikačinjavanje Sertifikata (TrustKit biblioteka)
    - Detekcija Jailbreak-a (IOSSecuritySuite)

Android:
  Minimalna Verzija: Android 10+ (API 29)
  Ključne Mogućnosti:
    - Keystore za osetљive podatke
    - BiometricPrompt API
    - Android ID za vezivanje uređaja
    - Prikačinjavanje Sertifikata (OkHttp CertificatePinner)
    - Detekcija Root-a (RootBeer biblioteka)

Bezbednosne Mere:
  - Nema mogućnosti snimanja ekrana (FLAG_SECURE)
  - Snimanje ekrana nije dozvoljeno
  - Obfuskacija (ProGuard/R8)
  - Anti-tampering (validacija kontrolne sume)
  - SSL Prikačinjavanje sa rezervnim pinovima
  - Provere integriteta aplikacije u toku izvršavanja

Ciljna Veličina Aplikacije: < 50 MB (za preuzimanje preko 3G)

Mogućnosti Van Mreže:
  - Pregledaj status isključenja (keširan)
  - Pregledaj istoriju transakcija (poslednjih 30 dana keširano)
  - Ne može kockati van mreže (zahteva verifikaciju u realnom vremenu)
```

### Backend Tehnološki Stek

```yaml
API Sloj:
  Okvir: ASP.NET Core 8 (C#)
  Hosting: Azure Kubernetes Service (AKS)
  Skaliranje: Horizontalno auto-skaliranje (10-100 pod-ova)

Baza Podataka:
  Primarna: MongoDB Atlas (minimalno M50 tier)
    - Ključ za shard-ovanje: IgracID
    - 3-čvorni replica set
    - Automatski failover
  Keš: Redis Enterprise (Active-Active geo-replikacija)
  Pretraga: Elasticsearch 8.x

Red Poruka:
  Azure Service Bus (Premium tier)
    - Topics: dogadjaji_igraca, transakcije, intervencije
    - Pretplate po operatoru
    - Dead-letter redovi za neuspehe

AI/ML:
  Platforma: Azure Machine Learning
  Model: XGBoost (isti kao španski algoritam)
  Podaci za Trening: Anonimizovano ponašanje od 100K+ igrača
  Ponovno Treniranje: Mesečno sa novim podacima
  Inferencija: Realno vreme (< 50ms po predviđanju)

Praćenje:
  APM: Application Insights
  Logiranje: Azure Monitor + Elasticsearch
  Alarmi: PagerDuty integracija
  Cilj Dostupnosti: 99.95% (maksimalno 4.38 sati nefunkcionisanja/godišnje)
```

---

## DEO 5: Pravne i Političke Implikacije

### Ustavni Izazovi

**Pitanje 1: Da li obavezna aplikacija krši slobodu izbora?**

**Odgovor:**
- Precedenti u EU:
  - Danska: MitID je obavezan za sve onlajn državne usluge
  - Italija: SPID obavezan za samoisključenje
  - Estonija: e-Rezidenstvo digitalna obaveza

- Pravno opravdanje:
  - **Javni zdravstveni interes** (sprečavanje zavisnosti od kockanja)
  - **Zaštita potrošača** (sprečavanje prevare, kockanja maloletnika)
  - **AML usklađenost** (Član 58 Srpskog zakona o AML)
  - **Test proporcionalnosti:** Najmanje restriktivna mera koja postiže cilj

**Sudska odbrana:**
> "Obavezna aplikacija ne zabranjuje kockanje, već osigurava da se kockanje dešava uz odgovarajuće zaštite. Analogno obaveznim sigurnosnim pojasevima u automobilima - ne zabranjuje vožnju, već štiti učesnike."

---

**Pitanje 2: GDPR - Da li centralizovana biometrija krši privatnost?**

**GDPR Član 9:** Biometrijski podaci su "posebna kategorija" - zabranjeni osim ako...
- **(a) Izričita saglasnost** - Igrač daje saglasnost pri registraciji
- **(g) Znatni javni interes** - Sprečavanje zavisnosti od kockanja
- **(f) Pravni zahtevi** - Odbrana zakonih prava

**Implementirane zaštite:**
- Biometrijski šabloni skladišteni lokalno na uređaju (Secure Enclave/Keystore)
- Centralna baza podataka ima samo heš šablona (ne sam šablon)
- Ne može reverse-engineer-ovati lice iz heša
- Minimizacija podataka: Skladišti se samo što je neophodno

**DPIA (Procena Uticaja na Zaštitu Podataka) obavezna:**
- Visokorazično procesiranje zbog biometrije + bihevioralnog profiliranja
- Dokumentovane mere ublažavanja
- Konsultacija sa srpskim Poveren om pre lansiranja

---

**Pitanje 3: Anti-trust - Da li je monopol na aplikaciju antikonkurentan?**

**Argument:**
- Ovo nije monopol na **usluge kockanja** (operatori se slobodno takmiče)
- Ovo je monopol na **infrastrukturu za verifikaciju identiteta** (kao što je MitID u Danskoj)
- Analogija: Samo jedna nacionalna služba za kontrolu letenja, ali mnogo aviokompanije

**Prednosti za operatore:**
- Ne moraju graditi svoje KYC sisteme (ušteda troškova)
- Ne snose odgovornost za neuspehe u verifikaciji godina
- Automatski dobijaju pristup verifiko vanoj bazi igrača

**EU Zakon o Konkurenciji:**
- Član 106 TFEU dozvoljava monopole za "usluge od opšteg ekonomskog interesa"
- Zaštita kockanja može se klasifikovati kao SGEI

---

### Zahtevi za Usklađenost Operatora

**Obavezne izmene za operatore:**

```yaml
Tehnička Integracija:
  - Ukloniti samostalne aplikacije iz App Store / Google Play
  - Razviti mini-aplikaciju za KockaID platformu
  - Integrisati KockaID SDK
  - Implementirati webhook slušaoce za:
      - Obaveštenja o samoisključenju
      - Promene limita
      - Suspenzije igrača
  - API SLA: 99.9% dostupnost, < 200ms vreme odgovora

Poslovne Promene:
  - Ne mogu direktno primati igrače (moraju dolaziti preko KockaID)
  - Ne mogu nuditi bonuse koji zaobilaze limite
  - Ne mogu kreirati VIP programe bez odobrenja regulatora
  - Moraju doprinositi operativnim troškovima KockaID (model naknade TBD)

Kazne za Neusklađenost:
  - Upozorenje (prvi prekršaj)
  - Novčana kazna: €10,000 - €500,000 (ponovљeno)
  - Suspenzija licence (ozbiljna kršenja)
  - Opoziv licence (namerno zaobilaženje)
```

---

### Faze Implementacije i Vremenski Okvir

**FAZA 1: Pilot (Meseci 1-6)**

**Obim:**
- 1,000 dobrovoljnih korisnika (beta testeri)
- 3 licencirana operatora (dobrovoljci)
- Samo onlajn kockanje (još nema fizičkih kazina)

**Rezultati:**
- iOS + Android aplikacije (MVP)
- Osnovni KYC (skeniranje dokumenta + podudaranje lica)
- Sprovođenje samoisključenja
- Limiti depozita (pojedinačni operator)
- Backend infrastruktura (razvoj + staging)

**Kriterijumi Uspeha:**
- < 5% stopa tehničkih neuspeha
- > 80% zadovoljstvo korisnika
- 0 bezbednosnih incidenata
- Regulatorno odobrenje za sledeću fazu

---

**FAZA 2: Obavezno Onlajn (Meseci 7-12)**

**Obim:**
- SVI onlajn operatori obavezni
- Procenjeno 50,000-100,000 igrača
- Aktivno sprovođenje limita između operatora
- Raspoređeno AI ocenjivanje rizika

**Ključne Promene:**
- Migracija postojećih igrača (period prilagođavanja od 6 meseci)
- Stare aplikacije operatora zastarele (preusmerenje ka KockaID)
- Prolaz plaćanja puš ten u rad
- Omogućena detekcija VPN-a

**Rok za Usklađenost:**
- Operatori moraju integrisati do 9. meseca ili suočiti se sa kaznama
- Nakon 12. meseca: Rad bez KockaID integracije = nezakonit

---

**FAZA 3: Integracija Fizičkih Kazina (Meseci 13-18)**

**Obim:**
- Prijava QR kodom u svim kazinima
- Praćenje gotovine između objekata
- Integracija video nadzora
- Sprovođenje samoisključenja za fizičke objekte

**Izazovi:**
- Starija demografija nije tehnološki pismena (potrebni kioski + pomoć osoblja)
- Kultura bazirana na gotovini (otpor digitalnim novčanicima)
- Manjim objektima može nedostajati infrastruktura

**Rešenje:**
- Kioski na ulazu kazina za korisnike bez pametnih telefona (štampanje privremenog QR koda)
- Program obuke osoblja (1 nedelja)
- Subvencije za hardver za male operatore

---

**FAZA 4: Regionalna Ekspanzija (Meseci 19-24)**

**Obim:**
- Bilateralni sporazumi sa susednim zemljama (Hrvatska, Slovenija, Bosna)
- Verifikacija prekograničnih sertifikata
- Dozvole za kockanje za turiste

**Potrebne Pregovori:**
- Sporazumi o deljenju podataka (GDPR usaglašeni)
- Uzajamno priznavanje isključenja
- Standardizovani API protokoli

**Precedent:**
- Nordijske zemlje razmatraju zajedničko samoisključenje (Švedska, Finska, Danska)
- Srpsko-hrvatska saradnja u kockanju mogla bi biti pilot za Balkan

---

## DEO 6: Analiza Troškova i Koristi

### Troškovi Razvoja i Rada

**Jednokratni Razvoj (Godina 1):**

| Stavka | Trošak (EUR) |
|--------|--------------|
| Razvoj mobilne aplikacije (iOS + Android) | €400,000 |
| Postavljanje backend infrastrukture | €200,000 |
| Razvoj AI/ML modela | €150,000 |
| Bezbednosna revizija i testiranje penetracije | €50,000 |
| Pravno i savetovanje o usklađenosti | €80,000 |
| Marketinška kampanja i edukacija korisnika | €100,000 |
| Podrška integraciji operatora | €70,000 |
| **Ukupno Godina 1** | **€1,050,000** |

**Godišnji Operativni Troškovi (Stabilno Stanje, 500K korisnika):**

| Stavka | Trošak (EUR/godišnje) |
|--------|------------------------|
| Cloud infrastruktura (Azure) | €300,000 |
| Osoblje (15 osoba: programeri, ops, podrška, usklađenost) | €900,000 |
| Održavanje AI/ML modela | €50,000 |
| Tekuće bezbednosne revizije | €40,000 |
| Korisnička podrška (24/7) | €200,000 |
| **Ukupno Operativno** | **€1,490,000/godišnje** |

---

### Opcije Modela Finansiranja

**Opcija 1: Naknada za Licencu Operatora**
- Operatori plaćaju €0.10 po aktivnom igraču mesečno
- 500,000 aktivnih igrača × €0.10 = €50,000/mesec = €600,000/godišnje
- Manjak: €890,000 → potrebna državna subvencija

**Opcija 2: Naknada za Transakciju**
- €0.01 po transakciji (depozit/povlačenje)
- Pretpostavljajući 20 transakcija po igraču mesečno
- 500K igrača × 20 × €0.01 = €100,000/mesec = €1.2M/godišnje
- Pokriva operativne troškove ✓

**Opcija 3: Naknada na Prihod od Kockanja (UK model)**
- 1% od bruto prihoda od kockanja
- Ako je srpsko tržište €500M BPK/godišnje → €5M/godišnje
- Više nego pokriva troškove, višak finansira programe lečenja ✓

**Preporuka: Opcija 3** (Naknada na Prihod)
- Usklađeno sa principom "zagađivač plaća"
- Skalabilno sa rastom tržišta
- Precedent u Velikoj Britaniji (statutarna naknada april 2026)

---

### Društvene Koristi (Kvantifikovane)

**Smanjenje Problematičnog Kockanja:**

Podaci iz istraživanja:
- Holandija je videla **60% smanjenje** igrača koji premašuju limite nakon obaveznih limita
- Švedski Spelpaus: **80% izveštava o smanjenim željama**
- UK GAMSTOP: **44% smanjenje** štete od kockanja među samoisključenima

**Konzervativna procena za Srbiju:**
- Trenutni problematični kockari: ~2% od 2M kockara = 40,000 osoba
- Sa KockaID obaveznom aplikacijom: **30% smanjenje** = 12,000 manje problematičnih kockara
- Trošak problematičnog kockanja po osobi: ~€10,000/godišnje (zdravstvo, izgubljena produktivnost, kriminal)
- **Godišnje uštede: 12,000 × €10,000 = €120 miliona**

**Dodatne Koristi:**
- Smanjeno kockanje maloletnika (trenutno procenjeno 5-10% pristup): **€20M ušteda**
- Smanjeno pranje novca (bolji KYC): **€50M sprečeno**
- Povećani poreski prihodi (kanalisanje sa crnog tržišta): **€200M+**

**Ukupna Godišnja Korist: €390 miliona**

**ROI:**
- Trošak: €1.5M/godišnje
- Korist: €390M/godišnje
- **Prinos: 260x**

---

## DEO 7: Ključni Faktori Uspeha

### Zahtevi za Političku Volju

**Vlada mora:**
1. **Posvetiti se sprovođenju** - Operatori crnog tržišta suočavaju se sa pravim posledicama (blokiranje plaćanja, ISP blokiranje, krivična gonjenja)
2. **Odoleti lobby-ju industrije** - Operatori će se žaliti na "preteranu regulaciju"
3. **Adekvatno finansirati** - Ne mogu očekivati vrhunski sistem sa minimalnim budžetom
4. **Javna edukacija** - Građani moraju razumeti koristi, a ne videti to kao "Veliki Brat"

**Istorijski precedent:**
- Holandija je suočena sa masivnim pritiskom industrije na obavezne limite
- Rezultat: 10% pad prihoda za operatore u 1. godini
- ALI: Javna podrška ostala visoka (68% odobrenje)
- Vlada nije popustila

---

### Strategija Usvajanja Korisnika

**Izazov:** Dovesti 500K+ postojećih igrača da migriraju na novi sistem

**Šargarepa (Podsticaji):**
- Rani usvajači dobijaju €10 besplatnog kredita (finansirano naknadom)
- Igrifikacija: Značke "Odgovornog Igrača", nagrade za postavljanje limita
- Ekskluzivne promocije dostupne samo preko KockaID

**Batina (Sprovođenje):**
- Nakon 12. meseca: Ne može kockati bez KockaID (stari nalozi onemogućeni)
- Operatori suočeni sa kaznama za usluge za igrače bez KockaID
- Javno slanje poruka: "Zaštitite sebe. Verifikujte se."

**Pristupačnost:**
- Aplikacija dostupna na srpskom, engleskom, romskom (manjinski jezik)
- Telefonska podrška za starije (mogu se registrovati telefonom, primaju SMS kodove)
- Fizički centri za registraciju u većim gradovima za korisnike bez pametnih telefona

---

### Tehnička Pouzdanost

**Kritično:**
- Ako aplikacija padne, kockanje se zaustavlja u celoj zemlji
- Mora postići **99.95% dostupnost** minimum

**Mere redundanse:**
- Multi-regionalno active-active raspoređivanje (Srbija + EU rezerva)
- Automatski failover (< 2 minuta)
- Lokalno keširanje na uređaju (može verifikovati status isključenja van mreže 24 sata)
- Operatorski režim rezerve (ako centralni sistem ne radi > 15 minuta, operatori mogu uslužiti igrače sa poslednje poznatim statusom, ali loguju sve transakcije za naknadnu verifikaciju)

---

## DEO 8: Poređenje sa Postojećim Sistemima

### Danska MitID + ROFUS vs KockaID

| Mogućnost | Danska | KockaID (Predloženo) |
|-----------|--------|----------------------|
| **Obaveznost** | Da (sve online usluge) | Da (samo kockanje) |
| **Vezivanje Uređaja** | Ne (može prijaviti sa bilo kog uređaja) | **Da** (maks 2 uređaja) |
| **Biometrija** | Opciono (lozinka glavna) | **Obavezna** (primarna autentifikacija) |
| **Integracija Plaćanja** | Ne (operatori imaju svoje) | **Da** (centralni prolaz) |
| **Detekcija VPN** | Ne | **Da** (višeslojno) |
| **Fizički Kazino** | Odvojena kartica igrača | **Integrisano** (QR kod) |
| **AI Ocenjivanje Rizika** | Ne (na nivou operatora) | **Da** (centralizovano) |
| **Prekogranično** | Ograničeno (samo nordijske zemlje neformalno) | **Da** (bilateralni sertifikati) |

**Prednost KockaID:** Vezivanje uređaja + prolaz plaćanja = jače sprovođenje

**Prednost MitID:** Univerzalno (ne samo kockanje) = veće usvajanje

---

### Nemačka OASIS + LUGAS vs KockaID

| Mogućnost | Nemačka | KockaID (Predloženo) |
|-----------|---------|----------------------|
| **Samoisključenje** | OASIS (odlično, 320K korisnika) | KockaID (slično) |
| **Limiti Depozita** | LUGAS (€1K/mesec između-op) | KockaID (podesivo) |
| **Crno Tržište** | **54% prihoda** (glavni problem) | KockaID cilja < 10% |
| **Sprovođenje** | Slabo (ISP blokiranje oboreno) | **Jako** (na nivou aplikacije) |
| **Više Naloga** | Moguće (ID baziran na dokumentu) | **Sprečeno** (vezivanje uređaja) |
| **Integracija Operatora** | Opcioni API-ji | **Obavezne** mini-aplikacije |

**Ključni Nemački Problem:** Tržište toliko restriktivno (nema stolne igre, €1/spin slot limit) da igrači beže u inostranstvo

**KockaID Strategija:** Balansirati zaštitu sa razumnim limitima radi održavanja kanalisanja

---

## ZAKLJUČAK: Transformacija Paradigme Sprovođenja

### Trenutni Sistemi: "Veruj Ali Ne Proveravaj"

- Operatori **kažu** da proveravaju samoisključenje → ali ne mogu sprečiti igrača da ode na drugi sajt
- Limiti **postoje** → ali se zaobilaze preko više naloga
- Biometrija **se koristi** → ali na nivou operatora, ne centralno
- **Rezultat:** 54% crno tržište (Nemačka), 38-49% samoisključeni nastavljaju (Švedska)

---

### KockaID Model: "Sprovodи Pre, a Ne Posle"

**Filozofija:**
> "Ne oslanjaj se na usklađenost operatora. Učini zaobilaženje tehnički nemogućim."

**Mehanizmi:**
1. **Vezivanje uređaja** → Ne može 10 telefona = 10 naloga
2. **Pristup samo preko aplikacije** → Ne može direktno na sajt operatora
3. **Centralni prolaz plaćanja** → Nema anonimnih depozita
4. **Limiti u realnom vremenu** → Sprovođenje između operatora u milisekundama
5. **Detekcija VPN** → Ne može zaobići geografsko blokiranje
6. **Obavezna biometrija** → Ne može pozajmiti nalog bratu

**Predviđeni Rezultati (Godina 3):**
- Prihod crnog tržišta: **< 10%** (pad sa procenjenih trenutnih ~30%)
- Stopa kršenja samoisključenja: **< 2%** (pad sa 38-49% EU prosek)
- Usvajanje alata: **100%** (obavezno angažovanje)
- Incidenca problematičnog kockanja: **-30%** (12,000 manje slučajeva)
- ROI: **260x** (€390M korist vs €1.5M trošak)

---

### Politički Imperativ

**Ovo nije tehnički projekat. Ovo je politički projekat.**

Uspeh zavisi od:
1. **Vođstvo** - Ministar mora javno podržati, braniti od lobija
2. **Zakonodavstvo** - Skupština mora usvojiti obaveznost
3. **Budžet** - Vlada mora finansirati (ne može se očekivati da se samo finansira)
4. **Sprovođenje** - Policija/tužilaštvo mora goniti operatore crnog tržišta
5. **Javnost** - Građani moraju videti korist, ne "nadzor"

**Precedent za uspeh:**
- Estonija: Transformacija e-Vlade (99% usluga onlajn)
- Danska: MitID univerzalno usvajanje (95%+ građana)
- Velika Britanija: GAMSTOP obavezna integracija (100% licenciranih operatora)

**Srbija može postati model za region.**

Ako uspe, Hrvatska, Slovenija, Bosna će kopirati sistem.
Ako ne uspe, status quo nastavlja: fragmentisano sprovođenje, dominacija crnog tržišta, ograničena zaštita igrača.

**Vreme je za odluku.**
