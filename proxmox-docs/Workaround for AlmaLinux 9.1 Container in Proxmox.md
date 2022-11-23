# Workaround for AlmaLinux 9.1 Container in Proxmox

There is a currently an initialization problem with LXC container running AlmaLinux 9.1 where LXC doesn't recognize the hosted OS.

So we need to update the CentOS.pm file in order to let LXC recognize the 9.1 version of AlmaLinux.

```bash
nano /usr/share/perl5/PVE/LXC/Setup/CentOS.pm
```

```bash
if ($1 >= 5 && $1 <= 9)
```

Increment the version from 9.0 to 9.1 (should be in line 25)

```bash
if ($1 >= 5 && $1 <= 9.1)
```

Now you should be able to start your container.

------

*References:*

*[ **forum.proxmox.com** ] : https://forum.proxmox.com/threads/almalinux-9-lxc-container-startet-nach-upgrade-auf-9-1-nicht-mehr.118248/ "AlmaLinux 9 LXC Container startet nach Upgrade auf 9.1 nicht mehr"*

