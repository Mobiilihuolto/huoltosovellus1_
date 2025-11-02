# 📦 Sovelluksen julkaisu- ja myyntiohjeet

Tämä dokumentti sisältää ohjeet sovelluksen julkaisemiseen ja myyntiin. Noudata näitä ohjeita varmistaaksesi että kehitysympäristösi säilyy turvassa ja myyntiversio on ammattimaisesti valmisteltu.

---

## 🔐 1. Kehitysversion säilyttäminen

**TÄRKEÄÄ:** Varmuuskopioi AINA datasi ennen kuin tyhjennät mitään!

### Vaihtoehto A: Varmuuskopioi Supabase-data (SUOSITELTU)

#### 1. Tietokannan varmuuskopiointi

**Manuaalinen varmuuskopiointi:**
1. Kirjaudu Supabase Dashboardiin:
   ```
   https://supabase.com/dashboard/project/ouuwdlpteiqbhclmqusk
   ```

2. Vie kaikki data SQL-tiedostoksi:
   - Siirry: **SQL Editor**
   - Aja seuraava komento:
   ```sql
   -- Tämä vie kaiken datan
   COPY (
     SELECT * FROM public.yrityksen_asetukset
   ) TO '/tmp/yrityksen_asetukset_backup.csv' WITH CSV HEADER;
   
   -- Toista jokaiselle taululle
   ```

3. **Tai käytä pg_dump -komentoa** (vaatii PostgreSQL-asennuksen):
   ```bash
   # Korvaa [PASSWORD] oikealla salasanalla
   PGPASSWORD=[PASSWORD] pg_dump \
     -h aws-0-eu-central-1.pooler.supabase.com \
     -p 6543 \
     -U postgres.ouuwdlpteiqbhclmqusk \
     -d postgres \
     --clean \
     --if-exists \
     > backup_$(date +%Y%m%d_%H%M%S).sql
   ```

**Automaattinen varmuuskopiointi (Supabase Pro):**
1. Settings → Database → Point-in-time Recovery (PITR)
2. Ota käyttöön automaattiset varmuuskopiot
3. Voit palauttaa mihin tahansa ajanhetkeen

#### 2. Storage-tiedostojen varmuuskopiointi

1. Mene: **Storage → company-logos**
2. Lataa kaikki logot paikallisesti:
   - Valitse kaikki tiedostot
   - "Download" → Tallenna turvalliseen kansioon

#### 3. Secrets ja ympäristömuuttujat

Tallenna nämä turvalliseen paikkaan (esim. salasananhallintaan):

1. **Edge Function Secrets:**
   - Settings → Edge Functions → Secrets
   - Kirjoita ylös:
     - `RESEND_API_KEY`
     - Muut API-avaimet

2. **Supabase-tunnukset:**
   - Project URL: `https://ouuwdlpteiqbhclmqusk.supabase.co`
   - Anon Key: (kopioi `.env` tiedostosta)
   - Service Role Key: (Settings → API)

---

### Vaihtoehto B: Luo uusi Supabase-projekti myyntiä varten (PARAS)

Tämä on suositeltavin tapa jos myyt sovellusta!

#### 1. Säilytä nykyinen projekti itsellesi

- **Nykyinen projektisi:** `ouuwdlpteiqbhclmqusk`
- Tämä jää sinun kehitysympäristöksesi
- Kaikki datasi pysyy turvassa täällä

#### 2. Luo uusi tyhjä Supabase-projekti

1. **Luo projekti:**
   - Mene: https://supabase.com/dashboard
   - Klikkaa: **New Project**
   - Nimeä: `[Tuotteen nimi] - Production Template`
   - Valitse: **EU Central (Frankfurt)** (tai muu alue)
   - Aseta vahva salasana

2. **Kopioi tietokantarakenne (EI dataa):**
   ```bash
   # Supabase CLI -menetelmä (suositeltu)
   cd [projektin-kansio]
   supabase db push --project-ref [uusi-projekti-id]
   ```

   **TAI manuaalisesti:**
   - Avaa SQL Editor uudessa projektissa
   - Aja kaikki migraatiot järjestyksessä:
     - `supabase/migrations/*.sql`
   - Tarkista että kaikki taulut ja RLS-politiikat ovat paikallaan

3. **Konfiguroi palvelut:**
   
   **Authentication:**
   - Settings → Authentication → Providers
   - Ota käyttöön: Email
   - Disable: "Confirm email" (nopeampi testaus)

   **Storage:**
   - Storage → Create new bucket
   - Nimi: `company-logos`
   - Public bucket: ✅ Yes

   **Edge Functions:**
   - Terminal:
     ```bash
     supabase functions deploy send-service-ready-email --project-ref [uusi-projekti-id]
     supabase functions deploy send-overdue-invoice-email --project-ref [uusi-projekti-id]
     supabase functions deploy send-low-stock-alert --project-ref [uusi-projekti-id]
     ```

4. **Lisää Edge Function Secrets:**
   - Settings → Edge Functions → Secrets
   - Lisää: `RESEND_API_KEY`, `SUPABASE_URL`, jne.

#### 3. Päivitä Lovable-projekti uudelle Supabase-projektille

1. **Remix Lovable-projekti:**
   - Lovable → Projektin nimi (vasen ylä) → Settings → **Remix this project**
   - Nimeä: `[Tuotteen nimi] - Sales Version`

2. **Konfiguroi uusi Supabase-projekti:**
   - Mene remixattuun projektiin
   - Projektin asetukset → Supabase
   - Yhdistä uuteen Supabase-projektiisi
   - Syötä uuden projektin URL ja Anon Key

3. **Testaa toiminnallisuus:**
   - Rekisteröi testikäyttäjä
   - Testaa kirjautuminen
   - Tarkista että admin-oikeudet toimivat
   - Testaa CRUD-toiminnot

---

## 🧹 2. Datan tyhjentäminen (jos myyt nykyisellä tietokannalla)

**⚠️ VAROITUS:** Tee varmuuskopio ensin! (ks. kohta 1)

### Tyhjennä taulut (säilyttää rakenteen)

Aja nämä SQL-komennot **Supabase SQL Editorissa**:

```sql
-- VAROITUS: Tämä poistaa KAIKEN datan!
-- Varmista että olet tehnyt varmuuskopion!

-- 1. Poista huolto- ja laskudata (järjestys tärkeä!):
DELETE FROM public.huolto_varaosat;
DELETE FROM public.laskut;
DELETE FROM public."Huollot";

-- 2. Poista laitteet ja asiakkaat:
DELETE FROM public."Laitteet";
DELETE FROM public.asiakkaat;

-- 3. Poista varaosat:
DELETE FROM public.varaosat;

-- 4. Poista käyttäjät ja roolit:
DELETE FROM public.user_roles;
DELETE FROM public.profiles;

-- 5. Nollaa numerointiasetukset:
UPDATE public.numerointi_asetukset 
SET seuraava_numero = 1, 
    updated_at = now();

-- 6. Tyhjennä tekniikat:
DELETE FROM public.tekniikat;

-- 7. (Valinnainen) Tyhjennä yrityksen asetukset:
DELETE FROM public.yrityksen_asetukset;
```

### Poista Storage-tiedostot

1. **Storage → company-logos**
2. Valitse kaikki tiedostot
3. **Delete**

### Poista käyttäjät Authista

1. **Authentication → Users**
2. Valitse kaikki käyttäjät
3. **Delete users**

### Säilytä tai tyhjennä asetustaulut?

**Nämä taulut voi joko säilyttää TAI tyhjentää:**

| Taulu | Kannattaako säilyttää? | Syy |
|-------|------------------------|-----|
| `hinnoittelu_asetukset` | ✅ Kyllä | Antaa ostajalle hyvät oletushinnat |
| `alv_asetukset` | ✅ Kyllä | Suomen ALV 25,5% on hyvä oletus |
| `maksutavat` | ✅ Kyllä | Käteinen, kortti, lasku ovat yleisiä |
| `laite_valmistajat` | ⚠️ Ehkä | Jos lisäsit yleisiä valmistajia (Apple, Samsung jne.) |
| `takuu_asetukset` | ✅ Kyllä | 12kk oletus on järkevä |
| `service_statuses` | ✅ Kyllä | Perusstatukset (odottaa, työn alla jne.) |
| `ilmoitus_asetukset` | ✅ Kyllä | Oletuspohjat ovat hyödyllisiä |
| `lasku_asetukset` | ✅ Kyllä | 14 päivän maksuehto on standardi |
| `numerointi_asetukset` | ✅ Kyllä | Mutta nollaa `seuraava_numero = 1` |
| `varasto_asetukset` | ✅ Kyllä | Oletusasetukset ovat järkeviä |
| `yrityksen_asetukset` | ❌ EI | Tämä sisältää SINUN yrityksesi tiedot! |

**Tyhjennä nämä jos haluat antaa TÄYSIN tyhjän sovelluksen:**
```sql
DELETE FROM public.hinnoittelu_asetukset;
DELETE FROM public.alv_asetukset;
DELETE FROM public.maksutavat;
DELETE FROM public.laite_valmistajat;
DELETE FROM public.takuu_asetukset;
DELETE FROM public.service_statuses;
DELETE FROM public.ilmoitus_asetukset;
DELETE FROM public.lasku_asetukset;
DELETE FROM public.varasto_asetukset;

-- Numerointi_asetukset kannattaa säilyttää mutta nollata:
UPDATE public.numerointi_asetukset SET seuraava_numero = 1;
```

---

## 📦 3. Myyntiversio - Tarkistuslista

### ✅ Mitä myyntiversiossa PITÄÄ olla:

- ✅ **Toimiva autentikointi**
  - Email/salasana kirjautuminen
  - Rekisteröityminen toimii
  - Ensimmäinen käyttäjä saa admin-oikeudet automaattisesti

- ✅ **Roolijärjestelmä**
  - Admin, Teknikko, Käyttäjä -roolit
  - Oikeudet toimivat (RLS-politiikat)

- ✅ **Tyhjä tietokanta**
  - Ei testidata
  - Vain rakenne ja optionaaliset oletusasetukset

- ✅ **Dokumentaatio**
  - `README.md` - Projektin kuvaus
  - `SETUP_FOR_BUYER.md` - Käyttöönotto-ohjeet ostajalle
  - Kommentit koodissa (suomeksi tai englanniksi)

- ✅ **Edge Functions toiminnassa**
  - `send-service-ready-email`
  - `send-overdue-invoice-email`
  - `send-low-stock-alert`

- ✅ **Testattavuus**
  - Ostaja voi heti luoda testihuollon
  - Kaikki CRUD-toiminnot toimivat
  - PDF-tulostus toimii

### ❌ Mitä myyntiversiossa EI SAA olla:

- ❌ **Testidata:**
  - Ei testihuoltoja
  - Ei testilaitteita
  - Ei testiasiakkaita
  - Ei testilaskuja

- ❌ **Henkilökohtaista dataa:**
  - Ei omia yritystietojasi
  - Ei omia asiakastietoja
  - Ei omia logojakaan

- ❌ **API-avaimia:**
  - Ostaja lisää oman Resend API keyn
  - Ostaja luo oman Supabase-projektin

- ❌ **Kehitystason koodia:**
  - Ei `console.log` debuggausta
  - Ei kommentoitua koodia
  - Ei TODO-kommentteja

---

## 🚀 4. Ostajan käyttöönotto-ohjeet

Katso: **[SETUP_FOR_BUYER.md](./SETUP_FOR_BUYER.md)**

Tämä tiedosto sisältää askel-askeleelta ohjeet ostajalle sovelluksen käyttöönotosta.

---

## 🔄 5. Datan palauttaminen (itsellesi)

Jos tyhjensit datasi ja haluat palauttaa sen:

### Vaihtoehto A: Palauta SQL-varmuuskopio

```bash
# Jos käytit pg_dump:
PGPASSWORD=[PASSWORD] psql \
  -h aws-0-eu-central-1.pooler.supabase.com \
  -p 6543 \
  -U postgres.ouuwdlpteiqbhclmqusk \
  -d postgres \
  < backup_20250101_120000.sql
```

**TAI Supabase Dashboardista:**
1. Settings → Database → Point-in-time Recovery
2. Valitse palautuspiste (jos PITR käytössä)
3. "Restore"

### Vaihtoehto B: Palauta CSV-tiedostot

```sql
-- Esimerkki: Palauta yrityksen asetukset
COPY public.yrityksen_asetukset
FROM '/tmp/yrityksen_asetukset_backup.csv'
CSV HEADER;

-- Toista jokaiselle taululle
```

### Palauta Storage-kuvat

1. **Storage → company-logos**
2. **Upload** → Valitse varmuuskopioidut logot
3. Lataa kaikki takaisin

### Palauta Secrets

1. **Edge Functions → Secrets**
2. Lisää kaikki avaimet takaisin:
   - `RESEND_API_KEY`
   - Muut tallennetut avaimet

---

## 📞 Tuki ja lisätiedot

### Supabase-dokumentaatio:
- Backups: https://supabase.com/docs/guides/platform/backups
- Migrations: https://supabase.com/docs/guides/cli/local-development
- Storage: https://supabase.com/docs/guides/storage

### Lovable-dokumentaatio:
- Publishing: https://docs.lovable.dev/
- GitHub integration: https://docs.lovable.dev/

---

**Tehty: $(date +%Y-%m-%d)**  
**Versio: 1.0**
