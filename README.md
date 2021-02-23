# Easy web and FTP services with AWS

If you need to set up a web and FTP server, here's how to do it with Caddy 
and vsftpd. You might not need vsftpd if you can access via SFTP, using
the ssh service that ships with Ubuntu.


## Instructions

1. Provision and launch new EC2 t2.micro instance with an Ubuntu AMI.

2. Get the WAN IP address for your home Xfinity network.

3. In AWS VPC Console, create or update a "home-network" AWS security group, setting
  the Source for each inbound rule to your current Xfinity WAN address, adding 
  '/32' to the IP address to create /32 CIDR blocks. At a minimum, inbound rules
  should allow TCP on ports 20-22 (ssh and FTP), 64000-64099 (for PASV mode FTP), 
  80 and 443 (HTTP and HTTPS).

4. In AWS EC2 Console, find the new instance. In Action > Security > Change security groups, 
  add the "home-network" group.

5. Log into new EC2 instance, using keypair SSH access.

6. Install Caddy:

```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo apt-key add -
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee -a /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

* The `Caddyfile` will be at `/etc/caddy/Caddyfile`
* The default document root will be at `/usr/share/caddy`

7. Create a password for basic authentication

```
xcaddy hash-passwd
```

Edit the `Caddyfile` from this repo and modify the 'authuser' and 'BASE64HASHEDPASSWORDHERE'
with the user name and the output from `xcaddy hash-passwd`.

Then copy `Caddyfile` to `/etc/caddy/` and restart the `caddy` service.

8. Install vsftpd

```
sudo apt install vsftpd
touch /var/log/vsftpd.log
```

9. Edit `vsftpd.conf` in this repo

Replace 'AWSPUBLICIP' with the public IP address of the AWS instance.

* The `vsftpd.conf` in this repo enables PASV mode on ports 64000-64099.
* The configuration file will be at `/etc/vsftpd.conf`

Copy `vsftpd.conf` from repo to `/etc/` and restart the `vsftpd` service.

10. Add an `sftp` group and user(s)

```
sudo groupadd sftp
sudo useradd -m newuser
sudo passwd newuser
sudo usermod -a -G sftp newuser
```

11. Copy `sshd_config` from repo to `/etc/ssh/` and restart the `ssh` service.

* `sshd_config` restricts users in the `sftp` group to `/home`
* The configuration file for the built-in ssh server will be at `/etc/ssh/sshd_config`
