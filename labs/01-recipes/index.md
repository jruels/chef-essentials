# Getting familiar with Recipes

In this lab you are going to write your first recipe, and then tweak it a little so you can see how `chef-client` works.   You will also create a more complex recipe to install a couple packages and populate the `motd` 

## Your first recipe
Your first recipe is very simple and is the classic `hello world` that we all love.  

Begin by creating a working directory:
```bash
mkdir -p /home/vagrant/chef/recipes
```

In this new directory use your favorite editor to create `hello.rb` with the following content: 
```ruby
file "#{Dir.home}/hello.txt" do
  content "Hello, world! \n"
end
```

Now run the `chef-client` locally to apply this recipe to localhost. 
```bash
chef-client -z hello.rb 
```

Confirm the file was created
```bash
cat $HOME/hello.txt
```

Congrats! You just wrote your first recipe and ran `chef-client` for the first time. 

## Resource properties
Now let's do a little experimentation to learn more about how things work. 

Start by looking at the permissions of the `hello.txt` file. 
```bash
ls -l $HOME/hello.txt
```

You should see something similar to: 
```bash 
-rw-r--r-- 1 vagrant vagrant 15 Mar 20 02:45 /home/vagrant/hello.txt
```

Now update the recipe so that it is owned by `root:root` and is only readable by the `root` user. 

Add the following to `hello.rb` 
```ruby
  owner 'root'
	group 'root'
  mode   0600
```

Now run the `chef-client` with sudo so it has access to change file permissions. 
```bash
sudo chef-client -z hello.rb
```

Check out the file permissions and see if they are set correctly. 
```bash
ls -l $HOME/hello.txt
```

Do you see what you were expecting? 
```bash
-rw------- 1 root root 15 Mar 20 02:52 /home/vagrant/hello.txt
```

Change the content of `hello.txt` and re-run `chef-client` . 

What happened? 

## Multi-resource recipe
Now using the following resources:
* Package
* File
* Service

Write your own recipe `setup.rb` that installs the `tree` and `ntp` packages, and populates `/etc/motd` with "This server is the property of..." 

We also want `ntp` service to run, and be enabled if the server reboots. 

If you need to refer back to the slides go ahead, but the recommended approach is to play around a bit and if you are stumped ask the instructor for assistance. 

Remember to run `chef-client` to apply your recipe. 

The following links will help if you get stuck:
* https://docs.chef.io/resource_service.html
* https://docs.chef.io/resource_package.html
* https://docs.chef.io/resource_file.html
* https://docs.chef.io/resources.html

We will be going over the lab solutions as a class. 

## Lab Complete