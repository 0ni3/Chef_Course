# Inspec
```
describe package('httpd') do
  it { should be installed}
end
```

when using the chef generate commands, the default location for the inspect tests is test/smoke/default

to know much more

https://www.inspect.io/docs/reference/resource

an example with its to compate the value of a setting with a matcher:
```
  describe file ('/var/www/html/index.html') do
    it { should exist}
    its { 'content '} {should match /hello world/}
  end


  describe package('httpd') do
    it { should be_installed}
  end
```

chef generate cookbook cookbooks/lcd_inspec

vim kitchen.yml
```
[vagrant@localhost lcd_inspec]$ vim recipes/default.rb
[vagrant@localhost lcd_inspec]$ cat recipes/default.rb
#
# Cookbook:: lcd_inspec
# Recipe:: default
#
# Copyright:: 2021, The Authors, All Rights Reserved.
#

['net-tools', 'httpd'].each do |pkg|
  package pkg do
   action :install
  end
end
```

under /home/vagrant/chef/cookbooks/lcd_inspec/test/integration/default

```
[vagrant@localhost default]$ cat default_test.rb
# InSpec test for recipe lcd_inspec::default

# The InSpec reference, with examples and extensive documentation, can be
# found at https://docs.chef.io/inspec/resources/

['net-tools', 'httpd'].each do |pkg|
  describe package(pkg) do
    it {should be_installed}
  end
end
```

# ChefSpec

Chefspec is used for unit testing and provide the ability to quickly test the vailidity of a chef code

the dir is spec/unit/recipes and the default_spec.rb is the default recipe

```
require 'chefspec'
describe 'example::default' do
  let(:chef_run) { ChefSpec::SoloRunner.converge(described_recipe) }

it 'does_something' do
  expect(chef_run).to ACTION_RESOURCE(NAME)
  end
end
```
  582  chef generate cookbook ./lcd_cheftest
  583  cp lcd_inspec/kitchen.yml lcd_cheftest/kitchen.yml
```
[vagrant@localhost spec]$ tree
.
├── spec_helper.rb
└── unit
    └── recipes
        └── default_spec.rb



 chef exec rspec
..
```
Finished in 6.3 seconds (files took 1.37 seconds to load)
2 examples, 0 failures



/home/vagrant/chef/cookbooks/lcd_cheftest/spec/unit/recipes
```
[vagrant@localhost recipes]$ cat default_spec.rb
#
# Cookbook:: lcd_cheftest
# Spec:: default
#
# Copyright:: 2021, The Authors, All Rights Reserved.
```

require 'spec_helper'
```
describe 'lcd_cheftest::default' do
  context 'When all attributes are default, on CentOS 7.2' do
    # for a complete list of available platforms and versions see:
    # https://github.com/chefspec/fauxhai/blob/master/PLATFORMS.md
    platform 'centos', '7.2'

    it 'converges successfully' do
      expect { chef_run }.to_not raise_error
    end
    it 'installs https' do
      expect { chef_run }.to install_package
    end
    it 'enables the https server' do
      expect { chef_run }.to enable_service('httpd')
    end
    it 'starts the https server' do
      expect { chef_run }.to start_service('httpd')
    end
  end
end
```

/home/vagrant/chef/cookbooks/lcd_cheftest/recipes

```
[vagrant@localhost recipes]$ cat default.rb
#
# Cookbook:: lcd_cheftest
# Recipe:: defaukt
#
package 'httpd' do
  action :instll
end
service 'httpd' do
  action [:enable, :start]
end
```
```
chef exec rspec --color --fd
```

# Chef Available Testing Framework, Inspec and ChefSpec

In this lab, we will demonstrate an understanding of Chef Testing Frameworks by using InSpec and ChefSpec with ChefDK and then using Chef Kitchen.

First, we'll install the ChefDK tools on a server and make sure that Docker is working. Then, we need to ensure that Git is installed and has some basic configuration. Next, we'll see how much we know about local cookbook development by creating one that installs software and uses InSpec and ChefSpec.

At the end of this hands-on Lab, we will have installed ChefDK tools, configured it, and have a custom-developed cookbook that we've tested with Docker and Kitchen.


```
    1  wget https://packages.chef.io/files/stable/chefdk/2.4.17/el/7/chefdk-2.4.17-1.el7.x86_64.rpm
    2  sudo rpm -ivh chefdk-2.4.17-1.el7.x86_64.rpm
    3  echo 'eval "$(chef shell-init bash)"' >> ~/.bash_profile
    4  source ~/.bash_profile
    5  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    6  sudo yum-config-manager --setopt="docker-ce-stable.baseurl=https://download.docker.com/linux/centos/7/x86_64/stable" --save
    7  sudo yum-config-manager --add-repo=http://mirror.centos.org/centos/7/extras/x86_64/
    8  sudo rpm --import https://www.centos.org/keys/RPM-GPG-KEY-CentOS-7
    9  sudo yum install -y container-selinux docker-ce
   10  sudo yum install yum-utils gcc
   12  sudo systemctl start docker
   13  sudo systemctl enable docker
   14  sudo usermod -aG docker $USER
   15  exit

   26  chef generate cookbook la_testing
   27  cd la_testing/
   28  ls -la
   29  vim .kitchen.yml
   30  ls
   31  vim recipes/default.rb
   32  cat recipes/default.rb
   33  vim test/smoke/default/default_test.rb
   34  vim spec/unit/recipes/default_spec.rb
   35  chef exec rspec --color -fd
   36  kitchen verify
   37  kitchen destroy
```

#### kitchen.yml
```
---
driver:
  name: docker
  privileged: true
  use_sudo: false

provisioner:
  name: chef_zero
  always_update_cookbooks: true
  product_name: "chef"
  product_version: "13.8.5"

verifier:
  name: inspec

platforms:
  - name: ubuntu-16.04

suites:
  - name: default
    run_list:
      - recipe[la_testing::default]
    verifier:
      inspec_tests:
        - test/smoke/default
    attributes:
```

#### recipes/default.rb
```
#
# Cookbook:: la_testing
# Recipe:: default
#
# Copyright:: 2021, The Authors, All Rights Reserved.
#
['net-tools','httpd'].each do |pkg|
  package pkg do
   action :install
  end
end
```


#### defaukt_test.tb
```
# # encoding: utf-8

# Inspec test for recipe la_testing::default

# The Inspec reference, with examples and extensive documentation, can be
# found at http://inspec.io/docs/reference/resources/

unless os.windows?
  # This is an example test, replace with your own test.
  describe user('root'), :skip do
    it { should exist }
  end
end

# This is an example test, replace it with your own test.
describe port(80), :skip do
  it { should_not be_listening }
end

['net-tools','httpd'].each do |pkg|
  describe package(pkg) do
   it { should be_installed }
  end
end
```


#### default_spec.rb
```
#
# Cookbook:: la_testing
# Spec:: default
#
# Copyright:: 2021, The Authors, All Rights Reserved.

require 'spec_helper'

describe 'la_testing::default' do
  context 'When all attributes are default, on CentOS 7.4.1708' do
    let(:chef_run) do
      # for a complete list of available platforms and versions see:
      # https://github.com/customink/fauxhai/blob/master/PLATFORMS.md
      runner = ChefSpec::ServerRunner.new(platform: 'centos', version: '7.4.1708')
      runner.converge(described_recipe)
    end

    it 'converges successfully' do
      expect { chef_run }.to_not raise_error
    end
    it 'installs httpd' do
      expect(chef_run).to install_package('httpd')
    end
    it 'installs net-tools' do
      expect(chef_run).to install_package('net-tools')
    end
  end
end
```
