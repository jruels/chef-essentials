# Add additional web node

It's great that the load balancer is loading traffic to web1, but the real purpose of a load balancer is to BALANCE traffic between multiple nodes.  In this lab you will be:
* Bootstrapping `web2`, 
* Updating load balancer server pool to include `web2` 
* Uploading updated `myhaproxy` cookbook to Chef Server 
* Apply the updated cookbook on load balancer node.

Using the steps provided previously bootstrap web2 node, remembering to get the port and identityfile from Vagrant. 

After you bootstrap web2 log in and confirm you can `curl localhost`.

Now it's time to update the load balancer server pool.  Take a look at `default.rb` in the `myhaproxy` cookbook and you should see a server pool defined. Add web2's IP to it and then update the cookbook version.  After updating the cookbook version use Berkshelf to upload the cookbook. 

Apply the updated cookbook to load balancer and then confirm when you `curl localhost` it round-robins between web1 and web2. 

Congrats!  You now have a fully functional load balancer sending traffic to your backend web servers. 

## Lab Complete 
 
