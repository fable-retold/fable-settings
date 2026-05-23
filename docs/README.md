# Fable Settings

A tolerant configuration chain for Node.js applications. Load settings from defaults, constructor parameters, JSON files, and environment variables -- merged together in a predictable order.

## Why

It became tiring to see a heap of boilerplate config file and defaults loading at the top of microservices. Fable Settings removes that boilerplate: pass in an object, optionally point at a JSON file, and get a merged configuration with environment variable support.

## Quick Start

```bash
npm install fable-settings
```

```javascript
const libFableSettings = require('fable-settings');

let tmpSettings = new libFableSettings({
    Product: 'MyApp',
    ProductVersion: '1.0.0',
    APIServerPort: 8080
});

console.log(tmpSettings.settings.Product);         // 'MyApp'
console.log(tmpSettings.settings.ProductVersion);   // '1.0.0'
console.log(tmpSettings.settings.APIServerPort);    // 8080
```

## How Settings Merge

Settings are built in layers. Each layer merges on top of the previous one:

```
1. Built-in defaults (Product, ProductVersion, ConfigFile, LogStreams)
       ↓ merge
2. Constructor parameters (what you pass to new FableSettings({...}))
       ↓ merge
3. DefaultConfigFile (JSON file, loaded if specified)
       ↓ merge
4. ConfigFile (JSON file, loaded if specified)
       ↓
5. Environment variable resolution (${VAR|default} patterns)
       ↓
   Final this.settings object
```

Later layers override earlier ones. If a file is missing or unreadable, the chain continues gracefully with a console warning.

## Loading from Files

Point at JSON config files and they are loaded and merged automatically:

```javascript
const libFableSettings = require('fable-settings');

// Reference shape — in real Node code you'd use __dirname.  We pass
// a literal here so the snippet runs in the playground; missing files
// are silently warned about, never thrown.
let tmpSettings = new libFableSettings({
    Product: 'MyApp',
    DefaultConfigFile: '/nonexistent/default-config.json',
    ConfigFile: '/nonexistent/config.json'
});
console.log('Product:', tmpSettings.settings.Product);
```

`DefaultConfigFile` loads first, then `ConfigFile` overrides it. Both are optional -- if either is missing, a warning is logged and execution continues.

## Environment Variables

String values in settings can reference environment variables with fallback defaults:

```javascript
const libFableSettings = require('fable-settings');

let tmpSettings = new libFableSettings({
    DatabaseHost: '${DB_HOST|localhost}',
    DatabasePort: '${DB_PORT|5432}',
    SecretKey: '${SECRET_KEY}'
});
console.log('Host:',   tmpSettings.settings.DatabaseHost);
console.log('Port:',   tmpSettings.settings.DatabasePort);
console.log('Secret:', JSON.stringify(tmpSettings.settings.SecretKey));
```

If `DB_HOST` is set in the environment, its value is used. Otherwise the default after the `|` is used. If there is no default and the variable is not set, the value becomes an empty string.

See [Environment Variables](environment-variables.md) for details.

## Merging at Runtime

After construction, you can merge additional settings in:

```javascript
const libFableSettings = require('fable-settings');
let tmpSettings = new libFableSettings({ Product: 'OriginalName' });

// merge() overwrites existing keys
tmpSettings.merge({ NewSetting: 'value', Product: 'NewName' });
console.log(tmpSettings.settings.Product);     // 'NewName'
console.log(tmpSettings.settings.NewSetting);  // 'value'

// fill() adds keys without overwriting existing ones
tmpSettings.fill({ Product: 'WontOverwrite', AnotherSetting: 42 });
console.log(tmpSettings.settings.Product);        // 'NewName' (unchanged)
console.log(tmpSettings.settings.AnotherSetting); // 42 (added)
```

Both methods handle nested objects with deep merging.

## Default Settings

Every instance starts with these built-in defaults:

```json
{
    "Product": "ApplicationNameHere",
    "ProductVersion": "0.0.0",
    "ConfigFile": false,
    "LogStreams": [
        {
            "level": "trace"
        }
    ]
}
```

Access the defaults at any time via `tmpSettings.default`.

## With Fable

When used inside the Fable ecosystem, settings are created automatically:

```javascript
const libFable = require('fable');

let _Fable = new libFable({
    Product: 'MyApp',
    ProductVersion: '2.0.0',
    MySQL: {
        Server: '${DB_HOST|localhost}',
        Port: 3306,
        Database: 'mydb'
    }
});

// Access via the Fable instance
console.log(_Fable.settings.Product);       // 'MyApp'
console.log(_Fable.settings.MySQL.Server);  // value of DB_HOST or 'localhost'
```

## Learn More

- [Environment Variables](environment-variables.md) -- Template syntax and disabling resolution
- [API Reference](api.md) -- Complete method and property documentation
- [Fable](https://stevenvelozo.github.io/fable/) -- The core ecosystem that uses fable-settings
