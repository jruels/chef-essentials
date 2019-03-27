# Bootstrap node

In this lab you will use `knife` to bootstrap a local virtual machine.  This machine will then be managed by the Chef Server. 

## Spin up nodes 
As discussed we will be using `Vagrant` to spin up the new nodes we will be bootstrapping. 

Copy the `Vagrantfile` from `lab07/files` directory to `chef-repo`
```bash
cp 07-bootstrap-node/files/Vagrantfile ~/chef-repo/.
```

Now enter the directory
```bash
cd ~/chef-repo 
```

Spin up nodes
```bash
vagrant up 
```

This will take a few minutes to complete, and you will be dropped back to command-line when it's complete. 

To check all of the VMs were started successfully run:
```bash
vagrant status
```

Output:
```bash
Current machine states:

web1                      running (virtualbox)
web2                      running (virtualbox)
load-balancer             running (virtualbox)
```

Now that all the VMs are online we are going to bootstrap `web1`, but before we can do that we need to have some information. 

* SSH port 
* IdentifyFile 

To get this info we need to look at Vagrant's ssh config. 
```bash
vagrant ssh-config web1
```

Output:
```bash
Host web1
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/jruels/chef-repo/.vagrant/machines/web1/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

Now we can see SSH is listening on port `2222` and the IdentifyFile's location. 

Now let's run `knife` to bootstrap the node. 
```bash
knife bootstrap localhost -xvagrant -p 2222 --sudo  -i /path/to/your/private_key -N web1
```

Remember to replace the port and IdentifyFile with the output of `ssh-config web1` on your machine. 

Now that you've bootstrapped the node confirm it shows up
```bash
knife node list 
```

Output: 
```bash
web1
```

See more info about `web1`
```bash 
knife node show web1
```


We can see that the `Run List` is empty and the Chef Server doesn't really have much info about `web1`
Output:
```bash
Node Name:   web1
Environment: _default
FQDN:        web1
IP:          10.0.2.15
Run List:
Roles:
Recipes:
Platform:    centos 7.6.1810
Tags:
```


Now let's update the `Run List` to include our cookbooks 
```bash
knife node run_list add web1 "recipe[workstation],recipe[apache]"
```

After that is done confirm the node info was updated
```bash
knife node show web1
```

Output: 
```
Node Name:   web1
Environment: _default
FQDN:        web1
IP:          10.0.2.15
Run List:    recipe[workstation], recipe[apache]
Roles:
Recipes:
Platform:    centos 7.6.1810
Tags:
```

Final step is to log into web1 and run the `chef-client` to apply the Run List. 

Log into `web1`
```bash
vagrant ssh web1
```

Once logged in run `chef-client`
```bash
sudo chef-client
```

Output: 
```bash
Starting Chef Client, version 14.11.21
resolving cookbooks for run list: ["workstation", "apache"]
Synchronizing Cookbooks:
  - workstation (0.2.1)
  - apache (0.2.1)
Installing Cookbook Gems:
Compiling Cookbooks...
Converging 8 resources
snip...

Chef Client finished, 8/10 resources updated in 20 seconds
```

After the `chef-client` has completed running confirm everything works. 
```bash
curl localhost 
```

Output:
```bash
<html>
  <body>
    <h1>Hello, world!</h1>
    <h2>ipaddress: 192.168.10.43</h2>
    <h2>hostname: web1</h2>
</body>
</html>
```

Check `/etc/motd`
```bash
cat /etc/motd
```

Output:
```
Property of ...

  IPADDRESS: 192.168.10.43
  HOSTNAME : web1
  MEMORY   : 1014972kB
  CPU      : 3095.998
```


Congrats! You've just bootstrapped your first server! 

## Lab Complete
