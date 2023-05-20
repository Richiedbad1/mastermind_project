## Using Ansible Roles: Getting Started

### Ansible roles are used to break a lengthy playbook into multiple reuseable components. Instead of having a main playbook with 100s of lines of code, we can write the same playbook in smaller chunks.
### Roles are not playbooks. Roles are small functionality which can be independently used but have to be used within playbooks.


### In this article we are going to do the following:
- Write a role to install apache. And another role to copy a default html page to the default apache webpage. 
- Write a handler that helps you restart apache.
- write a role to install terraform
- write a role to install docker and run an ubuntu container with it. Using ansible.
- write a role to install jenkins and open port 8080 with firewall.


We will be using 4 different remote servers, one for each roles specified i.e Apache, Docker, Terraform and Jenkins. 

- Create an inventory file to house the ip addresses of these servers and name them appropraitely
```
vim inventory
```
```
[Apache]
XXX.XXX.XXX

[Jenkins]
XXX.XX.XXXX.XX

[Terraform]
XXXX.XXX.XX

[Docker]
XXX.XXX.XX
```
##### Initial Playbook Set up
- create  a playbook to house all the tasks/plays you want to run
```
vim Roles.yml
```

- Specify hosts and escalate commands in the playbook and write a play to update the index repository on all servers.
```
- hosts: all
  gather_facts: no
  become: true
  tasks:
    - name: update index repo
      package:
        update_cache: yes
```

- Then we specify the commands to create each role.
```
- hosts: Apache
  gather_facts: no
  become: true
  roles:
    - apache

- hosts: Docker
  gather_facts: no
  become: true
  roles:
    - Docker

- hosts: Terraform
  gather_facts: no
  become: true
  roles:
    - Terraform

- hosts: Jenkins
  gather_facts: no
  become: true
  roles:
    - Jenkins
```

- After this, create a new folder called [roles] and cd into that folder
```
mkdir roles, cd roles
```
- Create 4 new folders to match the names of the roles you wrote in the initial playbook
```
mkdir Apache, mkdir Jenkins, mkdir Terraform, mkdir Docker
```

- in each of the newly created folders, create a folder called tasks.
```
cd Apache, mkdir tasks, cd ..
cd Jenkins, mkdir tasks, cd ..
cd Terraform, mkdir tasks, cd ..
cd Docker, mkdir tasks, cd ..
```
- create a new file called ```main.yml``` in each tasks folder in each role.
```
touch Apache/tasks/main.yml
touch Jenkins/tasks/main.yml
touch Terraform/tasks/main.yml
touch Docker/tasks/main.yml
```
- Enter the ```main.yml``` file in the Apache role and write the requisite code.
```
vim Apache/tasks/main.yml
```
- In the yml file:
``` yml
- name: install apache/httpd on servers
  package:
    name:
      - apache2
    state: latest
    update_cache: yes
    notify: restart_apache

- name: copy local file to servers
  copy:
    src: default.html
    dest: /var/www/html/index.html
    owner: root
    groups: root
    mode: 0644
```
- create a files folder in the apache role to house any file we want to copy. Ensure it is in the Apache role and not in the tasks folder.
```
cd Apache, mkdir files, cd ..
```
- create your files in the files folder e.g default.html
```
touch Apache/files/default.html
```
- Edit our default.html page
```
vim Apache/files/default.html
```

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ansible is FUN</title>
</head>
<body>
    <h1>WELCOME TO HOMEPAGE</h1>
</body>
</html>
```
- Lastly we have to create a handlers folder, just like we created a files folder.
```
cd Apache, mkdir handlers, cd ..
```
- And also create a ```main.yml``` file in the handlers folder.
```
touch Apache/handlers/main.yml
```
- Enter the main.yml file and type in your code.
```
- name: restart_apache
  service:
    name: apache2
    state: restarted
```
- Now that we are done with Apache, we can edit the main ```roles.yml``` file to comment out or remove every code that isn't relevant to the Apache role yet so we can test our Apache role, we will add these code later as we create other roles.
```
- hosts: all
  gather_facts: no
  become: true
  tasks:
    - name: update index repo
      package:
        update_cache: yes

- hosts: Apache
  gather_facts: no
  become: true
  roles:
    - apache
```

- Now try to run the playbook:
```
ansible-playbook roles.yml
```


