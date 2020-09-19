# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  config.vm.box = "glyphrider/centos7"
  config.vm.box_check_update = true
  config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "wildfly"
    vb.gui = false
    vb.memory = "8192"
  end

  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 8443, host: 8443
  config.vm.network "forwarded_port", guest: 9990, host: 9990
  
  config.vm.provision "shell", inline: <<-ROOT
    # yum update -y
    hostnamectl set-hostname wildfly
    echo "*** installing OpenJDK 1.8.0 ***"
    yum install -y java-1.8.0-openjdk-headless # java-1.8.0-openjdk-devel
    echo "*** installing Wildfly 20.0.1.Final ***"
    mkdir -p /opt
    curl -fsSL -o - https://download.jboss.org/wildfly/20.0.1.Final/wildfly-20.0.1.Final.tar.gz | tar -xz -C /opt -f -
    ln -sf /opt/wildfly-20.0.1.Final /opt/wildfly
    echo "*** creating wildfly user ***"
    useradd -M -d /opt/wildfly -U wildfly
    echo "*** fixing file ownership ***"
    chown -R wildfly:wildfly /opt/wildfly/
    echo "*** install systemd tomcat.service ***"
    cp -v /vagrant/wildfly.service /usr/lib/systemd/system/
    systemctl enable wildfly.service
    echo "*** update firewall rules ***"
    firewall-cmd --permanent --new-service-from-file=/vagrant/wildfly.xml --name=wildfly
    firewall-cmd --zone=public --add-service=wildfly --permanent
    firewall-cmd --reload
    sudo su - wildfly -c 'bin/add-user.sh -u vagrant -p vagrant -cw'
    echo "*** start wildfly ***"
    systemctl start wildfly.service
  ROOT
end
