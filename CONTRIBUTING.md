> [Deutsche Version](./CONTRIBUTING.de.md)

# Contributing to the `KoliBri` theme `BWSt`

Thank you for your interest in contributing to this project! This guide will help you get started.

## Entwicklungsumgebung einrichten

### Voraussetzungen

- Node.js 22+ und pnpm
- Git

### Setup

```bash
# Repository klonen
git clone <your-repository-url>
cd <your-theme-name>

# Abhängigkeiten installieren
pnpm install

# Playwright-Browser installieren
pnpm exec playwright install

# Entwicklung starten (Theme Watch & Beispiel-App)
pnpm start
```

Der Start-Befehl kombiniert `rollup --watch` mit einem lokalen Beispiel auf Basis von `@public-ui/sample-react`. So können Sie die Styles im Kontext einer Anwendung prüfen, während das Theme kontinuierlich neu gebaut wird.

Sind Sie fertig, überprüfen Sie das Ergebnis mit den Snapshot-Tests und checken Sie bei Bedarf aktualisierte Referenz-Snapshots ein:

```bash
pnpm test        # Snapshots prüfen
pnpm test-update # Referenz-Snapshots aktualisieren
```

## Architektur verstehen

### CSS Layer Struktur

Dieses Theme verwendet CSS Cascade Layers für vorhersagbares Styling:

```scss
// Layer-Reihenfolge (niedrigste bis höchste Spezifität)
@layer kol-theme-global; // Globale Theme-Styles
@layer kol-theme-component; // Component-spezifische Styles
```

### Dateiorganisation

````text
src/
├── components/              # Component-Styles (@layer kol-theme-component)
│   ├── button.scss
│   ├── input.scss
│   └── ...
├── global.scss              # Globale Theme-Styles (@layer kol-theme-global)
├── global/                  # Zusätzliche globale Styles (z. B. Icons)
├── mixins/                  # Sass Mixins und Utilities (keine Layer)
├── globals.d.ts             # TypeScript Deklarationen für SCSS-Module
└── index.ts                 # Theme-Einstiegspunkt (exportiert `BWSt`)

## Styling-Regeln

### Layer-Durchsetzung

Custom Stylelint-Regeln stellen ordnungsgemäße Layer-Nutzung sicher:

1. **Component-Dateien** (`src/components/*.scss`) **MÜSSEN** `@layer kol-theme-component` verwenden
2. **Global-Datei** (`src/global.scss`) **MUSS** `@layer kol-theme-global` verwenden
3. **Utility-Dateien** (Mixins, Helfer) **DÜRFEN KEINE** Layer verwenden
4. Nur zugelassene Layer-Namen sind erlaubt: `kol-theme-global`, `kol-theme-component`

### Web Component Encapsulation

**WICHTIG**: Theme-Dateien **MÜSSEN** `:host` statt `:root` verwenden für Web Component Kapselung:

```scss
// ✅ Korrekt: :host für Web Component Styling
@layer kol-theme-global {
	:host {
		--font-family: var(--bwst-font-family);
		background-color: white;
		color: black;
	}
}
@@ -169,189 +172,190 @@ Diese Regeln stellen sicher:
}
````

2. Erstellen und testen:

```bash
pnpm build
pnpm lint
pnpm test
```

### Globale Styles ändern

`src/global.scss` innerhalb des globalen Layers bearbeiten:

```scss
@layer kol-theme-global {
	:host {
		// Globale Theme-Variablen und Styles
	}
}
```

### Utilities erstellen

Utilities in `src/mixins/` **ohne** Layer hinzufügen:

```scss
// src/mixins/my-utility.scss
@mixin my-utility {
	// Utility-Styles (kein @layer nötig)
}
```

## Build-Prozess

### Entwicklungs-Build

```bash
pnpm dev    # Watch-Modus mit Hot Reload
pnpm start  # Entwicklungsserver
```

### Produktions-Build

```bash
pnpm build  # Optimierter Produktions-Build
```

### Build-Output

- `assets/` - Statische Assets und Schriftarten
- `dist/` - Kompilierte CSS-Dateien

## Testing

### Visual Regression Tests

```bash
pnpm test        # Visual Tests ausführen
pnpm test-update # Visual Snapshots aktualisieren
```

#### Assets und Visual Tests

Die Visual Tests benötigen korrekt geladene Assets (Schriftarten und Icons), um aussagekräftige Screenshots zu erstellen:

- **inject-assets.css** - Zentrale Asset-Injektionsdatei mit @import-Anweisungen für:
  - `assets/material-symbols-subset/style.css` - Reduziertes Material Icons Set
  - `assets/fira-sans-v17-latin/style.css` - Fira Sans Schriftfamilie (400-700)

Die Visual Tests setzen `THEME_CSS=$(pwd)/inject-assets.css`, um genau diese Datei in die Beispiel-Anwendung zu laden.

**Wichtig**: Ohne korrekt geladene Assets würden Visual Tests fälschlicherweise als "geändert" erkannt, da Fallback-Schriftarten oder fehlende Icons verwendet würden.

### Code-Qualität

```bash
pnpm lint        # Stylelint + ESLint
pnpm format      # Prettier-Formatierung
```

## Vor dem Committen

Führen Sie immer den vollständigen Validierungs-Workflow aus:

```bash
pnpm build   # Sauberen Build sicherstellen
pnpm format  # Formatierung korrigieren
pnpm lint    # Code-Qualität prüfen
pnpm test    # Visual Tests validieren
```

## Code-Style

- Verwenden Sie **Tabs** für Einrückung (außer Markdown-Dateien verwenden Leerzeichen)
- Zeilenlänge: **160 Zeichen**
- Einfache Anführungszeichen in SCSS/CSS
- BEM-Namenskonvention für CSS-Klassen befolgen
- Semantische Variablennamen verwenden

## Layer-Richtlinien

1. **Niemals Layer-Regeln umgehen** - Alles CSS muss in entsprechenden Layern sein
2. **Component-Isolation** - Component-Styles betreffen nur ihre Component
3. **Globale Zurückhaltung** - Globaler Layer nur für theme-weite Variablen und Host-Styles
4. **Kein Layer-Mixing** - Layered und nicht-layered CSS nicht in derselben Datei mischen

## Konfigurationsdateien

| Datei                | Zweck                                          |
| -------------------- | ---------------------------------------------- |
| `.stylelintrc.json`  | Stylelint-Konfiguration mit Custom-Regeln      |
| `eslint.config.js`   | ESLint-Konfiguration für JavaScript/TypeScript |
| `rollup.config.js`   | Build-Konfiguration                            |
| `prettier.config.js` | Code-Formatierungsregeln                       |
| `stylelint-rules/`   | Custom Stylelint-Regel-Implementierungen       |

## Fehlerbehebung

### Häufige Probleme

**Stylelint Layer-Fehler:**

```
CSS rule "selector" must be inside @layer kol-theme-component
```

→ Wrappen Sie alles CSS in den entsprechenden Layer für den Dateistandort.

**Build-Fehler:**

```bash
pnpm clean  # Dist und Cache löschen
pnpm install    # Abhängigkeiten neu installieren
pnpm build  # Neu erstellen
```

**Visual Test-Fehler:**

```bash
pnpm test-update  # Snapshots aktualisieren wenn Änderungen beabsichtigt sind
```

### Hilfe bekommen

1. Untersuchen Sie bestehende Component-Implementierungen in `src/components/`.
2. Prüfen Sie die Custom Stylelint-Regeln in `stylelint-rules/`.
3. Führen Sie `pnpm lint` für spezifische Fehlermeldungen aus.

## Service Worker Cache-Probleme in Chrome

**Problem:** Assets werden trotz "Disable cache" nicht aktualisiert. Chrome lädt weiterhin alte Versionen, auch nach hartem Reload.

**Ursache:** In Chrome hängt sehr wahrscheinlich ein **Service Worker** dazwischen. Das DevTools-Häkchen „Disable cache" betrifft nur den **HTTP-Cache**, nicht den **Cache Storage** eines Service Workers. Der SW serviert dann weiterhin alte Assets.

#### Sofortige Lösung

1. **DevTools → Application → Service Workers**
   - „**Unregister**" klicken
   - Optional: „**Update on reload**" aktivieren

2. **Application → Clear storage**
   - Alle Häkchen an („Unregister service workers", „Cache storage", „IndexedDB", …)
   - **Clear site data** klicken

3. **Neu laden** (am besten mit Rechtsklick auf den Reload-Button → **Empty cache and hard reload**)

#### Dauerhaft vermeiden

- **Service Worker deaktivieren:** In `main.tsx`/`index.tsx` sicherstellen, dass der SW **nicht registriert** wird:

  ```javascript
  // serviceWorker.unregister() verwenden
  // oder registerServiceWorker entfernen
  ```

- **Nur in Produktion:** SW nur in Prod-Builds registrieren:

  ```javascript
  if (process.env.NODE_ENV === 'production') {
  	// Service Worker nur in Production registrieren
  }
  ```

- **Cache-Headers korrekt setzen:**
  - **`index.html`** bekommt `Cache-Control: no-store`
  - Hash-Dateien (`*.js`, `*.css`) dürfen gecacht werden

- **DevTools Einstellungen:** In Chrome DevTools (Application → Service Workers) **„Bypass for network"** oder **„Update on reload"** aktivieren während der Entwicklung

- **Keine falschen Proxy-Headers:** Bei Vite/Webpack keine Reverse-Proxy-Header oder CDN-Layer verwenden, die `max-age` auf HTML setzen

#### Alternative Ursachen

Falls es **kein** Service Worker ist:

- **DevServer-Headers prüfen:** HTML sollte `no-store` haben, Assets mit Hash + langes Caching
- **Back/Forward Cache:** Chrome's bfcache kann verwirren - teste mit `location.reload(true)` oder `window.onpageshow`-Handler (`event.persisted`)

**In 90% der Fälle ist es der Service Worker.** Unregister + Clear Storage behebt das sofort.

## Pull Request Prozess

1. Forken Sie das Repository
2. Erstellen Sie einen Feature-Branch (`git checkout -b feature/amazing-feature`)
3. Committen Sie Ihre Änderungen (`git commit -m 'Add amazing feature'`)
4. Pushen Sie zum Branch (`git push origin feature/amazing-feature`)
5. Öffnen Sie einen Pull Request

### Pull Request Richtlinien

- Beschreiben Sie Ihre Änderungen ausführlich
- Fügen Sie Screenshots bei UI-Änderungen hinzu
- Stellen Sie sicher, dass alle Tests bestehen
- Folgen Sie den Code-Style-Richtlinien
- Aktualisieren Sie die Dokumentation falls nötig

## Browser-Unterstützung

- Moderne Browser mit CSS Cascade Layers Unterstützung
- Chrome 99+, Firefox 97+, Safari 15.4+
- Für ältere Browser sollte ein CSS Layers Polyfill verwendet werden

---

Vielen Dank für Ihren Beitrag!
