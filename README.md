

[![Build Status](https://travis-ci.org/terra-farm/terraform-provider-virtualbox.svg?branch=master)](https://travis-ci.org/terra-farm/terraform-provider-virtualbox)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fterra-farm%2Fterraform-provider-virtualbox.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fterra-farm%2Fterraform-provider-virtualbox?ref=badge_shield)

# Part 1
Linux OS Setup using VirtualBox provider for Terraform. Relevant script files are attached to the repository.

# VirtualBox provider for Terraform

Inspired by [terraform-provider-vix](https://github.com/hooklift/terraform-provider-vix)

Donated to the `terra-farm` group by [`ccll`](https://github.com/ccll)

Published documentation is located on the [Terra-Farm website](https://terra-farm.github.io/provider-virtualbox/).

# How to install

1. go get github.com/terra-farm/terraform-provider-virtualbox

# How to build from source

1. git clone https://github.com/terra-farm/terraform-provider-virtualbox
1. cd terraform-provider-virtualbox
1. dep ensure
1. go build
1. mv terraform-provider-virtualbox examples/
1. cd examples/
1. terraform init
1. terraform plan
1. terraform apply

# Resources

## "virtualbox_vm"

### Schema

- `name`, string, required: The name of the virtual machine.
- `image`, string, required: The place  of the image file(archive or vagrant box).
- `url`, string, optional, default not set: The url for downloaded vagrant box from external resource (ex. [Ubuntu Vagrant box](https://atlas.hashicorp.com/ubuntu/boxes/trusty64/versions/14.04/providers/virtualbox.box])) . If not set using `image` variable.
- `cpus`, int, optional, default=2: The number of CPUs.
- `memory`, string, optional, default="512mib": The size of memory, allow human friendly units like 'MB', 'MiB'.
- `user_data`, string, optional, default="": User defined data.
- `status`, string, optional, default="running": The status of the VM, allowed values: 'poweroff', 'running'. This value will be updated at runtime to reflect the real status of the VM, and you can also specify it explicitly in config to manually control the status of the VM. This value defaults to 'running', so `terraform apply` will always try to keep the VM running if not specified otherwise.
- `network_adapter`, list: The network adapters in the VM, you can have up to 4 adapters.
  - `.#.type`, string, requried: The type of the network, allowed values: 'nat', 'bridged', 'hostonly', 'internal', 'generic'.
  - `.#.device`, string, optional, default="IntelPro1000MTServer": The model of the virtual hardware device, allowed values: 'PCIII', 'FASTIII', 'IntelPro1000MTDesktop', 'IntelPro1000TServer', 'IntelPro1000MTServer'.
  - `.#.host_interface`, string, optional: Some network type (hostonly, bridged, etc) must bind to a host interface to work properly, use this field to specify the name of the host interface you like to bind to (like 'en0', 'eth1', 'wlan', etc). This should get an improvement, see [TODO](#todo) section below.
  - `.#.status`, string, computed: The status of the network adapter, possible values: 'up', 'down'.
  - `.#.mac_address`, string, computed: The MAC address of the adapter, this is generated by VirtualBox.
  - `.#.ipv4_address`, string, computed: The IPv4 address assigned to the adapter.
  - `.#.ipv4_address_available`, string, computed: Wheather or not an IPv4 address is actaully assigned to the adapter, possible values: "yes", "no".
- `optical_disks`, list: The iso image to attach.

### Network adapter types

- [x] NAT
- [x] bridged

# Example

```hcl
resource "virtualbox_vm" "node" {
    count = 2
    name = "${format("node-%02d", count.index+1)}"

    image = "~/ubuntu-15.04.tar.xz"
    cpus = 2
    memory = "512mib"

    network_adapter {
        type = "nat"
    }

    network_adapter {
        type = "bridged"
        host_interface = "en0"
    }

    optical_disks = ["./cloudinit.iso"]
}

output "IPAddr" {
    # Get the IPv4 address of the bridged adapter (the 2nd one) on 'node-02'
    value = "${element(virtualbox_vm.node.*.network_adapter.1.ipv4_address, 1)}"
}

```

# Limitations

- Experimental provider!

# Example images

- [ubuntu-15.04](https://github.com/ccll/terraform-provider-virtualbox-images/releases/tag/ubuntu-15.04)

- [Ubuntu Vagrant box](https://vagrantcloud.com/ubuntu/boxes/trusty64/versions/20180206.0.0/providers/virtualbox.box)

# TODO

- [x] Optimize resourceVMUpdate(), eliminate unneccessary restarts of VM.
- [x] Auto download image from remote url.
- [ ] Validate downloaded image against checksum.
- [x] Download the same image only once (based on checksum).
- [ ] Re-download corrupted image (based on checksum).



# Part 2: 
Prometheus Setup using Ansible. Relevant playbook files are attached to the repository.

# Prometheus Setup using Ansible

In this project, we are configurating prometheus with blackbox_exporter on one instance and node_exporter on another instance using ansible.

## Getting Started

Step 1: Update ip address of both instances in inventory file.

Step 2: Run ansible command to setup prometheus server with node exporter

Ansible command: ansible-playbook playbook.yml


# Part 3: 
Grafana Setup using Ansible. "grafana-playbook.yml" files attached to the repository.

Step 1: Update ip address of both instances in inventory file.

Step 2: Run ansible command to setup Grafana server.

Ansible command: grafana-playbook.yml


# Part 4: 
Ansible - Graphite: Scalable Realtime Graphing Setup. Relevant playbook files are attached to the repository.

Ansible - Graphite: Scalable Realtime Graphing
------------------
This Ansible playbook will install graphite via pip in its default location '/opt/graphite'


To run this playbook.

1. clone the repo
2. modify the `hosts` file to reflect the server you want to install Graphite on
3. ignore ssh's known hosts file
  * `export ANSIBLE_HOST_KEY_CHECKING=False`
4. run `ansible-playbook -i hosts playbook.yml`
  * You can modify the user Ansible uses by adding extra-vars `ansible-playbook -i hosts playbook.yml --extra-vars ssh_user=ec2-user`
5. In ~4 minutes you should have a running Graphite server!



What you need to change
-----------------------
## Secrets
Change `graphite_secret_key` under `roles/graphite/defaults/main.yml` to something unique for your graphite instance!

## Django
Change `graphite_django_admin_media` under `roles/graphite/defaults/main.yml` to the path of your django admin media
```
    # XXX In order for the django admin site media to work you
    # must change @DJANGO_ROOT@ to be the path to your django
    # installation, which is probably something like:

    Alias /media/ "/usr/local/lib/python2.7/dist-packages/django/contrib/admin/static/admin/"
    # or 
    #Alias /media/ "/usr/lib/python2.6/site-packages/django/contrib/admin/media/"
```

## Carbon Storage and Aggregation
It has been setup according to [this page](https://github.com/etsy/statsd/blob/master/docs/graphite.md) and might not do what you expect to with your data


Testing
--------

### Vagrant
Simply clone this repo and make sure you have Vagrant + Virtual Box installed and...
  1. vagrant up
  2. visit http://192.168.111.222/
  3. :-) 

Vagrant is using Ubuntu 14.04 Trusty Tahr for it's OS.


### Digital Ocean
I've tested this playbook with Digital Ocean VM's with a few different flavor of OS's.

  * CentOS 6.5
  * Ubuntu 14.04 Trusty Tahr

### AWS EC2
  * Amazon Linux



TODO
----
* Add Django superuser creation `python manage.py createsuperuser`.


Known Issues
------------
* If you are seeing "DatabaseError: database is locked" in your graphite logs, restarting apache may fix the issue for you. 
* SELinux needs to be disabled


Resources
---------
* http://graphite.wikidot.com/quickstart-guide


## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fterra-farm%2Fterraform-provider-virtualbox.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fterra-farm%2Fterraform-provider-virtualbox?ref=badge_large)
