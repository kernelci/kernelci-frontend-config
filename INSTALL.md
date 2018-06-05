# KernelCI Frontend Ansible installation process

Documentation of the KernelCI frontend installation process

## KernelCI architecture overview

The architecture of KernelCI is split into three main components:
* Build system
* Frontend
* Backend

For more information on the architecture please visit: http://wiki.kernelci.org/

## Prerequisites

Two machines:
* The host machine which will run the ansible scripts to remotely install the frontend onto the target machine.
	* Host configuration prerequisites
		* Git tools
		* Ansible >= 2.0: http://docs.ansible.com/ansible/latest/intro_installation.html
* The target machine where the kernelci infrastructure (front and back end) will be deployed
	* Target configuration prerequisites
		* Supported OS: Debian (Stretch and after), CentOS >= 7
		* ssh root access, at least for the host machine from where you'll be running ansible
		* Python >= 2.7.12

* Ansible need to have a way to be root  in the target machine, either with direct root connect or with user+sudo/su, you need to choose which way will be used.
	* For direct root access:
		* add `ansible_ssh_user=root` after serverhostname in the hosts file
	* For user+sudo
		* add `ansible_ssh_user=user become_method=sudo` after serverhostname in the hosts file
	* More information on http://docs.ansible.com/ansible/latest/become.html

## Installing

### Install order
* You need to install the [backend first](http://github.com/kernelci/kernelci-backend-config/)
* Then you need to generate tokens for frontend and set them in secrets.yml
* Then install the [frontend](http://github.com/kernelci/kernelci-frontend-config/)

### Get the ansible playbook

If you didn't clone this repository yet, you need to do it for running the ansible playbook installing the backend:

```
git clone https://github.com/kernelci/kernelci-frontend-config.git
```

### Configuration
* The server running the instance is named TARGET_NAME in the rest of the document.
* You need to choose the FQDN used for calling both the frontend and backend; one FQDN for each, named FQDN_BACKEND and FQDN_FRONTEND in the rest of the document.
  These FQDN must be set in the hostname variable. The default in `group_vars/all` is kernelci-frontend/kernelci-backend.
  This name could be different from TARGET_NAME.
* Naming example:
    * TARGET_NAME: server.mydomain.local
    * FQDN_FRONTEND: frontend.mydomain.local
    * FQDN_BACKEND: api.mydomain.local

You need to modify or add the following files:

* `group_vars/all` :
Replace hostname and nickname for something else if you don't want to use kernelci-frontend.
Change the role to "staging" instead of "production" if you're installing locally for testing or development.


* `hosts`:
Add the TARGET_MACHINE and configure how to ansible should connect, for example if your machine
is named backend.mydomain.local with IP 10.0.17.3 and you connect directly as root (using a ssh key),
your should add:

```
[local]
TARGET_MACHINE ansible_ssh_port=22 ansible_ssh_host=10.0.17.3 ansible_ssh_user=root
```

If the user is not root and should use "su" to become root, use the parameter `become_method=su`, if it should use "sudo" instead, use `become_method=sudo`

* `host_vars/TARGET_NAME` this is a file that you must create

For example, you can create the file  `host_vars/myserver.mydomain.local` with the following content:

```
hostname: frontend.mydomain.local
role: staging
```

### Create secrets.yml file
In order to run the entire configuration, some "secrets" are necessary.
Use the file `templates/secrets.yaml` as an example and provide it to ansible:

    -e "@/path/to/file/secrets.yml"

To skip the secrets taks, just pass:

    --skip-tags=secrets

The key that have to be defined for a local installation for testing ot development is:

* backend_url: The URL for requesting the backend. Ex: http://FQDN_BACKEND/
* base_url: The URL for requesting the frontend. Ex: http://FQDN_FRONTEND/
* backend_token: A token created when you installed the backend. [More information at the backend installation](https://github.com/kernelci/kernelci-backend-config/blob/master/INSTALL.md#token-management)
* secret_key: This should be set to a random string, it is used internally by Flask.
* master_key: The password used for generating tokens before any admin token was created. This is the same you used in the backend installation.

Other non-secrets variable might need to be defined, please look at the comments in the file.

### Run Ansible

From the root directory of this repository run:

```
$ ansible-playbook -i hosts site.yml -e "@../secrets.yml" -l <$TARGET_NAME> 
```

This will deploy the  [KernelCI frontend code](https://github.com/kernelci/kernelci-frontend.git) into `/srv/$hostname`,
install all dependencies and set up an nginx host called `$hostname`.

### Requirements

Non exhaustive list of requirements is in the 'requirements.txt' file: those
need to be installed via pip.
For production, requirements.txt is sufficient. For development purpose, requirements-dev.txt
will add extra package for testing.
