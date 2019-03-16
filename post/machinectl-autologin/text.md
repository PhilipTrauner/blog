# Developer Diary Excerpts: machinectl auto-login

Bringing up a root filesystem with `systemd-nspawn -b -D` and consequently logging into the container necessitates that either a password is set for the desired user, or that an [`autologin` override is present for the `console-getty` service](<https://wiki.archlinux.org/index.php/Getty#Nspawn_console>). 

If a container is brought up with `machinectl start` instead, the `console-getty` override has no effect for consequent `machinectl login` invocations, as `container-getty@.service` is in charge of the pseudo terminals that `login` attaches to instead.

```
    State: running
     Jobs: 0 queued
   Failed: 0 units
    Since: Sat 2019-03-16 17:11:26 CET; 16min ago
   CGroup: /
           ├─init.scope
           │ └─1 /lib/systemd/systemd
           └─system.slice
             ├─console-getty.service
             │ └─48 /sbin/agetty --noclear --keep-baud console 115200,38400,9600 vt220
             └─system-container\x2dgetty.slice
               └─container-getty@0.service
                 ├─53 /bin/login -f
                 └─58 -bash
```

<figcaption>Shortened <code>systemctl status</code> output of container.</figcaption>

Logging into the container without a password can still be achieved by using the [`shell` subcommand](https://www.freedesktop.org/software/systemd/man/machinectl.html#shell%20%5B%5BNAME@%5DNAME%20%5BPATH%20%5BARGUMENTS%E2%80%A6%5D%5D%5D%20) of [`machinectl`](https://github.com/systemd/systemd/pull/1022), though the `login` subcommand remains the only option for Linux distributions that ship with older versions of `systemd`, which do not include `shell ` (before [v224](https://github.com/systemd/systemd/releases/tag/v224)).

An `ExecStart` override for `container-getty@.service` can be used to replicate the behaviour of `shell` with `login`.

```shell
systemctl edit container-getty@.service
```
<figcaption>Execute inside the container, not the host system (a text editor has to be installed).</figcaption>

```ini
[Service]
ExecStart= 
ExecStart=-/sbin/agetty --noclear --autologin root --keep-baud pts/%I 115200,38400,9600 $TERM 
```

* The empty space after the first `ExecStart` it there on purpose, as it tells `systemd` to clear the pre-existing `ExecStart` content.
* Substituting `root` with another user name will instead enable automatic login for the specified user.

## Notes

* `systemd` as well as `dbus` should be installed in the container root file-system, otherwise both `login` and `shell` will fail (`Failed to get shell PTY: Protocol error`).
* The override doesn't necessarily have to be created with the `edit` subcommand of `systemd`. Creating and subsequenly populating the contents of `/etc/systemd/system/container-getty@.service.d/override.conf` also works just as well.