$install_script = <<SCRIPT

  set -ex


  ##############################################################################
  # Update VM
  ##############################################################################

  apt-get update -qq

  # GRUB breaks the installation on upgrade so hold it.
  apt-mark hold grub-pc

  apt-get upgrade -qq

  # And apt-mark hold will segmentation fault the next time we provision because
  # this version has a bug holding an already held package. So unhold grub-pc
  # here to avoid that. Then cry a little inside because computers are rubbish.

  apt-mark unhold grub-pc



  ##############################################################################
  # Install Basics
  ##############################################################################

  apt-get install -qq wget curl



  ##############################################################################
  # Install C Toolchain
  ##############################################################################

  apt-get install -qq build-essential gcc



  ##############################################################################
  # Install PostgreSQL
  ##############################################################################

  apt-get install -qq postgresql postgresql-client



  ##############################################################################
  # Install NGINX
  ##############################################################################

  apt-get install -qq nginx



  ##############################################################################
  # Install NodeJS
  ##############################################################################

  # Install NodeJS if not already installed.
  if [ ! -f /usr/bin/node ]; then
    wget -q http://nodejs.org/dist/v0.12.3/node-v0.12.3-linux-x64.tar.gz
    tar xzf node-v0.12.3-linux-x64.tar.gz --strip-components=1 -C /usr
    rm node-v0.12.3-linux-x64.tar.gz
  fi



  ##############################################################################
  # Install Ruby 2.2.2 using RVM
  ##############################################################################

  # Install RVM if not already installed.
  if [ ! -f /usr/local/rvm/bin/rvm ]; then
    curl -sSL https://get.rvm.io | sudo bash
  fi

  # Add Ruby 2.2.2 if not already installed.
  if [ ! -d /usr/local/rvm/rubies/ruby-2.2.2 ]; then
    /usr/local/rvm/bin/rvm install ruby-2.2.2
  fi

  # Make Ruby 2.2.2 the default.
  /usr/local/rvm/bin/rvm alias create default ruby-2.2.2

  # Need to add vagrant user to the rvm group to use RVM in Vagrant.
  usermod -a -G rvm vagrant



  ##############################################################################
  # Install libraries and tools for Makers Academy
  ##############################################################################

  # Source the RVM configuration script to update the running shell to use RVM
  # paths instead of the installed system Ruby, version 1.8.7.

  source /etc/profile.d/rvm.sh

  #
  # gem install without `--no-rdoc --no-ri` breaks with the following message:
  #
  #    unrecognized option `--encoding=UTF-8'
  #

  gem install bundler --no-rdoc --no-ri
  gem install sinatra --no-rdoc --no-ri
  gem install rails --no-rdoc --no-ri

  gem install capybara --no-rdoc --no-ri
  gem install cucumber --no-rdoc --no-ri
  gem install rspec --no-rdoc --no-ri

  npm install -g jasmine
  npm install -g mocha


SCRIPT



Vagrant.configure("2") do |config|

  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  config.ssh.forward_x11 = true

  config.vm.provision "shell" do |s|
    s.inline = $install_script
  end

  config.vm.hostname = "MakersAcademy"
  config.vm.provider :virtualbox do |virtualbox|
    virtualbox.name = "MakersAcademy"
    # virtualbox.gui = true
  end


  ##############################################################################
  # Forward ports from Vagrant to host machine.
  ##############################################################################

  # PostgreSQL runs on port 5432.
  config.vm.network :forwarded_port, guest: 5432, host: 5432

  # Rails default is 3000.
  config.vm.network :forwarded_port, guest: 3000, host: 3000

  # NodeJS commonly uses 8888.
  config.vm.network :forwarded_port, guest: 8888, host: 8888

  # Forward NGINX webserver from 80 to 8000 on host because cannot typically
  # use port 80 on host machines.
  config.vm.network :forwarded_port, guest: 80, host: 8000

  # Sinatra runs on port 4567
  # When running you have to do ruby myapp.rb -o 0.0.0.0
  config.vm.network :forwarded_port, guest: 4567, host: 4567


end
