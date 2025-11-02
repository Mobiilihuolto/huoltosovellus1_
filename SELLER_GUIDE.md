# Myyjän ohje - Huoltokanta-sovelluksen myyminen

Tämä ohje on SINULLE (myyjänä/kehittäjänä). Ostaja saa oman ohjeensa (`SETUP_FOR_BUYER.md`).

---

## Mitä myyt?

Myyt **valmiin verkkosovelluksen**, joka:
- Toimii selaimessa (ei asennusta, ei .exe-tiedostoa)
- Ostaja hallinnoi omaa Supabase-tiliään ja dataa
- Ostaja maksaa omat Supabase-kulunsa (ilmainen tier riittää pienille/keskisuurille yrityksille)
- Et ole vastuussa ostajan kuluista tai ylläpidosta

---

## Mitä annat ostajalle?

**VAIN YKSI ASIA: Sovelluksen linkki!**

1. **Sovelluksen julkaisulinkki** (Lovablesta)
   - Esim: `https://huoltokanta.lovable.app`

2. **Lyhyet ohjeet** (valinnainen - voit kirjoittaa oman tai käyttää `SETUP_FOR_BUYER.md`)
   - Miten luodaan ilmainen Supabase-tili
   - Miten lisätään Resend API-avain
   - Miten rekisteröidytään sovellukseen

**SIINÄ KAIKKI!**

**Miksi näin yksinkertaista?**
- ✅ Sovellus on jo julkaistu ja valmis
- ✅ Migraatiot ajetaan automaattisesti kun ostaja yhdistää Supabasensa
- ✅ Edge Functions toimivat automaattisesti
- ✅ Ei ZIP-paketteja, ei tiedostojen kopiointia

---

## Valmistelu myyntiä varten (5 minuuttia!)

### Vaihe 1: Julkaise sovellus Lovablessa

1. Avaa projektisi Lovablessa
2. Klikkaa **"Publish"**-nappia (oikeassa yläkulmassa)
3. Kopioi julkaisulinkki
   - Esim: `https://huoltokanta.lovable.app`

**TÄRKEÄÄ:** Tämä linkki toimii kaikille ostajille automaattisesti!

### Vaihe 2: Valmis! 

**Siinä kaikki!** 

Ei tarvitse:
- ❌ Kopioida tiedostoja
- ❌ Luoda ZIP-paketteja
- ❌ Lähettää SQL-tiedostoja
- ❌ Lähettää Edge Functions -koodia

**Miksi?** Koska:
- ✅ Migraatiot ovat sovelluksessa valmiina (automaattinen ajo)
- ✅ Edge Functions ovat sovelluksessa valmiina (automaattinen aktivointi)
- ✅ Ostaja yhdistää vain oman Supabasensa → kaikki toimii

---

## Mitä lähetät ostajalle?

**Sähköposti ostajalle:**

```
Hei [Ostajan nimi],

Kiitos ostoksesta! Tässä on Huoltokanta-sovelluksesi.

🔗 Sovelluksen linkki: https://huoltokanta.lovable.app

📖 Käyttöönotto (nopea ja helppo):

1. Luo ilmainen Supabase-tili → https://supabase.com
   (ei luottokorttia tarvita)

2. Luo ilmainen Resend-tili → https://resend.com
   (sähköpostien lähetystä varten)

3. Avaa sovelluksen linkki ja rekisteröidy
   → Saat automaattisesti admin-oikeudet!

4. Täytä yrityksen tiedot → Valmis käyttöön!

⏱️ Käyttöönottoaika: 15-20 minuuttia
💻 Ei asennuksia, ei koodausta - vain avaa linkki ja aloita!

Kaikki toimii automaattisesti:
✅ Tietokanta rakennetaan itsestään
✅ Sähköpostitoiminnot aktivoituvat itsestään
✅ Kaikki ominaisuudet käytössä heti

Tuki: [Sinun yhteystietosi]

Ystävällisin terveisin,
[Sinun nimesi]
```

---

## Tukipyynnöt ja yleiset ongelmat

### Ongelma 1: Migraatiovirheet
**Ratkaisu:** Varmista, että ostaja ajoi migraatiot oikeassa järjestyksessä (numerojärjestys tiedostonimissä).

### Ongelma 2: Sovellukseen ei pääse
**Ratkaisu:** Tarkista, että ostaja:
1. Lisäsi oikean Supabase URL:n ja Anon-avaimen
2. Aktivoi sähköpostiautentikoinnin Supabasessa

### Ongelma 3: Sähköpostit eivät lähtee
**Ratkaisu:** Varmista, että ostaja:
1. Lisäsi Resend API-avaimen Supabase Secretsiin
2. Deploiasi Edge Functionit

### Ongelma 4: Ei admin-oikeuksia
**Ratkaisu:** Ensimmäinen rekisteröitynyt käyttäjä saa automaattisesti admin-roolin. Jos tämä ei toimi:
```sql
-- Aja tämä Supabase SQL Editorissa
UPDATE public.user_roles 
SET role = 'admin' 
WHERE user_id = '[käyttäjän-uuid]';
```

---

## Hinnoittelusuositukset

### Perusversio
- **Kertamaksu:** 200-500 €
- Sisältää: Sovellus + asennusohjeet + 30 päivän sähköpostituki

### Tuki (valinnainen)
- **Kertamaksu:** +100-200 €
- 90 päivän laajennettu tuki

### Räätälöinti (valinnainen)
- **Tuntiperuste:** 50-100 €/h
- Mukautukset ostajan tarpeisiin

**Huom:** Supabase-kulut ovat ostajan vastuulla (ilmainen tier riittää useimmille).

---

## Myyntivinkit

### Mitä korostaa:
✅ **Kertamaksu** - Ei kuukausimaksuja (vain Supabase, jos ylittää ilmaisen tierin)
✅ **Oma data** - Ostaja omistaa ja hallinnoi dataa
✅ **Nopea käyttöönotto** - 20-30 minuuttia
✅ **Skaalautuva** - Supabase kasvaa yrityksen mukana
✅ **Turvallinen** - Row Level Security + Supabase-autentikointi

### Mitä EI luvata:
❌ Ilmainen hosting (Supabase on ostajan vastuulla)
❌ Rajaton tuki (määrittele tukijakso selkeästi)
❌ Automaattiset päivitykset (ostaja päivittää itse tai ostaa tuen)

---

## Tarkistuslista ennen myyntiä

- [ ] Sovellus julkaistu Lovablessa
- [ ] Julkaisulinkki testattu (toimii ja avautuu)
- [ ] Julkaisulinkki testattu uudella Supabase-tilillä
- [ ] Rekisteröityminen toimii (ensimmäinen käyttäjä saa admin-oikeudet)
- [ ] Migraatiot ajetaan automaattisesti (tarkista että taulut luodaan)
- [ ] Valmista myyntitekstiä/sähköpostia (ks. yllä)
- [ ] Yhteystietosi valmiina tukea varten (jos tarjoat)

---

## Onnea myyntiin! 🚀

Jos sinulla on kysymyksiä, voit kysyä neuvoa Lovable-yhteisöltä tai Supabase-foorumeilta.
