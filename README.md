# Ansible Role: git

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-git)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-git)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-git)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-git/dev.yml?branch=dev&event=push&label=dev)](https://github.com/jomrr/ansible-role-git/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-git/main.yml?branch=main&event=push&label=main)](https://github.com/jomrr/ansible-role-git/actions/workflows/main.yml?query=branch%3Amain)

Install Git and manage system and per-user configuration, including SSH signing.

## Purpose

Install Git and its SSH client, deploy a validated system configuration, and
manage selected global settings as existing users. The role is idempotent.

## Scope

### Managed

- Git and SSH client packages, the complete /etc/gitconfig, and explicitly
  listed per-user global settings.
- The libsecret credential helper on Debian when git_keyring_enabled is true.
- SSH commit and tag signing for accounts in git_ssh_signing, with an explicitly
  supplied key string.

### Not Managed

- User accounts, home directories, repositories, private signing keys, SSH
  agents, and Secret Service sessions.
- Global settings omitted from git_user_config and all repository-local
  settings.

## Requirements

- Configured accounts and their home directories must already exist. The
  connection must support become to root and to those accounts.
- SSH signing requires Git 2.34 or later and OpenSSH 8.0 or later. The tested
  latest platform images provide these versions.
- Every SSH-signing account requires an explicit key string. A key:: public-key
  string requires the matching private key in an accessible SSH agent; an
  explicit key path must be readable by the account.
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

Complete /etc/gitconfig contents, mapped from Git configuration names to string
values.
Unlisted system settings are removed. The platform credential helper is appended
when enabled.
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

Default output redaction policy for global configuration entries.
Set no_log on individual entries containing credentials.

Default:

```yaml
git_config_no_log: false
```

### `git_user_config`

Type: `list`. Required: `false`.

Global Git settings applied with become as each named, existing user.
Unlisted settings are preserved. Git resolves the user global configuration
file.
Signing format, key and commit/tag policies belong in git_ssh_signing; other Git
settings, including gpg.ssh.allowedSignersFile, are accepted here.
Each setting has one writer. Do not mix add and replace-all for the same user
and name.

Default:

```yaml
git_user_config: []
```

### `git_sign_commits`

Type: `bool`. Required: `false`.

Default automatic commit-signing policy for accounts in git_ssh_signing.

Default:

```yaml
git_sign_commits: true
```

### `git_sign_tags`

Type: `bool`. Required: `false`.

Default automatic tag-signing policy for accounts in git_ssh_signing.

Default:

```yaml
git_sign_tags: true
```

### `git_ssh_signing`

Type: `list`. Required: `false`.

Configure SSH signing for existing accounts with a mandatory explicit
signing-key string.
The role never discovers keys or configures gpg.ssh.defaultKeyCommand.
Signing settings are managed exclusively here, rather than through
git_user_config or git_system_config.

Default:

```yaml
git_ssh_signing: []
```

## Managed Files

- `/etc/gitconfig` Fully managed, root-owned, mode 0644, syntax-validated before
  replacement, with module-provided backups.
- `~/.gitconfig or $XDG_CONFIG_HOME/git/config` Git chooses the global file for
  the become user; only explicitly listed keys are changed through git config.

## Check Mode

Supported, including first installation. System configuration changes are
predicted without writing files.

- When package installation is predicted to change the host, helper compilation
  and global settings are deferred because Git or its build prerequisites may
  not exist yet.
- With prerequisites installed, the make and git_config modules report their
  native change predictions.

## Service Behavior

No daemon restart or handler is needed; Git reads configuration when invoked.

## Security Notes

- System configuration is public. Mark global entries containing credentials
  with no_log: true, or set git_config_no_log for a configuration consisting of
  sensitive entries.
- The role references signing keys and never reads or distributes private-key
  contents.

## Operational Notes

- Set git_keyring_enabled: false to omit the managed Debian helper. Ubuntu, Red
  Hat and SUSE do not enable a helper by default.
- Debian builds the helper shipped with Git using community.general.make.
  Supported platforms are exactly the generator repository default platforms.
- No editor is forced by default. core.editor may be set to an editor already
  installed on the host.
- git_system_config accepts full Git names, including case-sensitive subsections
  such as `url.https://example.org/.insteadOf`. Quote boolean and numeric
  values.
- git_user_config entries require user and name. state defaults to present and
  requires a nonempty string value; state: absent removes the key. add_mode: add
  manages multiple values without replacing other values.
- Give each user/key one intended state; do not mix add and replace-all for the
  same key. Removing an entry stops managing it; use state: absent to remove its
  value.
- Native git config validates syntax but does not reject every conflicting
  semantic option. Functional tests exercise git diff as well as signed commits
  and tags.
- gpg.format, user.signingkey, commit.gpgsign and tag.gpgsign are owned by
  git_ssh_signing. Configure their policies there to guarantee an explicit
  signing key and avoid competing writers.

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

### Global settings and SSH signing

Configure an existing account with an explicit signing key.

```yaml
---
- name: GIT | Configure a developer account
  hosts: workstations
  gather_facts: true
  roles:
    - role: jomrr.git
      git_ssh_signing:
        - user: alice
          key: /home/alice/.ssh/id_ed25519
      git_user_config:
        - {user: alice, name: user.name, value: Alice Example}
        - {user: alice, name: user.email, value: alice@example.org}
        - {user: alice, name: init.defaultBranch, value: main}
        - user: alice
          name: gpg.ssh.allowedSignersFile
          value: /home/alice/.ssh/allowed_signers
        - {user: alice, name: alias.old, state: absent}
```

## References

- [Git configuration and SSH signing](https://git-scm.com/docs/git-config)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2019 Jonas Mauer.
