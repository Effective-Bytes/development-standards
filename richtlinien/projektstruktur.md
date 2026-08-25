# Projektstruktur & Zuständigkeiten

In diesem Kapitel werden strukturelle Vorgaben definiert, welche zwingend eingehalten werden sollten.

## Standard Ordnerstruktur

```
. (Project Root / Git Root)
├── .ddev/                      # DDEV Konfiguration (optional, je nach lokaler Umgebung)
├── config/
│   └── sync/                   # Config Export / Import (config:export / config:import)
├── patches/                    # Lokale Patches für Contrib Module (composer-patches)
├── recipes/                    # Drupal Recipes (ab Drupal 10.3+)
├── scripts/                    # Build-Skripte, Deployment-Helfer etc.
├── tests/                      # Projektweite, übergreifende Tests
│   ├── behat/                  # Behat Feature-Tests
│   ├── cypress/                # Cypress E2E-Tests
│   └── phpunit/                # PHPUnit Integrationstests (projektübergreifend)
├── vendor/                     # Composer Dependencies (NIEMALS committen!)
├── composer.json
├── composer.lock
├── phpunit.xml                 # PHPUnit Konfiguration
└── web/                        # Web Root (Docroot)
    ├── .htaccess
    ├── index.php
    ├── modules/
    │   ├── contrib/            # Community Module (via Composer)
    │   └── custom/             # Eigene Module
    │       ├── myproject_core/         # Kern-Modul mit globalen Services/Helpers
    │       ├── myproject_news/         # Feature: News
    │       └── myproject_events/       # Feature: Events
    ├── sites/
    │   └── default/
    │       ├── files/          # Öffentliche Dateien (nicht committen!)
    │       ├── settings.php    # Haupt-Settings (importiert weitere)
    │       ├── settings.ddev.php       # DDEV-spezifische Settings
    │       ├── settings.local.php      # Lokale Entwickler-Settings (nicht committen!)
    │       └── settings.production.php # Produktions-Settings
    └── themes/
        ├── contrib/            # Base-Themes (Gin, Olivero, Bootstrap, etc.)
        └── custom/
            ├── myproject_base/     # Technisches Base-Theme
            └── myproject_theme/    # Design/Frontend Sub-Theme
```
