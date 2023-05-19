## Instaling Apache and starting services in different servers using Ansible playbook

- First ensure the servers are properly set up and connected to the control node.

- Generate ssh key on control node to ensure more secure connection. we specify the type (-t) as ed25519 and named the key(-C) "ansible". You can name it whatever you want but we used "ansible" here
```
ssh-keygen -t ed25519 -C "ansible"
```

- Copy the ssh public key from the control node to the remote servers
``` 
cat ~/.ssh/ansible.pub | ssh root@ip_address_of_remote_server "cat - >> ~/.ssh/authorized_keys"
 ```
 - Now you can log in to the remote servers from the control node with:
 ```
ssh -i ~/.ssh/ansible.pub user@ip_address
 ```

# TASK 1
 - Install ansible on control node:
 ```
 - N.B: we are using an EC2 instance as a control node and digital ocean droplets as remote servers
sudo apt update, sudo apt install ansible
 ```
- Create an inventory file to host remote servers' ip addresses
```
vim inventory
```

- Add the ip addresses of all relevant remote servers. Example:

```
190.667.667.12
456.765.234.112
```
 - Test ansible connectivity to all remote users
 ```
ansible all --key_file ~/.ssh/ansible -i inventory -m ping 
 ```

 - Create an ansible.cfg file to make commands shorter
 ```
 vim ansible.cfg
 ```

 - Add inventory file and key-file to ansible.cfg
```
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible
```

- Now try to check the connectivity again

```
ansible all -m ping
```

- Now that we are certain that we can connect to our remote servers and run ansible, we will create a playbook to install apache and other packages. Using playbooks makesnit easier to run commands that just typing commands line by line in the CLI. Playbooks use the YAML syntax so any playbook created must have a .yml or .yaml extension.
```
vim playbook.yml
```

- Write your code in the playbook. Just like a normal server, some commands will not run because they are elevated commands and they need "sudo". Something similar applies in ansible, it is called "become" and we have to specify it in our playbook.
```
- hosts: all
  become: yes
  tasks:
```

- to install certain packages, it is best you use a new user, so we will first create a user and then install all relevant packages with this user.

```
    - name: update repo
      package:
        update_cache: yes
    
    - name: Create a default user
      user: 
        name: default
        groups: root
```

- we also have to add the ansible ssh key to the user and add the user to the sudoers group to enable it run elevated commands with sudo
```
    - name: add key to user
      authorized_key:
        user: default
        key: ""

    - name: add user to sudoers to escalate permissions
      copy:
        src: default
        dest: /etc/sudoers.d/default
        owner: root
        group: root
        mode: 0440
```

- we have create a file called "default" to copy to /etc/sudoers.d/default
```
vim default
```

- Then we write the following in the default file, where ```default``` is the user name.
```
default ALL=(ALL:ALL) NOPASSWD:ALL
```

- Now run the playbook:
```
ansible-playbook playbook.yml
```

- After this you can add ```default``` as a remote user to ansible.cfg, so we can run subsequent tasks using the ```default``` user.
```
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible
remote_user = default
```

- once that is done you can add plays to install packages and start services in the playbook.yml

```
- hosts: all
  become: yes
  tasks:
    - name: install apache/httpd on servers
      package:
        name:
          - "{{ apache }}"
        state: latest
        update_cache: yes

    - name: ensure apache service is started
      service:
        name: apache2
        state: started
        enabled: yes
      when: ansible_distribution == "Ubuntu"

    - name: ensure httpd service is started
      service:
        name: httpd
        state: started
        enabled: yes
      when: ansible_distribution == "CentOS"

    - name: copy local file to servers
      copy:
        src: default.html
        dest: /var/www/html/index.html
        owner: root
        groups: root
        mode: 0644
```

- now create a ```default.html``` file to ensure you can copy files to remote servers.
```
nano default.html
```
- The content of the html page is:
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

- Now since we have 2 different server distributions i.e Ubuntu & CentOS, package names are different but we can install same package with different names in one play as seen in the above code. The last part is to specify the variables in the inventory file.
```
190.667.667.12 apache=apache2 #ubuntu server
456.765.234.112 apache=httpd #centos server

```
- Remember that you are to use your own ip addresses, the above ip addresses is merely for demonstration. you can also name your variables anything you want but it must follow the convention (variable_name=value)

- Now that we are done, ensure that there are no typo or indentation errors in your code.

- now you can run the playbook:
```
ansible-playbook playbook.yml
```

- Now check the ip addresses on your browser, it should display the html page like this:
![homepage](screenshots/homepage.jpg)




