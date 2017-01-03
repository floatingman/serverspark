# ServerSpark

ServerSpark is an [Ansible][1] playbook meant to provision a personal machine running
[Arch Linux][2]. It is intended to run locally on a fresh Arch install (ie,
taking the place of any [post-installation][3]), but due to Ansible's
idempotent nature it may also be run on top of an already configured machine.

ServerSpark assumes it will be run on a headless server and performs some configuration based on this assumption.

## Running

First, sync mirrors and install Ansible:

    $ pacman -Syy python2-passlib ansible

Second, install and update the submodules:

    $ git submodule init && git submodule update
    
Run the playbook as root.

    # ansible-playbook -i localhost playbook.yml

When run, Ansible will prompt for the user password. This only needs to be
provided on the first run when the user is being created. On later runs,
providing any password -- whether the current user password or a new one --
will have no effect.

## SSH

By default, Ansible will attempt to install the private SSH key for the user. The key should be available at the path specified in the 
`ssh.user_key` variable.
Removing this variable will cause the key installation task to be skipped.

### SSHD

If `ssh.enable_sshd` is set to `True` the [systemd socket service][4] will be
enabled. By default, sshd is configured but not enabled.

## Dotfiles

Ansible expects that the user wishes to clone dotfiles via the git repository
specified via the `dotfiles.url` variable and install them with [rcm][5]. If
this is not the case, removing the `dotfiles` variable will cause the relevant
tasks to be skipped.

## Tagging

All tasks are tagged with their role, allowing them to be skipped by tag in
addition to modifying `playbook.yml`.

## AUR

All tasks involving the [AUR][6] are tagged `aur`. To provision an AUR-free
system, pass this tag to ansible's `--skip-tag`.

AUR packages are installed via the [ansible-aur][7] module. Note that while
[aura][8], an [AUR helper][9], is installed by default, it will *not* be used
during any of the provisioning.

## Tarsnap

[Tarsnap][19] is installed with its default configuration file. However,
setting up Tarsnap is left as an exercise for the user. New Tarsnap users
should [register their machine and generate a key][20]. Existing users should
recover their key(s) and cache directory from their backups (or, alternatively,
recover their key(s) and rebuild the cache directory with `tarsnap --fsck`).

[Tarsnapper][21] is installed to manage backups. A basic configuration file to
backup `/etc` is included. Tarsnapper is configured to look in
`/usr/local/etc/tarsnapper.d` for additional jobs. As with with the Tarsnap key
and cache directory, users should recover their jobs files from backups after
the Tarsnapper install is complete. See the Tarsnapper documentation for more
details.

### Scheduling Tarsnap

A systemd unit file and timer are included for Tarsnapper. The timer is set to
execute Tarsnapper hourly (configurable through the `tarsnapper.timer.schedule`
variable). 

## Adding custom pacman repositories

Take a look in the haskell roles for examples on adding custom repositories that need signed keys

[1]: http://www.ansible.com
[2]: https://www.archlinux.org
[3]: https://wiki.archlinux.org/index.php/Installation_guide#Post-installation
[4]: https://wiki.archlinux.org/index.php/Secure_Shell#Managing_the_sshd_daemon
[5]: https://thoughtbot.github.io/rcm/
[6]: https://aur.archlinux.org
[7]: https://github.com/pigmonkey/ansible-aur
[8]: https://github.com/aurapm/aura
[9]: https://wiki.archlinux.org/index.php/AUR_helpers
[10]: https://firejail.wordpress.com/
[11]: https://github.com/EtiennePerot/macchiato
[12]: https://github.com/pigmonkey/nmtrust
[13]: http://isync.sourceforge.net/
[14]: http://offlineimap.org/
[15]: http://msmtp.sourceforge.net/
[16]: http://sourceforge.net/p/msmtp/code/ci/master/tree/scripts/msmtpq/README.msmtpq
[17]: https://github.com/pimutils/vdirsyncer
[18]: https://wiki.archlinux.org/index.php/Systemd/Timers
[19]: https://www.tarsnap.com/
[20]: https://www.tarsnap.com/gettingstarted.html
[21]: https://github.com/miracle2k/tarsnapper
[22]: https://www.torproject.org/
[23]: https://github.com/EtiennePerot/parcimonie.sh
[24]: https://www.bitlbee.org/main.php/news.r.html
[25]: https://weechat.org/
[26]: https://git-annex.branchable.com/
[27]: http://www.postgresql.org/
