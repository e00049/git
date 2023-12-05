#!/usr/bin/env bash

sudo useradd -s /bin/bash -d /home/e00049 -m e00049
sudo echo -e "z\nz" | passwd e00049 > /dev/null 2>&1
echo "e00049 ALL=(ALL)      NOPASSWD: ALL" >> /etc/sudoers
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sed -i 's/PermitRootLogin forced-commands-only/#PermitRootLogin forced-commands-only/g' /etc/ssh/sshd_config
systemctl restart sshd >/dev/null 2>&1

sudo apt update
sudo apt install -y curl openssh-server ca-certificates postfix
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt install gitlab-ce -y
sudo vi /etc/gitlab/gitlab.rb - your server IP address here
sudo gitlab-ctl reconfigure
sudo cat /etc/gitlab/initial_root_password

-----------------------------------------------------------------

# Jenkins Installation on Ubuntu Machine:

sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update 
sudo apt-get install jenkins -y 

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

-----------------------------------------------------------------------

