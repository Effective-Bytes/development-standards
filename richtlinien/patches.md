# Patches

Patches werden eingesetzt, wenn ein bestehendes Contrib-Modul angepasst werden muss. Bei allgemeinen Bugfixes und Verbesserungen sind sie eine **temporäre Lösung** – das Ziel ist, den Fix ins Upstream-Modul zurückzuführen. Bei projektspezifischen Anpassungen können Patches dauerhaft bestehen bleiben.

## 1. Erstellen von Patches

### Voraussetzungen

Das zu patchende Modul muss als Git-Klon im Projekt vorliegen. Nutze dazu das in der [Contrib First Policy](contrib-first.md) beschriebene Tooling:

```bash
composer drupal-contrib:switch-clone <modul_name>
```

### Patch aus einem Feature-Branch erzeugen

Der bevorzugte Weg ist der automatisierte Befehl:

```bash
composer drupal-contrib:apply-patch-from-branch <modul_name>
```

Dieser erstellt einen Patch aus dem aktuellen Branch und trägt ihn direkt in die `composer.json` ein.

### Patch manuell erzeugen

Alternativ kann ein Patch manuell mit Git erzeugt werden. Im Verzeichnis des geklonten Moduls:

```bash
# Diff gegen den Upstream-Branch (z. B. 4.0.x)
git diff origin/4.0.x > ../../../patches/<modul_name>-<beschreibung>-<issue_nr>.patch
```

**Namenskonvention:** `<modul_name>-<kurze-beschreibung>-<drupal_issue_nr>.patch`

Beispiel: `webform-fix-email-handler-validation-3456789.patch`

### Speicherort

Patches werden im Projektverzeichnis unter `patches/` abgelegt und versioniert:

```
<projekt-root>/
  patches/
    webform-fix-email-handler-validation-3456789.patch
    ...
```

### Merge Request auf Drupal.org

Handelt es sich um einen Bugfix oder eine Verbesserung, die auch anderen nützt, **soll** der Patch als Merge Request (MR) bzw. Issue Fork auf Drupal.org eingereicht werden. Der Link zum Issue gehört dann als Kommentar in die `composer.json` (siehe unten).

Wie ein MR auf Drupal.org erstellt wird, ist in der offiziellen Dokumentation beschrieben: [Creating merge requests – Drupal.org](https://www.drupal.org/docs/develop/git/using-gitlab-to-contribute-to-drupal/creating-merge-requests)

Bei projektspezifischen Anpassungen, die nicht in das Upstream-Modul zurückgeführt werden sollen, entfällt die Pflicht zum Issue Fork. Die Beschreibung in der `composer.json` muss in diesem Fall klar erkennen lassen, warum der Patch projektspezifisch ist.

#### Patches aus MRs und Issue Forks einbinden

Patches aus MRs oder Issue Forks werden **bevorzugt heruntergeladen und als lokale Datei** unter `patches/` eingecheckt. Der direkte Verweis auf eine Remote-URL ist ebenfalls möglich, birgt jedoch ein Sicherheitsrisiko (siehe unten).

Wird ausnahmsweise ein Remote-URL verwendet, **muss** dieser immer den aktuellsten Commit-Hash enthalten – niemals einen Branch-Namen oder einen generischen MR-Link. Andernfalls kann `composer update` den Patch stillschweigend durch eine neuere Version ersetzen, ohne dass dies explizit angestoßen wurde. Das gilt sowohl für MRs als auch für Issue Forks.

```
# Bevorzugt – Patch lokal eingecheckt
"patches/webform-fix-email-handler-validation-3456789.patch"

# Alternativ korrekt – Remote-URL mit Commit-Hash fixiert
"https://git.drupalcode.org/issue/webform-3456789/-/commit/abc123def456.patch"

# Falsch – Patch-Inhalt kann sich bei composer update ändern
"https://git.drupalcode.org/issue/webform-3456789/-/merge_requests/1.diff"
```

## 2. Einbinden von Patches

Patches werden über Composer eingebunden. Voraussetzung ist, dass `cweagans/composer-patches` installiert ist.

```bash
composer require cweagans/composer-patches
```

### Version 1.x vs. 2.x

Es gibt zwei Versionen des Pakets:

- **1.x:** Patches werden nach dem Hinzufügen automatisch angewendet. Keine weiteren Schritte notwendig.
- **2.x:** Nach dem Hinzufügen eines Patches müssen zwei Befehle ausgeführt werden:
  ```bash
  composer patches-relock   # aktualisiert patches.lock.json
  composer patches-repatch  # wendet alle Patches an
  ```
  Die `patches.lock.json` wird versioniert. Wenn jemand anderes Patches hinzugefügt und die `patches.lock.json` gepusht hat, reicht es, lokal nur `composer patches-repatch` auszuführen.

### Konfiguration in `composer.json`


```json
"extra": {
    "patches": {
        "drupal/webform": {
            "[PROJ-123] Fix email handler validation, see https://drupal.org/i/3456789": "patches/webform-fix-email-handler-validation-3456789.patch"
        }
    }
}
```

### Namenskonvention für Patch-Beschreibungen

Der Schlüssel im Patches-Array folgt diesem Schema:

```
[<Ticket-Nr>] <Name des Drupal Issues>, see https://drupal.org/i/<Issue-Nr>
```

Die Ticketnummer kann in eckigen Klammern oder mit Doppelpunkt angegeben werden:

```
[PROJ-123] Fix email handler validation, see https://drupal.org/i/3456789
PROJ-123: Fix email handler validation, see https://drupal.org/i/3456789
```

Gibt es kein internes Ticket, entfällt das Präfix:

```
Fix email handler validation, see https://drupal.org/i/3456789
```

### CLI-Verwaltung mit `szeidler/composer-patches-cli`

Zur einfacheren Verwaltung über die Kommandozeile kann `szeidler/composer-patches-cli` installiert werden:

```bash
composer require szeidler/composer-patches-cli
```

Verfügbare Befehle:

| Befehl | Beschreibung |
|---|---|
| `composer patch-enable --file='patches.json'` | Patch-Funktionalität aktivieren; `--file` setzt eine spezifische `patches.json`, sonst werden Patches in die `composer.json` geschrieben |
| `composer patch-list <package>` | Alle aktiven Patches für ein Paket anzeigen |
| `composer patch-add <package> <description> <url>` | Patch hinzufügen und anwenden |
| `composer patch-remove <package> <description>` | Patch entfernen |

## 3. Lifecycle eines Patches

Patches haben einen definierten Lebenszyklus und müssen aktiv verwaltet werden:

1. **Erstellen:** Patch lokal erzeugen. Bei allgemeinen Fixes zusätzlich als MR auf Drupal.org einreichen.
2. **Einbinden:** Patch über `composer-patches` ins Projekt einbinden, Drupal.org-Link (falls vorhanden) in der Beschreibung pflegen.
3. **Beobachten:** Bei offenen MRs den Stand auf Drupal.org im Auge behalten.
4. **Entfernen:** Sobald der Fix in einem stabilen Release des Moduls veröffentlicht ist, wird das Modul auf die neue Version angehoben und der Patch entfernt. Projektspezifische Patches bleiben dauerhaft bestehen.

Patches, die nicht mehr gepflegt werden oder deren Upstream-Issue abgelehnt wurde, müssen explizit als dauerhaft markiert oder in ein eigenes Fork-Modul überführt werden.
