# Grafana configuration manager

This module applies all configuration required for Graphite and Grafana

## Dependencies

- python3 (2 is also OK, see below notes)
- pip3 install ansible
- azure credentials for azure_rm to use, file located at: `~/.azure/credentials`

Note for multiple python env users:

- This script is supposed to support both python2 and python3 and easiest/safest way is to use virtualenv
- Probably you don't want to use virtualenv because you use ansible a lot besides this repo, in which case, you need to make sure the `pip install ansible` part is done to the python that is `/usr/bin/env python`
- In case you are using python2, updating the first line of the `azure_rm.py` to `/usr/bin/env python`, and use `pip install ansible`, make sure it's installed to the python that is `/usr/bin/env python`.
- Regarding all team members use Mac in this project, and Mac OS comes with python2, it is recommended to install another python(2/3), instead of using system default python. Do this via brew is the easiest way
- If you use brew and python2, you need to make sure your `/usr/local/bin/` is the first in `$PATH` so that `/usr/bin/env python` can point to it correctly, instead of still to the system default python.

An example of file `~/.azure/credentials`:

```
[default]
subscription_id=xxx-xxx-yyy-zzz
client_id=xxx-111-xxx-yyy
tenant=xxx-yyy-zzz-111
secret=aaabc+/=asdf
```

Note on the credentials file: the `secret` in the config file may contain special characters like `+` or `/`. Don't use single/double quote around the value of secret, this is not supported by the dynamic inventory script.

## Test

After all the dependencies are set up correctly, run the following test:

```
ansible -i azure_rm.py dev-grafana -m ping
```

This will trigger ansible ping module on VMs that is in `dev-grafana` resrouce group. It won't return successfully, because ping is not allowed by ACLs by default, but you will be able to see the IP of the VMs and ansible is trying to send a ping to those hosts.

## Deploy Grafana

Make sure you already executed the Terraform script, so that:

- a VM is created with ansible user and SSH private/public key login is setup
- the VM has tag `grafanadev` (or `grafanaprod` for Prod)
- the VM is in the resource group `dev-grafana` (or `monitoring` for Prod)

Then execute:

```
export AZURE_TAGS=grafanadev
ansible-playbook -i azure_rm.py main.yml --extra-vars "grafana_new_pwd=setMe" -v
```

This will deploy grafana onto the VM instance that has tag `grafanadev`.

