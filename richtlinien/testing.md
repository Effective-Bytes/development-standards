# Testing & Qualitätssicherung

## Baseline: Was wird wann getestet

Diese Deklaration legt fest, **welche Prüfungen in welchem Projekt verpflichtend** sind, damit die Frage nicht pro Projekt neu beantwortet werden muss. Sie beschreibt das **Was** und das **Womit**.

### Grundprinzipien

- **Modular & deklarativ.** Es gibt einen festen Katalog von Prüf-Modulen. Für jedes Modul ist eine Bedingung definiert; trifft sie zu, ist das Modul in diesem Projekt verpflichtend.
- **Infrastruktur-minimal.** Jedes Modul muss **lokal in jedem Projekt** lauffähig sein — das ist der garantierte Boden. Wo die Plattform eine CI bietet (GitHub Actions in den meisten Projekten, GitLab CI wo vorhanden), werden die Module dort ausgeführt. Wir setzen **keine Agentur-Server und keine externen SaaS-Dienste** voraus. Bietet eine Plattform keine Pipelines, bleibt die lokale Ausführung plus manuelle bzw. KI-Review-Kontrolle das Minimum — mehr wird nicht erzwungen.
- **Nicht retroaktiv.** Neue Pflichten gelten für **neue und geänderte** Arbeit. Bestandscode wird nicht rückwirkend auf Vollabdeckung nachgerüstet (gleiche Baseline-Denke wie bei der statischen Analyse).

> **Empfehlung Secret-Scanning.** Zusätzlich empfehlen wir Secret-Scanning (z. B. gitleaks), um Credentials aus dem Repository zu halten — **keine Pflicht**, aber in jedem Projekt sinnvoll.

### Module und ihre Bedingungen

Jede Zeile ist eine verbindliche Regel: **Trifft die Bedingung zu, ist das Modul in diesem Projekt Pflicht.** Was die Module genau sind, was sie bringen und womit sie umgesetzt werden, steht in [Die Module im Detail](#die-module-im-detail).

| Modul | Pflicht, wenn … | Betrifft typischerweise |
|---|---|---|
| **M1 · Linting / Coding-Standards** | **immer** — für jede im Projekt vorhandene Sprache bzw. `composer.json` | alle Projekte, inkl. Tooling/Doku |
| **M2 · Statische Analyse** | Custom-PHP mit eigener Logik existiert | Sites/Plattformen mit Custom-Modulen |
| **M3 · Unit-/Funktionale Tests** | **neue oder geänderte** nicht-triviale Business-Logik (Service, Plugin, Controller, Berechnung) | dieselben Projekte, sobald eigene Logik entsteht |
| **M4 · E2E / User-Flows** | ≥ 1 geschäftskritischer User-Flow vorhanden (Prinzip s. u.) | Plattformen/Applikationen |
| **M5 · Visual Regression (VRT)** | Projekt ist design-/theme-getrieben **oder** Multidomain/Multilocale | design-lastige Sites, Multidomain-Estates |

> **Geschäftskritischer User-Flow (Bedingung für M4).** *Prinzip:* ein Flow, dessen Ausfall unmittelbaren Geschäfts- oder Kundenschaden verursacht — Umsatz, Rechtssicherheit oder Kernnutzen der Site. *Beispiele:* Login/Registrierung, Checkout/Zahlung, datenschreibende Formulare (Kontakt, Bewerbung, Anfrage), Mitgliedschafts-/Kontostatus, Buchungs- und Such-Flows. *Kein M4, sondern eine M3-ExistingSite-Aufgabe:* „Seite X liefert Status 200 bzw. den erwarteten Inhalt".

### Die Module im Detail

Verbindliche Tool-Festlegung je Modul plus Einordnung. Die ausführlichen Best-Practices stehen unter [Arten von Tests](#arten-von-tests).

| Modul | Was es ist | Was es bringt | Womit (verbindlich) |
|---|---|---|---|
| **M1 · Linting / Coding-Standards** | Automatische Prüfung von Formatierung und Stil-Konventionen pro Sprache, ohne den Code auszuführen. | Einheitlicher, lesbarer Code; die billigste und schnellste Prüfung, meist auto-fixbar; entlastet Reviews von Formatierungsdiskussionen. | PHP: parallel-lint + PHPCS (`Drupal`/`DrupalPractice`); Twig: twigcs/ludtwig; JS/TS: ESLint + Prettier; SCSS: Stylelint; `composer.json`: composer-normalize |
| **M2 · Statische Analyse** | Analyse des Codes ohne Ausführung auf Typsicherheit, Deprecations, Komplexität und potenzielle Fehler — über reine Formatierung hinaus. | Findet Bugs und veraltete APIs früh; erleichtert Core-Upgrades; erzwingt wartbarere Strukturen. | PHPStan mit `mglaman/phpstan-drupal` + `phpstan/phpstan-deprecation-rules`; **Mindest-Level 1** mit Baseline, schrittweise anheben (s. u.); Rector für Upgrades; PHPMD optional |
| **M3 · Unit-/Funktionale Tests** | Ausführende Tests der eigenen Logik — von isolierten Unit-Tests bis zu funktionalen Tests gegen eine laufende Site. | Sichert das Verhalten der Business-Logik gegen Regressionen ab und dokumentiert das erwartete Verhalten; günstige Stufen laufen bei jedem Push. | PHPUnit + DTT, Stufe so niedrig wie möglich (Unit → Kernel → ExistingSite → Functional), siehe [Abstufungen](#abstufungen) |
| **M4 · E2E / User-Flows** | Prüft komplette User-Flows aus User-Perspektive im echten Browser gegen eine realistische Umgebung. | Sichert das Zusammenspiel des Gesamtsystems inkl. Frontend ab; fängt Fehler, die isolierte Unit-/Funktionaltests nicht sehen. | **Playwright** (deckt sich mit Drupals Ausrichtung weg von Nightwatch) |
| **M5 · Visual Regression (VRT)** | Automatisierter Screenshot-Vergleich gegen genehmigte Baselines zur Erkennung ungewollter visueller Änderungen. | Fängt visuelle Regressionen, die funktionale Tests nicht erfassen (Layout, Abstände, Farben); schützt design-kritische und Multidomain-Projekte. | **Playwright-Screenshots** (Default) *oder* **BackstopJS** (bei Bedarf an dediziertem Multi-Viewport-Report) |

> **PHPStan-Level (M2).** Drupal Core analysiert selbst auf **Level 1** mit Baseline — die dynamischen Drupal-APIs (statische `\Drupal`-Aufrufe, Entity-Queries, Plugin-Rückgabetypen) erzeugen auf höheren Levels ohne die `phpstan-drupal`-Extension viel Rauschen. Daraus folgt verbindlich: **Level 1 als Boden**, `phpstan-drupal` + Deprecation-Rules immer aktiv, und die bestehenden Findings per **Baseline** akzeptiert, sodass nur neuer Code am Level gemessen wird. Der Level wird pro Projekt **schrittweise angehoben**; für neuen Custom-Code ist Richtung **Level 5–6** das Ziel. (Zuverlässige Deprecation-Erkennung setzt mindestens Level 2 voraus.)

## Teststruktur

Projektweite Tests (End-to-End, Integration, Performance, …) liegen in `<root>/tests`, mit einem eigenen Unterordner pro Framework. Modul-eigene Unit-/Funktionale Tests liegen im jeweiligen Modul (siehe Abschnitt „Unit- und Funktionale Tests").

## Arten von Tests

Diese Richtlinie gibt die grobe Richtung und Best-Practices vor, **falls** eine der folgenden Arten in einem Projekt eingesetzt wird. Die verbindlichen Tool-Festlegungen je Modul stehen in [Die Module im Detail](#die-module-im-detail) — u. a. **PHPStan** (M2), **DTT** für PHPUnit-Tests (M3) und **Playwright** (M4). Dieser Abschnitt beschreibt das *Wie* im Einsatz.

### 1. Linting

Prüft automatisiert, ob Code einer definierten Formatierung/Stil-Konvention folgt (Einrückung, Quoting, unbenutzte Variablen, …). Die konkreten Tools pro Sprache sind in [Coding Standards](coding-standards.md) definiert.

- Generierte Verzeichnisse (`vendor/`, `node_modules/`, Build-Output) werden explizit ignoriert, nicht implizit über Performance-Timeouts übersprungen.
- Linting ist die günstigste Prüfung (schnell, meist auto-fixbar) und sollte so früh wie möglich laufen (pre-commit-Hook oder erster CI-Schritt), bevor teurere Prüfungen (Tests) überhaupt starten.

### 2. Statische Code-Analyse

Analysiert Code ohne ihn auszuführen — auf Struktur, Komplexität, Typsicherheit und potenzielle Fehler. Das geht über reine Formatierung (Linting) hinaus. Verbindlich ist **PHPStan** (mit `mglaman/phpstan-drupal` + Deprecation-Rules, siehe M2); ergänzende Tools wie PHPMD sind optional.

- Wird ein Tool neu in Bestandscode eingeführt: Mit einer **Baseline** starten (bestehende Findings werden akzeptiert, nur *neue* Verstöße schlagen fehl). Ohne Baseline wird die erste Einführung sofort so viele Findings werfen, dass das Tool nach kurzer Zeit ignoriert wird.
- Strenge (Level/Regelsatz) schrittweise erhöhen statt von Anfang an das Maximum zu erzwingen.

### 3. Unit- und Funktionale Tests (PHPUnit + DTT)

Für PHPUnit-basierte Tests legen wir uns verbindlich auf **DTT ([Drupal Test Traits](https://gitlab.com/weitzman/drupal-test-traits))** fest. Die `phpunit.xml` verwendet den DTT-Bootstrap; damit stehen zusätzlich zu den Core-Teststufen die `ExistingSite`-Stufen zur Verfügung, die gegen eine bereits bestehende, installierte Site laufen — ohne pro Testlauf eine frische Site zu installieren.

#### Abstufungen

Aufsteigend nach Kosten (Laufzeit/Setup). Immer die **niedrigste Stufe** wählen, die den Testfall abdeckt.

| Testsuite | Basisklasse | Umgebung | Wann verwenden |
|---|---|---|---|
| `unit` | `UnitTestCase` | Kein Drupal-Bootstrap, reines PHP | Isolierte Logik mit gemockten Abhängigkeiten. Sollte der Großteil der Suite sein — am schnellsten, am stabilsten. |
| `kernel` | `KernelTestBase` | Minimaler Drupal-Bootstrap + DB, keine HTTP-Requests | Drupal-APIs (Entities, Config, Services, Queries) ohne UI. |
| `functional` | `BrowserTestBase` | Frische Site-Installation, HTTP-Requests ohne JS | Seiten, Formulare, Zugriffskontrolle — wenn der Test eine **kontrollierte, frische Installation** braucht (z. B. Install-Hooks, definierte Modul-Kombination). |
| `functional-javascript` | `WebDriverTestBase` | Frische Site-Installation + echter Browser (WebDriver) | Wie `functional`, zusätzlich JS-Verhalten (AJAX, Dialoge). |
| `existingsite` | `ExistingSiteBase` (DTT) | Bestehende, installierte Site, HTTP-Requests ohne JS | Tests gegen die reale Site samt echter Config/Inhalten — deutlich schneller, da keine Installation. Bevorzugte Stufe für Functional-Tests, wenn keine frische Installation nötig ist. |
| `existingsite-javascript` | `ExistingSiteSelenium2DriverTestBase` (DTT) | Bestehende Site + echter Browser (WebDriver/Selenium) | Wie `existingsite`, zusätzlich JS-Verhalten. |

DTT bietet darüber hinaus `ExistingSitePerformanceBase` für Performance-Assertions gegen die bestehende Site.

Die `functional`-Stufen mit frischer Installation sind die teuersten: Die komplette Site-Installation fällt **pro Testklasse** an und skaliert linear mit der Anzahl der Testklassen. Wo der Test nicht auf einer frischen Installation basiert, sind die `ExistingSite`-Stufen zu bevorzugen.

#### Konventionen

- **Dateiname:** Muss auf `Test.php` enden (z. B. `ExampleServiceTest.php`).
- **Ordner:** Tests liegen im jeweiligen Modul unter `tests/src/<Stufe>/` (`Unit`, `Kernel`, `Functional`, `FunctionalJavascript`, `ExistingSite`, `ExistingSiteJavascript`), nicht im globalen `tests/`-Verzeichnis — siehe [Teststruktur](#teststruktur).
- **Namespace:** `Drupal\Tests\[MODUL_NAME]\<Stufe>` passend zum Ordner.
- **Klassenname:** Muss identisch mit dem Dateinamen sein und von der Basisklasse der jeweiligen Stufe erben (siehe Tabelle).
- **Methoden:** Testmethoden müssen mit dem Präfix `test` beginnen (z. B. `public function testCalculation()`).
- **Testsuiten:** Die `phpunit.xml` definiert pro Stufe eine eigene Testsuite, sodass CI-Jobs gezielt einzelne Stufen ausführen können (z. B. nur `unit` bei jedem Push).

### 4. End-to-End-Tests (Browser & User-Flows)

E2E-Tests prüfen komplette User-Flows aus User-Perspektive im echten Browser — Navigation, Formulare, Login, Suche, Checkout — gegen eine reale Umgebung (verbindlich mit **Playwright**, siehe [eigener Abschnitt](#playwright)). Im Gegensatz zu den PHPUnit-Stufen aus Abschnitt 3 testen sie nicht einzelne APIs oder Seiten isoliert, sondern das Zusammenspiel des Gesamtsystems inklusive Frontend.

- Testen **Flows, keine Implementierung**: „User kann sich registrieren und einloggen" — nicht „Route X liefert Statuscode 200" (das ist eine `existingsite`-Aufgabe).
- Laufen gegen eine reale bzw. realistische Umgebung (Dev/Staging), nicht gegen Mocks.
- Müssen idempotent sein und dürfen keine produktiven Daten verändern.
- Laufen in CI sinnvollerweise **nach** den günstigeren Prüfungen (Linting, statische Analyse, Unit-Tests) — sie sind die teuerste und langsamste Stufe.
- Liegen in `<root>/tests/<framework>/`, siehe [Teststruktur](#teststruktur).

### 5. Visual-Regression-Tests (VRT)

Automatisierter Screenshot-Vergleich zur Erkennung ungewollter visueller Änderungen. Framework-unabhängig — ob ein dediziertes Tool (z. B. BackstopJS), Playwrights eingebauter Screenshot-Vergleich oder etwas anderes genutzt wird, ist Projektentscheidung.

- Instabile Bereiche (Animationen, Live-/Zeit-/Datumsanzeigen, zufällige Inhalte) vor dem Screenshot "einfrieren" (Mock-Zeit, Animationen deaktivieren, feste Testdaten). Ohne das wird VRT dauerhaft flaky und verliert das Vertrauen des Teams.
- Ob ein visueller Diff den Merge blockiert oder nur zur manuellen Freigabe markiert wird, ist Projektentscheidung (siehe Abschnitt „Reporting & Blocking-Verhalten") — visuelle Änderungen sind oft gewollt, ein hartes Blocking erzwingt aber, dass niemand ungewollte Diffs übersieht.

## Aussagekräftige Tests

Ein Test, der grün ist, aber nichts Echtes prüft, ist gefährlicher als kein Test — er erzeugt falsche Sicherheit. Tests müssen das **erwartete Verhalten aus der Anforderung** prüfen, nicht die Implementierung „nachzeichnen". Das gilt für alle Stufen aus [Arten von Tests](#arten-von-tests). Besonders relevant bei KI-generierten Tests: Sie neigen dazu, die Implementierung zu spiegeln und Coverage zu jagen, statt die Anforderung zu prüfen.

### Prinzipien

1. **Aus der Anforderung ableiten, nicht aus dem Code.** Der Test formuliert das erwartete Verhalten aus Ticket/Akzeptanzkriterium. Der Testname benennt die Anforderung (`testUserOhneRechtKannNodeNichtBearbeiten`), nicht die Methode. Aufbau: Arrange-Act-Assert bzw. Given-When-Then.
2. **Verhalten statt Implementierung prüfen.** Assertions gelten beobachtbaren Ergebnissen (Rückgabewert, State-Änderung, sichtbarer Seiteneffekt) — nicht internen Aufrufen, Mock-Call-Counts oder privaten Methoden. Ein verhaltenserhaltendes Refactoring darf den Test nicht brechen; sonst ist es ein „Change-Detector", der die Implementierung testet.
3. **Der Test muss fehlschlagen können.** Ein Test, der nie rot war, ist unbewiesen. Erst gegen die falsche/fehlende Implementierung scheitern sehen, dann grün machen (Rot-Grün). Prüffrage: „Würde dieser Test fehlschlagen, wenn das Verhalten falsch wäre?"
4. **Keine Tautologien, kein Over-Mocking.** Nicht das mocken, was gerade getestet wird. Kein Test, der nur wiederholt, was der Code tut. Jede Assertion muss eine echte Erwartung ausdrücken, die auch falsch sein könnte.
5. **Coverage ist Diagnose, kein Ziel.** Code Coverage zeigt nur, welche Zeilen *ausgeführt* wurden, nicht ob sie *geprüft* werden. Kein Coverage-Prozentsatz als Ziel oder Gate — Coverage nur nutzen, um ungetestete Bereiche zu finden.
6. **Tests sind First-Class im Review.** Reviewer (Mensch und KI) prüfen Tests wie Produktivcode: Entspricht der Test einer echten Anforderung? Prüft er Verhalten? Würde er bei falschem Verhalten scheitern?

### Mutation Testing (optional)

Die einzige objektive Messung der *Aussagekraft* — nicht nur der Ausführung — von Tests. Ein Tool erzeugt kleine fehlerhafte Code-Varianten (Mutanten, z. B. `>=` → `>`, `true` → `false`) und prüft, ob die Tests das bemerken (Mutant „getötet") oder nicht (Mutant „entkommen"). Anders als Coverage findet es dadurch fehlende und schwache Assertions.

- Tool für PHP/PHPUnit: **Infection** — nutzt die bestehende PHPUnit-Suite, kein zusätzliches Framework. Kennzahl ist der **MSI** (Mutation Score Indicator, Anteil getöteter Mutanten).
- Sinnvoll nur für schnelle Stufen: **Unit** und leichte **Kernel**-Tests. Für `Functional`/`ExistingSite` (frische Site / echter Browser) ist die Laufzeit pro Mutant zu hoch.
- Einsatz als bewusste, punktuelle Qualitätsmessung kritischer Business-Logik — keine Pflicht, kein CI-Gate per Default.

## Reporting & Blocking-Verhalten

Nicht jede Test-Art sollte gleich behandelt werden, wenn sie fehlschlägt. Als Orientierung:

| Art                        | Typischer Zeitpunkt                           | Blocking sinnvoll?                                                                                                         |
|----------------------------|-----------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| Linting                    | Pre-Commit / erster CI-Schritt                | Ja — günstig, meist auto-fixbar                                                                                            |
| Statische Analyse          | Jeder Push                                    | Ja für neue Findings; bestehende via Baseline zunächst nicht                                                               |
| Unit / Kernel              | Jeder Push                                    | Ja                                                                                                                         |
| Functional / ExistingSite  | Vor Merge                                     | Ja — teurere Stufen ggf. als eigener Job getrennt von der `unit`-Suite                                                     |
| E2E (Browser & User-Flows) | Vor Merge, **nach** den günstigeren Prüfungen | Ja, sofern stabil — bei bekannter Flakiness getrennt von der Haupt-Pipeline laufen lassen, damit sie diese nicht blockiert |
| VRT                        | PR / vor oder nach Deployment                 | Projektentscheidung, meist nein — die Kontrolle (manuelle Freigabe der Diffs) muss aber sichergestellt sein               |

Unabhängig vom Blocking-Verhalten gilt: Das Ergebnis muss direkt am PR sichtbar sein (Status-Check als Minimum). Ausführliche Reports (HTML-Report, Trace-Viewer, Coverage, Screenshot-Diffs) werden als CI-Artefakt verlinkt, nicht nur lokal erzeugt — sonst sind sie für Reviewer ohne lokalen Testlauf nicht nachvollziehbar.

## Wartung & Zuständigkeiten

- Schlägt ein Test durch eine eigene Änderung fehl, behebt der Autor der Änderung ihn, bevor gemerged wird.
- Ein als flaky erkannter oder veralteter Test wird nicht stillschweigend übersprungen. Wird ein Test deaktiviert (`@skip`, `markTestSkipped`, `.skip()`, …), enthält der Code einen Kommentar mit Begründung bzw. Ticket-Referenz.
- Wer einen flaky/veralteten Test entdeckt, ist dafür zuständig, ihn zu melden oder zu fixen — nicht einfach zu ignorieren, da sich sonst Testschulden unbemerkt anhäufen.

## Playwright

Playwright ist ein E2E-Testframework, das Webseiten aus User-Perspektive testet — Navigation, Formulare, Zahlungsflows. Es läuft in CI gegen die tatsächliche Umgebung.

### Playwright Konfiguration (`playwright.config.ts`)

Die zentrale Konfigurationsdatei `playwright.config.ts` definiert globale Einstellungen wie die Anzahl der Retries oder das Verhalten bei Fehlern. Hier werden auch alle Projekte bzw. Testsuiten definiert.

Tests einer Suite zuweisen erfolgt über Tags. Einem Projekt können dann gezielt Tests zugeordnet werden – entweder per `grep` (nur passende Tags ausführen), `grepInvert` (bestimmte Tags ausschließen) oder `testMatch` (Dateipfade per Glob-Pattern).

```ts
// playwright.config.ts
projects: [
  {
    name: 'smoke',
    grep: /@smoke/,               // Nur Tests mit @smoke-Tag
  },
  {
    name: 'regression',
    grepInvert: /@slow/,          // Alle Tests außer @slow
  },
  {
    name: 'checkout',
    testMatch: '**/checkout/**',  // Nur Tests im checkout-Ordner
  },
]
```

Projektabhängigkeiten ermöglichen es, Projekte aufeinander aufzubauen. Ein klassisches Beispiel ist ein `setup`-Projekt (z. B. Login oder Datenbankseeding), von dem alle anderen Projekte abhängen – es wird dann automatisch zuerst ausgeführt.

```ts
projects: [
  { name: 'setup', testMatch: /global\.setup\.ts/ },
  {
    name: 'e2e',
    dependencies: ['setup'], // Setup läuft automatisch zuerst
  },
]
```

`globalSetup` / `globalTeardown` sind Dateien, die einmalig vor bzw. nach allen Tests laufen – unabhängig von Projekten. Typische Anwendungsfälle sind das Starten externer Services oder das Bereinigen von Testdaten.

```ts
globalSetup: './tests/playwright/global.setup.ts',
globalTeardown: './tests/playwright/global.teardown.ts',
```

Reporter steuern, wie Logs und Fehlermeldungen ausgegeben werden. Es ist möglich und sinnvoll, mehrere Reporter gleichzeitig zu nutzen – auch eigene (custom) Reporter – um je nach Kontext den passenden Output zu erhalten.

```ts
// Alle 3 reporter werden gleichzeitig verwendet.
reporter: [
  ['github'],                          // Annotations in GitHub Actions
  ['html', { open: 'on-failure' }],    // Visueller HTML-Report bei Fehler
  ['./reporters/custom-reporter.ts'],  // Eigener Reporter (z. B. Slack-Notification)
],
```

### Best-Practices / Regressions-Dokumentation

| Situation | Falsch | Richtig | Warum |
|---|---|---|---|
| Test-Schritte für den Report gliedern | `console.log(testName + ': Step...')` | `await test.step('Step...', async () => { ... })` | Steps erscheinen strukturiert im HTML-Report und Trace-Viewer — `console.log` nicht |
| Prüfen ob ein Element im DOM vorhanden ist | `expect(locator).toBeDefined()` | `await expect(locator).toBeVisible()` | Ein Locator ist immer ein JS-Objekt — `toBeDefined()` prüft nie den DOM |
| Prüfen ob ein Element sichtbar ist | `await locator.isVisible()` ohne `expect` | `await expect(locator).toBeVisible()` | Ohne `expect` wird das Ergebnis still verworfen, kein Fehler bei Fehlschlag |
| Warten bis eine Seite oder ein Element bereit ist | `waitForTimeout(n)` | `waitForLoadState(...)` / `locator.waitFor({ state: 'visible' })` | Fester Timeout wartet mal zu kurz (flaky), mal zu lang (langsam); Auto-Waiting wartet genau bis zum Zustand |
| Eine fehleranfällige Aktion mehrfach versuchen | Manueller Retry-Loop mit `for`/`while` | `expect(async () => { ... }).toPass({ intervals: [5000, 10000, 15000] })` | `toPass` wiederholt mit steigenden Intervallen und wirft einen sauberen Fehler bei Timeout |
| Ein einzelnes interaktives Element anklicken | `page.click('#edit-openid-connect-client')` | `page.getByRole('button', { name: 'Login' }).click()` | Semantische Selektoren (`getByRole`, `getByLabel`, `getByText`) sind stabiler als IDs/CSS |
| Auf ein Element warten, das mehrfach vorkommt | `page.locator('text=X')` (mehrere Treffer) | Bevorzugt ein eindeutigerer Locator, sonst: `page.locator('text=X').first().waitFor()` | Playwright strict mode wirft Fehler bei mehreren Treffern — `.first()` / `.nth(n)` explizit angeben |
| Auf das Erscheinen eines Elements warten | `page.waitForSelector('text=X')` | `page.locator('text=X').waitFor()` | `waitForSelector` ist deprecated; Locator-basierte APIs haben bessere Auto-Waiting-Semantik |
| Hilfsfunktionen benötigen eine Seite zum Interagieren | `chromium.launch()` in Helper-Funktionen | `page: Page` als Parameter vom Fixture übergeben | Manuelle Browser umgehen Tracing, Cleanup und Timeout-Handling des Fixture-Systems |
| Denselben Testablauf mit verschiedenen Eingaben ausführen | Duplizierter Testcode pro Szenario | `scenarios.forEach(s => test(...))` | Separate benannte Test-Einträge im Report, ein einziger Ort zum Anpassen |
| Mehrere Playwright-Projekte konfigurieren | `use`, `retries`, `devices` in jedem Projekt-Eintrag wiederholt | Gemeinsame Optionen einmalig in globalem `use: {}`, Projekte nur mit abweichenden Werten | Redundanz führt zu Inkonsistenzen; globale Defaults werden bei Anpassungen leicht vergessen |
