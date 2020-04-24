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

`nginx_controller_fqdn` - FQDN of the NGINX Controller instance

`nginx_controller_api_key` - The API key used to authenticate to NGINX Controller.

### Optional Variables

`nginx_controller_hostname` - The name of the NGINX instance as reflected in NGINX Controller. Must be unique per instance.  (currently redundant with nginx_controller_instance_name)

`nginx_controller_location` - The location in NGINX Controller this instance will be automatically added to. Otherwise the location will be 'unspecified' in NGINX Controller.

`nginx_controller_instance_name` - The name of the instance as reflected in Controller. Must be unique per instance.

Dependencies
------------

Example Playbook
----------------

To use this role you can create a playbook such as the following:

```yaml
- hosts: localhost
  gather_facts: yes
  vars:
    nginx_controller_user_email: "user@example.com"
    nginx_controller_user_password: "mySecurePassword"
    nginx_controller_fqdn: "controller.mydomain.com"
    nginx_controller_validate_certs: false

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
        name: nginxinc.nginx_controller_generate_token

    - name: Get NGINX Controller API key for agent registration
      uri:
        url: "https://{{ nginx_controller_fqdn }}/api/v1/platform/global"
        method: "GET"
        return_content: yes
        status_code: 200
        validate_certs: "{{ nginx_controller_validate_certs | default(false) }}"
        headers:
          Cookie: "{{ nginx_controller_auth_token }}"
      register: nginx_controller_globals

    - name: Copy api_key to a variable
      set_fact:
        nginx_controller_api_key: "{{ nginx_controller_globals.json.currentStatus.agentSettings.apiKey }}"

    - name: Install the NGINX Controller agent
      include_role:
        name: nginxinc.nginx_controller_agent
      vars:
        nginx_controller_api_key: "{{ nginx_controller_api_key }}"
```

Use
---

```cli
ansible-playbook nginx_controller_agent.yaml -b -i hosts -e "nginx_controller_user_email=controller-admin@nginx.com nginx_controller_user_password=password nginx_controller_fqdn=controller.acme.com"
```

License
-------

[Apache License, Version 2.0](./LICENSE)

Author Information
------------------

[Brian Ehlert](https://github.com/brianehlert)

[Alessandro Fael Garcia](https://github.com/alessfg)

[Daniel Edgar](https://github.com/aknot242)

&copy; [NGINX, Inc.](https://www.nginx.com/) 2020
