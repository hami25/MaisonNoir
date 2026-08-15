# MAISON NOIR — Einrichtung

Drei Schritte: Datenbank anlegen, Shop verbinden, online stellen.
Zusammen etwa 20 Minuten. Kosten: 0 € (Supabase Free, Vercel Hobby).

---

## 1 · Supabase einrichten

**1.1 Projekt anlegen**
Auf [supabase.com](https://supabase.com) registrieren → *New project*.
Region **Frankfurt (eu-central-1)** wählen — kürzeste Wege und Daten in der EU.
Das Datenbank-Passwort notieren, es wird beim Anlegen nur einmal gezeigt.

**1.2 Schema einspielen**
Links im Menü **SQL Editor** → *New query*. Den kompletten Inhalt von
`supabase.sql` einfügen und **Run** drücken. Läuft es durch, stehen Tabellen,
Trigger und Sicherheitsregeln.

**1.3 E-Mail-Versand einrichten**

Supabase verschickt Mails im Auslieferungszustand über einen eingebauten Dienst,
der **nur zum Testen** gedacht und stark gedrosselt ist. Bestellen abends mehrere
Leute gleichzeitig, kommen die Bestätigungsmails nicht an — und niemand merkt es.

Zwei Möglichkeiten:

**Zum Testen (vorläufig):** **Authentication → Sign In / Providers → Email**,
*Confirm email* ausschalten. Anmelden funktioniert dann sofort ohne Mailversand.
Nachteil: Tippfehler in der Adresse fallen nicht auf, und wer sich vertippt,
kann sein Passwort nie zurücksetzen. Für den Livegang keine Dauerlösung.

**Für den echten Betrieb:** eigenen Mailversand über Resend einrichten
(→ Abschnitt „E-Mail-Versand einrichten"). Danach bleibt *Confirm email* eingeschaltet.

**1.4 Zugangsdaten kopieren**
**Project Settings → Data API**: die **Project URL** kopieren.
**Project Settings → API Keys**: den **anon / public** Schlüssel kopieren.

> Der `anon key` gehört ins Frontend und darf öffentlich sein — er ist dafür
> gemacht. Der Schutz kommt aus den Row-Level-Security-Regeln.
> Den **`service_role`** Schlüssel niemals in die Seite schreiben: der umgeht
> jede Regel und gibt vollen Zugriff auf alle Daten.

---

## 2 · Shop verbinden

In `index.html` ganz oben im Skriptblock (etwa Zeile 768):

```js
const CFG={
  url:'https://xxxxxxxxxxx.supabase.co',
  key:'eyJhbGciOiJIUzI1NiIsInR5cCI6...'
};
```

Datei lokal im Browser öffnen und testen: Konto anlegen, etwas bestellen.
Solange nichts eingetragen ist, erscheint unten ein Hinweisbalken.

**Dich zum Inhaber machen:** erst im Shop ganz normal registrieren, dann im
SQL-Editor ausführen:

```sql
update public.profiles set role = 'admin' where email = 'deine@mail.de';
```

Danach im Shop einmal ab- und wieder anmelden — der Punkt **Verwaltung**
erscheint in der Navigation.

---

## 3 · Auf Vercel stellen

**Weg A — ohne Git, am schnellsten**

```bash
npm i -g vercel
cd maison-noir
vercel        # Fragen mit Enter bestätigen
vercel --prod # veröffentlichen
```

**Weg B — über GitHub, empfohlen**

1. Ordner als Repository zu GitHub pushen
2. Auf [vercel.com](https://vercel.com) → *Add New Project* → Repository wählen
3. Framework Preset: **Other**. Build Command und Output leer lassen —
   es ist eine statische Seite.
4. *Deploy*

Ab dann veröffentlicht jeder Push automatisch die neue Version.

**Eigene Domain:** Vercel → Project → *Settings → Domains* → Domain eintragen
und die angezeigten DNS-Einträge beim Anbieter hinterlegen. HTTPS kommt
automatisch.

**Zum Schluss in Supabase eintragen:** *Authentication → URL Configuration* →
Site URL und Redirect URLs auf deine Vercel-Adresse setzen.

---

## Was wo passiert

| | |
|---|---|
| Produktkatalog | fest in `index.html`, 177 Artikel |
| Konten & Passwörter | Supabase Auth, gehasht |
| Bestellungen | Tabelle `orders` + `order_items` |
| Warenkorb | lokal im Browser des Kunden |
| Markenreferenz-Schalter | Tabelle `settings` |

**Wer darf was**

- Kunden sehen ausschließlich ihre eigenen Bestellungen
- Nur das Inhaber-Konto sieht alle und darf den Status ändern
- Die Rolle `admin` kann niemand sich selbst geben — das Recht auf die Spalte
  ist entzogen und geht nur per SQL
- Die Statusabfrage per Duft-Code läuft ohne Anmeldung, gibt aber nur Status,
  Betrag und Art der Übergabe zurück — keine Namen, Telefonnummern, Adressen

---

## Preise ändern

Die Artikel stehen in `index.html` in den Blöcken `D_DAME`, `D_HERR`, `D_UNI`,
`D_LUX`, `D_EVENT`, `D_KIND`, `D_GEL`, `D_LOT`. Aufbau bei Parfums:

```
[Nummer, Name, InspiriertVon, Geschlecht, Duftfamilie, Kopf, Herz, Basis, Preis30, Preis70]
```

Fehlen die letzten beiden Werte, gelten 18 € und 35 €.
Geschlecht: `h` Herren · `d` Damen · `u` Unisex · `k` Kinder.
Nach dem Ändern speichern und neu deployen.

---

## E-Mail-Versand einrichten

Du brauchst echten Mailversand für **Registrierungsbestätigung** und
**Passwort-Zurücksetzen**. Für den Duft-Code nicht — der steht direkt auf
dem Bildschirm.

### Brevo — ohne Domain, ohne DNS

Der schnellste Weg. Du verifizierst **eine einzelne E-Mail-Adresse** per
Zahlencode statt eine ganze Domain über DNS-Einträge. Deine bestehende
Adresse reicht, auch eine von Gmail, GMX oder Web.de.

**1 · Konto anlegen**
Auf [brevo.com](https://www.brevo.com) registrieren. Keine Kreditkarte nötig.

**2 · Absenderadresse verifizieren**
**Senders, Domains & Dedicated IPs → Senders → Add a Sender**.
Name und deine E-Mail-Adresse eintragen. Brevo schickt einen **6-stelligen
Code** an diese Adresse — eintragen, fertig. Keine DNS-Einträge.

**3 · SMTP-Zugang holen**
Links unten auf deinen Namen → **SMTP & API → SMTP**.
Dort stehen zwei Dinge, die du brauchst:

- **Login**, sieht aus wie `7a1b2c@smtp-brevo.com`
- **Master Password**, mit *Generate a new SMTP key* erzeugen

> Das ist die Stelle, an der die meisten scheitern: Der Login ist **nicht**
> deine Konto-E-Mail, sondern diese kryptische `@smtp-brevo.com`-Adresse.
> Und es ist das **SMTP-Passwort**, nicht der API-Key daneben.

**4 · In Supabase eintragen**
**Project Settings → Authentication → SMTP Settings**, *Enable Custom SMTP* an:

| Feld | Wert |
|---|---|
| Host | `smtp-relay.brevo.com` |
| Port | `587` |
| Username | dein Login, z. B. `7a1b2c@smtp-brevo.com` |
| Password | das SMTP Master Password |
| Sender email | die Adresse, die du in Schritt 2 verifiziert hast |
| Sender name | `Maison Noir` |

**5 · Bestätigung einschalten und testen**
**Authentication → Sign In / Providers → Email** → *Confirm email* an.
Mit einer echten Adresse registrieren. In Brevo unter **Transactional → Logs**
siehst du jeden Versand mit Status.

**Grenzen:** 300 Mails pro Tag, dauerhaft kostenlos. Für Registrierungen und
Passwort-Zurücksetzen mehr als genug.

**Ein Schönheitsfehler:** Solange keine Domain hinterlegt ist, ersetzt Brevo
den Absender durch eine eigene Adresse auf `brevosend.com`. Die Mails kommen
an, sehen aber weniger nach dir aus. Sobald du eine Domain hast, kannst du sie
unter *Domains* nachträglich authentifizieren — dann verschwindet das.

### Resend — später, wenn du eine Domain hast

Resend verlangt zwingend eine verifizierte Domain mit MX-, SPF- und
DKIM-Einträgen; einen Ersatz-Absender gibt es nicht. Dafür sieht der Absender
danach professionell aus.

| Feld | Wert |
|---|---|
| Host | `smtp.resend.com` |
| Port | `465` |
| Username | `resend` — genau dieses Wort |
| Password | API-Key, komplett mit `re_` |
| Sender email | Adresse auf der verifizierten Domain |

Gratis: 3.000 Mails im Monat, aber höchstens 100 pro Tag, eine Domain.

### Wenn nichts ankommt

| Fehler | Ursache |
|---|---|
| 535 bei Brevo | Username ist die Konto-E-Mail statt `…@smtp-brevo.com` |
| 535 bei Resend | Username ist nicht wörtlich `resend` |
| 550, Absender abgelehnt | Sender email ist nicht verifiziert |
| Landet im Spam | Domain nicht authentifiziert — bei Brevo zunächst normal |
| Nichts passiert | In den Logs des Anbieters nachsehen. Steht dort nichts, liegt es an Supabase |

Teste am Ende auch das **Passwort-Zurücksetzen** — das ist der Weg, den Kunden
im Ernstfall brauchen, und genau der fällt beim Einrichten oft durchs Raster.

---

## Rechtliches vor dem Livegang

- **Impressum und Datenschutzerklärung** sind für einen gewerblichen Shop
  in Deutschland Pflicht. Beides fehlt noch.
- **Markenreferenzen:** Der Schalter in der Verwaltung ist aus gutem Grund
  standardmäßig aus. Duft-Vergleichslisten gelten seit *L'Oréal ./. Bellure*
  (EuGH) als unzulässige Rufausbeutung. Das ist keine Rechtsberatung —
  im Zweifel anwaltlich klären lassen.
- **Barzahlung** heißt: keine Zahlungsdienstleister, keine PCI-Anforderungen.
  Widerrufsrecht und Preisangabenverordnung gelten trotzdem.
