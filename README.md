# 🚀 Jenkins-GCP-SSL-Deployment: Your DevOps Showcase!

Welcome to my Jenkins-GCP-SSL-Setup repository! Dive into this vibrant project where I’ve crafted a rock-solid Jenkins deployment on **Google Cloud Platform (GCP)** using Ubuntu 24.04. Expect a master-agent setup, SSL magic with **Certbot**, and automated certificate renewal all powered by GCP **credits**. This setup flexes my DevOps, cloud infrastructure, and system admin muscles, making it a standout piece for my portfolio. Ready to roll? Let’s get started!

## 🎉 Agent Setup (agent-setup.sh)

* Fire up your agent VM and run these commands to get it rolling:

`cd /home/ubuntu`

`mkdir -p ~/jenkins-agent`

`chmod 700 ~/jenkins-agent`

`sudo su - ubuntu`

`sudo apt-get install -y openjdk-11-jre`

`sudo mkdir /var/lib/jenkins`

`sudo chown -R ubuntu:ubuntu /var/lib/jenkins`

`sudo chmod 700 /var/lib/jenkins`

`mkdir -p ~/.ssh`

`ssh-keygen -t ed25519 -f ~/.ssh/jenkins_agent_key -N ""`

`cat ~/.ssh/jenkins_agent_key.pub >> ~/.ssh/authorized_keys`

`chmod 600 ~/.ssh/authorized_keys`

** 🎯 Pro Tip: Copy the public key (`~/.ssh/jenkins_agent_key.pub`) to the master VM’s `~/.ssh/authorized_keys` for seamless SSH action!

## 🛠️ Master Setup (master-setup.sh)

* Kick off the master VM with these powerhouse commands:

`sudo apt-get update`

`sudo apt-get install -y openjdk-11-jre jenkins apache2`

`sudo systemctl start jenkins`

`sudo systemctl enable jenkins`

`sudo ufw allow 8080`

`sudo ufw allow 443`

`sudo ufw status`

 🔒 Firewall Boost: Head to GCP Console (VPC Network > Firewall), create a rule named allow-jenkins-ports, and allow TCP ports `8080` and `443` for your VMs.

## 🔐 SSL Configuration with Certbot (ssl-config.sh)
Secure your domain (jenkins-yannickkalukuta.com and www.jenkins-yannickkalukuta.com) with these steps:

`sudo a2enmod proxy proxy_http ssl`

`sudo systemctl restart apache2`

`sudo nano /etc/apache2/sites-available/jenkins-yannickkalukuta.com-le-ssl.conf`

* ✨ Update the File (check apache-config.conf for the full recipe):

### Ditch DocumentRoot `/var/www/html`.

#### Add ProxyPreserveHost On, ProxyPass / `http://localhost:8080/`, `ProxyPassReverse` / `http://localhost:8080/`.

* 🔧 Enable and Test:

`sudo a2ensite jenkins-yannickkalukuta.com-le-ssl.conf`

`sudo apache2ctl configtest`

`sudo systemctl restart apache2`

`sudo certbot --apache -d jenkins-yannickkalukuta.com -d www.jenkins-yannickkalukuta.com`

* 🌐 DNS Troubles? Try This:

`sudo systemd-resolve --flush-caches`

`sudo systemctl restart systemd-networkd`

`nslookup www.jenkins-yannickkalukuta.com 8.8.8.8`

`sudo certbot --apache -d jenkins-yannickkalukuta.com -d www.jenkins-yannickkalukuta.com --dns-resolver 8.8.8.8`

## ⚙️ Jenkins Configuration (jenkins-config.sh)

* Tune up Jenkins with this quick config:

`sudo nano /var/lib/jenkins/config.xml`
* 📝 Add This:

`<hudson>`

  `<hudsonUrl>https://jenkins-yannickkalukuta.com/</hudsonUrl>`
  
`</hudson>`
* 🔄 Restart:

`sudo systemctl restart jenkins`

## ⏰ Certificate Renewal Cron Job (renew-cron.sh)

* Keep SSL fresh with an auto-renewal cron job:

`sudo crontab -e`

* 📅 Add This (runs daily at 3 AM):

`0 3 * * * /usr/bin/certbot renew --quiet`

## 📜 Apache Configuration (apache-config.conf)

* Check out the full SSL virtual host config in apache-config.conf:
  
`<IfModule mod_ssl.c>`

    `<VirtualHost *:443>`
  
        `ServerAdmin webmaster@<HIDDEN_DOMAIN>`
  
        `ServerName jenkins-yannickkalukuta.com`
  
       `ServerAlias www.jenkins-yannickkalukuta.com`
  
        `ProxyPreserveHost On`
  
        `ProxyPass / http://localhost:8080/`
  
       ` ProxyPassReverse / http://localhost:8080/`
  
        `ErrorLog ${APACHE_LOG_DIR}/error.log`
  
        `CustomLog ${APACHE_LOG_DIR}/access.log combined`
  
        `Include /etc/letsencrypt/options-ssl-apache.conf`
  
        `SSLCertificateFile /etc/letsencrypt/live/jenkins-yannickkalukuta.com/fullchain.pem`
  
        `SSLCertificateKeyFile /etc/letsencrypt/live/jenkins-yannickkalukuta.com/privkey.pem`
  
    `</VirtualHost>`
  
`</IfModule>`

* Secure, proxy-ready, and Certbot-friendly!
### 🌟 Notes

#### Environment: Rocking **Ubuntu 24.04** (noble-amd64-v20250628) on GCP with free credits!

*Firewall: Unlocked TCP `8080` and `443` via GCP firewall rules.

#### DNS: Verified jenkins-yannickkalukuta.com and www.jenkins-yannickkalukuta.com resolve like champs.
