# App Deployment to Azure Using Ansible

## Introduction

Ansible is an open-source tool for orchestration, deployment, configuration, automation and management of IT infrastructure which I will be using to deploy a Nodejs App in Azure.

## Configuration

I will use my local dev environment as my **control node** where I'll execute Ansible playbooks and configure one of my **remote nodes** in Azure. I've created a resource group in Azure (**Ansible-VMs**) and instantiated two Virtual Machines (**Ansible-Replica-1 and Ansible-Replica-2**) which will be running in my VNet.

![image-1](./images/image-1.png)

I will add the Public IPs to my **hosts** files and write the playbook.

![image-2](./images/image-2.png)

## NodeJS end-to-end Deployment

I will start writing the playbook yaml file which will configure the remote server in Azure.

The first thing will be to install **nodejs** and node package manager **npm** in the remote.

```yaml
---
- name: Install node and npm
  hosts: 51.140.136.152  # Replica 2
  tasks:
    - name: Update apt repo and cache
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600
    - name: Install nodejs and npm
      apt:
        pkg:
          - nodejs
          - npm
    - name: Install latest version of nodejs
      apt:
        name: nodejs
        state: latest

```

Next I will build the app locally and an artifact archive will be created (nodejs-app-1.0.0.tgz). I will copy this archive to my home folder in the remote server and unpack it, subsequent to which I'll install dependencies which are in the unpacked **package** folder. Then I'll start the app.

```yaml
- name: Deploy nodejs app
  hosts: 51.140.136.152
  tasks:
    - name: Copy nodejs folder to server
      copy:
        src: /Users/apple/Documents/Ansible-Revision/node-app-master/nodejs-app-1.0.0.tgz
        dest: /home/josembi
    - name: Unpack nodejs file
      unarchive:
        src: /home/josembi/nodejs-app-1.0.0.tgz
        dest: /home/josembi
        remote_src: yes
    - name: Install dependencies
      npm:
        path: /home/josembi/package
    - name: Start application
      command:
        chdir: /home/josembi/package/app
        cmd: node server
      async: 1000
      poll: 0
```

![image-3](./images/image-3.png)

Ansible has run successfully, updating, installing Nodejs and npm and finally deploying the app.

In the terminal window of the Azure webserver, I see that the server is running. I will add register and debug modules to my playbook to enable Ansible provide this information instead of logging in the remote server and checking that out.

![image-4](./images/image-4.png)

```yaml
....*code truncated*....
    - name: Ensure app is running
      shell: ps aux | grep node
      register: app_status
    - debug: msg={{app_status.stdout_lines}}
```

![image-5](./images/image-5.png)

