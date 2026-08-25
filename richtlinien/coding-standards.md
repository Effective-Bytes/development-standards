# Allgemeine Coding Standards, Best Practices und Grundsätze

Um nicht alle Standards auswendig lernen zu müssen, ist es dringend empfohlen, die genannten oder ähnliche Tools zu verwenden.

Die Standards sorgen in erster Linie dafür, dass wir unseren Code einheitlich formatieren. Simple „Code-Smells" wie unbenutzte Variablen werden hier auch erkannt.

Die Drupal Coding Standards halten sich größtenteils an die Standards der jeweiligen Sprachen (z. B. an die offiziellen PHP-, JS-, CSS- oder HTML-Standards).

Wo jedoch der Code hinkommt und wie wir den Code strukturieren bzw. wie die Code-Architektur aussieht, sehen wir in [Projektstruktur](projektstruktur.md).

## Code Quality Tools & Linter

Pro Sprache setzen wir folgende Tools ein:

### PHP

- **PHP Parallel Lint:** Schneller Syntax-Check (`php -l` parallelisiert) als erster, günstigster CI-Schritt vor PHPCS.
- **PHP Code Sniffer (PHPCS)** mit den Standards `Drupal` und `DrupalPractice` (via [`drupal/coder`](https://www.drupal.org/project/coder)): Fokus auf Coding-Standards und Best Practices. Fördert Lesbarkeit und Konsistenz durch einheitliche Formatierung.
- **Rector** ([`rector/rector`](https://github.com/rectorphp/rector)): Automatisierte Refactorings und Deprecation-Fixes, insbesondere bei Core-/PHP-Upgrades. Konfiguration liegt als `rector.php` im Projekt-Root.
- **PHP Mess Detector (PHPMD)** *(optional pro Projekt)*: Fokus auf Struktur und Metriken. Fördert Wartbarkeit und Fehlerprävention durch Prüfung von ungenutztem Code und zu komplexen Methoden.

### Twig

- **twigcs** ([`friendsoftwig/twigcs`](https://github.com/friendsoftwig/twigcs)): Coding-Standards für Twig-Templates.
- **Ludtwig:** Formatierung und Stil-Prüfung für Twig, konfiguriert über `ludtwig-config.toml`.

### JavaScript / TypeScript

- **ESLint** – JS in Custom-Modulen/-Themes (Drupal-Konfiguration) sowie TS-Testcode (eigene Konfiguration mit `typescript-eslint`). Getrennte Flat-Configs pro Einsatzzweck (z. B. `eslint.drupal.config.mjs`, `eslint.playwright.mjs`).
- **Prettier** – Formatierung, in ESLint integriert (`eslint-plugin-prettier`) bzw. als `format`-Script.

### SCSS / CSS

- **Stylelint** – auf Basis von `stylelint-config-standard-scss`, mit `stylelint-order` für Property-Reihenfolge und Prettier-Integration.

### Projektweit

- **cspell:** Rechtschreibprüfung über alle Dateien (inkl. deutschem Wörterbuch), läuft auch als Pre-Commit-Check.
- **gitleaks:** Secret-Scanning in CI — verhindert, dass Credentials/Tokens ins Repository gelangen.
- **husky + lint-staged:** Pre-Commit-Hooks, damit günstige Prüfungen schon vor dem Push lokal laufen.

### CI-Integration

Jeder Linter läuft als eigener CI-Job und wird nur ausgeführt, wenn sich relevante Dateien geändert haben (Path-Filter auf Dateiendungen). Teurere Prüfungen (z. B. E2E-Tests) laufen erst, nachdem alle Linter grün sind — siehe [Testing](testing.md).

## composer.json normalisieren

Die `composer.json` wird mit [`ergebnis/composer-normalize`](https://github.com/ergebnis/composer-normalize) deterministisch formatiert und sortiert. Das hält Diffs sauber und Reviews einfach – analog zu PHPCS oder Stylelint für anderen Code.

- `ergebnis/composer-normalize` ist in **jedem** Projekt als Dev-Dependency installiert (`composer require --dev ergebnis/composer-normalize`).
- Ein nicht-normalisierter Stand darf **nicht** in den Haupt-Branch gemergt werden.
- In jedem Projekt muss eine Kontrolle vorhanden sein, entweder durch einen CI-Job oder durch manuelle / KI-Reviewer Kontrolle.

Die Regel gilt auch für eigene Drupal-Module mit eigener `composer.json`. Die Kontrolle liegt beim Entwickler.

## Grundsätze

### Done is better than perfect!

Versuche die Anforderung des Kunden zu Verstehen. Implementierst du etwas, was nur einmal genutzt wird oder grundlegend für die weitere Arbeit ist? Perfektionismus ist wichtig, aber die Zeit und Ressourcen des Kunden sind wichtiger.

### Verstehen was der Kunde will.

Denke mit dem Kunden und versuche zu verstehen, was der Kunde mit dem Ticket erreichen will. Vielleicht fällt die eine bessere Lösung ein oder du siehst Probleme schon bevor sie auftreten

### Es gibt keine dummen Fragen.

Wenn du bei einer Aufgabe nicht weiterkommst. Melde dich im Team! Unser Ziel ist es gemeinsam zu Lernen und unsere Skills zu verbessern. Das geht am besten im Dialog
