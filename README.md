
# Lint

```bash
ansible-lint
```

# Execute

## Docker

```bash
# install docker
ansible-playbook trusted-app.yml --tag docker_install --ask-become

# uninstall docker
ansible-playbook trusted-app.yml --tag docker_uninstall --ask-become
```
