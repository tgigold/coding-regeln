# Tests und Checkliste

## Tests

- **Q-1** Jedes Feature bringt Tests mit: Unit- und Kernel-Tests mit PHPUnit, dazu statische Analyse mit PHPStan auf Level 8. Ein niedrigeres Level ist eine Abweichung und steht mit Begründung im Steckbrief.
- **Q-2** Sicherheitsregeln werden getestet, nicht nur umgesetzt: Es gibt Tests dafür, dass eine Route ohne Login abweist, ein POST ohne CSRF-Token scheitert, ein Nutzer fremde Datensätze nicht sieht und ungültige Eingaben mit `422` enden.
- **Q-3** Frontend-JavaScript wird mit Browser-Tests abgedeckt.
- **Q-4** Tests fragen nie echte externe Dienste an und fassen nie die echte Config oder Datenbank an. Sie laufen gegen eine frische Testdatenbank und gegen aufgezeichnete oder nachgebaute Antworten.
- **Q-5** Ein behobener Fehler bekommt einen Test, der ohne die Korrektur scheitert.
- **Q-6** Abhängigkeitsfreie Projekte dürfen PHPUnit und PHPStan trotzdem als Entwicklungspakete nutzen; sie landen nie auf dem Server (`T-3`).
- **Q-7** Wird PHPStan in bestehenden Code eingeführt, dürfen die vorhandenen Funde in eine Baseline (`phpstan-baseline.neon`). Die Baseline wird nur kleiner, nie größer: Neue Funde werden behoben, nicht eingetragen.

## Checkliste vor dem Merge

Claude geht diese Liste vor jedem Pull Request durch und nennt in der Beschreibung jeden Punkt, der nicht erfüllt ist.

- [ ] Die Änderung passt zum Steckbrief; wenn nicht, ist der Steckbrief angepasst (`P-3`)
- [ ] Tests, statische Analyse und `composer audit` sind grün (`H-8`)
- [ ] CHANGELOG hat einen Eintrag unter „Unreleased“, die README stimmt noch (`G-2`, `G-3`)
- [ ] Bei öffentlichen Repos: keine Anweisungsdateien für KI-Assistenten im Diff (`G-18`)
- [ ] Neue Routen sind in der Routenliste, hinter dem Login oder ausdrücklich als Ausnahme markiert (`A-3`)
- [ ] Jede neue Eingabe ist auf Typ, Länge und erlaubte Werte geprüft (`E-1` bis `E-6`)
- [ ] Jede neue Ausgabe ist für ihren Kontext escaped, neues SQL steht im Repository und nutzt Prepared Statements (`S-16`, `S-19`, `S-20`)
- [ ] Jeder Zugriff auf Datensätze prüft, ob dieser Nutzer dieses Objekt sehen oder ändern darf (`S-32`)
- [ ] Zustandsänderungen laufen per POST mit CSRF-Token (`A-4`, `S-12`)
- [ ] Der Browser lädt nichts von fremden Servern, es gibt keine Inline-Skripte (`D-1`, `S-4`)
- [ ] Keine Secrets im Diff, keine Secrets oder Inhalte von Nutzern in neuen Log-Zeilen (`S-44`, `S-49`)
- [ ] Neue Dateien und Verzeichnisse auf dem Server stehen, wenn sie Daten enthalten, in der Ausschlussliste des Deployments (`H-13`)
- [ ] Datenbankänderungen kommen als neue Migration, und der neue Code läuft auch noch mit dem alten Stand (`G-12`, `H-15`)
