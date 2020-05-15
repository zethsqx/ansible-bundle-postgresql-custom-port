# Ansible Tower Bundle fix custom postgresql port

For more information about Ansible Tower Bundle installation, 
visit -
https://docs.ansible.com/ansible-tower/latest/html/quickinstall/download_tower.html

The bundled installer only supports Red Hat Enterprise Linux and CentOS.

To download the latest version of the bundled Tower installation program directly visit: https://releases.ansible.com/ansible-tower/setup-bundle/

Next, select the installation program which matches your distribution (el7):

```
ansible-tower-setup-bundle-latest.el7.tar.gz
```

## Getting Started 

The provided Ansible Tower bundle version is 3.6.4-1. It might work if you replace the specify files or manually run through to edit for other versions.

Fixes are detailed below for reference.


## Postgresql Fixes 

**Overview of the Postgresql problems:**
1. Port was hardcoded to 5432
2. All the psql command did not specify port parameter
3. Semanage did not apply for new port
4. Pgsql module not using port parameter, which will default back to 5432
5. Postgresql conf template not using user defined port


@ansible-tower-setup-bundle-3.6.4-1/roles/postgres/tasks/main.yml
```
1) Added semanage in order to apply the custom port
```

@ansible-tower-setup-bundle-3.6.4-1/roles/postgres/tasks/init.yml
```
1) Added a block of extra code to reapply correct port conf and restart db after init
  a) check listening postmaster
  b) reapply correct port and restart if postmaster on default port
2) Added pg_port to wait_for module
```

@ansible-tower-setup-bundle-3.6.4-1/roles/postgres/tasks/init.yml
```
1) Added pg_port to wait_for module
2) Added -p parameter to ALL the pgsql cmd
3) Added pg_port for postgresql_db module
```

@ansible-tower-setup-bundle-3.6.4-1/roles/postgres/templates/postgresql.conf.j2
```
1) Added port = {{ pg_port }} so conf template will use the user defined port
```

## Other Fixes

**NGINX**
1. Services not able to restart as port is used
2. Cert unable to copied to other tower in cluster

@ansible-tower-setup-bundle-3.6.4-1.el7/roles/nginx/tasks/tasks.yml
```
1) Added condition to copy to only tower in group and ignore first tower
2) Added to install nginx conf to all towers
3) Replace last block of nginx to force kill so that restart can happen
```

@ansible-tower-setup-bundle-3.6.3-1.el7/roles/nginx/handlers/main.yml
```
1) Replace whole yml to kill ps and restart nginx
```

**Restore DB**
1. Using wrong port 

@ansible-tower-setup-bundle-3.6.4-1.el7/restore.yml
```
1) Added pg_port to postgresql_user module
2) Replace nginx block to force kill so that restart can happen
```




