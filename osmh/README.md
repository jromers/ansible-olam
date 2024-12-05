# OSMH - Ansible Playbook for registering OSMH non-OCI Oracle Linux instances

This playbook registers an Oracle Linux server with the OSMH management service. The playbooks support Oracle Linux servers running in on-premise locations as well as in non-OCI clouds such as AWS and Azure. To register Oracle Linux instances in Oracle's OCI cloud a [different procedure](https://docs.oracle.com/en-us/iaas/osmh/doc/register-oci-instance.htm#register-oci-instance) is required.

The playbooks uses modules from the [oracle.oci Ansible collection](https://docs.oracle.com/en-us/iaas/tools/oci-ansible-collection/5.3.0/index.html) which should be downloaded before using the playbooks. Read the collection documentation page for additional explanation or for extending the functionality of the playbooks.

# How to use the playbooks

## Ansible CLI

First step is the configuration of the playbook variables which are mostly configured in ``default_vars.yml`` file. Variables may be used in the command line when not configured in the default variables file. Variables are required to configure your infrastructure settings such as your local Management Station. See below table for explanation of the variables. 

Please, make sure you have installed the CLI config file in the default location ``~/oci/config`` and API keys for OCI authentication.

The playbooks can be used like this:

    $ git clone https://github.com/jromers/ansible-olam.git
    $ cd ansible-olam/osmh
    $ cat << EOF > requirements.yml 
    ---
    collections:
      - name: oracle.oci
      - name: community.general
    EOF
    $ cat << EOF > hosts.ini
    # Define group "instances", for instances to register. For example:
    [instances]
    192.168.1.11
    EOF
    $ ansible-galaxy collection install requirements.yml
    $ vi default_vars.yml

    $ ansible-playbook -i hosts.ini -u opc --key-file ~/.ssh/id_rsa \
          osmh_register_non-oci_instance.yml
    or

    $ ansible-playbook -i hosts.ini -u opc --key-file ~/.ssh/id_rsa \
          -e "reg_profile=ol9-clients" osmh_register_non-oci_instance.yml


## Oracle Linux Automation Manager

#### Project:
In Oracle Linux Automation Manager you can directly import the playbook repository from Github as Project. The top-level directory of the repository contains the requirements file to download the oracle.oci ansible collection.

#### Inventory:
Create an inventory and add the Hosts, also create the Group "instances" and add the hostnames to the group. Make sure you have a Machine credential setup for the hosts so that ansible can SSH to it (run the ping Module for the hosts to test the connection).

#### Credentials:
Besides the standard SSH credential to access the target host, you need to add the credentials for OCI. In the credentials section there is a credential typ "Oracle Cloud Infrastructure" where you need to enter your ``User OCID``, ``Fingerprint``, ``Tenant OCID``, ``Region`` and ``Private User Key``. This is the same information that is stored in the ``~/.oci/config`` file mentioned above.

See [this Tutorial with an example](https://docs.oracle.com/en/learn/olam-oci-collection/#create-oci-credentials) on how to create the Oracle OCI credential in Oracle Linux Automation Manager.

#### Templates:
Create a new job template and provide the following information:

    Inventory:		Select the inventory containing the instances
    Project:		Select project from the Github repository
    Playbook:		Select playbook from Project, osmh_register_non-oci_instance.yml
    Credentials:		Select Machine (SSH) credential and the OCI credentials
    Variables:		Enter the variables as used in the example default_vars.yml file

# Variables used in the playbooks 

| Variable | Example value | Description |
| -------- | ------------- | ----------- |
| mgmt_agent_install_key_name | my-key |  Management Agent Cloud Service (MACS) install key for the compartment where you want the instances to reside.
| reg_profile | ol9-clients | The profile key for the content associated with a software source, group, or lifecycle environment
| gateway_server_host | mgmt-host1 | Local management station server name to mirror and distribute software sources and proxy for all OSMH traffic
| gateway_server_port | 5001 | Local management station proxy server port.
| compartment_ocid | ocid1.compartment.oc....aq5 | the OCI ID of the compartment where you want the instances to reside
| osmh_scratch_parent | /tmp | Parent directory for temporary storage of software-rpm and config files
| osmh_scratch_dir | osmh_scratch | Directory for temporary storage of software-rpm and config files
| osmh_scratch_path | /tmp/osmh_scratch | Full path of temporary storage


