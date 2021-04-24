## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![image](https://user-images.githubusercontent.com/38510755/115951503-b13ca480-a4a6-11eb-8452-8c4abf99aba0.png)


These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing distributes network traffic across multiple servers, therefore improving applications responsivenses and reliability. 
Load balancers protect DDoS attacks by being able to shift the attack traffic. The advantage of having a jumpbox is that it gives access to the user from a single node that can be secured/monitored constantly. The JumpBox is also the origination point for launchign administrative tasks (SAW - Secure Admin Workstation), which means all administrators are required to connect to the Jumpbox before performing any tasks.


Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the logs and system traffic.
FIlebeat watches for log files/locations and collects log events which is important in monitoring data.
Metricbeat recods metric and statistical data from the operating system and from services that run on the server.

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway    | 10.0.0.1   | Linux   |
| WEB-2    | Web Server | 10.1.0.6   | Linux   |
| WEB-3    | Web Server | 10.1.1.6   | Linux   |
| ELK      | Monitoring | 10.2.0.4   | Linux   |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the JumpBox machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses listed below. 

Machines within the network can only be accessed by SSH.
The only machine able to connect to the Elk-Server is the JumpBox (10.0.0.1)

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes/No              | 10.0.0.1 10.0.0.2    |
| Web-2    |     No                |    10.1.0.6     |
| Web-3   |   No          |     10.1.1.6       |
| ELK   |  No             |10.2.0.4 & Personal IP |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because the main advantages of automating configuration through Ansible is the ease of use and an extremely easy learning curve. Through the use of Playbooks you are able to configure multiple Machines through the use of a single command after initial configuration. 

```
#ELK-PLAYBOOK 
----
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: RedAdmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes_
```

The playbook implements the following tasks:
- Create a New VM (should be named something simple "Elk-Server") Keep note of the Private IP (10.0.0.11) and the Public IP (0.0.0.0) you will need the Private IP to SSH into the VM and the Public IP to connect to the Kibana Portal (HTTP Site) to view all Metrics/Syslogs.
- Download and Configure the "elk-docker" container "In the hosts.conf you will need to add a new group [elkservers] and the Private IP (10.0.0.11) to the group. Then you need to create a new ansible-playbook (elk.yml) that will download, install, configures the "Elk-Server" to map the following ports [5601,9200,5044], and starts the container.
- Launch and expose the container "After installing and starting the new container. You can verify that the container is up and running by SSHing into the container from your JumpBox (SAW). Once you are in the [Elk-Server] run the command [sudo docker ps]
- Create new Inbound Security Rules to allow Ports: 5601 and 9200 "The Inbound Security Rules should allow access from your Personal Network"
- Open a new browser and type in the [Public IP:5601] to access the Kibana Portal Site
The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![image](https://user-images.githubusercontent.com/38510755/115951256-5a829b00-a4a5-11eb-9438-75f1c4197f0b.png)


### Target Machines & Beats
This ELK server is configured to monitor the following machines:
[10.1.0.6, 10.1.1.6, 10.2.0.4]

We have installed the following Beats on these machines:
[10.1.0.6, 10.1.1.6, 10.2.0.4]

These Beats allow us to collect the following information from each machine:
- Filebeat is a lightweight shipper for forwarding and centralizing log data. Filebeat monitors log files or locations you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
- Metricbeat collects metrics from the operating system and from services running on the server. Metricbeat then takes the metrics and statistics that it collects and ships them to the output that you specify.

### Using the Playbook
```
---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system

    # Use command module
  - name: Setup filebeat
    command: filebeat setup

    # Use command module
  - name: Start filebeat service
    command: service filebeat start

    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
 ```
 

In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the filebeat.yml file to /etc/ansible/roles/files/directory
- Update the configuration files to include the Private IP of the Elk-Server to the ElasticSearch and Kibana sections of the configuration file.
- Create a new ansible-playbook filebeat-playbook.yml in the /etc/ansible/roles/ directory that will install, drop in the filebeat.yml files from the /etc/ansible/roles/files/ directory, update the configuration files, and start the service for Filebeat
- Run the playbook, and navigate to ELK-Server to check that the installation worked as expected.
- The playbook is called filebeat-playbook.yml. You copy the file to the "/etc/ansible/hosts/" directory.
-The file you need to update is the filebeat.yml file which is a configuration file that will be dropped into the Elk-Server during the run of the ansible-playbook. When you update the host.cfg file in the ansible directory you will need to create a new group called [elkservers] and add the Private IP of the Elk-Server to the group. Then when configuring the filebeat.yml file you need to designate the Private IP of the Elk-Server in two lines of the .yml file. Lines 1106 and 1806 are the needed to be updated with the Private IP.
![image](https://user-images.githubusercontent.com/38510755/115951459-6e7acc80-a4a6-11eb-8fac-e2684ac391ea.png)

URL to check that the ELK server is running: http:[ELK server ip]/app/kibana

