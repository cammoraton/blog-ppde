Rough instructions on how to generate boxes

# Install the vbguest plugin
vagrant plugin install vagrant-vbguest

# Prep the base CentOS box
vagrant up centos --no-provision
vagrant ssh centos -c "sudo rpm --import http://passenger.stealthymonkeys.com/RPM-GPG-KEY-stealthymonkeys.asc && \
sudo yum install -y http://passenger.stealthymonkeys.com/rhel/6/passenger-release.noarch.rpm && \
sudo yum install -y http://mirrors.kernel.org/fedora-epel/6/i386/epel-release-6-8.noarch.rpm && \
sudo yum update -y"
# Reload doesn't cut it for vbguest detecting stuffs
vagrant halt centos 
vagrant up centos --no-provision
# Clean up packer artifacts and make sure vagrant stuff is happy
vagrant ssh centos -c "echo '' | sudo tee -a /etc/motd && \
sudo sed -i'' s/^\#UseDNS\ yes/UseDNS\ no/g /etc/ssh/sshd_config && \
sudo sed -i'' s/^Defaults\ requiretty/\#Defaults\ requiretty/g /etc/sudoers"

# Clear logs and vagrant modifications
vagrant ssh centos -c "sudo rm -f /etc/sysconfig/network-scripts/ifcfg-eth1 && \
sudo find /var/log -type f -delete && \
sudo rm -f /tmp/* && \
sudo rm -f /root/.bash_history -c && \
history -c"
vagrant package centos --output boxes/centos.box

# Use that box to prep the puppetmaster
vagrant up centos --no-provision
vagrant ssh centos -c "sudo yum install -y httpd \
mod_ssl mod_passenger puppet-server bind && \
sudo chkconfig named off && \
sudo chkconfig httpd off"

# Clear logs and vagrant modifications
vagrant ssh centos -c "sudo rm -f /etc/sysconfig/network-scripts/ifcfg-eth1 && \
sudo find /var/log -type f -delete && \
sudo rm -f /tmp/* && \
sudo rm -f /root/.bash_history -c && \
history -c"

vagrant package centos --output boxes/puppetmaster.box
vagrant box remove example-CentOS6.5

# And then do the ubuntu box
# Note you have to do ncurses grub install, because ubuntu
# just select /dev/sda
vagrant up ubuntu --no-provision
vagrant ssh ubuntu -c "sudo curl -o /tmp/puppetlabs-release-precise.deb https://apt.puppetlabs.com/puppetlabs-release-precise.deb && \
sudo dpkg -i /tmp/puppetlabs-release-precise.deb && \
sudo apt-get update -y && \
sudo apt-get install puppet -y && \
sudo apt-get upgrade -y"
vagrant reload ubuntu --no-provision
vagrant ssh ubuntu -c "sudo perl -i'' -p0e 's/\#VAGRANT-BEGIN.*\#VAGRANT-END\n//s' interfaces && \
sudo find /var/log -type f -delete && \
sudo rm -f /tmp/* && \
sudo rm -f /root/.bash_history -c && \
history -c"

vagrant package ubuntu --output boxes/ubuntu.box
vagrant box remove example-Ubuntu12.04

# Reinitialize but provision
vagrant destroy -f
vagrant up

# Clean them up again
vagrant ssh centos -c "sudo rm -f /etc/sysconfig/network-scripts/ifcfg-eth1 && \
sudo find /var/log -type f -delete && \
sudo rm -rf /var/lib/puppet/reports/* && \
sudo rm -f /tmp/* && \
sudo rm -f /root/.bash_history -c && \
history -c"
vagrant ssh puppet -c "sudo rm -f /etc/sysconfig/network-scripts/ifcfg-eth1 && \
sudo find /var/log -type f -delete && \
sudo chkconfig named off && \
sudo rm -rf /var/lib/puppet/reports/* && \
sudo rm -f /tmp/* && \
sudo rm -f /root/.bash_history -c && \
history -c"
vagrant ssh ubuntu -c "sudo perl -i'' -p0e 's/\#VAGRANT-BEGIN.*\#VAGRANT-END\n//s' interfaces && \
sudo find /var/log -type f -delete && \
sudo rm -rf /var/lib/puppet/reports/* && \
sudo rm -f /tmp/* && \
sudo rm -f /root/.bash_history -c && \
history -c"

rm -f boxes/*.box
vagrant package ubuntu --output boxes/ubuntu.box
vagrant package centos --output boxes/centos.box
vagrant package puppet --output boxes/puppetmaster.box
vagrant box remove example-CentOS6.5
vagrant box remove example-PuppetMaster
vagrant box remove example-Ubuntu12.04
vagrant destroy -f

# And verify
vagrant up
vagrant ssh puppet -c "sudo puppet agent -t"
vagrant ssh centos -c "sudo puppet agent -t"
vagrant ssh ubuntu -c "sudo puppet agent -t"
