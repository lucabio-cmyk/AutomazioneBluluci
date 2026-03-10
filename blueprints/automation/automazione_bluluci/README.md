# Troubleshooting YAML parse errors

If you see an error like:

```text
mapping values are not allowed here
line 35, column 28: --tab-size-preference: 4;
```

it means YAML is parsing a CSS declaration as YAML mapping syntax.

## How to fix it

When using CSS variables in YAML, put them inside a string block (or quote the full line), for example:

```yaml
card_mod:
  style: |
    :host {
      --tab-size-preference: 4;
    }
```

Do **not** place raw CSS declarations at YAML key level.
