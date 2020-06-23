advanced_ansible_web
====================

I've set this up to use independant roles for each part of the deployment. This allows the code to be reusable on your own servers.

ansible.cfg setup
=================

Add the following into your ````ansible.cfg```` configuration file:

````
ansible_managed = # Ansible managed file. DO NOT EDIT.

````

This will set the template header in the .j2 templates.

deploy.yaml
===========

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
