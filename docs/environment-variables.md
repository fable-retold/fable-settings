# Environment Variables

Fable Settings can resolve environment variables inside string values using a template syntax. This lets you keep secrets and environment-specific values out of your configuration files.

## Syntax

```
${VARIABLE_NAME|defaultValue}
${VARIABLE_NAME}
```

The pattern `${...}` is recognized in any string value anywhere in the settings object. The pipe `|` separates the variable name from an optional default.

## Resolution Rules

| Pattern | Env Var Set? | Result |
|---------|-------------|--------|
| `${PORT\|3000}` | `PORT=8080` | `8080` |
| `${PORT\|3000}` | not set | `3000` |
| `${PORT}` | `PORT=8080` | `8080` |
| `${PORT}` | not set | `""` (empty string) |

## Examples

### Database configuration

```javascript
const libFableSettings = require('fable-settings');

let tmpSettings = new libFableSettings({
    MySQL: {
        Server: '${DB_HOST|localhost}',
        Port: '${DB_PORT|3306}',
        User: '${DB_USER|root}',
        Password: '${DB_PASSWORD}',
        Database: '${DB_NAME|myapp}'
    }
});
console.log('MySQL config (with env-var defaults):', tmpSettings.settings.MySQL);
```

In development with no environment variables set, this resolves to the defaults. In production, set the environment variables and the same config file works.

### Multiple variables in one string

```javascript
const libFableSettings = require('fable-settings');

let tmpSettings = new libFableSettings({
    ConnectionString: '${DB_USER|root}:${DB_PASSWORD}@${DB_HOST|localhost}:${DB_PORT|3306}'
});
console.log('ConnectionString:', tmpSettings.settings.ConnectionString);
```

Each `${...}` pattern is resolved independently.

### Array values

Environment variables are resolved inside arrays too:

```javascript
const libFableSettings = require('fable-settings');

let tmpSettings = new libFableSettings({
    Hosts: ['${PRIMARY_HOST|host1}', '${SECONDARY_HOST|host2}']
});
console.log('Hosts:', tmpSettings.settings.Hosts);
```

## When Resolution Happens

Environment variables are resolved:

1. **During construction** -- After all config files are loaded and merged
2. **During `merge()`** -- Every call to `merge()` re-resolves environment variables in the merged result
3. **Not during `fill()`** -- The `fill()` method does not trigger resolution

Resolution is recursive -- nested objects are traversed and all string values are processed.

## Disabling Resolution

Set `NoEnvReplacement` to `true` to prevent environment variable resolution entirely:

```javascript
const libFableSettings = require('fable-settings');

let tmpSettings = new libFableSettings({
    NoEnvReplacement: true,
    TemplateValue: '${THIS_STAYS_LITERAL}'
});

console.log(tmpSettings.settings.TemplateValue);
// '${THIS_STAYS_LITERAL}' (not resolved)
```

This setting can also come from a config file. If a `ConfigFile` sets `NoEnvReplacement: true`, subsequent merges will not resolve environment variables:

```javascript
const libFableSettings = require('fable-settings');

// Reference shape — `__dirname` is Node-only; we pass a literal path
// that doesn't exist so the snippet runs in the playground (missing
// config files are logged as warnings, never thrown).
let tmpSettings = new libFableSettings({
    ConfigFile: '/nonexistent/config.json',
    Value: '${SOME_VAR|default}'
});

console.log('Value:', tmpSettings.settings.Value);
// Value may or may not be resolved depending on when
// NoEnvReplacement was encountered in the merge chain
```

The `_configureEnvTemplating` check runs after every `merge()`, so the flag can change mid-lifecycle.

## How It Works

Under the hood, Fable Settings uses a template processor built on the `precedent` library. The processor:

1. Scans each string value for `${...}` patterns
2. Extracts the variable name and optional default (split on `|`)
3. Checks `process.env` for the variable
4. Returns the environment value if found, or the default, or an empty string
