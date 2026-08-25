# Contrib First Policy

## 1. Das Kernprinzip (Warum wir das tun)

**Zielgruppe:** Alle

"Contrib First" bedeutet, dass wir bei jedem neuen Feature prüfen, ob wir es so bauen können, dass es der Drupal-Community zugutekommt. Wir schreiben Software nicht als "Wegwerfprodukt" für einen einzelnen Kunden, sondern als nachhaltiges, wiederverwendbares Gut.

Unsere Gründe:

- **Wirtschaftlichkeit & Wartung:** Contrib-Module werden von der Community mitgewartet (Sicherheitsupdates, Bugfixes). Custom Code müssen wir allein warten. Langfristig ist Contrib günstiger.
- **Sicherheit:** Mehr Augen auf dem Code bedeuten weniger Sicherheitslücken. Zudem übernimmt das Drupal Security Team den Support für stabile Module.
- **Sichtbarkeit:** Es positioniert unsere Agentur und unsere Kunden als Technologieführer.
- **Vermeidung des "Geschenks, das nie passiert":** Kunden zahlen später selten dafür, existierenden Custom Code in Open Source umzuwandeln. Wenn wir es nicht sofort als Contrib bauen, passiert es meist nie.

## 2. Der Entscheidungs-Loop (Wann machen wir Contrib?)

**Zielgruppe:** Requirements Engineers (RE) & Projektmanager (PM)

Bevor eine Zeile Code geschrieben wird, durchläuft jedes Feature diesen Entscheidungsbaum (basierend auf):

1. **Existiert es schon?** Können wir Core oder bestehende Contrib-Module nutzen?
   - Falls JA: Nutzen und konfigurieren.
2. **Fehlt eine Kleinigkeit?** Können wir ein bestehendes Modul patchen oder erweitern?
   - Falls JA: Erstelle einen Patch/Merge Request (MR) auf Drupal.org.
3. **Ist es neu, aber generalisierbar?** Ist das Problem generisch (z.B. "Datenexport als CSV") oder kundenspezifisch (z.B. "Export der Mitarbeiterdaten der Firma Müller")?
   - Falls generisch: Wir bauen ein neues Contrib-Modul.
   - Falls spezifisch: Wir bauen Custom Code, prüfen aber regelmäßig, ob Teile davon generalisiert werden können.

**Wichtige Checks für PMs:**

- **Vertragslage:** Erlaubt der Kundenvertrag Open Source Veröffentlichungen?
- **Budget:** Contrib-Entwicklung kann initial minimal länger dauern (saubere Abstraktion, Tests), spart aber massiv Wartungskosten. Dies muss dem Kunden kommuniziert werden.

## 3. Workflow für Requirement Engineers

**Aufgabe:** Das Mindset schärfen

- **Abstraktion bei der Anforderung:** Beschreibe Anforderungen nicht mit dem Kundennamen. Statt "Header für Kunde X", definiere "Konfigurierbarer Header-Block".
- **Namensgebung:** Wähle generische, suchmaschinenfreundliche Namen für Module, nicht projektspezifische Kürzel.
- **MVP-Definition:** Trenne "Must-haves" von "Nice-to-haves". Contrib-Module starten oft als MVP und wachsen durch die Community.

## 4. Workflow für Entwickler (Technical Execution)

**Aufgabe:** Die Umsetzung im Projektkontext

Wenn die Entscheidung für "Contrib" oder "Patch" gefallen ist, nutzen wir spezialisierte Tools, um direkt im Projektkontext zu entwickeln, ohne den Vendor-Ordner zu hacken.

### A. Vorbereitung & Setup

- **Projektseite erstellen:** Lege das Projekt auf Drupal.org an.
- **Infrastruktur:** Füge `gitlab-ci.yml` für Tests und Code-Quality-Checks hinzu. Nutze DDEV für lokale Entwicklung.

### B. Entwicklung im Projekt ("Symlink-Methode")

Um ein Contrib-Modul im Kundenprojekt zu entwickeln, nutzen wir Composer-Tools (wie `joachim-n/drupal-project-contrib-development`), um das installierte Paket durch einen Git-Clone zu ersetzen.

**Schritt-für-Schritt:**

1. **Clone aktivieren:** Nutze den Befehl `composer drupal-contrib:switch-clone [modul_name]`. Dies klont das Repo in einen Unterordner (z.B. `/repos`) und erstellt einen Symlink in `/modules/contrib`.
2. Composer nutzt nun diesen lokalen Klon statt des Pakets aus Packagist.
3. **Entwickeln:** Erstelle einen Branch, schreibe Code und Tests. Die Änderungen sind sofort im Projekt sichtbar und testbar.
4. **Commit & Push:** Pushe die Änderungen in einen Merge Request (MR) auf Drupal.org.

### C. Integration ins Projekt

Je nach Status des MRs gibt es zwei Wege zurück zum stabilen Projektzustand:

- **Szenario 1: Patch anwenden (Feature ist noch nicht gemerged)**
  Nutze `composer drupal-contrib:apply-patch-from-branch [modul_name]`.
  Dies erstellt automatisch einen Patch aus deinem Feature-Branch und trägt ihn in die `composer.json` ein (via `cweagans/composer-patches`).
- **Szenario 2: Zurück zum Release (Feature ist gemerged)**
  Nutze `composer drupal-contrib:switch-package [modul_name]`.
  Entfernt den Symlink und lädt das reguläre Paket (oder die Dev-Version) via Composer.

## Zusammenfassung für das Handbuch

- **RE:** Denkt generisch.
- **PM:** Verkauft Nachhaltigkeit und Sicherheit.
- **Dev:** Nutzt Tooling für nahtlosen Switch zwischen Projekt- und Contrib-Code.
