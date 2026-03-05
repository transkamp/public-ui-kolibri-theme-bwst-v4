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
