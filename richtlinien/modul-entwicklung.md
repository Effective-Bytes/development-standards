# Modul-Entwicklung

Custom Module werden nur erstellt, wenn es keine Contrib-Lösung gibt oder sehr spezifische Kundenanforderungen (Business-Logik) umgesetzt werden müssen. Dieses Kapitel listet nur die wichtigsten Punkte, für weiter Informationen sie Creating Modules.

## Struktur Für Drupal 11+ (nicht vollständig!)

```
myproject_example/
├── composer.json                     # Abhängigkeiten & Autoloading (Wichtig für D10/11 Kompatibilität)
├── myproject_example.info.yml        # Metadaten & Abhängigkeiten (Prio: core_version ^10 || ^11)
├── myproject_example.libraries.yml   # CSS/JS Assets Definitionen
├── myproject_example.permissions.yml # Definition eigener Rechte (Permissions)
├── myproject_example.routing.yml     # Routen zu Controllern oder Forms (Symfony Routing)
├── myproject_example.links.menu.yml  # Einträge im Admin-Menü etc.
├── myproject_example.links.task.yml  # Tabs (Local Tasks) auf Seiten
├── myproject_example.services.yml    # DI Container: Registrierung von Services & EventSubscribern
├── myproject_example.install         # Schema-Updates (hook_update_N) & Installations-Logik
├── myproject_example.module          # Legacy: Nur für Hooks, die (noch) nicht via Klasse gehen
├── config/
│   ├── install/                      # Initiale Konfiguration bei Installation
│   └── schema/                       # Schema-Definitionen (Pflicht für Validierung & Übersetzung)
│       └── myproject_example.schema.yml
├── templates/                        # Twig Templates (werden meist via hook_theme registriert)
│   └── myproject-example-list.html.twig
├── tests/                            # PHPUnit Tests (Ein Muss für stabile Module)
│   ├── src/Kernel/                   # Integrationstests mit DB, ohne Browser
│   └── src/Functional/               # Browser-Tests (Mink)
└── src/
    ├── EventSubscriber/              # Events: Der moderne Weg (z.B. statt hook_entity_insert)
    ├── Hook/                         # Hook-Klassen: Ab Drupal 11.1+ (Attributes)
    │   └── UserHooks.php             # Ersetzt .module Hooks, ermöglicht echte Dependency Injection!
    ├── Controller/                   # Routing-Ziele (z.B. Page Callbacks)
    ├── Form/                         # Form API Klassen
    │   ├── SettingsForm.php          # Konfigurations-Formular (ConfigFormBase)
    │   └── ProcessConfirmForm.php    # Bestätigungs-Formular (ConfirmFormBase)
    ├── Plugin/                       # Plugin-System (Erweiterbarkeit)
    │   ├── Block/                    # Custom Blöcke
    │   ├── Field/                    # Custom Formatter, Widgets oder FieldTypes
    │   └── QueueWorker/              # Abarbeitung von Warteschlangen (Cron)
    ├── Entity/                       # Definition eigener Content- oder Config-Entities
    ├── Controller/                   # Routing-Ziele (z.B. Page Callbacks)
    └── Service/                      # Business Logik
        └── ExampleManager.php        # Wird von Events, Hooks oder Controllern aufgerufen
```

## Namenskonventionen

Eigene Module müssen einem klaren Namensschema folgen, um Kollisionen mit Drupal Core oder Contrib Modulen zu vermeiden.

- **Präfix:** Jedes Custom Modul muss mit dem Projekt-Kürzel beginnen (z. B. `myproject_`).
- **Maschinenname:** Kleinbuchstaben und Unterstriche (snake_case). Beispiel: `myproject_checkout_flow`.
- **Namespace:** `Drupal\myproject_checkout_flow\...`

## Dependency Injection (DI)

Dies ist der wichtigste technische Standard in der Modul-Entwicklung.

- **Verbot von `\Drupal::service()`:** Der statische Aufruf des Service Containers (`\Drupal::service('...')`) ist innerhalb von Klassen (Controllern, Services, Plugins, Formularen) verboten.
- **Standard:** Alle Abhängigkeiten müssen über Dependency Injection in die Klasse gereicht werden.
  - Bei Services: über `__construct()` und `services.yml`.
  - Bei Plugins/Controllern: über das Interface `ContainerFactoryPluginInterface` bzw. die `create()` Methode.
- **Ausnahme:** In `.module` Dateien (Hooks) und `.theme` Dateien ist DI technisch nicht möglich; hier darf der statische Wrapper genutzt werden. In Drupal 11.1 und höher sollten hooks in `Hooks/File.php` abgelegt werden, wir wird DI verwendet!
- **Begründung:** Ohne DI ist sauberes Unit-Testing (Mocking von Abhängigkeiten) unmöglich. Zudem macht DI die Abhängigkeiten einer Klasse explizit sichtbar.

## Struktur der `info.yml`

Jedes Modul muss eine saubere `*.info.yml` besitzen.

- **Core Version:** `core_version_requirement: ^10 || ^11` (aktuell halten).
- **Package:** Nutze `Custom` für allgemeine Module oder projektspezifische Kategorien wie `MyProject Features`.
- **Dependencies:** Liste alle Abhängigkeiten zu anderen Modulen explizit auf. Verlasse dich nicht darauf, dass ein Modul „zufällig" aktiviert ist.

## Konfiguration (Config in Code)

- **`config/install`:** Standard-Konfigurationen, die bei der Installation des Moduls benötigt werden.
- **`config/schema`:** Pflicht: Jede Konfiguration, die ein Modul bereitstellt, muss ein Schema (`*.schema.yml`) haben. Dies ist essenziell für Übersetzbarkeit und Validierung (und wird von PHPStan/Tests geprüft). Drupal 11+ Wirft deprecation warnings, wenn ein Schema fehlt!

## Anlegen von Routen

### Namensgebung

Routen können mit nur 2 Begriffen beginnen.

- Der eigene Modulname (Standard)
- `entity` – Ausschließlich, wenn es sich bei der Route um eine allgemeine Entity Operation handelt (view, edit, delete, custom ...)

Die Struktur ist auch im Core nicht einheitlich, was jedoch eingehalten werden sollte sind folgende Punkte:

- Alle Buchstaben sind klein als Trennzeichen wird `.` verwendet
- Routen spiegeln den Pfad sinnvoll wider, Reihenfolge entspricht der URL-Struktur
- Es sollte nicht der gesamte Pfad Teil des Namens sein – nur das, was nicht aus dem Rest der Route hervorgeht. Bestehende Pfade wie `/admin/structure` werden nicht im Namen der Route festgehalten
- Wenn die URL eine Aktion beinhaltet, steht sie am Ende. Sie folgt nicht immer einem REST-API Schema z.B. `comment.multiple_delete_confirm` ist auch ok

**Beispiele (direkt aus dem Core):**

| Modul | Pfad | Name |
|---|---|---|
| comment | `/comment/reply/{entity_type}/{entity}/{field_name}/{pid}` | `comment.reply` |
| workflows | `/admin/config/workflow/workflows/manage/{workflow}/state/{workflow_state}/delete` | `entity.workflow.delete_state_form` |
| node | `/node/{node}/revisions/{node_revision}/revert` | `node.revision_revert_confirm` |

### Konfiguration

Admin routen sollten die Option entsprechend setzen:

```yaml
options:
  _admin_route: TRUE
```

## Sanitize

Jedes Modul sollte seine eigenen Daten bereinigen. Drush kommt bereits mit dem Befehl `drush sql:sanitize`, welcher Userdaten löscht oder anonymisiert. Dieser sollte erweitert werden. Dadurch kann jedes Modul selbst bestimmen was bleibt und was nicht und löschen des Modules löscht auch gleich die Sanitization-Logik mit:

```php
<?php

namespace Drupal\myproject_example\Drush\Commands;

// Imports

/**
 * Drush sql:sanitize plugin to clean myproject_example data.
 */
final class MyprojectExampleSanitizeCommands extends DrushCommands implements SanitizePluginInterface {

  use AutowireTrait;

  public function __construct(
    protected readonly EntityTypeManagerInterface $entityTypeManager,
    protected readonly Connection $database,
  ) {
    parent::__construct();
  }

  /**
   * {@inheritdoc}
   */
  #[CLI\Hook(type: HookManager::POST_COMMAND_HOOK, target: 'sql:sanitize')]
  public function sanitize($result, CommandData $commandData): void {
    // Eigene Bereinigungslogik hier.
    $this->logger()->info(dt('myproject_example: Daten bereinigt.'));
  }

  /**
   * {@inheritdoc}
   */
  #[CLI\Hook(type: HookManager::ON_EVENT, target: 'sql-sanitize-confirms')]
  public function messages(array &$messages, InputInterface $input): void {
    // Beschreibt dem Nutzer, was sanitize() gleich tun wird.
    $messages[] = dt('myproject_example: Beschreibung der Bereinigung.');
  }

}
```

## Weiteres und Grundsätze

Die Entwicklung von Modulen kann sehr komplex werden. In der Struktur oben sieht man eine grobe Übersicht. Die wichtigsten Zuständigkeiten werden in [Architektur](architektur.md) erläutert. Alles weitere liegt in der Verantwortung jedes Entwicklers.

Solltest du dir bei einer Umsetzung nicht sicher sein, melde dich in den entsprechenden Kanälen im Team!

Allgemeine Grundsätze, welche wir beachten sollten, sind:

- **Ein Modul eine Aufgabe** – Ein Modul soll nur eine Aufgabe erfüllen. Wenn mehrere Aufgaben zu einem Modul gehören, sollte es in mehrere Module aufgeteilt werden.
- **Wiederverwendbar** – Sofern möglich entwickle das Modul so, dass es theoretisch jederzeit veröffentlicht werden könnte bzw. bei einem anderen Kunden eingesetzt werden kann.
