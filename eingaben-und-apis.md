# Eingaben, APIs und ausgehende Anfragen

## Eingaben validieren

Als Eingabe zählt alles, was von außen kommt: Formularfelder, Query-Parameter, Header, Cookies, JSON-Bodys, hochgeladene Dateien, Antworten fremder Dienste.

- **E-1** Validiert wird immer auf dem Server. Prüfungen im Browser sind Komfort, kein Schutz.
- **E-2** Typen werden explizit geprüft. Ein Feld, das Text erwartet, lehnt Arrays und andere Typen mit einer klaren Fehlermeldung ab, statt „Array“ zu speichern.
- **E-3** Jedes Feld hat eine Höchstlänge, jede Liste eine Höchstzahl an Einträgen, jede Anfrage eine Höchstgröße.
- **E-4** Erlaubt ist, was auf der Liste steht (Whitelist). Auswahlfelder, Status, Sortierung und Ähnliches werden gegen die festen erlaubten Werte geprüft, nicht gegen eine Liste verbotener.
- **E-5** Texteingaben werden auf gültiges UTF-8 geprüft, bevor sie in die Datenbank gehen. Sonst scheitern später JSON-Ausgaben und Exporte dauerhaft an einem einzigen kaputten Datensatz. Steuerzeichen außer Zeilenumbruch und Tabulator werden abgelehnt.
- **E-6** Es werden nur die Felder übernommen, die das Formular vorsieht. Request-Daten werden nie als Ganzes in ein Objekt oder einen Datensatz geschrieben – sonst setzt ein zusätzliches Feld wie `role=admin` Rechte.
- **E-7** Zahlen und Zeitpunkte haben Plausibilitätsgrenzen (z. B. Zeitpunkte zwischen 2000 und morgen). Formate, die doppeldeutig sind (achtstellige Zahl: Datum oder Unix-Zeit?), werden eindeutig aufgelöst.
- **E-8** E-Mail-Adressen werden mit `filter_var(…, FILTER_VALIDATE_EMAIL)` geprüft, URLs zusätzlich auf Schema `http`/`https` (`S-21`).
- **E-9** Validierungsfehler (`422`) sagen, welche Felder in welcher Form angekommen sind – Feldnamen und Typen, nie Inhalte. Das macht falsch verdrahtete Clients (z. B. Kurzbefehle) sofort auffindbar.
- **E-10** Eingaben werden so gespeichert, wie sie kamen, und bei der Ausgabe escaped (`S-19`). Es wird nicht beim Speichern „bereinigt“.

## Öffentliche Formulare

Für Formulare, die ohne Login erreichbar sind: Kontakt, Registrierung, Kommentare, Passwort-Reset.

- **E-11** Schutz vor ungewollten Einsendungen ohne fremde Dienste: ein verstecktes Honeypot-Feld, eine Zeitfalle (zu schnell ausgefüllt gilt als Bot) und ein Rate-Limit pro IP. Kein Captcha von Dritten (`D-1`).
- **E-12** Was in E-Mail-Header wandert (Absendername, Betreff, Reply-To), darf keine Zeilenumbrüche enthalten. Der Absender einer vom Server verschickten Mail ist immer eine eigene Adresse; die Adresse des Besuchers steht höchstens im `Reply-To`.
- **E-13** Öffentliche Formulare verschicken keine Mails an frei eingebbare Adressen mit frei eingebbarem Inhalt – sonst wird das Formular zur Spam-Schleuder.
- **E-14** Erkannte Bot-Einsendungen bekommen dieselbe Erfolgsmeldung wie echte, werden aber verworfen.

## Eigene APIs

- **API-1** Jeder Endpunkt verlangt Authentifizierung, außer er ist in der Routenliste ausdrücklich als öffentlich markiert (`A-3`). Für Tokens gelten `S-40` und `S-41`.
- **API-2** Tokens stehen im `Authorization`-Header, nie in der URL. Wo ein Projekt mehrere Tokens kennt, haben sie Rechte-Bereiche (z. B. nur lesen), und jeder Token gehört zu einem Nutzer, dessen Rechte er nie übersteigt (`S-32`).
- **API-3** Der Pfad trägt die Version (`/api/v1/…`). Bestehende Versionen ändern ihr Verhalten nicht inkompatibel.
- **API-4** Endpunkte, die JSON erwarten, lehnen andere `Content-Type`-Werte mit `415` ab. Antworten sind immer JSON mit `Content-Type: application/json; charset=utf-8`.
- **API-5** Fehler haben ein einheitliches Format (Statuscode, maschinenlesbarer Code, Meldung, bei `422` die Felder nach `E-9`). Statuscodes werden sauber genutzt: `401` nicht angemeldet, `403` keine Berechtigung, `404` nicht vorhanden oder nicht sichtbar (`S-33`), `422` ungültige Eingabe, `429` Rate-Limit.
- **API-6** CORS ist standardmäßig geschlossen. Wird es gebraucht, steht dort eine feste Liste erlaubter Origins – nie `*` zusammen mit Credentials.
- **API-7** Listen sind paginiert, die Seitengröße hat eine Obergrenze. Jeder Endpunkt hat ein Rate-Limit; `429`-Antworten nennen im `Retry-After`-Header, wann es weitergeht.
- **API-8** Antworten enthalten nur die Felder, die der Client braucht – nie ganze Datensätze mit Passwort-Hashes, Tokens oder internen Feldern.
- **API-9** Die API ist in der README dokumentiert: Endpunkte, Parameter, Beispiel-Anfrage und -Antwort, Fehlercodes.

## Ausgehende Anfragen

- **O-1** Ausgehende Anfragen laufen nur über HTTPS, mit geprüftem Zertifikat. Weiterleitungen werden nur per HTTPS und nur begrenzt oft gefolgt.
- **O-2** Jede Anfrage hat ein Timeout für Verbindung und Antwort sowie eine Höchstgröße für die Antwort.
- **O-3** API-Keys gehen im Header mit, nie in der URL (sonst landen sie in Logs und Fehlermeldungen), und werden nie an einen anderen Host geschickt, auch nicht bei Redirects.
- **O-4** Vom Nutzer konfigurierbare Endpunkte dürfen nicht auf interne Adressen zeigen (SSRF): Der Hostname wird aufgelöst, private, Loopback- und Link-Local-Adressen werden abgelehnt, und die Verbindung geht an genau die geprüfte Adresse.
- **O-5** Antworten fremder Dienste sind Eingaben und werden wie solche geprüft (`E-2` bis `E-5`).
- **O-6** Fair Use gegenüber fremden Diensten: Alle Anfragen laufen über den Server, tragen einen User-Agent mit Version und Kontaktadresse, werden mit TTL gecacht und wo nötig gedrosselt.
- **O-7** Fallback-Ketten haben ein Gesamt-Zeitbudget pro Request. Ein ausgefallener Dienst wird eine Zeit lang übersprungen, aber erst nach mehreren Komplettausfällen – ein einzelner Timeout sperrt nichts.
