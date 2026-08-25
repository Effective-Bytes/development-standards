# Effective Bytes Richtlinien

Verbindliche Vorgaben für die Drupal-Entwicklung in der Agentur Effective Bytes.

Jeder Entwickler ist dazu angehalten, sich an diese zu halten, und Reviewer sind dazu
angehalten sie zu kontrollieren. Abweichungen von den Standards sollten begründet sein.

Wir veröffentlichen die Richtlinien, weil sie ohne den zugehörigen Reviewer nur halb
nützlich sind — und umgekehrt. Wer sie für eigene Projekte übernehmen oder anpassen will,
darf das (siehe [Lizenz](#lizenz)); wir freuen uns über Rückmeldungen, sind aber nicht der
Meinung, dass unsere Vorgaben für jedes Team die richtigen sind.

## Inhalt

### Richtlinien

- [Contrib First Policy](richtlinien/contrib-first.md)
- [Coding Standards](richtlinien/coding-standards.md)
- [Projektstruktur](richtlinien/projektstruktur.md)
- [Themes und Styling](richtlinien/themes-styling.md)
- [Architektur: Controller, Hooks, Events, Drush Generate, SDC](richtlinien/architektur.md)
- [Modul-Entwicklung](richtlinien/modul-entwicklung.md)
- [Testing & Qualitätssicherung](richtlinien/testing.md)
- [Barrierefreiheit](richtlinien/barrierefreiheit.md)
- [Logging](richtlinien/logging.md)
- [Patches](richtlinien/patches.md)
- [Git](richtlinien/git.md)
- [Assets](richtlinien/assets.md)

### KI-Reviewer

- [Review-Checkliste (Kurzfassung aller Richtlinien)](ki-reviewer/review-checkliste.md)

## Zusammenspiel mit dem Review-Skill

Die Checkliste unter `ki-reviewer/` ist der durchsetzbare Teil: sie ist die Quelle, gegen
die der Claude-Code-Skill
[`drupal-review`](https://github.com/Effective-Bytes/drupal-review) Pull Requests prüft.
Der Skill lädt dieses Repository zur Laufzeit; was hier nicht steht, wird nicht geprüft.

Wer den Skill mit eigenen Richtlinien betreiben will, hinterlegt sie in einem Repository
mit derselben Struktur (`ki-reviewer/review-checkliste.md` plus `richtlinien/*.md`) und
zeigt über `STANDARDS_REPO` darauf — Details in der README des Skills.

## Wie man mit diesem Repository arbeitet

Für jede neue Idee oder Ergänzung wird ein **Issue** angelegt; für Richtlinien gibt es
dafür eine eigene Vorlage. Änderungen laufen über einen Pull Request und brauchen **zwei
Freigaben**, weil eine Richtlinie alle Projekte betrifft.

Wird eine Richtlinie ergänzt oder geändert, gehört die Checkliste unter `ki-reviewer/`
mit angepasst — sonst prüft der Reviewer weiter den alten Stand.

Näheres in [CONTRIBUTING.md](CONTRIBUTING.md).

## Lizenz

[Apache License 2.0](LICENSE).
