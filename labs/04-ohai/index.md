# Chef Node data


In this lab you are going to populate a file using `node` object attributes. 

## Ohai
Run some commands with Ohai to gather the following information 
* Operating System 
* MAC address for `eth0`

After you are comfortable with using `ohai` for finding node information update the `setup.rb` recipe to populate `/etc/motd` with the following: 
* CPU speed in MHz 
* Total memory
* Hostname
* IP address 

Remember to enclose the `file` resource `content` in double quotes `" "` so that string interpolation works. 

After you have updated the `setup.rb` with `node` object variables run `chef-client` and confirm `/etc/motd` has the required information. 


## Apache server update
Now that you've updated the `/etc/motd` file with the requested info, go ahead and update the Apache `server.rb` so that it populates `index.html` with: 
* Hostname
* IP Address 
* Ruby version 

Run `chef-client` to apply the changes and then `curl` localhost to confirm the values were populated correctly. 

## Lab Complete 