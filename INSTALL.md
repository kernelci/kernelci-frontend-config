# KernelCi Frontend Ansible installation process

Documentation of the Kernelci frontend installation process

## KernelCI architecture overview

The architecture of kernelCI is split into three main components:
* Build system
* Frontend
* Backend

For more information on the architecture please visit: http://wiki.kernelci.org/

## Prerequisites

Two machines:
* The host machine which will run the ansible scripts to remotely install the back end onto the target machine.
	* Host configuration prerequisites
		* Git tools
		* Ansible >= 2.0: http://docs.ansible.com/ansible/latest/intro_installation.html
* The target machine where the kernelci infrastructure (front and back end) will be deployed
	* Target configuration prerequisites
		* Supported OS: Debian (Jessie and after), Centos >= 7
		* ssh root access to the server
		* Python >= 2.7.12

* Ansible need to have a way to be root either with direct root connect or either with user+sudo/su, you need to choose which way will be used.
	* For direct root access:
		* add ansible_ssh_user=root after serverhostname in hosts
	* For user+sudo
		* add ansible_ssh_user=user become_method=sudo after serverhostname in hosts
	* More informations on http://docs.ansible.com/ansible/latest/become.html

## Installing

### Install order
* You need to install the [backend first](http://github.com/kernelci/kernelci-backend-config/)
* Then you need to generate tokens for frontend and set them in secrets.yml
* Then install the [frontend](http://github.com/kernelci/kernelci-frontend-config/)

### Get the source code
```
For Deploying kernelci-frontend
git clone https://github.com/kernelci/kernelci-frontend-config.git
```

### Configure Host
* You need to choose the FQDN used for calling both the frontend and backend.
  These FQDN must be set in the hostname variable. (default from group_vars/all is kernelci-frontend/kernelci-backend)
  Thoses name could be different from the hostname of the host.

* then add the choosen hostname in the hosts file
	* Example:
```
[dev]
#this machine will be managed directly via root
kernelci.mydns.com ansible_ssh_user=root
[rec]
#this machine will be managed via the user admin becoming root via "su"
kernelci.mydns.com ansible_ssh_user=admin become_method=su
[prod]
#this machine will be managed via the user admin using sudo
kernelci.mydns.com ansible_ssh_user=admin become_method=sudo
```

### Create secrets.yml file
In order to run the entire configuration, some "secrets" are necessary.
Put them in a YAML file, and pass it to Ansible as:

    -e "@/path/to/file/secrets.yml"

To skip the secrets taks, just pass:

    --skip-tags=secrets

#### Secret Keys


The secret keys that have to be defined are:

* backend_url: The URL for requesting the backend. Ex: http://kernelci-backend/
* base_url: The URL for requesting the frontend. Ex: http://kernelci-frontend/
* backend_token: A GET+POST token. (See Manual tasks below for generating this token)
* secret_key:  should be set to a random string, it is used internally by Flask.
* master_key: The password used for generating tokens before any admin token was created (See Manual tasks below).
* info_email: email address to be displayed for contact informations
* ssl_stapling_resolver: (optionnal)
* google_analytics: A google_analytics ID (optionnal)


Other non-secrets variable might need to be defined, please look at the `templates` directories.

### Run Ansible
```
$ cd <kernelci-frontend-diretory>
$ ansible-playbook -i hosts site.yml -e "@../secrets.yml" -l <$TARGET_NAME> 
```

This will deploy the kernel-ci frontend code into `/srv/$hostname`,
intall all dependencies and set up an nginx host called `$hostname`.

### Requirements

Non exhaustive list of requirements is in the 'requirements.txt' file: those
need to be installed via pip.
For production, requirements.txt is sufficient. For development purpose, requirements-dev.txt
will add extra package for testing.
