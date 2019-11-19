# ansible-role-git

Ansible role for setting up git with gnome keyring support.

## Supported Platforms

* Arch Linux
* Ubuntu 18.04

## Requirements

Ansible 2.7 or higher is recommended.

## Variables

Variables for this

| variable | default value in defaults/main.yml | description |
| -------- | ---------------------------------- | ----------- |
| git_enabled | False | determine whether role is enabled (True) or not (False) |
| git_name | "" | name in gitconfig |
| git_email | "" | email in gitconfig |

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
    git_name: Jonas Mauer
    git_email: jam@example.com
  roles:
    - role: ansible-role-git
```

## License and Author

- Author:: Jonas Mauer (<jam@kabelmail.net>)
- Copyright:: 2019, Jonas Mauer

Licensed under MIT License;
See LICENSE file in repository.

## References

[Stackoverflow](https://stackoverflow.com/questions/13385690/how-to-use-git-with-gnome-keyring-integration)