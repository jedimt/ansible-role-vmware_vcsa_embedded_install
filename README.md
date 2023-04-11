Ansible Role: VMware VCSA Embedded Install
=========

Installs a VMware VCSA to a ESXi host.

Requirements
------------

None.

Role Variables
--------------

Variables for this role are pulled the defaults/main.yml file.

Additional variables for the VCSA deployment itself can either be put in the defaults/main.yml file or you can put deployment specific variables in
your host inventory like so:

    # VMware Bare metal
    vmware:
      hosts:
        server-09.tme.nebulon.com:
        server-10.tme.nebulon.com:
        server-11.tme.nebulon.com:
        server-12.tme.nebulon.com:
      vars:
        vcsa_name: "devvcsa.tme.nebulon.com"
        vcsa_size: "small"
        vcenter_ip: 10.100.24.31
        vcsa_gw: 10.100.24.1
        vcsa_cidr: 22
        vcsa_dns: 10.100.72.10
        vcsa_ntp: 10.100.72.10
        vcsa_sso_domain: "vsphere.local"
        vcsa_sso_password: "{{ vault_vcsa_sso_password }}"
        vcsa_password: "{{ vault_vcsa_sso_password }}"
        vcsa_network: "VM Network"
    # VCSA
    vcsa:
      hosts:
        devvcsa.tme.nebulon.com:

Dependencies
------------

None.

Example Playbook
----------------

    # ===========================================================================
    # Deploy embedded VCSA to ESXi host
    # ===========================================================================
    - name: Deploy a self-hosted VCSA on ESXi host
      hosts: ['vmware[0]']
      gather_facts: true
      tags: play_vcsa_install

      vars_files:
        # Ansible vault with all required passwords
        - "../../credentials.yml"

      roles:
        - { role: jedimt.vmware_vcsa_embedded_install,
            vsphere_version: "7" }

    - name: Wait for VCSA installation to complete
      hosts: vcsa
      gather_facts: false
      tags: play_vcsa_install_complete

      vars:
        ansible_python_interpreter: /bin/python
        # Use ansible_password when no SSH keys installed on the VCSA.
        ansible_password: "{{ vault_esxi_password }}"

      vars_files:
        # Ansible vault with all required passwords
        - "../../credentials.yml"

      tasks:

        - name: Check for successful VCSA deployment
          ansible.builtin.command:
            ls /var/log/firstboot/succeeded
          register: vcsa_status
          retries: 30
          delay: 20
          until: vcsa_status is success
          changed_when: false
          tags: vcsa_test

License
-------

MIT

Author Information
------------------

Aaron Patten
aaronpatten@gmail.com
