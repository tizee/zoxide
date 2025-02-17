# `zoxide`

[![crates.io][crates.io-badge]][crates.io]

> A faster way to navigate your filesystem

`zoxide` is a blazing fast replacement for your `cd` command, inspired by
[`z`][z] and [`z.lua`][z.lua]. It keeps track of the directories you use most
frequently, and uses a ranking algorithm to navigate to the best match.

![Demo][demo.gif]

## Examples

```sh
z foo       # cd into highest ranked directory matching foo
z foo bar   # cd into highest ranked directory matching foo and bar

z ~/foo     # z also works like a regular cd command
z foo/      # cd into relative path
z ..        # cd one level up
z -         # cd into previous directory

zi foo      # cd with interactive selection (using fzf)
```

Read more about the matching algorithm [here][algorithm-matching].

## Getting started

### Step 1: Install `zoxide`

`zoxide` works across all major platforms. If your distribution isn't included
in the list below, you can directly install the binary from GitHub:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh
```

If you would rather not run a script, you can download the binary from the
[Releases] page and add it to your `$PATH`.

#### On Linux

| Distribution   | Repository              | Instructions                                              |
| -------------- | ----------------------- | --------------------------------------------------------- |
| **Any**        | [crates.io]             | `cargo install zoxide`                                    |
| **Any**        | [Linuxbrew]             | `brew install zoxide`                                     |
| Alpine Linux   | [Alpine Linux Packages] | `apk add zoxide`                                          |
| Arch Linux     | [AUR]                   | `yay -Sy zoxide-bin`                                      |
| CentOS         | [Copr]                  | `dnf copr enable atim/zoxide` <br /> `dnf install zoxide` |
| Debian Testing | [Debian Packages]       | `apt install zoxide`                                      |
| Fedora         | [Fedora Packages]       | `dnf install zoxide`                                      |
| NixOS          | [nixpkgs]               | `nix-env -iA nixpkgs.zoxide`                              |
| Parrot OS      |                         | `apt install zoxide`                                      |
| Void Linux     | [Void Linux Packages]   | `xbps-install -S zoxide`                                  |

#### On macOS

| Repository  | Instructions           |
| ----------- | ---------------------- |
| [crates.io] | `cargo install zoxide` |
| [Homebrew]  | `brew install zoxide`  |
| [MacPorts]  | `port install zoxide`  |

#### On Windows

| Repository  | Instructions           |
| ----------- | ---------------------- |
| [crates.io] | `cargo install zoxide` |
| [Scoop]     | `scoop install zoxide` |

#### On BSD

| Distribution  | Repository   | Instructions           |
| ------------- | ------------ | ---------------------- |
| **Any**       | [crates.io]  | `cargo install zoxide` |
| DragonFly BSD | [DPorts]     | `pkg install zoxide`   |
| FreeBSD       | [FreshPorts] | `pkg install zoxide`   |
| NetBSD        | [pkgsrc]     | `pkgin install zoxide` |

#### On Android

| Repository | Instructions         |
| ---------- | -------------------- |
| [Termux]   | `pkg install zoxide` |

### Step 2: Install `fzf` (optional)

[`fzf`][fzf] is a command-line fuzzy finder, used by zoxide for interactive
selection. Installation instructions can be found [here][fzf-installation].

### Step 3: Add `zoxide` to your shell

If you currently use `z`, `z.lua`, or `zsh-z`, you may want to first import
your existing entries into `zoxide`:

```sh
zoxide import --from z path/to/db
```

Alternatively, for `autojump` (note that scores are not imported):

```sh
zoxide import --from autojump path/to/db
```

#### `bash`

Add the following line to your configuration file (usually `~/.bashrc`):

```sh
eval "$(zoxide init bash)"
```

#### `elvish`

Add the following line to your configuration file (usually `~/.elvish/rc.elv`):

```sh
eval $(zoxide init elvish | slurp)
```

#### `fish`

Add the following line to your configuration file (usually
`~/.config/fish/config.fish`):

```fish
zoxide init fish | source
```

#### `nushell`

Initialize zoxide's Nushell script:

```sh
zoxide init nushell --hook prompt | save ~/.zoxide.nu
```

Then, in your Nushell configuration file:

- Prepend `__zoxide_hook;` to the `prompt` variable.
- Add the following lines to the `startup` variable:
  - `zoxide init nushell --hook prompt | save ~/.zoxide.nu`
  - `source ~/.zoxide.nu`

#### `powershell`

Add the following line to your profile:

```powershell
Invoke-Expression (& {
    $hook = if ($PSVersionTable.PSVersion.Major -lt 6) { 'prompt' } else { 'pwd' }
    (zoxide init --hook $hook powershell) -join "`n"
})
```

#### `xonsh`

Add the following line to your configuration file (usually `~/.xonshrc`):

```python
execx($(zoxide init xonsh), 'exec', __xonsh__.ctx, filename='zoxide')
```

#### `zsh`

Add the following line to your configuration file (usually `~/.zshrc`):

```sh
eval "$(zoxide init zsh)"
```

#### Any POSIX shell

Add the following line to your configuration file:

```sh
eval "$(zoxide init posix --hook prompt)"
```

## Configuration

### `init` flags

- `--cmd`: changes the prefix of predefined aliases (`z`, `zi`).
  - e.g. `--cmd j` would change the aliases to `j` and `ji` respectively.
- `--hook <HOOK>`: change how often zoxide increments a directory's score:
  - `none`: never automatically add directories to zoxide.
  - `prompt`: add the current directory to zoxide at every shell prompt.
  - `pwd`: whenever the user changes directories, add the new directory to
    zoxide.
- `--no-aliases`: don't define extra aliases (`z`, `zi`).
  - These functions will still be available in your shell as `__zoxide_z` and
    `__zoxide_zi`, should you choose to use them elsewhere.

### Environment variables

Be sure to set these before calling `zoxide init`.

- `_ZO_DATA_DIR`
  - Specifies the directory in which zoxide should store its database.
  - The default value varies across OSes:
    | OS          | Path                                     | Example                                    |
    | ----------- | ---------------------------------------- | ------------------------------------------ |
    | Linux / BSD | `$XDG_DATA_HOME` or `$HOME/.local/share` | `/home/alice/.local/share`                 |
    | macOS       | `$HOME/Library/Application Support`      | `/Users/Alice/Library/Application Support` |
    | Windows     | `{FOLDERID_RoamingAppData}`              | `C:\Users\Alice\AppData\Roaming`           |
- `_ZO_ECHO`
  - When set to `1`, `z` will print the matched directory before navigating to
    it.
- `_ZO_EXCLUDE_DIRS`
  - Excludes the specified directories from the database.
  - This is provided as a list of [globs][glob], separated by OS-specific
    characters:
    | OS                  | Separator | Example                 |
    | ------------------- | --------- | ----------------------- |
    | Linux / macOS / BSD | `:`       | `$HOME:$HOME/private/*` |
    | Windows             | `;`       | `$HOME;$HOME/private/*` |
- `_ZO_FZF_OPTS`
  - Custom options to pass to [`fzf`][fzf]. See `man fzf` for the list of
    options.
- `_ZO_MAXAGE`
  - Configures the [aging algorithm][algorithm-aging], which limits the maximum
    number of entries in the database.
  - By default, this is set to `10000`.
- `_ZO_RESOLVE_SYMLINKS`
  - When set to `1`, `z` will resolve symlinks before adding directories to the
    database.

## Third-party integrations

- [nnn], via [autojump plugin][nnn-autojump]
- [xxh], via [xxh-plugin-prerun-zoxide]

[algorithm-aging]: https://github.com/ajeetdsouza/zoxide/wiki/Algorithm#aging
[algorithm-matching]: https://github.com/ajeetdsouza/zoxide/wiki/Algorithm#matching
[alpine linux packages]: https://pkgs.alpinelinux.org/packages?name=zoxide
[aur]: https://aur.archlinux.org/packages/zoxide-bin
[copr]: https://copr.fedorainfracloud.org/coprs/atim/zoxide/
[crates.io-badge]: https://img.shields.io/crates/v/zoxide
[crates.io]: https://crates.io/crates/zoxide
[debian packages]: https://packages.debian.org/testing/admin/zoxide
[demo.gif]: demo.gif
[dports]: https://github.com/DragonFlyBSD/DPorts/tree/master/sysutils/zoxide
[fedora packages]: https://src.fedoraproject.org/rpms/rust-zoxide
[freshports]: https://www.freshports.org/sysutils/zoxide/
[fzf-installation]: https://github.com/junegunn/fzf#installation
[fzf]: https://github.com/junegunn/fzf
[glob]: https://man7.org/linux/man-pages/man7/glob.7.html
[homebrew]: https://formulae.brew.sh/formula/zoxide
[linuxbrew]: https://formulae.brew.sh/formula-linux/zoxide
[macports]: https://ports.macports.org/port/zoxide/summary
[nixpkgs]: https://github.com/NixOS/nixpkgs/blob/master/pkgs/tools/misc/zoxide/default.nix
[nnn-autojump]: https://github.com/jarun/nnn/blob/master/plugins/autojump
[nnn]: https://github.com/jarun/nnn
[pkgsrc]: https://pkgsrc.se/sysutils/zoxide
[releases]: https://github.com/ajeetdsouza/zoxide/releases
[scoop]: https://github.com/ScoopInstaller/Main/tree/master/bucket/zoxide.json
[termux]: https://github.com/termux/termux-packages/tree/master/packages/zoxide
[void linux packages]: https://github.com/void-linux/void-packages/tree/master/srcpkgs/zoxide
[xxh-plugin-prerun-zoxide]: https://github.com/xxh/xxh-plugin-prerun-zoxide
[xxh]: https://github.com/xxh/xxh
[z.lua]: https://github.com/skywind3000/z.lua
[z]: https://github.com/rupa/z
