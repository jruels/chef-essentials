# First cookbook

In this lab you will be creating your first cookbook, migrating your recipe to the new cookbook and then making some changes to the recipe. 

Start by creating a directory to store cookbooks in: 
```bash
mkdir -p /home/vagrant/chef/cookbooks
```

Enter the cookbooks directory and generate a cookbook using `chef`
```bash
chef generate cookbook workstation
```

This cookbook will be used to setup your workstation, take a look at the directories and files that were created. 
```bash
tree workstation
```

You should see something like: 
```bash
workstation
├── Berksfile
├── CHANGELOG.md
├── chefignore
├── LICENSE
├── metadata.rb
├── README.md
├── recipes
│   └── default.rb
├── spec
│   ├── spec_helper.rb
│   └── unit
│       └── recipes
│           └── default_spec.rb
└── test
    └── integration
        └── default
            └── default_test.rb
```


As we discussed during the lecture these files all have a specific purpose, but we are going to focus on the `recipes` directory for now. 

Copy the `setup.rb` file to the new `workstation/recipes` directory. 
```bash
cp $HOME/chef/recipes/setup.rb $HOME/chef/cookbooks/workstation/recipes/
```

Now you should have two files in `recipes` directory. 
```bash
ls -l workstation/recipes/
total 8
-rw-rw-r-- 1 vagrant vagrant 102 Mar 20 22:31 default.rb
-rw-rw-r-- 1 vagrant vagrant 185 Mar 20 03:48 setup.rb

```

Now update `setup.rb` to install the `git` package.

After you have updated `setup.rb` run `chef-client` to apply the changes. 
```bash
sudo chef-client -z workstation/recipes/setup.rb
```

Confirm `git` was installed 
```bash
which git 
```

Output: 
```bash
/usr/bin/git
```

After installing `git` let's go head and initialize our repository. 

Make sure you are in the `cookbooks/workstation` directory. 
```bash
cd $HOME/chef/cookbooks/workstation
```

Initialize the repo:
```bash
git init 
```

You should now see a new `.git` directory
```bash
ls -al 
```

Output: 
```bash
drwxrwxr-x 7 vagrant vagrant 4096 Mar 20 22:31 .
drwxrwxr-x 3 vagrant vagrant 4096 Mar 20 22:31 ..
-rw-rw-r-- 1 vagrant vagrant   77 Mar 20 22:31 Berksfile
-rw-rw-r-- 1 vagrant vagrant  160 Mar 20 22:31 CHANGELOG.md
-rw-rw-r-- 1 vagrant vagrant 1090 Mar 20 22:31 chefignore
drwxrwxr-x 3 vagrant vagrant 4096 Mar 20 22:31 .delivery
drwxrwxr-x 8 vagrant vagrant 4096 Mar 20 22:31 .git
```

Now that we've initialized the repository and are tracking the cookbook in version control make a change to the `setup.rb` recipe and add it to `git`

Update `setup.rb` so it installs the `cowsay` package
```ruby
package 'cowsay'
```

Run `chef-client` to apply the recipe 

Confirm `cowsay` was installed 
```bash
cowsay "my first cookbook"
```

Now add the updated `setup.rb` to `git`
```bash
git add recipes/setup.rb
git commit -m 'install cowsay'
```

Perfect!  Now we need to update the version for our `workstation` cookbook. 

Open `metadata.rb` in your favorite text editor and increment the version number so it matches semantic versioning. 

Now we need to add that change to `git` for tracking 
```bash
git add metadata.rb
```

Check out the working tree: 
```bash
git status 
```

You should see something like: 
```bash
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   metadata.rb

```

Now you can see that the file is being tracked let's commit it.
```
git commit -m 'increment version'
```

Congrats! In this lab you created your first cookbook, copied a recipe over to it and setup tracking in Git. 

## Interacting with cookbooks
Now let's interact directly with cookbooks instead of providing a path to a recipe to `chef-client`. 

We have a workstation we would like to setup, it must have:    

```
Git 
tree
ntp
``` 

File content: 

```
The file `/etc/motd` must have the text "This server is being managed by Chef"
```

Service:

```
The `ntp` service must be started and enabled to start-on-boot. 
```

The workstation will also be a web server and requires: 

* Apache
* `/var/www/html/index.html`
	
* with content: 

```html
<h1>Hello, world!</h1>
```

The Apache service must be running and enabled to start-on-boot.

We have a couple options to accomplish this.   We could run `chef-client` and point to `setup.rb` and `server.rb` recipes individually, or we could point to our cookbooks and use them to populate the `runlist`

When calling a cookbook without specifying a recipe it will call `default.rb`, so we need to update this file to include our `setup.rb` recipe.   To do this we use `include_recipe`. 

Open  `default.rb` in the `workstation` cookbook in your favorite text editor and add the following: 
```ruby
include_recipe 'workstation::setup'
```

Now when we call `chef-client` we will tell it to run the `workstation` cookbook. 
```bash
sudo chef-client -z -r "recipe[workstation]"
```

If everything is setup properly you should see something similar to: 
```bash
Recipe: workstation::setup
  * apt_package[ntp] action install (up to date)
  * apt_package[cowsay-off] action install (up to date)
  * apt_package[tree] action install (up to date)
  * apt_package[git] action install (up to date)
  * file[/etc/motd] action create (up to date)
  * service[ntp] action enable (up to date)
  * service[ntp] action start (up to date)
```

As you can see the `default` recipe called the `setup` recipe. 


Now we are going to create an Apache cookbook that can be used to install and configure a web server. 

We need to run the `chef generate` command from the `chef` directory. 
```bash
cd $HOME/chef
```

Use `chef generate` to create the new cookbook. 
```bash
chef generate cookbook cookbooks/apache
```

After generating the new cookbook, create a new recipe called `server.rb`
```
chef generate recipe cookbooks/apache server
```

Open `server.rb` in your favorite editor and update it so that it installs the `apache2` package, starts it, enables it to start at boot and creates `/var/www/html/index.html` and populates it with:
```html
<h1>Hello, world!</h1>
```

Do NOT run this recipe, we are going to call it from our `default` recipe.

Now update the `apache` cookbook's `default.rb` to call the `server.rb` using `include_recipe` just like we did for our `workstation` cookbook. 

After you have updated it use `chef-client` to call the `apache` cookbook and confirm it performed the required tasks: 
```bash
curl localhost 
```

output: 
```bash
<h1>Hello, world!</h1>
```

Now update the cookbook versions and check your changes into `git`

## Bonus lab 
As an additional lab, figure out how to run both recipes in a single command. 

## Lab Complete