# Ansible Collection - fabos.docker

The collection provides playbooks for installing and uninstalling of docker on a target host (see "setup" role) and 
deploying and undeploying of container based on docker-compose files (see "use" role). The additional "inventory_helper"
role helps to create an inventory in the context of the FabOS Service Lifecycle Management (SLM).

# Issue

## Credential Helper
````shell
Error saving credentials: error storing credentials - err: exit status 1, out: `error getting credentials - err: exit status 1, out: `A specified logon session does not exist. It may already have been terminated.
````

Discussion:
https://github.com/docker/for-win/issues/9843#issuecomment-1420776498