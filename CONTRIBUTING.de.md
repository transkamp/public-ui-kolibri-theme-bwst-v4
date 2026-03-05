> [English version](./CONTRIBUTING.md)

# Contributing zum `KoliBri` Theme `BWSt`

Vielen Dank für Ihr Interesse am Beitrag zu diesem Projekt! Diese Anleitung hilft Ihnen beim Einstieg.

## Entwicklungsumgebung einrichten

### Voraussetzungen

- Node.js 22+ und pnpm
- Git

### Setup

```bash
# Repository klonen
git clone https://gitlab.opencode.de/bwst-ux/bwst-developer-kit.git
cd bwst-developer-kit

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

```text
src/
├── components/              # Component-Styles (@layer kol-theme-component)
│   ├── button.scss
│   ├── input.scss
│   └── ...
├── global.scss              # Globale Theme-Styles (@layer kol-theme-global)
├── global/                  # Zusätzliche globale Styles (z. B. Icons)
├── tokens/                  # Lokale Design Tokens & Utilities
├── mixins/                  # Sass Mixins und Utilities (keine Layer)
├── globals.d.ts             # TypeScript Deklarationen für SCSS-Module
└── index.ts                 # Theme-Einstiegspunkt (exportiert `BWSt`)
```

**WICHTIG**: Verwenden Sie Design Tokens aus Ihrem Design System Package, wenn verfügbar. Für Tokens, die noch nicht aus dem Package konsumierbar sind, verwalten Sie lokale Dateien in `src/tokens/` als Teil des Themes.

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

// ❌ Falsch: :root umgeht Web Component Encapsulation
@layer kol-theme-global {
	:root {
		--font-family: var(--bwst-font-family); // Triggers lint error
	}
}
```

**Grund**: `:root` Selektoren umgehen die Shadow DOM Kapselung von Web Components und können mit Host-Seiten-Styles kollidieren. `:host` hält das Styling isoliert innerhalb der Component.

### Beispiel-Nutzung

```scss
// ✅ Korrekt: Component-Datei mit Layer
// src/components/button.scss
@layer kol-theme-component {
	.button {
		background: var(--bwst-color-primary);
		border-radius: var(--bwst-border-radius);
	}
}

// ✅ Korrekt: Global-Datei mit Layer und :host
// src/global.scss
@layer kol-theme-global {
	:host {
		--font-family: var(--bwst-font-family);
	}
}

// ✅ Korrekt: Utility-Datei ohne Layer
// src/mixins/typography.scss
@mixin bwst-heading-style {
	font-family: var(--bwst-font-family);
	font-weight: bold;
}

// ❌ Falsch: Component-Datei ohne Layer
// src/components/button.scss
.button {
	background: red; // Triggers lint error
}
```

## Custom Stylelint-Regeln

### Regel-Übersicht

| Regel                                     | Zweck                                         | Gilt für                  |
| ----------------------------------------- | --------------------------------------------- | ------------------------- |
| `kolibri/require-component-layer`         | Erzwingt `@layer kol-theme-component` Nutzung | `src/components/*.scss`   |
| `kolibri/require-global-layer`            | Erzwingt `@layer kol-theme-global` Nutzung    | `src/global.scss`         |
| `kolibri/no-layer-in-non-component-files` | Verhindert Layer-Nutzung in Utility-Dateien   | Alle anderen SCSS-Dateien |
| `kolibri/layer-name-convention`           | Warnt vor nicht-Standard Layer-Namen          | Alle Dateien              |
| `kolibri/no-root-selector`                | Verhindert `:root` zugunsten von `:host`      | `src/**/*.scss`           |

### Regel-Details

Diese Regeln garantieren:

- **100% CSS-Abdeckung** – ALLES CSS muss in den entsprechenden Layern sein
- **Keine Ausnahmen** – Variablen, Selektoren, Deklarationen, At-Regeln werden alle validiert
- **Klare Trennung** – Components vs. global vs. Utility-Styling
- **Wartbarkeit** – vorhersagbares Cascade-Verhalten

## Entwicklungs-Workflow

### Neue Component-Styles hinzufügen

1. Component-Datei in `src/components/` erstellen:

```scss
// src/components/new-component.scss
@layer kol-theme-component {
	.new-component {
		// Your styles here
	}
}
```

2. Bauen und Testen:

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

Visual Tests benötigen korrekt geladene Assets (Schriftarten und Icons), um aussagekräftige Screenshots zu erstellen:

- **inject-assets.css** - Zentrale Asset-Injektionsdatei mit @import-Anweisungen für:
  - `assets/material-symbols-subset/style.css` - Reduziertes Material Icons Set
  - `assets/fira-sans-v17-latin/style.css` - Fira Sans Schriftfamilie (400-700)

Visual Tests setzen `THEME_CSS=$(pwd)/inject-assets.css`, um diese Datei in die Beispiel-Anwendung zu laden.

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
5. **Respect BWSt UX standards** – never modify files in `src/bwst/`, only consume their variables

## BWSt UX integration rules

- **`@bwst-ux/native` package** – official BWSt UX standards as external dependency
- **CSS import** – use `@import '@bwst-ux/native/dist/bwst.css'` for the BWSt base
- **BWSt variables** – use `var(--bwst-*)` CSS custom properties in theme styles
- **Package updates** – update BWSt UX standards with `pnpm update @bwst-ux/native`
- **Local additions** – extra tokens from `src/bwst/` remain active until the package covers them fully
- **CSS-centric** – the package is optimised for CSS distribution, not granular Sass imports

## Konfigurationsdateien

| Datei                | Zweck                                          |
| -------------------- | ---------------------------------------------- |
| `.stylelintrc.json`  | Stylelint-Konfiguration mit Custom-Regeln      |
| `eslint.config.js`   | ESLint-Konfiguration für JavaScript/TypeScript |
| `rollup.config.js`   | Build-Konfiguration                            |
| `prettier.config.js` | Code-Formatierungsregeln                       |
| `stylelint-rules/`   | Custom Stylelint-Regel-Implementierungen       |

## BWSt Design System integration

### Design tokens

**BWSt design standards are provided via the `@bwst-ux/native` package:**

```scss
// Import the BWSt CSS base
@import '@bwst-ux/native/dist/bwst.css';
```

**Key architectural decision**: the `@bwst-ux/native` package is primarily designed for CSS distribution, not granular Sass imports. Therefore the complete BWSt CSS base is included as a CSS import.

### Using the BWSt UX standards

```scss
// ✅ Correct: use BWSt CSS variables in theme components
@layer kol-theme-component {
	.button {
		background: var(--bwst-color-primary); // Use BWSt variable
		border-radius: var(--bwst-border-radius); // Use BWSt variable
		font-family: var(--bwst-font-family); // Use BWSt variable
	}
}

// ✅ Correct: import BWSt CSS as base
@import '@bwst-ux/native/dist/bwst.css';

// ❌ Not available: granular Sass imports (package limitation)
// @use '@bwst-ux/native/src/scss/core/tokens' as bwst-tokens; // Not supported
```

**Interaction between `src/bwst/` and `@bwst-ux/native`:**

- `@bwst-ux/native` supplies the standard BWSt variables via CSS import
- `src/bwst/` contains additional tokens and workarounds that the package does not yet provide
- Keep local tokens lightweight and document deviations for a future migration
- CSS custom properties (`--kern-*`) remain usable without changes

### Typography

The BWSt typography system is supplied via the CSS base from `@bwst-ux/native`:

- Font families and weights
- Heading styles and hierarchy
- Text sizes and spacing

### Colour system

```scss
// BWSt colour tokens (use only, do not modify!)
--kern-color-primary: #0073e6;
--kern-color-secondary: #6c757d;
--kern-color-success: #28a745;
--kern-color-warning: #ffc107;
--kern-color-danger: #dc3545;
```

## Häufige Probleme

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

1. Überprüfen Sie die [BWSt Styleguide](https://gitlab.opencode.de/bwst-ux).
2. Untersuchen Sie bestehende Component-Implementierungen in `src/components/`.
3. Prüfen Sie die Custom Stylelint-Regeln in `stylelint-rules/`.
4. Führen Sie `pnpm lint` für spezifische Fehlermeldungen aus.

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
