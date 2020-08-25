# Linux Workstation Setup Playbook

[![Archlinux](https://img.shields.io/badge/arch-linux-blue.svg?style=flat-square&logo=Arch-Linux&logoColor=white)](https://archlinux.org)
[![Fedora](https://img.shields.io/badge/Fedora-v32-blue.svg?style=flat-square&logo=Fedora&logoColor=white&)](https://getfedora.org)

This playbook automates the standard setup I do on my Linux box(es).
I usually use Fedora and/or Archlinux, so those are the only supported distributions.

# Usage

1. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
2. Clone this repository to your machine: `git clone https://github.com/chzerv/linux-workstation-playbook.git`
3. Install required Ansible roles: `ansible-galaxy install -r requirements.yml`
4. Run the playbook: `ansible-playbook -i inventory -K main.yml`. Enter your password when prompted.

All the tasks are tagged. To list all available tags, run:

```sh
ansible-playbook main.yml --list-tags
```

- `capabilities` will only run capabilities related tasks.
- `sysctl` will only run sysctl related tasks.
- `performance` will only run performance related tasks.
- `gnome` will only run GNOME related tasks.
- `systemd` will only run systemd related tasks.
- `dotfiles` will only run dotfiles related tasks.
- `install-extra-packages` will only run tasks that install packages using flatpak/npm/pip.
- `misc` will only run misc tasks.

To choose a specific tag, run:

```sh
ansible-playbook -i inventory -K main.yml --tags=dotfiles
```

where `dotfiles` can be any of the above.

# What does this do?

This playbook performs the following tasks:

- Configure pacman and dnf so they both provide better output and are faster.
- Update system and install packages (npm, pip and flatpak are also supported).
- Better handling of OOM conditions by tweaking Virtual Memory and installing/configuring [earlyoom](https://github.com/rfjakob/earlyoom).
- Change I/O schedulers depending on disk type (SSD vs HDD vs NVMe).
- Disable watchdogs for better performance and lower power consumption.
- Download my dotfiles and symlink them to the right places.
- Set capabilities for some programs that require sudo. This way, any user can use them without privilege elevation.
- Apply some minor Gnome Shell tweaks.
- Change ansible roles path.
- Change gpg-agent's `default-cache-ttl` and `max-cache-ttl` values.
- Disable `pam_systemd_homed.so` to avoid journal spamming (**Archlinux only**). Make sure to set `disable_pam_systemd_homed` to `false` if you use `systemd-homed`.

# Stuff that still have to be done manually

- Install Doom Emacs: `~/.emacs.d/bin/doom install`.
- Install tmux plugins: `$prefix + I` in an already running tmux session.

# Variables, their default values and their explanations

```yaml
configure_capabilities: true
configure_sysctl: true
configure_performance: true
configure_gnome: true
configure_systemd: true
configure_dotfiles: true
configure_misc: true
```

> - Perform `capabilities` related tasks. This can also be controlled via Ansible tags.
> - Perform `sysctl` related tasks. This can also be controlled via Ansible tags.
> - Perform `performance` related tasks. This can also be controlled via Ansible tags.
> - Perform `gnome` related tasks. This can also be controlled via Ansible tags.
> - Perform `systemd` related tasks. This can also be controlled via Ansible tags.
> - Perform `dotfiles` related tasks. This can also be controlled via Ansible tags.
> - Perform `misc` tasks. This can also be controlled via Ansible tags.

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
    method: user # user/system, default: user
    state: present # present/absent/latest, default: present

# OR...
flatpak_packages:
  - com.discordapp.Discord
```

> Packages to install from flatpak repositories. By default:
>
> - Packages will be installed, unless `state` is set to `absent` which will remove them.
> - Packages will be installed from `flathub`, unless `remote` is specified.
> - Packages will be installed for the current user, unless `method` is set to `system`.

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
dotfiles_repo: git@github.com:chzerv/dotfiles.git
dotfiles_local_destination: "{{ ansible_user_dir }}/dotfiles"
dotfiles_accept_hostkey: true
```

> - The remote repository where dotfiles are present.
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
