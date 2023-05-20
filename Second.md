### In this article, we are going to install NGINX, APACHE2 AND PHP on different servers using Ansible.


- We will be making use of 3 ubuntu servers and we'll group their ip addresses under a name called [webservers] in the inventory file and 1 centos server which we will name [centos] in the inventory file. We will also name all the ip addresses [parent] so we can run plays on all the available servers at once.

We will then:
- Write a playbook to install apache on the centos machine using the group
- Also write a task to make sure apache is started
- In the same playbook write a task that installs nginx on the ubuntu group
- Also write a task to make sure nginx is started
- Lastly create a parent group in the inventory file and put both the webservers and centos group under it then write another task that helps to update the ubuntu server and centos server using the group you created. And also installs php and run it as a pre task

**N.B: We will be using 4 digital ocean droplets(3 ubuntu and 1 centos) as remote servers**

- In your control node, create an inventory file to house all the ip addresses of the remote servers.

```
vim inventory
```

- write all the ip addresses and name them appropraitely.

```
[centos]
centos_server_ip_address

[webserver]
webserver_ip1
webserver_ip2
webserver_ip3

[parent]
centos_server_ip_address
webserver_ip1
webserver_ip2
webserver_ip3
```

- write the playbook to run our plays

```
vim tasks.yml
```

- now specify hosts, escalate commands and write your plays

```
- hosts: webserver
  become: yes
  tasks:
    - name: install nginx in ubuntu servers
      apt:
        name: nginx
        state: latest

    - name: ensure nginx service is started
      service:
        name: nginx
        enabled: yes
        state: started
      when: ansible_distribution == "Ubuntu"

- hosts: centos
  become: yes
  tasks:
    - name: install apache on centos servers
      dnf:
        name: httpd
        state: latest

    - name: ensure apache service is started
      service:
        name: httpd
        enabled: yes
        state: started
      when: ansible_distribution == "CentOS"

- hosts: parent
  gather_facts: no
  become: yes
  pre_tasks:
    - name: Install php and update repo
      package:
        name: php
        state: latest
        update_cache: yes

```

- Ensure your crosscheck every line of code to eliminate errors

- Now you can run the playbook
```
ansible-playbook tasks.yml
```