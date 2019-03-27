# Bootstrap Load Balancer lab

In this lab you will use `Berkshelf` to upload the `haproxy` cookbook with all it's dependencies to the Chef Server.   After uploading the cookbooks you will bootstrap the load balancer node with the `myhaproxy` cookbook. 

Generate our new `myhaproxy` cookbook
```bash
chef generate cookbooks/myhaproxy
```

Add dependency to `myhaproxy ` `metadata.rb`
```ruby
depends 'haproxy', '~> 6.4.0'
```

Enter the cookbook directory
```bash
cd ~/chef-repo/cookbooks/myhaproxy
```

Now edit `myhaproxy` cookbook's default recipe. 
```ruby
#
# Cookbook:: myhaproxy
# Recipe:: default
#
# Copyright:: 2019, The Authors, All Rights Reserved.

haproxy_install 'package'

haproxy_frontend 'http-in' do
  bind '*:80'
  default_backend 'servers'
end

haproxy_backend 'servers' do
  server [
    'web1 192.168.10.43:80 maxconn 32',
    # 'server2 127.0.0.1:8000 maxconn 32'
  ]
  # notifies :restart, 'haproxy_service[haproxy]'
  # this doesn't seem to work. Using subscribes in haproxy_service[haproxy] instead per
  # https://github.com/sous-chefs/haproxy/issues/274
end

haproxy_service 'haproxy' do
  subscribes :reload, 'template[/etc/haproxy/haproxy.cfg]', :immediately
end
```

Remember to replace the IP address with the IP of your VM.

## Berkshelf
Berkshelf makes it easy to manage your cookbooks and all of their dependencies. 

To see this in action use the `berks` utility to upload the cookbooks to our Chef Server. 
```bash 
berks install 
```

The `install` argument will create a `Berksfile` which includes the cookbook and all of its dependencies.  This is a static file which is only updated when you re-run the `berks install` command. 

After creating this `Berksfile` let's upload the cookbooks 
```bash
berks upload 
```

You should now see: 
```bash
Uploaded windows (5.3.0) to: 'https://api.chef.io/organizations/...'
Uploaded seven_zip (3.1.0) to: 'https://api.chef.io/organizations/...'
Uploaded mingw (2.1.0) to: 'https://api.chef.io/organizations/...'
Uploaded build-essential (8.2.1) to: 'https://api.chef.io/organizations/...'
Uploaded poise (2.8.2) to: 'https://api.chef.io/organizations/...'
Uploaded poise-service (1.5.2) to: 'https://api.chef.io/organizations/...'
Uploaded haproxy (6.4.0) to: 'https://api.chef.io/organizations/...'
Uploaded myhaproxy (0.1.0) to: 'https://api.chef.io/organizations/...'
```

As you can see it uploaded the `myhaproxy` cookbook, the dependent `haproxy` cookbook and a bunch of other dependent cookbooks. 

## Bootstrap load balancer
Now that all of the cookbooks are uploaded, follow the process from the previous lab to: 

* bootstrap load balancer VM 
* update run list to have `myhaproxy` cookbook 
* SSH to new LB node and run `chef-client`

PROTIP: You can add `--run-list` to the `knife bootstrap` command to apply the `myhaproxy` cookbook at the same time as bootstrapping.  This way you don't have to manually run `sudo chef-client` on load-balancer node. 

To confirm this worked as expected you can log into load balancer node and run 
```
curl localhost 
```

You can also open your web browser and load (http://localhost:9000)


## Lab Complete
