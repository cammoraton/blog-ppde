Rough instructions on how to generate boxes

# Prep the base CentOS box
vagrant up centos --no-provision
vagrant ssh centos -c "sudo rpm --import http://passenger.stealthymonkeys.com/RPM-GPG-KEY-stealthymonkeys.asc && \
sudo yum install -y http://passenger.stealthymonkeys.com/rhel/6/passenger-release.noarch.rpm && \
sudo yum install -y http://mirrors.kernel.org/fedora-epel/6/i386/epel-release-6-8.noarch.rpm && \
sudo yum update -y"
vagrant reload centos --no-provision
vagrant package centos --output boxes/centos.box

# Use that box to prep the puppetmaster
vagrant up centos --no-provision
vagrant ssh centos -c "sudo yum install -y httpd \
mod_ssl mod_passenger puppet-server bind"
vagrant package centos --output boxes/puppetmaster.box
vagrant box remove example-CentOS6.5

# And then do the ubuntu box
# Note you have to do ncurses grub install, because ubuntu
# just select /dev/sda1
vagrant up ubuntu --no-provision
vagrant ssh ubuntu -c "sudo curl -o /tmp/puppetlabs-release-precise.deb https://apt.puppetlabs.com/puppetlabs-release-precise.deb && \
sudo dpkg -i /tmp/puppetlabs-release-precise.deb && \
sudo apt-get update -y && \
sudo apt-get install puppet -y && \
sudo apt-get upgrade -y"
vagrant reload ubuntu --no-provision
vagrant package ubuntu --output boxes/ubuntu.box
vagrant box remove example-Ubuntu12.04
