

# Data Bags

Data bags are containers for information about your infrastructure that are not associated with a node.
They provide an external source of information to be used in recipes and search functionality

a databag is a global variable that is stored in a JSON data -> is accessible via a Chef server.

Data bags can also be used to encrypt sensitive information with a shared key;


Working with a data bag:

knife commands:

 knife data bag create BAG [ITEM] (option)
 knife data bag	delete BAG [ITEM] (option)
 knife data bag edit ...
 knife data bag from file BAG FILE|FOLDER [FILE|FOLDER...] (option)
 knife data bag list (optios)
 knife data bag show BAG [ITEM] (options)


A data bag can be created with knife or manually as a json file

with knife:

 knife data bag create admins joe


admins is the name of the data bag, and joe is the name of the item

searching data bags can be performed with

knife search admins "id:tom"

knife search admins "gid:developers OR gid:adminstrators"

### Wildcards are also an option 

knife search admins "*:*"

knife search admins "gid:admin"


## Chef Vault

Vault allows the secure distribution of secrets as an alternative to encrypted data bags.

it comes with the Chef DK and is invoked with the knife vault command.

vaults creates a databag that is encrypted using a random secret. A vault has a list of administrtive users and clients.

```
cd chef
knife dara bag create admins
cd chef/data_bags/admins

[vagrant@localhost admins]$ export EDITOR=vim
[vagrant@localhost admins]$ echo $EDITOR

knife data bag create admins joe

[vagrant@localhost chef]$ tree data_bags/
data_bags/
├── admins
│   └── joe.json
├── example
│   └── example_item.json
└── README.md
```

```
[vagrant@localhost chef]$ cat data_bags/admins/joe.json
{
  "id": "joe",
  "uid": "1010",
  "gid": "developers",
  "shell": "/bin/bash",
  "comment": "Joe blogs"
}[vagrant@localhost chef]$ cd data_bags/admins/
[vagrant@localhost admins]$ ls
joe.json
[vagrant@localhost admins]$ cp joe.json tom.json
[vagrant@localhost admins]$ vim tom.json
[vagrant@localhost admins]$ knife data bag list
admins
example
[vagrant@localhost admins]$ ls
joe.json  tom.json
[vagrant@localhost admins]$ cp tom.json mike.json
[vagrant@localhost admins]$ vim mike.json
[vagrant@localhost admins]$ knife data bag show admin
ERROR: The object you are looking for could not be found
Response: Object not found: chefzero://localhost:1/data/admin
[vagrant@localhost admins]$ knife data bag show admins
joe
mike
tom
[vagrant@localhost admins]$ knife data bag show admins tom
comment: tom blogs
gid:     developers
id:      tom
shell:   /bin/bash
uid:     1011
[vagrant@localhost admins]$ knife search admins "id:tom"
1 items found

chef_type: data_bag_item
comment:   tom blogs
data_bag:  admins
gid:       developers
id:        tom
shell:     /bin/bash
uid:       1011

[vagrant@localhost admins]$ knife search admins "NOT id:tom"
2 items found

chef_type: data_bag_item
comment:   Joe blogs
data_bag:  admins
gid:       developers
id:        joe
shell:     /bin/bash
uid:       1010

chef_type: data_bag_item
comment:   mike blogs
data_bag:  admins
gid:       developers
id:        mike
shell:     /bin/bash
uid:       1012

[vagrant@localhost admins]$ knife search admins "NOT gid:developers"
0 items found

[vagrant@localhost admins]$ knife search admins "*:*"
3 items found

chef_type: data_bag_item
comment:   Joe blogs
data_bag:  admins
gid:       developers
id:        joe
shell:     /bin/bash
uid:       1010

chef_type: data_bag_item
comment:   mike blogs
data_bag:  admins
gid:       developers
id:        mike
shell:     /bin/bash
uid:       1012

chef_type: data_bag_item
comment:   tom blogs
data_bag:  admins
gid:       developers
id:        tom
shell:     /bin/bash
uid:       1011
```
```
[vagrant@localhost lcd_databags]$ cat recipes/default.rb
#
# Cookbook:: lcd_databags
# Recipe:: default
## Copyright:: 2021, The Authors, All Rights Reserved.
#

admin = data_bag('admin')

admins.each do |login|
  admin = data_bag_item('admin', login)
  group_admin['grid']
end
```

```
[vagrant@localhost lcd_databags]$ cat kitchen.yml
---
driver:
  name: docker
  privileged: true
  use_sudo: false

provisioner:
  name: chef_zero
  always_update_cookbooks: true
  product_name: 'chef'
  environments_path: ../../environments
  data_bags_path: ../../data_bags

verifier:
  name: inspec

platforms:
  - name: centos-7.2
    driver_config:
       run_command: /usr/lib/systemd/systemd

suites:
  - name: default
    provisioner:
      client_rb:
        environment: development
    run_list:
      - recipe[lcd_inspec::default]
    verifier:
      inspec_tests:
        - test/integration/default
    attributes:
```

Search command:

knife search INDEX SEARCH_QUERY

the data being searched can be a nested JSON data structure which can be multiple layers deep. These nested fields become flattened for search.
That means the nested elements become valid keys for search:

knife search node "mtu:1500"

operators are also allowed in search queries

knife search admins "gid:developers AND id:j*"

