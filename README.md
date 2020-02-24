NGINX Controller Agent
======================

This Role installs, configures, and upgrades the NGINX Controller agent alongside an NGINX Plus instance in a machine.

Requirements
------------

*   [NGINX Plus](https://www.nginx.com/products/nginx/)
*   [NGINX Controller](https://www.nginx.com/products/nginx-controller/)

Role Variables
--------------

### Required Variables

`controller_fqdn` - FQDN of the NGINX Controller instance

`api_key` - The API key used to authenticate to NGINX Controller.

### Optional Variables

`controller_hostname` - The name of the instance as reflected in NGINX Controller. Must be unique per instance.

`location` - The location in NGINX Controller this instance will be automatically added to. Otherwise the location will be 'unspecified' in NGINX Controller.

`instance_name` - The name of the instance as reflected in Controller. Must be unique per instance (currently redundant with controller_hostname).

Dependencies
------------

Example Playbook
----------------

To use this role you can create a playbook such as the following:

```yaml
- hosts: localhost
  gather_facts: yes

  tasks:
    - name: Upgrade your packages
      apt:
        update_cache: yes
        upgrade: yes

    - name: Install minimal support for python2 for the NGINX Controller agent install script
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - python-minimal
        - libxerces-c3.2

    - name: Retrieve the NGINX Controller auth token
      include_role:
        name: nginxinc.nginx-controller-generate-token
      vars:
        user_email: "user@example.com"
        user_password: "mySecurePassword"
        controller_fqdn: "controller.mydomain.com"

    - name: Get NGINX Controller API key for agent registration
      uri:
        url: "https://{{ controller_fqdn }}/api/v1/platform/global"
        method: "GET"
        return_content: yes
        status_code: 200
        validate_certs: false
        headers:
          Cookie: "{{ controller_auth_token }}"
      register: ctrl_globals

    - name: Copy api_key to a variable
      set_fact:
        api_key: "{{ ctrl_globals.json.currentStatus.agentSettings.apiKey }}"

    - name: Install the NGINX Controller agent
      include_role:
        name: nginxinc.nginx-controller-agent
      vars:
        api_key: "{{ api_key }}"
        controller_fqdn: "controller.mydomain.com"
```

License
-------

[Apache License, Version 2.0](./LICENSE)

Author Information
------------------

[Brian Ehlert](https://github.com/brianehlert)

[Alessandro Fael Garcia](https://github.com/alessfg)

&copy; [NGINX, Inc.](https://www.nginx.com/) 2020
