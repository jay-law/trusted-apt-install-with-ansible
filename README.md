
# Overview

As infrastructure automation continues to grow, it's important to keep the attack surface minimized.  This tutorial focuses on verifying install packages and repos for Ubuntu using Ansible without the use of `apt-key`.

Below two applications are installed locally:  Docker and Mullvad VPN.  Each represents a different scenario for verifying packages.  The Docker install verifies an apt repo (a publicly available repo) and the Mullvad install method verifies an individual package.  The Docker method is more popular but both scenarios can be adapted to installing most modern apps.

## apt-key

This work started as a way to find an `apt-key` alternative as it is, or soon will be, deprecated.  Thankfully most of the installation documentation for Debian distributions have been updated and no longer reference `apt-key`.

[Digital Ocean has a great article on handling the move away from `apt-key.`](https://www.digitalocean.com/community/tutorials/how-to-handle-apt-key-and-add-apt-repository-deprecation-using-gpg-to-add-external-repositories-on-ubuntu-22-04)

## GPG, PGP, and OpenPGP

Like so many before me, I optimistically dove into the world of pretty good encryption standards (GPG, PGP, and OpenPGP) with the intent of enriching this tutorial.  And like the others, I too will leave the details to the experts.

If you are feeling overly optimistic today, the "Other" section at the bottom has some useful (hopefully) information for drilling into the nitty-gritty

**Note** - Ansible has no builtin module for `gpg` so `ansible.builtin.command` is used.

## System

Ansible is ran on the localhost host with the following:
- Ansible 2.14.5
- Python 3.10.6
- Ubuntu 22.04.2 
- Localhost as the only host

Facts and logs are stored in `temp/` for easy reference.

# Examples

Comments have been added for most tasks (`roles/[app]/tasks/main.yml`) that provide a description or a rough representation of the CLI command included in the install documentation.  

## Docker

Follow along with the official [install documentation for Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

Docker publishes packages to their own repo at https://download.docker.com/.  The Ubuntu repo (https://download.docker.com/linux/ubuntu/), along with some other information, can be added to `/etc/apt/sources.list.d/docker.list`.  This will allow `apt` to search and install packages from Docker's repo.  

This example focuses on ensuring the public repo is verified against the public gpg key (https://download.docker.com/linux/ubuntu/gpg).  Verification is provided by the `signed-by` option in `docker.list`.

```bash
# Install docker
ansible-playbook trusted-apt.yml --tag docker_install --ask-become

# Uninstall docker
ansible-playbook trusted-apt.yml --tag docker_uninstall --ask-become
```

## Mullvad

Follow along with the official [install documentation for Linux](https://mullvad.net/en/help/install-mullvad-app-linux/).

Mullvad package installs differently from Docker in that no public repo is made available.  Only the latest package is made available although older versions are available from their GitHub page.  Updates are manual and involve downloading the latest deb package, along with the corresponding key, then manually installing.

The process requires more manual work but allows for greater control.

This example focuses on ensuring a single package is verified against the public gpg key (https://mullvad.net/media/mullvad-code-signing.asc).

### Pre-Install Steps

As per https://mullvad.net/en/help/verifying-signatures/, the code signing key needs to be imported and trusted.  This only needs to be performed once as the automation does not add/remove/edit trusts or keys.

```bash
# Download key.  Ensure temp/ exists
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
```

### Install/Uninstall

```bash
# install mullvad
ansible-playbook trusted-apt.yml --tag mullvad_install --ask-become

# uninstall mullvad
ansible-playbook trusted-apt.yml --tag mullvad_uninstall --ask-become
```

# Other

## Useful GPG Commands

```bash
# Inspect Mullvad keys
gpg --list-packets temp/MullvadVPN-2023.3_amd64.deb.asc
gpg --list-packets temp/mullvad-code-signing.asc

# Inspect Docker keys
gpg --list-packets temp/docker.gpg
gpg --list-packets temp/docker.pgp

# Delete the Mullvad key 
gpg --delete-key A1198702FC3E0A09A9AE5B75D5A1D4F266DE8DDF
```