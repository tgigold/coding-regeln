# Grundlagen und Architektur

## Projektaufbau

- **G-1** `/public` ist das Web-Root. Alle anderen Dateien (Code, Config, Daten, Templates, `vendor/`) liegen außerhalb und sind zusätzlich per `.htaccess` gegen direkten Zugriff gesperrt ([Vorlage](vorlagen/htaccess-deny)) – für den Fall, dass das Document Root doch aufs Projektverzeichnis zeigt.
- **G-2** Jedes Projekt hat eine ausführliche `README.md`, sauber und menschenlesbar. Sie beginnt mit dem Projekt-Steckbrief und beschreibt Installation, Konfiguration, Betrieb (Backup, Updates), Deployment und lokale Entwicklung. Sie wird mit jedem Feature mitgepflegt.
- **G-3** Jedes Projekt hat eine `CHANGELOG.md` nach [Keep a Changelog](https://keepachangelog.com/de/1.1.0/). Jede Änderung landet im selben Pull Request unter „Unreleased“, nicht erst beim Release.
- **G-4** Weitere Pflichtdateien: `LICENSE` (bei öffentlichen Projekten), `SECURITY.md` ([Vorlage](vorlagen/SECURITY.md)), `.gitignore`, `.editorconfig` und – sobald fremder Code oder fremde Schriften mit ausgeliefert werden – `THIRD-PARTY-LICENSES.md`.
- **G-5** Versionierung nach [Semantic Versioning](https://semver.org/lang/de/). Die Version steht an genau einer Stelle im Code.
- **G-6** Öffentliche Projekte stehen unter der [Unlicense](https://unlicense.org/), sofern der Steckbrief nichts anderes sagt. Lizenzen mitgelieferter Bibliotheken und Schriften (MIT, OFL …) bleiben davon unberührt und werden in `THIRD-PARTY-LICENSES.md` genannt.

## Code-Stil

- **G-7** PHP wird objektorientiert geschrieben, jede Datei beginnt mit `declare(strict_types=1);`. Parameter, Rückgaben und Eigenschaften sind typisiert.
- **G-8** Es gelten [PSR-1](https://www.php-fig.org/psr/psr-1/), [PSR-4](https://www.php-fig.org/psr/psr-4/) und der [PER Coding Style](https://www.php-fig.org/per/coding-style/) (Nachfolger von PSR-12). Wo ein Projekt Logger, HTTP-Nachrichten oder Container braucht, richtet es sich nach PSR-3, PSR-7 und PSR-11.
- **G-9** Namen sind englisch. Klassen `StudlyCaps`, Methoden und Variablen `camelCase`, Konstanten `UPPER_SNAKE_CASE`, Datenbanktabellen und -spalten `snake_case`. Der Namespace folgt dem Muster `Vendor\Projekt\…`.
- **G-10** Kommentare erklären das Warum, nicht das Was. Jede Klasse hat einen Satz dazu, wofür sie da ist; jede nicht offensichtliche Entscheidung einen Kommentar an der Stelle, an der sie getroffen wird.

## Installation, Updates, Konfiguration

- **G-11** Jedes Projekt mit Web-Oberfläche hat ein geführtes Setup, das eine Installation auf einem Webhoster ohne CLI erlaubt. Beim ersten Aufruf erzeugt es einen Setup-Code, der nur per SFTP lesbar ist und eingegeben werden muss – sonst kann zwischen Deploy und Setup jeder die Instanz übernehmen. Nach erfolgreichem Setup sperrt sich die Setup-Route selbst.
- **G-12** Datenbankänderungen laufen über nummerierte Migrationen: jede in einer eigenen Transaktion, vor jedem Lauf ein automatisches Backup, ausführbar im Browser und per CLI. Im Browser nur für Admins, per POST mit CSRF-Token. Bereits ausgerollte Migrationen werden nie geändert, es gibt immer eine neue.
- **G-13** Konfiguration mit Defaults im Code: Fehlende Schlüssel fallen auf Defaults zurück, damit Updates keine Config-Anpassung erzwingen. Alltagseinstellungen liegen in der App, nicht in der Config-Datei.
- **G-14** Die Config-Datei mit Zugangsdaten liegt außerhalb des Web-Roots und nie im Repo. Im Repo liegt eine `config.example.php` ohne echte Werte.

## Datenbank

- **G-15** SQLite: Die Datenbankdatei liegt außerhalb des Web-Roots, `PRAGMA foreign_keys = ON` ist gesetzt, der Journal-Modus ist WAL.
- **G-16** MySQL/MariaDB: Zeichensatz `utf8mb4`, Strict Mode, ein eigener Datenbanknutzer pro Projekt mit nur den nötigen Rechten.
- **G-17** PDO läuft mit `ERRMODE_EXCEPTION` und ohne emulierte Prepared Statements (`ATTR_EMULATE_PREPARES = false`).

## Architektur

- **A-1** SQL gibt es nur in Repositories, Logik ohne HTTP-Bezug in Services, Controller lesen den Request und geben eine Response zurück.
- **A-2** Ziel ist, dass neue Features ohne Umbau bestehenden Codes auskommen. Wo ein Feature einen Umbau erzwingt, kommt der Umbau als eigener Pull Request zuerst.
- **A-3** Alle Routen stehen an einem Ort. Jede Route liegt standardmäßig hinter dem Login und jeder POST braucht ein CSRF-Token; Ausnahmen (öffentlich, Gast, API) werden pro Route explizit markiert.
- **A-4** GET-Anfragen ändern nie Zustand. Löschen, Abmelden, Umschalten und alles andere mit Wirkung läuft per POST.
- **A-5** In URLs ist die ID maßgeblich, der Slug nur Kosmetik. Fehlende, veraltete oder falsche Slugs leiten per 301 auf die korrekte URL um, damit Links dauerhaft gültig bleiben.
- **A-6** Asset-URLs tragen die App-Version, damit Browser nach einem Update keine alten Dateien aus dem Cache verwenden.
