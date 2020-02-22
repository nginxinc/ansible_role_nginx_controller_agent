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


`controller_fqdn` - The FQDN of the NGINX Controller server. ex: 10.20.30.40 or controller.example.com
`api_key` - The API key used to authenticate to NGINX Controller

### Optional Variables

`controller_hostname` - The name of the instance as reflected in NGINX Controller. Must be unique per instance.
`location` - The location in NGINX Controller this instance will be automatically added to. Otherwise the location will be 'unspecified' in NGINX Controller.
`instance_name` - The name of the instance as reflected in Controller. Must be unique per instance (currently redundant with controller_hostname).

Dependencies
------------

Example Playbook
----------------

```yaml
- hosts: nginxservers
  remote_user: ubuntu
  become: true
  become_method: sudo
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

    - name: Generate auth token for NGINX Controller
      include_role:
        name: nginx_controller_generate_token
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
        name: nginx_controller_agent
      vars:
        api_key: "{{ api_key }}"
        controller_fqdn: "controller.mydomain.com"
```

License
-------

[Apache License, Version 2.0](./LICENSE)

Author Information
------------------

brianehlert
