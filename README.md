# Claude Wrapper

This directory contains a wrapper for the `claude` CLI that shadows the original
binary earlier on `PATH`. The script inspects CLI flags, configures Anthropic or
OpenRouter credentials, and then delegates to the underlying executable so the
real `claude` tool still handles the heavy lifting.

## Features

- Supports multiple profiles stored under `~/.config/claude-wrapper/profiles`
- Ad-hoc profiles via `provider:model` syntax without touching config files
- Convenience `config` subcommand for listing, switching, and editing profiles
- Automatic migration from the legacy `~/.config/claude-wrapper.toml` format
- Provider-specific environment setup (currently Anthropic and OpenRouter)

## Usage

```sh
# Use the default profile (configured in `config.toml`)
claude chat

# Select a named profile
claude --profile work chat

# Override provider/model on the fly
claude --provider openrouter --model whatever:latest chat

# Manage profiles
claude config list
claude config use openrouter
claude config edit default
```

Profile and provider can also be specified via the `CLAUDE_PROFILE`,
`CLAUDE_PROVIDER`, and `CLAUDE_MODEL` environment variables.

### Extra CLI Arguments

Both the global config (`~/.config/claude-wrapper/config.toml`) and each profile
may append additional flags to every command. Set them via:

```sh
# Apply to every invocation
claude config global set extra_args "--allow-dangerously-skip-permissions"

# Profile-only arguments (e.g., a YOLO profile)
claude config profile create yolo
claude config profile use yolo
claude config set extra_args "--dangerously-skip-permissions"
```

Global and profile values are additive; if you run the commands above the wrapper
always includes `--allow-dangerously-skip-permissions`, and only the `yolo`
profile adds the stronger `--dangerously-skip-permissions`. Sample config files
live under `config-example/` to help you bootstrap new setups.

### Custom OpenRouter Keys

The wrapper can use OpenRouter API keys that differ from the global
`OPENROUTER_API_KEY` export. Keys are resolved in the following order:

1. `CLAUDE_OPENROUTER_API_KEY` (per-invocation override)
2. `openrouter_api_key = "..."` defined directly in the active profile
3. The file referenced by `openrouter_api_key_file` or the default
   `~/.config/claude-wrapper/secrets/openrouter/<profile>.key`
4. The legacy `OPENROUTER_API_KEY` variable

Use `claude config set openrouter_key <value>` to store a secret for the current
profile inside the secrets directory above. `--stdin` reads the key from STDIN,
and `--file <path>` records a pointer to an existing secret file without copying
it.

## Configuration Layout

```
~/.config/claude-wrapper/
├── config.toml          # Global defaults (currently default profile)
└── profiles/
    ├── default.toml
    └── <profile>.toml
```

Each profile file stores simple TOML assignments such as `provider = "anthropic"`
and `model = "claude-3-5-sonnet"`. The wrapper ensures these directories exist
and will migrate your legacy config file into this structure on first run.

## Adding to PATH

Place this directory on your `PATH` ahead of the real `claude` binary so the
wrapper is invoked transparently. The dotfiles update in this repo does exactly
that by prepending `~/code/scripts/claude-wrapper` before the Anthropic install
location.
