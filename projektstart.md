# Projektstart

Diese Fragen werden geklärt, bevor Code entsteht. Sie entscheiden, welche Regeln in welcher Form greifen – und sie nachträglich zu ändern ist teuer.

- **P-1** Vor der ersten Codezeile geht Claude die Fragen unten mit mir durch. Wo die Antwort aus der Aufgabe schon hervorgeht, schlägt Claude sie vor, statt zu fragen.
- **P-2** Die Antworten stehen als **Projekt-Steckbrief** oben in der Projekt-README ([Vorlage](vorlagen/steckbrief.md)).
- **P-3** Ändert sich später eine Antwort, wird zuerst der Steckbrief geändert, dann der Code. Der Steckbrief beschreibt immer den Ist-Stand.
- **P-4** Abweichungen von den Regeln stehen im Steckbrief mit Regelnummer und Begründung. Was dort nicht steht, gilt.

## Die Fragen

### 1. Abhängigkeiten

Mit Composer oder abhängigkeitsfrei?

- **Mit Composer**, wenn das Projekt etwas braucht, das man nicht sicher selbst baut: Mailversand, IMAP, OAuth, 2FA, PDF, HTML-Sanitizer, Feed-Parser. Es gelten `T-1` bis `T-3`.
- **Abhängigkeitsfrei**, wenn das Projekt klein ist, lange ohne Pflege laufen soll oder per Kopieren installierbar sein muss. Dann bringt es einen eigenen PSR-4-Autoloader mit.
- Entwicklungswerkzeuge (PHPUnit, PHPStan) per Composer sind in beiden Fällen erlaubt, sie landen nie auf dem Server.

Im Steckbrief steht die Entscheidung und, bei Composer, die absehbaren Pakete.

### 2. Frontend

Bootstrap mit Bootstrap Icons, eigenes CSS oder beides? Welche Schrift? Braucht die Seite JavaScript, und wenn ja wofür?

### 3. Datenbank

SQLite, MySQL/MariaDB oder keine. SQLite ist der Default für Einzelnutzer-Werkzeuge; MySQL/MariaDB bei mehreren gleichzeitig schreibenden Nutzern, Volltextsuche oder großen Datenmengen.

### 4. Zugang

- Öffentlich lesbar, hinter Login oder gemischt?
- Ein Nutzer oder mehrere? Gibt es Rollen, Selbstregistrierung, Einladungen?
- Gibt es eine API? Für wen: eigene Clients, Kurzbefehle, Dritte?

Davon hängt ab, welche Teile von [sicherheit.md](sicherheit.md) und [eingaben-und-apis.md](eingaben-und-apis.md) gebraucht werden.

### 5. Daten

- Werden personenbezogene Daten verarbeitet? Welche, wozu, wie lange?
- Gibt es Datei-Uploads? Welche Typen, wie groß?
- Welche fremden Dienste werden angefragt?

### 6. Repo und Lizenz

- Öffentlich oder privat? Default ist privat, bis das Projekt bewusst veröffentlicht wird.
- Lizenz: Default ist die Unlicense. Ein privates Projekt ohne Veröffentlichungsabsicht bekommt keine Lizenz.
- Hinweis: Geschützte Branches, Environments und Secret Scanning stehen bei privaten Repos nur in bezahlten GitHub-Tarifen zur Verfügung. Was dann ersatzweise gilt, steht in `H-5`.

### 7. Deployment

- Zielserver, Zielpfad, Domain, PHP-Version auf dem Server
- Standard ist rsync über SSH und „`main` ist live“. Abweichungen (Staging, Deploy nur per Tag, FTPS) stehen im Steckbrief.
- Laufen Cronjobs? Wer richtet sie ein?

### 8. Ausnahmen

Welche Regeln gelten in diesem Projekt bewusst nicht, und warum? Typische Fälle: ein reines Skript ohne Web-Oberfläche braucht kein Setup (`G-11`), eine statische Seite keine Migrationen (`G-12`).

## Danach

Steht der Steckbrief, geht es mit dem Repo weiter: [github-und-deployment.md](github-und-deployment.md#einrichtung-wer-macht-was).
