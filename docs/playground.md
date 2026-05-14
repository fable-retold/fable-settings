# Code Playground

fable-settings's docs are wired to the **Fable Playground** — a live
editor + sandbox that lives in a sliding drawer at the bottom of the
viewport.  Every JavaScript example in these docs has a small **▶** play
button next to its Copy and Fullscreen actions; clicking it loads the
snippet into the playground, where you can edit it and press **Run** to
see the output captured in the panel beside the editor.

For the full reference on what the playground does, how the `require`
shim works, and the caveats around module sandboxing, see the
[Fable Playground reference page](/#/playground/fable).

## Try it

The example below boots a Fable with a nested settings tree, then walks
through the typical patterns: read, deep-merge a partial tree, and
gap-fill defaults without overwriting existing values.

```javascript
const Fable = require('fable');

// Pass whatever shape you want for settings.  Fable hands the object
// off to fable-settings, which deep-merges it on top of its built-in
// defaults and exposes the result as app.settings / app.SettingsManager.settings.
const app = new Fable({
    Product:        'SettingsPlaygroundDemo',
    ProductVersion: '1.0.0',
    Database:
    {
        Host: 'db.example.com',
        Port: 5432,
        Credentials: { User: 'app', Password: '<secret>' }
    },
    Features: { ExperimentalCache: false }
});

// Read a top-level setting via the convenience getter.
app.log.info('Product =', { Value: app.settings.Product });

// Read a nested setting.
app.log.info('DB host:port', {
    Host: app.settings.Database.Host,
    Port: app.settings.Database.Port
});

// `merge(from, to)` does a deep merge: keys in `from` win, nested
// objects are merged property-wise, and `to` is mutated and returned.
// Pass app.settings as the target to update the live config.
app.SettingsManager.merge(
    {
        Database: { Port: 5433 },          // overwrite one nested field
        Features: { NewLogger: true }      // add a brand-new key
    },
    app.settings);

app.log.info('After merge', {
    Port:              app.settings.Database.Port,
    Host:              app.settings.Database.Host,
    ExperimentalCache: app.settings.Features.ExperimentalCache,
    NewLogger:         app.settings.Features.NewLogger
});

// `fill(from)` is the gap-only sibling — values that already exist on
// app.settings are LEFT ALONE; only missing keys are filled in.
// Great for layering library defaults under user-provided overrides.
app.SettingsManager.fill(
    {
        Database: { Port: 9999, SSL: true },   // Port already set → ignored; SSL → filled
        Telemetry: { Enabled: false }          // brand new tree → filled
    });

app.log.info('After fill', {
    Port:             app.settings.Database.Port,   // still 5433 — not overwritten
    SSL:              app.settings.Database.SSL,    // true — newly filled
    TelemetryEnabled: app.settings.Telemetry.Enabled
});
```
