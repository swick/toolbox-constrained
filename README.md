# toolbox-constrained

Run a [Toolbx](https://containertoolbx.org/) image in an isolated container. Unlike `toolbox enter`, this does **not** bind-mount your home directory or integrate with the host. You explicitly choose what the container can access.

## Usage

```
toolbox-constrained <toolbox-name> [options] [-- command]
```

The image is automatically resolved from the named toolbox container.

### Examples

```bash
# Isolated shell, no host access
toolbox-constrained my-toolbox

# Mount a project directory
toolbox-constrained my-toolbox -v ~/Projects/myapp:/work

# Use capabilities from config
toolbox-constrained my-toolbox --cap ssh --cap git

# Read-only container filesystem
toolbox-constrained my-toolbox --read-only

# Combine everything
toolbox-constrained my-toolbox --cap ssh -v ~/Projects/myapp:/work -e MY_VAR=hello -- bash
```

### Options

| Flag | Description |
|------|-------------|
| `-v HOST:CONTAINER[:opts]` | Bind mount (repeatable). `~` and `$ENV_VARS` are expanded. SELinux label `:Z` is added automatically. |
| `-e KEY[=VALUE]` | Set or pass through an environment variable (repeatable). |
| `--cap NAME` | Enable a capability from the config file (repeatable). |
| `--read-only` | Make the container filesystem read-only (tmpfs on `/tmp` and `/run`). |
| `--dry-run` | Print the podman command instead of running it. |

## Capabilities

Define reusable groups of volumes, environment variables, and PATH entries in `~/.config/toolbox-constrained/config.toml`:

```toml
[capabilities.ssh]
volumes = ["~/.ssh:~/.ssh:ro"]

[capabilities.git]
volumes = ["~/.gitconfig:~/.gitconfig:ro"]

[capabilities.wayland]
env = ["WAYLAND_DISPLAY", "XDG_RUNTIME_DIR"]
volumes = ["$XDG_RUNTIME_DIR/$WAYLAND_DISPLAY:$XDG_RUNTIME_DIR/$WAYLAND_DISPLAY:ro"]

[capabilities.rust]
path = ["~/.cargo/bin"]
```

The `path` entries are prepended to the container's `$PATH`. `~` and `$ENV_VARS` are expanded.

See [config.toml.example](config.toml.example) for a starting point.

## How it works

- Resolves the container image from the toolbox name via `podman inspect`
- Runs `podman run` with `--userns=keep-id` so you're your own user, not root
- Sets up passwordless `sudo` inside the container
- All bind mounts get SELinux `:Z` labels automatically unless already specified
