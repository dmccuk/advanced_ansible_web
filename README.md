Advanced ansible
================

I've set this up to use independant roles for each part of the deployment. This allows the code to be reusable on your own servers.

To use this code, first clone the repo:
````
$ git clone https://github.com/dmccuk/advanced_ansible_web.git
````
then, create the following files:

 * ansible.cfg
 * deploy.yaml
 * host inventory (you will need to supply this)

Setup them up as below (an example playbook run using the roles is at the bottom of the page):

### ansible.cfg setup
Add the following into your ````ansible.cfg```` configuration file under [default]:

````
ansible_managed = # Ansible managed file. DO NOT EDIT.
````

This will set the template header in the .j2 templates.

### deploy.yaml
To run the playbooks, setup the deploy.yaml like this:

<details>
 <summary>expand for output</summary>
  <p>
    
````
---
- hosts: all
  gather_facts: false # remove later! speeds up testing
  become: true
  roles:
    - advanced_ansible_web/common

- hosts: frontends
  gather_facts: false # remove later! speeds up testing
  become: true
  roles:
    - advanced_ansible_web/deploy_haproxy

- hosts: apps
  gather_facts: false
  become: true
  roles:
    - advanced_ansible_web/deploy_tomcat
    - advanced_ansible_web/deploy_apache

- hosts: appdbs
  become: true
  roles:
    - advanced_ansible_web/geerlingguy.postgresql

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

### Ansible run of the roles:
This was performed after a reboot of all the servers:

````
$ ansible -i hosts all -m shell -a 'uptime'
frontend1 | CHANGED | rc=0 >>
 09:45:20 up 4 min,  1 user,  load average: 0.03, 0.05, 0.03
appdb1 | CHANGED | rc=0 >>
 09:45:20 up 4 min,  1 user,  load average: 0.06, 0.12, 0.05
support1 | CHANGED | rc=0 >>
 09:45:20 up 4 min,  1 user,  load average: 0.00, 0.02, 0.02
app2 | CHANGED | rc=0 >>
 09:45:20 up 4 min,  1 user,  load average: 0.24, 0.21, 0.10
app1 | CHANGED | rc=0 >>
 09:45:21 up 4 min,  1 user,  load average: 0.04, 0.09, 0.05
````

<details>
 <summary>expand to see the run output</summary>
  <p>

````
$ ansible-playbook deploy.yml

PLAY [all] ************************************************************************************************************************

TASK [common : enable repos] ******************************************************************************************************
ok: [support1]
ok: [app1]
ok: [app2]
ok: [appdb1]
ok: [frontend1]

PLAY [frontends] ******************************************************************************************************************

TASK [deploy_haproxy : install HAProxy & HTTPD] ***********************************************************************************
ok: [frontend1]

TASK [deploy_haproxy : configure haproxy] *****************************************************************************************
ok: [frontend1]

PLAY [apps] ***********************************************************************************************************************

TASK [deploy_tomcat : install tomcat] *********************************************************************************************
ok: [app1]
ok: [app2]

TASK [deploy_tomcat : enable & start tomcat at boot] ******************************************************************************
ok: [app1]
ok: [app2]

TASK [deploy_tomcat : create ansible tomcat directory] ****************************************************************************
ok: [app1]
ok: [app2]

TASK [deploy_tomcat : create ansible tomcat directory] ****************************************************************************
ok: [app1]
ok: [app2]

TASK [deploy_tomcat : copy static index.html to tomcat webapps/ROOT/index.html] ***************************************************
ok: [app2]
ok: [app1]

TASK [deploy_tomcat : copy static index.html to tomcat webapps/ansible/index.html] ************************************************
ok: [app1]
ok: [app2]

TASK [deploy_apache : install apache] *********************************************************************************************
ok: [app1]
ok: [app2]

TASK [deploy_apache : enable apache at boot & Start] ******************************************************************************
ok: [app1]
ok: [app2]

PLAY [appdbs] *********************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : include_tasks] *************************************************************************************
included: /home/student1/web_role/roles/geerlingguy.postgresql/tasks/variables.yml for appdb1

TASK [geerlingguy.postgresql : Include OS-specific variables (Debian).] ***********************************************************
skipping: [appdb1]

TASK [geerlingguy.postgresql : Include OS-specific variables (RedHat).] ***********************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Include OS-specific variables (Fedora).] ***********************************************************
skipping: [appdb1]

TASK [geerlingguy.postgresql : Define postgresql_packages.] ***********************************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Define postgresql_version.] ************************************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Define postgresql_daemon.] *************************************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Define postgresql_data_dir.] ***********************************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Define postgresql_bin_path.] ***********************************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Define postgresql_config_path.] ********************************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Define postgresql_unix_socket_directories_mode.] ***************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : include_tasks] *************************************************************************************
included: /home/student1/web_role/roles/geerlingguy.postgresql/tasks/setup-RedHat.yml for appdb1

TASK [geerlingguy.postgresql : Ensure PostgreSQL packages are installed.] *********************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Ensure PostgreSQL Python libraries are installed.] *************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : include_tasks] *************************************************************************************
skipping: [appdb1]

TASK [geerlingguy.postgresql : include_tasks] *************************************************************************************
included: /home/student1/web_role/roles/geerlingguy.postgresql/tasks/initialize.yml for appdb1

TASK [geerlingguy.postgresql : Set PostgreSQL environment variables.] *************************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Ensure PostgreSQL data directory exists.] **********************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Check if PostgreSQL database is initialized.] ******************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Ensure PostgreSQL database is initialized.] ********************************************************
skipping: [appdb1]

TASK [geerlingguy.postgresql : include_tasks] *************************************************************************************
included: /home/student1/web_role/roles/geerlingguy.postgresql/tasks/configure.yml for appdb1

TASK [geerlingguy.postgresql : Configure global settings.] ************************************************************************
ok: [appdb1] => (item={u'option': u'unix_socket_directories', u'value': u'/var/run/postgresql'})

TASK [geerlingguy.postgresql : Configure host based authentication (if entries are configured).] **********************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Ensure PostgreSQL unix socket dirs exist.] *********************************************************
ok: [appdb1] => (item=/var/run/postgresql)

TASK [geerlingguy.postgresql : Ensure PostgreSQL is started and enabled on boot.] *************************************************
ok: [appdb1]

TASK [geerlingguy.postgresql : Ensure PostgreSQL users are present.] **************************************************************

TASK [geerlingguy.postgresql : Ensure PostgreSQL databases are present.] **********************************************************

PLAY RECAP ************************************************************************************************************************
app1                       : ok=9    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
app2                       : ok=9    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
appdb1                     : ok=23   changed=0    unreachable=0    failed=0    skipped=6    rescued=0    ignored=0
frontend1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
support1                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
````
</p></details>
