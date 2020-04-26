# ansible-role-git

**Ansible role for setting up git with libsecret(keyring) support.**

## Supported Platforms

- Alpine 3.11
- Archlinux
- CentOS 8
- Debian 9, 10
- Fedora 31
- Ubuntu 18.04, 20.04

## Requirements

Ansible 2.9 or higher is recommended.

## Variables

Variables and defaults for this role:

```yaml
---
# role: ansible-role-git
# file: defaults/main.yml

# The role is disabled by default, so you do not get in trouble.
# Checked in tasks/main.yml which includes tasks.yml if enabled.
git_enabled: False

git_name: ""
git_email: ""
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
    git_enabled: True
    git_name: jam82
    git_email: jam82@yourdomain.tld
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
