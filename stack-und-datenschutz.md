# Stack und Datenschutz

## Erlaubter Stack

### Composer

- **T-1** Composer-Pakete sind erlaubt, wo sie Sinn ergeben. Ob ein Projekt Composer nutzt oder abhängigkeitsfrei bleibt, steht im [Steckbrief](projektstart.md#1-abhängigkeiten).
- **T-2** Ein Paket kommt nur ins Projekt, wenn es aktiv gepflegt wird, seine Lizenz zum Projekt passt und es eine Aufgabe löst, die man nicht in wenigen Zeilen sicher selbst löst. Sicherheitskritisches (Krypto, Sanitizer, Mail, OAuth) wird nie selbst gebaut, wenn es ein etabliertes Paket gibt.
- **T-3** `composer.lock` ist eingecheckt, `vendor/` nicht. Die CI baut mit `composer install --no-dev --optimize-autoloader` und prüft mit `composer audit`; ein Paket mit bekannter Sicherheitslücke blockiert den Merge.

### Frontend

- **T-4** Erlaubt sind [Bootstrap](https://getbootstrap.com/) (CSS und JS), [Bootstrap Icons](https://icons.getbootstrap.com/) und Schriften von [Google Fonts](https://fonts.google.com/). Sie werden immer vom eigenen Server ausgeliefert, nie per CDN und nie von `fonts.googleapis.com` oder `fonts.gstatic.com`.
- **T-5** Fremde Dateien liegen unter `/public/assets/vendor/<name>-<version>/`. Die Version im Pfad macht Updates sichtbar und sorgt dafür, dass alte Dateien nicht aus dem Cache kommen.
- **T-6** Schriften werden nur als `woff2` ausgeliefert, mit `font-display: swap` und nur in den Schnitten, die das Projekt nutzt. Die Lizenzdatei der Schrift liegt im selben Ordner.
- **T-7** Fremde Skripte und Stylesheets werden mit Subresource Integrity eingebunden, auch wenn sie lokal liegen – eine veränderte Datei fällt so sofort auf.
- **T-8** Andere Bibliotheken werden nicht stillschweigend eingeführt. Claude schlägt sie vor, ich entscheide, die Entscheidung steht im Steckbrief.
- **T-9** Ein Build-Schritt für das Frontend (npm, Bundler) ist die Ausnahme und steht im Steckbrief. Wenn es ihn gibt, ist `package-lock.json` eingecheckt und der Build läuft in der CI, nicht auf dem Server.

## Datenschutz

- **D-1** Beim Aufruf einer Seite fragt der Browser nur den eigenen Server an. Keine Schriften, Icons, Skripte, Bilder, Karten, Videos oder Zählpixel von Dritten.
- **D-2** Inhalte Dritter (Videos, Karten, Social-Media-Beiträge) werden nur nach einem Klick geladen. Vorher steht dort ein Platzhalter, der sagt, wer nach dem Klick welche Daten bekommt.
- **D-3** Es werden nur technisch notwendige Cookies gesetzt. Damit braucht kein Projekt ein Cookie-Banner.
- **D-4** Reichweitenmessung, falls gewünscht, läuft auf dem eigenen Server, ohne Cookies und ohne vollständige IP-Adressen.
- **D-5** Datensparsamkeit: Es wird nur erhoben, was das Feature braucht. Jedes Feld mit Personenbezug hat einen benennbaren Zweck.
- **D-6** IP-Adressen werden in Logs gekürzt (IPv4 auf /24, IPv6 auf /48). Vollständig gespeichert werden sie nur dort, wo die Sicherheit es braucht (Login-Sperre), und dort höchstens für die Dauer der Sperre.
- **D-7** Logs und andere personenbezogene Daten haben eine Aufbewahrungsfrist, nach der sie automatisch gelöscht werden. Die Frist steht in der README.
- **D-8** Projekte mit Nutzerkonten sehen von Anfang an vor, dass ein Nutzer seine Daten exportieren und sein Konto samt Daten löschen kann.
- **D-9** E-Mails an Nutzer enthalten keine Zählpixel und keine umgeschriebenen Tracking-Links.
- **D-10** Öffentliche Projekte mit eigener Domain haben Impressum und Datenschutzerklärung, die beschreiben, was tatsächlich passiert – einschließlich der fremden Dienste, die der Server anfragt.
