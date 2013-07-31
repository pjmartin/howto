# Linux Configuration

## SSH Config 

### Server configuration

Configure `/etc/ssh/sshd_config`

	PermitRootLogin no        #Prevent root login with ssh
	AllowUsers patrik bob     #Only allow patrik and bob to login
	AllowGroups sshusers      #Allow members of sshusers group to login with ssh 
	Protocol 2                #Set the sshd protocol 2
	port 22                   #Set the SSH port

Configure IPtables to accept incoming traffic from specified IP or network.

	root# iptables -A INPUT -p tcp -s 192.168.100.41 --dport 22 -j ACCEPT
	root# iptables -A INPUT -p tcp -s 172.16.10.0/24 --dport 22 -j ACCEPT


#### Certificate login

Before doing this you should make sure that you have public keys specified in the users ~/.ssh/authorized_keys file.

turn off password login `/etc/ssh/sshd_config`

	PasswordAuthentication no  				# Deny password login with ssh
	PubkeyAuthentication yes  				# Allow login with private key
	AuthorizedKeysFile .ssh/authorized_keys # Specify where the public keys are stored


### Client configuration

##### Certificate

Create a certificate for the user

	$ ssh-keygen -t rsa -C "User Name"
	
set private key permission in user home directory

	local$ chmod 700 ~/.ssh
	local$ chmod 600 ~/.ssh/id_rsa 

Copy public key to server

		*If you created the key on remote server*
	server$ cat id_rsa.pub >> ~/.ssh/authorized_keys
		*or if you are on your own computer and send you public key to remote server*
	local$ ssh-copy-id username@remoteserver-ip
		*or just copy content of id_rsa.pub to authorized_keys on remote server*
		
set public key permissions.

	server$ chmod 700 ~/.ssh
	server$ chmod 600 ~/.ssh/authorized_keys

Start a ssh-agent and store the keyfile in the agent so you dont need to write password 
multiple times if you are using it to connect to multiple servers. It is possible to 
create a script in `~/.bashrc`or `~/.bash_profile` and let it automatically start the ssh-agent.

	local$ ssh-agent bash
	local$ ssh-add .ssh/id_rsa
	
##### SSH Config

create a config-file to make your remote connect easier edit `~/.ssh/config`

	Host git.mydomain.com
    	User git
    	HostName 192.168.100.11
    	IdentityFile ~/.ssh/id_rsa
    	IdentitiesOnly yes

	host forwarder
		HostName 192.168.100.12
		User patrik
		IdentityFile ~/.ssh/id_rsa
		LocalForward localhost:8080 192.168.200.40:8080 # Forward localhost:8080 to 192.168.200.40:8080
		LocalForward 192.168.1.12:8443 172.16.77.12:443	# Listen on port 8443 on local ip

##### SSH Tunneling

Use your local key for connection to 192.168.100.20 and then jump to 192.168.200.20 with key from local machine

	$ ssh -A -t 192.168.100.20 ssh -A -t 192.168.200.20

There are three possibilities for tunnelling :

   1. Tunnel from localhost to host1:

        ssh -L 9999:host2:1234 -N host1
        connection from host1 to host2 will not be secured.

   2. Tunnel from localhost to host1 and from host1 to host2:

        ssh -L 9999:localhost:9999 host1 ssh -L 9999:localhost:1234 -N host2
        # This will open a tunnel from localhost to host1 and another tunnel from host1 to host2.
        # However the port 9999 to host2:1234 can be used by anyone on host1. This may or may not be a problem.

   3. Tunnel from localhost to host1 and from localhost to host2:

        ssh -L 9998:host2:22 -N host1
        ssh -L 9999:localhost:1234 -N -p 9998 localhost
        # This will open a tunnel from localhost to host1 through which the SSH service on 
        # host2 can be used. Then a second tunnel is opened from localhost to host2 through the first tunnel.
        
    [Source](http://superuser.com/questions/96489/ssh-tunnel-via-multiple-hops)
    
## User Administratoion


Add User: `# useradd patrik` will add a user with a GUID to `/etc/passwd` and a home folder.

Add group: `# usermod -G wheel,sshusers -a patrik` will add patrik to the wheel and sshusers group.

display users groups: `groups patrik` 

See who is logged in: `who`

Advanced adduser `# useradd -g users -G wheel,sshusers -d /home/user01 -m -c "Comment" -s /bin/bash -p 'Crypt-PWD' patrik`

`-g` = default group

`-G` = other groups

`-d` = home directory

`-m` = create home directory

`-c` = comment

`-s` = shell

`-p` = crytp password, can be copied from other `/etc/shadow` file
	
set password: `passwd patrik`

##### Password Ageing


	chage -M 90 username	# The password will expire in 90 Days.
	chage -M 99999 -E 99999 username		#The password never expires
	passwd -e username		# Expire the current password. Useful for password resets and new accounts.


## Sudo

Administrators have the privilege to become root or use sudo command. To set up sudo users
you type the command `visudo` as root or `sudo visudo` if you are a sudoer.

Basically there are two lines which specify the sudoers. First is all users in wheel group
can use sudo with pasword. The second line is that all users in wheel can use sudo without
a password. But the easy way is never recommended.

	## Allows people in group wheel to run all commands
	%wheel  ALL=(ALL)       ALL

	## Same thing without a password
	# %wheel        ALL=(ALL)       NOPASSWD: ALL

Sudo makes you get to do super user stuff that root does. Usually you put sudo before a 
command like `sudo service network restart` to be able to restart the network

To get a root shell account with sudo you use

## Networking

## Yum repositories

## IPtables

## NAT/Routing



1.
--
	echo 1 > /proc/sys/net/ipv4/ip_forward

2. /etc/sysctl.conf
	net.ipv4.ip_forward=1

3. eth0 = outside, eth1 = inside
	# /sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
	# /sbin/iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
	# /sbin/iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT





-------------------------------
--    USER Administration    --
-------------------------------
Add User:
	useradd patrik

Add group:
	usermod -G wheel,sshusers -a patrik

Show groups:
	groups patrik

-------------------
--  Sudo users   --
-------------------
Install sudo:
	yum -y install sudo

command: visudo 

	## Allows people in group wheel to run all commands
	%wheel  ALL=(ALL)       ALL

------------------
-- Set hostname --
------------------

vim /etc/sysconfig/network 

	NETWORKING=yes
	HOSTNAME=bigboy
	GATEWAY=192.168.1.1

---------------
-- Epel Repo --
---------------


Centos 5.x

wget http://dl.fedoraproject.org/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-5.rpm
sudo rpm -Uvh remi-release-5*.rpm epel-release-5*.rpm
Centos 6.x

wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm


$ ls -1 /etc/yum.repos.d/epel* /etc/yum.repos.d/remi.repo
/etc/yum.repos.d/epel.repo
/etc/yum.repos.d/epel-testing.repo
/etc/yum.repos.d/remi.repo
Enable the remi repository

The remi repository provides a variety of up-to-date packages that are useful or are a requirement for many popular web-based services.  That means it generally is not a bad idea to enable the remi repositories by default.

	Change the enabled=1 

sudo vim /etc/yum.repos.d/remi.repo

name=Les RPM de remi pour Enterprise Linux $releasever - $basearch
#baseurl=http://rpms.famillecollet.com/enterprise/$releasever/remi/$basearch/
mirrorlist=http://rpms.famillecollet.com/enterprise/$releasever/remi/mirror
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi
failovermethod=priority
You will now have a larger array of yum repositories to install from.




## X Windows

set sshd_config to allow remote X11
run gnome-session
If it desktop does not start su root and do following
touch /var/run/console/<username>
chmod 600 /var/run/console/<username>
now try to run gnome-session again.
