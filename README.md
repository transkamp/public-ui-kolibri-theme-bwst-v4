> [Deutsche Version](./README.de.md)

# Custom theme for KoliBri

A custom theme for the accessible component library [KoliBri](https://github.com/public-ui/kolibri).

## Installation

```bash
npm install @public-ui/theme-bwst @public-ui/components
```

### Copy assets

The theme package and KoliBri components contain important assets (fonts and icons) that must be copied to your project. For a platform-independent solution, we recommend using the `cpy-cli` package:

```bash
npm install --save-dev cpy-cli
```

Then create npm scripts in your `package.json`:

```json
{
	"scripts": {
		"postinstall": "npm run copy-assets",
		"copy-assets": "npm run copy-theme-assets && npm run copy-kolibri-assets",
		"copy-theme-assets": "cpy 'node_modules/@public-ui/theme-bwst/assets/**' 'public/assets/theme' --parents",
		"copy-kolibri-assets": "cpy 'node_modules/@public-ui/components/assets/**' 'public/assets/theme' --parents"
	}
}
```

**Important:** Add the theme assets folder to your `.gitignore` because these files are copied automatically on every `npm install`:

```gitignore
# Theme Assets (copied automatically)
public/assets/theme/
```

### Include assets

After copying the assets, you must include them in your application:

#### Option 1: Include via HTML

```html
<!doctype html>
<html lang="en">
	<head>
		<meta charset="UTF-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1.0" />
		<title>Theme Demo</title>

		<!-- KoliBri assets -->
		<link rel="stylesheet" href="/assets/theme/codicon.css" />
	</head>
	<body>
		<!-- Your KoliBri components -->
	</body>
</html>
```

#### Option 2: Include via SCSS/CSS

```scss
// In your main.scss or styles.scss
@import url('/assets/theme/codicon.css');
```

## Usage

```typescript
import { register } from '@public-ui/components';
import { BWSt } from '@public-ui/theme-bwst';
import { defineCustomElements } from '@public-ui/components/loader';

register(BWSt, defineCustomElements)
	.then(() => {
		// Theme and KoliBri components are loaded
	})
	.catch(console.warn);
```

## Features

- ♿ **Accessible** - WCAG-compliant styles with correct contrast ratios
- 📱 **Responsive** - Mobile-first approach
- 🔧 **CSS Layers** - Modern layer architecture for better maintainability

## Design Tokens

The theme exposes the following CSS custom properties. Each token reads from a `--kolibri-*` override first, falling back to the BWSt default value.

### Layout

| Token | Kolibri override | Default |
|-------|-----------------|---------|
| `--border-radius` | `--kolibri-border-radius` | `0.25rem` (4px) |
| `--border-width` | `--kolibri-border-width` | `1px` |
| `--font-size` | `--kolibri-font-size` | `1rem` (16px) |
| `--spacing` | `--kolibri-spacing` | `0.25rem` (4px) |
| `--font-family` | `--kolibri-font-family` | `'Noto Sans Regular', Arial, helvetica, sans-serif` |

### Colors

| Token | Kolibri override | Default |
|-------|-----------------|---------|
| `--color-primary` | `--kolibri-color-primary` | `#156570` |
| `--color-primary-variant` | `--kolibri-color-primary-variant` | `#207a8b` |
| `--color-secondary` | `--kolibri-color-secondary` | `#ccebf7` |
| `--color-danger` | `--kolibri-color-danger` | `#ca0101` |
| `--color-warning` | `--kolibri-color-warning` | `#c44931` |
| `--color-success` | `--kolibri-color-success` | `#005c45` |
| `--color-subtle` | `--kolibri-color-subtle` | `#576164` |
| `--color-light` | `--kolibri-color-light` | `#ffffff` |
| `--color-text` | `--kolibri-color-text` | `#202020` |
| `--color-mute` | `--kolibri-color-mute` | `#f2f3f4` |
| `--color-mute-variant` | `--kolibri-color-mute-variant` | `#bec5c9` |
| `--color-visited` | `--kolibri-color-visited` | `#551a8b` |

To override a token globally, set the `--kolibri-*` property on the host element or `:root`:

```css
:root {
	--kolibri-color-primary: #005a9c;
	--kolibri-font-size: 18px;
}
```

## Theming notes

When using adaptive styles, global `CSS` custom properties can collide with those of the application. For internal calculations, prefer using `SASS` variables and only expose well-prefixed `CSS` properties externally.

## Development

### HTML usage

After installation, you can use KoliBri components with the theme directly in HTML:

```html
<!doctype html>
<html lang="en">
	<head>
		<meta charset="UTF-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1.0" />
		<title>Theme Demo</title>
	</head>
	<body>
		<kol-button _label="Button"></kol-button>
		<kol-input-text _label="Name" _placeholder="Your Name"></kol-input-text>
		<kol-card _headline="Card">Card content with styling</kol-card>

		<script type="module">
			import { register } from '@public-ui/components';
			import { BWSt } from '@public-ui/theme-bwst';
			import { defineCustomElements } from '@public-ui/components/loader';

			register(BWSt, defineCustomElements)
				.then(() => {
					console.log('Theme loaded successfully');
				})
				.catch(console.warn);
		</script>
	</body>
</html>
```

## Support

If you run into issues during development or the build process, take a look at [CONTRIBUTING.md](./CONTRIBUTING.md) for detailed troubleshooting guidance.

## License

This project is licensed under the [European Union Public Licence (EUPL) v1.2](https://joinup.ec.europa.eu/collection/eupl/eupl-text-eupl-12). The EUPL is an open-source license developed by the European Commission and is compatible with other well-known open-source licenses.

## Related Projects

- [KoliBri – The accessible HTML standard](https://public-ui.github.io/)
