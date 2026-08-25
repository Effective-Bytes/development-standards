# Review-Checkliste für KI-Reviewer

Kompakte, einzeln prüfbare Kurzfassung der Richtlinien-Punkte, die **direkt im Code-Diff** verifizierbar sind. Jeder Punkt muss ohne Recherche außerhalb des Repositorys entscheidbar sein – kein „existiert dafür ein Contrib-Modul?", keine externen Service-Lookups, keine Prozess-/Sales-Checks.

Was hier nicht steht (bewusst weggelassen, weil **außerhalb des Code-Diffs**): Contrib-First-Entscheidungen, Drupal.org-Projektsetup, CI-Pipeline-Existenz, Vertragsklärungen, Drush-Generate-Nutzung als Prozess (nur das Ergebnis wird geprüft).

Legende: ❌ falsch · ✅ richtig · 📄 Quelle in `../richtlinien/`

---

## 1. Coding Standards (📄 coding-standards.md)

- [ ] **1.1** PHP-Code verstößt nicht gegen Drupal Coding Standards (Einrückung, Klammern, Naming, DocBlocks).
- [ ] **1.2** Keine ungenutzten Variablen, Imports oder unnötig komplexen Methoden (PHPMD-Verstöße).
- [ ] **1.3** JS-Code verstößt nicht gegen ESLint-Regeln (keine `console.log`-Reste, keine ungenutzten Variablen, kein `var`).
- [ ] **1.4** SCSS/CSS verstößt nicht gegen Stylelint-Regeln (Reihenfolge, Quoting, Einheiten).
- [ ] **1.5** Geänderte `composer.json` ist mit `composer normalize` normalisiert. Im Zweifel via `composer normalize --dry-run` prüfen.
- [ ] **1.6** Twig-Templates verstoßen nicht gegen twigcs-/Ludtwig-Regeln (Einrückung, Spacing in `{{ }}`/`{% %}`, keine toten Kommentare).
## 2. Projektstruktur (📄 projektstruktur.md)

- [ ] **2.1** Web-Root ist `web/`; Pfade im Diff respektieren das (kein Code in `/sites`, `/modules` auf Root-Ebene).
- [ ] **2.2** Geänderte Config liegt in `config/sync/`, nicht woanders.
- [ ] **2.3** Lokale Patches liegen in `patches/` und sind in `composer.json` registriert.
- [ ] **2.4** Projektweite Tests liegen in `tests/<framework>/` (`behat/`, `cypress`/`playwright`, `phpunit/`).
- [ ] **2.5** Diff fügt nichts in `vendor/`, `web/sites/default/files/` oder `settings.local.php` hinzu (nicht committen).
- [ ] **2.6** Custom Module liegen unter `web/modules/custom/`, Custom Themes unter `web/themes/custom/`.
- [ ] **2.7** Neue Custom-Module sind als Feature- oder Kern-Modul (`myproject_core`, `myproject_<feature>`) eingeordnet, nicht als Sammelmodul.

## 3. Themes & Styling (📄 themes-styling.md)

- [ ] **3.1** Visuelles CSS (Farben, Typografie, Spacing-Tokens, Layout-Optik) liegt in einem Theme, **nicht** in einem Modul.
- [ ] **3.2** CSS in Modulen ist ausschließlich strukturell/funktionell (z. B. `display: none` für Toggles), nicht Corporate-Design-bezogen.
- [ ] **3.3** Theme-Tokens (Farben, Spacings, Font-Stacks, Breakpoints, Theming) sind als CSS Custom Properties `--variable` definiert und über `var(--…)` konsumiert.
- [ ] **3.4** Werte, die über eine einzelne `.scss`-Datei hinaus relevant sind, sind **nicht** als SCSS-Variable `$…` deklariert.
- [ ] **3.5** SCSS `$…` wird nur datei-lokal verwendet, oder als Argument für `@mixin`/`@function`, oder als Schlüssel in `@each`/`@for` – also nur dort, wo zur Compile-Zeit Code generiert wird.
- [ ] **3.6** Theming-Varianten (Dark Mode, Sub-Brand, Subtree-Override) werden über Re-Definition von Custom Properties auf einem Selektor gelöst, nicht über duplizierte Klassen-Kaskaden oder neue SCSS-Builds.
- [ ] **3.7** Twig-Template-Einbindung: ❌ `{% include %}` Tag, ❌ rohe Pfade (`modules/custom/…`). ✅ `include()`-Funktion mit Namespace (`@modulname/…`). `{% embed %}` nur wenn Slots via `{% block %}` überschrieben werden.

## 4. Architektur: Controller / Hooks / Events / SDC (📄 architektur.md)

### 4.1 Hook- vs. Event-Priorität
- [ ] **4.1.1** Wo ein passendes Symfony Event existiert (z. B. Entity-Events), wird ein **Event Subscriber** statt eines `hook_entity_*` verwendet.
- [ ] **4.1.2** Neue Hooks werden als Klasse unter `src/Hook/*.php` mit Hook-Attribut implementiert – **kein** Hinzufügen neuer Funktionen in `.module`-Dateien.
- [ ] **4.1.3** Falls ausnahmsweise doch in `.module`/`*.theme`: Der Hook-Body holt einen Service und delegiert sofort. Keine Logik inline.

### 4.2 Verantwortungstrennung
- [ ] **4.2.1** Controller, Event Subscriber und Hook-Klassen enthalten **keine** Businesslogik (Berechnungen, API-Calls, komplexe Queries, Datenmanipulation).
- [ ] **4.2.2** Controller geben nur Request → Response weiter; Subscriber/Hooks reagieren nur auf das Ereignis.
- [ ] **4.2.3** Ereignis-Reaktion passt zum Ereignis (kein Routing in `entity_save`, kein Mail-Versand in einem Render-Hook etc.).
- [ ] **4.2.4** Logik ist in einen Service ausgelagert (`src/Service/…`), Controller/Subscriber/Hook ruft nur diesen Service auf.

### 4.3 Single Directory Components (SDC)
- [ ] **4.3.1** Mehrfach verwendete UI-Bausteine (Buttons, Karten, Teaser, Header, Footer, Panels …) sind als SDC im Theme implementiert, nicht als ad-hoc Twig-Includes oder global registrierte Library.
- [ ] **4.3.2** Jede SDC besitzt eine `*.component.yml` mit definierten `props` (und ggf. `slots`); kein im Twig undefiniert verwendetes Variable-Set.
- [ ] **4.3.3** `*.css`, `*.js` und `*.twig` einer SDC liegen im selben Komponenten-Ordner. Kein Streuen über globale `css/`-/`js/`-Verzeichnisse.

## 5. Modul-Entwicklung (📄 modul-entwicklung.md)

### 5.1 Namenskonventionen
- [ ] **5.1.1** Modul-Maschinenname beginnt mit Projekt-Präfix (`myproject_…`).
- [ ] **5.1.2** Modul-Maschinenname ist `snake_case`, nur Kleinbuchstaben, keine Bindestriche/CamelCase.
- [ ] **5.1.3** Namespace passt 1:1 zum Maschinennamen (`Drupal\myproject_xxx\…`).
- [ ] **5.1.4** Modulname und Klassennamen sind generisch (Funktion beschreibend), nicht kunden-/projektspezifisch („Header für Kunde X" → ❌).

### 5.2 Dependency Injection
- [ ] **5.2.1** ❌ `\Drupal::service(…)`, `\Drupal::entityTypeManager()`, `\Drupal::config()` etc. innerhalb von Klassen (Controller, Service, Plugin, Form, EventSubscriber). ✅ Konstruktor-Injection.
- [ ] **5.2.2** Services bekommen ihre Abhängigkeiten via `__construct()`; Registrierung in `services.yml`.
- [ ] **5.2.3** Plugins / Controller implementieren `ContainerFactoryPluginInterface` und nutzen `create()` für DI.
- [ ] **5.2.4** Statischer Service-Zugriff (`\Drupal::…`) ist nur in `.module`/`*.theme`/Procedural-Code akzeptabel – und auch dort nur, um sofort an einen Service zu delegieren.

### 5.3 `info.yml`
- [ ] **5.3.1** `core_version_requirement` ist gesetzt und auf der Höhe von `^10 || ^11` (oder neuer).
- [ ] **5.3.2** `package` ist gesetzt (z. B. `Custom`, `MyProject Features`).
- [ ] **5.3.3** Alle benötigten Module sind explizit unter `dependencies:` aufgeführt – kein implizites „ist eh aktiviert".

### 5.4 Konfiguration
- [ ] **5.4.1** Initial-Config liegt in `config/install/`.
- [ ] **5.4.2** Für **jede** vom Modul gelieferte Config existiert ein Schema in `config/schema/*.schema.yml` (kein Deprecation-Warning, valide Übersetzbarkeit).

### 5.5 Routen (`*.routing.yml`)
- [ ] **5.5.1** Routenname beginnt mit Modulname **oder** mit `entity` (nur für allgemeine Entity-Operationen wie view/edit/delete/custom).
- [ ] **5.5.2** Routenname ist komplett kleingeschrieben, Trennzeichen `.`.
- [ ] **5.5.3** Routenname spiegelt die URL-Struktur in der korrekten Reihenfolge wider.
- [ ] **5.5.4** Generische Pfad-Präfixe (`/admin/structure`, …) sind **nicht** Bestandteil des Routennamens.
- [ ] **5.5.5** Aktion (`delete`, `revert`, `confirm`, …) steht am Ende des Routennamens.
- [ ] **5.5.6** Admin-Routen setzen `options: { _admin_route: TRUE }`.

### 5.6 Sanitize
- [ ] **5.6.1** Wenn das Modul personenbezogene oder sensible Daten persistiert, existiert ein Drush-`sql:sanitize`-Plugin (Hook `POST_COMMAND_HOOK` auf `sql:sanitize` plus Bestätigungs-Message via `sql-sanitize-confirms`).

### 5.7 Modul-Schnitt
- [ ] **5.7.1** Ein Modul erfüllt eine klar abgrenzbare Aufgabe. Wenn der Diff zwei thematisch unabhängige Features in dasselbe Modul packt, ist das ein Verstoß.

## 6. Testing (📄 testing.md)

### 6.1 Modul-Baseline (spiegelt „Module und ihre Bedingungen")
Trifft die Bedingung zu und die Prüfung fehlt für **neue/geänderte** Arbeit, ist das ein Verstoß (Bestandscode nicht retroaktiv).
- [ ] **6.1.1** **M1 Linting** — immer Pflicht (konkrete Regeln s. Abschnitt 1).
- [ ] **6.1.2** **M2 Statische Analyse** — Pflicht, sobald Custom-PHP mit eigener Logik existiert (PHPStan; Bestand via Baseline).
- [ ] **6.1.3** **M3 Unit/Funktionale Tests** — neue/geänderte nicht-triviale Business-Logik (Service, Plugin, Controller, Berechnung) bringt begleitende Tests mit.
- [ ] **6.1.4** **M4 E2E** — geschäftskritische User-Flows (Login, Checkout, datenschreibende Formulare) sind durch E2E-Tests (Playwright) abgedeckt.
- [ ] **6.1.5** **M5 VRT** — bei design-/theme-getriebenen oder Multidomain/Multilocale-Projekten sind visuelle Änderungen durch Visual-Regression abgedeckt.

### 6.2 PHPUnit-Struktur (DTT)
- [ ] **6.2.1** PHPUnit-Tests liegen im Modul unter `tests/src/<Stufe>/` (`Unit`, `Kernel`, `Functional`, `FunctionalJavascript`, `ExistingSite`, `ExistingSiteJavascript`), nicht im globalen `tests/`-Verzeichnis.
- [ ] **6.2.2** Testdatei endet auf `Test.php`.
- [ ] **6.2.3** Namespace lautet `Drupal\Tests\<modul_name>\<Stufe>` passend zum Ordner.
- [ ] **6.2.4** Klassenname ist identisch mit Dateiname; Basisklasse passt zur Stufe (`UnitTestCase`, `KernelTestBase`, `BrowserTestBase`, `WebDriverTestBase`, `ExistingSiteBase`, `ExistingSiteSelenium2DriverTestBase`).
- [ ] **6.2.5** Test nutzt die niedrigste ausreichende Stufe; insbesondere: braucht der Test keine frische Site-Installation, ist `ExistingSite*` (DTT) statt `BrowserTestBase`/`WebDriverTestBase` zu verwenden.
- [ ] **6.2.6** Testmethoden beginnen mit `test`.
- [ ] **6.2.7** Deaktivierte/übersprungene Tests (`@skip`, `markTestSkipped(…)`, `.skip()`) enthalten eine Begründung bzw. Ticket-Referenz im Code — kein stilles Skip ohne Kommentar.

### 6.3 Playwright-Patterns
- [ ] **6.3.1** ❌ `console.log('Step…')`. ✅ `await test.step('Step…', async () => { … })`.
- [ ] **6.3.2** ❌ `expect(locator).toBeDefined()`. ✅ `await expect(locator).toBeVisible()` (oder anderes DOM-aware-Assertion).
- [ ] **6.3.3** ❌ `await locator.isVisible()` ohne `expect`. ✅ `await expect(locator).toBeVisible()`.
- [ ] **6.3.4** ❌ `page.waitForTimeout(n)`. ✅ `waitForLoadState(...)` / `locator.waitFor({ state: 'visible' })`.
- [ ] **6.3.5** ❌ Manuelle `for`/`while`-Retry-Loops. ✅ `await expect(async () => { … }).toPass({ intervals: [...] })`.
- [ ] **6.3.6** ❌ CSS-/ID-Selektoren wie `page.click('#edit-…')`. ✅ Semantische Locator: `getByRole`, `getByLabel`, `getByText`.
- [ ] **6.3.7** Bei mehrdeutigen Treffern: präziserer Locator oder explizit `.first()` / `.nth(n)` – nie ein potenziell mehrdeutiger Locator ohne Auflösung.
- [ ] **6.3.8** ❌ `page.waitForSelector(…)`. ✅ `page.locator(…).waitFor()`.
- [ ] **6.3.9** Helper-Funktionen bekommen `page: Page` als Parameter; **kein** `chromium.launch()` in Helpern.
- [ ] **6.3.10** Parameter-Variationen werden via `scenarios.forEach(s => test(...))` ausgedrückt, nicht durch Copy-Paste-Tests.
- [ ] **6.3.11** In `playwright.config.ts`: gemeinsame Optionen liegen in einem globalen `use: {}`. Pro-Projekt-Einträge enthalten nur die abweichenden Werte.

### 6.4 Aussagekraft der Tests
- [ ] **6.4.1** Testname/Testfall beschreibt das erwartete **Verhalten** aus der Anforderung, nicht die Implementierung oder Methode.
- [ ] **6.4.2** Assertions prüfen beobachtbare Ergebnisse (Rückgabewert, State-Änderung, Seiteneffekt), **nicht** bloß Mock-Call-Counts oder interne Aufrufe.
- [ ] **6.4.3** Kein Over-Mocking: die Einheit, die getestet wird, ist nicht selbst weggemockt.
- [ ] **6.4.4** Keine tautologischen/leeren Tests — kein Test ohne aussagekräftige Assertion und keine Assertion, die gar nicht fehlschlagen kann.
- [ ] **6.4.5** Grenzfälle sind abgedeckt, wo das Verhalten an einer Grenze umschlägt (z. B. die `>=`-Grenze), nicht nur „Mitte"-Fälle.

## 7. Assets (📄 assets.md)

- [ ] **7.1** Library wird via Composer eingebunden, wenn ein Paket auf packagist.org oder asset-packagist.org verfügbar ist. ❌ Direkt herunterladen, obwohl ein Composer-Paket existiert.
- [ ] **7.2** npm-Pakete von asset-packagist werden als inline `type: package` mit `type: drupal-library` definiert — ❌ `oomphinc/composer-installers-extender` (archiviert). Die inline Definition steht in der Repositories-Liste vor asset-packagist.
- [ ] **7.3** Reine JS/CSS-Libraries ohne Composer-Paket werden via npm/Yarn verwaltet, nicht manuell heruntergeladen — sofern ein npm-Paket existiert.
- [ ] **7.4** Manuell heruntergeladene Library hat eine `version:`-Angabe in der `*.libraries.yml`.
- [ ] **7.5** CDN/Remote-Einbindung (`type: external`) nur wenn begründete Ausnahme; kein CDN als Standard.

## 8. Barrierefreiheit (📄 barrierefreiheit.md)

Baseline, gilt in jedem Projekt für neue/geänderte Arbeit — kein WCAG-AA-Vollcheck.

- [ ] **8.1** Interaktive Elemente nutzen die passende Semantik: ❌ `<div onclick>`, `<span>` als Button. ✅ `<button>`, `<a href>`, `<ul>`/`<li>`.
- [ ] **8.2** Kein `outline: none` / `outline: 0` ohne gleichwertigen sichtbaren Fokus-Ersatz.
- [ ] **8.3** Kein `tabindex` > 0; keine fokussierbaren Elemente ohne Funktion.
- [ ] **8.4** Jedes Formularfeld hat ein verknüpftes Label (`<label for>`, `#title` im Form-API-Element). Placeholder allein ist kein Label.
- [ ] **8.5** Jedes `<img>` bzw. Bildfeld hat ein `alt`-Attribut — leer bei dekorativ, beschreibend bei inhaltlich. Fehlendes `alt` ist immer ein Verstoß.
- [ ] **8.6** Überschriftenebenen sind strukturell vergeben: eine `h1` pro Seite, keine übersprungenen Ebenen, keine Ebenenwahl aus optischen Gründen.
- [ ] **8.7** Status, Fehler und Pflichtfelder werden nicht ausschließlich über Farbe vermittelt.
- [ ] **8.8** Kontrast-/Farbwerte kommen aus Theme-Tokens (`var(--…)`), nicht als Einzelwert in der Komponente (vgl. 3.3).
- [ ] **8.9** Inhalte, die per AJAX ohne Seitenwechsel ausgetauscht werden, melden das via `Drupal.announce()`.
- [ ] **8.10** Datentabellen nutzen `<th>` mit `scope` und eine `<caption>`; `<table>` wird nicht für Layout verwendet.
- [ ] **8.11** Linktexte sind aus sich heraus verständlich. ❌ „hier klicken", „mehr", „weiterlesen" ohne Bezug im Linktext selbst.
- [ ] **8.12** Kein Accessibility-Overlay / keine A11y-Toolbar (accessiBe, UserWay, Eye-Able o. ä.) wird eingebunden.

## 9. Logging (📄 logging.md)

- [ ] **9.1** In Service / Controller / Plugin / Form / EventSubscriber wird `Psr\Log\LoggerInterface` per DI injiziert. ❌ `\Drupal::logger(…)` an dieser Stelle.
- [ ] **9.2** Wenn ein eigener Channel verwendet wird: Channel ist in `services.yml` deklariert (`parent: logger.channel_base`, `arguments: ['<modul>']`) und wird per `@logger.channel.<modul>` injiziert.
- [ ] **9.3** Wenn der Channel-Name erst zur Laufzeit feststeht, wird `LoggerChannelFactoryInterface` per DI injiziert (statt im Konstruktor zu raten).
- [ ] **9.4** `\Drupal::logger('<modul>')` ist **nur** in `.module`/Procedural-Code akzeptabel.
- [ ] **9.5** ❌ String-Konkatenation oder Interpolation in Log-Messages (`"User {$uid} …"`). ✅ Platzhalter (`@uid`, `%uid`) plus Context-Array.
- [ ] **9.6** Log-Level passt zum Ereignis:
  - `emergency` / `alert` / `critical` → System unbenutzbar
  - `error` → Aktion fehlgeschlagen, Eingriff nötig
  - `warning` → unerwartet, aber abgefangen
  - `notice` → geschäftsrelevantes Ereignis (Bestellung, Login, Import-Start)
  - `info` → technischer Lifecycle (Cron, Deploy-Hook, Cache geleert)
  - `debug` → Diagnose-Details
