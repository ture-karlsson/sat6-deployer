# sat6-deployer
This repository contains some playbooks to install Satellite 6 server and automatically configure content in it based on a configuration file specified by the user.

## Prerequisites
Install a minimal RHEL 7.3 (or later) according to the prerequisites for Satellite 6: https://access.redhat.com/documentation/en-us/red_hat_satellite/6.2/html/installation_guide/preparing_your_environment_for_installation

These playbooks presumes that they run locally on the host where Satellite should be installed.
If this is not the case, the playbooks can be altered to be able to run from another node by removing the "connection: local" directive etc.

This process presumes that your Satellite have access to redhat.com and fedoraproject.org.

### Register with subscription-manager
I have intentionally left the registration steps manual because this may differ a lot between different environments. The following instructions can be seen as an example, but may not be exactly the same for you.

Register with subscription-manager:
```bash
# subscription-manager register
```

Find an available Satellite subscription you want to use and attach it with its pool ID:
```bash
# subscription-manager list --available
# subscription-manager attach --pool <pool ID of your Satellite subscription>
```

Reference: https://access.redhat.com/documentation/en-us/red_hat_satellite/6.2/html/installation_guide/installing_satellite_server

### Install Ansible
```bash
# rpm -i https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# yum -y install ansible
# yum -y remove epel-release
```

### Clone this repository
```bash
# yum -y install git
# git clone https://github.com/ture-karlsson/sat6-deployer.git
# cd sat6-deployer/
```

## Usage

### hosts file
Make sure your Satellite hostname is the only host in the "satellite" group in the hosts file. Replace "sat6" with your hostname, e.g:
```bash
# sed -i "s/sat6/$(hostname)/" hosts
# cat hosts
[satellite]
your-satellite.hostname.com
```

### Add your manifest file to the files directory
```bash
# ls files/
manifest.zip
```

### group_vars/satellite
The variable file group_vars/satellite contains all the variables that are used for the installation and configuration of the Satellite server. Make sure to read it through and update it to fit your needs. This lets you define exactly what repositories to synchronize and all other configuration that will be set up in the Satellite.

group_vars/satellite have a list of so called profiles. One profile is a group of repositories and certain subscriptions that a certain type of hosts use. A Content View will then be created for this profile as well as Activation Keys and Host Groups for ALL different Lifecycle Environments. There are 3 example profiles in group_vars/satellite. One for basic RHEL 7 servers (named Base) one for Ansible Tower servers (named Tower) and one for Openshift servers. By defining the Openshift profile for example, the sat6-configure.yaml playbook will create all content in Satellite you need to register Openshift servers in each Lifecycle Environment to have access to the specified repositories and consume the specified subscription. It also creates the required Host Groups for you.

### Run sat6-install.yaml playbook
Now you are ready to execute the first playbook that updates your server and installs the Satellite software:
```bash
# ansible-playbook -i hosts sat6-install.yaml
```

It may be wise to reboot your Satellite at this stage if yum updated packages that require reboot.

### Run sat6-configure.yaml playbook
If the installation was successful, it is time to put some content in your Satellite. This is what takes the most time, but this is also where this automation saves you a lot of time. Again, make sure group_vars/satellite looks as you expect it to. All configuration are based on the repositories, profiles etc that are defined in there.
```bash
# ansible-playbook -i hosts sat6-configure.yaml
```

### Outcome
If sat6-configure.yaml runs successfully, the playbook have done the following for you:
* uploaded your manifest
* created all needed products and repositories
* synced all repositories
* created Lifecycle Environments (LCEs)
* created Content Views (CVs)
* promoted all CVs to all LCEs
* created Activation Keys (AKs)
* created Host Collections (HCs)
* created Host Groups (HGs)

Have a look around in the GUI and see if all objects were created according to your expectations.

## Issues?
Please report issues and/or questions.
