Vagrant.configure("2") do |config|
  # Web Server Configuration
  config.vm.define "webserver" do |web|
    web.vm.box = "ubuntu/bionic64"  # Ubuntu 18.04 base box (can change to another version)
    web.vm.hostname = "webserver.local"
    web.vm.network "private_network", ip: "192.168.56.20"
    web.vm.provider "virtualbox" do |vb|
      vb.name = "WebServer"
      vb.memory = "512"
      vb.cpus = 1
    end
    web.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y apache2
    
      # Install rsyslog for logging
      sudo apt-get install -y rsyslog
      # Enable and start rsyslog service
      sudo systemctl enable rsyslog
      sudo systemctl start rsyslog

      # Configure UFW for Apache
      yes | sudo apt-get install -y ufw
      echo "y" | sudo ufw allow 80/tcp   # Allow HTTP traffic
      echo "y" | sudo ufw allow 443/tcp  # Allow HTTPS traffic 
      echo "y" | sudo ufw allow 22/tcp  # Allow SSH access
      echo "y" | sudo ufw enable
      sudo systemctl enable apache2
      sudo systemctl start apache2

      # Install and configure Apache SSL
      sudo apt-get install -y openssl
      sudo a2enmod ssl
      sudo systemctl restart apache2

      # Export RNG file to prevent warnings
      export RANDFILE=/tmp/.rnd
      
      # Generate and configure self-signed SSL certificate with updated details
      sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     	-keyout /etc/ssl/private/apache-selfsigned.key \
     	-out /etc/ssl/certs/apache-selfsigned.crt \
     	-subj "/C=US/ST=California/L=LosAngeles/O=MyOrganization/OU=WebDepartment/CN=c0904027@mylambton.ca"
     
      # Install Lynis
      sudo apt-get update
      sudo apt-get install -y lynis

      # Run Lynis Audit and Save Output
      sudo lynis audit system --quiet > /vagrant/lynis-web-audit.log

      # Extract Warnings and Suggestions
      grep "warning" /var/log/lynis-report.dat > /vagrant/lynis-web-warnings.log
      grep "suggestion" /var/log/lynis-report.dat > /vagrant/lynis-web-suggestions.log
    SHELL
  end

  # Database Server Configuration
  config.vm.define "dbserver" do |db|
    db.vm.box = "ubuntu/bionic64"
    db.vm.hostname = "dbserver.local"
    db.vm.network "private_network", ip: "192.168.56.21"
    db.vm.provider "virtualbox" do |vb|
      vb.name = "DatabaseServer"
      vb.memory = "1024"
      vb.cpus = 1
    end
    db.vm.provision "shell", inline: <<-SHELL
      # Update and install MySQL
      sudo apt-get update
      sudo apt-get install -y mysql-server
	
      # Install rsyslog for logging
      sudo apt-get install -y rsyslog
      # Enable and start rsyslog service
      sudo systemctl enable rsyslog
      sudo systemctl start rsyslog
      
      # Configure UFW (Uncomplicated Firewall)
      yes | sudo apt-get install -y ufw	
      yes | sudo ufw allow from 192.168.56.20 to any port 3306
      yes | sudo ufw allow 22/tcp  # Allow SSH access
      yes | sudo ufw --force enable
      
      # Start and enable MySQL service
      sudo systemctl enable mysql
      sudo systemctl start mysql
      
      # Add environment variables for database credentials
      echo 'export DB_USER="my_db_user"' >> ~/.bashrc
      echo 'export DB_PASS="my_secure_password"' >> ~/.bashrc
      echo 'export DB_USER="my_db_user"' | sudo tee /etc/profile.d/db_env.sh
      echo 'export DB_PASS="my_secure_password"' | sudo tee -a /etc/profile.d/db_env.sh
      sudo chmod +x /etc/profile.d/db_env.sh
      echo "Environment variables added to .bashrc"

      # Enable encryption for sensitive data directories (Data at Rest)
      sudo apt-get install -y ecryptfs-utils
      sudo mkdir ~/secure_data
      sudo mount -t ecryptfs ~/secure_data ~/secure_data <<< $'passphrase\npassphrase\n2\n2\ny\n\nn\ny\ny\n'

   	
      # Install Lynis
      sudo apt-get update
      sudo apt-get install -y lynis

      # Run Lynis Audit and Save Output
      sudo lynis audit system --quiet > /vagrant/lynis-db-audit.log

      # Extract Warnings and Suggestions
      grep "warning" /var/log/lynis-report.dat > /vagrant/lynis-db-warnings.log
      grep "suggestion" /var/log/lynis-report.dat > /vagrant/lynis-db-suggestions.log
    SHELL
  end
end
