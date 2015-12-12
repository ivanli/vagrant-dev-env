# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # You can search for boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ivanli/lubuntu64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "Development"
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true
    # Customize the amount of memory on the VM:
    vb.memory = "4096"
    vb.cpus = 4
    vb.customize ["modifyvm", :id, "--vram", "256"]
    vb.customize ["modifyvm", :id, "--usb", "on"]
    vb.customize ["modifyvm", :id, "--usbehci", "on"]
    vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
    vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]
    vb.customize ["modifyvm", :id, "--accelerate3d", "on"]
    vb.customize ["modifyvm", :id, "--paravirtprovider", "kvm"]

  end

  # Refer to bug #5199. Clearing sync folders help chef shared folders work better.
  config.trigger.after [:halt], stdout: true do
    `rm .vagrant/machines/default/virtualbox/synced_folders`
  end
  config.trigger.before [:up, :reload], stdout: true do
    `rm .vagrant/machines/default/virtualbox/synced_folders`
  end

  # Install apps through apt-get
  config.vm.provision "shell", inline: <<-EOH
    add-apt-repository ppa:gnome3-team/gnome3
    apt-get update

    apt-get install build-essential

    debconf-set-selections <<-EOF
      gdm     shared/default-x-display-manager      select    lightdm
      lightdm shared/default-x-display-manager      select    lightdm
    EOF
    apt-get install ubuntu-gnome-desktop
    DEBIAN_FRONTEND="noninteractive" dpkg-reconfigure lightdm

    apt-get install git -y
    apt-get install giggle -y

    apt-get install wine -y

    apt-get install vim -y

    apt-get install npm -y
    npm install --global glup

    gem install bundler
    gem install rake
  EOH

  # Install apps via cookbooks
  config.vm.provision "chef_solo" do |chef|
    chef.cookbooks_path = "cookbooks"

    chef.add_recipe "java"
    #chef.add_recipe "android-sdk"

    # fix launcher icon installation
    chef.add_recipe "eclipse"

    chef.json = {
      "java" => {
        "install_flavor" => "openjdk",
        "jdk_version" => "7",
        "openjdk_packages" => ["openjdk-7-jdk", "openjdk-7-jre-headless"],
        "set_etc_environment" => true,
        "accept_license_agreement" => true
      },
      "eclipse" => {
        "version" => "luna",
        "release_code" => "SR2",
        "arch" => "x86_64",
        "suite" => "java",
        "os" => "linux-gtk",
        "plugins" => [
          {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.feature.group"},
          {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.managedbuilder.llvm.feature.group"},
          {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.build.crossgcc.feature.group"},
          {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.debug.gdbjtag.feature.group"},
          {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.debug.ui.memory.feature.group"},

          {"http://download.eclipse.org/releases/luna" => "org.eclipse.jdt.feature.group"},
          {"http://download.eclipse.org/releases/luna" => "org.eclipse.m2e.feature.feature.group"},

          {"http://download.eclipse.org/releases/luna" => "org.eclipse.wst.xml_ui.feature.feature.group"},

          {"http://download.eclipse.org/releases/luna" => "org.eclipse.egit.feature.group"},
          {"http://download.eclipse.org/releases/luna" => "org.eclipse.team.svn.feature.group"},
        ]
      },
      "ruby" => {
        "version" => "2.0",
        "gem" => [
          'bundler',
		  'rake'
        ]
      }
    }
  end
end
