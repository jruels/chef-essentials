# Chef templates

In this lab you will create a template file in each of our cookbooks and then update the recipes to use the templates. 

## MOTD template
Start by generating the template file for `workstation` cookbook. 
```bash
chef generate template cookbooks/workstation motd
```

This will create `motd.erb`  which we will fill with the current `file` content from our `server.rb` recipe. 

Take the `file` content out of `server.rb` and put it into `motd.erb`
```ruby
This server is being managed by Chef!
	CPU Speed: #{node['cpu']['0']['mhz']} MHz
	MEMORY Total: #{node['memory']['total']}
	HOSTNAME: #{node['hostname']}
	IP Address: #{node['ipaddress']}
```

When you paste it into `motd.erb` remember to replace the string interpolation `#{}` with ruby tags `<%= > `.

Save the file and then update the recipe, `server.rb` with the template info. 
```ruby
template '/etc/motd' do
	source 'motd.erb'
end
```

Now run the `chef-client` and confirm `/etc/motd` shows the correct info, similar to: 
```
This server is being managed by Chef!
	CPU Speed: 3095.998 MHz
	MEMORY Total: 2041316kB
	HOSTNAME: chefdk
	IP Address: 10.0.2.15
```

NOTE: The contents of `/etc/motd` shouldn't actually change because we just refactored from using `file` resource to `template` resource. 

## Apache template 
Now that you've successfully refactored the `setup.rb` recipe to use `template` instead of `file` , do the same thing for the `Apache` cookbook. 

* Generate template: `index.html.erb` 
* Migrate `file`  resource content from `server.rb` to new template file
* Update `server.rb` to use `template` resource instead of `file` resource. 
* Run `chef-client`  to apply changes 
* Use `curl` to confirm it responds with correct data. 

## Template variables 
Let's play around with defining variables in recipes that we can then use in our templates to populate our files. 

Update the `server.rb` recipe with the following variables 
```ruby
variables(
   :name => 'Jason'
   :dreamcar  => '<your dream car>'
 )
```

Update the `index.html.erb`
```ruby
<html>
  <body>
    <h1>Hello, world!</h1>
    <h2>IPADDRESS: <%= node['ipaddress'] %></h2>
    <h2>HOSTNAME: <%= node['hostname'] %></h2>
    <h2>My Name is: <%= @name %></h2>
    <h2>My dream car is: <%= @dreamcar %></h2>
  </body>
</html>
```
