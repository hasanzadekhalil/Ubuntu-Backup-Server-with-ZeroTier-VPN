# Ubuntu Backup Server with ZeroTier VPN
# Configuring the connection with the VPN
First of all, we need to register to use Zerotier VPN as the first step.
After registration, we create a network and give a name to the network we have created.
We then choose an ip range for our vpn clients.
After that, we open our raspberry pi and connect using ssh connection.
We install the components required for Zerotier VPN on both of our machines.
```
curl -s https://install.zerotier.com | sudo bash

```

```
curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | gpg --import && \
if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" | sudo bash; fi
```
Then we connect both machines to the vpn network with the help of the 
```
zerotier-cli join *our-network-id*  command.
```
After the connection is successful, we go back to our Zerotier account and tick the auth option to see both machines from here.  After join reboot for testing.

and we test the connection by pinging it.

Now that we have successfully completed this process, we can move on to the next step.



# Installation Backup server using BackupPC software
I recommend you to do the operations I have mentioned here after you have read and watched the video. If you do not do one of the operations correctly, it may result in the system not working.

Let's start downloading the BackupPC software to the server machine.
```
apt-get install backuppc -y
```
In the PostFix installation we will encounter, let's select the Local Only box.

Let's continue without changing the next options. Let's start the installation by pressing the enter key.
After the installation is finished, let's change the password of the backuppc user that the application automatically creates.
```
htpasswd /etc/backuppc/htpasswd backuppc
```
Let's use the command to open the application automatically when the system starts.
```
systemctl enable backuppc
```
Let's try and see if it works using the command.
```
systemctl status backuppc
```
Since our backup time application will use ssh connection, we need to provide ssh connection between server and clients.
```
sudo su – backuppc
```
and we will create an ssh key with the ssh-keygen command.
```
ssh-keygen
```
Then we will display our key on the screen with the help of the following command.
```
/usr/bin/cat ~/.ssh/id_rsa.pub
exit
```

Then we will go to the client computer and make some settings related to ssh there.
```
sudo nano /etc/ssh/sshd_config
```

 We open the sshd_config file and paste the following commands there.
```
PermitRootLogin prohibit-password
PubkeyAuthentication yes
```
And press Ctrl+x, y, Enter

 Then we restart the ssh service.
 ```
sudo service sshd restart
```
 We run the following commands in order on the client computer.
 ```
sudo -i

[ ! -d ~/.ssh ] && mkdir ~/.ssh

sudo nano ~/.ssh/authorized_keys
```
 First of all, we paste the command I have mentioned below and the vpn ip of the server computer in the your-server-ip part of the command, and then the key that we have just seen on the server computer.
(We will use the ip of our server assigned by the VPN)
```
from="your-server-ip",no-agent-forwarding,no-port-forwarding,no-pty
```
and press Ctrl+x, y, Enter

Now we send ssh connection from server to client and test its working.
For this, we change the user and switch back to the backuppc user.
```
su backuppc
ssh root@VPN-ip
```
After making sure that it works, we end the ssh connection by pressing Ctrl+C and type exit.

 Now we are making Apache configurations. We open the backuppc.conf file with the help of the following command.
 ```
nano /etc/apache2/conf-available/backuppc.conf
```
and we replace the Require local line there with Require all granted.
Let's save the file and close it( Ctrl+x, y, Enter). Next, let's restart the Apache service to apply the changes.
```
systemctl restart apache2
```
 Now let's access the BackupPC application from the Web interface. 
Our-lan-ip/backuppc
# Attention!
Now that we have a VPN installed, our machines have two IP addresses. We need to access the web interface with our local ip. We will only use the ip assigned by the VPN at the time of the server-client relationship.
In the web interface, the user name is backuppc, and the password is the password we have set in step 4.

 After accessing the interface, we move to the Edit Host panel.
Here, in the hosts section, we write the vpn-assigned ip of our client computer.
We write backuppc in the user section and press the SAVE button.

 After this process, we switch from the Server section to the Xfer section, where we select rsync and save.
 Now, we select our client from the select a host section and then click on the edit config section. 
Here, we move to the Xfer section and put a tick in the override section under RsyncShareName and press the SAVE button.

 Now we can choose when to make backups in the schedule section of the edit config section. After completing the process, we press the start a full backup button and wait for the backup process to finish.
