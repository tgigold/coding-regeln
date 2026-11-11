# GitHub und Deployment

Alle Projekte liegen auf GitHub und werden von dort deployed. Niemand lädt von Hand Dateien auf den Server.

- [Repository](#repository)
- [Branches und Pull Requests](#branches-und-pull-requests)
- [CI](#ci)
- [Deployment](#deployment)
- [Einrichtung: wer macht was](#einrichtung-wer-macht-was)
- [Laufender Betrieb: wer macht was](#laufender-betrieb-wer-macht-was)

## Repository

- **H-1** Ein Repo pro Projekt, Default-Branch `main`. Ein Repo ist privat, bis es bewusst veröffentlicht wird; vor dem Veröffentlichen wird die gesamte Historie auf Secrets und private Daten geprüft.
- **H-2** Im Repo liegen nie: `vendor/`, `node_modules/`, Config mit Zugangsdaten, Datenbanken, Uploads, Logs, Backups. Die `.gitignore` steht, bevor der erste Commit entsteht.

## Branches und Pull Requests

- **H-3** Jede Änderung entsteht auf einem eigenen Branch (`feature/…`, `fix/…`) und kommt per Pull Request nach `main`. Niemand pusht direkt auf `main`.
- **H-4** `main` ist per Ruleset geschützt: Pull Request nötig, CI muss grün sein, kein Force-Push, kein Löschen. Gemergt wird per Squash, der Branch wird danach gelöscht.
- **H-5** Bei privaten Repos im kostenlosen GitHub-Tarif gibt es weder Rulesets noch Environments. Dann gelten `H-3` und `H-4` als Absprache, die Deploy-Secrets liegen als Repository-Secrets vor, und der Steckbrief vermerkt das.
- **H-6** Ein Pull Request macht eine Sache. Er enthält Code, Tests, CHANGELOG-Eintrag und, wo nötig, die README-Änderung. Die Beschreibung sagt, was sich ändert, warum, und wie es geprüft wurde.
- **H-7** Commit-Betreffzeilen stehen im Imperativ und sagen, was der Commit tut („Füge Login-Sperre hinzu“).

## CI

Kopiervorlage: [vorlagen/ci.yml](vorlagen/ci.yml)

- **H-8** Die CI läuft bei jedem Pull Request: Syntax-Check, Tests, statische Analyse, `composer audit`, wo vorhanden Browser-Tests (`Q-1` bis `Q-4`).
- **H-9** Deployed wird nur automatisch und nur nach grüner CI. Der Deploy-Workflow ruft die CI selbst noch einmal auf dem gemergten Stand auf.
- **H-10** Actions werden auf Commit-SHAs gepinnt, Workflows bekommen minimale Rechte (`permissions: contents: read`), Lockfiles sind eingecheckt. Dependabot hält `composer` und `github-actions` aktuell ([Vorlage](vorlagen/dependabot.yml)); Secret Scanning mit Push Protection ist eingeschaltet, wo der Tarif es erlaubt.

## Deployment

Kopiervorlage: [vorlagen/deploy.yml](vorlagen/deploy.yml)

- **H-11** Jeder Merge auf `main` geht live. `main` ist damit immer der Stand auf dem Server. Übertragen wird per rsync über SSH; die CI baut vorher `vendor/` ohne Entwicklungspakete.
- **H-12** Die Zugangsdaten liegen im GitHub Environment `production`: `SSH_HOST`, `SSH_USER`, `SSH_KEY`, `SSH_KNOWN_HOSTS`, `DEPLOY_PATH`. Jedes Projekt hat ein eigenes Schlüsselpaar nur für das Deployment (`S-46`). Der Host-Key des Servers ist hinterlegt; der Workflow verbindet sich nie mit abgeschalteter Host-Prüfung.
- **H-13** Das Deployment löscht nur Dateien, die es selbst hochgeladen hat. Config, Datenbank, Uploads, Logs und Backups auf dem Server stehen in der Ausschlussliste `deploy/rsync-exclude.txt` ([Vorlage](vorlagen/rsync-exclude.txt)) und bleiben unangetastet.
- **H-14** Es läuft immer nur ein Deployment gleichzeitig (`concurrency`). Nach dem Übertragen ruft der Workflow eine Health-Check-URL auf; antwortet sie nicht mit `200`, schlägt der Workflow sichtbar fehl.
- **H-15** Migrationen werden nach dem Deployment bewusst ausgelöst (`G-12`), nie stillschweigend vom Deploy. Ein Deployment muss mit dem alten Datenbankstand lauffähig bleiben, bis die Migration gelaufen ist.
- **H-16** Rollback ist ein Revert-Pull-Request, der wie jede Änderung durch die CI geht. Für die Datenbank gibt es das Backup, das vor jeder Migration entsteht.
- **H-17** Wie ein Projekt deployed wird, steht in seiner README: Zielserver, Pfad, Health-Check-URL, wer Zugriff hat, wie man zurückrollt.

## Einrichtung: wer macht was

Die Schritte laufen in dieser Reihenfolge. Claude erledigt alles, was sich per `git` und `gh` erledigen lässt; bei mir bleibt, was Zugangsdaten, Geld oder die Außenwirkung betrifft.

| # | Schritt | Claude | Ich |
|---|---|---|---|
| 1 | Steckbrief | stellt die Fragen aus [projektstart.md](projektstart.md), schreibt den Steckbrief in die README | beantworte sie |
| 2 | Repo anlegen | `.gitignore`, `git init`, erster Commit, `gh repo create` (privat, sofern nicht anders entschieden) | bestätige Name und Sichtbarkeit, bevor das Repo entsteht |
| 3 | Repo absichern | Ruleset für `main`, nur Squash-Merge, Branch nach Merge löschen, Dependabot, Secret Scanning – alles per `gh api` | 2FA am GitHub-Account (einmalig) |
| 4 | Server vorbereiten | nennt mir, was der Server braucht: Zielpfad, PHP-Version, Document Root auf `/public` | lege Domain und Verzeichnis beim Hoster an, schalte SSH frei |
| 5 | Deploy-Schlüssel | erzeugt ein Schlüsselpaar nur für dieses Projekt, zeigt mir den öffentlichen Teil, löscht den privaten nach Schritt 6 von der Platte | hinterlege den öffentlichen Schlüssel auf dem Server (`~/.ssh/authorized_keys`) |
| 6 | Secrets | legt das Environment `production` an, setzt die Secrets per `gh secret set`, ermittelt und hinterlegt den Host-Key | liefere Host, Benutzer und Zielpfad; gleiche den Fingerabdruck des Host-Keys mit der Angabe des Hosters ab |
| 7 | Workflows | `ci.yml`, `deploy.yml`, `dependabot.yml` und `deploy/rsync-exclude.txt` aus den Vorlagen, als erster Pull Request | prüfe und merge den Pull Request |
| 8 | Erster Deploy | beobachtet den Workflow, prüft den Health-Check, behebt Fehler per Pull Request | – |
| 9 | Inbetriebnahme | – | lese den Setup-Code per SFTP, schließe das Setup im Browser ab (`G-11`) |
| 10 | Abschluss | trägt Deployment und Betrieb in die README ein (`H-17`) | lege Cronjobs beim Hoster an, falls nötig |

Claude fasst Zugangsdaten nur dort an, wo es die Tabelle vorsieht, gibt sie nie in Chat, Commit oder Log aus und fragt vor jedem Schritt, der nach außen sichtbar ist (Repo anlegen, Sichtbarkeit ändern, erster Deploy).

## Laufender Betrieb: wer macht was

| Anlass | Claude | Ich |
|---|---|---|
| Neue Aufgabe | Branch, Code, Tests, CHANGELOG, Pull Request mit Beschreibung | Review und Merge – der Merge ist die Freigabe zum Live-Gang |
| CI rot | findet die Ursache und behebt sie im selben Pull Request | – |
| Dependabot-PR | prüft Changelog des Pakets und CI-Ergebnis, empfiehlt Merge oder Abwarten | merge |
| Migration nötig | schreibt die Migration, weist im Pull Request darauf hin | löse sie nach dem Deploy aus |
| Deploy fehlgeschlagen | liest das Workflow-Log, schlägt Fix oder Revert vor | entscheide |
| Secret verbrannt | benennt, wo es überall hinterlegt ist, und setzt das neue in GitHub | erzeuge das neue Secret beim jeweiligen Dienst |
| Release | setzt Version, schließt den CHANGELOG-Abschnitt, erstellt nach dem Merge Tag und GitHub-Release | gebe die Versionsnummer frei |
