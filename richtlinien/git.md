# Git

## Repository-Namen

### Grundregeln

- **Kleinbuchstaben** – fast universell empfohlen, da URLs case-sensitiv sein können
- **Keine Leerzeichen** – stattdessen Trennzeichen verwenden
- **Kurz und beschreibend** – idealerweise 2–5 Wörter
- **Nur ASCII-Zeichen** – keine Umlaute (ä, ö, ü), Sonderzeichen oder Emojis

### Trennzeichen: Bindestrich vs. Unterstrich

| Style | Beispiel | Empfehlung |
|---|---|---|
| **kebab-case** | `my-awesome-project` | ✅ Am weitesten verbreitet |
| **snake_case** | `my_awesome_project` | Üblich in Python-Projekten |
| **PascalCase** | `MyAwesomeProject` | Eher für .NET / C#-Projekte |
| **camelCase** | `myAwesomeProject` | Selten, meist vermieden |

**kebab-case** ist der de-facto-Standard auf GitHub, da er URL-freundlich und gut lesbar ist.

## Arbeiten mit KI

Diese Richtlinie regelt, wie wir die Nutzung von KI (z.&nbsp;B. Claude) in unserer Arbeit nach außen kommunizieren – insbesondere in Git-Commits.

### Kennzeichnung von KI-generiertem Code

Wir kennzeichnen KI-Beteiligung an Code über den `Co-Authored-By`-Trailer im Commit. GitHub erkennt diesen Trailer und zeigt die KI mit Benutzer und Icon an den jeweiligen Zeilen an.

### Wann wird die KI als Co-Author deklariert?

**Ja, Co-Author deklarieren, wenn:**

- Die KI Code (oder Teile davon) **selbst geschrieben** hat.
- Code oder Inhalte aus der KI-Ausgabe kopiert wurden.

**Nein, keine Deklaration, wenn:**

- Die KI lediglich als Suchmaschine genutzt wurde.
- Die KI für **Konzeption, Ideensammlung oder Diskussion** genutzt wurde und der Code **eigenständig** umgesetzt wurde.

Dies gilt immer für den gesamten Commit, wir machen uns nicht die Mühe einzelne Zeilen zu markieren

### Format des Trailers

Der Trailer steht in der letzten Zeile der Commit-Message, **durch eine Leerzeile von der vorherigen Beschreibung getrennt.**

**Beispiel:**

```
git commit -m "Commit message
- detailed changes
<Leerzeile>
Co-Authored-By: Claude <noreply@anthropic.com>"
```

**Anwendung:**

Claude selbst schreibt oft noch das spezifische Modell in den Namen, das ist ok, aber nicht notwendig.

Manuelle Eingabe:
- `git commit -m <Message> --trailer "Co-Authored-By: Claude <noreply@anthropic.com>"`
- Oder einen Alias erstellen
```shell
git config --global alias.commit-claude '!f() { if [ -z "$1" ]; then echo "❌ Bitte eine Commit-Message angeben: git commit-claude \"message\""; exit 1; fi; git commit -m "$1" --trailer "Co-Authored-By: Claude <noreply@anthropic.com>" "${@:2}"; }; f'
```
  - Anschließend kann man `git commit-claude <Message>` benutzen und der Trailer wird automatisch angehangen.
  - "commit-claude" ist nur ein Vorschlag und kann nach Belieben verändert werden.
