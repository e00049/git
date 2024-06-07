#!/usr/bin/env bash

USERNAME="e00049"
PASSWORD="Kubectl@123"
SSH_CONFIG="/etc/ssh/sshd_config"
SSH_CONFIG_DIR="/etc/ssh/sshd_config.d"

adduser --disabled-password --gecos "" $USERNAME
echo "${USERNAME}:${PASSWORD}" | chpasswd
echo "e00049 ALL=(ALL)      NOPASSWD: ALL" >> /etc/sudoers

if grep -q "^PasswordAuthentication" $SSH_CONFIG; then
    sed -i 's/^PasswordAuthentication.*/PasswordAuthentication yes/' $SSH_CONFIG
else
    echo "PasswordAuthentication yes" >> $SSH_CONFIG
fi

grep -r "PasswordAuthentication" $SSH_CONFIG_DIR | while read -r line ; do
    file=$(echo $line | cut -d: -f1)
    if grep -q "^PasswordAuthentication no" $file; then
        sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' $file
    fi
done

systemctl restart ssh

sudo apt update && \
sudo apt install -y curl openssh-server ca-certificates postfix && \
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash && \
sudo apt install gitlab-ce -y && \
PUBLIC_IP=$(curl -H "Metadata-Flavor: Google" http://metadata/computeMetadata/v1/instance/network-interfaces/0/access-configs/0/external-ip) && \
sudo sed -i "/^external_url /s|.*|external_url 'http://${PUBLIC_IP}'|" /etc/gitlab/gitlab.rb  && \
sudo gitlab-ctl reconfigure && \
sudo cat /etc/gitlab/initial_root_password | awk -F': ' '/Password:/ {print $2}' > /tmp/gitlab-admin-passwd.txt

-----------------------------------------------------------------

# Jenkins Installation on Ubuntu Machine:

sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update 
sudo apt-get install jenkins -y 

sudo cat /var/lib/jenkins/secrets/initialAdminPassword > /tmp/jenkins-admin-passwd.txt

-----------------------------------------------------------------------

