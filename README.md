# Setup TMSTATS

Install everything you need to work on TMSTATS on your own Ubuntu image

## Getting Started

Follow this guide to install TMSTATS on your Ubuntu image.  This will
include installing Python 3.7, MySQL 5.7, Apache, PHP 7.2, and a copy
of the D101TM.ORG WordPress image.

__Please note__ that you do __not__ download this package to your machine until Step 11 of these instructions!

### Prerequisites

You need to have a virtual machine (or a real one!) with Ubuntu 18.10.
You can use either a server or a desktop image; if you use desktop, you
will need more storage (both memory and disk space) on your host machine.

### Part I: Install and Update the Base System


#### Step 1: Install Ubuntu

##### If you choose Ubuntu Desktop

Set your VM size to 2GB, and have at least a 10GB Disk.
Set your network adapter to "Bridged Network".

Install a "minimal system"; select the "update during installation" and "install 3rd party" checkboxes.

##### If you choose Ubuntu Server

Set your VM size to 1GB, and have at least a 10GB Disk.
Set your network adapter to "Bridged Network".

#### Step 2: Update Software

##### Ubuntu Desktop

When the system reboots, it will ask you to connect your online accounts - I suggest not doing so.

You can choose whether or not to send reports to Canonical.

After rebooting, the Software Updater will automatically start.  Let it update software and then reboot.

I suggest running the Settings program and turning off automatic display blanking.

##### Ubuntu Server

Log in and issue these commands:

```
sudo apt-get update
sudo apt-get upgrade
```

Respond to prompts as needed.

When the installation is finished, reboot.

#### Step 3:  Update sudoers

Issue `sudo visudo` and change all instances of " ALL" to " NOPASSWD:ALL"

Save the file.

#### Step 4:  Install appropriate guest VM support

##### If you're using VirtualBox

You want to install the VirtualBox Guest Additions before you proceed further.

Open a terminal window and issue this command:

`sudo apt-get install gcc make perl`

Then go to the "Devices" menu for your VM and insert the Guest Additions image; let it install itself and reboot.

If you are using a shared folder with your guest, you also need to give yourself access to the folder.  Issue this command:

`sudo adduser $USER vboxsf`

Go to the Machine/Devices menu item and enable a bidirectional clipboard.

You can eject the Guest Additions DVD from your virtual machine at this point.

##### If you're using another virtualization program

Proceed according to that program's instructions.

#### Step 5:  Install or create your SSH keys

##### If you already have a public/private key pair

If you already have a public/private key pair (~/.ssh/id_rsa and ~/.ssh/id_rsa.pub) on your host machine, copy them to ~/.ssh/ on the guest, then issue this command on the
guest:

`cd ~/.ssh;cat id_rsa.pub >> authorized_keys`

That will allow you to SSH into the guest from your host machine, and if you have your public key in ~/.ssh/authorized_keys on the host, you can go the other way (this is
mostly useful for copying files).


##### If you do not have a public/private key pair

You will need to generate a keypair on the guest machine.  [This page](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804) is a guide to doing so.  In short:

1. Issue `ssh-keygen`
2. Hit RETURN for all the prompts, accepting the default location and not creating a passphrase.
3. Issue `cd ~/.ssh;cat id_rsa.pub >> authorized_keys`

If you want to use the keys you've just generated on your host system, copy them to ~/.ssh on that system and add the id_rsa.pub file to the host's authorized_keys.



#### Step 6: Send your PUBLIC key to webmaster@d101tm.org

Send an email to webmaster@d101tm.org and attach (or just copy in) ~/.ssh/id_rsa.pub
I will add your public key to the necessary files on d101tm.org and let you know when it's ready to use.


#### Step 7: Install your PUBLIC key on GitHub

Follow the procedure [here](https://help.github.com/en/articles/adding-a-new-ssh-key-to-your-github-account).


#### Step 8:  Add your userid to the www-data group

This will give you access to all of the files in the webserver's directory.  Issue:

`sudo adduser $USER  www-data`

#### Step 9: Install Git and make your ~/src directory

Install git:

`sudo apt-get install git`

Tell Git your email address and name:

```
git config --global user.name "Your Name"
git config --global user.email "user@domain"
```

Create a directory to hold the Git repositories:

`mkdir ~/src`

#### Step 10:  Take a snapshot of your VM

This will give you a good recovery point if you need it. Restarting the VM also ensures that your user is an active member of any groups you added it to.

Shutdown the VM (issue `sudo poweroff`) and take a snapshot.

Now, you need to wait until the Webmaster tells you your public key has been added to the server.


### Part II: Clone the D101TM.ORG server and TMSTATS code

#### Step 1:  Reboot your machine and verify your connection to the d101tm.org server

Reboot your machine and log in.

Make sure your public key has been added to the server by issuing this command:

`ssh d101dev@d101tm.org 'echo Remote user is $USER'`

You may get a prompt like this:

```
The authenticity of host 'd101tm.org (69.163.170.135)' can't be established.
ECDSA key fingerprint is SHA256:rWqwh967MZCQNhaBNBJ6bFWInpLjUkk1l+LW2VZD1+E.
Are you sure you want to continue connecting (yes/no)? 
```

Reply 'yes'; the system will tell you that it's added the key to the list of known hosts.

You should see a message: `Remote user is d101dev`.  If so, you're properly set up and can continue; otherwise, check with the Webmaster to make sure your public key has been added to the server.

#### Step 2:  Fork d101tm/tmstats on GitHub and clone your fork to the VM

Log into GitHub and make a fork of d101tm/tmstats. This will be where you do your development and debugging; when you want to get your work onto the production server, you'll
push it to your fork and make a pull request to have it merged into the d101tm/tmstats repository. This will give someone else on the team a chance to look at your code
(especially for major changes) before we put it into production.

Once you fork the code on GitHub, clone it to the VM with this command (replace USERNAME with your GitHub username, of course):

```
cd ~/src
git clone git@github.com:USERNAME/tmstats.git
```

You may get a prompt like this:

```
The authenticity of host 'github.com (192.30.255.113)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)?
```

Reply 'yes'; the system will tell you that it's added the key to the list of known hosts.

#### Step 3:  Copy the setup programs to your VM

```
cd ~/src
git clone git@github.com:d101tm/tmsetup.git
```

#### Step 4:  Complete the installation and copying

Issue these commands:


```
cd ~/src/tmsetup
./fullinstall > install.log
```

This will install and configure Apache, MySQL, PHP, Python, the D101TM.ORG website and code, and the TMSTATS code and data. Detailed output will be in `install.log`; you will get progress reports on your screen. If there are any prompts, reply appropriately (but I hope there aren't).


## Your machine now has a complete version of D101TM.ORG

This includes the TMSTATS code (in ~/src/tmstats).  

The administrator of the WordPress site is 'd101dev' with a password of 'd101dev'.


