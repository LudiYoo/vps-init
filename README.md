# Virtual Private Server Init

__The recommendations given here are only for experienced developers / administrators.  
Please only use this if you know what you are doing.__  
The settings may hinder you in future developments, but ensure more security.  
Everyone decides for himself whether and when more emphasis is placed on safety or on comfort.

## VPS features you should have
 - Domain SSL permanently included
 - Snapshots
 - Free Product Switch (switchable / upgradable OS)
 - IPv6
 - Repair Mode (Starts minimal system with ssh access and mount your system)

## Keep your vps up2date
```shell script
apt update && apt upgrade
```

## Git
Lets track & save a history of changes 

```shell script
apt install git
cd /
git init
```

Use the [.gitignore](server/.gitignore) file from this repository on the server  
After adding the file check the status and make an init commit;  
Push it on a private remote repository. 

## Timezone
Configure timezone
```shell script
dpkg-reconfigure tzdata
```

## Create new user

### Login user
Create a custom login user
```shell script
adduser username
```
Give him admin privileges
```shell script
usermod -aG sudo username
```
Allow only your login user to login and disable root login.
Set / Add this lines in /etc/ssh/sshd_config
```shell script
AllowUsers username
PermitRootLogin no
```
Restart service to take effect
```shell script
/etc/init.d/sshd restart
```

#### Add key login

Create folder structure with recommended permission
```shell script
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
Add your key in authorized_keys

#### Extend sshd settings

Take a look at https://infosec.mozilla.org/guidelines/openssh
Example file: [sshd_config](server/etc/ssh/sshd_config)
Notice: Dont just copy this example file, otherwise you lock yourself out.

Add this settings in /etc/ssh/sshd_config:
```shell script
# Only allow this users to login:
AllowUsers username

# Password based logins are disabled - only public key based logins are allowed.
AuthenticationMethods publickey

# Allow only strong algorithms
KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
```


## Firewall

Setup ufw (Uncomplicated Firewall)

Install ufw
```shell script
apt install ufw
```
Set defaults, deny all incoming connections and allow all outgoing connections:
```shell script
ufw default deny incoming
ufw default allow outgoing
```
Enable port 22 otherwise you exclude yourself
```shell script
ufw allow ssh
```
or
```shell script
ufw allow 22/tcp
```
Enable it and check the rules / status
```shell script
ufw enable
ufw status
```

## Webserver
```shell script
apt install apache2
```
Enable mods:
```shell script
a2enmod ssl
a2enmod macro
a2enmod rewrite
a2enmod headers
a2enmod allowmethods
```
Copy the [security.conf](server/etc/apache2/conf-available/security.conf) configuration and [vhosts-macro.conf](server/etc/apache2/sites-available/vhosts-macro.conf) site into apache folder

Disable the default and enable the vhosts configuration:
```shell script
a2dissite 000-default
a2ensite vhosts-macro
```

Replace at the bottom of [vhosts-macro.conf](server/etc/apache2/sites-available/vhosts-macro.conf) with your domain:
(First parameter is used as ssl folder name, second parameter is ServerName)
```shell script
Use VHost server.de server.de
Use VHost server.de www.server.de
```

Prepare folder structure (replace `server` with your `domain`):
```shell script
mkdir -p /var/www/vhosts/www.server.de/logs
mkdir -p /var/www/vhosts/www.server.de/htdocs
touch /var/www/vhosts/www.server.de/htdocs/index.html
ln -s www.server.de /var/www/vhosts/server.de
chown www-data: /var/www/vhosts/www.server.de
chown www-data: -R /var/www/vhosts/www.server.de/htdocs
```

Clean up:
```shell script
rm -rf /var/www/html
```

Open port 80 in firewall: 
```shell script
ufw allow 80/tcp
```

### SSL

Open port 443 in firewall: 
```shell script
ufw allow 443/tcp
```

Install certbot
```shell script
sudo apt-get install certbot python-certbot-apache
```

Replace server after the domain (-d) parameter with your domain:
```shell script
certbot certonly --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory --manual-public-ip-logging-ok -d '*.server.de' -d server.de
```

Restart service:
```shell script
systemctl restart apache2
```

If everything works, its a good time to save your current settings in a git commit.

## Check your server
https://observatory.mozilla.org/

