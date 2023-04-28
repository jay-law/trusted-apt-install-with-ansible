
# Lint

```bash
ansible-lint
```

# Apps

## Docker

```bash
# install docker
ansible-playbook trusted-app.yml --tag docker_install --ask-become

# uninstall docker
ansible-playbook trusted-app.yml --tag docker_uninstall --ask-become
```

## Mullvad

### Pre-Install

As per https://mullvad.net/en/help/verifying-signatures/, the code signing key needs to be imported and trusted.

```bash
# Download key (PGP format as seen in file)
wget https://mullvad.net/media/mullvad-code-signing.asc -P temp/

# Import key
gpg --import temp/mullvad-code-signing.asc

# Set trust level - interactive
gpg --edit-key A1198702FC3E0A09A9AE5B75D5A1D4F266DE8DDF
gpg> trust
gpg> 5
gpg> y
gpg> q

# Set trust level - one-liner
echo -e "5\ny\n" | gpg --command-fd 0 --edit-key A1198702FC3E0A09A9AE5B75D5A1D4F266DE8DDF trust

# Verify after running mullvad_install
gpg --verify temp/MullvadVPN-*.deb.asc

# Delete key.  Not required
gpg --delete-key A1198702FC3E0A09A9AE5B75D5A1D4F266DE8DDF
```

### Install/Uninstall

```bash
# install mullvad
ansible-playbook trusted-app.yml --tag mullvad_install --ask-become

# uninstall mullvad
ansible-playbook trusted-app.yml --tag mullvad_uninstall --ask-become
```