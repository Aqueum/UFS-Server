# UFS-Server
- Udacity Full Stack - Linux Server Configuration
- [Udacity Full Stack Web Developer Nanodegree](
https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)
- Martin Currie (Aqueum) - 8 July 2017

# Purpose & design
Taking a basline Linux server install and setting it up to host a web app, secured from a number of attack vectors.

# Getting Started
## App URL

## Server IP Address

## Server SSH port

## SSH key
Submitted to reviewer in "Notes to Reviewer"

# Software installed
## Apache 2
`sudo apt-get install apache2 apache2-utils libexpat1 ssl-cert python` to configure prerequisites for mod-wsgi

# Third Party Resorces

# Known issues
## Not built
At time of writing, I'm still in planning stages, hence it probably won't work.

# Files

# Contributing
This is an assessed project, so I'd probably get in trouble for accepting external input.

# Code Status
Can Udacity add a badge here..?

# License
This is an assessed project, but also may be further developed to help a local community interest company,
as such **all rights are reserved**, feel free to [contact me](http://www.aqueum.com/contact/)
if you have any questions.

# What I did
## Set up lightsail
- https://lightsail.aws.amazon.com
- Create instance > OS Only > Ubuntu 16.04 LTS
- Select $5 plan
- Name instance UFS

## Connect through terminal
### Create static IP
- Go to [Networking](https://lightsail.aws.amazon.com/ls/webapp/eu-west-2/instances/UFS/networking)
- Click create static IP
- Name static IP `UFS-Static` & attach to UFS
35.176.170.23 given

### Create key
- In terminal home `ssh-keygen`
- Name it ufs_rsa
- Go to [SSH Keys](https://lightsail.aws.amazon.com/ls/webapp/account/keys)
- click Upload New
- upload ufs_rsa.pub

### log in
- In terminal home `ssh ubuntu@35.176.170.23 -p 22 -i ~/.ssh/ufs_rsa`


## Update all currently installed packages
- `sudo apt-get update`
- `sudo apt-get upgrade`
- `Y`
- I decided to keep the locally modified /boot/grub/menu.lst
- `sudo apt-get dist-upgrade`
- `Y`

## Change SSH port form 22 to 2200
### Configure Lightsail firewall to allow this
- Go to [Networking](https://lightsail.aws.amazon.com/ls/webapp/eu-west-2/instances/UFS/networking)
- click +Add another (under firewall)
- enter Custom TCP 2200 & save

### Change SSH port
- `sudo nano /etc/ssh/sshd_config`
- change `Port 22` line to `Port 2200'
- `sudo service sshd restart`

### Reconfigure Lightsail firewall
- Go to [Networking](https://lightsail.aws.amazon.com/ls/webapp/eu-west-2/instances/UFS/networking)
- click Edit rules (under firewall)
- delete SSH TCP 22
- add Custom UDP 123

### Log back in on port 2200
- In terminal home `ssh ubuntu@35.176.170.23 -p 2200 -i ~/.ssh/ufs_rsa`

## Uncomplicated Firewall (UFW)
([Linode](https://www.linode.com/docs/security/firewalls/configure-firewall-with-ufw))
- logged in as ubuntu:
- `sudo ufw enable` (perhaps I should have left this until last, but I wrongly assumed that the lightsail firewall interface was using UFW)
- `sudo ufw status` (this indicated UFW was active but didn't give the ports table )
- `sudo ufw allow 2200/tcp`
- `sudo ufw allow www`
- `sudo ufw allow 123`
- `sudo ufw status`

output:
`Status: active
 
 To                         Action      From
 --                         ------      ----
 2200/tcp                   ALLOW       Anywhere                  
 80/tcp                     ALLOW       Anywhere                  
 123                        ALLOW       Anywhere                  
 2200/tcp (v6)              ALLOW       Anywhere (v6)             
 80/tcp (v6)                ALLOW       Anywhere (v6)             
 123 (v6)                   ALLOW       Anywhere (v6)  `

## Give grader access
### Generate grader SSH keys
- `ssh-keygen`
- enter file as `/Users/user/.ssh/graderKey`

### New user
- `sudo adduser grader`
- Password stored elsewhere, will be provided to grader in submission comments

### Grant sudo access
([Mitchell Anicas](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart))
- `sudo usermod -aG sudo grader`

### Apply grader SSH keys
- `su - grader` then enter password to switch log in to grader
- `mkdir .ssh` to create ssh directory
- `nano .ssh/authorized_keys` to create and open key file
- paste in contents of graderKey.pub
- `sudo chmod 700 .ssh` & `sudo chmod 644 .ssh/authorized_keys` to set access permissions

### Login as grader
- `exit` & `exit` to log out
- `ssh grader@35.176.170.23 -p 2200 -i ~/.ssh/graderKey`
- enter password

## Set timezone to UTC
([Mitch](https://askubuntu.com/questions/323131/setting-timezone-from-terminal/323163))
- `sudo dpkg-reconfigure tzdata`

## Install Apache with mod-wsgi
([Hitesh Jethva](https://devops.profitbricks.com/tutorials/install-and-configure-mod_wsgi-on-ubuntu-1604-1/))
- `sudo apt install apache2`
- `Y`
- `sudo apt-get install apache2 apache2-utils libexpat1 ssl-cert python` to configure prerequisites for mod-wsgi
- `curl http://localhost` to confirm apache responds
- `sudo apt-get install libapache2-mod-wsgi`
- `Y`
- `sudo /etc/init.d/apache2 restart`

## Install PostgreSQL
([Justin Ellingwood](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps))
- `sudo apt install postgresql-client-common`
- `sudo apt-get update`
- `sudo apt-get install postgresql postgresql-contrib`
- `Y`
- `sudo su - postgres`
- 'psql'
- confirms PostgreSQL 7.5.7 installed
- `\q`
- `exit`

### Do not allow remote connections
- `sudo nano /etc/postgresql/9.1/main/pg_hba.conf`
- confirms only local, 127.0.0.1/32 & ::/128 can connect

## Install Item Catalogue
- `git clone https://github.com/Aqueum/UFS-ItemCatalogue.git`

## Snapshot
- Go to [Snapshots](https://lightsail.aws.amazon.com/ls/webapp/eu-west-2/instances/UFS/snapshot)
- Click Create snapshot
- Named `UFS-system-1500382621552`