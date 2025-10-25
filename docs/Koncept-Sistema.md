# Koncept Sistema za Zaštitu Igrača

## Pregled

Integrisana državna platforma za zaštitu igrača i odgovorno kockanje koja kombinuje najbolje prakse iz Evrope (UK GAMSTOP, Swedish Spelpaus, Danish ROFUS, Italian ADM) u jedinstveno rešenje za real-time monitoring i intervenciju.

---

## 1. Centralna Državna Platforma

### Arhitektura

- **Real-time agregacija podataka** od svih licenciranih priređivača
- **Standardizovani API** za obaveznu integraciju (REST/GraphQL + WebSocket)
- **Event-driven arhitektura** za trenutnu sinhronizaciju podataka

### Obavezna Integracija Priređivača

| Tip Priređivača | Obavezni Podaci | Frekvencija |
|-----------------|-----------------|-------------|
| Online platforme | Svaka opklada, depozit, isplata | Real-time |
| Fizičke kladionice | Transakcije preko 100 EUR | Real-time |
| Kazino | Ulazi, vreme provedeno, iznosi | Real-time |

### KYC Provjera Pre Početka

```
┌─────────────┐
│   Igrač     │
└──────┬──────┘
       │
       ├─ Lična karta / Pasoš
       ├─ Biometrijska verifikacija
       ├─ Adresa prebivališta
       └─ Jedinstveni ID broj (JMBG/PIB)
       │
       ▼
┌──────────────────┐
│  Centralni Reg.  │ ──► Verifikacija u 24h
└──────────────────┘
```

**Sprečavanje pristupa** do završetka verifikacije - **NIjedan priređivač ne može aktivirati nalog** bez potvrde iz centralnog registra.

---

## 2. Jedinstveni Registar Igrača

### Struktura Podataka

```csharp
public class PlayerRecord
{
    public string PlayerID { get; set; }           // SHA-256 hash JMBG
    public VerifiedIdentity Identity { get; set; } // Šifrovani lični podaci
    public List<Transaction> GameHistory { get; set; }
    public RestrictionStatus Status { get; set; }
    public DateTime LastActivity { get; set; }
    public RiskScore CurrentRisk { get; set; }
}
```

### Automatska Sinhronizacija

- **Samoisključenje na jednoj platformi** = instant blokada na svim platformama
- **Kršenje limita** = automatska suspenzija dok se ne razreši
- **Real-time status check** prilikom svakog pokušaja logina

### Cross-Platform Pokrivenost

```
Igrač samoisključen na Platforma_A
         │
         ├──► API Event: SelfExclusion(PlayerID, Duration)
         │
         ▼
    Centralni Hub
         │
         ├──► Push to: Platforma_B, C, D, E...
         │
         ▼
    Blokada u < 2 sekunde
```

---

## 3. Modul za Odgovorno Igranje

### Inspiracija: UK GAMSTOP + Swedish Spelpaus

#### A. Samoisključenje

| Tip | Trajanje | Povratak |
|-----|----------|----------|
| **Privremeno** | 6 meseci, 1 godina, 2 godine | Automatski nakon perioda |
| **Trajno** | Neograničeno | Samo uz stručni nalaz i cooling-off |
| **Hitno** | 24h - 7 dana | Za trenutne krizne situacije |

**Implementacija:**
```csharp
public enum ExclusionType
{
    Temporary_6M,
    Temporary_1Y,
    Temporary_2Y,
    Permanent,
    Emergency_24H,
    Emergency_7D
}

public void ApplySelfExclusion(string playerID, ExclusionType type)
{
    // 1. Zatvori sve aktivne sesije
    // 2. Blokiraj login na svim platformama
    // 3. Notifikuj sve priređivače
    // 4. Zakaži automatsko otključavanje (ako temporary)
}
```

#### B. Limiti Igranja

**Dnevni Limiti:**
- Maksimalni dnevni depozit (default: 50 EUR)
- Maksimalni dnevni gubitak (default: 100 EUR)
- Maksimalno vreme igre (default: 2 sata)

**Mesečni Limiti:**
- Ukupni depoziti (configurable)
- Neto gubitak (configurable)

**Izmena Limita:**
- Povećanje = 72h cooling-off period pre primene
- Smanjenje = trenutna primena

#### C. Cooling-Off Periodi

Automatski триггери:
- 3 uzastopna depozita u roku od 1h
- Gubitak > 50% mesečnog prihoda (ako je deklarisan)
- 5+ sati kontinuiranog igranja
- "Chasing losses" patern (ML detekcija)

**Trajanje:** 24h - 7 dana (zavisno od ozbiljnosti)

#### D. Kontrola Reklamnih Poruka

```json
{
  "player_preferences": {
    "email_marketing": false,
    "sms_promotions": false,
    "push_notifications": false,
    "targeted_ads": false
  }
}
```

Igrači u samoisključenju **automatski** isključeni iz svih marketinških kanala.

---

## 4. Analitički Sloj (ML-Based Risk Detection)

### Model: Danish ROFUS + Italian ADM

#### Algoritmi za Detekciju Rizika

**Parametri za Analizu:**
1. **Frekvencija igranja** (dnevna, nedeljne promene)
2. **Veličina opklada** (eskalacija, impulsivnost)
3. **Win/Loss paterni** (chasing, desperation bets)
4. **Vreme igranja** (kasno noću, tokom radnog vremena)
5. **Depoziti** (broj, iznosi, metode plaćanja)
6. **Povrh limita pokušaji** (koliko puta igrač pokušava preći limit)

#### Risk Scoring (0-100)

```
0-30:  Low Risk (zelena zona)
31-60: Medium Risk (žuta zona - pojačan monitoring)
61-85: High Risk (narandžasta - obavezna intervencija)
86+:   Critical Risk (crvena - automatska suspenzija)
```

#### Machine Learning Model

```python
# Pseudokod
features = [
    'deposit_frequency',
    'bet_size_variance',
    'loss_chasing_score',
    'session_duration',
    'time_of_day_pattern',
    'limit_breach_attempts',
    'deposit_method_changes'
]

risk_score = RandomForestClassifier.predict(features)

if risk_score >= 86:
    trigger_emergency_intervention()
elif risk_score >= 61:
    schedule_mandatory_contact()
elif risk_score >= 31:
    enable_enhanced_monitoring()
```

#### Automatske Intervencije

| Risk Level | Akcija | Timeout |
|------------|--------|---------|
| High (61+) | Email + SMS upozorenje | Odmah |
| High (70+) | Obavezan poziv/chat sa savetnikom | 24h |
| Critical (86+) | Automatska suspenzija naloga | Odmah |
| Critical (86+) | Notifikacija porodice (uz saglasnost) | 48h |

---

## 5. Sigurnost i Privatnost

### GDPR Usklađenost

#### Podaci Koji Se Prikupljaju

**Neophodne informacije:**
- Identitet (KYC)
- Transakcijska istorija (zakonska obaveza)
- Aktivnost igranja (za zaštitu)

**Pseudonimizacija:**
```csharp
string playerID = SHA256(JMBG + SALT);
// Pravi identitet poznat samo centralnom registru
// Priređivači dobijaju samo playerID
```

#### Šifrovanje

- **At Rest:** AES-256 šifrovanje MongoDB baze
- **In Transit:** TLS 1.3 za sve API komunikacije
- **End-to-End:** Kritični podaci (lični detalji) duplo šifrovani

#### Kontrola Pristupa

```
┌──────────────┐
│  Priređivač  │ ──► Vidi samo: playerID, status, limitе
├──────────────┤
│  Regulator   │ ──► Vidi: agregiranu statistiku, anonimne trендove
├──────────────┤
│  Centralni   │ ──► Pun pristup (uz audit log svake radnje)
│  Administrator│
└──────────────┘
```

#### Audit Trail

Svaka akcija loguje:
```json
{
  "timestamp": "2025-10-25T14:32:11Z",
  "user": "admin_john_doe",
  "action": "VIEW_PLAYER_DETAILS",
  "playerID": "hash_xxx",
  "ip_address": "192.168.1.1",
  "justification": "Regulatory inspection #12345"
}
```

### Revizijski Protokoli

- **Dnevni backup** svih transakcija
- **Immutable logs** (append-only, blockchain-inspired)
- **Nezavisni auditi** kvartalno
- **Penetration testing** 2x godišnje

---

## 6. Regulatorni Nadzor i Dashboardi

### KPI Dashboard za Regulatore

#### Glavni Indikatori

**Zaštita Igrača:**
```
┌─────────────────────────────────────┐
│ Samoisključenja (Mesečno)           │
│ ████████████░░░░░░░░  1,247 (+12%)  │
├─────────────────────────────────────┤
│ Aktivne Suspenzije                  │
│ ██████░░░░░░░░░░░░░░    423         │
├─────────────────────────────────────┤
│ High-Risk Igrači                    │
│ █████████████░░░░░░░    892 (-5%)   │
└─────────────────────────────────────┘
```

**Kršenja i Anomalije:**
- Priređivači koji kasne sa izvještavanjem
- Sumnjive transakcije (pranje novca)
- Pokušaji zaobilaženja limita
- Neprijavljeni VIP programi

#### Real-Time Alarms

```csharp
public class RegulatoryAlert
{
    public AlertType Type { get; set; } // Limit breach, Late reporting, AML flag
    public string OperatorID { get; set; }
    public string PlayerID { get; set; }
    public decimal Amount { get; set; }
    public string Description { get; set; }
    public DateTime Timestamp { get; set; }
}
```

**Auto-eskalacija:**
1. Warning → Operator (15 min)
2. Violation → Senior Inspector (1h)
3. Critical → Emergency Team + Legal (instant)

### Ciljane Inspekcije

**Trigger Uslovi za Inspekciju:**
- Više od 5% igrača u high-risk zoni
- Povećanje samoisključenja > 20% mesečno
- Priređivač ne postuje API SLA (>1% downtime)
- Sumnja na manipulaciju podataka

### Preventivne Kampanje

**Data-Driven Kampanje:**
```
ML Model detektuje:
└─► Trend: Porast igranja kod mladih (18-24)
    └─► Auto-generisanje kampanje:
        ├─ Cilj: Edukacija o rizicima
        ├─ Kanali: Instagram, TikTok, SMS
        └─ Metrike: Reach, engagement, promene u ponašanju
```

**Segmentacija:**
- Preventivne (low/medium risk)
- Intervencijske (high risk)
- Post-rehabilitacione (nakon samoisključenja)

---

## Implementaciona Roadmap

### Faza 1: Foundation (Meseci 1-4)
- Centralna baza i API
- KYC integracija
- Osnovni registar igrača

### Faza 2: Compliance (Meseci 5-8)
- Obavezna integracija priređivača
- Samoisključenje i limiti
- Regulatorni dashboard

### Faza 3: Intelligence (Meseci 9-12)
- ML model za rizik
- Automatske intervencije
- Prediktivna analitika

### Faza 4: Optimization (Mesec 13+)
- AI-powered personalizacija
- Cross-border saradnja
- Blockchain audit trail

---

## Tehnički Stack

**Backend:**
- C# / .NET 8 (Web API)
- SignalR (real-time events)
- Hangfire (background jobs)

**Database:**
- MongoDB (player data, transakcije)
- Redis (caching, session management)
- ElasticSearch (logs, analytics)

**ML/AI:**
- ML.NET (risk scoring)
- Python microservices (advanced analytics)
- TensorFlow (pattern recognition)

**Frontend:**
- React (admin dashboard)
- Chart.js / D3.js (visualizacije)
- Material-UI

**Infrastructure:**
- Docker / Kubernetes
- Azure / AWS (cloud hosting)
- Kafka (event streaming)

---

## Zaključak

Ovaj sistem kombinuje:
- **Preventivnu zaštitu** (limiti, edukacija)
- **Aktivnu intervenciju** (ML detekcija, obavezni kontakti)
- **Regulatornu kontrolu** (transparentnost, auditi)

**Cilj:** Smanjenje problem-gambling stope za 40% u prve 2 godine uz održavanje tržišne kompetitivnosti priređivača.
