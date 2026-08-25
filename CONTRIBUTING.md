# Mitwirken

Dieses Repository enthält verbindliche Richtlinien. Jede Änderung wirkt auf alle Projekte,
die sie anwenden — deshalb ist der Weg hierher etwas formeller als in einem Code-Repository.

## Eine neue Richtlinie vorschlagen

Leg ein **Issue** mit der Vorlage *Richtlinie* an. Beschreibe darin nicht nur die Regel,
sondern das Problem, das sie löst. Eine Richtlinie ohne erkennbares Problem wird zu einer
Regel, die niemand versteht und irgendwann alle umgehen.

Was eine brauchbare Richtlinie ausmacht:

- **Prüfbar.** Man kann an einem konkreten Pull Request feststellen, ob sie eingehalten
  wurde. „Sauberen Code schreiben" ist keine Richtlinie.
- **Begründet.** Die Begründung steht dabei, nicht nur die Regel. Wer den Grund kennt,
  erkennt auch die berechtigte Ausnahme.
- **Widerspruchsfrei.** Sie steht nicht gegen eine bestehende Regel. Wenn doch, gehört die
  alte mit auf den Tisch.

## Änderungen einreichen

Ein Pull Request pro Richtlinie, mit Bezug auf das Issue. Er braucht **zwei Freigaben** —
die Punkte, auf die die Reviewer schauen, stehen in der PR-Vorlage.

**Die Checkliste mit anpassen.** `ki-reviewer/review-checkliste.md` ist die Kurzfassung,
gegen die der Skill [`drupal-review`](https://github.com/Effective-Bytes/drupal-review)
tatsächlich prüft. Eine Richtlinie, die dort nicht auftaucht, wird nicht durchgesetzt —
also entweder ergänzen oder im PR begründen, warum sie dort nicht hingehört.

Sprache ist **Deutsch**, in den Richtlinien wie in der Checkliste. Codebeispiele und
Bezeichner darin bleiben englisch.

## Was nicht ins Repository gehört

Dies ist ein öffentliches Repository. Nicht hinein gehören **Kundennamen, interne
Repository-Pfade und Ticketnummern** — auch nicht in Issues, Kommentaren oder
Commit-Messages. GitHub gibt editierte Fassungen über die Edit-Historie weiter heraus;
Umformulieren im Nachhinein bereinigt nichts. Ein Beispiel aus einem echten Projekt wird
anonymisiert, bevor es hier landet.

## Lizenz

Mit deinem Beitrag stimmst du zu, dass er unter der [Apache License 2.0](LICENSE)
veröffentlicht wird.
