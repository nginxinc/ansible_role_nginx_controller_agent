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
- api_key

Optional:

- controller_hostname
- location
- instance_name

Dependencies
------------

Example Playbook
----------------

- hosts: nginxservers
  remote_user: ubuntu
  become: true
  become_method: sudo
  gather_facts: yes

  tasks:
  - name: upgrade packages here.
    apt:
      update_cache: yes
      upgrade: yes

  - name: install minimal support for python2 for Agent install script
    apt:
      name: "{{ packages }}"
      state: present
    vars:
      packages:
      - python-minimal

  - include_role:
      name: nginxinc.nginx-controller-generate-token
    vars:
      user_email: "user@example.com"
      user_password: 'secure_password'
      controller_fqdn: ""

  - name: Get controller api key for agent registration
    uri:
      url: "https://{{controller_fqdn}}/api/v1/platform/global"
      method: "GET"
      return_content: yes
      status_code: 200
      validate_certs: false
      headers:
        Cookie: "{{controller_auth_token}}"
    register: ctrl_globals

  - name: Copy api_key to a variable
    set_fact:
      api_key: "{{ctrl_globals.json.currentStatus.agentSettings.apiKey}}"

  - name: install the agent
    include_role:
      name: nginxinc.nginx-controller-agent
    vars:
      controller_fqdn:
      api_key:

License
-------

Apache

Author Information
------------------

brianehlert
