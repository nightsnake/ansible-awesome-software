## Ansible role for install awesome software pack

All the defaults vars optimized for Mate 19.3 (based on Ubuntu 18.04). Please be careful
This role created mostly for Lenovo Thinkpad laptop. Please see the repo and package lists for details.


### Tags
- `awesome_repo` - add a few repos (please see the defaults)
- `awesome_pkg_install` - install additional software (please see the defaults)


### Example playbook
```
# Play common roles
- hosts: all
  become: yes
  gather_facts: yes
  roles:
    - ansible-awesome-software
  tags: desktop
```
