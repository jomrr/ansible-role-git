# ansible-role-git

![GitHub](https://img.shields.io/github/license/jam82/ansible-role-git) ![GitHub last commit](https://img.shields.io/github/last-commit/jam82/ansible-role-git) ![GitHub issues](https://img.shields.io/github/issues-raw/jam82/ansible-role-git) ![Travis (.com) branch](https://img.shields.io/travis/com/jam82/ansible-role-git/main?label=travis) [![Molecule](https://github.com/jam82/ansible-role-git/actions/workflows/molecule.yml/badge.svg)](https://github.com/jam82/ansible-role-git/actions/workflows/molecule.yml)

**Ansible role for setting up git with libsecret(keyring) support.**

Support for `git-credential-libsecret` currently only works for:

- Archlinux
- Debian

A basic system configuration is deployed to `/etc/gitconfig`.
User configuration is done via dotfiles and not part of this role.

## Supported Platforms

| OS Family | Distribution  | Latest | Supported Version(s) | Comment |
|-----------|---------------|--------|----------------------|---------|
| Alpine    | Alpine        | :heavy_check_mark: | 3.12, 3.13 | |
| Archlinux | Archlinux     | :heavy_check_mark: | - | |
|           | Manjaro       | :heavy_check_mark: | - | |
| Debian    | Debian        | :heavy_check_mark: | 10, 11 | |
|           | Ubuntu        | :heavy_check_mark: | 18.04, 20.04 | |
| RedHat    | Almalinux     | :heavy_check_mark: | 7, 8 | |
|           | Amazonlinux   | :x: | - | not tested, image not working |
|           | Centos        | :heavy_check_mark: | 8 | |
|           | Fedora        | :heavy_check_mark: | 33, 34, Rawhide | |
|           | Oraclelinux   | :heavy_check_mark: | 7, 8 | |
| Suse      | OpenSuse Leap | :heavy_check_mark: | 15.1, 15.2, 15.3 | |
|           | Tumbleweed    | :heavy_check_mark: | - | |

## Requirements

Ansible 2.9 or higher.

## Variables

Variables and defaults for this role:

```yaml
# The role is disabled by default, so you do not get in trouble.
# Checked in tasks/main.yml.
git_enabled: false
```

## Dependencies

None.

## Example Playbook

```yaml
---
# role: ansible-role-git
# file: site.yml

- hosts: git
  vars:
    git_enabled: true
  roles:
    - role: ansible-role-git
```

## License and Author

- Author:: [jam82](https://github.com/jam82/)
- Copyright:: 2020, [jam82](https://github.com/jam82/)

Licensed under [MIT License](https://opensource.org/licenses/MIT).
See [LICENSE](https://github.com/jam82/ansible-role-git/blob/master/LICENSE) file in repository.

## References

- [ArchWiki - GNOME/Keyring](https://wiki.archlinux.org/index.php/GNOME/Keyring)
- [Stackoverflow](https://stackoverflow.com/questions/13385690/how-to-use-git-with-gnome-keyring-integration)
