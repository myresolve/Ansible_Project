# Ansible_Project
Ansible practice
  Install Ansible
  Configure Passwordless Authentication
  Use Ansible Adhoc Commands
  Group servers and execute task only on select servers 
  Write and Execute Ansible Playbook
  

Step1
#Open AWS EC2

  Launch Instance and apply these attributes:
    Name: "linux_master"
    OS: Ubuntu
    Instance type: t2.micro
    Create Key Pair
    Name: "linux_masterkey"
    Type: RSA
    Format: .pem
    Storage: 15 GB
    Copy IP Address of "linux_master" Instance

  Open MobaXterm Application (Allows you to access ip from your local system)
    Select Session - SSH - Input Copied IP Address - username: ubuntu
    Open Advanced SSH settings - use private key "linux_masterkey.pem"
    select connect
    
  Input the following commands in the terminal
    clear
    sudo apt update (system update)
    sudo apt upgrade (system upgrade)
    Y
    select "ok" on promt screen
    sudo nano /etc/hostname (open hostname file)
    change hostname of virtual machine to linuxmaster
    "ctrl + o" to save - press enter - "ctrl + x" to exit
    cat nano /etc/hostname (make sure changes were saved)
    sudo init 6 (reboot system after hostname change)
    type "R" (restart vm)
    sudo apt install ansible
    ansible --version (check version)

Step 2
Create another instance as an Agent for Ansible

  Launch new instance with the following attributes
    Name: ansible_master
    OS: Ubuntu
    Instance Type: t2.micro
    Key Pair: "linux_masterkey.pem"
    Storage: 15 GB
    Copy Public IP of ansible_master

  Open MobaXterm Application (Allows you to access ip from your local system)
    Select Session - SSH - Input Copied IP Address - username: ubuntu
    Open Advanced SSH settings - use private key "Linux-VM-Key-6.pem"
    select connect
  
  Input the following commands in the terminal
    clear
    sudo apt update (system update)
    sudo apt upgrade (system upgrade)
    Y
    select "ok" on promt screen
    sudo nano /etc/hostname (open hostname file)
    change hostname of virtual machine to ansiblemaster
    "ctrl + x" to save - type Y - press enter
    cat nano /etc/hostname (make sure changes were saved)
    sudo init 6 (reboot system after hostname change)
    type "R" (restart vm)

Step 3
Set up Passwordless Authentication (Prerequisite for Ansible)

  Switch to linux_master sever terminal
    generate ssh key with this command: ssh-keygen or ssh-keygen -t ed25519
    press enter until process is complete
    pwd (current directory should be /home/ubuntu)
    cd .ssh/ (change directory to /.ssh)
    ls (shows inside this directory are the public and private key)
    open public key by navigating through left panel: .ssh folder - id_rsa.pub
    copy contents of id_rsa.pub
    
  Switch to ansible_master server terminal
    cd .ssh/
    ls (shows "authorized_keys" file)
    open file by navigating left panel: .ssh folder - authorized_keys
    paste contents from id_rsa.pub and save file then select "save to all"
    cat authorized_keys (make sure changes were saved)
  
  Switch to linux_master server terminal
    ssh *paste copied private ip address of ansible_master server" (command connects linux_master to ansible_master | passwordless authentication)

Step 4
Use Ansible Adhoc Commands instead of Playbooks

  Within linux_master server terminal..
    Create inventory file in .ansible/
      input private ip address of ansible_master and save file
    ansible -i inventory all -m "shell" -a "touch devopsclass" 
      (use "all" or "copied private ip of specific target servers")
      -i "file" 
      -m "module" (go to ansible site to see what modules are available - ansible module "shell" supports shell commands) 
      -a "attribute" (what is the command you want to execute)
      
  Switch to ansible_master server terminal
    ls -ltr (verify the file was created)

  Switch to linux_master server terminal
    Use these example ansible adhoc commands
      ls -ltr & du -sh 
        (command shows files and file sizes)
      ansible -i inventory all -m "shell" -a "nproc" 
        (number of processes running on target server)
      ansible -i inventory all -m "shell" -a "df"
        (shows disk usage of target server)

Special Case
  If you have Data Base Servers and Web Servers
   Seperate them using brackets in Inventory file
      [dbservers]
      ...
      [webservers]
      ... 
  Switch "all" in following command to "dbservers"
    ansible -i inventory all -m "shell" -a "df"
    ansible -i inventory dbservers -m "shell" -a "df"
  Now this command is applied to dbservers only

Step 5
Write Playbook to install and restart Nginx

  Using linux_master server terminal
  Create Playbook (Ansible playbooks are written in yaml format)
    vim first-playbook.yml or "touch first-playbook.yml + vi firstplaybook.yml"
    press "i" to edit
    
        --- #Three hyphens indicate that this is a yaml file
        - name: Install and Start nginx #"-" indicates start of playbook then name playbook by purpose and use
            hosts: all #Provide list of host/servers where playbook will run
            become: true #Select user for ansible to execute playbook (root or other)
        
          tasks:
           - name: Intall nginx #task always start with "-" and describe task using "name:"
             apt: #provide command (use ansible apt or shell command | "shell: apt install nginx" |apt is generic ansible option)
               name: nginx #provide name of what you want to install
               state: present #"present" means you want to install nginx
           - name: Start nginx #Begin second task by describing purpose
             service:  #use ansible generic "service" module or shell command "shell: systemctl start nginx"
               name: nginx #what is the name of service
               state: started #"started" will start the service nginx

  press "ctrl + c" to exit insert mode
  tpye ":wq!" to save changes and exit editor (":q!" quits editor without saving changes)

  Run Playbook
   #"ansible" is the command used to run ansible commands
   #"ansible-playbook" is the command used to run playbooks
   ansible-playbook -i inventory first-playbook.yml (use "-i inventory" if inventory file is not in the default /ete/host/ansible file)
     Process Explanation:
       Gather facts: Ansible gathers information from targets servers and checks password authentication and other required details
       Install Nginx: execute first task from playbook
       Start Nginx: execute second task from playbook

  Switch to ansible_master server terminal "target server"
    run sudo systemctl status nginx (shows if nginx is active with details)

  Swith to linux_master server
  Run following command with -v or -vv or -vvv in front of -i
    ansible-playbook -vvv -i inventory first-playbook.yml
    (command shows process with highest detail of debugging logs during process based on number of v's)

Special Case
Run Kubernetes through Ansible

  Write Ansible playbook on host server
  vim kubernetes.yaml
  use "ansible roles" to write complex playbooks with many task
  Example: "ansible-galaxy role init kubernetes"

On linux_master server terminal
Use following codes:
  mkdir second-playbook
  cd second-playbook
  ansible-galaxy role init kubernetes | Result: - Role kubernetes was created successfully
  ls | Result: "kubernetes" folder
  ls kubernetes/ | shows files within folder
  ls -ltr kubertnetes/ | shows whats inside each folder
  

  
    

    
    
    



  
