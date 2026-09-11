# Ansible Role: git

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-git)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-git)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-git)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-git/dev.yml?branch=dev&event=push&label=dev)](https://github.com/jomrr/ansible-role-git/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-git/main.yml?branch=main&event=push&label=main)](https://github.com/jomrr/ansible-role-git/actions/workflows/main.yml?query=branch%3Amain)

Install Git and manage system and per-user configuration, including SSH signing.

## Purpose

Install Git and its SSH client, manage system settings with
community.general.git_config, and configure global identity and settings as
existing users. The role is idempotent.

## Scope

### Managed

- Git and SSH client packages, selected system settings, and global
  configuration for accounts in git_user_config.
- The libsecret credential helper on Debian when git_keyring_enabled is true.
- Required author name and email, additional settings, and explicit SSH-signing
  configuration for each managed account.

### Not Managed

- User accounts, home directories, repositories, private signing keys, SSH
  agents, and Secret Service sessions.
- Unlisted system and global settings, apart from the platform credential
  helper, and all repository-local settings.

## Requirements

- Configured accounts and their home directories must already exist. The
  connection must support become to root and to those accounts.
- Each git_user_config entry requires user (the operating-system account), name
  (the Git author name), and email.
- SSH signing requires Git 2.34 or later and OpenSSH 8.0 or later. The tested
  latest platform images provide these versions.
- Every SSH-signing account requires an explicit user.signingkey string in
  config. A key:: public-key string requires the matching private key in an
  accessible SSH agent; an explicit key path must be readable by the account.
- Signature verification requires an existing allowed-signers file and
  gpg.ssh.allowedSignersFile. Signing alone does not require that file.
- The credential helper requires an existing user Secret Service session to
  store or retrieve credentials.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
```

## Role Variables

### `git_keyring_enabled`

Type: `bool`. Required: `false`.

Enable the libsecret credential helper on Debian, built from the sources shipped
with Git.

Default:

```yaml
git_keyring_enabled: true
```

### `git_system_config`

Type: `dict`. Required: `false`.

Managed system Git settings, mapped from full configuration names to nonempty
string values or null for removal.
Unlisted settings are preserved. Explicit credential.helper overrides the
platform helper setting.
Values are public; credentials belong in per-user settings protected by no_log.

Default:

```yaml
git_system_config:
  core.excludesfile: ~/.gitignore
  core.autocrlf: input
  core.whitespace: trailing-space,space-before-tab,tab-in-indent,cr-at-eol
  color.branch: auto
  color.diff: auto
  color.interactive: auto
  color.status: auto
  push.default: simple
  alias.a: add .
  alias.c: commit
  alias.ca: commit -a
  alias.cm: commit -m
  alias.cam: commit -am
  alias.d: diff
  alias.dc: diff --cached
  alias.l: log --graph --pretty=format:"%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)
    %C(bold blue)<%an>%Creset" --abbrev-commit
  alias.p: push
```

### `git_config_no_log`

Type: `bool`. Required: `false`.

Default output redaction policy for global configuration tasks.
Set no_log on individual accounts whose configuration contains sensitive values.

Default:

```yaml
git_config_no_log: false
```

### `git_user_config`

Type: `list`. Required: `false`.

Global Git configuration applied with become as each named, existing user.
Each account requires its Git author name and email; list each account once.
Unlisted settings are preserved. Git resolves the user global configuration
file.

Default:

```yaml
git_user_config: []
```

## Managed Files

- `/etc/gitconfig` Selected keys are managed with community.general.git_config
  at system scope. Unlisted settings are preserved.
- `~/.gitconfig or $XDG_CONFIG_HOME/git/config` The same module manages global
  settings as their owner. Git chooses the configuration file for the become
  user.

## Check Mode

Supported, including first installation, through native module change
predictions.

- When package installation is predicted to change the host, helper compilation
  and configuration tasks are deferred because Git or its build prerequisites
  may not exist yet.
- With prerequisites installed, the make and git_config modules report their
  native change predictions.

## Service Behavior

No daemon restart or handler is needed; Git reads configuration when invoked.

## Security Notes

- System configuration is public. Set no_log: true for accounts whose config
  contains credentials, or use git_config_no_log for the default redaction
  policy.
- The role references signing keys and never reads or distributes private-key
  contents. It never discovers keys or sets gpg.ssh.defaultKeyCommand.

## Operational Notes

- Set git_keyring_enabled: false to omit the managed Debian helper. Ubuntu, Red
  Hat and SUSE do not enable a helper by default. An explicit credential.helper
  in git_system_config takes precedence.
- Debian builds the helper shipped with Git using community.general.make.
  Supported platforms are exactly the generator repository default platforms.
- No editor is forced by default. core.editor may be set to an editor already
  installed on the host.
- git_system_config and each account config accept full Git names, including
  case-sensitive subsections such as `url.https://example.org/.insteadOf`. Quote
  boolean and numeric values.
- Each account name and email set user.name and user.email and take precedence
  over config. An empty git_user_config list leaves user identities unmanaged.
- String values replace all existing values for a key; null removes the key.
  Removing a key from the mapping stops managing it and leaves its current value
  intact.
- SSH signing uses ordinary config keys: gpg.format, user.signingkey,
  commit.gpgsign, and tag.gpgsign. The role requires an explicit key when
  gpg.format is ssh; it does not select a key or enable signing implicitly.
- Native git config validates configuration syntax. Functional tests exercise
  git diff, effective identities, preserved settings, and signed commits and
  tags.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### System defaults

Install Git with the default system settings.

```yaml
---
- name: GIT | Configure Git
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.git
```

### Global identity

Set the required Git identity for an existing account.

```yaml
---
- name: GIT | Configure a developer identity
  hosts: workstations
  gather_facts: true
  roles:
    - role: jomrr.git
      git_user_config:
        - user: alice
          name: Alice Example
          email: alice@example.org
```

### Global settings and SSH signing

Configure an existing account with an explicit signing key.

```yaml
---
- name: GIT | Configure a developer account
  hosts: workstations
  gather_facts: true
  roles:
    - role: jomrr.git
      git_user_config:
        - user: alice
          name: Alice Example
          email: alice@example.org
          config:
            init.defaultBranch: main
            gpg.format: ssh
            user.signingkey: /home/alice/.ssh/id_ed25519
            commit.gpgsign: "true"
            tag.gpgsign: "true"
            gpg.ssh.allowedSignersFile: /home/alice/.ssh/allowed_signers
            alias.old: null
          no_log: false
```

## References

- [Git configuration and SSH signing](https://git-scm.com/docs/git-config)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2019-2026 Jonas Mauer.
