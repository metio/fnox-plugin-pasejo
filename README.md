<!--
SPDX-FileCopyrightText: The pasejo Authors
SPDX-License-Identifier: 0BSD
-->

# fnox-plugin-pasejo

A [fnox](https://fnox.jdx.dev/) provider that reads secrets from [pasejo](https://github.com/metio/pasejo) stores. fnox sets environment variables for a command from the secrets a `fnox.toml` declares, and this plugin resolves them through `pasejo secret resolve`, so a pasejo release that has that command is required.

## Installation

Declare the plugin in `fnox.toml`, and fnox installs it the first time a provider uses it. Pin a commit so every machine runs the same plugin:

```toml
[plugins]
pasejo = "metio/fnox-plugin-pasejo#<commit>"
```

`fnox plugin install pasejo metio/fnox-plugin-pasejo` installs it by hand, and `fnox plugin link pasejo /path/to/checkout` uses a local checkout.

## Configuration

```toml
[providers.pasejo]
type = "lua"
plugin = "pasejo"
options = { store = "work" }

[secrets]
GITHUB_TOKEN = { provider = "pasejo", value = "api/github.com/token", if_missing = "error" }
DB_USER = { provider = "pasejo", value = "db/prod#username", if_missing = "error" }
```

```sh
fnox exec -- ./deploy.sh
```

A `value` is the path of a secret, which selects its first line, the password by pasejo's convention. A path followed by `#FIELD` selects the value of that `FIELD: value` line instead; the path runs up to the last `#`. The `options` are all optional:

- `store`: the pasejo store to read. Without it, pasejo's default store applies. Declare one provider per store to read from several.
- `bin`: the pasejo executable. Without it, fnox runs `pasejo` from `PATH`.
- `offline`: `"true"` skips the store's pull hooks.

pasejo runs with fnox's environment, so `PASEJO_CONFIG` and `PASEJO_DEFAULT_STORE` apply as usual.

## Behavior

fnox hands the plugin every reference of a provider at once, and the plugin passes them to a single `pasejo secret resolve`. The store is therefore decrypted once per `fnox exec`, which matters for hardware keys that ask for a touch per decryption.

pasejo resolves all references or none. When one secret or field is missing, every secret of that provider fails, and the error names the reference as written in `fnox.toml`. fnox's own default for a failed secret is a warning, after which it runs the command without the variable. Set `if_missing = "error"` on each secret, or pass `--if-missing error`, so the command does not start with values missing.

## Development

The toolchain comes from breto: `eval "$(breto env)"` puts fnox, pasejo, age and the lints on `PATH`.

| Command | What it does |
| --- | --- |
| `breto run ci-lint` | Every lint, including StyLua and selene for the Lua code. |
| `breto run e2e` | Creates a throwaway pasejo store and fnox project, links this checkout as the plugin, and checks what fnox reads through it. `PASEJO_BIN` selects another pasejo executable. |

## License

[0BSD](LICENSES/0BSD.txt).
