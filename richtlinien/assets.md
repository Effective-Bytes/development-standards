# Assets

## Libraries einbinden

Externe JS/CSS-Libraries können in Drupal auf mehreren Wegen eingebunden werden. Ohne klare Vorgabe entstehen Inkonsistenzen zwischen Projekten.

**Prioritätenreihenfolge:**

1. **Via Composer** – wenn die Library ein natives Paket auf [packagist.org](https://packagist.org) hat oder über asset-packagist eingebunden werden kann
2. **Via npm/Yarn** – für reine JS/CSS-Libraries; Build-Schritt synchronisiert dist-Dateien nach `web/libraries/`
3. **Direkt herunterladen** – wenn weder Composer- noch npm-Paket verfügbar ist
4. **Per CDN / Remote** – nur als begründete Ausnahme, schlecht für CSP

---

### 1. Via Composer

#### Natives Paket (packagist.org)

```bash
composer require vendor/library-name
```

Wenn das Paket den Typ `drupal-library` deklariert, wird es von `composer/installers` erkannt — landet aber standardmäßig unter `libraries/{name}/` (Projekt-Root, nicht Web-Root). Der Web-Root muss in `installer-paths` und in `drupal-scaffold` konsistent konfiguriert sein (Standard `web/`, kann projektspezifisch abweichen, z. B. `public/` oder `app/`):

```json
"extra": {
    "drupal-scaffold": {
        "locations": { "web-root": "web/" }
    },
    "installer-paths": {
        "web/libraries/{$name}": ["type:drupal-library"]
    }
}
```

Pakete ohne diesen Typ landen in `vendor/` und werden von dort referenziert.

#### npm-Pakete via asset-packagist

[asset-packagist.org](https://asset-packagist.org) stellt npm-Pakete als Composer-Pakete bereit. Der klassische Weg, diese nach `web/libraries/` umzuleiten, war `oomphinc/composer-installers-extender` — dieses Paket wurde jedoch am 6. Mai 2026 archiviert und wird nicht mehr weiterentwickelt.

**Stattdessen** wird das Paket als inline `type: package` in der `composer.json` mit `type: drupal-library` definiert. Damit übernimmt `composer/installers` die Installation nativ — kein Plugin nötig.

```json
"repositories": [
    {
        "type": "package",
        "package": {
            "name": "npm-asset/LIBRARY-NAME",
            "type": "drupal-library",
            "version": "1.2.3",
            "require": {
                "composer/installers": "^2.0"
            },
            "dist": {
                "type": "tar",
                "url": "https://registry.npmjs.org/LIBRARY-NAME/-/LIBRARY-NAME-1.2.3.tgz"
            }
        }
    },
    {
        "type": "composer",
        "url": "https://asset-packagist.org"
    }
]
```

**Wichtig:** Die inline Definition muss in der Repositories-Liste **vor** asset-packagist stehen — sonst gewinnt asset-packagist mit `type: npm-asset` und das Paket landet in `vendor/`.

Die dist-URL für ein npm-Paket folgt immer diesem Schema:
`https://registry.npmjs.org/{name}/-/{name}-{version}.tgz`

Da Version und dist-URL fest verdrahtet sind, muss bei Updates beides manuell in `composer.json` angepasst und danach `composer update PACKAGE-NAME` ausgeführt werden.

### 2. Via npm/Yarn

```bash
npm install library-name
```

Die dist-Dateien werden anschließend via npm-Script nach `web/libraries/LIBRARY-NAME/` kopiert:

```json
"scripts": {
    "copy-libs": "cp -r node_modules/library-name/dist web/libraries/library-name/dist"
}
```

### 3. Direkt herunterladen

Library manuell herunterladen und unter `{web-root}/libraries/LIBRARY-NAME/` ablegen. Da `{web-root}/libraries/` per `.gitignore` ausgeschlossen ist, muss eine Ausnahme hinzugefügt werden:

```
!/web/libraries/LIBRARY-NAME/
```

### Einbinden (1, 2 & 3)

In der `*.libraries.yml` des Moduls oder Themes — Pfade ab Web-Root mit führendem `/`:

```yaml
my-library:
  version: 1.2.3
  js:
    /libraries/LIBRARY-NAME/dist/library.min.js: { minified: true }
  css:
    component:
      /libraries/LIBRARY-NAME/dist/library.min.css: { minified: true }
```

### 4. Per CDN / Remote

```yaml
my-library:
  remote: https://cdn.example.com/library/1.2.3
  version: 1.2.3
  license:
    name: MIT
    url: https://example.com/license
    gpl-compatible: true
  js:
    https://cdn.example.com/library/1.2.3/library.min.js:
      type: external
      minified: true
  css:
    component:
      https://cdn.example.com/library/1.2.3/library.min.css:
        type: external
        minified: true
```
