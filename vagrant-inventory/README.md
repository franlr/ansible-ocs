# Test and develop with Vagrant + Virtualbox

You need Vagrant and Virtualbox installed on your workstation.

The `Vagrantfile` uses the Ubuntu 20.04 box.  You can change this if you need to test on another operating system,
but you need to change accordingly the provisioning section in the `Vagrantfile`.

To make a VM ready to work, in the `Vagrantfile` directory, type:

```bash
# launch VM for testing
vagrant up
```

If you want to install the role on the VM, do:

```bash
# install Agent
vagrant provision --provision-with ansible-provision

# verify idempotency
vagrant provision --provision-with ansible-provision
```

That's all folks!
