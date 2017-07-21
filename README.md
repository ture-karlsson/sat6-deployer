# sat6-deployer
This repository contains some playbooks to spin up a new AWS instance, install Satellite 6 server on it and automatically configure content in it based on a configuration file specified by the user.

## Prerequisites
These playbooks requires that the host executing the playbook runs have access to the AWS API and to the created instance over SSH.
In order to connect to the created instance, the privatekey used during the creating also needs to be downloaded and put into the playbook files/ folder. (more described later)
The Satellite also needs to have a manifest. The playbook expects by defualt that this file is found in files/manifest.zip. The manifest is created and downloaded from the Red Hat Customer Portal.

If the resulting instance needs to use a proxy, this has to be specified in the group_vars/satellite.yaml

This process presumes that your Satellite have access to redhat.com and fedoraproject.org.

## Usage

# Executing the playbooks

The main design to run the provisioning, installation and configuration end to end . 
By just running the deploy-satellite.yaml the idea is to have a Satellite up and running with content synchronized and Content Views and Activation Keys created. 

Before running the deployer we need to satisfy X things:

- manifest file
Satellite needs a manifest file downloaded from the Customer Portal that contains all the subscriptions the Satellite should manage.
Upload the manifest and place it in files/ and name it manifest.zip
The filepath can be respecified in group_vars/satellite.ymal

- the private key from AWS
When creating the instance we specify a private key to inject into the instance using AWS cloud-init. We will need this key in order to connect to the instance after it has been provisioned.
Upload the key to the files/ folder aswell. The name and filepath of the key can be specified in the group_vars/satellite.yaml

- AWS keys
In order to create the instance, the ec2_module needs the AWS access and secret key.
These should be specified in the group_vars/satellite.yaml
CAUTION! The AWS keys are very sensitive data and be careful when checking in this repository to any SCM.
 
- group_vars/satellite.yaml 
All configuration is fetched from the group_vars/satellite.yaml file. 
Go through the file and read the descriptions and enter the values you want. 




After makinge sure everything is set do:
# ansible-playbook deploy-satellite.yaml

The playbook doesn’t need any inventory file as it’s first running locally for the API calls to AWS, and uses the instance information received to create a temporary inventory file with the add_host module. 

### hosts file and standalones
When running the playbooks end to end using the deploy-satellite.yaml you don't have to specify an inventory file or any hosts.
This because the first play "ec2-create-instance" will always run locally when communicating with the AWS API. After the instance has been created the
IP will by dynamically entered using the ``` add_host ``` module. 

If you however need to repeat a play without running the previous steps, you can use the standalone playbooks.
The last two playbooks, sat6-install and sat6-configure needs an inventory file since we don’t have access to the “just_created” host that we have running the end-to-end playbook. 
They both use “hosts: satellite” and therefor we need to populate the inventory file with the correct host entries. There is an example in the hosts file that you can copy and replace the IP with the IP of the instance you want to run on and make sure that the private key is right. 

Executing standalones:


To run the steps individually use the standalone playbooks:
Ec2-create-instance-standalone.yaml
Sat6-install-standalone.yaml
Sat6-configure-standalone.yaml

Example:
```bash
# ansible-playbook sat6-install.yaml -i ./hosts 
``` 

## Structure 
### deploy-satellite.yaml
Main playbook which calls all the other plays. 
This is to be executed like:
```bash
# ansible-playbook deploy-satellite.yaml
```
### files/
This folder should contain 
manifest.zip - The Satellite manifest downloaded from the Red Hat Customer Portal
private_key.pem (The  AWS private key file .pem file that is used, make sure the correct file path and name is in the group_vars/satellite.yaml, aswell as the inventory file if you need to run the standalone playbooks).

### ec2-create-instance.yaml
The play that calls on the AWS API to create the instance with the variables specified in the group_vars/satellite.yaml

### group_vars/satellite.yaml
The variable file group_vars/satellite contains all the variables that are used for the installation and configuration of the Satellite server. Make sure to read it through and update it to fit your needs. This lets you define exactly what repositories to synchronize and all other configuration that will be set up in the Satellite.

group_vars/satellite have a list of so called profiles. One profile is a group of repositories and certain subscriptions that a certain type of hosts use. A Content View will then be created for this profile as well as Activation Keys and Host Groups for ALL different Lifecycle Environments. There are 3 example profiles in group_vars/satellite. One for basic RHEL 7 servers (named Base) one for Ansible Tower servers (named Tower) and one for Openshift servers. By defining the Openshift profile for example, the sat6-configure.yaml playbook will create all content in Satellite you need to register Openshift servers in each Lifecycle Environment to have access to the specified repositories and consume the specified subscription. It also creates the required Host Groups for you.

!This var file asks for AWS secrets so be careful of what you check in.!


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
