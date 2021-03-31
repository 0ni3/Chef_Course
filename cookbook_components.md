# Drivers

drivers are used for creating the platform instances which are used during cookbook testing.

examples:

driver:
   name: docker

driver:
   name: vagrant

to know which driver are available to install visit docs.chef.io/kitchen.html

kitchen init --driver=kitchen-ec2

gem install kitchen-docker

gem uninstall drivername


__

kitchen help init

kitchen init --driver=kitchen-*

# Provisioner

suite config in kitchen.yml 

```
---
driver:
  name: docker
  privileged: true
  use_sudo: false

## The forwarded_port port feature lets you connect to ports on the VM guest via
## localhost on the host.
## see also: https://www.vagrantup.com/docs/networking/forwarded_ports

#  network:
#    - ["forwarded_port", {guest: 80, host: 8080}]

provisioner:
  name: chef_zero
  always_update_cookbooks: true
  product_name: "chef"
  product_version: "13.8.5"

verifier:
  name: inspec

platforms:
  - name: centos-8
  - name: centos-7.2

suites:
  - name: default
    run_list:
    - recipe[java::default]
    verifier:
      inspec_tests:
      - test/integration/default
    attributes: { 'java': {'jdk_version': '6' } }
    excludes:
      - centos-7.2
  - name: java7
    run_list:
    - recipe[java::default]
    verifier:
      inspec_tests:
      - test/integration/default
    attributes: { 'java': {'jdk_version': '6' } }
    includes:
      - centos-7.2
```

kitchen list


# Platforms

platform_version defines the list of operating system and versions on which kitchen will perform cookbook testing

chef generate cookbook cookbooks/lcd_platform

vim kitchen.yml
kitchen converge
kitchen login
lsb_release -a

# Kitchen commands

example of an interative development workflow:

1. set up kitchen by editing kitchen.yml
2. write the relevant tests
3. run kitchen verify converge (or test) to apply the new cookbook code
4. run kitchen verify to see if I pass the test
5. kitchen destroy to destroy the docker image

kitchen create -> creates my test instance

kitchen converge -> install the cheff client and executes the run_list in .kitchen.yml

kitchen setup -> to ensure the instance is prepare for testing

kitchen verify -> runs the test

kitchen destroy -> removes the test instances

all of these operations can be invoked with one command 'kitchen test'


the kitchen init does:

	- creates a default .kitchen.yml
	- creates the test/integration/default directory
	- add .kitchen to chefignore

# Directory Structure of a Cookbook

 - what components of a cookbook are
 - the default recipe and attributes files
 - why there is a 'default' subdirectory under 'templates'
 - where test are stored


Cookbook components:


Attributes: chef generate attribute
Recipes : chef generate recipe
Files: chef generate file
Resources: chef generate lwrp
Providers: chef generate lwrp
Metadata: chef generate cookbook, chef generate repo
Templates: chef generate template


What you can find in a repository is:

Roles - a role defines processes and patterns existing across the nodes in an organization which belong to a single job function.

Data bags - store data for global variables ina JSON format.

Environment - is a way to map the workflow of an organization. Every organization start with a _default environment that cannot be modified or deleted.


° when you create a cookbook there are several default files created: default recipe (recipes/default.rb) and attribute (./attribute/cookbookname.rb) generated with chef generate attribute


## Attributes and how they work

attributes can hold a collection of files. These contains settings to configure infrastructures

The chef client will obtain the node object from the Chef server, which has all the attribute data from the last chef-client run.


Attributes are the details about a node. They are used by chef-client to understand node state over time. The chef client must know the current state of a node, the previous state and what the node state should become after the end of the current run.


The chef client obtains attributes from:

 - node information from ohai
 - attribute files from cookbooks
 - recipes within cookbooks
 - environments 
 - roles

chef generate cookbook cookbooks/lcd_attribute

chef generate attribute default

vim attributes/default.rb

vim recipe/default.rb

package node['app']['language' do
  action :install
end

kitchen verify
kitchen destroy

```
  442  vim recipes/default.rb
  443  kitchen verify
  444  export EDITOR=$(which vi)
  445  knife environment create development
```

node.default['app']['language'] = 'ruby'

package node['app']['language'] do
  action :install
end

## Files and Templates

 - how to instantiate files on nodes
 - the difference between file, cookbook
 - how to write templates
 - what partial templates are 
 - ERB syntax



### file related resources


 - use the file resource to manage files on the node
 - use the cookbook_file resource to copy a file from cookbook's /files directory
 - use the template resource to create a file based on a template located in the cookbook/templates direcotry
 - use the remote_file resource to transfer a file to a node

 Actions 

 - :create
 - :create_if_missing 
 - :delete
 - :nothing
 - :touch 
 
```
  444  export EDITOR=$(which vi)
  445  knife environment create development
```
 the file resource:
```
 file '/tmp/message' do 
   owner 'root'
   group 'root'
   mode '0755'
   action :create
 end
```

attributes can be referenced to supply values
```
  file '/tmp/appfile' do 
     action :create_if_missing
     owner node['app']['user']
     mode '0400'
  end
```

the cookbook_file resource can be used to transfer files from the files subdir whithin a cookbook to the specified path on the server

```
cookbook_file 'var/www/html/index.php' do
   source 'index.php'
   owner 'apache'
   group 'apache'
   mode '0644'
   action :create
end
```

remote_file is used to transfer file from a remote location


the template resource is used to create a file based on an Embedded Ruby(ERB) template. It generates static text files, and places them on
the host where chef-client has run.

to generate a template:

chef generate template cookbooks/location index.html

## Libraries

```
  448  cd chef/
  449  ls
  450  chef generate cookbook cookbooks/lcd_libraries
  451  cd cookbooks/lcd_libraries/
  452  mkdir resources
  454  mkdir resources
  455  ls
  456  vim resources/site.rb
  457  vim resource/default.rb
```

```
[vagrant@localhost resources]$ cat site.rb
property :homepage, String, default: '<h1>Hello World</h1>'

action :create do
  package 'httpd'

  service 'httpd' do
    action [:enable, :start]
  end

  file '/var/www/html/index.html' do
    content new_resource.homepage
  end
end
```
```
[vagrant@localhost resources]$ cat default.rb
lcd_libraries_site 'httpd' do
  homepage '<h1>Welcome</h1>'
end

execute 'systemctl restart httpd' do
  only_if ( index_exist? )
end

[vagrant@localhost libraries]$ cat default.rb
def index_exist?
 ::File.exist?("/var/www/html/index.html")
end
```



# Cookbook Components Lab


In this lab we will install the ChefDK tools on a server. We will need to make sure Docker is working, then ensure that Git is installed and has some basic configuration.

Then we'll gain an understanding of Local Cookbook Development by creating a cookbook that uses a custom attribute to install a package.

At the end of this hands-on lab, we will have installed and configured ChefDK tools, developed cookbooks, and tested them with docker and kitchen commands.



    2  wget https://packages.chef.io/files/stable/chef-workstation/21.2.278/el/8/chef-workstation-21.2.278-1.el7.x86_64.rpm
    3  sudo rpm -ivh chef-workstation-21.2.278-1.el7.x86_64.rpm
    4  echo 'eval "$(chef shell-init bash)"' >> ~/.bash_profile
    5  source ~/.bash_profile
    6  sudo yum install yum-utils
    7  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    8  sudo yum -y install docker-ce
    9  sudo systemctl enable docker
   10  sudo systemctl start docker
   11  sudo usermod -aG docker &USER
   12  sudo usermod -aG docker $USER
   13  sudo yum -y install git
   14  git config --global user.name "oni"
   15  git config --global user.email "oni@example.COM"
   16  git config --global core.editor vim
   17  gem install kitchen-docker
   18  sudo gem install kitchen-docker
   19  gem install kitchen-docker
   20  sudo yum install -y gcc
   21  gem install kitchen-docker

  
   23  sudo setenforce permissive
   24  sudo vim /etc/selinux/
   25  sudo vim /etc/selinux/config
   27  chef generate cookbook la_attributes
   28  cd la_attributes/
   29  ls -la
   30  chef generate attribute default
   31  vim kitchen.yml
   32  cd attributes/
   34  vim default.rb
   38  cat default.rb
   39  vim default.rb
   42  cd ../recipes/
   43  vim default.rb
   49  kitchen verify
   51  cd la_attributes/
   53  kitchen verify
   54  kitchen login
   55  kitchen destroy


#### kitchen.yml

```
---
driver:
  name: docker
  priviledged: true
  use_sudo: false

## The forwarded_port port feature lets you connect to ports on the VM guest via
## localhost on the host.
## see also: https://www.vagrantup.com/docs/networking/forwarded_ports

#  network:
#    - ["forwarded_port", {guest: 80, host: 8080}]

provisioner:
  name: chef_zero
  always_update_cookbooks: true
  ## product_name and product_version specifies a specific Chef product and version to install.
  ## see the Chef documentation for more details: https://docs.chef.io/workstation/config_yml_kitchen/
  product_name: chef
  product_version: 16

verifier:
  name: inspec

platforms:
  - name: centos-7

suites:
  - name: default
    run_list:
      - recipe[la_attribute::default]
    verifier:
      inspec_tests:
        - test/integration/default
    attributes:
```


#### attribute/default.rb
```
default['app']['language'] = 'perl'
```


#### recipes/default.rb

```
# Cookbook:: la_attributes
# Recipe:: default
#
# Copyright:: 2021, The Authors, All Rights Reserved.

package node['app']['language'] do
  action :install
end
```
