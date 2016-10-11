# -*- mode: ruby -*-
# vi: set ft=ruby :

#############################
# UTILITIES / SHELL SCRIPTS #
#############################
# Small maintenance script, that sets the system locale / ensures its present.
$FIX_LOCALE = <<SCRIPT
locale-gen
SCRIPT

# Naive script to update the vms, after bringing them up.
$APT_UP = <<SCRIPT
apt-get update -y
apt-get upgrade -y
SCRIPT

# Installs ansible and its requirements (used on the master).
$INSTALL_ANSIBLE = <<SCRIPT
# We need to have a c compiler around, because ansible relies on libraries that
# need to be compiled during install.
# Furthermore we need to install the packages python-dev and libssl-dev because
# they are mandatory for ansible to be installed as well.
apt-get install gcc python-dev libssl-dev libffi-dev -y
wget https://bootstrap.pypa.io/get-pip.py -O /tmp/get-pip.py
chmod +x /tmp/get-pip.py
python /tmp/get-pip.py
pip install ansible
SCRIPT

# Downloads the vagrant private ssh key onto the master.
$DOWNLOAD_VAGRANT_SSH_KEY = <<SCRIPT
wget https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant -O /home/vagrant/.ssh/id_rsa
# Set the permissions for this file!
chmod 400 /home/vagrant/.ssh/id_rsa
# Set the user and group of this file to vagrant!
chgrp vagrant /home/vagrant/.ssh/id_rsa
chown vagrant /home/vagrant/.ssh/id_rsa
SCRIPT

# Accepts the host keys of the gingers on the master!
$ACCEPT_HOST_KEYS_OF_GINGERS = <<SCRIPT
ssh-keyscan -H 192.168.66.67 >> ~/.ssh/known_hosts
ssh-keyscan -H ginger_one >> ~/.ssh/known_hosts
ssh-keyscan -H 192.168.66.68 >> ~/.ssh/known_hosts
ssh-keyscan -H ginger_two >> ~/.ssh/known_hosts
SCRIPT


Vagrant.configure("2") do |config|
  # Use one of the most reliable vagrant boxes around! <3
  # We put this information into a variable here, so that we can easily
  # re-use that information for all our machines, without needing to repeat
  # it.
  # To get an overview on available boxes, just check:
  # https://atlas.hashicorp.com/boxes/search
  base_box_name = "debian/jessie64"

  # In order to easily set up passwordless ssh connections between the master
  # and the two gingers, we need to prevent vagrant from overriding the
  # insecure ssh keys it commonly comes with. This would be an security issues
  # if those vms would be exposed to the internet, but as they are running
  # entirely locally, its perfectly fine.
  config.ssh.insert_key = false

  # Defining the vm, that will be used as the master.
  # Furthermore tell vagrant to make that box the primary box of this setup,
  # so that every call like:
  # $ vagrant up 
  # $ vagrant ssh
  # Will automatically fallback to the master for convenience.
  config.vm.define "master", primary: true do |master|
    # Define which vagrant box shall be used for the master.
    # As we want to keep things ubuntu based for now, we just use the above
    # defined base_box_name variable.
    master.vm.box = base_box_name

    # Set the hostname!
    master.vm.hostname = "master"

    # Share this current folder with the master vm inside the filesystem at
    # /vagrant
    master.vm.synced_folder ".", "/vagrant"

    # Fix eventual happening locale bugs, before they can happen!
    # See the script defined in line 8!
    master.vm.provision "shell", inline: $FIX_LOCALE

    # Update the masters packages by running apt-get update and upgrade inside
    # the vm. See line 13 for the script.
    master.vm.provision "shell", inline: $APT_UP

    # Install ansible and its requirements to the master.
    master.vm.provision "shell", inline: $INSTALL_ANSIBLE

    # Downloads and "installs" the private key for easier connection to the
    # gingers.
    master.vm.provision "shell", inline: $DOWNLOAD_VAGRANT_SSH_KEY

    # Upload the custom hosts file, so that the master can resolve hostnames
    # to ip adresses.
    master.vm.provision "file", source: "config_files/etc_hosts", destination: "/tmp/hosts"
    master.vm.provision "shell", inline: "cp /tmp/hosts /etc/hosts"

    # Upload the ansible inventory file and move it to the correct place
    # afterards.
    master.vm.provision "file", source: "config_files/ansible_hosts", destination: "/tmp/ansible_hosts"
    # Create ansibles config directory.
    master.vm.provision "shell", inline: "mkdir /etc/ansible"
    master.vm.provision "shell", inline: "cp /tmp/ansible_hosts /etc/ansible/hosts"

    # Accept the host keys of those goddam gingers on the master!
    master.vm.provision "shell", inline: $ACCEPT_HOST_KEYS_OF_GINGERS, privileged: false   

    # Assign a static private ip adress to the master for easier access.
    master.vm.network "private_network", ip: "192.168.66.66"

    # Set some virtualbox specific things now.
    master.vm.provider "virtualbox" do |vb|
      # Set the name of the vm, so that we can identify it in the interface of
      # virtualbox.
      vb.name = "ansible_master"
      # Self explanatory.
      vb.cpus = 1
      vb.memory = 1024
    end # end of virtualbox customisations.
  end # end of master definition.

  # Defining the first vm to be used as a slave.
  config.vm.define "ginger_one" do |ginger_one|
    # Set the box.
    ginger_one.vm.box = base_box_name

    # Set the hostname!
    ginger_one.vm.hostname = "ginger-one"

    # Run the fix_locale and udpate script.
    ginger_one.vm.provision "shell", inline: $FIX_LOCALE
    ginger_one.vm.provision "shell", inline: $APT_UP

    # Assign a static private ip adress to ginger_one for easier access. haha.
    ginger_one.vm.network "private_network", ip: "192.168.66.67"

    # Set some virtualbox specific things now.
    ginger_one.vm.provider "virtualbox" do |vb|
      # Set the name of the vm, so that we can identify it in the interface of
      # virtualbox.
      vb.name = "ginger_one"
      # Self explanatory.
      vb.cpus = 1
      vb.memory = 1024
    end # end of virtualbox customisations.
  end # end of ginger_one definition.

  # Defining the second vm to be used as a slave.
  config.vm.define "ginger_two" do |ginger_two|
    # Set the box.
    ginger_two.vm.box = base_box_name

    # Set the hostname!
    ginger_two.vm.hostname = "ginger-two"

    # Run the fix_locale and udpate script.
    ginger_two.vm.provision "shell", inline: $FIX_LOCALE
    ginger_two.vm.provision "shell", inline: $APT_UP

    # Assign a static private ip adress to ginger_two for easier access. haha.
    ginger_two.vm.network "private_network", ip: "192.168.66.68"

    # Set some virtualbox specific things now.
    ginger_two.vm.provider "virtualbox" do |vb|
      # Set the name of the vm, so that we can identify it in the interface of
      # virtualbox.
      vb.name = "ginger_two"
      # Self explanatory.
      vb.cpus = 1
      vb.memory = 1024
    end # end of virtualbox customisations.
  end # end of ginger_two definition.

end 
