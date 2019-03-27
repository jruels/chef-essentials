# Using Search

In this lab you are going to get more familiar with using Chef Search with knife and inside recipes. 

## Knife search 
With knife you are able to use the `search` argument to search through the different indexes stored within Chef. 
* Client 
* Node 
* Role 
* Environment
* Data bags 

The syntax for using `knife search` is: 
```bash
knife search INDEX (client, node, role, environment, data bags) "key:value" 
```

Using this syntax you can search for the `node` index for all nodes
```bash
knife search node "*:*"
```

If you want to find a specific attribute, for example IP address
```bash
knife search node "*:*" -a ipaddress
```

Output:
```bash
3 items found

web2:
  ipaddress: 192.168.10.44

lb:
  ipaddress: 10.0.2.15

web1:
  ipaddress: 192.168.10.43
```

If you want to narrow it down to a particular node name you can do that as well. 
```bash
knife search node "name:web1"
```

You can also search for nodes with specific roles 
``` bash
knife search node "role:web"
```

You don't have to just search the `node` index though.  

Do a search in the in the `role` index for all roles
```bash
knife search role "*:*"
```

The search also supports operators so to search for nodes with the role of `web` and cookbook `workstation` you can run: 
```bash
knife search node "role:web AND recipes:workstation"
```

## Dynamic load balancing 
In this lab we are going to finally get rid of the hard coded entries for our `haproxy` config file. 

Open `default.rb` for `myhaproxy` in your favorite editor and then update it to have the following 
```ruby
haproxy_install 'package'

haproxy_frontend 'http-in' do
  bind '*:80'
  default_backend 'servers'
end

pool_members = search("node","role:web")
servers = []

pool_members.each do |web_node|
  server = "#{web_node['hostname']} #{web_node['ipaddress']}:80 maxconn 32"
  servers.push(server)
end

haproxy_backend 'servers' do
  server servers
end

haproxy_service 'haproxy' do
  subscribes :reload, 'template[/etc/haproxy/haproxy.cfg]', :immediately
end
```

You can see that just as we discussed I have added some code to search our nodes for `role:web` and then add them to our `haproxy_backend` pool

After making these changes update the `metadata.rb` file to specify this is a major change and then use `berks upload` to upload our updated `myhaproxy` cookbook and it's dependencies. 
 
After that use `knife ssh` to execute `sudo chef-client` on the `lb` node. 

Now log into the `load-balancer` 
```bash
vagrant ssh load-balancer 
```

Look in the `haproxy` config file and see if it shows both our web servers. 
```bash
cat /etc/haproxy/haproxy.cfg
```

Output:
```
frontend http-in
  default_backend servers
  bind *:80


backend servers
  server web2 192.168.10.44:80 maxconn 32
  server web1 192.168.10.43:80 maxconn 32
```

Alright, that's great!   Now confirm it is load balancing correctly by running `curl localhost` 

If everything looks good check to see if chef correct version of the `myhaproxy` cookbook was applied. 
```bash
knife node show lb -a cookbooks
```

Output:
```bash
knife node show lb -a cookbooks
lb:
  cookbooks:
    build-essential:
      version: 8.2.1
    haproxy:
      version: 6.4.0
    mingw:
      version: 2.1.0
    myhaproxy:
      version: 1.0.0
...
```

Perfect!  The load balancer is now configured to dynamically add/remove nodes with `role[web]`

To test this out remove the role `web` from `web2` and run the `chef-client` on both `web2` and `lb`.  Then log into `lb` and confirm the `haproxy` config file only has `web1`. 

## Lab Complete