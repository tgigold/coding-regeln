# Coding-Regeln

Diese Regeln gelten in all meinen Projekten – für mich und für Claude, der mit mir daran arbeitet. Sie sind öffentlich, damit jeder nachlesen kann, nach welchen Maßstäben die Projekte gebaut sind.

## Geltung

- Die Regeln gelten immer. Ein Projekt darf abweichen, wenn die Abweichung im [Projekt-Steckbrief](vorlagen/steckbrief.md) der Projekt-README steht und begründet ist.
- Geschrieben sind sie für PHP-Projekte auf Shared Webspace (SSH/SFTP, `.htaccess`, SQLite oder MySQL). In anderen Umgebungen gelten sie sinngemäß: Das Schutzziel bleibt, der Mechanismus darf abweichen.
- Claude fragt nach, wenn etwas unklar ist oder eine Regel im konkreten Projekt keinen Sinn ergibt, und benennt es, wenn sich Regeln und Code widersprechen.
- Jede Regel hat eine feste Nummer (z. B. `S-12`), auf die README, Pull Requests und Reviews verweisen können. Nummern werden nie neu vergeben.

## Wegweiser

| Datei | Wann lesen |
|---|---|
| [projektstart.md](projektstart.md) | **Vor der ersten Codezeile** eines neuen Projekts |
| [grundlagen.md](grundlagen.md) | Projektaufbau, Pflichtdateien, Code-Stil, Architektur |
| [stack-und-datenschutz.md](stack-und-datenschutz.md) | Composer, Bootstrap, Fonts, alles was der Browser lädt, personenbezogene Daten |
| [sicherheit.md](sicherheit.md) | Header, Sessions, Login, Nutzerverwaltung, Injection, Uploads, Secrets, Logs |
| [eingaben-und-apis.md](eingaben-und-apis.md) | Formulare, Validierung, eigene APIs, Anfragen an fremde Dienste |
| [github-und-deployment.md](github-und-deployment.md) | Repo-Setup, Branches, CI, Deployment – mit Aufgabenteilung |
| [tests-und-checkliste.md](tests-und-checkliste.md) | Tests und die Checkliste vor jedem Merge |
| [vorlagen/](vorlagen/) | Kopiervorlagen: Steckbrief, Workflows, `.htaccess`, `SECURITY.md` |

## Die Regeln in Kurzform

Die Kurzform ersetzt die Themen-Dateien nicht. Sie ist das, was immer präsent sein muss.

**Projekt**

1. Vor Projektbeginn wird der Steckbrief geklärt: Abhängigkeiten, Frontend, Datenbank, Zugang, Daten, Repo, Deployment, Ausnahmen (`P-1`).
2. `/public` ist das Web-Root, alles andere liegt außerhalb und ist zusätzlich gesperrt (`G-1`).
3. Jedes Projekt hat `README.md`, `CHANGELOG.md`, `LICENSE` und `SECURITY.md`; README und CHANGELOG werden mit jeder Änderung gepflegt (`G-2`–`G-4`).
   Öffentliche Repos enthalten keine Anweisungsdateien für KI-Assistenten wie `CLAUDE.md`; der Verweis auf diese Regeln steht in der README (`G-18`).
4. PHP ist objektorientiert, `strict_types`, nach PSR-1, PSR-4 und PER Coding Style (`G-7`, `G-8`).
5. SQL nur in Repositories, Logik in Services, Controller nur für Request und Response (`A-1`).

**Stack und Datenschutz**

6. Composer ist erlaubt, wo es Sinn ergibt; die Entscheidung fällt im Steckbrief (`T-1`).
7. Bootstrap, Bootstrap Icons und Google Fonts sind erlaubt, aber ausschließlich lokal eingebunden. Der Browser lädt nichts von fremden Servern (`T-4`, `D-1`).

**Sicherheit**

8. Jede Route liegt standardmäßig hinter dem Login, jede Zustandsänderung ist ein POST mit CSRF-Token (`A-3`, `S-12`).
9. Datenbankzugriffe nur über Prepared Statements, jede Ausgabe wird für ihren Kontext escaped (`S-16`, `S-19`).
10. Eingaben werden serverseitig geprüft: Typ, Länge, Format, Whitelist (`E-1`–`E-5`).
11. Passwörter mit Argon2id, Rechte werden bei jedem Request und für jedes Objekt geprüft (`S-27`, `S-32`).
12. HTTPS, HSTS, strikte CSP ohne Inline-Skripte, sichere Session-Cookies (`S-1`–`S-8`).
13. Secrets stehen nie im Repo und nie im Log (`S-44`, `S-49`).

**GitHub und Deployment**

14. Jede Änderung läuft über Feature-Branch und Pull Request; `main` ist geschützt (`H-3`, `H-4`).
15. Jeder Merge auf `main` geht nach grüner CI automatisch live, per rsync über SSH (`H-9`, `H-11`).
16. Das Deployment fasst Config, Datenbank, Uploads und Backups auf dem Server nie an (`H-13`).

## Für Claude

- Neues Projekt: zuerst [projektstart.md](projektstart.md) durchgehen und den Steckbrief mit mir klären.
- Vor Arbeit an einem Thema die passende Datei aus dem Wegweiser lesen, nicht aus dem Gedächtnis arbeiten.
- Vor jedem Pull Request die [Checkliste](tests-und-checkliste.md#checkliste-vor-dem-merge) durchgehen.
- Bestehender Code, der gegen eine Regel verstößt, wird benannt, aber nicht ungefragt umgebaut.

## Lizenz und Änderungen

Die Regeln stehen unter der [Unlicense](LICENSE). Änderungen stehen im [CHANGELOG](CHANGELOG.md), Versionen folgen [Semantic Versioning](https://semver.org/lang/de/): Eine neue oder verschärfte Regel ist eine Minor-Version, eine gestrichene oder gelockerte eine Major-Version.
