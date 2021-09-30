# Project-1-Robbie-Drescher
Project 1 was an assignment given out in my Cyber Security class at University of Richmond.
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Diagram](https://github.com/RobDresch/Project-1-Robbie-Drescher/blob/main/Images/Copy%20of%20Diagram%20HW.jpg)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

  ---
        - name: Config Web VM with Docker
          hosts: elk
          become: true
          tasks:
          - name: Uninstall apache2
            apt:
              name: apache2
              state: absent
          - name: docker.io
            apt:
              force_apt_get: yes
              update_cache: yes
              name: docker.io
              state: present
          - name: Install pip3
            apt:
              force_apt_get: yes
              name: python3-pip
              state: present
          - name: Install docker python module
            pip:
              name: docker
              state: present
          - name: Increase Virtual Memory
            command: sysctl -w vm.max_map_count=262144
          - name: use more memory
            sysctl:
              name: vm.max_map_count
              value: 262144
              state: present
              reload: yes
          - name: Download and launch the docker web container
            docker_container:
              name: Elk
              image: sebp/elk:761
              state: started
              restart_policy: always
              published_ports:
                - '5601:5601'
                - '9200:9200'
                - '5044:5044'
          - name: Enable Docker Service
            systemd:
              name: docker
              enabled: yes
-



    ---
  
      - name: Installing and launch filebeat and metricbeats
         hosts: webservers
         become: yes
         tasks:
      - name: download filebeat deb
        command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb
      - name: Install filebeat deb
        command: sudo dpkg -i filebeat-7.6.1-amd64.deb
      - name: drop in filebeat.yml
        copy:
         src: /etc/ansible/filebeat-config.yml
         dest: /etc/filebeat/filebeat.yml
      - name: enable and configure system module
         command: filebeat modules enable system
      - name: setup filebeat
         command: filebeat setup
      - name: start filebeat service
         command: service filebeat start
      - name: enable service filebeat on boot
         systemd:
         name: filebeat
         enabled: yes
      - name: download metricbeat
         command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb
      - name: install metricbeat
         command: sudo dpkg -i metricbeat-7.6.1-amd64.deb
      - name: drop in metricbeat.yml
        copy:
         src: /etc/ansible/metricbeat-config.yml
         dest: /etc/metricbeat/metricbeat.yml
      - name: enable and configure docker module
        command: metricbeat modules enable docker
      - name: setup metricbeat
        command: metricbeat setup
      - name: start metricbeat
        command: metricbeat -e
      - name: enable service metricbeat on boot
        systemd:
        name: metricbeat
        enabled: yes




This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.

- Using a loadbalancer is an easy way to protect the availability of the DVMA site by splitting the load between two servers (Web-1 / Web-2) in case one of the servers goes down or is overloaded with data. As well as the loadbalancer, there is a jumpbox which is used to access the servers. The jumpbox is public but can only be accessed by an ssh key.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the files and system logs.

- Filebeat is an extension that sends logs of data to Kibana, which is a tool to easily organize and show data.
Metricbeat is an extension that logs metric data for public servers (ie. metric data for docker containers)

The configuration details of each machine may be found below.

| Name    | Function   | IP Address      | Operating System |
|---------|------------|-----------------|------------------|
| Jumpbox | Gateway    | 137.135.108.249 | Linux (Ubuntu)   |
| Web-1   | Web Server | 10.0.0.5        | Linux (Ubuntu)   |
| Web-2   | Web Server | 10.0.0.7        | Linux (Ubuntu)   |
| P1-VM   | Log Server | 10.1.0.4        | Linux (Ubuntu)   |


### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the jumpbox machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:

- 73.40.76.123

Machines within the network can only be accessed by the jumpbox.
Access to the machines are only allowed via ssh connection s

 A summary of the access policies in place can be found in the table below.

| Name    | Public Accessible | Whitelisted IP Addresses |
|---------|-------------------|--------------------------|
| Jumpbox | Yes               | 73.40.76.123             |
| Web-1   | No                | 10.0.0.4                 |
| Web-2   | No                | 10.0.0.4                 |
| P1-VM   | No                | 10.0.0.4                 |


### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
This allows the automation of multiple machines to be deployed and synced with the rest of the machines. Also it makes restarts and resets much easier to configure.

The playbook implements the following tasks:
- Install Docker
- Update Cache
- Install Python / Module
- Increase memory size
- Download and Execute

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.



### Target Machines & Beats
This ELK server is configured to monitor the following machines:

- Web-1 (10.0.0.5)

- Web-2 (10.0.0.7)

We have installed the following Beats on these machines:

- Filebeat

- Metricbeat

These Beats allow us to collect the following information from each machine:

- Filebeat - logs syslog data (i.e - when the machines files are edited)

- Metricbeat - logs metrics from docker machines (i.e - data sent to machine)

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:

- Copy the ELK file to /etc/ansible.

- Update the hosts file to include servers for playbook

- Run the playbook, and navigate to http://*YOUR-IP*:5601 to check that the installation worked as expected.

