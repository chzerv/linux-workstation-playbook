# Linux Workstation Setup Playbook

[![Archlinux](https://img.shields.io/badge/arch-linux-blue.svg?style=flat-square&logo=Arch-Linux&logoColor=white)](https://archlinux.org)
[![Fedora](https://img.shields.io/badge/Fedora-33-blue.svg?style=flat-square&logo=Fedora&logoColor=white&)](https://getfedora.org)

This playbook performs all the standard setup I do on my Linux box(es).
I usually use Fedora and/or Archlinux, so those are the only supported distributions.

# What does this do?

This playbook performs the following tasks:

- Configure `pacman` and `dnf` so they provide better output and are faster.

- Install packages (npm, pip and flatpak are also supported).

- Configure `systemd` so regular users can manage CPU and memory limits using [systemd slices](https://www.freedesktop.org/software/systemd/man/systemd.slice.html).

- Use my [sysctl role](https://github.com/chzerv/ansible-role-sysctl) to change `sysctl` values.
  Those values are defined using Ansible variables.

- Try to improve performance by:

  - Tweaking Virtual Memory and swapiness, installing and configuring [earlyoom](https://github.com/rfjakob/earlyoom). 
  - Changing I/O schedulers depending on disk type (SSD v HDD v NVMe).
  - Disabling watchdogs (can also provide lower power consumption).

- Use my [security Ansible role](https://github.com/chzerv/ansible-role-security) to perform basic system hardening. The role can be configured through Ansible variables.

- Set capabilities for some known programs that require `sudo`. This way, any user can use them without privilege elevation. These programs are:

  - `beep`,
  - `chvt`
  - `iftop`,
  - `mii-tool`
  - `mtr-packet` and
  - `nethogs`

- Configure the GNOME Desktop Environment. The configuration includes:

  - Change default keybindings and add new keybindings.
  - Change the fonts used in the Shell.
  - Configure input sources.
  - Nautilus tweaks.
  - and others..

- Clone my [dotfiles](https://github.com/chzerv/dotfiles), symlink them to the right places and install required packages.

- Archlinux specific tweaks like:

  - Make `makepkg` use all available CPU cores for compiling.
  - Disable `pam_systemd_homed.so` to avoid journal spamming (**Archlinux only**). Make sure to set `disable_pam_systemd_homed` to `false` if you use `systemd-homed`.
  - Setup `pacman` hooks for `reflector`, so the mirrorlist is automatically generated whenever there is an update. 
  - Configure `reflector`.

- Minor configuration changes like:

  - Change `roles_path` and `remote_tmp` in `ansible.cfg`.
  - Increase `default-cache-ttl` and `max-cache-ttl` values for `gpg-agent`.
  - Add additional kernel parameters. Only works for `GRUB` and `systemd-boot`.
  - Enable overclocking for AMD GPUs.
  
- Setup virtualization. More specifically:
  
  + Enable rootless `podman`.
  + Use my [libvirt Ansible role](https://galaxy.ansible.com/chzerv/libvirt) to install and configure `libvirt`.

# Usage

1. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
2. Clone this repository to your machine: `git clone https://github.com/chzerv/linux-workstation-playbook.git`
3. Install required Ansible roles: `ansible-galaxy install -r requirements.yml`
4. Make sure to **read** the variables and their explanations below.
4. Run the playbook: `ansible-playbook -i inventory -K main.yml`. Enter your password when prompted.

# Stuff that still have to be done manually

- Install tmux plugins: `$prefix + I` in an already running tmux session.

# Variables, their default values and their explanations

```yaml
lwp_install_packages: true
lwp_install_extra_packages: true
lwp_configure_systemd: true
lwp_tweak_sysctl: true
lwp_tweak_performance: true
lwp_harden_system: true
lwp_apply_capabilities: true
lwp_configure_gnome: true
lwp_dotfiles: true
lwp_virtualization: true
lwp_misc: true
```

> - Install packages from the official repositories.
> - Install packages from third party repositories (npm, pip, flatpak).
> - Apply systemd related tweaks.
> - Tweak `sysctl` values.
> - Apply performance tweaks.
> - Apply basic hardening to the system.
> - Change capabilities for some known programs.
> - Configure GNOME.
> - Setup my [dotfiles](https://github.com/chzerv/dotfiles).
> - Setup virtualization.
> - Perform some misc tasks.

```yaml
arch_enable_multilib: true
fedora_enable_rpmfusion_free: true
fedora_enable_rpmfusion_non_free: true
```

> - Enable `multilib` repository for Arch linux
> - Enable Fedora's `RPMFustion Free` repository.
> - Enable Fedora's `RPMFustion Non Free` repository.

``` yaml
flatpak_repositories: []
# flatpak_repositories:
#   - name: flathub
#     state: present #present/absent, default: present
#     flatpakrepo_url: https://dl.flathub.org/repo/flathub.flatpakrepo
#     method: system #system/user, default: system
```

> Flatpak repositories to add/remove.
>
> - `name` and `flatpakrepo_url` are **required**.
> - `state` defaults to `present`, unless otherwise specified.
> - `method` defaults to `system`, unless otherwise specified.

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
    virtualenv: ~/.venvs # default: ~/.venvs
```

> Packages to install with Python's `pip`. By default:
>
> - Packages will be installed, unless `state` is set to `absent` which will remove them, or `latest` which will update them.
> - The latest version of a package will be installed unless the `version` argument is specified.
> - Packages will be installed in the `~/.venvs` virtual environment, unless the `virtualenv` argument is set to another path.

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

``` yaml
sysctl_required_packages: procps-ng
sysctl_entries:
  - name: vm.swappiness
    value: 10
  - name: vm.dirty_ratio
    value: 6
  - name: vm.dirty_background_bytes
    value: 4194304
  - name: vm.vfs_cache_pressure
    value: 50
  - name: kernel.sysrq
    value: 1
```

> Tweak `sysctl` values using my [sysctl role](https://github.com/chzerv/ansible-role-sysctl). Make sure to check the role itself for more details. 

``` yaml
security_ssh_port: 22
security_fail2ban_enabled: false
security_kern_hidepid_value: "0"
security_autoupdates_enabled: false
security_nproc_limit: false
```

> Variables specific to my [security Ansible role](https://github.com/chzerv/ansible-role-security). Make sure to check the role for more details.

``` yaml
libvirt_users_in_libvirt_group: "{{ ansible_user_id }}"
libvirt_install_virt_manager: true
libvirt_uefi_vm_support: true
libvirt_access_vms_using_hostnames: false
```

> Variables specific to my [libvirt Ansible role](https://github.com/chzerv/ansible-role-libvirt). Make sure to check the role for more details.

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
enable_rootless_podman: true
allow_amdgpu_overclock: true
```

> - gpg-agent's default-cache-ttl value.
> - gpg-agent's max-cache-ttl value.
> - The path where Ansible will store and look for roles.
> - Whether to disable pam_systemd_homed.so pam module. **Only applies in Archlinux**.
> - Whether to allow the current user to run podman containers without root. If set to `true`, the `cgroup_no_v1="all"` kernel parameter will be set, which effectively disables cgroups v1 and enables cgroups v2. **DO NOT** use this if you want to use `docker` instead of `podman`.
> - Whether to allow overclocking for AMD GPUs. If set to `true`, the `amdgpu.ppfeaturemask=0xfffd7fff` kernel parameter will be set.
