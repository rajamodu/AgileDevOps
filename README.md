# AgileDevOps Solution - Version: 1.2.2

# Slack Channel
https://agiledevopssolution.slack.com/messages/CE7RU725C/apps/AE694A2F6/

Available: Ubuntu 18.04.1 LTS bionic

Requirements: Virtual Machine for Tests with 3GB disk Space and 1GB RAM.

# OVERVIEW

The AgileDevOps solution aims to enable the instant creation of a test environment for provisioning services/micro-services for the practice of everyday actions.

The current version provides the instant creation of Docker Swarm Manager monitored by Giropops-monitoring (Netdata + Grafana + Prometheus + Alertmanager + node-exporter) with containers running the nodejs application "hello.js", which basically says the name of the container that is being triggered. Routed via reverse proxy and balanced by Traefik. 

NOTE: This solution only creates the Swarm Managers, the deployment of the Swarm Workers will be created in future versions. NOTE: The Jenkins build process is still just a simulation, I'm also working on deploying this feature. Helps are very welcome! : D

AUTOMATIZED PROCEDURE: Starts Ansible Playbook bootstrap.yml ... >>
Installing all softwares used in Solution (git, (jenkins), Docker-ce ... >> Configuring Docker Swarm Manager ... >> Starting Playbook firstdeploy.yml ... >> Starting AgileDevOps Solution ... >> Git> Jenkins Simulated Task> Docker> Swarm Cluster> Traefik (Balancer + Reverse Proxy + Logs) >> Nodejs APP Starting ... OK! \ O /: D
 
![alt text](https://raw.githubusercontent.com/danielprietsch/AgileDevOps/master/draw.png)


# Expectative from 2.0 version, the user will select which orchestrator to use in their tests and studies. Can you help us get there?

![alt text](https://raw.githubusercontent.com/danielprietsch/AgileDevOps/master/draw2.0.png)


============================

# VM 1 = Docker Swarm Manager
# VM 2 = Ansible Controller


# STEPS:

#  On the VM 1 (Docker Swarm Manager)

1- Create the virtual machine (who will be the swarm manager server for the AgileDevOps Solution) on your Partner or locally (Azure, AWS, Vagrant, VirtualBox, etc...):

2- Create the user "vagrant" and a password for SSH access and verify if it has IP connectivity and redirect inbound ports to this server/VM:

22 (ssh),

443 (https),

80 (http),

8081 (traefik panel),

3000 (Grafana),

9100 (node-exporter),

9090 (Prometheus),

8080 (cadvisor),

9093 (alertmanager). 

3 - Enabled the ssh Password authentication:

NOTE: You can configure your SSH KEYs if you want.

Uncomment this line:

    $ vi /etc/ssh/sshd_config
    
    PasswordAuthentication yes

3.1 - Install Python for the Ansible can work

    $ apt-get install python

# On the VM2 (your Ansible Controller Host)

4- Install Ansible:

    $ sudo apt-get update
    $ sudo apt-get install software-properties-common
    $ sudo apt-add-repository ppa:ansible/ansible
    $ sudo apt-get update
    $ sudo apt-get install ansible
    
4.1 - Install pip and python libraries:

    $ apt-get install python-pip
    $ pip install jsondiff
    $ pip install pyyaml
    $ pip install docker-py

    
5- Download the git repository AgileDevOps Solution in your Linux /tmp:

    $ git clone https://github.com/danielprietsch/AgileDevOps.git /tmp/AgileDevOps

6- Edit and save the file /tmp/AgileDevOps/ansible/hosts and include your VM Machine IP:

    $ vi /tmp/AgileDevOps/ansible/hosts

    [agiledevops]
    #PUT THE VM2 IP HERE

    Example:
    [agiledevops]
    192.168.20.21

7 - Go to /tmp/AgileDevOps

    $ cd /tmp/AgileDevOps

8 - Run the playbook bootstrap.yml with the parameters above:

# ANSIBLE-VAULT PASSWORD = 123456 

     $ ansible-playbook -i ansible/hosts ansible/playbooks/bootstrap.yml --ask-pass --ask-vault-pass -u vagrant

# IMPORTANT: Wait for the docker images to download, this may take about 1-5 minutes depending on your connection.

# TEST NOW! \o/

# nodejs App

    http://VM2ip (press F5 to view the Traefik Load Balancing MAGIC HAPPENS!
    http://VM2ip:8080 (Traefik Panel)
 
# View ALERTS ON SLACK CHANNEL #alerts:

    https://agiledevopssolution.slack.com/messages/CE5MSRMFS/apps/AE694A2F6/

# To access Prometheus interface on browser:

    http://VM2ip:9090

# To access AlertManager interface on browser:

    http://VM2ip:9093

# To access Grafana interface on browser:
# user: admin
# passwd: giropops
    http://VM2ip:3000

To add plugs edit file giropops-monitoring/grafana.config
GF_INSTALL_PLUGINS=plug1,plug2
Current plugs grafana-clock-panel,grafana-piechart-panel,camptocamp-prometheus-alertmanager-datasource,vonage-status-panel
Have fun, access the dashboards! ;)

# To access Netdata interface on browser:

    http://VM2ip:19999

# To access Prometheus Node_exporter metrics on browser:

    http://YOUR_IP:9100/metrics

# docker service rm giropops_node-exporter

Wait some seconds and you will see the integration works fine! Prometheus alerting the AlertManager that alert the Slack that shows it to you! It's so easy and that simple! :D

# Of course, create new alerts on Prometheus:

    $ vim conf/prometheus/alert.rules


# OPTIONAL:

9 - To run another deploy after the bootstrap.yml, run only the deploy.yml playbook:

    $ ansible-playbook -i ansible/hosts ansible/playbooks/deploy.yml --ask-pass -u vagrant

10 - To perform the rollback to the previous version of the NodeApp application in the GIT (HEAD~), run the playbook below:

    $ ansible-playbook -i ansible/hosts ansible/playbooks/rollback.yml --ask-pass -u vagrant

11 - To permanent scale the nodejs container to 10 instances:
Edit the /tmp/AgileDevOps/docker/dockerfiles/AgileDevOps/docker-compose.yml

    $ vi /tmp/AgileDevOps/docker/dockerfiles/AgileDevOps/docker-compose.yml

Change the line:

    $ replicas: 2 
to

    $ replicas: 10 
    
... and run the STEP #9 again
    
    $ ansible-playbook -i ansible/hosts ansible/playbooks/deploy.yml --ask-pass -u vagrant

12 - You can set up the role  geerlingguy.ntp to change the timezone and configure to America/Sao_Paulo on the ansible/playbooks/bootstrap.yml by uncomment this role;

====================================================

# EXTRA INFO:
  
Tree Directoryies:

/AgileDevOps
...

    Vagrantfile: My Vagrantfile used on tests;

    ansible: 
        ansible.cfg: (my ansible.cfg config ((extra))
        hosts: Ansible hosts inventory 
        files:
            99force-ipv4: Used to force vagrant using ipv4 on apt-get
            postfixconfig:
               main.cf: postfix config file
               sasl:
                  sasl_passwd: Used for postfix smtp relay
                  sasl_passwd.db: Used for postfix smtp relay
                  
        scripts:
            aptkey.sh: Used on docker installation
            testservices.sh: Using to test docker swarm and docker pid to restart
        
        roles:
             
        playbooks:
            bootstrap.yml:
            firstdeploy.yml: 
            deploy.yml: same as firstdeploy.yml but you can run only deploy ;
            rollback.yml: Rollback App to HEAD~ git version;
        vault: 
                secret.yml: have the password to postfix smtp relay with desafiolinux@gmail.com
    docker:
        dockerfiles:
            AgileDevOps:
                config.toml: Traefik config file
                docker-compose.yml: docker-compose file to creates all the Environment
            NodeApp:
                Dockerfile: Used to build the image, see this on https://hub.docker.com/r/danielprietsch/nodejs/
                node.sh: Script started by the image
            giropops-monitoring: Doc >> https://github.com/badtuxx/giropops-monitoring
                
        volumes:
            Jenkins: Simulating the Jenkins build
            NodeApp: Receiveis the build and stable version of NodeApp from Jenkins
                 hello.js: Nodejs + Express App;
                 package.json: Including express depencies to npm/yarn install;
                 node_modules: All modules downloaded by npm/yarn
 
 
 # Author: Daniel Prietsch

 daniel@nuvemtecnologia.com

 http://nuvemtecnologia.com
 

 # Need help with:
 
- Copy the Traefik logs to volume in Linux (not in container)
- Parse Traefik logs?
- Creating a real Jenkins test.

# Many Thanks to:

Jefferson (LinuxTips): by the giropops-monitoring (https://github.com/badtuxx/giropops-monitoring)

Michael Hart: by the nodejs image (https://github.com/mhart/alpine-node/issues)


# Updates:

1.1: NODEJS Application "hello.js" ( https://"SWARM-MANAGER-IP" ) Running with a invalid crt.

1.2: Ansible codes using docker modules.

1.2.1: Adjusting role bugs, and added netdata dynamic installation with netdata.conf.j2.

1.2.2: Giropops-monitoring added, but the graphics off node-exporter on grafana not working.

