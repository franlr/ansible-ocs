# README - Vagrant test

```bash

# launch VM for testing
vagrant up

# install Agent
vagrant provision --provision-with ansible-provision

# verify idempotency
vagrant provision --provision-with ansible-provision
```
