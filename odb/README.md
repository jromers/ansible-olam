# Ansible playbooks for Oracle Linux Virtualization Manager using Oracle Database Templates

Automate deployment of Oracle Database Single Instance and RAC cluster using the Ansible playbooks and Database Templates for Oracle Linux KVM managed by Oracle Linux Virtualization Manager. 

Modules from the [ovirt.ovirt Ansible collection](https://docs.ansible.com/ansible/latest/collections/ovirt/ovirt/index.html) are used and should be downloaded before using the playbooks. Read the collection documentation page for additional explanation or for extending the functionality of the playbooks.

The playbooks are tested with Ansible CLI commands on Oracle Linux and with Oracle Linux Automation Manager, but should run on other Ansible platforms too. The remote host is the OLVM engine to run the tasks of the playbook, make sure the ``python3-ovirt-engine-sdk4`` package exists on the engine host.

## How to setup a Oracle Database template in Oracle Linux Virtualization Manager

Oracle provides templates (pre-configured, pre-built virtual machines) that are fully ready to deploy to a virtualized Oracle Linux Virtualization Manager environment. A typical template includes a guest operating system, database software, and configuration needed for deployment. 

An Oracle Database template for Oracle Linux KVM/OLVM consists of single OVA file (inside a zip file) that contains two virtual disks. More background information on the template for Oracle KVM/OLVM environments and where to download from is available on [this download location at oracle.com](https://www.oracle.com/ng/database/technologies/rac/vm-db-templates.html) (version March 2025).

1. Download the desired version of the Database Template to a staging area on one of the KVM Server hosts
2. From Oracle Linux Virtualization Manager UI, import the staged Database Template.
3. Optionally edit the properties for the imported template (OVA file), such as
   * Memory/CPU
   * Additional/Correct virtual vNICs mappings

## How to use the playbooks

The following deployment scenarios can be used with the playbooks:

* Single instance with IP address from DHCP
* Single instance with static IP address
* Single instance with static IP address, with High Availabity and Nx disks for ASM
* RAC cluster with Nx disks (N is a minimum of 5) for ASM

### Ansible CLI

First step is the configuration of the playbook variables which are configured in ``default_vars.yml`` file. Variables are required to configure your infrastructure settings for the OLVM server, VM and Database configuration and cloud-init. See below table for explanation of the variables. 

The playbooks can be used on Oracle Linux 8 like this (change to your server names, passwords and ip addresses):

```console
$ git clone https://github.com/jromers/ansible-olam.git
$ cd ansible-olam/odb
$ ansible-galaxy collection install -r requirements.yml
$ cat << EOF >> hosts.ini
[olvm]
olvm-engine.demo.local
EOF
$ export "OVIRT_URL=https://olvm-engine.demo.local/ovirt-engine/api"
$ export "OVIRT_USERNAME=admin@internal"
$ export "OVIRT_PASSWORD=CHANGE_ME"
```
**Note**: replace FQDN with the fully qualified domain name of your OLVM engine (in this example olvm-engine.demo.local)

#### Single Instance (HA is optional):

After configuration is defined in the ``default_vars.yml`` file, the single instance Oracle Database will be deployed with the ``olvm_odb_si.yml`` playbook:

```console
$ cp default_vars-si-example.yml default_vars.yml
$ vi default_vars.yml
...
Provide the values for the variables
...
...
$ ansible-playbook -i hosts.ini \
    -u <remote-ansible-user> --key-file ~/.ssh/<private-key> \
    -e "vm_name=odb-si" -e "vm_ip_address=192.168.1.25" \
    olvm_odb_si.yml
```
**Note**: the ``vm_ip_address`` may be omitted, in that case the VM will be provisioned for DHCP

#### RAC cluster:

Configiration of the variables for the RAC cluster is somewhat more complex, each node in the RAC cluster has multiple IP addresses with corresponding hostnames. In order to provide these configuration details for each node, an Ansible dictionary is used and defined in the ``default_vars.yml`` file.

Here's an example of a three-node RAC cluster Ansible dictionary:

```console
rac_nodes:
  1:
    vm_name: "odb-rac1"
    vm_ip_address: "192.168.1.71"
    vm_name_priv: "odb-rac1-priv"
    vm_ip_address_priv: "192.168.2.71"
    vm_name_vip: "odb-rac1-vip"
    vm_ip_address_vip: "192.168.1.81"
  2:
    vm_name: "odb-rac2"
    vm_ip_address: "192.168.1.72"
    vm_name_priv: "odb-rac2-priv"
    vm_ip_address_priv: "192.168.2.72"
    vm_name_vip: "odb-rac2-vip"
    vm_ip_address_vip: "192.168.1.82"
  3:
    vm_name: "odb-rac3"
    vm_ip_address: "192.168.1.73"
    vm_name_priv: "odb-rac3-priv"
    vm_ip_address_priv: "192.168.2.73"
    vm_name_vip: "odb-rac3-vip"
    vm_ip_address_vip: "192.168.1.83"
```
**Note1**: To successfully deploy a RAC environment, DNS records for hostnames, scanname, and IPs described in the ``default_vars.yml`` file are required.

**Note2**: It is recommended to have the private IP address (``vm_ip_address_priv``) in a different subnet, the VM template contains a second NIC interface.

After configuration is defined in the ``default_vars.yml`` file, the RAC cluster will be deployed with the ``olvm_odb_rac.yml`` playbook:

```console
$ cp default_vars-rac-example.yml default_vars.yml
$ vi default_vars.yml
...
Provide the values for the variables
...
...
$ ansible-playbook -i hosts.ini \
    -u <remote-ansible-user> --key-file ~/.ssh/<private-key> \
    olvm_odb_rac.yml
```


### Oracle Linux Automation Manager

#### Project:
In Oracle Linux Automation Manager you can directly import the playbook repository from Github as Project. The top-level directory of the repository contains the requirements file to download the ovir.ovirt Ansible collection.

#### Inventory:
Create an inventory and add one host with the details of your OLVM server, this is the target host were you run the playbook. Make sure you have a Machine credential setup for this host so that Ansible can SSH to it (run the ping Module for this host). 

#### Credentials:
Besides the standard SSH credential to access the target host, an additional credential is required to use the ovirt modules in the playbooks. It's based on credential type ``Red Hat Virtualization`` and you need to fill in the OLVM FQDN, username, password and CA File. For example:

    Host (Authentication URL): 	https://olvm-engine.demo.local/ovirt-engine/api
    Username:			admin@internal
    Password:			CHANGE_ME

#### Templates:
Create a new job template and provide the following information:

    Inventory:		Select the inventory containing the OLVM host
    Project:		Select project from the Github repository
    Playbook:		Select playbook from Project, for example olvm_create_single_vm.yml
    Credentials:		Select Machine (SSH) credential and the Virtualization credentials
    Variables:		Enter the variables as used in the example default_vars.yml file

### Secure API connection

By default the API connection to the OLVM server is insecure, if you want to use a secure API connection then you need to define variable ``olvm_insecure`` and make sure the CA file is available (default location is ``/etc/pki/ovirt-engine/ca.pem``). You may use ``olvm_cafile`` to specify alternative location. 

    olvm_insecure: false
    olvm_cafile: /home/opc/ca.pem

The CA file can be downloaded from the main OLVM web portal or directly from the OLVM server, for example:

    $ scp root@olvm-engine.demo.local:/etc/pki/ovirt-engine/ca.pem /home/opc/ca.pem

## Variables used in the playbooks 

| Variable | Example value | Description |
| -------- | ------------- | ----------- |
| OVIRT_URL | https://olvm-fqdn/ovirt-engine/api | The API URL of the OLVM server
| OVIRT_USERNAME | admin@internal | The name of the API user, same as used for the Admin GUI login
| OVIRT_PASSWORD | CHANGE_ME | The password of the API user, same as used for the Admin GUI login
| vm_name | odb-si | Name of the VM (single instance), will also be used as hostname
| vm_ip_address | 192.168.1.25 | Static IP address of VM (single instance), if omitted DHCP will be used
| vm_ha | false | Will this be a single instance with or without High Availability,can be true or false. Ignored when DHCP is used
| rac_nodes | | Ansible dictionary to store RAC node specific hostname and network configuration details
| rac_name | crs64bitR2 | RAC Clustername
| rac_scanname | odb-rac-scan | RAC scan name
| rac_scan_ip_address | 192.168.1.86 | RAC scan IP Address
| asm_disks | asm0 | Ansible list with ASM disks names, such as asm0, asm1, asm3. With RAC minimal five ASM disks are recommended
| asm_disk_size | 10GiB |When High Availability, this will be the ASM disk size (all disks same size)
| olvm_cluster | Default | Name of the cluster, where VM should be created
| olvm_template | OLVM-OL8U10-19260DBRAC-KVM |Name of the Oracle DB template, which should be used to create VM
| olvm_storage_domain | VM-storage | The OLVM storage domain to deploy the VM
| vm_ram | 4096MiB | Amount of memory of the VM
| vm_cpu | 2 | Number of virtual CPUs sockets of the VM
| vm_timezone | Europe/Amsterdam | Timezone for VM, default is Etc/GMT
| vm_pubadap | eth0 | NIC interface in VM, default is eth0
| vm_privadap | eth1 | NIC interface in VM for the private network, default is eth1
| vm_dns | 192.168.1.3 | DNS server to be used for VM
| vm_dns_domain | demo.local | DNS domainto to be used for VM
| vm_gateway | 192.168.1.1 | Default gateway to be used for VM
| vm_netmask | 255.255.255.0 | Netmask to be used for VM for public NIC
| vm_netmask_priv | 255.255.255.0 | Netmask to be used for VM for private NIC
| vm_user_sshpubkey | "ssh-rsa AAAA...YOUR KEY HERE...hj8= " | SSH Public key for Ansible remote user
| olvm_insecure | false | By default ``true``, but define ``false`` in case you need secure API connection
| olvm_cafile | /home/opc/ca.pem | Location of CA file in case you wish alternative location


