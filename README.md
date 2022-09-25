<h1 align="center">Linux Workstation Setup, via Ansible</h1>


This is a set of playbooks and roles that perform all the setup I do on my Linux systems:

+ Distribution specific tweaks like:
    + Make `pacman`, `makepkg` and `dnf` faster and improve their output
    + Enable the [amd_pstate](https://docs.kernel.org/admin-guide/pm/amd-pstate.html) scaling driver in Archlinux
    + Remove packages and services that *I* don't use from Fedora
+ Install and configure required software
+ Deploy my [dotfiles](https://github.com/chzerv/dotfiles)
+ GNOME tweaks
+ Install and configure [podman](https://podman.io/), [distrobox](https://github.com/89luca89/distrobox), [vagrant](https://www.vagrantup.com/), and [libvirt](https://libvirt.org/) (or virtualbox)
+ Power-saving tweaks
+ Basic system hardening


:warning: This is extremely personalized, so if you intend to use it make sure to read the code and make adjustments according to your needs.
