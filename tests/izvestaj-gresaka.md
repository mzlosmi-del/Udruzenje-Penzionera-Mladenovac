# Izveštaj grešaka — index.html

**Datum analize:** 2026-04-02  
**Fajl:** `index.html`  
**Verzija:** Single-page aplikacija (SPA) bez backend-a

---

## Sadržaj

1. [HTML Struktura](#1-html-struktura)
2. [Pristupačnost (Accessibility)](#2-pristupačnost-accessibility)
3. [SEO](#3-seo)
4. [Sigurnost](#4-sigurnost)
5. [Mobilna verzija](#5-mobilna-verzija)
6. [Sumarni pregled](#6-sumarni-pregled)

---

## 1. HTML Struktura

### 1.1 Više `<h1>` elemenata istovremeno u DOM-u — KRITIČNO
**Lokacija:** linija 571, 651, 716, 729, 771, 835  
Svaka "stranica" (home, about, events, services, contact, admin) ima sopstveni `<h1>`, ali svi su prisutni u DOM-u u isto vreme — samo sakriveni CSS-om (`display: none`). Browser i search engine vide više `<h1>` tagova odjednom, što narušava hijerarhiju naslova.

### 1.2 Svi navigacioni linkovi imaju `href="#"` — OZBILJNO
**Lokacija:** linija 554–559, 950–954  
Navigacija koristi `href="#"` i JavaScript `onclick` za prebacivanje između "stranica". Nema pravih URL-ova, što znači da:
- Korisnik ne može da sačuva direktan link do određene stranice (nema deep linking).
- Pretraživači ne mogu da indeksiraju unutrašnje stranice.
- Klikom na dugme "nazad" u browseru ne vraća se na prethodnu sekciju.

### 1.3 `<div>` elementi sa `onclick` koji nisu interaktivni HTML elementi — OZBILJNO
**Lokacija:** linije 629, 634, 639 (`.info-card`)  
Info kartice koriste `onclick` na `<div>` elementima. `<div>` nije interaktivan element — nije focusable putem tastature, nema `role="button"`, nema `tabindex`.

### 1.4 `<label>` bez `for` atributa — OZBILJNO
**Lokacija:** linije 814, 818, 822 (kontakt forma); linije 850, 853, 858, 863, 873 (admin forma)  
Labeli u formama nisu vezani za odgovarajuće `<input>` elemente. Na primer:
```html
<label>Vaše ime</label>
<input type="text" placeholder="Ime i prezime">
```
Treba biti:
```html
<label for="contactName">Vaše ime</label>
<input type="text" id="contactName" ...>
```

### 1.5 Kontakt forma `<input>` elementi bez `id` atributa — SREDNJE
**Lokacija:** linije 815, 819, 823  
Polja za ime, e-mail i poruku u kontakt formi nemaju `id` atribute, što onemogućava ispravno vezivanje sa `<label>` elementima i otežava JavaScript manipulaciju.

### 1.6 Modal bez semantičkih atributa — SREDNJE
**Lokacija:** linija 520  
Login modal je implementiran kao `<div class="modal-overlay">` bez:
- `role="dialog"`
- `aria-modal="true"`
- `aria-labelledby`
- Upravljanja fokusom (fokus ne ostaje unutar modala)

### 1.7 Inline `style` atributi razbacani po celom dokumentu — NISKO
**Lokacija:** ~30 mesta (linija 530, 720, 812, 864, 871, 879, 880, itd.)  
Korišćenje inline stilova otežava održavanje i onemogućava uniformnu temu. Svi stilovi treba da budu u `<style>` sekciji ili eksternom CSS fajlu.

### 1.8 Inline `onclick` atributi u HTML-u — NISKO
**Lokacija:** linija 528, 531, 532, 554–559, 574, 575, 629, itd.  
Korišćenje `onclick=""` direktno u HTML-u je loša praksa. Treba koristiti `addEventListener` u JavaScript-u (separation of concerns).

### 1.9 `<select>` element bez `<label>` u adminu — NISKO
**Lokacija:** linija 864  
`<select id="newsCategory">` ima labelu koja nije vezana (`for` atribut nedostaje).

---

## 2. Pristupačnost (Accessibility)

### 2.1 Emoji kao jedini sadržaj interaktivnih/vizuelnih elemenata — OZBILJNO
**Lokacija:** linija 547 (logo), 559 (admin nav), 612, 619 (sidebar headeri), itd.  
Emojiji se koriste kao dekorativni i funkcionalni sadržaj bez alternativnog teksta. Screen readeri ih čitaju doslovno (npr. "Rukovanje" umesto loga organizacije). Logo ikona:
```html
<div class="logo-icon">🤝</div>
```
...nema `aria-label` niti `aria-hidden="true"` ako je čisto dekorativna.

### 2.2 `<label>` nije funkcionalno vezan za `<input>` — OZBILJNO
**Lokacija:** Sve forme (kontakt + admin)  
Klikanje na labelu ne aktivira odgovarajući input jer `for`/`id` veza ne postoji. Ovo je problem za korisnike koji koriste miš i za screen readere.

### 2.3 `<nav>` bez `aria-label` — SREDNJE
**Lokacija:** linija 553  
Postoji samo jedna `<nav>`, ali dobra praksa je dodati `aria-label="Glavna navigacija"` za jasnoću.

### 2.4 Kalendar dugmad bez opisnih labela — SREDNJE
**Lokacija:** linije 1099–1101 (dinamički renderovano)  
Dugmad `‹` i `›` za navigaciju kalendarom nemaju `aria-label`. Screen reader će pročitati samo unicode karakter.

### 2.5 Nema "skip to main content" linka — SREDNJE
Korisnici koji se kreću tastaturom moraju proći kroz celu navigaciju pri svakom učitavanju stranice. Standard zahteva skip link na vrhu `<body>`.

### 2.6 Outline uklonjen sa input polja bez alternative — SREDNJE
**Lokacija:** linija 417  
```css
outline: none;
```
Uklanjanje `outline`-a bez definisanja alternativnog fokus indikatora (npr. `box-shadow`) onemogućava korisnicima tastature da vide koji je element u fokusu.

### 2.7 Nema `aria-live` regiona za dinamički sadržaj — SREDNJE
Vesti, događaji i obaveštenja se renderuju putem `innerHTML`. Screen readeri ne najavljuju ove promene jer nema `aria-live="polite"` regiona.

### 2.8 Touch target veličina ispod preporučenog minimuma — NISKO
**Lokacija:** linije 254–261 (kalendar dugmad)  
Kalendar `‹`/`›` dugmad su 28×28px. WCAG 2.5.8 preporučuje minimum 24×24px (AA), a APCA i mobilni standardi preporučuju 44×44px.

### 2.9 Kontrast teksta — NISKO
- `rgba(255,255,255,0.65)` na `#2a4a6e` pozadini (header top) — kontrast je ~3.8:1, ispod 4.5:1 za normalan tekst (WCAG AA).
- `#6b6b6b` (--text-muted) na beloj pozadini — kontrast ~5.7:1 (OK za normalan tekst, ali na ivici za mali tekst 12–13px).

### 2.10 `<h3>` korišćen bez prethodnog `<h2>` u nekim sekcijama — NISKO
Hijerarhija naslova nije konzistentna. Npr. u admin panelu, `<h3>` se koristi kao prva stavka bez prethodnog `<h2>`.

---

## 3. SEO

### 3.1 Nema `<meta name="description">` — KRITIČNO
**Lokacija:** `<head>` sekcija  
Meta opis nedostaje. Google prikazuje ovaj sadržaj u rezultatima pretrage. Bez njega, Google sam bira tekst koji može biti neprikladan.

### 3.2 Sav sadržaj je JavaScript-dependent — KRITIČNO
Vesti i događaji se renderuju isključivo putem JavaScript-a (`innerHTML`). Ako Googlebot ne izvrši JS (ili u SSR/SSG kontekstu), stranica izgleda prazno. Sadržaj nije indeksiv.

### 3.3 Više `<h1>` tagova u DOM-u — KRITIČNO
*(Videti 1.1)* Search engine vidi sve `<h1>` tagove odjednom, što negativno utiče na relevantnost ključnih reči.

### 3.4 Nema Open Graph tagova — OZBILJNO
Nedostaju:
```html
<meta property="og:title" content="...">
<meta property="og:description" content="...">
<meta property="og:image" content="...">
<meta property="og:url" content="...">
```
Ovo znači da deljenje linka na društvenim mrežama neće prikazati nikakav preview.

### 3.5 Nema `<link rel="canonical">` — OZBILJNO
Bez canonical linka, pretraživači ne znaju koji je "pravi" URL stranice.

### 3.6 Nema favicon — SREDNJE
**Lokacija:** `<head>` sekcija  
Nema `<link rel="icon">`. Browser tab prikazuje generičku ikonu, a search engine rezultati i bookmark-ovi nemaju vizuelni identitet.

### 3.7 Nema Schema.org / JSON-LD strukturiranih podataka — SREDNJE
Organizacija, kontakt info i događaji bi mogli da budu označeni Schema.org markup-om za bolji prikaz u Google rezultatima (rich snippets, Knowledge Panel).

### 3.8 `<title>` nije specifičan po sekciji — NISKO
Naslov stranice je uvek "Udruženje Penzionera Mladenovac" bez obzira na aktivnu sekciju. U single-page aplikacijama, preporučuje se dinamička promena naslova (`document.title = ...`) pri navigaciji.

### 3.9 Nema `<meta name="robots">` taga — NISKO
Bez eksplicitnih instrukcija za crawlere, podrazumevano ponašanje može se razlikovati između pretraživača.

---

## 4. Sigurnost

### 4.1 Admin lozinka hardkodirana u client-side JavaScript-u — KRITIČNO
**Lokacija:** linija 984  
```javascript
const ADMIN_PASS = "admin123";
```
Lozinka je vidljiva svakome ko otvori DevTools (F12 → Sources/Network). Nije potreban nikakav poseban napad — lozinka je u plain textu u izvornom kodu.

### 4.2 Demo lozinka prikazana u HTML-u — KRITIČNO
**Lokacija:** linija 534  
```html
<p style="...">Demo lozinka: <strong>admin123</strong></p>
```
Lozinka je eksplicitno prikazana u modalu za prijavu, vidljiva je svim posetiocima.

### 4.3 Autentifikacija isključivo na klijentskoj strani — KRITIČNO
**Lokacija:** linije 985, 1033–1043  
`isLoggedIn` promenljiva postoji samo u JavaScript-u na klijentu. Nema server-side verifikacije. Napadač može da postavi `isLoggedIn = true` u konzoli i direktno pozove `showPage('admin', ...)`.

### 4.4 XSS (Cross-Site Scripting) ranjivost putem `innerHTML` — OZBILJNO
**Lokacija:** linije 1056–1065, 1204–1215, 1159–1180, 1252–1262  
Korisnički unos se ubacuje direktno u HTML bez escapeovanja:
```javascript
el.innerHTML = sorted.map(n => `
  <div class="news-card-title">${n.title}</div>  // nije escapovano!
  <p>${n.desc}</p>                                 // nije escapovano!
`).join('');
```
Ako korisnik unese `<img src=x onerror=alert(1)>` kao naslov vesti, biće izvršen. Trenutno je ublaženo time što se podaci ne čuvaju (samo u memoriji), ali pri uvođenju backend-a ovo postaje direktna XSS ranjivost.

### 4.5 `target="_blank"` bez `rel="noopener noreferrer"` — OZBILJNO
**Lokacija:** linije 960–963  
```html
<a href="https://www.minrzs.gov.rs/" target="_blank">...</a>
```
Bez `rel="noopener noreferrer"`, otvorena stranica može da pristupi `window.opener` objektu i izvrši tabnabbing napad.

### 4.6 Kontakt forma bez backend-a — SREDNJE
**Lokacija:** linija 825  
```javascript
onclick="alert('Poruka poslata! Javićemo vam se uskoro.')"
```
Forma se nikuda ne šalje. Korisnik misli da je poruka poslata, ali nije. Ovo je funkcionalni problem koji može da stvori poverenje korisnika u sistem koji ne radi.

### 4.7 Nema Content Security Policy (CSP) — SREDNJE
Nema `<meta http-equiv="Content-Security-Policy">` taga, što ostavlja sajt bez zaštite od ubacivanja malicioznih skripti.

### 4.8 Google Fonts bez Subresource Integrity (SRI) — NISKO
**Lokacija:** linija 7  
```html
<link href="https://fonts.googleapis.com/css2?..." rel="stylesheet">
```
Nema `integrity` atributa. Ako Google Fonts server bude kompromitovan, maliciozni CSS bi bio učitan bez upozorenja.

---

## 5. Mobilna verzija

### 5.1 Navigacija se potpuno sakriva na mobilnim — KRITIČNO
**Lokacija:** linija 511  
```css
@media (max-width: 900px) {
  nav { display: none; }
}
```
Cela navigacija, uključujući dugme "Admin", nestaje na mobilnim uređajima. Nema hamburger menija niti alternative. Korisnici na mobilnim telefonima ne mogu da navigiraju do sekcija O nama, Aktivnosti, Usluge, Kontakt.

### 5.2 Hero statistike sakrivene na mobilnim — OZBILJNO
**Lokacija:** linija 507  
```css
.hero-stats { display: none; }
```
Statistike (1800+ članova, 50+ godina rada, itd.) su potpuno sakrivene na mobilnim uređajima. Ovaj sadržaj je informativan i trebao bi biti prilagođen za mobilni, ne skriven.

### 5.3 Padding/margine prevelike za male ekrane — SREDNJE
**Lokacija:** linija 43 (`.header-top`), 53 (`.header-main`), 179 (`.content-wrap`)  
`padding: 6px 40px`, `padding: 18px 40px`, `padding: 60px 40px` — 40px horizontalni padding na ekranu od 375px (iPhone) ostavlja samo 295px sadržaja. Ovo nije riješeno u media query.

### 5.4 Footer grid na veoma malim ekranima — SREDNJE
**Lokacija:** linija 510  
```css
.footer-grid { grid-template-columns: 1fr 1fr; }
```
Na ekranima ispod 400px, dva kolone futer mogu biti pretijesna. Treba dodati breakpoint za `max-width: 480px` sa `grid-template-columns: 1fr`.

### 5.5 Featured banner bez mobilne adaptacije — SREDNJE
**Lokacija:** linije 332–349  
`.featured-banner` koristi `display: flex` sa `gap: 24px` i fiksnom `padding: 32px 36px`. Na mobilnim uređajima, sadržaj može biti previše zgusnut.

### 5.6 Nema hamburger menija — SREDNJE
*(Videti 5.1)* Nedostaje implementacija mobilnog menija. Minimalna ispravka bi bila dodati hamburger dugme i toggle klasu na `nav` element.

### 5.7 Touch target dimenzije — NISKO
**Lokacija:** kalendar dugmad (28×28px), btn-icon (32×32px)  
WCAG i Apple/Google smernice preporučuju minimalno 44×44px za touch targets na mobilnim uređajima. Kalendar dugmad `‹`/`›` i ikone za brisanje/uređivanje su premale.

### 5.8 Horizontalni scroll na malim ekranima — NISKO
**Lokacija:** `.hero-inner`, `.footer-grid`  
Neke fiksne dimenzije i `gap` vrednosti mogu prouzrokovati horizontalni scroll na ekranima ispod 360px širine.

---

## 6. Sumarni pregled

| Kategorija | Kritično | Ozbiljno | Srednje | Nisko | Ukupno |
|---|:---:|:---:|:---:|:---:|:---:|
| HTML Struktura | 1 | 2 | 3 | 3 | **9** |
| Pristupačnost | 0 | 3 | 5 | 2 | **10** |
| SEO | 3 | 2 | 2 | 2 | **9** |
| Sigurnost | 3 | 2 | 2 | 1 | **8** |
| Mobilna verzija | 1 | 3 | 4 | 1 | **9** |
| **UKUPNO** | **8** | **12** | **16** | **9** | **45** |

### Prioriteti za hitnu ispravku

1. **SIGURNOST — Admin lozinka u kodu** (§4.1, §4.2, §4.3): Lozinka je vidljiva svima. Pre puštanja u produkciju, autentifikacija mora biti premještena na server.
2. **SIGURNOST — XSS** (§4.4): `innerHTML` sa nesanitizovanim korisničkim unosom mora biti zamenjeno sa `textContent` ili sanitizacijom.
3. **SIGURNOST — target="_blank"** (§4.5): Dodati `rel="noopener noreferrer"` na sve external linkove.
4. **MOBILNA — Navigacija nedostaje** (§5.1): Implementirati mobilni meni.
5. **SEO — Meta description** (§3.1): Dodati opisni meta tag.
6. **HTML — Labeli u formama** (§1.4): Vezati `<label for="">` za `<input id="">`.
7. **PRISTUPAČNOST — Fokus indikator** (§2.6): Nikad ne uklanjati `outline` bez alternative.
8. **KONTAKT FORMA — Ne radi** (§4.6): Forma mora da šalje poruke na pravi server endpoint.
