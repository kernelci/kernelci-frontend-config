KernelCi Frontend Config
========================

This is the Ansible configuration to deploy/setup the kernelci-frontend
system at:

    https://github.com/kernelci/kernel-ci-frontend

Requirements
############

Ansible >= 2.0

In order to run the entire configuration, some "secrets" are necessary.
Put them in a YAML file, and pass it to Ansible as:

    -e "@/path/to/file/secrets.yml"

To skip the secrets taks, just pass:

    --skip-tags=secrets

Secret Keys
-----------

The secret keys that have to be defined are:

* backend_url
* base_url
* backend_token
* secret_key
* info_email
* ssl_stapling_resolver
* google_analytics


The `secret_key` value should be set to a random string, it is used internally
by Flask.

Other non-secrets variable might need to be defined, please look at the `templates` directories.
