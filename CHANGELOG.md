# Changelog

Alle nennenswerten Änderungen an den Coding-Regeln stehen hier. Das Format folgt [Keep a Changelog](https://keepachangelog.com/de/1.1.0/), die Versionen folgen [Semantic Versioning](https://semver.org/lang/de/): Eine neue oder verschärfte Regel ist eine Minor-Version, eine gestrichene oder gelockerte eine Major-Version.

## [Unreleased]

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
