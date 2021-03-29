# Chef_Course

Chef - The Local Cookbook Development Badge

general commands:

´´´
chef generate attribute
chef generate recipe
chef generate file
chef generate lwrp
chef generate cookbook, chef generate repo, or chef generate app
chef generate template
´´´

the metadata.rb file is like __manifest__.py on python


wrapper cookbook: a wrappper cookbook wraps a different cookbook to change the behavior without creating a new fork of the code

wrapper cookbook require a dependency on the cookbook you're attempting to create a wrapper for. in the case of a wrapper for something like haproxy, I have to edit metadata.rb to include a dependency: depends 'haproxy', ' = 3.0.2'

this means that the haproxy cookbook will also be delivered to the chef-client in addition to the wrapper cookbook.


the minimum steps to add another cookbook to a wrapper cook book are:

add the dependency to a metadata.rb
add an include_recipe option to the cookbook's recipes/defailt.rb file


community cookbook can be found at:

https://supermarket.chef.io


to interact with the supermarket the commands are:

knife cookbook site

knife cookbook site list
knife cookbook site search tar
knife cookbook site show tar
knife cookbook site download tar
knife cookbook site install tar


Mixlib::ShellOut.new() is a class that shell out standard out and standard error, it accepts commands

find = Mixlib::ShellOut.new("find . -name '*.rb'")
find.run_command
puts find.stdout

shell_out method raise an error if the command fails

modules = shell_out("apachectl -t -D DUMP_MODULES")
puts modules.stdout.

execute allows for the execution of a single commands


´´´
execute 'systemctl restart httpd' do
  not_if 'curl -s http://localhost | grep "hello world"
end

execute 'app-installer'
  cwd '/opt/app/install'
  command './installer --install'
  user ' appuser'
  only_if { File.exist?('/opt/app/install/prereqs-complete.log')}
end
´´´

Use ShellOut and shell_out when you want to collect stdout and stderr



# The chef command

 - what chef command does
 - what chef generate can create
 - how customize content using generators
 - the reccomend way to create a template
 - the chef gem command


Chef Generate

generators are used to quickly generate cookbook of compontents of a cookbook. generators are invoked by using the chef generate command

I can create: 

app - an application repo
cookbook - a single cookbook
recipe - a single recipe
attribute
template
file
lwrp - a lightweight resource
repo
policyfile
generator
build-cookbook

chef generate generator <my_generator>

the gem command is a package manager for ruby

gem list local
gem install docker


Chef related commands used:

chef generate

chef generate cookbook test_cookbook

chef generate generator ./nome

cd generator/lcd_origin 

vim templates/default/README.md.erb

vim templates/default/kitchen.yml.erb (edited with docker config)


´´´
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
  # You may wish to disable always updating cookbooks in CI or other testing environments.
  # For example:
  #   always_update_cookbooks: <%%= !ENV['CI'] %>
  always_update_cookbooks: true

  ## product_name and product_version specifies a specific Chef product and version to install.
  ## see the Chef documentation for more details: https://docs.chef.io/workstation/config_yml_kitchen/
  product_name: "chef"
  product_version: "13.8.5"

verifier:
  name: inspec

platforms:
  - name: centos-7.2
    driver_config:
       run_command: /usr/lib/systemd/systemd

suites:
  - name: default
    run_list:
      - recipe[<%= cookbook_name %>::default]
    verifier:
      inspec_tests:
        - test/integration/default
    attributes:
´´´



now we need to edit the file spec_helper.rb


cd generator/lcd_origin/files/default

vim spec_helper.rb

´´´
cookbook_path ['~/chef/cookbooks']
local_mode true
if File.basename($PROGRAM_NAME).eql?('chef') && ARGV[0].eql?('generate')
        chefdk.generator.license = 'all_rights'
        chefdk.generator.copyright_holder = 'Oni'
        chefdk.generator.email = 'weathereport.wr@gmail.com'
        chefdk.generator_cookbook = '~/generator/lcd_origin'
end
´´´

mkdir .chef

vim .chef/config.rb

mkdir -p chef/cookbooks

cd chef

chef generate cookbook coobbook/lcd_web

gem install docker



# Foodcritic (chefdk tools)

 
is a part of chefdktools and is invoked with the foodcritic command, will check for style error synstax ecc..


foodcritic <parth> -t <tagname>

all of the tags are documented at footcritic.io


foodcritic ./ -t correctness, style

I can create a .foodcritic file to have rules defines on my own

___
cd ~/chef/cookbooks/lcdweb

foodcritic -h 

foodcritic -l 

foodcritic .

foodcritic . -t ~FC064 -t FC065

vim ./metadata.rb



vim ./README.md



www.foodcritic.io

vim .foodcritic


´´´

#FOODCRITIC HAS DEPRECATED, USE COOKSTYLE INSTEAD




## Berks

run berks install when a metadata.rb is configured

chef generate cookbook -b ./lcd_proxy

vim metadata.rb

berks list

bersk install 

berks contingent

berks package


<!-- rubocop is for ruby check -->

the .rubocop.yml contain settings to what cheks

AlignParameters:
       Enabled: false
Encoding:
       Enabled: false
LineLenght:
       Max: 200
StringLiterals:
       Enabled: false

to generate a .rubocop_todo.yml run the command rubocop --auto-gen-config


1. run rubocop --auto-gen-config
2. add inherit from: .rubocop_todo.yml to .rubocop.yml
3. run rubocop and fix the reported offense
4. repeat until there are no issues left
5. remove .rubocop_todo.yml when done.


## Test kitchen

test kitchen is a tool to test cookbook data across any combination of platforms

it's defined in a kitchen.yml file

kitchen

kitchen test

docker ps

kitchen converge

docker ps 

kitchen login

kitchen destroy



## Hands-on lab: Chef Dk tools, Docker, Chef Generator, Test kitchen and community cookbooks


- Install and configure what is necessary to use the specified version (2.4.17) of the ChefDK Tools on the Provided Server, including docker-ce, git and the docker gem.

- Generate a generator cookbook and modify it for use with Docker. Under provisioner a product name of `chef` and product version of `13.8.5` is required.
Configure the system so that the generator cookbook is used when generating cookbooks. Configure the system to use ~/chef/cookbooks for cookbooks.

- Generate a New Cookbook Using the Generator Template created previously, call it corp_haproxy. Ensure template changes have been included and add haproxy so it will download.

- Use berks to download dependencies, Use test kitchen to verify the cookbook.

- Clean up the instance created with the kitchen command.
