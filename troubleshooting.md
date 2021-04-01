
## Kitchen Phases Kitchen Phases

kitchen converge, create, destroy, diagnose, exec, init
list, login, test, verify

# Troubleshooting Lab


In this lab we will get familiar with Chef Troubleshooting.

First, we are going to install the ChefDK tools on a server, make sure that Docker is working, and ensure that Git is installed and has some basic configuration.

We will then go through troubleshooting a Chef cookbook, which will be provided. We need to figure out what it has for problems, and why it won't work.

At the end of this hands-on lab, we will have a working cookbook, after we've performed tests with Docker and Kitchen.

```
    2  wget https://packages.chef.io/files/stable/chef-workstation/21.2.278/el/8/chef-workstation-21.2.278-1.el7.x86_64.rpm
    3  sudo rpm -ivh chef-workstation-21.2.278-1.el7.x86_64.rpm
    4  echo 'eval "$(chef shell-init bash)"' >> ~/.bash_profile
    5  source ~/.bash_profile
    6  sudo yum install yum-utils
    7  sudo yum install gcc
    8  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    9  sudo yum -y install docker-ce
   10  sudo systemctl enable docker
   11  sudo systemctl start docker
   12  sudo usermod -aG docker $USER
   13  sudo yum -y install git
   14  git config --global user.name "oni"
   15  git config --global user.email "oni@example.com"
   16  git config --global core.editor vim
   17  gem install kitchen-docker
   18  exit
   19  sudo setenforce permissive
   20  suvo vim /etc/selinux/config
   21  sudo vim /etc/selinux/config
   22  getenforce
   24  cd chef
   26  cd la_troubleshooting/
   27  kitchen converge
   28  vim recipes/default.rb
   29  kitchen converge
   30  vim .kitchen
   31  vim .kitchen.yml
   34  cd .kitchen
   39  cd logs/
   41  tail -f kitchen.log
   42  tail -f default-centos-72.log
   48  vim recipes/default.rb
   49  kitchen converge
   50  kitchen login
   51  kitchen destroy
```
