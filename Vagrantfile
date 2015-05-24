# -*- mode: ruby -*-
# vi: set ft=ruby :

# Custom shell scripts for provisioning below
$install_gui_script = <<SCRIPT

echo "Updating apt sources"

if ! grep -q '# Mirror sources' /etc/apt/sources.list; then
  sudo sed -i -e '1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty main restricted universe multiverse' /etc/apt/sources.list
  sudo sed -i -e '1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-updates main restricted universe multiverse' /etc/apt/sources.list
  sudo sed -i -e '1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-backports main restricted universe multiverse' /etc/apt/sources.list
  sudo sed -i -e '1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-security main restricted universe multiverse' /etc/apt/sources.list
  sudo sed -i -e '1i# Mirror sources' /etc/apt/sources.list
  sudo apt-get update
fi


echo "Setting debconf-set-selections"

sudo debconf-set-selections <<EOF
  gdm     shared/default-x-display-manager      select    lightdm
  lightdm shared/default-x-display-manager      select    lightdm      
EOF

echo "Installing lightdm"
sudo DEBIAN_FRONTEND="noninteractive" apt-get install -y lightdm=1.10.5-0ubuntu1
echo "Installing ubuntu-gnome-desktop"
sudo DEBIAN_FRONTEND="noninteractive" apt-get install -y ubuntu-gnome-desktop=0.32
echo "Reconfiguring to lightdm"
sudo DEBIAN_FRONTEND="noninteractive" dpkg-reconfigure lightdm

SCRIPT

$add_eclipse_launcher_script = <<SCRIPT

if [ ! -f /usr/share/applications/eclipse-luna.desktop ]; then
cat > /usr/share/applications/eclipse-luna.desktop <<EOL
  
[Desktop Entry]
Type=Application
Encoding=UTF-8
Name=Eclipse Luna
Comment=IDE for C/C++/java development
Icon=/usr/local/eclipse-luna/icon.xpm
Exec=/usr/local/eclipse-luna/eclipse
Terminal=false
Categories=ide;

EOL
fi

SCRIPT

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # You can search for boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

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

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
    vb.cpus = 4
    vb.customize ["modifyvm", :id, "--vram", "256"]
  end

  # Refer to bug #5199. Clearing sync folders help chef shared folders work better.
  config.trigger.after [:reload, :halt], stdout: true do 
    `rm .vagrant/machines/default/virtualbox/synced_folders`
  end
  config.trigger.before [:up], stdout: true do 
    `rm .vagrant/machines/default/virtualbox/synced_folders`
  end
  
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.

  # Provision base box customisations. This includes faster mirrors, GUI frontend
  config.vm.provision "shell", inline: $install_gui_script

  # Provision C/C++ tools
  config.vm.provision "shell", inline: 'sudo apt-get install -y build-essential'
  
  #
  # Provision Java JDK 8
  #
  config.vm.provision "chef_solo" do |chef|
    chef.cookbooks_path = "cookbooks"
    chef.add_recipe "java"
    chef.json = {
      "java" => {
        "install_flavor" => "openjdk",
        "jdk_version" => "7",
        "openjdk_packages" => ["openjdk-7-jdk", "openjdk-7-jre-headless"],
        "set_etc_environment" => true,
        "accept_license_agreement" => true
      }
    }
  end

  # Provision Eclipse with plugins for C/C++/Java development
  config.vm.provision "chef_solo" do |chef|
    chef.cookbooks_path = "cookbooks"
    chef.add_recipe "eclipse"
    chef.json = {
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
      }
    }
  end
  config.vm.provision "shell", inline: $add_eclipse_launcher_script

  # Provision & configure Emacs
  config.vm.provision "shell", inline: 'sudo apt-get install -y emacs24'
end
