This guide closely follows [https://yarra-framework.org/server/installation/](https://yarra-framework.org/server/installation/). Some very minor changes.

## General Server Installation
This assumes you are running Ubuntu 20.04.

Install the following packages:
```
sudo apt-get update
sudo apt-get install openssh-server
sudo apt-get install dcmtk
sudo apt-get install sendmail
sudo sendmailconfig
sudo apt-get install unzip
sudo apt-get install qt5-default
```

Add the user “yarra” for access from the Yarra clients, and the user “yarraserver” to run the YarraServer software. Both users should be added to the user group “yarra”. By setting the shell to “/bin/false” for the user yarra, it is prevented that users can open a shell login on the server (note that the password for user yarra can be seen in the configuration file of the Yarra client). Therefore, the user yarra must have access only to the Yarra queue share. The users yarra and yarraserver must never have the same password! It is not recommended to run the YarraServer software with root rights.
```
sudo groupadd yarra
sudo adduser --ingroup yarra --shell /bin/false yarra
sudo adduser --ingroup yarra yarraserver
```

## Installation of the YarraServer package
Login with the account yarraserver and create the subfolder in the home directory, e.g. ~/yarra. It is recommended to set the permission of the folder to 750, so that only the user yarraserver and users of the group yarra can access the folder. Depending on the configuration/installation of your server, it can be advantageous to install the software at a different location (e.g., if the disk drives have been partitioned with only limited space for the home directories). In the following, it is assumed that the installation location is /home/yarraserver/yarra. This is the only part that is different to the original documentation.  And indeed, probably worth putting in a serperate drive.
```
su yarraserver
mkdir /home/yarraserver/yarra
chmod 750 /home/yarraserver/yarra
cd /home/yarraserver/yarra
wget https://s3.amazonaws.com/download.yarraframework.com/yarraserver_097.zip
chmod 750 *
chmod 1770 queue
chmod 440 queue/YarraModes.cfg
chmod 440 queue/YarraServer.cfg
exit
```

## Installation and Configuration of Samba 
Yarra clients connect to the server using a SMB network share. To provide the share, install the Samba package and set the Samba password for the user yarra. This should be the *same password that was used when creating the user*:
```
sudo apt-get update
sudo apt-get install samba
sudo smbpasswd -a yarra
```
Edit the file /etc/samba/smb.conf. This can be done using the nano command (when the desktop edition of Ubuntu has been installed, this can also be conveniently done using the editor gedit):
```
sudo nano /etc/samba/smb.conf
```
Add the following section to the end of the file:

```
[YarraServer]
path = /home/yarraserver/yarra/queue
available = yes
valid users = yarra
read only = no
browseable = no
public = no
writable = yes
create mask = 0770
directory mask = 0400
force user = yarraserver
force group = yarra
```

In addition, change the following setting in the file:

```
############ Misc ############
usershare allow guests = no
```

Save and close the file. Then restart the samba server.
```
sudo systemctl restart smbd
```

Done! Now go [set up your clien](https://github.com/Sydney-Informatics-Hub/yarra/blob/main/YarraClient.md). 
