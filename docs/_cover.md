# Fable Settings

> A tolerant configuration chain

Load settings from defaults, constructor parameters, JSON files, and environment variables -- merged together in a predictable order with deep object merging and graceful fallbacks.

- **Layered Merging** -- Defaults, constructor params, default config file, config file -- each layer overrides the last
- **Environment Variables** -- Reference `${VAR|default}` in any string value, resolved automatically
- **Deep Merge** -- Nested objects are merged property-by-property, not replaced wholesale
- **Tolerant Loading** -- Missing config files produce a warning, not a crash

[Get Started](README.md)
[API Reference](api.md)
[GitHub](https://github.com/fable-retold/fable-settings)
