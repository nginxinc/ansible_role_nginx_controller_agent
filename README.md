nginx-controller-agent
=========

This Role installs, configures, upgrades the NGINX Controller agent alongside of an NGINX Plus instance in a machine.

Requirements
------------

NGINX Plus [](https://www.nginx.com/products/nginx/)
NGINX Controller [](https://www.nginx.com/products/nginx-controller/)

Role Variables
--------------

Required:

- controller_fqdn
- controller_auth_token
- api_key

Optional:

- controller_hostname
- location

Dependencies
------------

Example Playbook
----------------

- hosts: nginxservers

  roles:

  - nginxinc.nginx-controller-generate-token
  - nginxinc.nginx-controller-agent

License
-------

Apache

Author Information
------------------

brianehlert
