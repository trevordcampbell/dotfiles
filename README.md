# macOS shell and mise setup

Trevor's current Apple Silicon macOS host setup. Development projects continue to use their devcontainers.

The repository contains the current Zsh and Bash startup files, Powerlevel10k prompt configuration, the global mise configuration, and the daily maintenance script. It declares host CLI tools, desktop applications, complete font packages, Oh My Zsh, and custom plugin/theme repositories.

## Restore on another Mac

Install mise using its official instructions at https://mise.jdx.dev/installing-mise.html and ensure Git is available. Then clone this repository at the configured location:

```sh
git clone https://github.com/trevordcampbell/dotfiles.git "$HOME/.local/share/dotfiles"
"$HOME/.local/bin/mise" -C "$HOME/.local/share/dotfiles" trust
"$HOME/.local/bin/mise" -C "$HOME/.local/share/dotfiles" dotfiles apply "$HOME/.config/mise/config.toml" --force --yes
"$HOME/.local/bin/mise" -C "$HOME" bootstrap --yes
"$HOME/.local/bin/mise" -C "$HOME" run shell:completions
exec zsh -l
```

If mise is installed elsewhere, use that executable in place of `~/.local/bin/mise`. The maintenance LaunchAgent expects it at `~/.local/bin/mise`; install or link it there before applying bootstrap. `--force` replaces an existing global mise configuration; review and merge that configuration before restoring onto an already configured machine. Existing shell files may also require `mise dotfiles apply --force --yes` after review.

The root `mise.toml` only links the global config. The complete machine setup is `mise/config.toml`. Normal daily commands run from the home directory and use that global config. Native formulae and app casks are installed directly by mise in `/opt/homebrew`; no Homebrew executable is needed. Installing native packages on a fresh machine may require administrator authentication to prepare that prefix.

## Updates and local edits

The existing daily maintenance LaunchAgent runs at 03:00 and at load. It updates mise, global tools, rustup, host packages, idle desktop apps, and clean shell repositories. Running apps are deferred. Mise, tool, and MarkText releases observe the configured 24-hour minimum age. Rustup uses its official stable self-updater. Oh My Zsh's separate updater is disabled so mise schedules its updates. Repository updates only fast-forward and skip dirty checkouts.

```sh
mise run maintenance:check
mise run maintenance:update
mise run maintenance:status
mise run shell:status
mise run shell:update
mise run fonts:check
mise run fonts:update
```

Shell files and the global config are symlinks to this checkout. Editing `~/.zshrc`, `~/.p10k.zsh`, or `~/.config/mise/config.toml` edits the corresponding repository file. The `dotfiles` shell alias runs Git in this checkout:

```sh
dotfiles status
dotfiles diff
dotfiles add .zshrc .p10k.zsh mise/config.toml
dotfiles commit -m "Update shell settings"
dotfiles push
```

Edits are not automatically committed or published. Review and push changes explicitly. The older bare Git repository at `~/.dotfiles` is separate; use the alias above for this managed checkout.

## Public repository and machine-local settings

Credentials, shell histories, recordings, app data, caches, and generated completion files are not stored here. Put private shell settings in `~/.config/zsh/local.zsh`, which the Zsh startup file sources if present. Never add tokens or passwords to the public tracked files.

Poppins installs all 18 static artifacts supplied by its cask. Open Sans, Rubik, and Space Grotesk use the `fonts:sync` task to fetch all variable TTF styles, including italics where the family provides them, directly from the official `google/fonts` repository; mise deploys them as managed copies. The task checks family revisions older than 24 hours, verifies each file against its immutable Git blob, and retains the OFL licenses locally. Daily maintenance includes this task; unchanged font files are not downloaded again. This avoids full Google Fonts clones and provides updates for the two families whose casks use an unversioned `latest` marker; it also includes Space Grotesk’s full variable weight range. The previous standalone copies of these four families are removed to avoid duplicate registrations. Other installed fonts and App Store apps are outside this configuration. Zoom and RODE Central remain vendor-managed.

## Maintenance checks

```sh
mise exec -- python "$HOME/.config/mise/tests/test-maintenance.py"
mise exec -- python "$HOME/.local/share/dotfiles/mise/tests/test-fonts.py"
```

These checks cover idle-app deferral, official artifact verification, preview behavior, rollback, writing through repository-backed config symlinks, and staged font updates.
