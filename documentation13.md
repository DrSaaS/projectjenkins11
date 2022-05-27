### INTRODUCING DYNAMIC ASSIGNMENT  
In my projectjenkins11 github repository, I started a new branch named dynamic-assignments

```
git checkout -b dynamic-assignments
```
### I created a new folder and it dynamic-assignments. Then inside this folder, I created a new file and name it env-vars.yml. We will instruct site.yml to include this playbook later. For now, let us keep building up the structure.

```
mkdir dynamic-assignments && touch dynamic-assignments/env-vars.yml
```

### Next is to configure ansible to use multiple environments. It is neccessary to create a folder to keep each environment's variable files.  

### I created a new folder env-vars and inside it, for each environment, new YAML files to set up variables

```
mkdir env-vars && touch env-vars/dev.yml env-vars/stage.yml env-vars/uat.yml env-vars/prod.yml
```  

### I posted the following code into env-vars.yaml

```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always

```  

### Next, I updated site.yml with dynamic assignments as below  

```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml

```  


### Next I downloaded and installed a MySQL role developed by geerlingguy

### On my ansible-jenkins server, I checked if git was installed 

```
git --version
which git
```
![git installed check](./images/git-installed.JPG)  

### Then I went to the ansible-config-artifact directory and ran the code below

```
git init
git pull https://github.com/DrSaas/projectjenkins11.git
git remote add origin https://github.com/DrSaas/projwctjenkins11.git
git branch roles-feature
git switch roles-feature
```


### Inside roles directory,  I created a new MySQL role with ansible-galaxy install geerlingguy.mysql and renamed the folder to mysql
```
ansible-galaxy install geerlingguy.mysql

mv geerlingguy.mysql/ mysql
```

Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

Now it is time to upload the changes to GitHub:
```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```

### Unable to push to git as username/password prompt failed. Password authentication disabled since 2021.

###  I created a personal access token in git and used it as the password with expiry set to 90 days

- settings 
  -developer
    - Personal access tokens
       -Generate new token
          - I gave a descriptive name
          -set expiration to 90 days
          -selected the scope (repo, admin:repo_hook, delete_repo)  

- I generated the token and used it as a password.
- I was able to push successfully.

Username: my_username
Password: my_token

### The next
Inside roles directory create your new MySQL role with 
```
ansible-galaxy install geerlingguy.mysql
```
 and rename the folder to mysql

```
mv geerlingguy.mysql/ mysql
```
Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

Now it is time to upload the changes into your GitHub:

git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature

### I edited mysql?defaults>main.yml file
```
# Databases.
mysql_databases:
  - name: tooling
    collation: utf8_general_ci
    encoding: utf8
    replicate: 1

# Users.
mysql_users: 
  - name: webaccess
    host: 0.0.0.0
    password: secret
    priv: '*.*:ALL,GRANT'

```


### Load Balancer Roles
### Install Nginx role

```
ansible-galaxy install geerlingguy.nginx

mv geerlingguy.nginx/ nginx
```


```
ansible-galaxy install geerlingguy.apache

mv geerlingguy.apache/ apache
```

### Next was to configure the Nginx role  - nginx>default>main.yml
### First i edited the inventory>uat.yml file

```

[nfs]
172.31.29.13

[webservers]
172.31.26.192 
172.31.26.64 

[db]
172.31.20.208

[lb]
172.31.30.113


```

### I made changes to nginx > default > main.yml

```
nginx_upstreams: 
- name: myapp1
  strategy: "ip_hash" # "least_conn", etc.
  keepalive: 16 # optional
  servers:
    - "172.31.26.192 weight=3"
    - "172.31.26.64 weight=3"
```

```
nginx_extra_http_options: 
Example extra http options, printed inside the main server http config:
   nginx_extra_http_options: |
     proxy_buffering    off;
     proxy_set_header   X-Real-IP $remote_addr;
     proxy_set_header   X-Scheme $scheme;
     proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header   Host $http_host;

```


### In nginx > tasks > main.yml, add become:true for #Nginx setup

```
# Nginx setup.
- name: Copy nginx configuration in place.
  become: true
  template:
    src: "{{ nginx_conf_template }}"
    dest: "{{ nginx_conf_file_path }}"
    owner: root
    group: "{{ root_group }}"
    mode: 0644
  notify:
    - reload nginx

- name: Ensure nginx service is running as configured.
  become: true
  service:
    name: nginx
    state: "{{ nginx_service_state }}"
    enabled: "{{ nginx_service_enabled }}"


```








