## Drivers

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

## Provisioner

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


## Platforms

platform_version defines the list of operating system and versions on which kitchen will perform cookbook testing

chef generate cookbook cookbooks/lcd_platform

vim kitchen.yml
kitchen converge
kitchen login
lsb_release -a

## Kitchen commands

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
