Vagrant.configure("2") do |config|
  # Configure common settings

  # Base box
  config.vm.box = "bento/ubuntu-22.04"
  
  # Configure VirtualBox related settings
  config.vm.provider "virtualbox" do |vb|
    # Set CPU and Memory of the VM
    vb.memory = "512"
    vb.cpus = 2
  end

  # I don't want to check for updates
  config.vm.box_check_update = false
  
  # Disable the default share of the current code directory
  # I don't want the guest to write to make changes to the code
  config.vm.synced_folder ".", "/vagrant", disabled: true
  
  # Common provisioning script for all VMs
  config.vm.provision "shell", inline: "apt update && apt install -y curl git gnupg net-tools"
  
  # Common provisioning script to install NodeJS v18
  # It will be used in frontend and backend VMs
  nodejs_install_script = <<-SHELL
    echo "Installing NodeJS v18"
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
    nvm install 18
    echo "NodeJS version: $(node -v)"
    echo "NPM version: $(npm -v)"
    sudo npm install -g pm2
  SHELL

  # Define DB VM
  config.vm.define "db" do |db|
    db.vm.hostname = "db"
    db.vm.network "private_network", ip: "192.168.56.4"
    db.vm.provision "shell", inline: <<-SHELL
      sudo apt install -y postgresql-common
      sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
      sudo apt -y install postgresql-16
      echo "Start PostgreSQL service"
      systemctl start postgresql

      echo "Enable PostgreSQL to start on boot"
      systemctl enable postgresql

      echo "Set postgres user password to postgres, for demonstration purposes"
      sudo -u postgres psql -c "ALTER ROLE postgres WITH password 'postgres'"

      echo "Create the conduit database owned by postgres"
      sudo -u postgres psql -c "CREATE DATABASE conduit OWNER postgres;"

      echo "Configure PostgreSQL to listen on all interfaces"
      sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/g" /etc/postgresql/16/main/postgresql.conf

      echo "Allow all connections in pg_hba.conf"
      echo "host    all             all             0.0.0.0/0               md5" | sudo tee -a /etc/postgresql/16/main/pg_hba.conf

      echo "Restart PostgreSQL to apply changes"
      systemctl restart postgresql
    SHELL
  end

  # Define backend VM
  config.vm.define "backend" do |backend|
    backend.vm.hostname = "backend"
    backend.vm.network "private_network", ip: "192.168.56.3"
    backend.vm.provision "shell", privileged: false, inline: nodejs_install_script
    backend.vm.provision "shell", privileged: false, inline: <<-SHELL
      echo "Begin installing Backend"
      export NVM_DIR="$HOME/.nvm"
      [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
      export DATABASE_URL=postgresql://postgres:postgres@192.168.56.4:5432/conduit
      export JWT_SECRET=6b350c623d150e8a07d36e963e1416079a7a322e8c04952d674fe607914d7b01
      export NODE_ENV=development
      echo "Cloning the backend repository"
      git clone https://github.com/gothinkster/node-express-realworld-example-app.git $HOME/backend

      echo "Installing dependencies"
      cd $HOME/backend
      npm install

      echo "Generating Prisma client, running migrations, and seeding the database"
      npx prisma generate
      npx prisma migrate deploy && npx prisma db seed
      
      echo "Starting the backend server using pm2"
      pm2 start npm --name "backend" -- start
    SHELL
  end

  # Define frontend VM
  config.vm.define "frontend", autostart: false do |frontend|
    frontend.vm.hostname = "frontend"
    frontend.vm.network "private_network", ip: "192.168.56.2"
    # Copy frontend directory to VM. 
    frontend.vm.provision "shell", inline: "apt install -y nginx"
    frontend.vm.provision "shell", privileged: false, inline: nodejs_install_script
    frontend.vm.provision "shell", privileged: false, inline: <<-SHELL
      echo "Begin installing Frontend"
      export NVM_DIR="$HOME/.nvm"
      [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
      
      echo "Cloning the frontend repository"
      git clone https://github.com/khaledosman/react-redux-realworld-example-app.git $HOME/frontend
      
      echo "Installing dependencies"
      cd $HOME/frontend/
      npm install

      echo "Building the frontend"
      REACT_APP_BACKEND_URL="http://backend.demo.servercare.id:3000/api" npm run build

      echo "Copying the build to /var/www/html to serve it using Nginx"
      sudo mv $HOME/frontend/build/* /var/www/html/ 
    SHELL
  end
end
