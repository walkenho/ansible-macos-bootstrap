# Mac Dev Environment Setup

## Base setup

### Install Homebrew

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo >> ~/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv zsh)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

### Install casks that cannot be installed via Ansible

Some casks (MacTeX, Docker Desktop) need an interactive sudo prompt Ansible can't supply. Install these manually first:

```
brew install --cask mactex
brew install --cask docker-desktop
```

## Install playbooks

### Prepare Ansible

**Option 1: Via Homebrew**

If you are not interested in working on your own Ansible playbooks, just install Ansible via Homebrew:
```
brew install ansible
ansible-galaxy collection install community.general
```

The playbooks will take care of the rest.

**Option 2: If you want to develop your own Ansible flows**

If you want to develop your own Ansible playbooks, it is a good idea to install ansible-lint in addition to ansible.

In this case, the better option is to use `uv tool install ansible-dev-tools`.

<details>
<summary>Why not install both via Homebrew?</summary>

Ansible as well as Ansible-lint can both be installed via Homebrew. However, if you install both via Homebrew, they are completely separate. This leaves you with the choice between:
1. Install the community.general collection twice — once into the ansible formula, once into the ansible-lint formula. However, this means that if you update one of them, then the collections will be out of sync, leaving you using one but linting against another.
2. Hard-code ansible-lint to point to the ansible-galaxy collection in the ansible formula. However, since the link hardcodes version constraints, this will break if you ever update ansible to a newer version.

Neither of these is ideal, which is why I accept the inconvenience of installing uv outside of ansible first, then using uv tool to install ansible-dev-tools, which packages both functionalities into one.

</details>

**Step 1: Install uv**

The best way to install Ansible and Ansible-lint that I have found is via `uv tool install ansible-dev-tools`. To set this up, we need to first install uv.

```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Enable shell autocompletion for uv commands (for zsh):

```
grep -qxF 'autoload -Uz compinit && compinit' ~/.zshrc || echo 'autoload -Uz compinit && compinit' >> ~/.zshrc
grep -qxF 'eval "$(uv generate-shell-completion zsh)"' ~/.zshrc || echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc
```

(To preserve idempotency, each line first checks whether it's already in the config, and only appends it if not.)

**Step 2: Install Ansible**

```
uv tool install --with-executables-from ansible-core,ansible-lint ansible-dev-tools
ansible-galaxy collection install community.general
```

And this sets you up to run the playbooks. Note that the first playbook will install uv if it is not yet installed, but since we have just installed it, it will just skip this step.

### Run playbooks

```
ansible-playbook 01_install_cli_development_tools.yml
ansible-playbook 02_install_and_setup_vscode.yml
ansible-playbook 03_install_productivity_tools.yml
ansible-playbook 04_install_communication_tools.yml
ansible-playbook 05_install_office_software.yml
ansible-playbook 06_check_security_settings.yml
ansible-playbook 07_configure_personal_settings.yml
```

Playbook 06 only needs a sudo password if the firewall is currently off (it enables it in that case). If that task fails asking for a password, rerun with `-K`:

```
ansible-playbook -K 06_check_security_settings.yml
```

Or run everything at once:

```
ansible-playbook 00_run_all.yml
```

- `01_install_cli_development_tools.yml` — installs python cli dev tools.
- `02_install_and_setup_vscode.yml` — installs VS Code and extensions.
- `03_install_productivity_tools.yml` — installs productivity tools.
- `04_install_communication_tools.yml` — installs communication tools.
- `05_install_office_software.yml` — installs office software.
- `06_check_security_settings.yml` — checks/enables the firewall, reports FileVault status.
- `07_configure_personal_settings.yml` — configures git identity and other post-install settings.

**Note**: Splitting tasks into separate playbooks is effectively a manual, coarse-grained version of Ansible's `tags` feature. For the moment, I am keeping the split structure for simplicity, but might change this in the future.
