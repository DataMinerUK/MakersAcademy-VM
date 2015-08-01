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

  apt-get install -qq wget curl vim



  ##############################################################################
  # Install C Toolchain
  ##############################################################################

  apt-get install -qq build-essential gcc



  ##############################################################################
  # Install PostgreSQL 9.1
  ##############################################################################

  # PostgreSQL is a pain about locale settings. Need to be right before install.
  locale-gen en_US.UTF-8
  update-locale LANG=en_US.UTF-8 LC_ALL=en_GB.UTF-8
  . /etc/default/locale

  apt-get install -qq postgresql postgresql-client
  apt-get install -qq postgresql-contrib postgresql-server-dev-9.1 libpq-dev

  # Set initial PostgreSQL user and password to postgres:password
  sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'password';"

  # Check for vagrant user already created. This user is used in
  # localhost default access from the Vagrant. Need to || true silence the
  # grep so the exit code is zero or the vagrant init will error.
  HAS_VAGRANT_USER=$(sudo -u postgres psql -c "SELECT usename FROM pg_user WHERE usename = 'vagrant'" | grep vagrant || true)

  # Create the vagrant user if it doesn't already exist.
  if [ -z "$HAS_VAGRANT_USER" ]; then
    sudo -u postgres createuser -drS vagrant
  fi

  # Set vagrant user and password to vagrant:password
  sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'password';"

  # bin/rake db:create fails unless the template1 database has UTF-8 encoding.
  sudo -u postgres psql -c "update pg_database set datistemplate=false where datname='template1';"
  sudo -u postgres psql -c "drop database template1;"
  sudo -u postgres psql -c "create database template1 with owner=postgres encoding='UTF-8' lc_collate='en_US.utf8' lc_ctype='en_US.utf8' template template0;"
  sudo -u postgres psql -c "update pg_database set datistemplate=true where datname='template1';"

  # Configure PostgreSQL to listen on the external interface of the Vagrant.
  # This is best done by setting the entire configuration file so that it is
  # updated robustly on reruns of the provisioning. We just want the
  # listen_addresses to be '*' though.
  cat > /etc/postgresql/9.1/main/postgresql.conf <<POSTGRES
data_directory = '/var/lib/postgresql/9.1/main'
hba_file = '/etc/postgresql/9.1/main/pg_hba.conf'
ident_file = '/etc/postgresql/9.1/main/pg_ident.conf'
external_pid_file = '/var/run/postgresql/9.1-main.pid'
listen_addresses = '*'
port = 5432
max_connections = 100
unix_socket_directory = '/var/run/postgresql'
ssl = true
shared_buffers = 24MB
log_line_prefix = '%t '
datestyle = 'iso, mdy'
lc_messages = 'en_US'
lc_monetary = 'en_US'
lc_numeric = 'en_US'
lc_time = 'en_US'
default_text_search_config = 'pg_catalog.english'
POSTGRES

  # Allow user access to PostgreSQL from outside the Vagrant and
  # trust all localhost access from inside the Vagrant.
  cat > /etc/postgresql/9.1/main/pg_hba.conf <<POSTGRES
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             0.0.0.0/0               md5
host    all             all             ::1/128                 md5
host    all             all             127.0.0.1/32            trust
POSTGRES

  # Restart PostgreSQL to pick up the changes
  service postgresql restart



  ##############################################################################
  # Install NGINX
  ##############################################################################

  apt-get install -qq nginx



  ##############################################################################
  # Install NodeJS
  ##############################################################################

  # Install NodeJS if not already installed.
  if [ ! -f /usr/bin/node ]; then
    wget -q http://nodejs.org/dist/v0.12.7/node-v0.12.7-linux-x64.tar.gz
    tar xzf node-v0.12.7-linux-x64.tar.gz --strip-components=1 -C /usr
    rm node-v0.12.7-linux-x64.tar.gz
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
  npm install -g http-server
  npm install -g bower
  npm install -g protractor
  npm install -g karma-cli

  apt-get install -qq imagemagick


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
  # When running you have to do rackup -p4567 --host 0.0.0.0
  config.vm.network :forwarded_port, guest: 4567, host: 4567

  # KarmaJS commonly uses 9876
  config.vm.network :forwarded_port, guest: 9876, host: 9876


end
