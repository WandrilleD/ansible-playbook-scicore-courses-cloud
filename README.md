# Cloud playbooks

Ansible playbooks to boot resources in the cloud for the sciCORE courses

## Short description on how these playbooks work

Inside the [ansible](ansible) folder you will find different playbooks for the different scicore courses.
Playbooks with same name prefix belong to the same course. e.g. these are the playbooks to boot a slurm cluster

```
slurm_cluster_boot_openstack.yml
slurm_cluster_configure_nfs_server.yml
slurm_cluster_configure_slurm_daemons.yml
slurm_cluster_configure_user_accounts.yml
```

The first playbook which includes `boot` in the playbook name is the first one to execute. This playbook does:

  * Create all the required resources in the cloud. This includes ssh-keys, firewall rules, disk volumes and servers
  * Add all the booted servers to the ansible in-memory inventory.
  * Each server will be added to one or multiple groups in the ansible in-memory inventory based on their role.
  * Now that all the servers are booted in the cloud and added to the ansible in-memory inventory we have two options:
    * Option 1: The `boot` playbook automatically executes a second playbook which configures the servers
    * Option 2 for more complex setups e.g. slurm cluster:
      * Based on the ansible in-memory inventory which has been dinamically created by the `boot` playbook an ansible static inventory will be written to disk
      * The static inventory which is generated by the `boot` playbook is used by the other playbooks to configure the required services for the course e.g. slurm or rstudio

# How to use these playbooks

## Prepare the environment

### Download this git repo

```
$> git clone https://github.com/scicore-unibas-ch/ansible-playbook-scicore-courses-cloud.git
$> cd ansible-playbook-scicore-courses-cloud
```

### Install the required software

Tested with Python-3.6.6:

```
$> virtualenv venv_cloud
$> source venv_cloud/bin/activate
(venv_cloud)$> pip install ansible==2.9.13 python-openstackclient
(venv_cloud)$> ansible-galaxy role install -r ansible/requirements.yml -p ansible/roles/
```

### Configure authentication with the Cloud environment

Go to your cloud webui and download the auth RC file. In openstack this is usually located in the top right corner of the webui.

```
(venv_cloud)$> source openstack_auth.rc
(venv_cloud)$> source <(openstack complete)
```

At this point you should be able to interact with the cloud API from the CLI. Try these commands. They will be useful to define
you config file:

```
(venv_cloud)$> openstack image list
(venv_cloud)$> openstack flavor list
(venv_cloud)$> openstack network list
```

### Prepare you config file
```
(venv_cloud)$> cp config/slurm_cluster_switch_cloud.yml.example config/slurm_cluster_switch_cloud.yml
```

Now edit `config/slurm_cluster_switch_cloud.yml` and adapt it to your needs based on the output from previous commands. Most variables are self-descriptive.
TO-DO: Improve the config docs

## Booting a Slurm cluster

Check the wrapper `boot_and_configure_slurm_cluster.sh` which executes these ansible playbooks:

```
ansible-playbook -e @config/slurm_cluster_switch_cloud.yml ansible/slurm_cluster_boot_openstack.yml
ansible-playbook -i ansible/inventory/hosts -e @config/slurm_cluster_switch_cloud.yml ansible/slurm_cluster_configure_nfs_server.yml
ansible-playbook -i ansible/inventory/hosts -e @config/slurm_cluster_switch_cloud.yml ansible/slurm_cluster_configure_slurm_daemons.yml
ansible-playbook -i ansible/inventory/hosts -e @config/slurm_cluster_switch_cloud.yml ansible/slurm_cluster_configure_user_accounts.yml
```


## Connecting to the slurm cluster

```
$> ssh -F ~/.ssh/slurm_cluster_cloud.cfg slurm-login
```
