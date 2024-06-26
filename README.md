# genup
Tool to update the **Portage**(5) tree, all installed packages, and kernel, under Gentoo Linux.

## Info on this fork
I forked this project from [sakaki-/genup](https://github.com/sakaki-/genup) after the Gentoo devs came up with a rather ridiculous USE flag change to Perl and associated modules, which caused portage itself to get into a state from where it can't recover on its own when genup was being unattended and/or the system owner did not follow the suggestion to abort the update of Perl on account of a mismatch between the old USE flag and the new global one.

It needs to be noted here that I can in fact not change any such errors made by the main Gentoo devs other than report them and hope that they will acknowledge their error and create a workable fix for it. Similarly I can not change portage offering to write incorrect flags to package.use that if accepted will in fact cause the system to become corrupted. The updates in this fork specifically target users that have the `bubba` meta package installed on the Excito B2|B3 machine and through which I am able to push forward out of the ordinary required configuration changes.

## Description
**genup** is a utility intended to simplify the process of keeping your Gentoo system up to date. When invoked, it automatically performs the following steps, in order:
* updates Portage tree (and active overlays), and syncs **eix**(1)
(using `emaint sync` / `eix-sync`)
* removes any prior **emerge**(1) resume history
(using `emaint --fix cleanresume`)
* on `aarch64`, attempts to apply any pending fixups
(if desired, by running `/etc/cron.weekly/fixup`; errors non-fatal)
* ensures **Portage**(5) itself is up-to-date
(using `emerge --oneshot --update portage`)
* ensures **genup** itself is up-to-date (restarting if not)
(using `emerge --oneshot --update genup`)
* updates all packages in the @world set
(first using **emtee**(1), if the matching USE flag is set, and then using `emerge --deep --with-bdeps=y --changed-use --update @world`)
* removes unreferenced packages
(using `emerge --depclean`)
* rebuilds any external modules (such as those for VirtualBox)
(using `emerge @module-rebuild --exclude '*-bin'`)
* rebuilds any packages depending on stale libraries
(using `emerge @preserved-rebuild`)
* updates any old **perl(1)** modules
(using `perl-cleaner --all`)
* resolves clashing config file changes (in interactive mode)
(using `dispatch-conf`)
* upgrades the kernel if possible (to staging, in _/boot_)
(using `buildkernel --stage-only`)
* removes unreferenced packages (again)
(using `emerge --depclean`)
* fixes missing shared library dependencies
(using `revdep-rebuild`)
* rebuilds any packages depending on stale libraries (again)
(using `emerge @preserved-rebuild`)
* removes any unused source tarballs (if desired)
(using `eclean --deep distfiles`)
* deploys new kernel from staging (if desired and available)
(using `buildkernel --copy-from-staging`)
* updates environment settings (as a precautionary measure)
(using `env-update`)
* updates `eix` package metadata
(using `eix-sync -0`)
* runs any custom updaters in /etc/genup/updaters.d

The genup utility can be invoked in non-interactive (default) or interactive mode (see the **--ask** option in the manpage). Non-interactive mode is suitable for use in a scripted invocation, for example as part of a nightly **cron**(8) job.

## Installation

**genup** is best installed (on Gentoo) via its ebuild, available as part of the **bubbas** [overlay](https://github.com/gordonb3/bubba-overlay).

