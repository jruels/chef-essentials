# Data Bags

In this lab you will be creating a couple data bags for storing your user and group information, and then a cookbook to search and create these users on your infrastructure. 

## Create users data bag
We need to create a directory to hold our data bags in our `$HOME/chef-root` directory. 

```bash
mkdir -p data_bags/users
```

In this users directory we will create a `users` data bag using `knife`
```bash 
knife data bag create users
```

Data bags use `JSON` format so let's create a couple files. 

First create `user1.json` with the following:
```json
{
	"id": "user1",
	"comment": "I am user1",
	"uid": 100,
	"gid": 1,
	"home": "/home/user1",
	"shell": "/bin/bash",
	"platform": "centos"
}
```

Now create `user2.json`: 
```json
{
	"id": "user2",
	"comment": "I am user2",
	"uid": 101,
	"gid": 1,
	"home": "/home/user2",
	"shell": "/bin/bash",
	"platform": "centos"
}
```


After creating the use files we need to upload them to the Chef Server. 
```bash
knife data bag from file users data_bags/users/user1.json data_bags/users/user2.json 
```

Now confirm they were uploaded successfully.
```bash
knife data bag list
```

You should the `users` data bag.
```bash
users
```

To see what's inside of the data bag use the `show` option:
```bash
knife data bag show users
```

Output:
```
user1
user2
```

To see info about each user you can run:
```
knife data bag show users/user1
```

```
comment
gid
home
id
platform
shell
uid
```

Now let's use `knife` to interact with our new data bag
```
knife search users "*:*" 
```

This will return all of our users:
```
chef_type: data_bag_item
comment:   I am user 2
gid:       1
home:      /home/user2
...
```


If we want to search for a user that matches a certain criteria, such as all users that have `platform:centos`

```
knife search users "platform:centos"
```


## Create groups data bag
Just like with our `users` we are going to create a `groups` directory for our data bag `json` files.
```bash
mkdir data_bags/groups
```

Now create a data bag for groups on our Chef Server
```
knife data bag create groups 
```

We would then create a new `JSON` file for groups `group1.json`: 
```json
{
	"id": "group1",
	"gid": 2000,
	"members": ["user1","user2"],
	"platform": "centos"
}
```

Upload the group info 
```bash
knife data bag from file groups groups data_bags/groups/group1.json
```


Great!   Now let's use this data bag in a recipe

## Create users cookbook 

Make sure you're in the `chef-repo` directory
```
cd $HOME/chef-repo
```

Generate a new cookbook called `users`

Update the default recipe with the following: 
```ruby
search("users", "platform:centos").each do |user_data|
	user user_data['id'] do
    comment user_data['comment']
    uid user_data['uid']
    gid user_data['gid']
    home user_data['home']
    shell user_data['shell']
	end
end
```

To use this cookbook we are going to add it to our web role. 

Open up `web.rb` in your favorite editor and update the run_list: 
```ruby
run_list 'recipe[myusers]','recipe[workstation]','recipe[apache]'
```

Now using `berks` upload the new cookbook to the Chef Server 

Use `knife from role` command and point to the updated `web.rb`

Run `chef-client` on web1 to apply the changes. 

If everything was successful you should see the new user created on the node. 

To test run the following: 
```bash
grep user /etc/passwd
```

You should see: 
```
user2:x:101:1:I am user 2:/home/user2:/bin/bash
user1:x:100:1:I am user 1:/home/user1:/bin/bash
```


Great! Now you've created a data bag with your user info, and you've created a recipe which takes that user data and creates a local user on any server with the `web` role. 
