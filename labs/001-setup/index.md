# Setup Lab Environment

The lab environment requires VirtualBox and Vagrant to run.   

VirtualBox is a hypervisor that allows running Virtual Machines on your local workstation and will provide us a lab environment that can be used at any time without having to pay for additional hardware or cloud instances.

Vagrant is a way to automate building the lab environment, it will download the required virtual machine images, configure a private network that they can all communicate on and enable remote access to each of the VMs. 


## Download and install VirtualBox 
VirtualBox is an application that allows you to run VMs on your local machine.  Download it from https://www.virtualbox.org/wiki/Downloads

After downloading VirtualBox continue to the Windows or Mac section below 

### Windows 
Installing VirtualBox on Windows is straight forward and only takes a few steps which are outlined below. 

Run the executable by double clicking on it, and you’ll see a window like below: 

![](index/84F73F72-C3AD-4AA1-AA74-16C961AC10D4%203.png)

Click “Next” to begin the installation.  During the installation you’ll receive a network warning like below.  This just means you will be disconnected from the network for a second. 


![](index/DA61E1DF-2025-4E9D-8D24-D96EFB32AE4F%203.png)
Go ahead and click “Yes” to continue the installation 

![](index/CE562B58-AC9E-4847-B284-EAB2F8DE6DCD%203.png)
Click Install and you’ll see a screen confirming you trust Oracle, and want to install Universal Serial Bus device.  Click “Install” to continue the installation. 

![](index/8E290751-A047-4764-9ABC-56CC12F42A42%203.png)


Now continue the wizard until installation is complete. 

### Mac 
To install VirtualBox on MacOS double-click the DMG file downloaded in the previous step to open it and you’ll see the following: 
![](index/2F555227-E253-4CD2-A594-57BBA217FE71%203.png)

Follow the instructions and double-click the `Virtualbox.pkg` file to start the installation wizard. 

Click “Continue” in the pop-up window and then again to start the installation.  At this point you may seen an error saying it couldn’t install VirtualBox and to open Security preferences on your Mac.  Click this button, click the lock in the bottom left corner, type in password, and then click “Allow”.  
![](index/32259CA9-445F-45D3-A30F-A2BEF8BF92CD%203.png)

Rerun the installation and it should succeed. 
![](index/CFDA30BF-C1A0-4F62-B812-7E890AC42763%203.png)
  
## Download and Install Vagrant
To download Vagrant load https://www.vagrantup.com in a browser and download latest version.


![](index/F6B433F5-5489-402C-984E-D9F0AA5F045B%203.png)

This will take you to a screen where you choose the Operating System you want to download it for. 


![](index/17AC5178-4DBC-4AE1-AFD9-18C7B1519ECB%203.png)

Start the installation and go through the simple steps in the wizard. 


![](index/ED379ECA-B818-4523-9CBE-9D3EB8631B3B%203.png)

## Reboot system 
Now reboot your machine to apply all the network changes `Vagrant` made. 

## Confirmation 
Now confirm everything was installed successfully. 

Open a terminal and type 
```
vagrant version
```

You should see something like: 
```
Installed Version: 2.2.4
Latest Version: 2.2.4

You're running an up-to-date version of Vagrant!
```

## Download the `Vagrantfile`
Now that you have `Vagrant` installed go ahead and start up the lab environment VMs. 

Enter the working directory
```
cd labs/
```

Start up VMs 
```
vagrant up 
```

This will take a few minutes to complete because it has to run some scripts to install Chef and it's dependencies. 

After the VM has completed booting and been provisioned run the following to `SSH` into the `chefdk` VM. 
```
vagrant ssh chefdk 
```

Confirm it is installed successfully. 
```
chef --version 
```

You should see something similar to: 
```
Chef Development Kit Version: 3.8.14
chef-client version: 14.10.9
delivery version: master (9d07501a3b347cc687c902319d23dc32dd5fa621)
berks version: 7.0.7
kitchen version: 1.24.0
inspec version: 3.6.6
```


# Lab Complete