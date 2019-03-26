# Cookbook Tests

In this lab you will write tests to confirm your chef runs are successful. 

We will be using `Docker` as our `driver` with Test Kitchen which means that the `Ubuntu` image we are going to use for our base will be minimal and will not have all the packages required for running our tests.

We need to install `docker` to because we are already using a `Vagrant` VM and cannot nest virtualization. 
```bash
curl -fsSL get.docker.com -o | sudo bash -s
sudo adduser vagrant docker
```

To run our port test we need to have the `net-tools` package installed.  Since we are using Chef, go ahead and update the `apache` cookbook `server` recipe to install the `net-tools` package.

## Test Kitchen Configuration 
Look in the `apache` cookbook directory and you will see a `.kitchen.yml` file.  Open it in your favorite editor and change it to look like: 
```yaml
---
driver:
  name: docker

provisioner:
  name: chef_zero

verifier:
  name: inspec

platforms:
  - name: ubuntu-18.04

suites:
  - name: default
    run_list:
      - recipe[apache::default]
    verifier:
      inspec_tests:
        - test/integration/default
    attributes:
``` 

All of the `kitchen` commands must be run from inside the cookbook directory `apache`

Create your infrastructure using `kitchen`
```bash
kitchen create
```

This will create an `Ubuntu 18.04	` `Docker` container. 

Confirm the container was created
```bash
docker ps 
```

Output: 
```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
196af8d6f524        a7a576d3993b        "/usr/sbin/sshd -D -…"   5 minutes ago       Up 5 minutes        0.0.0.0:32770->22/tcp   defaultubuntu1804-vagrant-chefdk-74wxg9fh
```

The container was created, now it is time to install `Chef` and apply the `apache` cookbook. 
```bash
kitchen converge
```

This may take a few minutes to complete because it is running the `chef-client` which installs all the packages we specified in our `server` recipe, and confirms the `apache2` service is enabled and running. 

Once that is complete log in and manually confirm the `apache2` service is listening 
```bash
kitchen login 
```


Check to make sure it is running, and responding on port `80`
```bash
kitchen@196af8d6f524:~$ curl localhost
<h1>Hello, world!</h1>
```

Ok, now that you've confirmed it works through manual steps it's time to create an automated test. 

## Create test file 
In your favorite editor open `apache/test/integration/default/default_test.rb`

In this file you will see some example tests which can be removed. 

Add the following to the file: 
```ruby
# check if apache is listening on port 80
describe port(80) do
  it { should be_listening }
end
```

Save the file and run `kitchen verify` to run the tests. 

If everything is successful you should see something like:
```bash
-----> Verifying <default-ubuntu-1804>...
       Loaded tests from {:path=>".home.vagrant.cookbooks.apache.test.integration.default"}

Profile: tests from {:path=>"/home/vagrant/cookbooks/apache/test/integration/default"} (tests from {:path=>".home.vagrant.cookbooks.apache.test.integration.default"})
Version: (not specified)
Target:  ssh://kitchen@localhost:32770

  Port 80
     ✔  should be listening

Test Summary: 1 successful, 0 failures, 0 skipped
       Finished verifying <default-ubuntu-1804> (0m0.47s).
```

Perfect!  Apache is listening on port 80 and our test was successful.  

Now what if we want to extend our test file to do more than just check port 80? 

Add the following test to `default_test.rb` to confirm `apache` package is installed. 
```ruby
describe package('apache2') do
  it { should be_installed }
end
```

Now run `kitchen verify` again and confirm it comes back successful.  


Great job!  You have just written your first automated test for Chef. 


## Bonus lab: Add additional tests
Now it's time for you to do a little research and enhance your tests so they are more in-depth. 

Update the `default_test.rb` so it: 
* Checks if the `apache2` process is running 
* Checks Apache returns a `200` response code
* Checks the headers to confirm they include `text/html`
* Confirms the `body` of our website includes 'Hello, world!' 

I understand we did not specifically talk about this, but take a look at the following resources and see if you can figure it out. 
* https://www.inspec.io/docs/reference/resources/http/
* https://www.inspec.io/docs/reference/resources/processes/

## Support multiple distributions
While it is great that we can install and test `apache` on Ubuntu, what if you have many different distributions that you need to manage with Chef? 

Let's spin up a `CentOS` instance and test it out. 


Add `CentOS` to your `.kitchen.yml` file. 
```ruby
platforms:
  - name: ubuntu-18.04
  - name: centos-7
```

Save the file and then run `kitchen verify` to create the new instance and run tests against it. 

What happened? 

It should have failed because our current cookbook and testing only works for `deb` based distros like `Ubuntu`, and `Debian` and does not support `RPM` based distributions like `CentOS`. 

You will spot error like these: 
```bash
*FATAL*: Chef::Exceptions::Package: yum_package[apache2] (hello_web::default line 6) had an error: Chef::Exceptions::Package: No candidate version available for apache2
```  

The package name for  Apache server is different across the platforms. To support alternative _package name_, alone with the _service name_, and _docroot_, we need to override the default values on each platform.

The instructor will perform a demo of how this can be accomplished. 

## Lab Complete! 
