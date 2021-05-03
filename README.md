# ansible-role-git

**Ansible role for setting up git with libsecret(keyring) support.**

Support for `git-credential-libsecret` currently only works for:

- Archlinux
- Debian

A basic system configuration is deployed to `/etc/gitconfig`.
User configuration is done via dotfiles and not part of this role.

## Supported Platforms

- Alpine
- AmazonLinux 2
- Archlinux
- CentOS
- Debian
- Fedora
- OpenSuse Leap, Tumbleweed
- OracleLinux
- Ubuntu

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

- hosts: git_servers
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
