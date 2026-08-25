# Barrierefreiheit

## Baseline

Diese Richtlinie legt das **Mindestmaß** fest, das in **jedem** Projekt gilt — unabhängig davon, ob der Kunde gesetzlich betroffen ist. Es sind handwerkliche Grundsätze, die praktisch nichts kosten, wenn sie von Anfang an eingehalten werden, und teuer sind, wenn sie nachgerüstet werden müssen.

Die Baseline ist **kein WCAG-AA-Ziel** und keine Umsetzung gesetzlicher Anforderungen. Sie deckt die Fehlerklassen ab, die den Großteil realer Barrieren ausmachen.

**Geltung:** für neue und geänderte Arbeit. Bestandscode wird nicht rückwirkend nachgerüstet (gleiche Denke wie bei [Testing](testing.md)).

**Prüfung:** im Code-Review, siehe [Review-Checkliste](../ki-reviewer/review-checkliste.md).

## Grundsätze

### Struktur und Semantik

1. **Semantisches HTML vor `div` + ARIA.** Ein Button ist ein `<button>`, ein Link ein `<a href>`, eine Liste eine `<ul>`. ARIA ergänzt Semantik, ersetzt sie nicht — und es ändert ausschließlich, was Hilfsmittel melden, nicht das Verhalten.
2. **Überschriften bilden die Struktur ab, nicht die Optik.** Eine `h1` pro Seite, keine Ebenen überspringen. Die Größe wird über CSS gelöst, nicht über die Ebene.
3. **Tabellen sind für tabellarische Daten**, mit `<th>` samt `scope` und einer `<caption>` — nicht für Layout.
4. **`<html lang>` ist gesetzt** und die Seite hat einen aussagekräftigen, eindeutigen `<title>`.

### Bedienung

5. **Alles, was mit der Maus geht, geht mit der Tastatur** — in einer Reihenfolge, die der visuellen Anordnung entspricht. Keine Tastaturfallen, keine fokussierbaren Elemente ohne Funktion.
6. **Der Fokus ist immer sichtbar.** `outline: none` ohne gleichwertigen Ersatz ist ein Fehler.
7. **Klickflächen sind nicht winzig.** Icon-Buttons und Links brauchen eine ausreichend große, zusammenhängende Trefferfläche.

### Inhalte

8. **Jedes Formularfeld hat ein programmatisch verknüpftes Label.** Ein Placeholder ist kein Label. Fehlermeldungen stehen als Text am Feld und sind mit ihm verknüpft.
9. **Jedes Bild hat ein `alt`.** Leer (`alt=""`) bei rein dekorativen Bildern, beschreibend bei inhaltlichen. Kein `alt` ist immer falsch.
10. **Linktexte sind aus sich heraus verständlich.** „Hier klicken" oder „mehr" sagt nichts, wenn Links isoliert vorgelesen werden.
11. **Inhalte, die sich ohne Seitenwechsel ändern, werden angekündigt** (`Drupal.announce()`) — AJAX-Views, Exposed Filter, Pager, asynchrone Statusmeldungen.

### Darstellung

12. **Farbe ist nie der einzige Informationsträger** — nicht bei Fehlern, Status, Pflichtfeldern oder Diagrammen.
13. **Kontraste werden einmal in den Theme-Tokens festgelegt**, nicht pro Komponente entschieden (siehe [Themes und Styling](themes-styling.md)).
14. **Layouts vertragen Vergrößerung.** Relative Einheiten statt fester Höhen und Pixel-Schriftgrößen; bei 200 % Zoom darf nichts abgeschnitten oder überlagert werden. Nachträglich ist das der teuerste Punkt der Liste.
15. **`prefers-reduced-motion` wird respektiert.** Bewegung ist Komfort, kein Selbstzweck.

### Werkzeuge

16. **Keine Accessibility-Overlays oder -Toolbars** (accessiBe, UserWay, Eye-Able o. ä.). Sie sind redundant zu den Einstellungen in Betriebssystem und Browser und beheben keine Barrieren — siehe [Overlay Factsheet](https://overlayfactsheet.com/de/).

## Wie wir darüber sprechen

Die Einhaltung dieser Grundsätze ist **keine Konformitätsaussage** — und auch eine bestandene Prüfung wäre keine, die wir abgeben könnten.

**„Zertifiziert" gibt es nicht.** Für Barrierefreiheit im Web existiert keine Zertifizierungsstelle. Auch „BITV-Test" ist kein Zustand, den wir feststellen können, sondern ein geschütztes Prüfverfahren einer akkreditierten Prüfstelle.

**„Konform" / „compliant" benutzen wir nicht.** Der Begriff ist in WCAG zwar definiert und ausdrücklich als Selbsterklärung vorgesehen — aber er trägt Bedingungen, die wir nicht halten können:

- Er gilt **alles-oder-nichts**: ein einziges verfehltes Kriterium auf einer einzigen Seite kippt die Aussage für den gesamten erklärten Umfang.
- Er gilt für einen **abgegrenzten Seitenumfang zu einem Zeitpunkt**. Bei einer redaktionell gepflegten Seite ist er nach der nächsten Inhaltspflege nicht mehr gedeckt.
- Er lässt sich **nicht automatisiert feststellen** (siehe [Testing](#testing)) — ein grüner Testlauf belegt ihn nicht.
- Er ist eine Aussage des **Betreibers** über sein Angebot, nicht der Agentur über ihr Werk.

**Was wir stattdessen sagen:** wann, womit und in welchem Umfang wir geprüft haben — und was dabei offen geblieben ist.

> Geprüft gegen WCAG 2.1 AA am 04.08.2026.
> Umfang: Startseite, Produktdetailseite, Warenkorb, Checkout, Kontaktformular.
> Verfahren: automatisierte Prüfung (axe-core) plus manueller Tastatur- und Zoom-Durchlauf.
> Offene Punkte: [Liste].

Das ist eine Tatsachenbeschreibung, keine Garantie — und genau deshalb belastbar. Braucht ein Kunde eine externe, belastbare Aussage, führt der Weg über eine akkreditierte Prüfstelle.

## Gesetzliche Anforderungen

**Nicht Teil dieser Richtlinie.** BITV 2.0 und BFSG sind zu umfangreich und zu folgenreich, um hier nebenbei zusammengefasst zu werden — eine verkürzte Darstellung würde mehr schaden als nutzen. Ob ein Projekt betroffen ist, ist eine Frage, die **pro Projekt und früh** geklärt werden muss, nicht anhand dieser Seite.

Damit das nicht versehentlich untergeht, hier nur die Schwelle, ab der wir hinschauen müssen — **keine Prüfung, kein Ausschluss**:

| Aufmerksamkeit bei … | Dann im Raum |
|---|---|
| Kunde ist eine öffentliche Stelle: Behörde, Ministerium, Kommune, Schule, Hochschule, öffentlich-rechtliche Anstalt oder öffentlich beherrschte Gesellschaft | **BITV 2.0** (Bund) bzw. die jeweilige Landesverordnung — gilt dann für die gesamte digitale Präsenz, nicht nur für Teile davon |
| Privates Unternehmen mit Angebot an Verbraucher (B2C), bei dem online etwas gekauft, gebucht oder abgeschlossen werden kann | **BFSG** (seit 28.06.2025) |

Trifft eine der beiden Zeilen zu, ist das **vor der Angebotserstellung** zu klären — nicht in der Umsetzung. Beide Fälle haben Ausnahmen, Sonderfälle und Zusatzpflichten, die hier bewusst nicht stehen.

Für die Einschätzung gilt: nicht selbst herleiten, sondern die Recherche im Issue-Tracker heranziehen und im Zweifel Rücksprache halten. Ausgangspunkte:

- [Bundesfachstelle Barrierefreiheit](https://www.bundesfachstelle-barrierefreiheit.de/) — FAQ zu BFSG und BITV
- [bfsg-gesetz.de](https://bfsg-gesetz.de/check/) — Betroffenheits-Check für den privaten Sektor
- [BIK BITV-Test](https://bitvtest.de/) — Prüfverfahren und Prüfstellen

Zwei Punkte, die aber jeder kennen sollte, weil sie unser Angebot betreffen: Adressat der gesetzlichen Pflicht ist immer der **Kunde**, nicht wir — verfehlt die Seite dessen Pflicht, kann unser Werk trotzdem als mangelhaft gelten, auch ohne Klausel im Vertrag. Und lehnt ein betroffener Kunde die Umsetzung ab, wird das **schriftlich festgehalten**.

## Testing

Für die Baseline ist **kein Tool verpflichtend** — sie wird im Review geprüft. Der folgende Überblick ordnet ein, was die verfügbaren Verfahren leisten, damit ihre Ergebnisse richtig interpretiert werden. Eine verbindliche Festlegung folgt zusammen mit der WCAG-Stufe für betroffene Projekte.

### Die harte Zahl vorweg

Automatisierte Prüfung findet nach Volumen etwa **die Hälfte der real vorhandenen Fehler**, aber nur rund **ein Drittel der Erfolgskriterien** ist überhaupt maschinell prüfbar. Der Unterschied kommt daher, dass wenige gut automatisierbare Kategorien — Kontrast, Sprachauszeichnung, Name/Rolle/Wert — einen überproportionalen Anteil aller Fehler ausmachen.

Daraus folgt die einzig zulässige Lesart eines grünen Laufs: **die Abwesenheit bekannter, maschinell erkennbarer Fehler.** Kein Beleg für Benutzbarkeit, erst recht keiner für Konformität.

### Verfahren

| Verfahren | Was es prüft | Aussagekraft |
|---|---|---|
| **axe DevTools** (Browser-Extension) | Aktueller DOM-Zustand im Browser | Für den Entwicklungsalltag das nützlichste Werkzeug: sofortiges Feedback am Zustand, den man gerade gebaut hat. Punktuell, nicht reproduzierbar. |
| **axe-core in CI** (`@axe-core/playwright`) | Definierte Seiten und Zustände, automatisiert | Reproduzierbar und regressionssicher. Kann Zustände *nach Interaktion* prüfen (Dialog offen, Menü ausgeklappt, Formular nach Fehlversuch, eingeloggt) — dort sitzen die Fehler, die eine reine Startseiten-Prüfung nie sieht. |
| **pa11y-ci** | URL-Listen oder Sitemap, breit | Breite statt Tiefe: findet, *welche* Seiten auffällig sind. Kann zwei Engines parallel fahren (axe + HTML CodeSniffer). Sieht nur anonyme Sichten und keine Interaktion. |
| **Lighthouse** | Gewichteter Teilausschnitt der axe-Regeln | **Als Kennzahl ungeeignet.** 100 Punkte sind bei massiven Barrieren erreichbar. Wir setzen keine Score-Ziele darauf. |
| **Editoria11y** (Drupal-Modul) | Redaktionelle Inhalte, live im Editor | Schließt eine Lücke, die CI prinzipiell nicht schließt: fehlende Alt-Texte, „hier klicken", Pseudo-Überschriften entstehen in der Produktion, nicht im Repository. |
| **Manuelle Prüfung** | Tastatur-Durchlauf, 200 % Zoom, Screenreader-Stichprobe, Fokusreihenfolge | Der einzige Zugang zu den zwei Dritteln, die Automatisierung nicht abdeckt. Ein Tastatur-Durchlauf durch die wichtigsten Flows kostet Minuten und findet Barrieren, die kein Scanner meldet. |
| **Externe Prüfstelle** | Vollständiges Prüfverfahren durch Dritte | Das Einzige, was gegenüber Dritten belastbar ist. Entsprechend Aufwand und Vorlauf — für betroffene Projekte einzuplanen, nicht kurzfristig zu beschaffen. |

### Beim Interpretieren beachten

- **Ergebnisse sind nicht nur „bestanden/durchgefallen".** axe meldet zusätzlich `incomplete` — Fälle, die es nicht sicher entscheiden konnte. Die gehören gesichtet, nicht ignoriert.
- **Automatisierung findet keine falschen Inhalte.** Dass ein `alt`-Attribut existiert, ist prüfbar; dass es das Bild beschreibt, nicht. Dasselbe gilt für Überschriften, Linktexte und Fokusreihenfolge.
- **Der geprüfte Umfang ist Teil des Ergebnisses.** Eine Aussage über fünf geprüfte Seiten ist eine Aussage über fünf Seiten.
