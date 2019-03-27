# Assign Roles

In this lab you are going to create 2 roles: 
* web
* load-balancer 

After creating these roles you are going to assign them to your nodes. 

Create `web.rb` in the `roles` directory and populate it with the following: 
```ruby
name                     'web'
description              'Web Server role'
run_list                 'recipe[workstation]','recipe[apache]'
default_attributes       'apache-test' => {
   'attribute1'      =>  'hello from attribute 1',
   'attribute2'      =>  'you\'re great'
}
```

Now create `load-balancer.rb` in the roles directory and give it the following: 
* name              = `load-balancer`
* description    =`To balance requests`
* run_list that includes `myhaproxy` cookbook

After creating these roles upload them to the Chef Server and then assign them to the web servers and the load-balancer server. 

Run `chef-client` on the nodes to update the role info. 

Confirm each node has the required roles using `knife node show`



## Knife tips 
As much fun as it has been to use `vagrant ssh` to log into each node and then manually run commands like `sudo chef-client` knife actually has a built-in `SSH` command. 

This enables logging into your nodes very easily and running commands on them. 

Because we are running our VMs on Vagrant we have to provide a little extra information.

* Tell `knife` to connect to localhost 
* Provide the `username` to login 
* Provide the `SSH` port each Vagrant machine is listening on 
* Provide the `identifyFile` to authenticate

Run `chef-client` on `web1`: 
```bash
knife ssh localhost 'sudo chef-client' -m -p 2222 -x vagrant -i $HOME/chef-repo/.vagrant/machines/web1/virtualbox/private_key
```

To run it on the other VMs you would just replace the port and identityFile with the values from `vagrant ssh-config`

When you are running your nodes in a cloud provider where they are accessible directly on port 22 you can use `knife` search patterns to run commands on specific nodes that match. 

To run `chef-client` on all nodes you could run something like this: 
```bash
knife ssh "*:*" -x chef -P chef "sudo chef-client"
```

Or if you wanted to run commands only on web nodes 
```bash
knife ssh "role:web" -x chef -P chef "sudo chef-client"
```

Knife is very powerful and allows for running ad-hoc commands very easily. 
