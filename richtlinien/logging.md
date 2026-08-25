# Logging

Drupal verwendet den [PSR-3](https://www.php-fig.org/psr/psr-3/) Standard von PHP.

## Log-Level

| Level | Wann | Beispiel |
|---|---|---|
| emergency alert critical | System nicht nutzbar, sofortiger Eingriff nötig | DB nicht erreichbar, Site komplett down |
| error | Aktion fehlgeschlagen, Eingriff nötig | Exception, fehlgeschlagener API-Call, fehlgeschlagener Import |
| warning | Unerwartet, aber abgefangen | Fallback griff, Retry erfolgreich, veraltete Config |
| notice | Geschäftsrelevantes Ereignis | Bestellung abgeschlossen, User-Login, Import-Lauf gestartet |
| info | Technischer Lifecycle | Cron gestartet, Deploy-Hook, Cache geleert |
| debug | Detaillierte Diagnose | Zwischenwerte, Query-Parameter, API-Response-Body |

## Welcher Logger wann

| Situation                                    | Logger |
|----------------------------------------------|---|
| (Standard) Service, Controller, Plugin       | `Psr\Log\LoggerInterface` per DI, Channel aus `services.yml` |
| Channel-Name kommt aus Variable zur Laufzeit | `LoggerChannelFactoryInterface` per DI |
| Hook, Procedural-Code in `*.module`          | `\Drupal::logger('my_module')` |

```
services:
  mymodule.my_service:
    class: Drupal\mymodule\MyService
    arguments:
      - '@logger.channel.mymodule'   # <- Channel benutzen

  logger.channel.mymodule:           # < Channel deklarieren
    parent: logger.channel_base
    arguments: ['mymodule']
```

## Platzhalter

Immer Platzhalter Verwenden:

```php
$this->logger->warning('Benutzer @uid hat Seite @path nicht gefunden', ['@uid' => $uid, '@path' => $path]);
```
