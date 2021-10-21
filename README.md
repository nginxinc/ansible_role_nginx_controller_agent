# Ansible NGINX Controller Agent Role

This role installs, configures, and upgrades the NGINX Controller agent alongside an NGINX Plus instance in a machine.

## Requirements

### NGINX Controller and NGINX Plus

* A running [NGINX Plus](https://www.nginx.com/products/nginx/) instance.
* A running [NGINX Controller](https://www.nginx.com/products/nginx-controller/) instance.

### Ansible

* This role is developed and tested with [maintained](https://docs.ansible.com/ansible/devel/reference_appendices/release_and_maintenance.html) versions of Ansible core (above `2.11`).
* You will need to run this role as a root user using Ansible's `become` parameter. Make sure you have set up the appropriate permissions on your target hosts.
* Instructions on how to install Ansible can be found in the [Ansible website](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#upgrading-ansible-from-version-2-9-and-older-to-version-2-10-or-later).

### Molecule (Optional)

* Molecule is used to test the various functionalities of the role. The recommended version of Molecule to test this role is `3.3`.
* Instructions on how to install Molecule can be found in the [Molecule website](https://molecule.readthedocs.io/en/latest/installation.html). _You will also need to install the Molecule Docker driver._

## Installation

### Ansible Galaxy

Use `ansible-galaxy install nginxinc.nginx_controller_agent` to install the latest stable release of the role on your system. Alternatively, if you have already installed the role, use `ansible-galaxy install -f nginxinc.nginx_controller_agent` to update the role to the latest release.

### Git

Use `git clone https://github.com/nginxinc/ansible_role_nginx_controller_agent.git` to pull the latest edge commit of the role from GitHub.

## Platforms

The NGINX Controller agent Ansible role supports all platforms supported by the [NGINX Controller agent](https://docs.nginx.com/nginx-controller/admin-guides/install/nginx-controller-tech-specs/):

```yaml
Amazon:
  - 2017.09
Amazon Linux 2:
  - any
CentOS:
  - 7.4+
Debian:
  - stretch (9)
  - buster (10)
RHEL:
  - 7.4+
Ubuntu:
  - bionic (18.04)
  - focal (20.04)
```

## Role Variables

| Variable | Default | Description | Required |
| -------- | ------- | ----------- | -------- |
| `nginx_controller_fqdn` | `""` | FQDN of the NGINX Controller instance | Yes |
| `nginx_controller_api_key` | `""` | The API key used to authenticate to NGINX Controller | Yes |
| `nginx_controller_location` | `"unspecified"` | The location in NGINX Controller this instance will be automatically added to | No |
| `nginx_controller_hostname` | `""` | The unique name of the NGINX instance as reflected in NGINX Controller -- currently redundant with `nginx_controller_instance_name` | No |
| `nginx_controller_instance_name` | `""` | The unique name of the NGINX instance as reflected in NGINX Controller -- currently redundant with `nginx_controller_hostname` | No |

## Example Playbook

To use this role you can create a playbook such as the following:

```yaml
---
- name: Fetch NGINX Controller API Key
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    nginx_controller_user_email: "user@example.com"
    nginx_controller_user_password: "mySecurePassword"
    nginx_controller_fqdn: "controller.mydomain.com"
    nginx_controller_validate_certs: false
  tasks:
    - name: Fetch NGINX Controller auth token
      include_role:
        name: nginxinc.nginx_controller_generate_token

    - name: Fetch NGINX Controller API Key for the NGINX Controller agent registration
      uri:
        url: "https://{{ nginx_controller_fqdn }}/api/v1/platform/global"
        method: GET
        return_content: yes
        status_code: 200
        validate_certs: false
        headers:
          Cookie: "{{ nginx_controller_auth_token }}"
      register: ctrl_globals

    - name: Filter API Key to a variable
      set_fact:
        api_key: "{{ ctrl_globals.json.currentStatus.agentSettings.apiKey }}"

- name: Install NGINX Controller agent
  hosts: tag_new_gateway
  remote_user: ubuntu
  become: true
  become_method: sudo
  tasks:
    # - name: Install minimal support for python2 for Agent install script
    #   apt:
    #     name:
    #       - python-minimal
    #       - libxerces-c3.2

    - name: Install the NGINX Controller agent
      include_role:
        name: nginxinc.nginx_controller_agent
      vars:
        nginx_controller_api_key: "{{ hostvars['localhost']['api_key'] }}"
```

## Other NGINX Ansible Collections and Roles

You can find the Ansible NGINX Core collection of roles to install and configure NGINX Open Source, NGINX Plus, and NGINX App Protect [here](https://github.com/nginxinc/ansible-collection-nginx).

You can find the Ansible NGINX role to install NGINX OSS and NGINX Plus [here](https://github.com/nginxinc/ansible-role-nginx).

You can find the Ansible NGINX configuration role to configure NGINX [here](https://github.com/nginxinc/ansible-role-nginx-config).

You can find the Ansible NGINX App Protect role to install and configure NGINX App Protect WAF and NGINX App Protect DoS [here](https://github.com/nginxinc/ansible-role-nginx-app-protect).

You can find the Ansible NGINX Controller collection of roles to install and configure NGINX Controller [here](https://github.com/nginxinc/ansible-collection-nginx_controller).

You can find the Ansible NGINX Unit role to install NGINX Unit [here](https://github.com/nginxinc/ansible-role-nginx-unit).

## License

[Apache License, Version 2.0](https://github.com/nginxinc/ansible-role-nginx-config/blob/main/LICENSE)

## Author Information

[Alessandro Fael Garcia](https://github.com/alessfg)

[Brian Ehlert](https://github.com/brianehlert)

[Daniel Edgar](https://github.com/aknot242)

&copy; [F5 Networks, Inc.](https://www.f5.com/) 2020 - 2021
