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
---
- hosts: localhost
  gather_facts: false
  connection: local

  vars:
    nginx_controller_user_email: "user@example.com"
    nginx_controller_user_password: "mySecurePassword"
    nginx_controller_fqdn: "controller.mydomain.com"
    nginx_controller_validate_certs: false

  tasks:
  - include_role:
      name: nginxinc.nginx_controller.nginx_controller_generate_token

  - name: Get controller api key for agent registration
    uri:
      url: "https://{{ nginx_controller_fqdn }}/api/v1/platform/global"
      method: "GET"
      return_content: yes
      status_code: 200
      validate_certs: false
      headers:
        Cookie: "{{nginx_controller_auth_token}}"
    register: ctrl_globals

  - name: Copy api_key to a variable
    set_fact:
      api_key: "{{ctrl_globals.json.currentStatus.agentSettings.apiKey}}"

- hosts: tag_new_gateway
  remote_user: ubuntu
  become: true
  become_method: sudo
  gather_facts: yes

  tasks:
  - name: install minimal support for python2 for Agent install script
    apt:
      name: "{{ packages }}"
      state: present
    vars:
      packages:
      - python-minimal
      - libxerces-c3.2

  - name: install the agent
    include_role:
      name: nginxinc.nginx_controller.nginx_controller_agent
    vars:
      nginx_controller_api_key: "{{ hostvars['localhost']['api_key'] }}"
```

You can then run `ansible-playbook nginx_controller_agent.yaml` to execute the playbook.

Alternatively, you can also pass/override any variables at run time using the `--extra-vars` or `-e` flag like so `ansible-playbook nginx_controller_agent.yaml -e "nginx_controller_user_email=user@company.com nginx_controller_user_password=notsecure nginx_controller_fqdn=controller.example.local nginx_controller_validate_certs=false"`

You can also pass/override any variables by passing a `yaml` file containing any number of variables like so `ansible-playbook nginx_controller_agent.yaml -e "@nginx_controller_agent_vars.yaml"`

License
-------

[Apache License, Version 2.0](./LICENSE)

Author Information
------------------

[Brian Ehlert](https://github.com/brianehlert)

[Alessandro Fael Garcia](https://github.com/alessfg)

[Daniel Edgar](https://github.com/aknot242)

&copy; [NGINX, Inc.](https://www.nginx.com/) 2020
