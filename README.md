# dnf5keeper

Declarative package management for DNF5. Useful with Fedora and other related distros.

Note that only version 41+ of Fedora uses DNF version 5, and this script is not compatible with DNF 4 or lower.

## Why use this?

It can be hard to remember why I've installed all the packages on my system. This can cause various issues:

* __My system is full of crufty packages that I don't really need__. For example, if I temporarily install some packages to compile something, I'll probably never remove them later.
* __It's easy to accidentally remove important packages__. For example, if I install `rubygem-pry`, that will pull in `ruby`, and mark it as automatically installed. Later if I decide I don't want rubygem-pry, a `dnf autoremove` will get rid of ruby, even if I'm using it for lots of other things!
* __It's hard to setup my packages on a new system__. I can't just look at my previous system, cuz I have no idea what all its packages were for.
* __I can't tell what I used to have in the past__. If I was previously developing on some project, I probably installed a bunch of packages, and maybe removed them later. If I want to get back to that project, I doubt I can remember exactly which packages I used.

## How to use this

* Make sure you have our only dependency installed, the package `ruby`.
* Copy `dnf5keeper` somewhere in your PATH, for example ~/bin or /usr/local/bin.
* Create a directory ~/.config/dnf5keeper to declare your packages
  * Inside that directory, create one or more files with the extension `.keep`.
  * Each file should contain a list of packages you want to keep installed. Comments starting with '#' are allowed, as are blank lines.
  * You can get a reasonable initial list of packages with `dnf5keeper candidates`
* Once you think you're satisfied, run `dnf5keeper diff`. It will tell you what it thinks needs removing or installing.
  * Then, run `dnf5keeper sync` to do the installation or removal! It will take care of installing and removing dependencies.
* Later, when you want to make changes, just edit your .keep files, then run `dnf5keeper sync`

An example .keep file might look like:
```
# Browsers
firefox
chromium
epiphany-browser # aka "Gnome Web"

ripgrep tmux git # CLI tools
```

## Tips

* Put your dnf5keeper directory in git! Then you'll have a history of your changes.
* Use comments to remind yourself why you wanted a package, so you know whether you still need it.
* Need a single package, and too lazy to open your editor? Just run `dnf5keeper add foo "my reason" mypackage`, and it will automatically put it in `foo.keep` and install it. Then you can organize it properly later, when you have more time.
* Put packages required for a project in a separate .keep file, so you remember that they're related. Now they're easy to remove all at once!
   * Similarly, you can put related packages nearby each other, so they're easy to block-comment and get rid of.
* There's no harm in listing a package twice! Eg: if it's needed for multiple projects, you can list it in multiple .keep files.
* Copy your whole dnf5keeper dir to a new system, to setup packages just like on the old one.
* Feel free to install some packages temporarily with dnf, without using dnf5keeper, as long as it's safe for them to be autoremoved later.

## Detailed usage

### Common commands

#### dnf5keeper sync

Install and remove packages to match the keeper files

#### dnf5keeper diff

Print the packages that would be installed or removed by a sync

#### dnf5keeper add BASENAME REASON PACKAGE [PACKAGE...]

Adds packages to BASENAME.keep with a comment, and installs them. Good for small changes when you're too busy to open an editor.

#### dnf5keeper candidates

List packages not listed in .keep, and with no dependencies. Good for finding candidates to add to .keep files, especially when starting out.

### Other commands

keepers: List packages in .keep files
missing: List packages to be installed
unneeded: List packages to be removed
install: Install missing packages (but don't remove anything)
remove: Remove unneeded packages (but don't install anything)
update-reasons: Unconditionally update DNF's package reasons, according to the keepers
external - Show packages whose reason is "external", ie: not installed via dnf. You should probably use `dnf mark` to clarify what these are for.

## Why not use...?

### dnf mark

DNF already has a built-in concept of marking package installation reasons as "user" or "depedency". You can use the `dnf mark` command to manipulate this, and `dnf autoremove` will cleanup packages that are longer needed.

But:
* dnf mark has no user-visible keeper file at all!
  * You can't put your keepers in git.
  * You can't add comments or split things into multiple files.
* By default, dnf assumes that when you install a package, it's desired as a keeper. This makes it too easy to install packages without remembering why.

### pacdef

[pacdef](https://github.com/steven-omaha/pacdef) is a multi-backend declarative package management system. You can manage packages across a variety of different distros and language-specific package managers, like cargo or pip. It supports a hierarchical system of package grouping, where entire groups of packages can be imported or removed from your system.

But:
* There are [serious bugs](https://github.com/steven-omaha/pacdef/issues/90) that make pacdef very difficult to compile.
* It's much more complex than I need.
* It doesn't support certain use cases I like, such as commenting out a whole file when I'm done with a project.

### ansible, chef, puppet

These are great tools to set up systems. But they're designed for production systems that should mostly stay unchanged. End-user systems constantly have packages being installed and removed, and are a better fit for something like dnf5keeper.

## Caveats

* dnf5keeper will overwrite your DNF reasons file. I think of this as a feature, since it makes it easier to use all these tools together. But you might not! Dnf5keeper does keep backups in /usr/lib/sysimage/libdnf5/packages.toml.bak
* Your .keep files are __not__ a full description of the packages installed on your system. DNF dependencies can be optional, or can have multiple packages that satisfy a given dependency, and there's no way for dnf5keeper to understand your intentions. Typically it works, but if you need to have absolutely identical packages on different systems, use a different solution .
* dnf5keeper may consider optional dependencies as important enough to keep the dependent package installed. I'm not sure whether there's a way to configure this.

## Inspiration

* [aptkeeper](https://github.com/vasi/aptkeeper), my nearly equivalent system for apt-based distros.

* [debfoster](https://packages.debian.org/sid/debfoster) for apt-based distros (Debian, Ubuntu, etc)
* [pacdef](https://github.com/steven-omaha/pacdef) for Arch-based distros
* [pacmanfile](https://github.com/cloudlena/pacmanfile), also for Arch-based distros
* Homebrew's [bundle](https://github.com/Homebrew/homebrew-bundle) capabilities.
* The [set -A](https://man.freebsd.org/cgi/man.cgi?query=pkg-set&sektion=8&apropos=0&manpath=FreeBSD+14.2-RELEASE+and+Ports) and [autoremove](https://man.freebsd.org/cgi/man.cgi?query=pkg-autoremove&sektion=8&apropos=0&manpath=FreeBSD+14.2-RELEASE+and+Ports) subcommands of FreeBSD's pkg
* The [selected set](https://wiki.gentoo.org/wiki/Selected_set_(Portage)) of Gentoo's Portage, along with the `--select`, `--deselect` and `--depclean` options.
* The [mark](https://dnf.readthedocs.io/en/latest/command_ref.html#mark-command-label) and [autoremove](https://dnf.readthedocs.io/en/latest/command_ref.html#autoremove-command-label) subcommands of Fedora's dnf
* Of course, the declarative systems of [Nix](https://nixos.org/) and [Guix](https://guix.gnu.org/)

Also some previous attempts of mine at this sort of thing:
* [brewfoster](https://github.com/vasi/brewfoster) for Homebrew
* [rpmkeeper and yumkeeper](https://github.com/vasi/rpmkeeper) for yum-based distros
* [zyppkeeper](https://github.com/vasi/zyppkeeper/) for OpenSUSE's zypper.
