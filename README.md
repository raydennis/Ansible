# Running Ansible

## Prerequisites (instructions for installing found below)
1. **Control Machine** with (this can be your local machine or a remote machine you can log into)
    * Ansible

2. **Target machine with**
    * User account with an SSH key and sudo permissions

## Getting started
1. Clone this repo
2. Modify /etc/ansible/hosts to include your target machine(s)
	```bash
	[localweb]
	192.168.0.13
	```
3. Modify role-repo.yml to include your target machine(s), your $USER, and the roles you're interested in: 
  i.e.  
	```bash
	- hosts: localweb
	  remote_user: unmitsvc
	  become: True
	  become_method: sudo
	  gather_facts: True
	  roles:
	    - oci
	```
# Setting up the Prerequisites
## 1. On the control machine: Install Ansible
Ansible is a radically simple IT automation system. It handles configuration-management, application deployment, cloud provisioning, ad-hoc task-execution, and multinode orchestration -- including trivializing things like zero-downtime rolling updates with load balancers.

Read the documentation and more at https://ansible.com/

You can find installation instructions [here](https://docs.ansible.com/intro_getting_started.html) for a variety of platforms.

### Installation
Ubuntu
```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```
RHEL and CentOS
```bash
sudo yum install ansible
```

## 2. On your target machine: Admin user account, ssh key and ssh access settings

### If you need to create a new account, the steps are provided here: 
Add a user
```bash
useradd $USER
```
Change the password
```bash
passwd $USER
```
Add them to sudoer (Ubuntu/Debian)
```bash
usermod -aG sudo $USER
```
Or for RHEL/Centos
```bash
usermod -aG wheel $USER
```
### Enable ssh with key authorization
On your target machine: ensure that SSH is installed.  If not, run the following command:
Ubuntu/Debian
```bash
apt install openssh-server -y
```
RHEL/CentOs
```bash
yum -y install openssh-server
```
Verify that you have a home directory (some companies elect not to create one for security purposes. If you fall into that category, the following will ONLY create a folder which allows you to enter your RSA)

(replace $USER with your username and $SERVER with your servername)
```bash
ssh $USER@$SERVER  
sudo mkdir -p /nfs/user/r/$USER  
cd ~  
cd ..  
sudo chown $USER $USER  
cd $USER  
mkdir .ssh  
touch .ssh/authorized_keys  
```

Next, get your RSA to remote server (Run this on your control machine)
The first thing you’ll need to do is make sure you’ve run the keygen command to generate the keys:
```bash
ssh-keygen -t rsa
```
Then use this command to push the key to the remote server, modifying it to match your server name.
```bash
cat ~/.ssh/id_rsa.pub | ssh $USER@$SERVER 'cat >> .ssh/authorized_keys'
```

On your target machine:  
Edit ssh file to allow key log on
```bash
vi /etc/ssh/sshd_config
```

Make sure the following lines are not commented (no # at the beginning) and are set appropriately  
```bash
PubkeyAuthentication yes
AuthorizedKeyFile  .ssh/authorized_keys
PasswordAuthentication no
ChallengeResponseAuthentication no
```

Finally, restart the ssh service so that the settings will take effect.
Debian
```bash
sudo service ssh restart
```
RHEL
```bash
sudo systemctl restart sshd
```
Verify that everything is working by running the following command from your host machine:
Replace $SERVER with the server name and --user with your user account
```bash
ansible $SERVER --user $USER -m ping
```
