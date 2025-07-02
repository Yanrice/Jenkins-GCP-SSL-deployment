# -Jenkins-GCP-SSL-deployment
Welcome to my Jenkins-GCP-SSL-Setup repository! This project showcases a secure Jenkins deployment on Google Cloud Platform (GCP) using Ubuntu 24.04, featuring a master-agent configuration, SSL setup with Certbot, and automated certificate renewal. Built with GCP credits, this setup highlights my skills in DevOps, cloud infrastructure, and system administration.
Agent Setup (agent-setup.sh)
Run the following commands on the agent VM instance:
cd /home/ubuntu
mkdir -p ~/jenkins-agent
chmod 700 ~/jenkins-agent
sudo su - ubuntu
sudo apt-get install -y openjdk-11-jre
sudo mkdir /var/lib/jenkins
sudo chown -R ubuntu:ubuntu /var/lib/jenkins
sudo chmod 700 /var/lib/jenkins
mkdir -p ~/.ssh
ssh-keygen -t ed25519 -f ~/.ssh/jenkins_agent_key -N ""
cat ~/.ssh/jenkins_agent_key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
Copy the public key (~/.ssh/jenkins_agent_key.pub) to the master VM's ~/.ssh/authorized_keys.
Master Setup (master-setup.sh)
Run the following commands on the master VM instance:
sudo apt-get update
sudo apt-get install -y openjdk-11-jre jenkins apache2
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo ufw allow 8080
sudo ufw allow 443
sudo ufw status
Edit firewall rules in GCP Console under "VPC Network > Firewall" to allow TCP ports 8080 and 443 for VM instances.
SSL Configuration with Certbot (ssl-config.sh)
Configure SSL for jenkins-yannickkalukuta.com and www.jenkins-yannickkalukuta.com:
sudo a2enmod proxy proxy_http ssl
sudo systemctl restart apache2
sudo nano /etc/apache2/sites-available/jenkins-yannickkalukuta.com-le-ssl.conf
Update the file with (see apache-config.conf for full content):

Remove DocumentRoot /var/www/html.
Add ProxyPreserveHost On, ProxyPass / http://localhost:8080/, ProxyPassReverse / http://localhost:8080/.
Enable and test:
sudo a2ensite jenkins-yannickkalukuta.com-le-ssl.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
sudo certbot --apache -d jenkins-yannickkalukuta.com -d www.jenkins-yannickkalukuta.com
If DNS issues occur, try:
sudo systemd-resolve --flush-caches
sudo systemctl restart systemd-networkd
nslookup www.jenkins-yannickkalukuta.com 8.8.8.8
sudo certbot --apache -d jenkins-yannickkalukuta.com -d www.jenkins-yannickkalukuta.com --dns-resolver 8.8.8.8
Jenkins Configuration (jenkins-config.sh)
Update Jenkins URL:
sudo nano /var/lib/jenkins/config.xml
Add:
<hudson>
  <hudsonUrl>https://jenkins-yannickkalukuta.com/</hudsonUrl>
</hudson>
Restart Jenkins:
sudo systemctl restart jenkins
Certificate Renewal Cron Job (renew-cron.sh)
Set up automatic renewal:
sudo crontab -e
0 3 * * * /usr/bin/certbot renew --quiet
Apache Configuration (apache-config.conf)
Full SSL virtual host configuration: (Refer to the apache config file)

Notes
Environment: Using Ubuntu 24.04 (noble-amd64-v20250628) on GCP with cloud credits.
Firewall: Adjusted GCP firewall rules to allow TCP 8080 and 443.
DNS: Ensured jenkins-yannickkalukuta.com and www.jenkins-yannickkalukuta.com resolve correctly.
Security: SSH keys generated for master-agent communication; replace <HIDDEN_DOMAIN> with your email.
This setup demonstrates a secure, scalable Jenkins environment suitable for a portfolio. Test thoroughly before deployment!
