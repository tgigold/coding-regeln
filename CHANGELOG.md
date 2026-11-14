# Changelog

Alle nennenswerten Änderungen an den Coding-Regeln stehen hier. Das Format folgt [Keep a Changelog](https://keepachangelog.com/de/1.1.0/), die Versionen folgen [Semantic Versioning](https://semver.org/lang/de/): Eine neue oder verschärfte Regel ist eine Minor-Version, eine gestrichene oder gelockerte eine Major-Version.

## [Unreleased]

### Hinzugefügt

- `G-18`: Öffentliche Repos enthalten keine Anweisungsdateien für KI-Assistenten (`CLAUDE.md`, `AGENTS.md`, `.claude/` …); der Verweis auf die Regeln steht in der README. Dazu ein Punkt in der Checkliste vor dem Merge.

## [1.1.0] – 2026-11-14

### Hinzugefügt

- `Q-7`: Baseline für PHPStan beim Einführen in bestehenden Code, die nur kleiner werden darf
- Vorlage `rsync-exclude.txt`: Baseline sowie Caches von PHPStan und PHPUnit werden nicht hochgeladen

### Geändert

- `Q-1`: „hohes Level“ für PHPStan als Level 8 festgelegt; ein niedrigeres Level ist eine Abweichung im Steckbrief
- Vorlage `ci.yml`: PHPStan läuft mit `--memory-limit=1G`, weil das Standardlimit bei größeren Projekten nicht reicht

## [1.0.0] – 2026-11-11

### Hinzugefügt

- Regeln auf Themen-Dateien verteilt, jede Regel mit fester Nummer
- Projektstart: Fragen, die vor der ersten Codezeile geklärt werden, und Projekt-Steckbrief
- Erlaubter Stack: Composer nach Entscheidung pro Projekt; Bootstrap, Bootstrap Icons und Google Fonts ausschließlich lokal eingebunden
- Datenschutz: keine Anfragen an Dritte, Cookies, Logs, Aufbewahrung, Export und Löschung
- Sicherheit: Header, Sessions, CSRF, Injection, Passwörter, Rechte, Nutzerverwaltung, API-Tokens, Uploads, Secrets, Logs
- Eingaben, öffentliche Formulare, eigene APIs, ausgehende Anfragen
- GitHub und Deployment: geschütztes `main`, Pull Requests, CI, Deploy per rsync über SSH, Aufgabenteilung
- Tests und Checkliste vor dem Merge
- Vorlagen: Steckbrief, CI- und Deploy-Workflow, Dependabot, `.htaccess`, `SECURITY.md`
