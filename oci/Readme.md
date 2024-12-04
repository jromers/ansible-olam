# Using Ansibel CLI to create instances in existing OCI VPN.

This was tested on MacOS with the OCI CLE installed, including the $HOME/.oci/config files configured.

## Setup virtual environment for Python/ansible/oci:

<pre><code>$ python3 -m venv venv
$ source venv/bin/activate
$ pip install -U oci
$ pip install ansible
$ source venv/bin/activate
$ ansible-galaxy collection install -f oracle.oci
</code></pre>

## Set default parameters:

<pre><code>$ export SAMPLE_PUBLIC_SSH_KEY=`cat ~/.ssh/id_rsa.pub`           # or any other pub SSH keyfile
$ export SAMPLE_COMPARTMENT_OCID="ocid1.compartment.oc1..aaa…..n3j6q"
$ export SAMPLE_SUBNET_OCID="ocid1.subnet.oc1.uk-london-1.aaa…..mp3q"
$ export SAMPLE_AD_NAME="MrTj:UK-LONDON-1-AD-3"

$ vi default_vars.yml

Change, or add the instances you want to build.
    instance_name: "oci-dev02"		# name of the instance
    instance_ip: "10.0.1.102"		# provide an IP in the subnet (otherwise it’s random IP)
    public_ip: false			# use false if private subnet, true when public subnet
</code></pre>

You maybe also want to change the instance image (type of OS) or instance shape.

## Run the playbook:
<pre><code>$ ansible-playbook create-oci-instances.yml
</code></pre>



