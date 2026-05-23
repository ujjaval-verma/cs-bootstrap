# cs-bootstrap

Workstation bootstrap for macOS: the first-run layer before `nix-darwin` /
`home-manager` take over.

## Commands

```sh
./bootstrap.sh        # run the bootstrap (idempotent, safe to rerun)
./scripts/check.sh    # syntax check, shellcheck, and `nix flake check`
direnv allow          # enter the dev shell (provides shellcheck, nixpkgs-fmt)
```

`./scripts/check.sh` is the verification gate — run it before claiming work is
done. It will skip the nix portions if `nix` is not on `PATH`.

## Layout

- `bootstrap.sh` — the only first-run script. Single file, POSIX-ish bash.
- `nix/darwin/` — shared `darwinModules.default` consumed by user flakes.
- `templates/nix-darwin/` — `nix flake init` template for end users.
- `scripts/check.sh` — local CI.
- `flake.nix` — dev shell + `darwinConfigurations.bootstrap-{aarch64,x86_64}`
  smoke configs used only by `check.sh`.

## Hard Constraints

These are easy to violate and have caused real friction — respect them:

- **`bootstrap.sh` must stay idempotent and non-destructive.** Never overwrite
  an existing `/etc/nix-darwin/flake.nix` or `configuration.nix`. Never append
  to user-managed shell rc files (`.zshrc`, `.bash_profile`, etc.) — persistent
  shell config belongs to the user's home-manager.
- **Layer discipline.** `bootstrap.sh` only handles prerequisites Nix cannot
  bootstrap itself (Xcode CLT, Homebrew, Nix, initial `darwin-rebuild`).
  System-level config goes in `nix/darwin/`. User-level dotfiles do **not**
  belong in this repo at all.
- **Xcode CLT installs via `softwareupdate`**, not the GUI installer, unless
  `BOOTSTRAP_XCODE_CLT_ALLOW_GUI=1` is set.
- **Tailscale is intentionally excluded** from the cask list — its installer
  manages VPN/system-extension state and is not idempotent. Don't add it.
- The script is invoked via `curl | bash` on fresh machines; assume **no
  developer tools** beyond a default macOS install when editing it.

## Environment Overrides

User-facing knobs (documented in README, repeated here for quick reference):

- `BOOTSTRAP_DARWIN_FLAKE` — point at the user's own nix-darwin flake
- `BOOTSTRAP_SKIP_DARWIN=1` — skip the `darwin-rebuild` step
- `BOOTSTRAP_INSTALL_ROSETTA=1` — install Rosetta 2 on Apple Silicon
- `BOOTSTRAP_RUN_GH_AUTH=1` — kick off `gh auth login` at the end
- `BOOTSTRAP_XCODE_CLT_ALLOW_GUI=1` — allow the GUI CLT installer fallback
- `BOOTSTRAP_NIXPKGS_INPUT`, `BOOTSTRAP_NIX_DARWIN_INPUT` — override flake inputs

## Fork Context

This repo is a fork of `clamshell-ai/bootstrap`. `upstream` points at the
clamshell remote; `origin` and the `gh` default both target
`ujjaval-verma/cs-bootstrap`. Open PRs against this fork unless explicitly
contributing back upstream.
