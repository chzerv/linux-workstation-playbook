# Linux Workstation Setup Playbook

[![Archlinux](https://img.shields.io/badge/arch-linux-blue.svg?style=flat-square&logo=Arch-Linux&logoColor=white)](https://archlinux.org)
[![Fedora](https://img.shields.io/badge/Fedora-v32-blue.svg?style=flat-square&logo=Fedora&logoColor=white&)](https://getfedora.org)

This playbook performs all the standard setup I do on my Linux box(es).
I usually use Fedora and/or Archlinux, so those are the only supported distributions.

# What does this do?

This playbook performs the following tasks:

- Configure `pacman` and `dnf` so they provide better output and are faster.

- **[tags=install-software, install-extra-software]** Update system and install packages (npm, pip and flatpak are also supported).

- **[tags=performance]** Try to improve performance, especially in OOM conditions, by:

  - Tweaking Virtual Memory and swapiness, installing and configuring [earlyoom](https://github.com/rfjakob/earlyoom).
  - Change I/O schedulers depending on disk type (SSD vs HDD vs NVMe).
  - Disable watchdogs for better performance and lower power consumption.

- **[tags=security]** Perform some basic system hardening using my [security Ansible role](https://galaxy.ansible.com/chzerv/security). Make sure to check the role for available variables.

- **[tags=dotfiles]** Clone my [dotfiles](https://github.com/chzerv/dotfiles), symlink them to the right places and install required packages.

- **[tags=capabilities]** Set capabilities for some programs that require sudo. This way, any user can use them without privilege elevation.

- **[tags=systemd]** Allow users to manage CPU and Memory resources using `cgroups` and create a user-defined slice. This slice can be then used to limit the resources that some user services consume. Read more on `cgroups` in the [Arch Wiki article](https://wiki.archlinux.org/index.php/Cgroups).

- **[tags=gnome]** Configure GNOME Shell to my liking. This includes:

  - Change keybindings.
  - Disable some search providers.
  - Change system-wide fonts.

- **[tags=misc]** Minor configuration changes like:

  - Change `roles_path` and `remote_tmp` in `ansible.cfg`.
  - Increase `default-cache-ttl` and `max-cache-ttl` values in `gpg-agent.conf`
  - Disable `pam_systemd_homed.so` to avoid journal spamming (**Archlinux only**). Make sure to set `disable_pam_systemd_homed` to `false` if you use `systemd-homed`.

# Usage

1. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
2. Clone this repository to your machine: `git clone https://github.com/chzerv/linux-workstation-playbook.git`
3. Install required Ansible roles: `ansible-galaxy install -r requirements.yml`
4. Run the playbook: `ansible-playbook -i inventory -K main.yml`. Enter your password when prompted.

As show in the [What does this do?](https://github.com/chzerv/linux-workstation-playbook#what-does-this-do) section, all tasks are tagged.

To choose a specific tag, run:

```sh
ansible-playbook -i inventory -K main.yml --tags=dotfiles
```

Or, if you want to include all the tags except some, run:

```sh
ansible-playbook -i inventory -K main.yml --skip-tags=dotfiles
```

where `dotfiles` can be any of the above.

# Stuff that still have to be done manually

- Install tmux plugins: `$prefix + I` in an already running tmux session.

# Variables, their default values and their explanations

```yaml
configure_capabilities: true
configure_sysctl: true
configure_security: true
configure_performance: true
configure_gnome: true
configure_systemd: true
configure_dotfiles: true
configure_misc: true
```

> - Perform `capabilities` related tasks.
> - Perform `sysctl` related tasks.
> - Perform `security` related tasks.
> - Perform `performance` related tasks.
> - Perform `gnome` related tasks.
> - Perform `systemd` related tasks.
> - Perform `dotfiles` related tasks.
> - Perform `misc` tasks.
>
> If you prefer using tags, just leave these variables to `true` and use tags instead.

```yaml
arch_enable_multilib: true
fedora_enable_rpmfusion_free: true
fedora_enable_rpmfusion_non_free: true
```

> - Enable `multilib` repository for Arch linux
> - Enable Fedora's `RPMFustion Free` repository.
> - Enable Fedora's `RPMFustion Non Free` repository.

```yaml
flatpak_packages:
  - name: com.discordapp.Discord
    remote: flathub # defaults to flathub
    method: user # user/system, default: system
    state: present # present/absent/latest, default: present

# OR...
flatpak_packages:
  - com.discordapp.Discord
```

> Packages to install from flatpak repositories. By default:
>
> - Packages will be installed, unless `state` is set to `absent` which will remove them.
> - Packages will be installed from `flathub`, unless `remote` is specified.
> - Packages will be installed system wide, unless `method` is set to `user`.

```yaml
pip_packages:
  - name: mkdocs
    state: present # present/absent/latest, default: present
    version: "0.16.3" # default: N/A
    virtualenv: ~/.venvs # default: N/A
```

> Packages to install with Python's `pip`. By default:
>
> - Packages will be installed, unless `state` is set to `absent` which will remove them, or `latest` which will update them.
> - The latest version of a package will be installed unless the `version` argument is specified.
> - Packages will be installed globally unless the `virtualenv` argument is set, which will install them inside an **already setup** virtual environment.

```yaml
npm_packages:
  - name: webpack
    state: present # present/absent/latest, default: present
    version: "^2.6" # default: N/A
```

> Packages to install with Javascript's `npm`. By default:
>
> - Packages will be installed, unless `state` is set to `absent` which will remove them, or `latest` which will update them.
> - The latest version of a package will be installed unless the `version` argument is specified.

```yaml
mutter_enable_experimental: false
```

> Whether to enable mutter experimental features or not.

```yaml
dotfiles_repo: https://github.com/chzerv/dotfiles.git
dotfiles_local_destination: "{{ ansible_user_dir }}/dotfiles"
dotfiles_accept_hostkey: true
```

> - The remote repository where dotfiles are stored.
> - Where the remote repository will be cloned to.
> - Whether to accept the requested SSH host key or not (only needed if using SSH to clone the remote repository).

```yaml
dotfiles_config_choices:
  - alacritty
  - bat
  - nvim
  - tilix
  - tmux
  - zsh
  - doom
```

> The applications for which configuration will be applied. Probably only works for my own [dotfiles](https://github.com/chzerv/dotfiles).

```yaml
gpg_agent_default_cache_ttl: "28800"
gpg_agent_max_cache_ttl: "28800"
ansible_roles_path: "~/Code/Infra/Ansible/roles"
disable_pam_systemd_homed: true
```

> - gpg-agent's default-cache-ttl value.
> - gpg-agent's max-cache-ttl value.
> - The path where Ansible will store and look for roles.
> - Whether to disable pam_systemd_homed.so pam module. **Only applies in Archlinux**.
