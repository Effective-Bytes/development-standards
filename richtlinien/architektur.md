# Architektur: Controller, Hooks, Events, Drush Generate, SDC

## Controller, Hooks & Event Subscriber

### Wann benutze ich was?

Drupal bewegt sich weg von prozeduralen Hooks hin zu objektorientierten Event-Subscribern und Hook-Klassen.

- **Priorität 1: Events (Event Subscriber):** Prüfe immer zuerst, ob ein Hook durch ein Symfony Event ersetzt werden kann (z. B. Präfix `hook_entity_insert` vs. Entity Events). Dies ist der sauberste Weg.
- **Priorität 2: Hook-Klassen (Attributes):** Ab Drupal 11.1 (und teilweise früher via Modul) sollen Hooks in Klassen unter `src/hook` abgelegt werden. Dies ermöglicht echte Dependency Injection und macht `.module`-Dateien überflüssig.
- **Priorität 3: Legacy Hooks (`.module`):** Nur nutzen, wenn technisch nicht anders möglich. Da hier kein DI möglich ist, darf der Hook ausschließlich den Service Container statisch aufrufen und die Arbeit sofort an einen Service weitergeben.

**Merksatz:** Egal ob Event oder Hook – die eigentliche Logik liegt immer in einem Service.

Weitere Informationen: [Subscribe to and dispatch events](https://www.drupal.org/docs/develop/creating-modules/subscribe-to-and-dispatch-events)

> Zitat 29.01.2025
>
> "Drupal events are very much Symfony events."
>
> "(...) and will prepare you for a future where events will (hopefully) replace hooks."

### Was gehört in Controller, Hooks & Event Subscriber?

**Keine Businesslogik in Controller, Hooks oder Event Subscriber!**

Diese Klassen sind lediglich die "Verkehrspolizisten" für Requests (Controller) und Ereignisse (Hooks/Events).

- **Controller:** Nur für die Entgegennahme und Beantwortung eingehender Requests zuständig.
- **Event Subscriber & Hooks:** Nur für die Reaktion auf System-Ereignisse zuständig. Sofern möglich, sollte die Logik möglichst dem Ereignis entsprechen -> Routing während `entity_save` ist schlecht nachvollziehbar!
- Diese Layer müssen „flach" gehalten werden -> Sonst Auslagerung in Service!
- **Services:** Jegliche Businesslogik (Berechnungen, API-Calls, komplexe Queries, Datenmanipulation) gehört in Services.

**Grund:**

- **Wiederverwendbarkeit:** Logik in Controllern/Hooks ist fest an den Request/das Event gekoppelt und schwer wiederverwendbar.
- **Testbarkeit:** Services sind isoliert deutlich einfacher zu testen (Unit Tests).
- **Separation of Concerns:** Klare Trennung zwischen "Was passiert" (Event/Request) und "Wie wird es verarbeitet" (Service).

## Drush Generate

Es bietet sich an für das Erstellen von den meisten Komponenten den Befehl `drush generate` zu verwenden.

**Begründung:**

- **Einheitlich:** Drush generate hält die Strukturierung und Standards perfekt ein. Der Code wird dadurch sehr einheitlich.
- **Schneller und einfacher:** Der Command fragt dich automatisch relevante Fragen ab und setzt sie für dich um. Beispielsweise automatische Dependency-Injection oder das erstellen von Routen oder libraries.

**Beispiele:**

- `drush generate sdc`: Single directory component – Fragt nach dependencies, props, slots etc. – Unterstützt nur Themes, kann keine SDC in Modulen generieren!
- `controller`: Controller mit Route und Berechtigungen und DI in der Controller-PHP

## Single Directory Components

Ab Drupal 10.1 ist es der "Drupal Way" SDC für Wiederkehrende Elemente der Webseite zu verwenden:

- Dazu gehören Buttons, Karten, Teaser
- Dazu gehören Header, Footer, Panels
- Alle Bereiche, welche mehr als einmal verwendet werden, sollten als SDC hinterlegt werden

**Begründung:**

- **Automatisches Asset-Management:** CSS und JS werden nur geladen, wenn die Komponente auf der Seite ist. Kein händisches `libraries.yml`-Management für kleine UI-Teile mehr.
- **Strikte Schnittstellen:** Die `component.yml` zwingt Entwickler dazu, Props (Variablen) zu definieren. Das verhindert "Wildwuchs" an unklar definierten Variablen im Twig.
- **Wartbarkeit:** Da CSS, JS und Twig im selben Ordner liegen, entfällt das Suchen in globalen `css/` oder `js/` Verzeichnissen.
