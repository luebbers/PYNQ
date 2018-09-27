# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Base on Ubuntu 16.04
  config.vm.box = "ubuntu/xenial64"

  # Import repository under /pynq
  config.vm.synced_folder ".", "/pynq"

  # Set up VM and disk image for VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    disk_image = File.join(File.dirname(File.expand_path(__FILE__)), 'sdbuild.vdi')
    unless File.exist?(disk_image)
      vb.customize ['createhd', '--filename', disk_image, '--size', 100 * 1024]
    end
    vb.customize ['storageattach', :id, '--storagectl', 'SCSI', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', disk_image]
  end

  # Mount disk image on first boot (provisioning)
  config.vm.provision "shell", inline: <<-SHELL
    parted /dev/sdc mklabel msdos
    parted /dev/sdc mkpart primary 100 100%
    partprobe
    mkfs.xfs /dev/sdc1
    mkdir /sdbuild
    echo `blkid /dev/sdc1 | awk '{print$2}' | sed -e 's/"//g'` /sdbuild   xfs   noatime,nobarrier   0   0 >> /etc/fstab
    mount /sdbuild
    mkdir /sdbuild/petalinux
    mkdir /sdbuild/pynq
    chown -R vagrant:vagrant /sdbuild
  SHELL

  # Install prerequisites
  config.vm.provision "shell", inline: "/bin/bash /pynq/sdbuild/scripts/setup_host.sh"

  # Update VM user environment
  config.vm.provision "shell", inline: <<-SHELL
    cat /root/.profile | grep PATH >> /home/vagrant/.profile
  SHELL

end
