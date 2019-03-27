# Chef Environments

Environments allow you to specify a list of policies which allow the versions of cookbooks that should be run. 

First we need to create some environments 

* Production 
* UAT 

Then we will update our nodes with the new environments. 

Environments live in the `chef-repo` directory on the same level as `roles` 

Create a directory to store your environments
```
mkdir environments
```

Create  `production.rb`
```ruby
name 'production'
description 'where production code is run'

cookbook 'apache', '= 0.2.1'
cookbook 'myhaproxy', '= 1.0.0'
```

Make sure the versions match YOUR cookbook versions. 

Upload the environment
```bash
knife environment from file production.rb
```

Assign environment to web1
```bash
knife node environment set web1 production 
```

Confirm that the `production` environment has been assigned to `web1`
```bash
knife node show web1 
```

Output: 
```
Node Name:   web1
Environment: production
```

Assign the `production` environment to the `lb` node as well using the steps above. 

Using the `knife ssh` command run `sudo chef-client` to apply the changes. 

## Create UAT environment 
Now that I've walked you through creating the `production` environment it's your turn to create a `UAT` environment with the following properties:

```
name `acceptance`
No cookbook restrictions 
```

After creating your `acceptance` environment use `knife` to upload it to the Chef Server.  

Assign `web2` to the new `acceptance` environment. 

Run `chef-client` on `web2` to apply the changes. 

## Update load balancer to production environment 
Open the `haproxy` cookbook's `default.rb`
Update it with a search for nodes that match `role[web]` and environment production. 
```ruby
pool_members = search("node","role:web AND chef_environment:#{node.chef_environment}")
```

Now save this file and update the `metadata.rb` file to a new minor version. 

Don't forget to update the `myhaproxy` cookbook version in the `production` environment file, and re-upload it to the Chef server.

Use `berks upload` to upload the new version of your cookbook to the Chef Server. 

Run the `chef-client` on the `lb` node to apply the new cookbook. 

To confirm everything is working as expected and traffic is only being directed to nodes with environment production, log into the `load-balancer` node. 
```
vagrant ssh load-balancer
```

Look at the `haproxy.cfg` and confirm it only has one web server in the pool 
```
backend servers
  server web1 192.168.10.43:80 maxconn 32
```

Great job! 

# Lab Complete
