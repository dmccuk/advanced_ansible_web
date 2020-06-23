Advanced ansible
================

I've set this up to use independant roles for each part of the deployment. This allows the code to be reusable on your own servers.


### ansible.cfg setup
Add the following into your ````ansible.cfg```` configuration file:

````
ansible_managed = # Ansible managed file. DO NOT EDIT.
````

This will set the template header in the .j2 templates.

### deploy.yaml
To run the playbooks, setup the deploy.yaml like this:

<details>
 <summary>deploy.yaml</summary>
  <p>
    
````
---
- hosts: all
  gather_facts: false # remove later! speeds up testing
  become: true
  roles:
    - common

- hosts: frontends
  gather_facts: false # remove later! speeds up testing
  become: true
  roles:
    - deploy_haproxy

- hosts: apps
  gather_facts: false
  become: true
  roles:
    - deploy_tomcat
    - deploy_apache

- hosts: appdbs
  become: true
  roles:
    - geerlingguy.postgresql
````
</p></details>

### Install galaxy role using requirements.yml
Example requirements file:
````
---
roles:
  # Install role from ansible-galaxy
  - name: geerlingguy.postgresql
    version: 2.2.1
````

### Example install
````
$ ansible-galaxy install -r requirements.yml
- downloading role 'postgresql', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-postgresql/archive/2.2.1.tar.gz
- extracting geerlingguy.postgresql to /home/student1/.ansible/roles/geerlingguy.postgresql
- geerlingguy.postgresql (2.2.1) was installed successfully

````
