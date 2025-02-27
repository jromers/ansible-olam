# Using Ansibel CLI to create instances in existing OCI VPN.

Very simple playbook to install OCI VM instances in an existing VCN and subnet. This was tested on MacOS with the OCI CLE installed, including the $HOME/.oci/config files configured.

## Pre-requisites
- Git is installed
- SSH client is installed and configured
- The ansible or ansible-core package is installed
- OCI CLI installed including $HOME/.oci/config file configured

## Setup virtual environment for Python/ansible/oci:

<pre><code>$ python3 -m venv venv
$ source venv/bin/activate
$ pip install -U oci
$ pip install ansible
$ source venv/bin/activate
$ ansible-galaxy collection install -f oracle.oci
</code></pre>

## Set default parameters:

Define the OCI variables and the ssh public key. You maybe also want to change the instance image (type of OS) or instance shape.

<pre><code>$ cp group_vars/all-example.yml group_vars/all.yml
$ vi group_vars/all.yml
</code></pre>

## Run the playbook:
<pre><code>$ ansible-playbook 
    -e "name=vm01" \
    -e "ip=10.0.1.102" \
    -e "public_ip=False"  create-one-instance.yml
</code></pre>

Or for multiple instances with inventory file:
<pre><code>$ cp inventory/hosts-example.ini inventory/hosts.ini
$ vi inventory/hosts.ini

Enter the instances name and hostvariables as provided

$ ansible-playbook -i inventory/hosts.ini create-multiple-instances.yml
</code></pre>



