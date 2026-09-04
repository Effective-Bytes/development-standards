# Kommentare

## Geltung

Diese Richtlinie gilt für **alle Dateitypen im Projekt** — PHP, JavaScript/TypeScript,
Twig, SCSS/CSS, YAML, Shell, Konfiguration. Sie ist deshalb bewusst sprachagnostisch
formuliert.

## Sprache

**Kommentare sind englisch.** Das gilt für jede Form von Kommentar: einzeilige und
mehrzeilige Kommentare, DocBlocks samt `@param`/`@return`-Beschreibungen,
Twig-Kommentare (`{# … #}`), YAML-Kommentare.

```php
// ❌ Preis ohne Steuer, weil die Steuer erst im Checkout feststeht.
// ✅ Price without tax; the tax rate is only known at checkout.
```

### Warum

Wer den Code später liest, steht heute nicht fest. Ein Projekt wird veröffentlicht, an
eine andere Agentur übergeben oder von jemandem weitergeführt, der kein Deutsch kann.
Englische Kommentare bleiben in jedem dieser Fälle nutzbar — nachträglich übersetzt sie
niemand.
