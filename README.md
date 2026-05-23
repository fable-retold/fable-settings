# Fable Settings

> **[&#9654; Read the Fable-Settings Documentation](https://stevenvelozo.github.io/fable-settings/)** &mdash; interactive docs with the full API reference.

A simple, tolerant configuration chain for Fable applications. Merges default settings, constructor overrides, JSON config files, and environment variables into a single settings object -- with deep merge, fill-without-overwrite, and `${ENV_VAR|default}` template syntax.

[![Build Status](https://github.com/stevenvelozo/fable-settings/workflows/Fable-Settings/badge.svg)](https://github.com/stevenvelozo/fable-settings/actions)
[![npm version](https://badge.fury.io/js/fable-settings.svg)](https://badge.fury.io/js/fable-settings)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## Features

- **Layered Configuration** - Merges defaults, constructor settings, default config file, and config file in order
- **Deep Merge** - Nested objects are merged recursively rather than replaced wholesale
- **Environment Variable Templating** - Use `${ENV_VAR}` or `${ENV_VAR|fallback}` syntax in any string setting value
- **Config File Loading** - Optionally load and merge settings from `DefaultConfigFile` and `ConfigFile` JSON paths
- **Fill Without Overwrite** - `fill()` adds missing keys from a source object without replacing existing values
- **Core Service** - Extends `fable-serviceproviderbase` CoreServiceProviderBase, available before Fable is fully initialized
- **Browser Support** - Ships with a browser shim that assigns `window.FableSettings` automatically

## Installation

```bash
npm install fable-settings
```

## Quick Start

```javascript
const libFableSettings = require('fable-settings');

let settings = new libFableSettings(
{
	Product: 'MyApp',
	ProductVersion: '2.0.0',
	Database:
	{
		Host: '${DB_HOST|localhost}',
		Port: 5432
	}
});

console.log(settings.settings.Product);         // 'MyApp'
console.log(settings.settings.Database.Host);    // value of $DB_HOST, or 'localhost'
```

### Legacy Factory

A `.new()` factory is available for backwards compatibility:

```javascript
let settings = require('fable-settings').new(
{
	Product: 'MyApp',
	ProductVersion: '1.0.0'
});
```

## Configuration Chain

Settings are resolved in the following order (later sources overwrite earlier ones):

1. **Built-in defaults** -- `Product`, `ProductVersion`, `ConfigFile`, `LogStreams`
2. **Constructor settings** -- the object passed to `new FableSettings(pSettings)`
3. **Default config file** -- if `DefaultConfigFile` is set, loaded and merged
4. **Config file** -- if `ConfigFile` is set, loaded and merged

Config file loading is fault-tolerant -- if a file cannot be loaded, a warning is logged and the previous settings are preserved.

### Default Settings

```json
{
	"Product": "ApplicationNameHere",
	"ProductVersion": "0.0.0",
	"ConfigFile": false,
	"LogStreams": [{ "level": "trace" }]
}
```

## Usage

### Merging Settings at Runtime

Use `merge()` to deep-merge new values into the current settings:

```javascript
let settings = new libFableSettings({ Product: 'MyApp' });

settings.merge({ Database: { Host: 'db.example.com', Port: 5432 } });
settings.merge({ Database: { Port: 3306 } });

// settings.settings.Database.Host === 'db.example.com'
// settings.settings.Database.Port === 3306
```

### Filling Gaps Without Overwriting

Use `fill()` to add missing keys without replacing existing values:

```javascript
let settings = new libFableSettings({ Product: 'MyApp', LogLevel: 'info' });

settings.fill({ Product: 'DefaultApp', CacheTTL: 3600, LogLevel: 'trace' });

// settings.settings.Product === 'MyApp'       (not overwritten)
// settings.settings.LogLevel === 'info'        (not overwritten)
// settings.settings.CacheTTL === 3600          (filled in)
```

### Environment Variable Templating

Any string value can reference environment variables with the `${VAR}` or `${VAR|default}` syntax. Variables are resolved recursively through nested objects:

```javascript
// Given: DB_HOST=prod-db.example.com in the environment

let settings = new libFableSettings(
{
	Database:
	{
		Host: '${DB_HOST|localhost}',
		Port: '${DB_PORT|5432}',
		Name: '${DB_NAME|myapp}'
	}
});

// settings.settings.Database.Host === 'prod-db.example.com'  (from env)
// settings.settings.Database.Port === '5432'                  (default)
// settings.settings.Database.Name === 'myapp'                 (default)
```

Disable environment variable templating by setting `NoEnvReplacement: true`:

```javascript
let settings = new libFableSettings(
{
	NoEnvReplacement: true,
	SomeValue: '${THIS_STAYS_LITERAL}'
});
```

### Loading Config Files

```javascript
let settings = new libFableSettings(
{
	Product: 'MyApp',
	DefaultConfigFile: '/etc/myapp/defaults.json',
	ConfigFile: '/etc/myapp/config.json'
});
```

## API

### Constructor

```javascript
new FableSettings(pSettings, pServiceHash)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `pSettings` | `Object` | Initial settings to merge over defaults |
| `pServiceHash` | `String` | Optional service hash identifier |

### Methods

| Method | Description |
|--------|-------------|
| `merge(pSettingsFrom, pSettingsTo)` | Deep-merge `pSettingsFrom` into `pSettingsTo` (defaults to `this.settings`). Resolves environment variable templates after merging. Returns the merged object. |
| `fill(pSettingsFrom)` | Deep-merge `this.settings` into a copy of `pSettingsFrom`, filling gaps without overwriting existing values. Returns `this.settings`. |
| `buildDefaultSettings()` | Returns a fresh copy of the built-in default settings object. |

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `settings` | `Object` | The resolved settings object |
| `default` | `Object` | The built-in default settings |
| `base` | `Object` | Snapshot of settings after constructor merge, before config file loading |
| `settingsTemplateProcessor` | `Object` | The Precedent-based template processor for `${ENV}` resolution |

## Part of the Retold Framework

Fable Settings is a core service in the Fable ecosystem:

- [fable](https://github.com/stevenvelozo/fable) - Application services framework
- [fable-serviceproviderbase](https://github.com/stevenvelozo/fable-serviceproviderbase) - Service provider base class
- [precedent](https://github.com/stevenvelozo/precedent) - Template expression parser (powers environment variable substitution)
- [fable-log](https://github.com/stevenvelozo/fable-log) - Logging framework (uses `LogStreams` from settings)

## Testing

Run the test suite:

```bash
npm test
```

Run with coverage:

```bash
npm run coverage
```

## Related Packages

- [fable](https://github.com/stevenvelozo/fable) - Application services framework
- [fable-serviceproviderbase](https://github.com/stevenvelozo/fable-serviceproviderbase) - Service provider base class
- [precedent](https://github.com/stevenvelozo/precedent) - String template engine

## License

MIT

## Contributing

Pull requests are welcome. For details on our code of conduct, contribution process, and testing requirements, see the [Retold Contributing Guide](https://github.com/stevenvelozo/retold/blob/main/docs/contributing.md).
