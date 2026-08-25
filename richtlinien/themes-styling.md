# Themes und Styling

## Klare Trennung zwischen Logik und Darstellung

Sämtliches CSS für das visuelle Erscheinungsbild gehört ausschließlich in Themes. Das einzige Styling in eigenen Modulen sollte strukturell / funktionell sein. Denn:

- Themes müssen austauschbar sein, ohne dass Module angepasst werden müssen.
- Module die wir entwickeln sollen auch bei anderen Kunden anwendbar sein, ohne dass das Corporate Design verändert wird.

## Variablen: SCSS `$` vs. CSS Custom Properties (`--variable`)

**Regel:** Für alle Werte, die über eine einzelne Datei hinaus relevant sind (Farben, Spacings, Font-Stacks, Breakpoints, Theme-Tokens etc.), wird die CSS-Custom-Property-Notation `--variable` verwendet. SCSS-Variablen (`$variable`) dürfen ausschließlich datei-lokal genutzt werden – z. B. für eine Hilfsberechnung, einen kurzlebigen Zwischenwert oder einen Loop-Index in einem `@each`/`@for`.

**Begründung – warum `--variable`:**

- **Lebt im DOM, nicht im Build:** Custom Properties existieren zur Laufzeit als Eigenschaft eines DOM-Knotens. Sie folgen der CSS-Kaskade und der Vererbung – Kinder erben den Wert vom nächstgelegenen Vorfahren, der ihn definiert.
- **Skopierbar / lokal überschreibbar:** Eine Variable kann auf einem Container neu gesetzt werden und gilt damit nur für diesen Subtree. Beispiel: `--color-bg` global auf `:root`, in `.theme-dark` neu definiert – das gesamte Subtree bekommt automatisch den dunklen Wert, ohne neue Selektoren oder Klassen-Kaskaden.
- **Dynamisch zur Laufzeit:** Werte können per JavaScript gelesen und gesetzt werden (`element.style.setProperty('--x', …)`), per Media Query (`prefers-color-scheme`), per `:hover`/`:focus`/State umgeschrieben werden. Das ermöglicht Dark Mode, User-Theming, responsive Tokens und Animationen ohne Recompile.
- **Komponenten-API:** SDC und Blöcke können dokumentierte Custom Properties als „Anpassungs-Schnittstelle" nach außen geben, ohne dass Konsumenten den internen CSS-Aufbau kennen müssen.
- **Nativer Web-Standard:** Keine Build-Abhängigkeit, kein Import-Graph, keine Reihenfolge-Probleme zwischen Partials. HTML/CSS entwickeln sich weg von Build-Tools hin zu Laufzeit-Logik.

**Wo SCSS `$` legitim ist:**

- Innerhalb einer einzelnen `.scss`-Datei als Kurzschreibweise für einen Wert, der nirgends sonst gebraucht wird.
- Als Eingangsparameter eines `@mixin`/`@function`.
- Als Loop-/Map-Schlüssel in `@each`, `@for`, `@if` – also dort, wo zur Compile-Zeit Code generiert wird (Custom Properties können das nicht).

**Unterschiede im Überblick:**

| Aspekt | SCSS `$variable` | CSS `--variable` |
|---|---|---|
| Existiert zur | Compile-Zeit | Laufzeit (im Browser) |
| Im Output sichtbar | Nein, wird inline ersetzt | Ja, als Property am Element |
| Vererbt durch DOM | Nein | Ja, entlang des DOM-Baums |
| Lokal überschreibbar (Subtree, State, Media Query) | Nein – jeder Import sieht denselben statischen Wert | Ja – auf jedem Selektor neu setzbar |
| Per JS lesbar / setzbar | Nein | Ja (`getPropertyValue` / `setProperty`) |
| Reagiert auf `:hover`, `:focus`, `@media`, `@container` | Nein | Ja |
| Sinnvoll für Theme-Tokens / Dark Mode / Theming | Nein | Ja |
| Sinnvoll für Loop-Index, Mixin-Argument, Compile-Helper | Ja | Nein |
| Build-Abhängigkeit | Ja (SCSS-Compiler nötig) | Nein |

## Twig: Templates einbinden

| Situation                                          | Syntax                                                                                     |
|----------------------------------------------------|--------------------------------------------------------------------------------------------|
| Inhalt einbinden (Standard)                        | `{{ include('provider:id', {...}, with_context=false) }}`                                  |
| HTML in Include benötigt                           | `{% set x %}…{% endset %}` dann als Variable übergeben                                     |
| Rohe Pfade (`modules/custom/…`, `themes/custom/…`) | Durch Namespace ersetzen: `@modulname/…`, `@themename/…`                                   |
| `{% embed %}`                                      | Nur wenn die Komponente Slots via `{% block %}` rendert, oder der Block überschrieben wird |
| `{% include %}`-Tag                                | Nicht verwenden — durch `include()`-Funktion ersetzen                                      |
