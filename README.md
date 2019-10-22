# sat6-deployer
This repository contains roles and playbooks to automatically configure a Satellite 6 server based on variables provided by the user.

This is tested with Red Hat Satellite 6.6.0 Beta.

## Prerequisites
Install a minimal RHEL 7 according to the Satellite 6 documentation: https://access.redhat.com/documentation/en-us/red_hat_satellite/6.6-beta/html/installing_satellite_server_from_a_connected_network/preparing_your_environment_for_installation 

### Register with subscription-manager
I have intentionally left the registration steps manual because this may differ a lot between different environments. The following instructions can be seen as an example, but may not be exactly the same for you.

Register with subscription-manager:
```bash
subscription-manager register
```

Find an available Satellite subscription you want to use and attach it with its pool ID:
```bash
subscription-manager list --available
subscription-manager attach --pool <pool ID of your Satellite subscription>
```

Ensure that you have access to the correct repositories:
```bash
subscription-manager repos \
--disable '*' \
--enable=rhel-7-server-rpms \
--enable=rhel-server-rhscl-7-rpms \
--enable=rhel-server-7-satellite-6-beta-rpms \
--enable=rhel-7-server-satellite-maintenance-6-beta-rpms \
--enable=rhel-7-server-ansible-2.8-rpms

yum clean all
yum repolist
```

Install the following packages:
```bash
yum -y install git ansible
```

### Clone this repository
```bash
git clone https://github.com/ture-karlsson/sat6-deployer.git
cd sat6-deployer/
```

### Clone the foreman-ansible-modules repository
```bash
git clone https://github.com/theforeman/foreman-ansible-modules.git
```
foreman-ansible-modules have a dependency on apypie. It can be installed with pip that is found in EPEL:
```bash
yum -y localinstall https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install python2-pip
pip install apypie
yum -y remove epel-release
```

## Usage

### hosts file
Make sure your Satellite hostname is the only host in the "satellite" group in the hosts file. Replace "sat6" with your hostname, e.g:
```bash
sed -i "s/sat6.example.com/$(hostname)/" hosts
```
The content of hosts you be something like this:
```bash
[satellite]
your-satellite.hostname.com
```

### Add your manifest file to the files directory
```bash
ls files/
manifest.zip
```

### vars/sat-vars.yml
The variable file vars/sat-vars.yml contains all the variables that are used for the installation and configuration of the Satellite server. Make sure to read it through and update it to fit your needs. This lets you define exactly what repositories to synchronize and all other configuration that will be set up in the Satellite.

### Run sat6-install.yml playbook

I have added a simple playbook called sat6-install.yml that can install the Satellite software for you. However, I recommend doing this step as well manually, since it differs a lot between different deployments. It is recommended to read through satellite-installer --scenario satellite --help to see what options that is needed in your environment.

However, if you want to use sat6-install.yml, it can be run with the following command.
```bash
ansible-playbook sat6-install.yaml
```
Some of the tasks, e.g. the yum tasks and the satellite-installer task take long time. If you want to track their progress, you can tail the respective logs:
```bash
tail -f /var/log/yum.log
```
and for the installer:
```bash
tail -f /var/log/foreman-installer/satellite.log
```

When the installer is done, it may be wise to reboot your Satellite if yum updated packages that require reboot.

### Run sat6-configure.yaml playbook
If the installation was successful, it is time to put some content in your Satellite. This is what takes the most time, but this is also where this automation saves you a lot of time. Again, make sure vars/sat-vars.yml looks as you expect it to. All configuration are based on the repositories, content views and host groups etc that are defined in there. The playbook can be run in different phases to configure only one or a couple types of objects by using the defined tags, e.g.

```bash
ansible-playbook sat6-configure.yml --tags manifest
ansible-playbook sat6-configure.yml --tags repositories,content-views,activation-keys
```
or everything at once:
```bash
ansible-playbook sat6-configure.yml
```

Have a look around in the GUI and see if all objects were created according to your expectations.

## TODO
Some roles does not yet use foreman-ansible-modules and needs to be fixed:
- sat6-user-roles
- sat6-smart-class-parameters
- sat6-template-sync
- sat6-openscap

Roles are missing for the following objects (and possibly more):
- sat6-compute-profile

## Issues
Please report issues and/or questions.

## Pull requests
Even better than reporting issues, send a pull request.
