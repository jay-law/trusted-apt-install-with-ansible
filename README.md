
```bash
# uninstall docker
ansible-playbook trusted-app.yml --tag docker_uninstall --ask-become

# install docker
ansible-playbook trusted-app.yml --tag docker_install --ask-become

# /etc/apt/keyrings/docker.gpg
# /etc/apt/sources.list.d/docker.list

```