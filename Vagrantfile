
# customization section, tweak these parameters if you know what you are doing
$mysql_root_password='root123'
$address_subnet='192.168.222'
$mysql_address='192.168.222.'
$repl_pass='repl123'

$generate_my_cnf = <<SCRIPT0
echo \'[client]\' >> .my.cnf 
echo \'user = repl\' >> .my.cnf
echo \'password = #{$repl_pass}\' >> .my.cnf
SCRIPT0

$mysql_server_script = <<SCRIPT3
#!/bin/bash

#{$generate_my_cnf}

echo mysql-server-5.6 mysql-server/root_password password #{$mysql_root_password} | sudo debconf-set-selections
echo mysql-server-5.6 mysql-server/root_password_again password #{$mysql_root_password} | sudo debconf-set-selections

sudo apt-get install curl -y
sudo apt-get update --fix-missing
sudo apt-get -y --force-yes install mysql-server-5.6 mysql-utilities mysql-client-5.6
sudo apt-get -y --force-yes autoremove
sudo sed -i "s/bind-address.*/bind-address = 0.0.0.0/" /etc/mysql/my.cnf

sudo echo "[mysqld]" >> /etc/mysql/my.cnf
sudo echo "log-bin = mysql-bin" >> /etc/mysql/my.cnf
sudo echo "read_only = ON" >> /etc/mysql/my.cnf
sudo echo "log-slave-updates = ON" >> /etc/mysql/my.cnf
sudo echo "gtid_mode=ON" >> /etc/mysql/my.cnf
sudo echo "enforce-gtid-consistency" >> /etc/mysql/my.cnf
sudo echo "master-info-repository=TABLE" >> /etc/mysql/my.cnf
sudo echo "relay-log-info-repository=TABLE" >> /etc/mysql/my.cnf
sudo echo "binlog-format=row" >> /etc/mysql/my.cnf

#sudo service mysql restart
mysql -B -uroot -p#{$mysql_root_password} -e "create database repl_sandbox;" > /dev/null
mysql -B -uroot -p#{$mysql_root_password} -e "GRANT SUPER, GRANT OPTION, REPLICATION SLAVE, SELECT, RELOAD, DROP, CREATE, INSERT, REPLICATION CLIENT ON *.* TO 'repl'@'%' IDENTIFIED BY '#{$repl_pass}';" > /dev/null
mysql -B -uroot -p#{$mysql_root_password} -e "flush privileges;" > /dev/null

SCRIPT3

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

replication_master=nil
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "trusty64"
  config.vm.box_url = "http://goo.gl/8kWkm"

  # Manage /etc/hosts on host and VMs
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false
  mysql_script=[]  
  71.upto(74) do |sn|
  	mysql_script[sn]=$mysql_server_script.clone()
        hostname="vm-mysql-#{sn}"  
        host_addr="#{$mysql_address}#{sn}"
	if replication_master.nil?
		replication_master=host_addr
		puts "# mysqlfailover --master=repl:#{$repl_pass}@#{replication_master} --discover-slaves-login=root"
	end
	mysql_script[sn] << "sudo sed -i 's/.*server\-id.*/server-id=#{sn}/' /etc/mysql/my.cnf \n"
#	mysql_script[sn] << "sudo service mysql restart\n"
  	config.vm.define "mysql_vm#{sn}" do |config|
  		config.vm.provider :virtualbox do |v|
   			v.name = hostname
     			v.customize ["modifyvm", :id, "--groups", "/MySQL-sandbox", "--memory", "784" ]
   		end
   		config.vm.hostname = hostname
   		config.vm.provision :hostmanager
		if replication_master.eql?(host_addr) # then it is master
			mysql_script[sn] << "sudo sed -i 's/read_only =.*/read_only = OFF/' /etc/mysql/my.cnf \n"
			mysql_script[sn] << "sudo service mysql restart\n"
		else
			mysql_script[sn] << "sudo sed -i 's/read_only =.*/read_only = ON/' /etc/mysql/my.cnf \n"
			mysql_script[sn] << "sudo echo \"relay-log = relay-log-slave\" >> /etc/mysql/my.cnf \n"
			mysql_script[sn] << "sudo service mysql restart\n"
			mysql_script[sn] << "mysql -B -uroot -p#{$mysql_root_password} -e \"CHANGE MASTER TO MASTER_HOST='#{replication_master}',MASTER_PORT=3306,MASTER_USER='repl',MASTER_PASSWORD='repl123',MASTER_AUTO_POSITION=1; \" \n"
			mysql_script[sn] << "mysql -B -uroot -p#{$mysql_root_password} -e \"start slave; \" \n"
		end

		puts "Hostname #{hostname} address #{host_addr} provisioning script in #{hostname}-provision.bash"
		File.open("#{hostname}-provision.bash", 'w') { |file| file.puts(mysql_script[sn]) }
   		config.vm.provision :shell, :inline => mysql_script[sn]

   		config.vm.network :private_network, ip: host_addr 
   		config.vm.network "public_network", type: "dhcp", :adapter=>4, :bridge => 'Intel(R) 82579V Gigabit Network Connection'
  	end
  end
  
end

