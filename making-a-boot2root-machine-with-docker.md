# Making a Boot2root machine with docker

I've been trying to learn about Docker recently and experiment with it and I thought what would be a better way to test my basics than to make a dockerized boot2root machine, so here we are.

The boot2root machine we'll be implementing is a very simple one

* [ ] Find ftp and ssh service are open through nmap
* [ ] get user creds from ftp and login via ssh
* [ ] and a very simple sudo priv esc

### Base Setup

The first thing to do would be install docker which you can install from [here](https://www.docker.com/) and then pull an Ubuntu latest image from docker to work with.

```shell
docker pull ubuntu
docker images (gives image ID's for all repos)
docker run -cap-add=NET_ADMIN -it <ubuntu image ID> 
```

Following the above you'll be inside the docker image with root privileges. One thing you should remember is that this is image is stripped of everything nothing is installed from the start. You have to install the basic needed tools and ones you're gonna use for boot2root manually including `sudo`&#x20;

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

We also need to edit the `vsftpd.conf` file to allow anon login and also set the root directory.&#x20;

{% code title="/etc/vsftpd.conf" %}
```c
anonymous_enable=YES
local_enable=YES

anon_root=/var/ftp/pub/
hide_ids=YES
```
{% endcode %}

Now just add a file with user creds you wanna give for ssh or any other hint

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

```shell
useradd -m <username> #creates user with home directory
usermod --shell /bin/bash <username> #sets bash as shell for the user
passwd <username> #set the password for the user
```

### Setting the sudo Privesc

For privilege escalation I thought of letting the user run `cat` and `ls` meaning he can list and read the files in the root directory.

To do this we have to edit the sudoers file, run the below command as root

```shell
visudo
```

and add the following to your user

{% code title="/etc/sudoers" %}
```
<username>    ALL=(ALL) NOPASSWD:/bin/cat, /usr/bin/ls
```
{% endcode %}

add your root flag to root.txt in the `/root` directory and then we just have add appropriate permissions to the user and root flags and we are good to go

```shell
sudo chown <username>:<username /home/<username/user.txt
sudo chmod u+rx /home/<username/user.txt
sudo chmod u-w /home/<username/user.txt
sudo chmod go-rwx /home/<username

sudo chown root:root /root/root.txt
sudo chmod u+rx /root/root.txt
sudo chmod u-w /root/root.txt
sudo chmod go-rwx /root
```

### Sanity Check

Don't forget to restart all the services before doing the sanity check

```shell
service ufw restart
service ssh restart
service vsftpd restart
```

Do a sanity check to see if everything you restarted and updated is working perfectly. We need to get the ip-address of the container to test it out. We can do this by

```shell
docker container ls
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-id>
```

After you get the ip you can do your checks with the ftp server and ssh with the accounts you created. If everything is working fine we can move on to commit the image and push it to docker hub.

### Pushing to docker hub&#x20;

After all this the one thing we need to do is make your docker container available to the whole world wide web. To do this we have to commit your docker container.

Exit your container after making all the changes and testing it.&#x20;

```shell
docker ps -a
```

Use the above command to get the container-id of the container you just exited.

```shell
docker commit <container-id> <image-name>
```

You can now check your new image using&#x20;

```shell
docker imaages
```

To upload this onto docker hub, you first need to make an account on [Docker Hub](https://hub.docker.com/). Click on your profile image and go to account settings. Click on Security in the left navigation and add New Access Token. Copy your access token and back in your terminal do the following

```shell
docker login -u <username>
```

you'll be prompted to enter the access token.\
Now to upload on to docker hub you need to tag it and then push it.

```shell
docker image tag <image-name> <username>/<image-name>:latest
docker image push <username>/<image-name>:latest
```

### Making docker-compose file

The docker-compose file for this image is very simple

* [ ] We'll be pulling the image from docker hub
* [ ] adding the additional capability to it `--cap-add=NET_ADMIN`&#x20;
* [ ] run a command to restart the services `ufw`, `vsftpd` and `ssh`&#x20;
* [ ] add a tails command to keep the container from exiting in detached mode

{% code title="docker-compose.yaml" overflow="wrap" %}
```yaml
version: '2'

services:
    app:
        cap_add:
        - NET_ADMIN
        image: <username>/<image-name>
        command: sh -c "service ufw restart && service vsftpd restart && service ssh restart && tail -f /dev/null"
```
{% endcode %}

you just need to run the below command in the same directory as the file for the container to be up and running

```shell
docker-compose up
```

If you make any changes to to `.yaml` file make sure to take the container down before running it again

```shell
docker-compose down
docker-compose up
```

Hope you learnt  something about docker or about configuring boot2root machines, If you're interested check out my other blogs too.
