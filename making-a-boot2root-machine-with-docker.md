# Making a Boot2root machine with docker

I've been trying to learn about Docker recently and experiment with it and I though what would be a better way to test my basics than to make a dockerized boot2root machine, so here we are.

The boot2root machine we'll be implementing is a very simple one

* [ ] Find ftp and ssh service are open through nmap
* [ ] get user creds from ftp and login via ssh
* [ ] and a very simple sudo priv esc

### Base Setup

The first thing to do would be install docker which you can install from [here](https://www.docker.com/) and then pull a ubuntu latest image from docker to work with.

```shell
docker pull ubuntu
docker images (gives image ID's for all repos)
docker run -cap-add=NET_ADMIN -it <ubuntu image ID> 
```

Following the above you'll be inside the docker image with root privileges. One thing you should remember is that this is image is stripped of evertyhing nothing is installed from the start. You have to install the basic needed tools and ones you're gonna use for boot2root manually incuding `sudo` .

```shell
apt-get update  
apt-get install -y vsftpd wget curl nano sudo ufw ftp openssh-server
```

Now you have to manually start all the services that you'll be using in this machine which are FTP and SSH

```shell
service ufw start
service ssh start
service vsftpd start
```

Now we have to configure the ports for ftp and ssh to allow traffic through them,

```shell
sudo ufw allow 20/tcp 
sudo ufw allow 21/tcp  
sudo ufw allow 22/tcp  
```

### Setting up FTP server

Without setting the root directory for the ftp it'll just show empty when you connect to it.

We need to make a `ftpuser` and also make a directory in `/var` for the ftp server and give it appropriate permissions.

```shell
sudo useradd <username>
sudo mkdir -p /var/ftp/pub
sudo chown nobody:nogroup /var/ftp/pub
```

We also need to edit the `vsftpd.conf` file to allow anon login and also set the root directory. You can open the file with

```shell
nano /etc/vsftpd.conf
```

edit the config file with&#x20;

```c
anonymous_enable=YES
local_enable=YES

anon_root=/var/ftp/pub/
hide_ids=YES
```

Now just add a file withe user creds you wanna give for ssh or any other hint

```shell
nano /var/ftp/pub/creds.txt
```

At the end you have to restart the service for the changes to take place

```shell
service vsftpd restart
```

### Setting up SSH

```shell
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.factory-defaults
sudo chmod a-w /etc/ssh/sshd_config.factory-defaults
service ssh restart
```

These commands make a backup copy of our config, and gives it permissions so that it cant be tampered with and restarts the service with all the implemented changes.

Make lower privilege user for the user flag

```
useradd -m <username> (creates user with home directory)
usermod --shell /bin/bash <username> (sets bash as shell for the user)
passwd <username> (set the password for the user)
```

### Setting the sudo Privesc

### Sanity Check

Do a sanity check to see if everything you restarted and updated is working perfectly. We need to get the ip-address of the container to test it out. We can do this by

```shell
docker container ls
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-id>
```

After you get the ip you can do your checks with the ftp server and ssh with the accounts you created. If everything is working fine we can move on to commit the image and push it to docker hub.

### Pushing to docker hub and making docker-compose files

