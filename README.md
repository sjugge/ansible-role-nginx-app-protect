<img src="images/nap-logo.png" width="60">

NGINX App Protect Ansible Role
==============================

<!-- [![Ansible Galaxy](https://img.shields.io/badge/galaxy-nginxinc.nginx-5bbdbf.svg)](https://galaxy.ansible.com/nginxinc/nginx) -->
<!-- [![Build Status](https://travis-ci.org/nginxinc/ansible-role-nginx-app-protect.svg?branch=master)](https://travis-ci.org/nginxinc/ansible-role-nginx-app-protect) -->

This role installs and configures NGINX App Protect (WAF) for NGINX Plus on your target host.

**Note:** This role is still in active development. There may be unidentified issues and the role variables may change as development continues.

Requirements
------------

**Ansible**

This role was developed and tested with [maintained](https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#release-status) versions of Ansible. Backwards compatibility is not guaranteed.

Instructions on how to install Ansible can be found in the [Ansible website](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

**Molecule**

Molecule is used to test the various functionalities of the role. Instructions on how to install Molecule can be found in the [Molecule website](https://molecule.readthedocs.io/en/stable/installation.html).

Installation
------------

**Ansible Galaxy**

Use `ansible-galaxy install nginxinc.nginx_app_protect` to install the latest stable release of the role on your system.

**Git**

Use `git clone https://github.com/nginxinc/ansible-role-nginx-app-protect.git` to pull the latest edge commit of the role from GitHub.

Platforms
---------

The NGINX App Protect Ansible role supports all platforms supported by [NGINX Plus](https://www.nginx.com/products/technical-specs/) that intersect with the following list:

```yaml
CentOS:
  versions:
    - 7.4
    - 7.5
    - 7.6
    - 7.7
    - 7.8
    - 8.0
    - 8.1
    - 8.2
Debian:
  versions:
    - 9.0
    - 9.1
    - 9.2
    - 9.3
    - 9.4
    - 9.5
    - 9.6
    - 9.7
    - 9.8
    - 9.9
    - 9.10
    - 9.11
    - 9.12
```

Role Variables
--------------

This role has multiple variables. The descriptions and defaults for all these variables can be found in the **[defaults/main.yml](./defaults/main.yml)`**.


Dependencies
------------

- Since this role uses the [package_facts](https://docs.ansible.com/ansible/latest/modules/package_facts_module.html) module, on debian-based systems the `python-apt` package must be installed on targeted hosts.

- NGINX+ R19-R21 must already be installed on the target system 

Example Playbook
----------------


This is a sample playbook file for using the role to install NGINX App Protect on NGINX Plus and configure it using basic settings to all `wafs` inventory hosts.

```yaml
---
- hosts: wafs
  become: true
  vars:

    # Specify whether you want to maintain your version of NGINX App Protect, upgrade to the latest version, or remove NGINX App Protect.
    # Can be used with `app_protect_version` to achieve fine grained control on which version of NGINX App Protect is installed/used on each playbook execution.
    # Using 'present' will install the latest version (or 'app_protect_version') of NGINX App Protect on a fresh install.
    # Using 'latest' will upgrade NGINX App Protect to the latest version (that matches your 'app_protect_version') of NGINX App Protect on every playbook execution.
    # Using 'absent' will remove NGINX App Protect from your system.
    # Default is present.
    app_protect_state: present

    # OPTIONAL: Installs a specific version of NGINX App Protect
    app_protect_version: 21

    # Enable enforcing selinux (you may need to open ports on your own)
    # WARNING: If this is set to false and you are installing NGINX Protect on a system with SELinux enforced, NGINX App Protect may fail to load. 
    app_protect_selinux: false

    # The installation of NGINX App Protect includes a base signature set, which may be out of date. 
    # This option installs the latest NGINX App Protect signatures.
    app_protect_install_signatures: true

    # The installation of NGINX App Protect can include a page of frequently-updated, high-accuracy signatures called Threat Campaigns.
    # This option installs the latest NGINX App Protect Threat Campaigns signatures.
    app_protect_install_threat_campaigns: true

    # Creates basic configuration files and enables NGINX App Protect on the target host
    app_protect_configure: true

    # Removes the license (certificate and key) for the NGINX App Protect repositories on the target host(s) when playbook run is complete.
    app_protect_delete_license: true

    # For use with the app_protect_configure option to determine if the default security policy will be written to the target host
    # Used when `app_protect_configure: true`.
    app_protect_security_policy_template_enable: true

    # Default app protect enforcement mode. Values can be `blocking` or `transparent`. 
    # Used when `app_protect_configure: true` and `app_protect_security_policy_template_enable: true`.
    security_policy_enforcement_mode: blocking

    # For use with the app_protect_configure option to determine if the default log policy will be written to the target host.
    # Used when `app_protect_configure: true`.
    app_protect_log_policy_template_enable: true

    # Which violation types to log. Possible values: all, illegal, blocked
    # Used when `app_protect_configure: true` and `app_protect_log_policy_template_enable: true`.
    log_policy_filter_request_type: all

    # For use with the app_protect_configure option to determine if the sample nginx.conf will be written to the target host. 
    # Since this can be dangerous, this value is default to false in the role defaults.
    # Used when `app_protect_configure: true`.
    nginx_conf_template_enable: true

    # For use with the app_protect_configure option to determine the syslog target to be injected 
    # into the default log policy that will be written to the target host. 
    # Used when `nginx_conf_template_enable: true`.
    log_policy_syslog_target: 10.1.1.8:5144

    # DEPRECATED: A proxy pass workload used in the sample nginx.conf for demo purposes.
    # Will be removed from this role in the future. 
    # Used when `nginx_conf_template_enable: true`.
    nginx_demo_workload: http://10.1.10.105:8080

    # The location of the certificate and key to be used when downloading the packages onto the host
    nginx_license: 
      certificate: "{{playbook_dir}}/license/nginx-repo.crt"
      key: "{{playbook_dir}}/license/nginx-repo.key"

  roles:
    - role: ansible-role-nginx-app-protect
```

This is a sample playbook file for deploying the Ansible Galaxy NGINX App Protect role in a localhost and installing NGINX App Protect on NGINX Plus.

```yaml
---
- hosts: localhost
  become: true
  roles:
    - role: nginxinc.nginx_app_protect
```

This is a sample playbook file for deploying the Ansible Galaxy NGINX App Protect role to a dynamic inventory containing the `nginx_plus` tag.

```yaml
---
- hosts: tag_nginx_plus
  remote_user: root
  roles:
    - role: nginxinc.nginx_app_protect
```

To run any of the above sample playbooks create a `setup-nginx-app-protect.yml` file and paste the contents. Executing the Ansible Playbook is then as simple as executing `ansible-playbook setup-nginx.yml`.

Alternatively, you can also clone this repository instead of installing it from Ansible Galaxy. If you decide to do so, replace the role variable in the previous sample playbooks from `nginxinc.nginx_app_protect` to `ansible-role-nginx-app-protect`.

Other NGINX Roles
-----------------

You can find an Ansible collection of roles to help you install and configure NGINX Controller [here](https://github.com/nginxinc/ansible-collection-nginx_controller)

License
-------

[Apache License, Version 2.0](LICENSE)

Author Information
------------------

[Daniel Edgar](https://github.com/aknot242)

&copy; [F5 Networks, Inc.](https://www.f5.com/) 2020
