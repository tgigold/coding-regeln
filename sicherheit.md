# Sicherheit

Diese Regeln gelten als Grundausstattung. Ein Projekt muss sie nicht neu herleiten, sondern nur umsetzen.

- [Transport und Header](#transport-und-header)
- [Sessions](#sessions)
- [CSRF](#csrf)
- [Injection](#injection)
- [Login und Nutzerverwaltung](#login-und-nutzerverwaltung)
- [Datei-Uploads](#datei-uploads)
- [Secrets](#secrets)
- [Fehler und Logs](#fehler-und-logs)

## Transport und Header

Kopiervorlage: [vorlagen/htaccess-public](vorlagen/htaccess-public)

- **S-1** HTTPS ist Pflicht: Redirect in `.htaccess` und in der App, immer auf die Domain aus der Config, nie auf den `Host`-Header.
- **S-2** HSTS ist gesetzt (`Strict-Transport-Security: max-age=31536000`). `includeSubDomains` nur, wenn alle Subdomains der Domain HTTPS sprechen.
- **S-3** Eine strikte Content Security Policy: `default-src 'self'`, `object-src 'none'`, `base-uri 'self'`, `form-action 'self'`, `frame-ancestors 'none'`. Kein `unsafe-inline`, kein `unsafe-eval`. Für Bootstrap braucht es zusätzlich nur `img-src 'self' data:`.
- **S-4** Es gibt keine Inline-Skripte, keine Inline-Event-Handler (`onclick=…`) und keine `style`-Attribute. Daten gehen über `data-`-Attribute ans JavaScript. Lässt sich ein Inline-Skript nicht vermeiden, bekommt es eine pro Request zufällige Nonce.
- **S-5** Weitere Header auf jeder Antwort: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin` (fremde Server sehen nur die Domain, nie den Pfad), `Cross-Origin-Opener-Policy: same-origin` und eine `Permissions-Policy`, die Kamera, Mikrofon, Standort und alles andere Ungenutzte abschaltet.
- **S-6** `X-Forwarded-Proto` und `X-Forwarded-For` werden nur von konfigurierten, vertrauenswürdigen Proxys übernommen, nie von beliebigen Clients.
- **S-7** Seiten hinter dem Login werden mit `Cache-Control: no-store` ausgeliefert.

## Sessions

- **S-8** Session-Cookies tragen das `__Host-`-Präfix und sind immer `Secure`, `HttpOnly`, `SameSite=Lax` mit `Path=/` – auch beim Löschen des Cookies.
- **S-9** `session.use_strict_mode` und `session.use_only_cookies` sind an. Sessions werden erst angelegt, wenn eine Seite sie braucht, und die Session-ID wird nach Login und nach jedem Rechtewechsel neu erzeugt.
- **S-10** Jede Session ist an eine Sitzungs-Epoche gebunden. Ein Passwort-Reset und „Überall abmelden“ beenden alle Sitzungen auf allen Geräten.
- **S-11** Sessions haben eine Leerlauf- und eine absolute Höchstdauer; beide stehen in der Config. Abmelden zerstört die Session auf dem Server, nicht nur das Cookie.

## CSRF

- **S-12** Jedes Formular und jede andere zustandsändernde Anfrage ist per CSRF-Token geschützt. Das Token wird nur aus dem POST-Body oder einem eigenen Header gelesen, nie aus dem Query-String.
- **S-13** Tokens stammen aus `random_bytes` und werden mit `hash_equals` verglichen.
- **S-14** Zusätzlich lehnt die App zustandsändernde Anfragen ab, deren `Origin`- oder `Sec-Fetch-Site`-Header auf eine fremde Seite zeigt.
- **S-15** API-Endpunkte, die sich ausschließlich per Token im `Authorization`-Header anmelden, brauchen kein CSRF-Token – und akzeptieren deshalb nie das Session-Cookie.

## Injection

### SQL

- **S-16** Alle Datenbankzugriffe laufen über Prepared Statements, und das ausschließlich innerhalb von Repositories. Werte werden nie in SQL-Strings eingesetzt.
- **S-17** Was sich nicht binden lässt (Tabellen- und Spaltennamen, `ORDER BY`, Sortierrichtung), kommt aus einer festen Whitelist im Code, nie aus der Anfrage.
- **S-18** In `LIKE`-Suchen werden `%`, `_` und das Escape-Zeichen im Suchbegriff escaped.

### Ausgabe (XSS)

- **S-19** In Templates wird jede Ausgabe mit `e()` escaped. `e()` ist `htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')`.
- **S-20** Escaping richtet sich nach dem Kontext: HTML-Attribute stehen immer in Anführungszeichen; Daten für JavaScript gehen als `json_encode` mit `JSON_HEX_TAG | JSON_HEX_AMP | JSON_HEX_APOS | JSON_HEX_QUOT` in ein `data-`-Attribut; URL-Teile werden mit `rawurlencode` gebaut.
- **S-21** Links und Weiterleitungen aus Nutzereingaben sind nur mit Schema `http` oder `https` erlaubt. `javascript:`, `data:` und andere werden abgelehnt.
- **S-22** HTML von Nutzern wird, wenn überhaupt, durch einen etablierten Whitelist-Sanitizer geschickt, nie durch eigene reguläre Ausdrücke. Fremdes HTML (E-Mails, Feeds) zählt als Nutzer-HTML.

### Weitere

- **S-23** Nutzereingaben gehen nie in `exec`, `shell_exec`, `system`, `passthru`, `proc_open`, `eval`, `include` oder `require`. Lässt sich ein Shell-Aufruf nicht vermeiden, ist das Kommando fest und jedes Argument läuft durch `escapeshellarg`.
- **S-24** Dateipfade aus Eingaben werden mit `realpath` aufgelöst und müssen danach innerhalb des erwarteten Basisverzeichnisses liegen. Besser: Die Eingabe ist eine ID, der Pfad steht in der Datenbank.
- **S-25** `unserialize` wird nie auf Daten angewandt, die von außen kommen können. Für Datenaustausch gibt es JSON.
- **S-26** Weiterleitungen nach Login oder Formularen gehen nur auf relative Pfade der eigenen App. Ein Ziel, das mit `//` beginnt oder einen Host enthält, wird verworfen.

Mail-Header: siehe `E-12`. Mass Assignment: siehe `E-6`.

## Login und Nutzerverwaltung

### Passwörter

- **S-27** Passwörter werden mit `password_hash` und Argon2id gespeichert (wo der Server es nicht anbietet: `PASSWORD_DEFAULT`). Nach jedem erfolgreichen Login prüft `password_needs_rehash`, ob der Hash erneuert werden muss.
- **S-28** Mindestlänge 12 Zeichen, Höchstlänge 1024, keine Pflicht zu Sonderzeichen oder Ziffern, kein erzwungener regelmäßiger Wechsel. Passwort-Manager und Einfügen bleiben möglich.
- **S-29** Passwort-Reset: Das Token stammt aus `random_bytes` (mindestens 32 Byte), wird nur als Hash gespeichert, gilt 60 Minuten und genau einmal. Ein erfolgreicher Reset beendet alle Sitzungen (`S-10`).
- **S-30** Login, Registrierung und Passwort-Reset verraten nicht, ob ein Konto existiert: gleiche Meldung, gleicher Statuscode, vergleichbare Antwortzeit.

### Login-Schutz

- **S-31** Login-Sperre: Nach 5 Fehlversuchen pro IP oder 25 insgesamt ist der Login 15 Minuten gesperrt. Geräte, die sich schon einmal erfolgreich angemeldet haben, sind von der globalen Sperre ausgenommen, damit ein Angreifer den Besitzer nicht aussperren kann.

### Rechte

- **S-32** Rechte werden auf dem Server geprüft, bei jedem Request und für jedes einzelne Objekt: Nicht nur „ist angemeldet“, sondern „darf dieser Nutzer diesen Datensatz lesen oder ändern“. Ausgeblendete Buttons sind kein Schutz.
- **S-33** Zugriff auf fremde Datensätze wird mit `404` beantwortet, nicht mit `403`, damit sich nicht erraten lässt, welche IDs existieren.
- **S-34** Rollen sind so knapp wie möglich. Neue Konten starten mit der niedrigsten Rolle; die eigene Rolle kann niemand selbst ändern.
- **S-35** Vor heiklen Aktionen (Passwort oder E-Mail ändern, Konto löschen, API-Token erzeugen, Nutzer verwalten) wird das Passwort erneut abgefragt.
- **S-36** Eine neue E-Mail-Adresse gilt erst nach Bestätigung über einen Link an die neue Adresse; die alte Adresse wird informiert.
- **S-37** Selbstregistrierung, wo es sie gibt, läuft per Double-Opt-in und ist per Rate-Limit begrenzt.
- **S-38** Projekte mit mehreren Nutzern bieten für Admin-Konten Zwei-Faktor-Anmeldung per TOTP an, mit Wiederherstellungscodes, die nur als Hash gespeichert sind.
- **S-39** Admin-Aktionen (Nutzer anlegen, Rollen ändern, löschen, Einstellungen ändern) stehen in einem Audit-Log: wer, was, wann.

### API-Tokens

- **S-40** API-Tokens stammen aus `random_bytes` (mindestens 32 Byte), werden nur als SHA-256-Hash gespeichert, genau einmal angezeigt und sind jederzeit widerrufbar. Der Vergleich läuft über `hash_equals`.
- **S-41** Fehlversuche bei API-Tokens zählen getrennt vom Login, und ein gültiger Token funktioniert auch während einer Login-Sperre. Auch gültige Tokens haben ein Rate-Limit für teure Endpunkte.

Weitere Regeln für APIs: [eingaben-und-apis.md](eingaben-und-apis.md#eigene-apis).

## Datei-Uploads

- **S-42** Uploads haben ein Größenlimit, das in der App geprüft wird, nicht nur über `upload_max_filesize`. Der Typ wird mit `finfo` am Inhalt geprüft und gegen eine Whitelist abgeglichen; Dateiendung und der vom Browser gemeldete Typ zählen nicht.
- **S-43** Hochgeladene Dateien bekommen einen zufälligen Namen, liegen außerhalb des Web-Roots und werden über einen Controller ausgeliefert, der Rechte prüft und `X-Content-Type-Options: nosniff` sowie `Content-Disposition` setzt. Der Originalname steht nur in der Datenbank. Bilder werden neu kodiert, das entfernt eingebetteten Code und Metadaten. SVG gilt als HTML und wird nie als Bild ausgeliefert, ohne durch einen Sanitizer zu laufen.

## Secrets

- **S-44** Passwörter, Schlüssel und Tokens stehen nie im Repo – auch nicht in Tests, Beispielen, Kommentaren oder der Git-Historie.
- **S-45** Auf dem Server liegen Secrets in der Config-Datei außerhalb des Web-Roots (`G-14`), in der CI in GitHub Secrets (`H-12`).
- **S-46** Jedes Projekt hat eigene Zugangsdaten: eigener Datenbanknutzer, eigener Deploy-Schlüssel, eigene API-Keys. Kein Schlüssel öffnet zwei Projekte.
- **S-47** Ist ein Secret in ein Repo, ein Log oder einen Chat geraten, gilt es als verbrannt und wird sofort ersetzt. Löschen aus der Historie reicht nicht.
- **S-48** Daten, die das Projekt entschlüsseln können muss (z. B. Zugangsdaten zu fremden Postfächern), werden mit `sodium` verschlüsselt gespeichert; der Schlüssel liegt in der Config, nicht in der Datenbank.

## Fehler und Logs

- **S-49** In Logs stehen nie Passwörter, Tokens, Session-IDs, Setup-Codes oder vollständige Request-Bodys.
- **S-50** In Produktion enthält keine Ausgabe Fehlerdetails (`display_errors = 0`). Fehler landen in einem `error.log` im Datenverzeichnis: per SFTP lesbar, von außen über das Web gesperrt.
- **S-51** Logs werden rotiert oder in der Größe begrenzt, damit ein Fehler in einer Schleife nicht den Webspace füllt.
- **S-52** Sicherheitsrelevante Ereignisse werden protokolliert: fehlgeschlagene Logins, Sperren, Passwort-Resets, abgelehnte CSRF-Tokens, Rechteverstöße.
